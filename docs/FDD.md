# Feature Design Document: Order Status Webhook Notifications

## 1. Context and Technical Motivation

Business customers currently poll `GET /orders` to discover order changes. That creates avoidable integration traffic and does not provide the sub-10-second notification experience the customers requested. The OMS has no existing event, queue, or webhook subsystem; order creation and status changes are implemented with Prisma transactions against MySQL.

The feature adds an outbound notification path without making an external HTTP call part of an order transaction. Each eligible event is captured as an immutable snapshot in a MySQL outbox in the same transaction as its source order operation. A separate Node.js process polls that outbox and sends signed HTTPS requests. Failed deliveries are retried and ultimately moved to a separate dead-letter queue (DLQ), where an administrator can request a replay.

The motivation and architectural decisions come from the meeting ([TRANSCRICAO.md, 09:00–09:07, 09:40–09:41, 09:48–09:52]) and the approved feature definitions in [PRD.md](./PRD.md) (F01–F05). The existing application is Node.js 20+, TypeScript, Express, Prisma, and MySQL ([package.json](../package.json), [schema.prisma](../prisma/schema.prisma)).

## 2. Technical Objectives

1. **Keep order writes atomic with eligible event capture.** A committed order operation must have its eligible outbox records; a rollback must leave neither the order-side changes nor an outbox record.
2. **Keep customer HTTP availability outside order transactions.** No customer request is made from the API request or inside the Prisma order transaction.
3. **Meet the agreed discovery cadence.** Run one separate worker that polls every two seconds, claiming at most 10 due records per batch, oldest first.
4. **Send verifiable, bounded requests.** Use HTTPS, the endpoint-specific HMAC-SHA256 secret, a 10-second HTTP timeout, and a 64 KiB serialized-body limit.
5. **Make delivery semantics explicit.** Provide at-least-once delivery with a stable event UUID. The initial send is followed by five retries at the agreed delays; duplicates are possible.
6. **Make failures diagnosable and recoverable.** Persist every HTTP attempt, expose the latest 100 attempts for an endpoint, move exhausted events into a separate DLQ, and record administrator replays.
7. **Reuse existing application conventions.** Use the current module layout, authentication, Zod validation, `AppError` response shape, Pino logger, and Prisma database configuration.

The sub-10-second customer experience is a product objective, not a hard delivery guarantee: the outbox is polled every two seconds, but endpoint latency, timeouts, backlog, and retries can delay successful delivery. The PRD's first-attempt target is at least 95% of eligible events receiving a 2xx response within 10 seconds; the implementation should measure this rather than claim it is guaranteed.

## 3. Scope and Exclusions

### Included

- Customer-scoped configuration of HTTPS webhook endpoints, with status filters, active state, generated endpoint-specific secrets, and secret rotation.
- `order.created` events for endpoints subscribed to `PENDING`.
- `order.status_changed` events for endpoints subscribed to the new order status.
- Transactional MySQL outbox insertion, storing the serialized payload snapshot at source-event time.
- A separate single-worker process, 2-second polling, 10-record batches, oldest-first processing, and a 30-second processing lease.
- HMAC-SHA256 request signing, encrypted secret persistence, endpoint/network destination checks, a 10-second HTTP timeout, and the 64 KiB payload ceiling.
- Six total sends per retry cycle: one initial send followed by five retries after 1 minute, 5 minutes, 30 minutes, 2 hours, and 12 hours.
- An append-only delivery-attempt history and a separate DLQ.
- Authenticated configuration/history APIs and an `ADMIN`-only, auditable DLQ replay API.

### Excluded or Deferred

- **Inbound webhooks:** notifications flow from the OMS to customer endpoints only. ([TRANSCRICAO.md, 09:02–09:03])
- **Email notifications or fallback:** explicitly deferred until delivery impact can be measured. ([TRANSCRICAO.md, 09:37–09:38])
- **Visual customer dashboard:** excluded; this version exposes APIs only. ([TRANSCRICAO.md, 09:39–09:40])
- **Outbound rate limiting:** observe production behavior first and decide later. ([TRANSCRICAO.md, 09:38–09:39])
- **Outbox archival and retention policy:** archival was outside the meeting's scope; approximately 30 days was only an example, not an agreed retention period. ([TRANSCRICAO.md, 09:08])
- **Multiple-worker scaling and global ordering:** the initial design uses one worker. Partitioning/locking and parallel scaling were deferred. ([TRANSCRICAO.md, 09:12–09:13])
- **Exactly-once delivery:** explicitly not provided; consumers deduplicate using `X-Event-Id`. ([TRANSCRICAO.md, 09:24–09:26])
- **Master encryption-key rotation procedure:** endpoint secret rotation is included; rotation of the application encryption key and re-encryption of stored secrets are out of scope for this feature.

### Confirmed FDD Clarifications

The following implementation details were not fully specified by the source documents and were confirmed during preparation of this FDD. They are decisions for this feature, not claims that the meeting or current code had already defined them.

| Topic | Confirmed decision |
|---|---|
| Retry count | One initial send plus five retries; six sends total per cycle. |
| Order creation | Include `order.created` for active endpoints subscribed to `PENDING`. |
| CRUD route family | Use `/customers/:customerId/webhooks`; history and replay retain the paths already specified in the PRD/transcript. |
| Secret rotation | New secret signs immediately; the customer accepts old and new secrets in parallel for 24 hours. |
| Secret at rest | AES-256-GCM, with an external 32-byte master key; master-key rotation is out of scope. |
| Secret generation | Generate 32 cryptographically random bytes and return Base64url without padding once. |
| HMAC header | `X-Signature: sha256=<lowercase-hex-digest>` over the exact UTF-8 request-body bytes. |
| Worker batch | At most 10 records per polling batch. |
| Processing recovery | 30-second lease; expired processing records become eligible again. |
| Payload maximum | 65,536 UTF-8 bytes; oversized payloads are not sent or truncated and go directly to the DLQ. |
| Delivery history | One immutable record per HTTP attempt; the API returns the latest 100 attempts. |
| Response capture | Store HTTP status and at most 65,536 UTF-8 bytes of response body, with a truncation indicator. |
| Endpoint deletion | Soft-delete; retain URL and encrypted secret while outbox or replayable DLQ records need them. |
| Pending deliveries after configuration change | Resolve the endpoint's current URL and secret for each attempt; the original event payload remains unchanged. |
| Retry state | Keep a retryable record in `FAILED` with `nextAttemptAt`; claim due `PENDING`/`FAILED` records. A final failure remains `FAILED` and is moved to DLQ. |
| Status filters | Require at least one unique status on create and patch. |
| New endpoint default | Active by default. |
| Configuration list | Reuse existing `page`/`pageSize` pagination: defaults 1/20, maximum page size 100. |
| DLQ replay | One replay per DLQ record; reset the retry count, retain attempt history and event UUID, and persist the administrator and replay time. A later exhausted cycle creates a new DLQ record. |
| Outbound destination security | Require HTTPS, reject private/loopback/link-local/reserved IP destinations after DNS resolution, pin the validated destination for the connection, and do not follow redirects. |
| Encryption configuration | `WEBHOOK_ENCRYPTION_KEY` is Base64 for exactly 32 bytes and is required by both API and worker processes. |
| Replay response | `202 Accepted` with the original event ID and `PENDING` state. A missing or already replayed DLQ record returns 404. |
| Worker failure codes | Persist `WEBHOOK_DELIVERY_TIMEOUT`, `WEBHOOK_CONNECTION_FAILED`, `WEBHOOK_HTTP_NON_2XX`, and `WEBHOOK_PAYLOAD_TOO_LARGE` as internal attempt/DLQ reason codes. |

### Still Open

- Retention and archival for delivered outbox rows, attempt history, and resolved DLQ records. No duration is specified.
- Whether/when outbound rate limiting is needed.
- A production tracing backend and metrics exporter. The current repository does not show an observability backend.
- Future multiple-worker strategy and any ordering guarantee beyond the single-worker initial design.
- Operational procedure for master-key rotation.
- Per-customer authorization. The current approved behavior allows any authenticated `ADMIN` or `OPERATOR` to manage any customer ID; this is a documented residual risk, not a guarantee of tenant isolation.

## 4. Architecture and Component Overview

### Proposed Components

The codebase does not yet contain webhook implementation files. The following are new components proposed to fit the existing module and entry-point patterns; they are not existing paths.

| Component | Responsibility |
|---|---|
| `src/modules/webhooks/` | Domain module containing webhook routes, controller, service, repository, Zod schemas, signing/encryption helpers, and worker processor. Follow the established domain-module organization. |
| `src/worker.ts` | Separate process entry point. Creates its own Prisma client using the same environment/database configuration, starts the webhook worker, and handles shutdown. The meeting proposed this path. |
| `WebhookService` | Validates and manages endpoint configurations; coordinates secret creation/rotation; provides history and replay operations. |
| `WebhookRepository` | Encapsulates endpoint, outbox, attempt, and DLQ persistence, including operations that receive a transaction client. |
| `publishWebhookEvent(tx, order, fromStatus, toStatus)` (proposed helper) | Reads eligible endpoint configurations and inserts immutable event snapshots using the order transaction client. It performs no network I/O. The meeting proposed this shape for status changes; use the same transaction-bound approach for order creation. |
| Worker processor | Claims due outbox rows, records attempts, sends signed HTTPS requests, schedules retries, and transfers exhausted/oversized events to the DLQ. |
| Existing centralized middleware | Authenticates API callers, validates request schemas, and formats `AppError`, Zod, Prisma, and unexpected errors. |
| MySQL via Prisma | Stores endpoint configuration, outbox records, attempt history, DLQ records, and replay attribution. No new queue service is introduced. |

### Boundaries

- **API boundary:** CRUD, rotation, delivery history, and admin replay are HTTP operations under the existing `/api/v1` prefix.
- **Transaction boundary:** event eligibility lookup and outbox inserts execute in the same Prisma transaction as the order source operation. The transaction does not call the customer.
- **Worker boundary:** an independent process owns polling and outbound delivery. It is not started inside the API server.
- **Persistence boundary:** event payloads are immutable snapshots. Endpoint configuration is read at each delivery attempt, so URL/secret updates affect pending retries; deleting an endpoint disables future enqueueing but preserves its configuration while existing outbox or replayable DLQ work needs it.
- **Delivery guarantee:** a worker crash after a customer receives a request but before success is persisted can cause a duplicate. This is consistent with at-least-once delivery.

### Request/Event Payload

The request body is the persisted event snapshot, serialized once when the source event is enqueued. The following example illustrates the agreed fields; UUIDs and timestamp values are illustrative:

```json
{
  "event_id": "80dd25fd-cc78-4bdb-a955-e6a48ddc5231",
  "event_type": "order.status_changed",
  "timestamp": "2026-11-05T13:40:00.000Z",
  "order_id": "ea1a03e4-f025-48cf-9f22-a4ad177b8e4e",
  "order_number": "ORD-000123",
  "from_status": "PROCESSING",
  "to_status": "SHIPPED",
  "customer_id": "b79554ac-8d35-4d08-9fad-3d3f26bea18",
  "total_cents": 12500
}
```

For `order.created`, `event_type` is `order.created`, `from_status` is `null`, and `to_status` is `PENDING`. Order items are excluded. The body timestamp describes the source event; `X-Timestamp` describes the individual delivery attempt.

## 5. Detailed Flows

### 5.1 Order Creation and Transactional Outbox

1. The authenticated order API validates the creation request through the existing Zod validation middleware.
2. `OrderService.create` starts its existing Prisma transaction and validates the customer/products, computes totals, reserves the order number, creates the order, and writes its initial `PENDING` status history.
3. Before returning from that transaction callback, the webhook enqueue helper loads active, non-deleted endpoints for the order's customer. It selects only endpoints whose `statuses` include `PENDING`.
4. For each match, it generates a UUID, creates the `order.created` payload with `from_status: null`, `to_status: PENDING`, and the order's persisted values, serializes the payload once, and inserts one `PENDING` outbox row for that endpoint.
5. If no endpoint matches, no outbox row is written. If any outbox insert fails, the transaction rejects and rolls back order, status history, inventory/sequence work, and all outbox inserts in that transaction.
6. After commit, the API returns the created order using the existing `201` response convention. It does not wait for webhook delivery.

The `order.created` behavior is included from PRD F02/F02 acceptance criteria and was explicitly confirmed for this FDD; it is not unambiguously stated in the meeting. Order transaction shape is verified in `src/modules/orders/order.service.ts`.

### 5.2 Status Change and Transactional Outbox

1. The authenticated status endpoint validates the order UUID and `toStatus`.
2. `OrderService.changeStatus` begins its existing Prisma transaction, loads the order and items, and checks the current state using the existing order status-transition rules.
3. Invalid transitions fail before enqueueing. Stock debit/replenishment, order update, and `OrderStatusHistory` insertion remain inside this transaction.
4. The enqueue helper receives the same `Prisma.TransactionClient`, order data, `fromStatus`, and `toStatus`. It loads active, non-deleted webhook endpoints for the order's customer and selects only endpoints subscribed to `toStatus`.
5. For each match, it generates an event UUID, serializes an immutable `order.status_changed` snapshot with `from_status`, `to_status`, `order_number`, `customer_id`, `total_cents`, event timestamp, and no order items, then inserts the outbox row as `PENDING`.
6. If the outbox insert fails, the entire order transaction rolls back. The API must not report a successful status change without the corresponding eligible outbox records.
7. Once committed, the API returns the changed order through its existing `200` response path. No outbound HTTP is performed in this request.

The existing transaction, stock handling, status history, and `changeStatus` method are in `src/modules/orders/order.service.ts`; transition rules are in `src/modules/orders/order.status.ts`. The transaction-bound enqueue approach is based on the meeting at 09:40–09:41 and ADR-001.

### 5.3 Worker Poll, Claim, Delivery, and Attempt Recording

1. The independent worker starts with a distinct Prisma client and the same `DATABASE_URL` and `WEBHOOK_ENCRYPTION_KEY` as the API process.
2. Every two seconds, it queries at most 10 eligible rows ordered oldest-first. Eligible rows are `PENDING`, retryable `FAILED` rows whose `nextAttemptAt <= now`, and `PROCESSING` rows whose 30-second lease has expired. Claiming a row changes its state to `PROCESSING` and records `lockedAt`.
3. It creates an attempt-history row with the cycle/attempt numbers and start time before making the request. The worker resolves the endpoint's current URL and secret; a soft-deleted endpoint remains available while its existing outbox/DLQ work is replayable.
4. The worker decrypts the secret with AES-256-GCM and serializes no new body: it sends the exact persisted UTF-8 payload bytes. If the body is greater than 65,536 bytes, it does not send or truncate it; it records `WEBHOOK_PAYLOAD_TOO_LARGE` and transfers the event directly to the DLQ without HTTP retries.
5. For an eligible payload, the worker computes HMAC-SHA256 over the exact body bytes and sends a POST over HTTPS. It includes:
   - `Content-Type: application/json`
   - `X-Event-Id: <stable event UUID>`
   - `X-Webhook-Id: <webhook endpoint UUID>`
   - `X-Timestamp: <current delivery-attempt timestamp in ISO 8601>`
   - `X-Signature: sha256=<lowercase hexadecimal HMAC>`
6. The HTTP request has a 10-second timeout. Any 2xx is success. A 3xx, 4xx, 5xx, connection failure, or timeout is a failed attempt. Redirects are not followed.
7. The worker updates the attempt record with status, response body (up to 65,536 UTF-8 bytes), truncation flag, response time, and failure code where applicable. It never writes request secrets or HMAC keys to logs or history.
8. A 2xx response changes the outbox row to `DELIVERED`, records completion time, clears its processing lease, and retains the attempt record.
9. A worker crash or persistence failure must not mark an event delivered without a persisted 2xx outcome. Once a processing lease expires, the row can be attempted again. If the remote endpoint received the previous request but the OMS did not persist its result, the repeated request is a permitted duplicate; the stable event ID lets the customer deduplicate.

The two-second interval, separate process, oldest-first processing, 10-second timeout, headers, payload, 64 KiB limit, and at-least-once guarantee come from the meeting/PRD, with the batch size, lease, exact signature encoding, and oversized-payload disposition confirmed for this FDD.

### 5.4 Retry Scheduling and DLQ Transfer

Each delivery cycle consists of an initial send and at most five retries:

| Send number | Meaning | Delay after preceding failed send |
|---:|---|---:|
| 1 | Initial send | None |
| 2 | Retry 1 | 1 minute |
| 3 | Retry 2 | 5 minutes |
| 4 | Retry 3 | 30 minutes |
| 5 | Retry 4 | 2 hours |
| 6 | Retry 5 | 12 hours |

For a timeout, connection failure, or non-2xx response, the worker persists the failed attempt and sets the outbox state to `FAILED` with the calculated `nextAttemptAt`. The worker claims it again only when due. Delay is measured from the preceding failed attempt. On failure of send number 6, the worker atomically persists the final attempt, inserts a separate DLQ record with event ID, endpoint ID, immutable payload, failure reason, and timestamp, and leaves the outbox row in terminal `FAILED` state. A DLQ persistence failure must not mark the event delivered or silently discard it; the final failed row remains eligible for DLQ-transfer recovery.

The retry schedule and separate DLQ are from [TRANSCRICAO.md, 09:14–09:19, 09:48] and ADR-003. The initial-send-plus-five-retries interpretation was confirmed for this FDD because the meeting's “five attempts” summary and PRD wording differed.

### 5.5 Administrative DLQ Replay

1. An administrator sends the replay request with a valid OMS Bearer token and `ADMIN` role.
2. The API loads the DLQ record and verifies that it has not already been replayed. A missing or already replayed record returns `404 WEBHOOK_DEAD_LETTER_NOT_FOUND`.
3. In one database transaction, the service changes the original outbox row back to `PENDING`, sets `nextAttemptAt` to now, clears its lease, resets the retry counter for the new cycle, preserves the event UUID and payload, marks the DLQ record replayed, and stores the acting `userId` and replay timestamp.
4. The API responds `202 Accepted` with the original event ID and `PENDING` state. The worker sends the event asynchronously using the same `X-Event-Id`.
5. A second replay of the same DLQ record is not allowed. The attempt history is preserved; if the replay cycle exhausts its retries, a new DLQ record is created for that cycle.
6. If replay persistence fails, neither the outbox reset nor the DLQ audit update commits, and the API does not return success.

The `/admin/webhooks/dead-letter/:id/replay` path, `ADMIN` requirement, and attribution requirement are from [TRANSCRICAO.md, 09:18–09:19, 09:35–09:36] and PRD F05. The one-replay-per-DLQ lifecycle and `202` result were confirmed for this FDD.

## 6. Public Contracts

All routes below are mounted below the existing `/api/v1` prefix, use JSON, and use the existing Bearer JWT authentication. Configuration, rotation, and delivery-history routes accept either existing authenticated role (`ADMIN` or `OPERATOR`); they do not enforce a user-to-customer relationship in this release. Replay requires `ADMIN`. `customerId`, `webhookId`, and DLQ IDs are UUIDs.

### 6.1 Create Webhook

**Method/path:** `POST /api/v1/customers/:customerId/webhooks`

**Purpose:** Create an active endpoint subscribed to one or more order statuses.

**Headers:** `Authorization: Bearer <JWT>`, `Content-Type: application/json`.

Request:

```json
{
  "url": "https://hooks.example.com/orders",
  "statuses": ["SHIPPED", "DELIVERED"]
}
```

Response `201 Created` (the `secret` is returned once only):

```json
{
  "id": "c1ff057b-0647-4e31-bd88-f9498ba39a10",
  "customerId": "b79554ac-8d35-4d08-9fad-3d3f26bea18",
  "url": "https://hooks.example.com/orders",
  "statuses": ["SHIPPED", "DELIVERED"],
  "active": true,
  "secret": "base64url-secret-value",
  "createdAt": "2026-11-05T13:40:00.000Z"
}
```

**Validation/semantics:** URL must be HTTPS and pass outbound destination checks. `statuses` must contain one or more unique values from `PENDING`, `PAID`, `PROCESSING`, `SHIPPED`, `DELIVERED`, `CANCELLED`. The endpoint is active on creation. The system generates a unique 32-byte secret, persists only its AES-256-GCM ciphertext/nonce/tag, and returns the plaintext once. There is no agreed endpoint-count limit.

**Status codes:** `201`; `400` with `WEBHOOK_INVALID_URL` for a non-HTTPS/invalid destination; `400 VALIDATION_ERROR` for malformed body/UUID/status list; `401 UNAUTHORIZED`; persistence and unexpected errors use the existing centralized error handling.

### 6.2 List Customer Webhooks

**Method/path:** `GET /api/v1/customers/:customerId/webhooks`

**Purpose:** List webhook configurations for the route's customer.

Request:

```http
GET /api/v1/customers/b79554ac-8d35-4d08-9fad-3d3f26bea18/webhooks?page=1&pageSize=20
Accept: application/json
```

**Headers:** `Authorization: Bearer <JWT>`.

Response `200 OK`:

```json
{
  "data": [
    {
      "id": "c1ff057b-0647-4e31-bd88-f9498ba39a10",
      "customerId": "b79554ac-8d35-4d08-9fad-3d3f26bea18",
      "url": "https://hooks.example.com/orders",
      "statuses": ["SHIPPED", "DELIVERED"],
      "active": true,
      "createdAt": "2026-11-05T13:40:00.000Z",
      "updatedAt": "2026-11-05T13:40:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 1,
    "totalPages": 1
  }
}
```

The `secret`, ciphertext, nonce, tag, and master key are never returned. Use `page` and `pageSize` query parameters with defaults 1 and 20 and maximum page size 100, matching `src/shared/http/response.ts` and existing list schemas.

**Status codes:** `200`; `400 VALIDATION_ERROR` for invalid route/query values; `401 UNAUTHORIZED`.

### 6.3 Update Webhook

**Method/path:** `PATCH /api/v1/customers/:customerId/webhooks/:webhookId`

**Purpose:** Change URL, subscribed statuses, and/or active state.

**Headers:** `Authorization: Bearer <JWT>`, `Content-Type: application/json`.

Request:

```json
{
  "url": "https://hooks.example.com/v2/orders",
  "statuses": ["PROCESSING", "SHIPPED"],
  "active": true
}
```

Response `200 OK`:

```json
{
  "id": "c1ff057b-0647-4e31-bd88-f9498ba39a10",
  "customerId": "b79554ac-8d35-4d08-9fad-3d3f26bea18",
  "url": "https://hooks.example.com/v2/orders",
  "statuses": ["PROCESSING", "SHIPPED"],
  "active": true,
  "updatedAt": "2026-11-05T13:45:00.000Z"
}
```

**Validation/semantics:** The body is a non-empty partial object containing only `url`, `statuses`, or `active`. If supplied, URL and statuses use the create validation rules; status arrays cannot be empty or contain duplicates. The secret is not returned. Pending attempts use the updated URL/secret, while their payload remains the original snapshot.

**Status codes:** `200`; `400 WEBHOOK_INVALID_URL` or `400 VALIDATION_ERROR`; `404 WEBHOOK_NOT_FOUND`; `401 UNAUTHORIZED`.

### 6.4 Delete Webhook

**Method/path:** `DELETE /api/v1/customers/:customerId/webhooks/:webhookId`

**Purpose:** Stop future event enqueueing for an endpoint.

Request:

```http
DELETE /api/v1/customers/b79554ac-8d35-4d08-9fad-3d3f26bea18/webhooks/c1ff057b-0647-4e31-bd88-f9498ba39a10
```

**Headers:** `Authorization: Bearer <JWT>`.

Response: `204 No Content`.

Deletion is logical, not immediate physical removal. It sets the endpoint inactive/deleted so it is excluded from new event selection. Its URL and encrypted secret remain available while existing outbox rows or replayable DLQ records require them.

**Status codes:** `204`; `404 WEBHOOK_NOT_FOUND`; `401 UNAUTHORIZED`.

### 6.5 Rotate Webhook Secret

**Method/path:** `POST /api/v1/customers/:customerId/webhooks/:webhookId/secret/rotate`

**Purpose:** Generate a new endpoint secret.

Request:

```http
POST /api/v1/customers/b79554ac-8d35-4d08-9fad-3d3f26bea18/webhooks/c1ff057b-0647-4e31-bd88-f9498ba39a10/secret/rotate
```

**Headers:** `Authorization: Bearer <JWT>`.

Response `200 OK` (new secret returned once):

```json
{
  "webhookId": "c1ff057b-0647-4e31-bd88-f9498ba39a10",
  "secret": "new-base64url-secret-value",
  "oldSecretValidUntil": "2026-11-06T13:45:00.000Z"
}
```

The new secret signs requests immediately. The client must accept both secrets for the 24-hour overlap; only the new secret is returned by this response. List, read, and update responses never expose a secret.

**Status codes:** `200`; `404 WEBHOOK_NOT_FOUND`; `401 UNAUTHORIZED`.

### 6.6 List Delivery Attempts

**Method/path:** `GET /api/v1/webhooks/:webhookId/deliveries`

**Purpose:** Return the latest 100 HTTP attempts for one webhook endpoint.

Request:

```http
GET /api/v1/webhooks/c1ff057b-0647-4e31-bd88-f9498ba39a10/deliveries
Accept: application/json
```

**Headers:** `Authorization: Bearer <JWT>`.

Response `200 OK`:

```json
{
  "data": [
    {
      "eventId": "80dd25fd-cc78-4bdb-a955-e6a48ddc5231",
      "eventType": "order.status_changed",
      "cycle": 1,
      "attempt": 2,
      "outcome": "FAILED",
      "requestPayload": {
        "event_id": "80dd25fd-cc78-4bdb-a955-e6a48ddc5231",
        "event_type": "order.status_changed",
        "timestamp": "2026-11-05T13:40:00.000Z",
        "order_id": "ea1a03e4-f025-48cf-9f22-a4ad177b8e4e",
        "order_number": "ORD-000123",
        "from_status": "PROCESSING",
        "to_status": "SHIPPED",
        "customer_id": "b79554ac-8d35-4d08-9fad-3d3f26bea18",
        "total_cents": 12500
      },
      "httpStatus": 503,
      "responseBody": "temporarily unavailable",
      "responseTruncated": false,
      "durationMs": 37,
      "failureCode": "WEBHOOK_HTTP_NON_2XX",
      "attemptedAt": "2026-11-05T13:42:00.000Z"
    }
  ]
}
```

The endpoint returns at most 100 records ordered by `attemptedAt` descending. Response bodies are capped at 65,536 UTF-8 bytes; `responseTruncated` indicates truncation. No signing secret or signature is included.

**Status codes:** `200`; `404 WEBHOOK_NOT_FOUND`; `401 UNAUTHORIZED`.

### 6.7 Replay a Dead-Letter Record

**Method/path:** `POST /api/v1/admin/webhooks/dead-letter/:id/replay`

**Purpose:** Requeue one exhausted event for asynchronous delivery.

Request:

```http
POST /api/v1/admin/webhooks/dead-letter/6060b3e0-e9c4-48b4-9f31-d07b1d1606b7/replay
```

**Headers:** `Authorization: Bearer <ADMIN JWT>`.

Response `202 Accepted`:

```json
{
  "eventId": "80dd25fd-cc78-4bdb-a955-e6a48ddc5231",
  "status": "PENDING"
}
```

The request has no body. Replay retains the immutable payload and event UUID, begins a new retry cycle, and records the acting administrator. A missing or already replayed DLQ record is returned as 404.

**Status codes:** `202`; `401 UNAUTHORIZED`; `403 FORBIDDEN`; `404 WEBHOOK_DEAD_LETTER_NOT_FOUND`; persistence failures use the centralized error path.

### Shared Error Response Shape

Feature API errors use the existing `AppError` envelope:

```json
{
  "error": {
    "code": "WEBHOOK_NOT_FOUND",
    "message": "Webhook not found"
  }
}
```

The existing middleware also emits generic `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, and `INTERNAL_SERVER_ERROR` codes where applicable. Do not promise custom messages for those shared errors.

## 7. Data Model and Persistence

No webhook model or webhook migration exists in the current repository. The following model names and fields are proposed to implement the confirmed behavior. UUID primary keys follow the Prisma/MySQL convention in `prisma/schema.prisma` (`String`, UUID, `CHAR(36)`).

### 7.1 `WebhookEndpoint` (proposed table: `webhook_endpoints`)

| Field | Type/constraint | Purpose |
|---|---|---|
| `id` | UUID, primary key | Endpoint identifier exposed as `X-Webhook-Id`. |
| `customerId` | UUID, FK to `Customer.id`, required | Customer whose orders may produce notifications. |
| `url` | Text, required | Current HTTPS destination. |
| `statuses` | JSON array of `OrderStatus`, required, non-empty, unique | Filters enqueueing for creation/status changes. |
| `active` | Boolean, default `true` | Controls new event enqueueing. |
| `secretCiphertext` | Text, required | AES-256-GCM ciphertext; never returned. |
| `secretNonce` | Binary/Base64 storage, required | Unique nonce for this encrypted secret. |
| `secretAuthTag` | Binary/Base64 storage, required | AES-GCM authentication tag. |
| `createdAt`, `updatedAt` | DateTime | Configuration timestamps. |
| `deletedAt` | Nullable DateTime | Soft-delete marker; excludes endpoint from new enqueueing. |

The endpoint list should be queried by `(customerId, active, deletedAt)`. The secret is generated using `node:crypto.randomBytes(32)`, encoded as Base64url without padding for the one-time API response, and encrypted before persistence. `WEBHOOK_ENCRYPTION_KEY` must decode from Base64 to exactly 32 bytes. Both API and worker require the same key. Nonces must be generated independently for each encryption. Do not store or log plaintext secrets.

### 7.2 `WebhookOutbox` (proposed table: `webhook_outbox`)

One row represents one event delivered to one endpoint. Its UUID is the stable `event_id`.

| Field | Type/constraint | Purpose |
|---|---|---|
| `eventId` | UUID, primary key | Stable event ID and `X-Event-Id`. |
| `webhookId` | UUID, FK to endpoint | Delivery destination configuration. |
| `orderId` | UUID value, not cascading FK | Source order identifier; payload remains useful even if order deletion occurs later. |
| `eventType` | VarChar/string, required | `order.created` or `order.status_changed`. |
| `payload` | Text, required, immutable | Exact serialized JSON body signed and sent on every attempt. |
| `status` | Enum: `PENDING`, `PROCESSING`, `FAILED`, `DELIVERED` | Delivery lifecycle. |
| `attemptsMade` | Integer, default `0` | Number of HTTP sends in the current cycle; initial send is 1 once started, maximum 6 per cycle. |
| `cycle` | Integer, default `1` | Increments on administrative replay; event UUID is unchanged. |
| `nextAttemptAt` | Nullable DateTime | Retry due time; set after a failed send. |
| `lockedAt` | Nullable DateTime | Start of the 30-second processing lease. |
| `lastFailureCode` | Nullable string | Most recent internal reason code. |
| `createdAt`, `updatedAt` | DateTime | Source enqueue and lifecycle timestamps. |
| `deliveredAt` | Nullable DateTime | Successful delivery timestamp. |

Recommended indexes: `(status, nextAttemptAt, createdAt)` for polling/lease recovery; `(webhookId, createdAt)` for endpoint history and operations. `payload` is stored as serialized text rather than a mutable order reference so the exact signed body is preserved.

### 7.3 `WebhookDeliveryAttempt` (proposed table: `webhook_delivery_attempts`)

One row records each individual HTTP attempt; attempt rows are retained across replay cycles.

| Field | Type/constraint | Purpose |
|---|---|---|
| `id` | UUID, primary key | Attempt record identifier. |
| `eventId` | UUID, FK to outbox | Stable event reference. |
| `cycle`, `attempt` | Integers, required | Retry cycle and send number (1–6). |
| `outcome` | Enum/string: `IN_PROGRESS`, `SUCCEEDED`, `FAILED` | Attempt lifecycle. |
| `startedAt`, `completedAt` | DateTime | Start/end timestamps. |
| `durationMs` | Nullable integer | HTTP attempt duration. |
| `httpStatus` | Nullable integer | Received status; null for timeout/connection failures. |
| `responseBody` | Nullable text, capped at 65,536 UTF-8 bytes | Captured response body. |
| `responseTruncated` | Boolean, default `false` | Whether the response body was truncated. |
| `failureCode` | Nullable string | Confirmed internal `WEBHOOK_*` reason code. |

Recommended uniqueness constraint: `(eventId, cycle, attempt)`. Recommended index: `(eventId, startedAt)` to retrieve the latest 100 attempts by endpoint through the outbox relation. Attempt history includes payload through its immutable outbox reference; it does not duplicate or mutate the secret.

### 7.4 `WebhookDeadLetter` (proposed table: `webhook_dead_letter`)

One row is created when a retry cycle is exhausted or an oversized payload is rejected. Each record is replayable once.

| Field | Type/constraint | Purpose |
|---|---|---|
| `id` | UUID, primary key | DLQ record identifier used by the replay path. |
| `eventId` | UUID, FK to outbox | Stable event UUID; replay uses the same value. |
| `webhookId` | UUID, FK to endpoint | Endpoint required for replay. |
| `cycle` | Integer, required | Exhausted cycle number. |
| `payload` | Text, required | Immutable payload evidence for investigation/replay. |
| `failureCode`, `failureReason` | Required string/text | Final failure reason. |
| `createdAt` | DateTime | DLQ insertion time. |
| `replayedAt` | Nullable DateTime | One-time replay marker. |
| `replayedByUserId` | Nullable UUID, FK to `User.id` | Administrator attribution. |

Recommended uniqueness constraint: `(eventId, cycle)`. Recommended indexes: `(webhookId, createdAt)` and `(replayedAt, createdAt)` for DLQ operations. Replay updates the existing outbox event and marks the DLQ record in one transaction. If a later replay cycle exhausts, it creates a new DLQ row with the incremented cycle.

### 7.5 Transaction and Lifecycle Invariants

- Endpoint configuration lookup and outbox insert participate in the source order transaction.
- A committed source operation cannot lack an eligible event because an outbox write failed; any failure aborts the transaction.
- A rolled-back source operation cannot leave an outbox row.
- A persisted payload is immutable and is never reconstructed from a later order state.
- Attempt, outbox completion, and DLQ transitions must be persisted without falsely reporting success.
- Endpoint deletion is logical while outbox/DLQ records may need its current URL and encrypted secret. Cleanup and long-term retention are not defined by this feature.
- Replay updates the outbox and audit fields in the same transaction; an API success cannot be returned if either write fails.

## 8. Error Matrix

The API error envelope and generic codes follow `src/shared/errors/app-error.ts` and `src/middlewares/error.middleware.ts`. Feature-specific codes use the required `WEBHOOK_` prefix. Worker reason codes are persisted in attempt/DLQ records; they are not HTTP errors returned to the customer API caller.

| Scenario | Code | Condition | Response/behavior | Recovery |
|---|---|---|---|---|
| Invalid/non-HTTPS URL or prohibited resolved address | `WEBHOOK_INVALID_URL` | Endpoint creation/update fails URL or destination checks. | API returns 400; no configuration change is committed. | Correct the URL and retry. |
| Endpoint absent or not available to the route | `WEBHOOK_NOT_FOUND` | Update/delete/rotation/history references an unavailable endpoint. | API returns 404. | Verify customer and endpoint IDs. |
| DLQ record absent or already replayed | `WEBHOOK_DEAD_LETTER_NOT_FOUND` | Replay target cannot be replayed. | API returns 404; no outbox/audit change. | Select an unreplayed DLQ record. |
| Invalid body/query/UUID/status enum | Existing `VALIDATION_ERROR` | Existing Zod validation rejects input. | API returns 400 with the shared validation envelope. | Correct request fields. |
| Missing/invalid JWT | Existing `UNAUTHORIZED` | Request lacks an accepted Bearer token. | API returns 401. | Authenticate and retry. |
| Non-ADMIN attempts replay | Existing `FORBIDDEN` | Authenticated caller lacks `ADMIN`. | API returns 403; no replay is recorded. | An administrator must perform the action. |
| Destination times out | `WEBHOOK_DELIVERY_TIMEOUT` | No response within 10 seconds. | Attempt is failed; `nextAttemptAt` follows the retry schedule. | Retry; after the fifth retry, move to DLQ. |
| DNS/connect/TLS connection fails | `WEBHOOK_CONNECTION_FAILED` | HTTPS request cannot establish a valid connection. | Attempt is failed; no success is reported. | Retry under the same schedule; eventual DLQ. |
| Destination returns 3xx/4xx/5xx | `WEBHOOK_HTTP_NON_2XX` | Response is not 2xx; redirects are not followed. | Record status/body and schedule retry. | Retry; eventual DLQ. |
| Serialized event body exceeds 65,536 UTF-8 bytes | `WEBHOOK_PAYLOAD_TOO_LARGE` | Size is checked before sending. | Do not send or truncate; record in DLQ immediately, without HTTP retry. | Investigate payload design; replay does not reduce its size. |
| Source transaction's outbox insert fails | No new public feature code specified | Prisma persistence error inside order transaction. | Transaction rejects; the source order operation and event writes roll back; existing error middleware formats the API response. | Retry the order operation after the underlying persistence issue is resolved. |
| Worker cannot persist attempt/result/DLQ | Operational persistence error; no feature HTTP code | Database operation fails. | Log explicitly; never mark delivered; leave the row recoverable through state/lease/DLQ-transfer recovery. | Worker retries persistence/recovery; alert operations. |
| Replay transaction fails | Existing centralized persistence/error handling | Outbox reset or audit update cannot commit. | Roll back both updates; do not return 202. | Retry only after persistence recovery. |

## 9. Resilience and Security

### Delivery and Consistency

- MySQL transactional outbox avoids a distributed transaction between the OMS and customer.
- One single worker polls oldest-first; there is no global ordering promise. Per-order ordering is expected only under this single-worker design.
- The 30-second lease exceeds the 10-second HTTP timeout. An expired `PROCESSING` record may be retried after worker restart or crash.
- A timeout or crash after a remote receiver accepted the request but before the OMS recorded success can cause a duplicate. Preserve the same event UUID and document customer-side deduplication.
- Retry schedule: initial request plus five retries, delays measured after each preceding failed send: 1m, 5m, 30m, 2h, 12h. After the final failure, persist to the separate DLQ.
- Oversized payloads are terminal for the current delivery and go directly to DLQ; retrying the unchanged payload cannot make it smaller.
- Outbound rate limiting and archival are not included. Pending growth and database usage must be observed.

### Request Authentication and Secret Handling

- Use HMAC-SHA256 over the exact UTF-8 bytes sent in the request body. Send `X-Signature: sha256=<lowercase hex>`.
- Use one independently generated 32-byte secret per endpoint. Return it once on creation/rotation; never expose it in list, update, error, log, or history responses.
- Store secret ciphertext, nonce, and authentication tag using AES-256-GCM. The 32-byte master key comes from `WEBHOOK_ENCRYPTION_KEY` (Base64) and is not stored in MySQL or logs.
- API and worker must receive the same key. Startup configuration validation must fail explicitly if it is missing, malformed, or not exactly 32 decoded bytes.
- On rotation, new deliveries immediately use the new secret. The customer is responsible for accepting the old and new values in parallel until 24 hours after rotation. The new value is returned only once.
- Master-key rotation/re-encryption is not designed here and must be planned separately before changing the configured key.
- Use TLS certificate verification. Require `https:` URLs, resolve destinations before every connection, reject loopback/private/link-local/reserved IPv4 and IPv6 addresses, connect to the validated resolved address, and do not follow redirects. This prevents a redirect or DNS change between validation and connection from bypassing the destination policy.
- All configuration and history routes still allow any authenticated `ADMIN` or `OPERATOR` to specify any customer ID. This is the explicitly accepted residual access-scope risk in PRD F01; do not describe the API as tenant-isolated.

### Availability and Data Growth

- API availability does not depend on a customer endpoint; event capture is local database work in the existing transaction.
- Worker downtime accumulates eligible outbox rows for later processing. The 2-second poll cadence resumes when the worker is healthy.
- DLQ replay retains the original event UUID and payload, resets the retry cycle, and records the administrator.
- No retention period or archive/delete job is specified for attempts, delivered outbox rows, or resolved DLQ records. Storage growth remains an operational risk and a follow-up decision.

## 10. Observability

The codebase already uses Pino through `src/shared/logger/index.ts` and request IDs through `src/middlewares/request-logger.middleware.ts`. No metrics exporter or tracing implementation was found in the repository. The items below distinguish existing behavior from proposed instrumentation.

### Logs

Use the existing Pino logger. Emit structured events for worker start/stop, poll start/completion, claim/lease recovery, delivery success/failure, retry scheduling, DLQ insertion/replay, and persistence failures. Useful fields include `requestId` (API), `eventId`, `webhookId`, `customerId`, `orderId`, `eventType`, cycle, attempt number, duration, HTTP status, failure code, and retry due time.

Never log the plaintext secret, master key, HMAC value, Authorization header, full request payload, or unbounded customer response body. Log response size/truncation and status instead of copying sensitive response content. The existing logger redacts authorization/cookie/password/token fields; webhook-specific secret fields must also be excluded from log objects.

### Metrics (proposed; no metrics backend is present)

| Metric | Measurement |
|---|---|
| `webhook_outbox_pending` | Current count of due/pending/processing records and age of oldest due record. |
| `webhook_delivery_attempts_total` | Attempts grouped by success/failure code and endpoint outcome. Avoid unbounded customer identifiers as metric labels. |
| `webhook_delivery_duration_seconds` | Time spent in outbound HTTP attempts. |
| `webhook_first_success_latency_seconds` | Source event creation to first successful 2xx, for the PRD's 95%/10-second target. |
| `webhook_dlq_records_total` | DLQ entries grouped by failure code. |
| `webhook_worker_poll_lag_seconds` | Time since the last successful poll / age of oldest eligible event. |
| `webhook_payload_rejected_total` | Payloads sent directly to DLQ for exceeding 64 KiB. |

Metrics names and the backend/exporter are proposals; the current code does not establish a metrics convention or dependency.

### Tracing (proposed; not currently implemented)

The repository does not show distributed tracing. A future tracing implementation should correlate the order API transaction, outbox event, worker attempt, and replay using `requestId`, `eventId`, and `webhookId`, without recording secrets or event bodies. Selecting an instrumentation library/backend is an open implementation decision; this FDD does not add an unapproved tracing dependency.

## 11. Integration with the Existing System

The following existing paths were verified in the repository. New webhook paths mentioned in this section are proposed, not existing files.

| Existing file | Verified behavior | Webhook integration |
|---|---|---|
| [`src/modules/orders/order.service.ts`](../src/modules/orders/order.service.ts) | `create` and `changeStatus` use Prisma interactive transactions; `changeStatus` updates stock, order status, and history. | Call the transaction-bound enqueue helper in `create` for `order.created` and in `changeStatus` for `order.status_changed`. Pass the current transaction client; do not make HTTP calls here. Any eligible outbox insert failure must abort the transaction. |
| [`src/modules/orders/order.status.ts`](../src/modules/orders/order.status.ts) | Defines valid `OrderStatus` transitions and the stock debit/replenishment rules. | Use the resulting persisted status as the subscription filter for status-change events. Do not duplicate or alter the state machine in the webhook module. |
| [`prisma/schema.prisma`](../prisma/schema.prisma) | Defines MySQL models, UUID `CHAR(36)` identifiers, `OrderStatus`, `UserRole`, and customer/order relations. | Add the proposed endpoint/outbox/attempt/DLQ Prisma models and indexes in a future implementation. Store status subscriptions as JSON, matching the existing use of a JSON field for customer address; no webhook models currently exist. |
| [`src/routes/index.ts`](../src/routes/index.ts) | Composes domain routers under their resource prefixes. | Add a webhook router and controller dependency to the API router composition. Preserve the customer-scoped CRUD paths and separate admin replay path. |
| [`src/app.ts`](../src/app.ts) | Builds controllers, mounts the API router at `/api/v1`, configures JSON parsing/request logging, and installs centralized error middleware. | Wire webhook repository/service/controller into `buildControllers`, add the router, and keep it before the not-found/error handlers. The worker is not initialized here. |
| [`src/middlewares/auth.middleware.ts`](../src/middlewares/auth.middleware.ts) | Provides Bearer JWT authentication and `requireRole`; roles are `ADMIN` and `OPERATOR`. | Apply `authenticate` to webhook API routes and `requireRole('ADMIN')` to replay. Configuration/history retain the approved authenticated-user behavior without customer ownership checks. |
| [`src/middlewares/error.middleware.ts`](../src/middlewares/error.middleware.ts) | Formats `AppError`, Zod, and known Prisma errors and logs unexpected errors with request context. | Throw `AppError` subclasses with `WEBHOOK_*` codes for feature errors; let existing middleware format responses. Do not add a competing response envelope. |
| [`src/shared/errors/app-error.ts`](../src/shared/errors/app-error.ts) and [`src/shared/errors/http-errors.ts`](../src/shared/errors/http-errors.ts) | `AppError` carries status/code/details; shared subclasses use the current 400/401/403/404/409/422 conventions. | Implement webhook-specific errors using the same base/error response pattern and approved `WEBHOOK_` codes. Keep generic auth/validation errors unchanged. |
| [`src/shared/http/response.ts`](../src/shared/http/response.ts) | Defines `{data, pagination}` responses for paginated lists. | Use the existing pagination shape for `GET /customers/:customerId/webhooks`; history has its fixed latest-100 contract and no pagination. |
| [`src/shared/logger/index.ts`](../src/shared/logger/index.ts) | Creates the shared Pino logger with redaction paths and environment-based log level. | Reuse the logger in the worker and API; add webhook secret fields to redaction/exclusion rules before logging related objects. |
| [`src/config/env.ts`](../src/config/env.ts) | Validates process environment with Zod and exits on invalid required configuration. | Add `WEBHOOK_ENCRYPTION_KEY` validation for Base64 encoding and exactly 32 decoded bytes; fail startup when invalid. |
| [`src/config/database.ts`](../src/config/database.ts) | Exposes a Prisma-client factory and an API-process client using `DATABASE_URL`. | The worker invokes the factory to create its own process-local Prisma client; it must disconnect on shutdown. |
| [`src/server.ts`](../src/server.ts) and [`package.json`](../package.json) | Existing API entry point has bootstrap/shutdown handling; scripts include build/start and Node `>=20`. | Add a distinct `src/worker.ts` build entry/output and a `worker` package script. The worker owns its own Prisma client and signal/error handling; no additional queue service or external HTTP package is approved. |
| [`tests/orders.test.ts`](../tests/orders.test.ts) and [`tests/setup.ts`](../tests/setup.ts) | Vitest/Supertest tests exercise API routes and order behavior. | Add integration coverage for transaction rollback/commit with outbox rows, filtered event capture, webhook contracts/auth, delivery retries, and replay. No production source changes are part of this document task. |

## 12. Dependencies and Compatibility

### Internal Dependencies

- Existing `OrderService` transaction and order status enum.
- Existing Prisma client, MySQL database, `DATABASE_URL`, and schema migration workflow.
- Existing Express router/controller/service/repository patterns, Zod middleware, authentication, `requireRole`, shared errors, pagination helper, and Pino logger.
- Existing Node.js `>=20` runtime. Use built-in `node:crypto` for random secret generation, AES-256-GCM, and HMAC-SHA256. Use a built-in HTTPS client with a validated/pinned DNS lookup to enforce the destination policy.

### Deployment and Configuration Impact

- Apply the new Prisma migration before starting code that reads the webhook models.
- Configure the same `WEBHOOK_ENCRYPTION_KEY` (Base64 for 32 bytes) for API and worker processes before enabling webhook creation/delivery.
- Deploy/start the API and worker as separate processes. The worker requires database connectivity and the same encryption-key configuration.
- Add a worker build/start script without replacing the existing API `start` script.
- No Redis, queue service, mail service, dashboard, new HTTP library, or metrics/tracing backend is approved by the source documents or this FDD.

### Compatibility

- Existing order API request/response shapes and order status behavior remain unchanged except that eligible order transactions also write outbox rows.
- Existing authenticated role semantics remain: any authenticated role may manage configuration/history for any customer ID; only replay requires `ADMIN`.
- Existing error response format remains the `AppError` middleware envelope. Shared codes such as `VALIDATION_ERROR`, `UNAUTHORIZED`, and `FORBIDDEN` are not renamed.
- Existing customer order polling remains available; this feature does not remove or change `GET /orders`.

## 13. Technical Acceptance Criteria

### Event Capture and Transactions

- Creating an order with an active endpoint subscribed to `PENDING` commits one `order.created` outbox record per eligible endpoint with `from_status: null`.
- Creating an order with no matching active endpoint creates no webhook outbox row.
- Changing status writes one `order.status_changed` snapshot only for active endpoints subscribed to the new status.
- Inactive, deleted, or unsubscribed endpoints receive no new outbox row.
- An outbox insert failure rolls back the order, status history, and inventory changes in the same transaction.
- A source transaction rollback leaves no outbox row.
- Payload values are snapshots and do not change if the order later changes; items are absent.

### API and Security

- API contracts exist for create, list, patch, delete, secret rotation, history, and admin replay at the paths and methods in Section 6.
- URL validation rejects non-HTTPS and resolved prohibited destinations; redirects are not followed.
- Creation returns a generated secret once; list/update/history never return it.
- Secret rotation returns a different secret, signs immediately with it, and exposes the 24-hour customer overlap deadline.
- HMAC is over the exact transmitted UTF-8 request bytes and matches `sha256=<lowercase hex>`.
- The request includes all required headers and a stable event ID across retries and replay.
- The worker/API reject missing or malformed `WEBHOOK_ENCRYPTION_KEY` during bootstrap.
- Configuration/history routes require authentication; replay rejects non-admin users and records the acting admin.

### Worker, Retry, and DLQ

- Worker is a separate process with its own Prisma client, polls every two seconds, claims at most 10 rows per batch, and orders them oldest-first.
- Processing leases expire after 30 seconds; expired work becomes eligible for reprocessing.
- Requests have a 10-second timeout. 2xx responses become `DELIVERED`; timeout, connection failure, and every non-2xx response are recorded as failures.
- Retry delays after each failed attempt are exactly 1m, 5m, 30m, 2h, and 12h, yielding one initial send plus five retries.
- Failure of the sixth send creates a separate DLQ record; an event larger than 65,536 UTF-8 bytes is sent directly to DLQ without a request.
- Every HTTP attempt has a history record, including status/outcome, duration, captured response body, truncation flag, and failure code.
- A worker/database failure cannot produce a success-shaped state or silently discard a record.
- Replay is one-time per DLQ record, requires `ADMIN`, retains event ID/payload, resets the retry count for a new cycle, and records `replayedByUserId`/`replayedAt`.

### History, Errors, and Observability

- Delivery history returns at most the latest 100 HTTP attempts in descending attempt time and does not expose secrets or signatures.
- Feature-specific API/worker codes use `WEBHOOK_`; shared authentication, validation, and internal-server codes remain compatible with current middleware.
- Structured logs include useful IDs and outcomes without secrets or full payloads.
- Instrumentation can measure the PRD's first-success latency target, retry outcomes, pending backlog, and DLQ count. Metrics/tracing backend selection remains open.
- Tests use the existing Vitest/Supertest patterns and verify database commit/rollback behavior, not only mocked service return values.

## 14. Risks and Mitigations

| Risk | Evidence/impact | Mitigation or status |
|---|---|---|
| Customer outage or latency delays notifications and can exhaust retries. | The meeting explicitly considered unavailable/slow endpoints; PRD success target is under 10 seconds for 95% first successes. | 10-second timeout, five scheduled retries, attempt history, and DLQ replay. Delivery after exhaustion is not guaranteed. |
| At-least-once behavior can produce duplicate requests. | A remote endpoint may accept a request while the OMS loses the response or crashes before persisting success. | Stable `X-Event-Id` across retry/replay; customer deduplication is required. |
| One worker and ordered batch processing can create backlog behind slow endpoints. | Initial deployment deliberately uses one worker; multi-worker scaling is deferred. | Monitor pending age/worker lag; observe before deciding on concurrency/rate limiting or scaling. No global ordering guarantee. |
| Any authenticated operator can manage any customer's webhook. | This is explicitly the approved F01 behavior, but current JWT roles do not encode customer ownership. | Document as residual risk; a future per-customer authorization decision is required to provide tenant isolation. |
| Outbound webhook URL could target internal services. | Endpoint URLs are supplied through the API and are fetched by an OMS process. | HTTPS only; resolve and reject prohibited IPv4/IPv6 ranges for each connection, pin the checked address, and do not follow redirects. |
| Endpoint secret or encryption key exposure. | Secrets are used to sign order payloads; the current logger redacts common auth/password/token fields but has no webhook-specific fields. | Per-endpoint 32-byte secrets, AES-256-GCM at rest, external master key, one-time secret response, no secret/payload logging, security review before deployment. Master-key rotation remains out of scope. |
| Database growth from outbox, attempt, and DLQ evidence. | Archival/retention was explicitly deferred; per-attempt history increases row volume. | Track row counts and oldest-record age; retention/archival is an unresolved operational policy. |
| Payload above 64 KiB cannot be delivered by this contract. | Payload ceiling and reject-not-truncate behavior were agreed; current disposition is direct DLQ. | Record `WEBHOOK_PAYLOAD_TOO_LARGE`; investigate the snapshot schema. Replay does not solve an unchanged oversized payload. |
| The 95%/10-second target may be missed during backlog or slow endpoint behavior. | A poll interval and HTTP timeout alone do not ensure end-to-end latency. | Measure first-success latency and worker lag; treat the metric as an objective, not an unconditional guarantee. |

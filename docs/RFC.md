# RFC: Order Status Webhook Notifications

## Metadata

| Field | Value |
|---|---|
| Author | TBD |
| Status | Proposed — Submitted for Review |
| Date | 2026-11-05 |
| Reviewers | Larissa (Tech Lead), Marcos (Product Manager), Bruno (Orders Engineer), Diego (Platform Engineer), Sofia (Security Engineer) |

## Executive Summary (TL;DR)

Three B2B customers currently poll the OMS to detect order changes and have requested notifications with a delay below 10 seconds. This RFC proposes customer-configured outbound webhooks, with eligible events recorded transactionally in the existing MySQL database and delivered asynchronously by a separate polling worker.

The approach keeps customer endpoint latency outside order transactions, avoids introducing a queueing product, and provides signed requests with bounded retry and an operator replay path. It follows the architecture discussed by the team; the items in **Open Questions** must be clarified before implementation where the meeting notes and PRD differ or leave details undecided.

## Context and Problem

Atlas Comercial, MaxDistribuição, and Nova Cargo requested real-time order status notifications because repeated `GET /orders` polling makes their integrations slower and more expensive. They consider a delay under 10 seconds acceptable; Atlas also raised possible migration if the capability is not delivered by quarter end ([09:00]–[09:02] Marcos).

The current order service performs order creation and status changes in Prisma transactions. In particular, status changes update the order, write status history, and adjust stock within the transaction ([`src/modules/orders/order.service.ts`](../src/modules/orders/order.service.ts)). Calling customer endpoints synchronously from that path could block order processing or fail independently of the database transaction ([09:03]–[09:06] Bruno, Larissa, Diego).

The existing application uses domain modules and shared Express middleware; examples include [`src/modules/orders/order.routes.ts`](../src/modules/orders/order.routes.ts), [`src/middlewares/error.middleware.ts`](../src/middlewares/error.middleware.ts), and [`src/shared/logger/index.ts`](../src/shared/logger/index.ts). The [README](../README.md) describes the current application as having no external notifications, events, queues, or webhooks.

## Technical Proposal

1. **Configure endpoints through the OMS API.** Store each webhook endpoint with its customer association, destination, selected order statuses, and endpoint-specific secret. Require HTTPS and authenticate management requests using existing OMS authentication. The meeting agreed that ordinary authenticated users can manage configuration, while DLQ replay is restricted to `ADMIN` ([09:31]–[09:37] Marcos, Bruno, Sofia, Larissa).
2. **Capture eligible events atomically.** For each order event confirmed to be in scope, write its notification record to a MySQL outbox in the same transaction. Filter by the endpoint's selected statuses before enqueueing. This preserves the relationship between a committed order change and its notification record ([09:33]–[09:34], [09:40]–[09:41] Bruno, Diego).
3. **Deliver outside the API process.** A separate worker polls the outbox every two seconds, sending requests to customer HTTPS endpoints. Sign the request body with HMAC-SHA256 and the endpoint secret. Use the agreed timeout and retry policy; move exhausted deliveries to a separate DLQ and allow an administrator to replay them ([09:09]–[09:11], [09:17]–[09:23], [09:42] Diego, Sofia, Larissa).
4. **Make duplicate delivery explicit.** Delivery is at-least-once. A stable `X-Event-Id` lets customer systems identify and deduplicate repeats, including retries ([09:24]–[09:26] Diego, Sofia, Marcos, Larissa).

This design reuses MySQL and the project's existing application conventions rather than adding a queueing product. The detailed request contracts, persistence schema, retry processing, and integration steps belong in the FDD, not this RFC.

## Alternatives Considered

| Alternative discussed | Why it was rejected | Evidence |
|---|---|---|
| Make the outbound HTTP request synchronously during the order transaction. | A slow or unavailable customer endpoint could block order changes; an external failure cannot safely determine whether the database transaction should commit. | [09:03]–[09:06] Bruno, Larissa, Diego |
| Use Redis Streams or another queueing product. | This adds infrastructure for a small team; the existing MySQL database can support the outbox approach. | [09:07] Diego |
| Use database triggers to notify the worker. | MySQL triggers do not notify an external process; an improvised signal was considered undesirable, and two-second polling meets the stated latency target. | [09:09] Diego |

## Open Questions

1. **Retry-count interpretation:** The meeting specifies five scheduled delays, but its summary calls this “five attempts” ([09:15]–[09:17], [09:48] Larissa). The PRD specifies five retries *after* the initial send. Confirm whether the initial send is included in the five attempts or precedes five retries; this RFC does not resolve the wording difference.
2. **Order creation events:** The PRD includes a distinct `order.created` notification for endpoints subscribed to `PENDING`. The meeting discusses status filters and status-change notifications ([09:31]–[09:34], [09:43]) but does not clearly settle a separate creation event. Confirm whether this event is in scope.
3. **Worker batch size:** The meeting calls for a small batch but does not set its size ([09:08] Diego). Select a value during design without changing the agreed polling interval.
4. **Deferred operational policies:** The team deferred outbound rate limiting until traffic is observed ([09:38]–[09:39] Diego, Larissa) and left record archival/retention for later, mentioning approximately 30 days only as an example ([09:08] Diego). Decide these policies based on operational evidence; no specific limit or retention period is agreed here.

## Impact and Risks

| Impact or risk | Expected effect | Mitigation or unresolved point |
|---|---|---|
| External endpoint latency is isolated from order processing. | Order operations do not wait for customer HTTP responses. | Record eligible events in the same database transaction; deliver asynchronously. |
| Customer outages can delay or prevent delivery. | Events may exhaust retries and reach the DLQ. | Bounded retries and administrator replay provide recovery; delivery after exhaustion is not guaranteed. |
| At-least-once delivery can produce duplicates. | Customers may process the same event more than once. | Preserve the event ID across retries and replay; customers must deduplicate with `X-Event-Id`. |
| A single worker limits parallel processing. | Per-order ordering is supported only while one worker runs; no global ordering is promised. | Keep one worker for the initial design. A multi-worker ordering strategy remains deferred ([09:12]–[09:13] Diego, Larissa). |
| Outbox and delivery records can accumulate. | Database storage and polling costs may grow. | Archival was explicitly left outside this feature; its retention policy remains open ([09:08] Diego). |
| Secret rotation behavior is not fully specified. | The sender's signing-secret selection during the 24-hour overlap could be implemented inconsistently. | Resolve the signing-key selection before implementation; endpoint-specific secrets and the overlap itself were agreed ([09:21]–[09:22] Sofia). |

The proposed benefit is reduced polling and prompt notification without coupling order operations to customer availability. The meeting's three-sprint estimate included security review, with at least two business days reserved for that review ([09:45]–[09:47] Marcos, Larissa, Sofia); it is an estimate, not a delivery commitment in this RFC.

## Related Decisions

- [ADR-001: Transactional Outbox on MySQL](adrs/ADR-001-transactional-outbox-on-mysql.md) records the atomic event-capture approach that anchors this proposal.
- [ADR-002: Separate Worker with Database Polling](adrs/ADR-002-separate-polling-worker.md) records the worker boundary, polling interval, and ordering limitation.
- [ADR-003: Bounded Retries and a Separate Dead-Letter Queue](adrs/ADR-003-bounded-retries-and-separate-dlq.md) records the agreed retry schedule and DLQ approach; attempt-count interpretation remains to be reconciled.
- [ADR-004: Endpoint-Scoped HMAC Authentication](adrs/ADR-004-endpoint-scoped-hmac-authentication.md) records the request-signing and secret-isolation decision.
- [ADR-005: At-Least-Once Delivery with a Stable Event ID](adrs/ADR-005-at-least-once-with-stable-event-id.md) records duplicate-delivery semantics and customer deduplication.
- [ADR-006: Reuse Existing Project Patterns](adrs/ADR-006-reuse-existing-project-patterns.md) records how the feature fits existing OMS conventions.

Coverage is tracked once per distinct requirement, decision, constraint, trade-off, or open question. Repeated explanatory prose and illustrative examples are not counted as separate items when they express the same tracked item. Proposed implementation details that are not approved are labeled as proposals in the FDD and are not represented as meeting decisions.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-PROBLEM-01 | docs/PRD.md | Problem | Three B2B customers poll `GET /orders`, making their integrations slower and more expensive. | TRANSCRICAO | [09:00] Marcos |
| PRD-PROBLEM-02 | docs/PRD.md | Acceptance Criterion | Customers consider delivery under 10 seconds to be real time. | TRANSCRICAO | [09:02] Marcos |
| PRD-PROBLEM-03 | docs/PRD.md | Scope Constraint | Webhooks are outbound only; customers receive notifications and do not send webhook events to the OMS. | TRANSCRICAO | [09:02] Marcos |
| PRD-FR-01 | docs/PRD.md | Functional Requirement | Authenticated users can create, list, update, and delete customer-scoped webhook configurations. | TRANSCRICAO | [09:31]–[09:33] Marcos, Bruno |
| PRD-FR-02 | docs/PRD.md | Functional Requirement | Each endpoint stores a list of order statuses, with filtering before outbox insertion. | TRANSCRICAO | [09:33]–[09:34] Marcos, Bruno |
| PRD-FR-03 | docs/PRD.md | Functional Requirement | Eligible status-change events are written to the MySQL outbox in the same transaction as the order change. | TRANSCRICAO | [09:06] Diego; [09:40]–[09:41] Bruno, Diego |
| PRD-FR-04 | docs/PRD.md | Functional Requirement | A separate worker polls the outbox every two seconds and processes older pending events first. | TRANSCRICAO | [09:09]–[09:12] Diego, Larissa |
| PRD-FR-05 | docs/PRD.md | Functional Requirement | Outbound requests use HMAC-SHA256 with a unique secret per endpoint, a 24-hour rotation overlap, and HTTPS destinations. | TRANSCRICAO | [09:20]–[09:23] Sofia |
| PRD-FR-06 | docs/PRD.md | Functional Requirement | The JSON payload contains the agreed order/event fields, excludes order items, and is rejected rather than truncated above 64 KB. | TRANSCRICAO | [09:23]–[09:24], [09:43] Diego, Larissa |
| PRD-FR-07 | docs/PRD.md | Functional Requirement | Failed deliveries use the agreed delay sequence and exhausted deliveries are persisted in a separate DLQ; attempt-count interpretation remains open. | TRANSCRICAO | [09:15]–[09:19] Diego, Bruno, Larissa |
| PRD-FR-08 | docs/PRD.md | Functional Requirement | Authenticated users can inspect the latest 100 delivery records, including outcome, payload, response, and response time. | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-09 | docs/PRD.md | Functional Requirement | An administrator can replay a DLQ record through an admin endpoint and the replay action records the actor. | TRANSCRICAO | [09:18]–[09:19] Diego, Larissa; [09:36] Sofia |
| PRD-FR-10 | docs/PRD.md | Functional Requirement | Delivery is at-least-once, and customers deduplicate repeats using the stable `X-Event-Id`. | TRANSCRICAO | [09:24]–[09:26] Diego, Bruno, Larissa |
| PRD-SEC-01 | docs/PRD.md | Security Requirement | Sign request bodies with HMAC-SHA256 using a unique secret per endpoint, with a 24-hour rotation overlap. | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| PRD-SEC-02 | docs/PRD.md | Security Requirement | Webhook destination URLs must use HTTPS; HTTP destinations are rejected. | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-01 | docs/PRD.md | Non-Functional Requirement | Reject payloads above the agreed 64 KB limit rather than truncate them. | TRANSCRICAO | [09:23] Sofia; [09:24] Diego |
| PRD-DEC-01 | docs/PRD.md | Delivery Guarantee | Delivery is at-least-once; customers deduplicate retries with the stable `X-Event-Id`. | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| PRD-DEC-02 | docs/PRD.md | Architecture Decision | Store eligible events in a MySQL outbox in the same transaction as the order change. | TRANSCRICAO | [09:06] Diego; [09:08] Larissa; [09:40] Bruno; [09:41] Diego |
| PRD-DEC-03 | docs/PRD.md | Architecture Decision | Run one worker as a separate process, polling the outbox every two seconds. | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:11] Diego |
| PRD-CONSTRAINT-01 | docs/PRD.md | Constraint | Ordering is limited to per-order processing while a single worker runs; no global ordering is promised. | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| PRD-DEC-04 | docs/PRD.md | Retry Policy | Use five scheduled retry delays: 1 minute, 5 minutes, 30 minutes, 2 hours, and 12 hours. | TRANSCRICAO | [09:17] Diego |
| PRD-DEC-05 | docs/PRD.md | Decision | Move exhausted deliveries to a separate dead-letter table and retain failure evidence for investigation and replay. | TRANSCRICAO | [09:17] Diego; [09:18] Diego |
| PRD-SEC-03 | docs/PRD.md | Authorization Requirement | Require the `ADMIN` role for DLQ replay and log the administrator who performed it. | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| PRD-OUT-01 | docs/PRD.md | Out of Scope | Email alerts after repeated delivery failures are deferred. | TRANSCRICAO | [09:37] Larissa |
| PRD-OUT-02 | docs/PRD.md | Out of Scope | Outbound rate limiting is deferred pending observation of delivery traffic. | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| PRD-OUT-03 | docs/PRD.md | Out of Scope | A visual customer dashboard is excluded; this phase exposes APIs only. | TRANSCRICAO | [09:39] Marcos; [09:40] Larissa |
| PRD-OUT-04 | docs/PRD.md | Out of Scope | Archival of delivered outbox rows is outside this feature; 30 days was only an example. | TRANSCRICAO | [09:08] Diego |
| PRD-PLAN-01 | docs/PRD.md | Estimate | The meeting estimated three sprints, including security review, with at least two business days reserved for that review. | TRANSCRICAO | [09:45] Larissa; [09:46] Sofia; [09:47] Larissa |
| RFC-DEC-01 | docs/RFC.md | Architecture Decision | Prefer the MySQL transactional outbox over synchronous external HTTP calls to isolate order processing from customer availability. | TRANSCRICAO | [09:04] Bruno; [09:06] Diego; [09:08] Larissa |
| RFC-ALT-01 | docs/RFC.md | Trade-off | Synchronous HTTP delivery was rejected because slow or unavailable customer endpoints could block order changes. | TRANSCRICAO | [09:03] Larissa; [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Trade-off | Redis Streams was rejected because it would add infrastructure for a small team. | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Trade-off | Database triggers were rejected because MySQL triggers do not notify an external worker; polling meets the latency need. | TRANSCRICAO | [09:09] Diego |
| RFC-DEC-02 | docs/RFC.md | Architecture Decision | Deliver outbox records asynchronously through a separate worker rather than in the order request. | TRANSCRICAO | [09:06] Diego; [09:11] Diego |
| RFC-DEC-03 | docs/RFC.md | Architecture Decision | Use HMAC-SHA256 signatures and endpoint-specific secrets for outbound requests. | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| RFC-DEC-04 | docs/RFC.md | Delivery Guarantee | Use at-least-once delivery with a stable event ID so consumers can deduplicate duplicates. | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| RFC-OPEN-01 | docs/RFC.md | Open Question | The RFC records a discrepancy between the scheduled retry delays and the meeting summary's “five attempts” wording. | TRANSCRICAO | [09:17] Diego; [09:48] Larissa |
| RFC-OPEN-02 | docs/RFC.md | Open Question | The RFC asks whether order creation should emit a distinct event; the meeting does not settle this behavior. | TRANSCRICAO | [09:31] Marcos; [09:33] Marcos |
| RFC-OPEN-03 | docs/RFC.md | Open Question | The worker batch size was left unspecified; only a small batch was discussed. | TRANSCRICAO | [09:08] Diego |
| RFC-OPEN-04 | docs/RFC.md | Open Question | Outbound rate limiting was deferred until delivery traffic can be observed. | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| RFC-OPEN-05 | docs/RFC.md | Open Question | Outbox archival/retention was left outside scope; approximately 30 days was not a decided policy. | TRANSCRICAO | [09:08] Diego |
| FDD-OBJ-01 | docs/FDD.md | Technical Objective | Capture eligible events atomically with source order operations so a rollback leaves no outbox event. | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| FDD-OBJ-02 | docs/FDD.md | Technical Objective | Keep customer HTTP calls outside the order transaction and process them asynchronously. | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| FDD-OBJ-03 | docs/FDD.md | Technical Objective | Use a separate worker with a two-second polling interval. | TRANSCRICAO | [09:09] Diego; [09:11] Diego |
| FDD-OBJ-04 | docs/FDD.md | Technical Objective | Make duplicate delivery explicit and retain a stable event UUID for customer deduplication. | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| FDD-SCOPE-01 | docs/FDD.md | Scope Constraint | The webhook flow is outbound only. | TRANSCRICAO | [09:02] Marcos |
| FDD-SCOPE-02 | docs/FDD.md | Out of Scope | Email fallback is deferred until delivery impact can be measured. | TRANSCRICAO | [09:37] Larissa; [09:38] Marcos |
| FDD-SCOPE-03 | docs/FDD.md | Out of Scope | A customer dashboard is excluded from this API-only phase. | TRANSCRICAO | [09:39] Marcos; [09:40] Larissa |
| FDD-SCOPE-04 | docs/FDD.md | Out of Scope | Outbound rate limiting is deferred for later evaluation. | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| FDD-SCOPE-05 | docs/FDD.md | Out of Scope | Multiple-worker scaling and its ordering strategy are deferred. | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| FDD-SCOPE-06 | docs/FDD.md | Out of Scope | Exactly-once delivery is not provided; consumers deduplicate with `X-Event-Id`. | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| FDD-FLOW-01 | docs/FDD.md | Data Consistency | The outbox insert must share the order status transaction; failure to insert aborts that transaction. | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| FDD-FLOW-02 | docs/FDD.md | Event Snapshot | Persist the rendered payload at enqueue time so later order changes do not alter the event. | TRANSCRICAO | [09:51] Larissa; [09:52] Diego |
| FDD-FLOW-03 | docs/FDD.md | Event Filter | Insert an event only when an endpoint subscribes to the status being delivered. | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno |
| FDD-FLOW-04 | docs/FDD.md | Payload Constraint | Include the agreed basic order fields but omit order items to keep the payload small. | TRANSCRICAO | [09:43] Diego |
| FDD-FLOW-05 | docs/FDD.md | Delivery Constraint | Use a 10-second HTTP timeout; a timeout is treated as a failed attempt for retry. | TRANSCRICAO | [09:42] Diego |
| FDD-FLOW-06 | docs/FDD.md | Delivery Contract | Send `X-Event-Id`, `X-Signature`, `X-Timestamp`, and `X-Webhook-Id` headers with JSON content. | TRANSCRICAO | [09:44] Diego; [09:44] Sofia |
| FDD-FLOW-07 | docs/FDD.md | Retry Policy | Schedule retries at 1 minute, 5 minutes, 30 minutes, 2 hours, and 12 hours after failures. | TRANSCRICAO | [09:17] Diego |
| FDD-FLOW-08 | docs/FDD.md | DLQ | Persist exhausted deliveries in a separate DLQ table with payload, failure reason, and timestamp. | TRANSCRICAO | [09:17] Diego; [09:18] Diego |
| FDD-FLOW-09 | docs/FDD.md | Replay | Provide a manual admin replay endpoint that requeues a DLQ event. | TRANSCRICAO | [09:18] Diego; [09:19] Larissa |
| FDD-FLOW-10 | docs/FDD.md | Authorization Requirement | Require `ADMIN` role for replay and record the actor for audit. | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| FDD-CONTRACT-01 | docs/FDD.md | API Contract | Provide create, update, delete, and customer-scoped list operations for webhook configuration. | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno |
| FDD-CONTRACT-02 | docs/FDD.md | API Contract | Supply the customer ID through the request route or body rather than deriving it from the JWT. | TRANSCRICAO | [09:32] Larissa |
| FDD-CONTRACT-03 | docs/FDD.md | API Contract | Generate an endpoint secret and return it at creation; support API-requested secret rotation. | TRANSCRICAO | [09:21] Sofia; [09:31] Marcos |
| FDD-CONTRACT-04 | docs/FDD.md | API Contract | Expose a delivery-history endpoint with up to the latest 100 delivery records. | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRACT-05 | docs/FDD.md | API Contract | Expose `POST /admin/webhooks/dead-letter/:id/replay` for manual DLQ replay. | TRANSCRICAO | [09:18] Diego |
| FDD-CONTRACT-06 | docs/FDD.md | API Contract | Restrict replay to `ADMIN`; other authenticated roles cannot perform it. | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| FDD-SEC-01 | docs/FDD.md | Security Requirement | Use HMAC-SHA256 over the request body and a unique secret for each endpoint. | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| FDD-SEC-02 | docs/FDD.md | Security Requirement | Keep the prior endpoint secret valid for a 24-hour rotation grace period. | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| FDD-SEC-03 | docs/FDD.md | Security Requirement | Require HTTPS destinations; reject HTTP URLs. | TRANSCRICAO | [09:23] Sofia |
| FDD-SEC-04 | docs/FDD.md | Non-Functional Requirement | Reject payloads exceeding 64 KB instead of truncating them. | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |
| FDD-DELIVERY-01 | docs/FDD.md | Delivery Guarantee | At-least-once delivery permits duplicates; the customer deduplicates with the stable `X-Event-Id`. | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| FDD-DELIVERY-02 | docs/FDD.md | Ordering Constraint | A single worker processes events by creation order; no global ordering or multi-worker guarantee is made. | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| FDD-OPEN-01 | docs/FDD.md | Open Question | The meeting does not resolve whether a distinct `order.created` notification is in scope. | TRANSCRICAO | [09:31]–[09:34] Marcos, Bruno, Diego |
| FDD-OPEN-02 | docs/FDD.md | Open Question | The meeting's five-attempt summary and five delay values do not settle whether the initial send is included. | TRANSCRICAO | [09:17] Diego; [09:48] Larissa |
| FDD-OPEN-03 | docs/FDD.md | Open Question | Worker batch size was described as small but was not specified. | TRANSCRICAO | [09:08] Diego |
| FDD-OPEN-04 | docs/FDD.md | Open Question | The sender's signing-secret selection during the 24-hour rotation overlap was not specified. | TRANSCRICAO | [09:21]–[09:22] Sofia, Diego |
| FDD-OPEN-05 | docs/FDD.md | Open Question | The meeting requires rejecting oversized payloads but does not settle their persistence or retry disposition. | TRANSCRICAO | [09:23]–[09:24] Sofia, Diego, Larissa |
| FDD-OPEN-06 | docs/FDD.md | Open Question | Outbox archival/retention was left outside scope; approximately 30 days was only an example. | TRANSCRICAO | [09:08] Diego |
| FDD-OPEN-07 | docs/FDD.md | Open Question | Outbound rate limiting was deferred pending observation of traffic. | TRANSCRICAO | [09:38]–[09:39] Diego, Larissa |
| FDD-CODE-01 | docs/FDD.md | Code Integration | Order creation and `changeStatus` already use Prisma transactions, and status changes update order history and stock. | CODIGO | src/modules/orders/order.service.ts |
| FDD-CODE-02 | docs/FDD.md | Code Integration | Existing order status transitions and stock debit/replenishment rules are defined in the order status module. | CODIGO | src/modules/orders/order.status.ts |
| FDD-CODE-03 | docs/FDD.md | Code Convention | Prisma models use UUID identifiers and define the existing order status and user-role enums. | CODIGO | prisma/schema.prisma |
| FDD-CODE-04 | docs/FDD.md | Authorization Convention | Existing authentication middleware exposes `authenticate` and `requireRole` for `ADMIN`/`OPERATOR` roles. | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-CODE-05 | docs/FDD.md | Error Handling Convention | `AppError` carries an HTTP status and error code for centralized API error handling. | CODIGO | src/shared/errors/app-error.ts |
| FDD-CODE-06 | docs/FDD.md | Logging Convention | The shared logger is built with Pino and configures redaction paths. | CODIGO | src/shared/logger/index.ts |
| FDD-CODE-07 | docs/FDD.md | Route Convention | Existing order routes apply authentication and request validation middleware. | CODIGO | src/modules/orders/order.routes.ts |
| ADR-001 | docs/adrs/ADR-001-transactional-outbox-on-mysql.md | Decision | Persist eligible webhook events in a MySQL outbox within the source order transaction. | TRANSCRICAO | [09:06] Diego; [09:08] Larissa; [09:40] Bruno |
| ADR-001-ALT-01 | docs/adrs/ADR-001-transactional-outbox-on-mysql.md | Trade-off | Synchronous HTTP delivery was rejected because customer latency and downtime could block order operations. | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| ADR-001-ALT-02 | docs/adrs/ADR-001-transactional-outbox-on-mysql.md | Trade-off | Redis Streams was rejected to avoid adding infrastructure for this feature. | TRANSCRICAO | [09:07] Diego |
| ADR-002 | docs/adrs/ADR-002-separate-polling-worker.md | Decision | Run a separate worker that polls the MySQL outbox every two seconds. | TRANSCRICAO | [09:09] Diego; [09:11] Diego |
| ADR-002-ALT-01 | docs/adrs/ADR-002-separate-polling-worker.md | Trade-off | Running the worker inside the API process was rejected because an API restart would also stop it. | TRANSCRICAO | [09:11] Diego |
| ADR-002-ALT-02 | docs/adrs/ADR-002-separate-polling-worker.md | Trade-off | Database triggers were rejected because MySQL triggers do not notify external processes. | TRANSCRICAO | [09:09] Diego |
| ADR-002-CONSTRAINT-01 | docs/adrs/ADR-002-separate-polling-worker.md | Ordering Constraint | Per-order ordering depends on using one worker; global ordering is not guaranteed. | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| ADR-003 | docs/adrs/ADR-003-bounded-retries-and-separate-dlq.md | Decision | Use the five-delay retry schedule and move exhausted events into a separate DLQ for manual replay. | TRANSCRICAO | [09:17] Diego; [09:18] Diego |
| ADR-003-ALT-01 | docs/adrs/ADR-003-bounded-retries-and-separate-dlq.md | Trade-off | Indefinite retries were rejected because an event could remain pending forever if a customer disappeared. | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | docs/adrs/ADR-003-bounded-retries-and-separate-dlq.md | Trade-off | Three retries were rejected as insufficient for a customer maintenance outage lasting around two hours. | TRANSCRICAO | [09:16] Bruno; [09:16] Diego |
| ADR-003-ALT-03 | docs/adrs/ADR-003-bounded-retries-and-separate-dlq.md | Trade-off | Keeping exhausted events only as failed outbox rows was rejected in favor of a separate table for investigation and replay. | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| ADR-004 | docs/adrs/ADR-004-endpoint-scoped-hmac-authentication.md | Decision | Sign each request with HMAC-SHA256 and a unique secret per endpoint; support 24-hour rotation overlap and HTTPS. | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:23] Sofia |
| ADR-004-ALT-01 | docs/adrs/ADR-004-endpoint-scoped-hmac-authentication.md | Trade-off | A global secret was rejected because compromise would affect every endpoint. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-02 | docs/adrs/ADR-004-endpoint-scoped-hmac-authentication.md | Security Constraint | Non-HTTPS webhook destinations are rejected. | TRANSCRICAO | [09:23] Sofia |
| ADR-005 | docs/adrs/ADR-005-at-least-once-with-stable-event-id.md | Decision | Provide at-least-once delivery with a stable UUID in `X-Event-Id`; customers deduplicate repeats. | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| ADR-005-ALT-01 | docs/adrs/ADR-005-at-least-once-with-stable-event-id.md | Trade-off | Exactly-once delivery was rejected because it would require complex coordination across both systems. | TRANSCRICAO | [09:25] Diego |
| ADR-006 | docs/adrs/ADR-006-reuse-existing-project-patterns.md | Decision | Reuse the existing module, error, logging, validation, authentication, and role-check patterns for webhooks. | TRANSCRICAO | [09:27] Bruno; [09:28] Bruno; [09:29] Bruno; [09:30] Larissa |
| ADR-006-CODE-01 | docs/adrs/ADR-006-reuse-existing-project-patterns.md | Code Convention | Existing order and product modules provide examples of the project's domain-router structure. | CODIGO | src/modules/orders/order.routes.ts |
| ADR-006-CODE-02 | docs/adrs/ADR-006-reuse-existing-project-patterns.md | Code Convention | The repository defines a shared `AppError` type carrying an HTTP status and error code. | CODIGO | src/shared/errors/app-error.ts |
| ADR-006-CODE-03 | docs/adrs/ADR-006-reuse-existing-project-patterns.md | Code Convention | The repository provides a shared Pino logger. | CODIGO | src/shared/logger/index.ts |

# ADR-001: Transactional Outbox on MySQL

## Status

Accepted

## Context

Order status changes update the order, status history, and inventory in a database transaction. A synchronous HTTP call to a customer endpoint could block that transaction or fail independently after the order has changed. The notification must be recorded if the order transaction commits and must not survive if it rolls back.

The OMS already uses MySQL, and the team wants to avoid introducing infrastructure for this feature.

## Decision

Persist each eligible webhook event in a MySQL outbox within the same transaction as its source order operation. Deliver the event asynchronously after the transaction commits.

## Alternatives Considered

- **Synchronous HTTP delivery from the order service (discussed and rejected):** a slow or unavailable customer endpoint would block order processing and cannot be made atomic with the database transaction. [09:03]–[09:06] Bruno, Larissa, Diego.
- **Redis Streams or another external queue (discussed and rejected):** it would require additional infrastructure; the team considered that disproportionate for the current team and feature. [09:07] Diego.

## Consequences

- The order change and event record share one database commit, preventing a committed order change from losing its notification record.
- Customer endpoint latency and downtime do not block the order transaction.
- The OMS avoids a new queueing dependency.
- The application must poll and manage outbox records, including their growth and eventual archival; archival policy remains outside this decision.

## References

- [09:04]–[09:08] Bruno, Larissa, Diego — synchronous delivery concerns and the MySQL outbox decision.
- [09:40]–[09:41] Bruno, Diego — enqueueing within the order transaction.
- `src/modules/orders/order.service.ts` — order creation and status changes use Prisma transactions.

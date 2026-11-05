# ADR-002: Separate Worker with Database Polling

## Status

Accepted

## Context

The outbox needs a process to discover pending deliveries and call customer endpoints. The agreed notification target is under 10 seconds, and the worker must not be coupled to the API process lifecycle. MySQL does not provide the external-process notification mechanism the team would need for a database-trigger approach.

## Decision

Run one worker as a process separate from the API. It polls the MySQL outbox every two seconds and processes pending events oldest first, using the same database configuration and application stack.

Per-order ordering is supported only while a single worker is running; global ordering is not guaranteed.

## Alternatives Considered

- **Run the worker inside the API process (discussed and rejected):** an API restart could stop the worker as well. [09:11] Diego.
- **Use database triggers to notify the worker (discussed and rejected):** MySQL triggers execute SQL but do not notify an external process; an improvised file or endpoint signal was considered undesirable. Two-second polling meets the latency need. [09:09] Diego, Marcos, Larissa.
- **Scale to multiple workers immediately (deferred):** parallel workers would lose the current ordering behavior; partitioning or locking was left for future consideration. [09:12]–[09:13] Diego, Larissa.

## Consequences

- The worker remains independent of API restarts while sharing the existing database.
- Two-second polling gives the agreed discovery cadence without adding a notification service.
- A single worker limits parallelism. Ordering is not promised globally and depends on continuing to run one worker.
- Batch size and a future multi-worker ordering strategy remain unspecified.

## References

- [09:09]–[09:13] Diego, Marcos, Larissa — polling cadence, separate process, and ordering limits.
- [09:29]–[09:30] Diego, Bruno, Larissa — separate Prisma client per process and shared database configuration.
- `src/modules/orders/order.service.ts` — order operations use the existing Prisma-backed service layer.

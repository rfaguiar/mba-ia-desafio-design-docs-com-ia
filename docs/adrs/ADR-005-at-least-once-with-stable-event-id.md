# ADR-005: At-Least-Once Delivery with a Stable Event ID

## Status

Accepted

## Context

Retries and manual replay can cause a customer to receive the same notification more than once. Guaranteeing exactly-once delivery across the OMS and an external customer system would require coordination between both sides.

## Decision

Provide at-least-once delivery and assign each event a stable UUID exposed in the `X-Event-Id` header. Customers are responsible for deduplicating repeated deliveries with that identifier. Retries and manual replay retain the original event ID.

## Alternatives Considered

- **Exactly-once delivery (discussed and rejected):** it would require coordination between the OMS and customer systems and was considered substantially more complex. [09:24]–[09:26] Diego, Sofia, Marcos, Larissa.

## Consequences

- The OMS can retry and replay without coordinating a distributed exactly-once transaction.
- A customer may receive duplicates and must implement deduplication using the stable identifier.
- The ID remains useful across retries and replay because it does not change.

## References

- [09:24]–[09:26] Diego, Sofia, Marcos, Larissa — delivery guarantee, duplicate handling, and stable event ID.
- [09:48] Larissa — summary of at-least-once delivery and `X-Event-Id`.

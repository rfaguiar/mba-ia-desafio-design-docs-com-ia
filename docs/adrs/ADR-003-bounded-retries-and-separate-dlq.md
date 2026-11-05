# ADR-003: Bounded Retries and a Separate Dead-Letter Queue

## Status

Accepted for the delay sequence and separate DLQ; attempt count pending clarification

## Context

Customer endpoints may be unavailable for hours. Retrying indefinitely could leave events pending forever, while too few retries could exhaust delivery during a planned customer outage. Failed deliveries must remain available for investigation and manual replay without cluttering the primary outbox.

## Decision

After a failed delivery, use the agreed retry-delay sequence of 1 minute, 5 minutes, 30 minutes, 2 hours, and 12 hours. Persist an exhausted event, its failure reason, and timestamp in a separate dead-letter table. Allow an administrator to request manual replay.

The meeting summary calls the policy "five attempts" ([09:17], [09:48]), while listing five delay values. Whether the initial send is included in the count remains unresolved; this ADR does not define the total number of sends or the DLQ threshold.

## Alternatives Considered

- **Retry indefinitely with backoff (discussed and rejected):** an event could remain pending forever if a customer endpoint disappears. [09:15] Diego.
- **Use only three retries (discussed and rejected):** the proposed window could be too short for a customer maintenance outage lasting around two hours. [09:16] Bruno, Diego.
- **Keep exhausted events only as failed rows in the outbox (discussed and rejected):** the separate table keeps the main outbox cleaner and retains evidence for debugging and replay. [09:17]–[09:18] Diego.

## Consequences

- Temporary outages receive a bounded retry window, and exhausted events remain available for operator investigation and replay.
- The system does not guarantee delivery if an endpoint remains unavailable after retries.
- The separate dead-letter table adds persistence and replay lifecycle responsibilities.
- Implementation of the retry counter and DLQ threshold is blocked until the initial-send/attempt-count interpretation is clarified.
- Other operational details beyond the retry schedule, dead-letter record, and manual replay remain unspecified.

## References

- [09:15]–[09:19] Diego, Bruno, Larissa — retry alternatives, schedule, separate dead-letter table, and replay.
- [09:48] Larissa — summary of the retry and DLQ decisions.

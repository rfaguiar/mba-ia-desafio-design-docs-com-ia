# ADR-004: Endpoint-Scoped HMAC Authentication

## Status

Accepted

## Context

Webhook requests carry order data to destinations outside the OMS. Customers need a way to verify request authenticity and detect payload tampering. A shared platform-wide secret would increase the impact of a single secret leak.

## Decision

Sign each request body with HMAC-SHA256 using a unique secret for each webhook endpoint. Support secret rotation with a 24-hour overlap. Require HTTPS destinations.

The meeting did not decide which secret signs outbound requests during the overlap; that detail remains open.

## Alternatives Considered

- **Use one global secret for all endpoints (discussed and rejected):** compromise of that secret would expose every endpoint. [09:20]–[09:21] Sofia, Bruno.
- **Accept non-HTTPS destinations (discussed and rejected):** the meeting made TLS mandatory and HTTP URLs invalid. [09:23] Sofia.

## Consequences

- Customers can authenticate requests and detect changes to the signed body.
- Compromise of one endpoint secret is isolated from other endpoints.
- Rotation provides a migration window but requires a final decision about which secret the sender uses during that overlap.
- HTTPS is a prerequisite for configured destinations.

## References

- [09:19]–[09:23] Sofia, Bruno, Diego — HMAC-SHA256, per-endpoint secrets, rotation, and HTTPS.
- [09:48] Larissa — summary of the security decision.

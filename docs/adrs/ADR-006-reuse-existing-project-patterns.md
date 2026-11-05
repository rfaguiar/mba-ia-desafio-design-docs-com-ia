# ADR-006: Reuse Existing Project Patterns

## Status

Accepted

## Context

The team agreed that the webhook feature should fit the existing OMS architecture rather than introduce a parallel module, logging, validation, or error-handling approach. Existing domain modules separate routes, controllers, services, repositories, and schemas. The API already has shared errors, centralized error middleware, role checks, validation middleware, and Pino logging.

## Decision

Implement webhook API functionality as a domain module following the existing module structure. Reuse the project's `AppError` and `WEBHOOK_` error-code convention, centralized error middleware, Pino logger, Zod validation, and existing authentication and role middleware. The separate worker remains a distinct process, not a second API module.

## Alternatives Considered

- **Introduce feature-specific module, error, and logging conventions (plausible alternative; not discussed in the meeting):** this could isolate the feature but would duplicate existing infrastructure and depart from the agreed reuse approach.

## Consequences

- The feature follows established API patterns and error responses.
- Existing middleware and logging behavior can be shared instead of reimplemented.
- The webhook module must remain compatible with shared error and route conventions.
- The worker's separate process boundary remains intact while its application logic follows the project's structure.

## References

- [09:27]–[09:30] Bruno, Diego, Larissa — module structure, error-code prefix, Pino, centralized errors, Zod, authentication, and reuse decision.
- `src/modules/orders/order.routes.ts` and `src/modules/products/product.routes.ts` — representative domain routers using authentication and validation middleware.
- `src/shared/errors/app-error.ts` and `src/middlewares/error.middleware.ts` — shared application errors and centralized handling.
- `src/shared/logger/index.ts` — shared Pino logger.
- `src/middlewares/auth.middleware.ts` — authentication and `requireRole` middleware.

---
name: backend-patterns
description: Rust backend architecture patterns for Tokio, Axum, SQLx, PostgreSQL, Kong, Keycloak, and Flipt.
---

# Backend Development Patterns

Use these patterns for WorldFlowAI backend services. Backend implementation uses Rust with Tokio, Axum, SQLx, and PostgreSQL. Do not introduce another application language or framework unless the user explicitly requests it.

## Clean Architecture

Dependencies point inward:

- `domain`: entities, value objects, domain services, events, and business invariants. No framework, SQL, network, or filesystem dependencies.
- `application`: use cases, commands/queries, result models, and ports required by use cases (repositories, clock, identity, external services, feature evaluation). Orchestrate domain behavior without owning transport or persistence details.
- `infrastructure`: Axum routes and WebSocket adapters, SQLx repositories and row mappings, Flipt/Keycloak/Kong integrations, configuration, and dependency wiring.

Keep Axum handlers thin: validate transport input, call one use case, and map its result to an HTTP response. Use cases must not return HTTP responses, SQL rows, or framework-specific errors. Wire concrete adapters at the composition root.

## API Design

- Model resources and operations around product capabilities; define request/response contracts independently of SQL row types.
- Validate input at the Axum boundary and enforce business invariants again in the domain where they must always hold.
- Use stable error responses, explicit pagination, idempotency for retryable writes, and request correlation IDs.
- Put Kong in front of services when centralized routing or gateway policies are required. Keep domain authorization in the backend; the gateway does not replace service-level checks.

## PostgreSQL and SQLx

- PostgreSQL is the persistent database. Access it from infrastructure adapters through SQLx; never expose SQLx rows or pools to domain objects.
- Prefer denormalized, query-oriented models and selective duplication. Avoid normalization by default; maintain duplicated values through transactions, constraints, idempotent writes, and reconciliation.
- Bind user-controlled values. Allow-list dynamic identifiers, since SQL parameters cannot represent table or column names.
- Select only required columns, use indexes that match observed access patterns, and avoid N+1 queries with joins or batched reads.
- Keep transactions explicit around atomic multi-step changes; do not hold a transaction open during network calls.

```rust
let markets = sqlx::query_as::<_, MarketRow>(
    "SELECT id, name, status FROM markets WHERE status = $1 ORDER BY created_at DESC LIMIT $2"
)
.bind(status)
.bind(limit)
.fetch_all(&pool)
.await?;

let mut transaction = pool.begin().await?;
sqlx::query("INSERT INTO notifications (user_id, message) VALUES ($1, $2)")
    .bind(user_id)
    .bind(message)
    .execute(&mut *transaction)
    .await?;
transaction.commit().await?;
```

Map `MarketRow` to a domain entity inside the repository adapter. Use migrations as the versioned source of schema changes.

## Authentication and Authorization

- Use Keycloak as the initial identity provider and JWT issuer.
- Validate JWT signatures using Keycloak's published keys and check issuer, audience, expiry, and required claims. Cache keys safely and support key rotation.
- Authenticate at Axum boundaries, then authorize every use case against the requested resource. Do not trust user IDs or roles supplied in request bodies.
- Kong can enforce edge policies, but each service must still validate identity and perform resource-level authorization.

## Feature Flags

- Use Flipt for feature-flag evaluation; do not implement a separate flag store or hard-code flag state in the service.
- Keep flag configuration in the GitHub repository designated by the user and use Flipt's supported Git-backed configuration workflow.
- When requirements indicate a feature needs a flag, the architect must ask which GitHub repository contains the Flipt configuration. Record the repository before finalizing the design; never guess it.
- Keep flag evaluation behind an application port so domain rules do not depend on Flipt SDK types. Define safe defaults and behavior when the flag service is unavailable.

## Asynchronous Work and Real-Time

- Use Tokio for asynchronous I/O. Bound concurrency, configure timeouts, and propagate cancellation where practical.
- Use Axum WebSockets for live updates. Authenticate the handshake, authorize each resource/event stream, validate message size and shape, and handle disconnects, backpressure, and token expiry.
- Add a durable queue or outbox only when reliability requirements call for it; do not assume an in-memory task survives process restarts.

## Error Handling and Observability

- Use typed errors within domain/application layers; map them to stable HTTP status and error codes in the Axum adapter.
- Do not return database internals, credentials, stack traces, or sensitive upstream responses to clients.
- Emit structured logs with request/correlation IDs. Redact secrets and personally identifying data.
- Instrument latency, failure rates, database pool saturation, WebSocket connections, and external-service timeouts.

## Testing

- Unit-test domain invariants without I/O.
- Test use cases with fake implementations of application ports.
- Run SQLx repository and migration tests against isolated PostgreSQL.
- Test Axum routes, authentication/authorization, and WebSocket lifecycle at adapter boundaries.
- Test Flipt outage/default behavior and Keycloak key rotation/claim validation.

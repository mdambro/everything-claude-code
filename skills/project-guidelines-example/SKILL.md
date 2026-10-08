---
name: project-guidelines-example
description: Reference architecture and delivery conventions for Flutter clients and Rust backend services.
---

# WorldFlowAI Project Guidelines

Use this skill when planning or implementing WorldFlowAI features. The technology standards here take precedence over generic examples elsewhere in the toolkit.

## Technology Standards

- **Current client**: Flutter/Dart.
- **Likely future client**: Rust with Dioxus, pending platform and ecosystem validation.
- **Backend**: Rust services using Tokio and Axum, including HTTP and WebSocket endpoints.
- **API gateway**: Kong where centralized routing or gateway policies are required.
- **Identity/JWT**: Keycloak is the initial identity provider and token issuer.
- **Feature flags**: Flipt, with configuration in a GitHub repository selected by the user.
- **Database**: PostgreSQL accessed through SQLx.
- **Caching**: Redis only when justified by access patterns and measured need.

Use another application language or framework only when the user explicitly requests it. PostgreSQL is the persistent database; do not add a hosted database platform in place of it.

## Backend Architecture

Apply Clean Architecture with inward-pointing dependencies:

```text
services/api/src/
  domain/          # Entities, value objects, invariants, domain policies
  application/     # Use cases, ports, commands, queries, result models
  infrastructure/  # Axum adapters, SQLx repositories, integrations, wiring
  main.rs          # Composition root
```

- Domain and application code must not depend on Axum, SQLx, or external-service SDKs.
- Define repository and service ports in the application layer; implement them in infrastructure.
- Keep HTTP handlers and WebSocket adapters thin and map transport data at the boundary.
- Keep SQLx rows separate from domain entities and use bound query parameters.
- Prefer denormalized, query-oriented PostgreSQL models. Use transactions, constraints, and idempotent updates to preserve consistency.

## Client Architecture

```text
apps/mobile/lib/
  app/             # Startup, routes, dependency setup
  features/        # Feature screens, state, and data adapters
  shared/          # Truly cross-feature widgets and utilities
```

Keep rendering separate from transport and business decisions. Use stable HTTP contracts for request/response flows and authenticated Axum WebSockets for live data. A future client migration must not require changing backend domain rules.

## Integrations

### Feature Flags

Use Flipt for evaluation and Git-backed flag configuration. During requirements gathering, whenever a feature needs a flag, ask which GitHub repository contains its Flipt configuration. Record the repository and never assume one.

### Gateway and Identity

- Route traffic through Kong when API gateway capabilities are required.
- Validate Keycloak JWT signatures and issuer, audience, expiry, and authorization claims in backend services.
- Enforce resource authorization inside each service; gateway policies do not replace it.

### WebSockets

Authenticate at connection establishment, authorize each resource or event stream, enforce message limits, and define reconnect, token-expiry, and backpressure behavior.

## Date and Time

- Store instants in UTC in PostgreSQL `timestamptz` columns.
- Exchange API timestamps as RFC 3339 with an explicit offset.
- Present times as zero-padded 24-hour `HH:mm:ss` after applying the intended display time zone.
- Preserve IANA time-zone identifiers for future local schedules and define behavior for daylight-saving gaps and overlaps.
- Inject a clock into application use cases and test time-dependent rules deterministically.

## Quality Gates

```bash
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
flutter analyze
flutter test
```

Test domain rules without I/O, application use cases with fake ports, SQLx adapters against isolated PostgreSQL, Axum/WebSocket boundaries with integration tests, and client behavior with widget/integration tests.

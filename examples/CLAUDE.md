# Example Project Instructions

Use this template for a project following the WorldFlowAI technology standards.

## Technology Standards

- Current client: Flutter/Dart.
- Likely future client: Rust with Dioxus, only after confirming target platforms and ecosystem fit.
- Backend: Rust with Tokio and Axum.
- API gateway: Kong when gateway routing/policy is needed.
- JWT identity: Keycloak as the initial issuer.
- Feature flags: Flipt with configuration in the user-designated GitHub repository.
- Database: PostgreSQL through SQLx.

Use another application language or framework only when the user explicitly requests it.

## Architecture

Apply Clean Architecture with inward dependencies:

```text
services/api/src/
  domain/
  application/
  infrastructure/
  main.rs

apps/mobile/lib/
  app/
  features/
  shared/
```

- Keep domain rules independent of Axum, SQLx, and external services.
- Define use cases and ports in application; implement adapters in infrastructure.
- Keep Axum handlers thin and authorize protected resources in the service.
- Keep widgets separate from data access and business decisions.
- PostgreSQL is the persistent store; prefer query-oriented denormalized models.

## Feature Flags

When a requirement needs a feature flag, the architect must ask which GitHub repository holds the Flipt configuration. Record the repository before finalizing the design; never assume it.

## Date and Time

Store instants in UTC as PostgreSQL `timestamptz`; use RFC 3339 at API boundaries and display hours as zero-padded 24-hour `HH:mm:ss` after applying the display time zone.

## Quality Gates

```bash
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
flutter analyze
flutter test
```

## Security

- Validate Keycloak JWT signatures and issuer, audience, expiry, and required claims.
- Use Kong for gateway policy where configured; do not rely on it as the only authorization layer.
- Keep privileged credentials out of client builds.
- Authenticate and authorize Axum WebSocket connections and each subscribed resource.

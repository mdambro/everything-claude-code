---
name: coding-standards
description: Coding standards for Rust backend services and Flutter clients.
---

# Coding Standards

Use Rust for backend services and Flutter/Dart for the current client. Use Dioxus only if the user confirms that future frontend direction. Do not introduce another application language or framework unless explicitly requested.

## General Principles

- Prefer clear names, small cohesive modules, and explicit boundaries.
- Organize backend code by Clean Architecture: `domain`, `application`, and `infrastructure`; dependencies point inward.
- Organize client code by feature and keep presentation, state, and transport concerns distinct.
- Favor simple designs that satisfy current requirements; avoid speculative abstractions.
- Keep public contracts stable and independent of database row types and UI implementation details.

## Rust

- Use `snake_case` for functions, modules, and fields; use `UpperCamelCase` for types and traits.
- Return `Result` for recoverable failures and use typed errors at domain/application boundaries.
- Keep domain invariants in entities and value objects; avoid I/O and framework types in the domain.
- Use Tokio for asynchronous I/O. Avoid blocking work on async executor threads.
- Use Axum handlers as transport adapters: parse and validate requests, invoke use cases, map results to protocol responses.
- Use SQLx with bound values. Keep SQLx rows and connection pools inside infrastructure adapters.
- Use explicit transactions for atomic multi-step writes and idempotency for retryable operations.
- Run `cargo fmt --all`, `cargo clippy --workspace --all-targets -- -D warnings`, and `cargo test --workspace`.

```rust
pub fn update_display_name(
    account: &Account,
    display_name: DisplayName,
) -> Account {
    Account {
        display_name,
        ..account.clone()
    }
}
```

## Flutter and Dart

- Use `UpperCamelCase` for types and widgets, `lowerCamelCase` for members, and `lower_snake_case.dart` for file names.
- Compose focused widgets and keep asynchronous effects outside rendering methods.
- Keep ephemeral state local; introduce shared state only when ownership crosses widgets or features.
- Model loading, success, empty, and failure explicitly.
- Dispose controllers and cancel subscriptions according to widget lifecycle.
- Use immutable state updates where practical.
- Run `dart format .`, `flutter analyze`, and `flutter test`.

## Validation and Errors

- Validate all external input at the boundary and enforce business invariants in the domain.
- Do not expose SQL errors, stack traces, secrets, or internal identifiers in user-facing responses.
- Map typed application errors to stable Axum status codes and response bodies.
- Treat client-side validation as a usability feature, not an authorization or security boundary.

## Security and Integrations

- Obtain service credentials from the runtime environment or a secret manager; never embed privileged secrets in the client.
- Use Keycloak as the initial JWT issuer. Validate signatures, issuer, audience, expiry, and required claims.
- Use Kong for gateway responsibilities when a gateway is needed; preserve service-level authorization.
- Use Flipt for feature flags. When requirements call for a flag, ask which GitHub repository stores the Flipt configuration before finalizing the plan.
- Use authenticated and authorized Axum WebSockets for live updates.

## Quality Checklist

- [ ] Module and dependency boundaries are clear
- [ ] Errors are handled explicitly without leaking internals
- [ ] Database writes are transactional where required
- [ ] Tests cover domain rules, use cases, adapters, and client states
- [ ] Formatting, static analysis, and relevant tests pass

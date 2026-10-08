---
name: tdd-workflow
description: Test-first implementation workflow for Rust backend services and Flutter clients.
---

# Test-Driven Development Workflow

Use the Red-Green-Refactor cycle for feature work, defect fixes, and refactors.

## Backend Test Layers

1. **Domain unit tests** verify invariants and value-object validation without I/O.
2. **Application tests** exercise use cases with fake repository, clock, identity, Flipt, and provider ports.
3. **SQLx integration tests** use isolated PostgreSQL to verify queries, migrations, mapping, constraints, and transaction behavior.
4. **Axum boundary tests** cover parsing, validation, Keycloak token verification, Kong-facing contracts, authorization, and stable error responses.
5. **WebSocket tests** cover handshake authentication, event/resource authorization, payload validation, reconnect, and disconnect cleanup.

## Flutter Test Layers

- Unit-test state transitions, formatters, and pure business presentation logic.
- Widget-test loading, success, empty, failure, and accessibility states.
- Integration-test critical user journeys against controlled backend services.

## Cycle

1. Write one failing test for the requirement.
2. Run that exact test and confirm it fails for the expected reason.
3. Implement the smallest behavior that makes it pass.
4. Rerun the test and nearby suite.
5. Refactor without changing behavior.
6. Run formatting, analysis, and relevant integration checks.

## Commands

```bash
cargo test --workspace
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
flutter test
flutter analyze
```

## Test Quality

- Cover success, invalid input, missing data, dependency failure, authorization denial, and relevant boundary values.
- Inject clocks and external dependencies for deterministic tests.
- Do not rely on production services, fixed wall-clock timing, or shared mutable test state.
- For flags, cover enabled, disabled, and Flipt-unavailable defaults.
- For identity, cover invalid signature, issuer, audience, expiry, and resource permissions.
- Clean up PostgreSQL fixtures and live connection resources after each integration test.

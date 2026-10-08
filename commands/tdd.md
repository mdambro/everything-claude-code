---
description: Enforce test-first implementation for Rust backend services and Flutter clients.
---

# TDD Command

Invoke the `tdd-guide` agent to write a failing test before implementation, then follow Red-Green-Refactor.

## Workflow

1. Restate the behavior and identify its domain, application, infrastructure, or client boundary.
2. Write the smallest failing test at the correct layer.
3. Run that test and confirm the failure matches the requirement.
4. Implement the minimum behavior.
5. Rerun the test, then nearby checks.
6. Refactor while preserving behavior and rerun relevant verification.

## Backend Test Selection

- Domain rule or value object: Rust unit test without I/O.
- Use case: test with fake repository, clock, identity, or Flipt ports.
- SQLx repository/migration: integration test against isolated PostgreSQL.
- Axum HTTP/WebSocket behavior: route or service integration test, including auth and authorization.

## Flutter Test Selection

- Pure client logic: unit test.
- Widget state and rendering: widget test.
- Navigation and backend interaction: integration test.

## Verification Commands

```bash
cargo test --workspace
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
flutter test
flutter analyze
```

Test both Flipt flag outcomes and its safe fallback whenever a feature uses a flag. Inject a clock for time-sensitive rules. Do not use real credentials or production data in tests.

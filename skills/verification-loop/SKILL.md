---
name: verification-loop
description: Build, format, analyze, test, and review loop for Rust services and Flutter clients.
---

# Verification Loop

Run the narrowest useful verification after each change, then broader gates before delivery. Stop after a failed build check, fix the cause, and rerun that exact check.

## Backend Checks

```bash
cargo fmt --all --check
cargo check --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

For persistence changes, run SQLx integration tests against isolated PostgreSQL. For Axum or WebSocket changes, exercise authentication, authorization, error mapping, connection lifecycle, and reconnect behavior.

## Flutter Checks

```bash
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test
```

Run integration tests for critical user journeys and platform-specific builds when the change affects native integration.

## Security and Integration Checks

- Run dependency advisory checks for Rust and Dart packages.
- Confirm Keycloak JWT signature and claim validation for protected routes.
- Confirm Kong policies do not replace service-level authorization.
- For Flipt changes, test enabled, disabled, and unavailable-service defaults.
- Verify secrets are not exposed in client assets, logs, or responses.

## Diff Review

```bash
git diff --check
git diff --stat
git status --short
```

Review modified files for unintended scope, missing error paths, authorization gaps, data consistency, and adequate tests.

## Report

Provide a concise status for formatting, compilation, static analysis, tests, security checks, and diff review. Distinguish passed checks from blocked or unrun checks, and identify the exact failing command when something fails.

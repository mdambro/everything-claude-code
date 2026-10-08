---
description: Plan and run end-to-end user-flow tests across Flutter, Axum, PostgreSQL, identity, gateway, feature flags, and WebSockets.
---

# End-to-End Verification

Use the `e2e-runner` agent for critical user journeys that span client and backend boundaries.

## Workflow

1. Select the user-visible flow and define expected success and failure states.
2. Identify required local services: Flutter app, Axum API, isolated PostgreSQL, test Keycloak realm, Kong routes, and Flipt configuration as applicable.
3. Seed deterministic test data and confirm credentials and flag state are test-only.
4. Run Rust integration tests for API and persistence boundaries.
5. Run Flutter integration tests for navigation, forms, API results, and live updates.
6. Capture logs and correlation IDs with credentials and personal data redacted.
7. Report passed, failed, and blocked steps separately.

## Required Coverage

- Authentication through Keycloak-issued tokens and authorization of protected resources.
- Gateway routing and rate limits when Kong participates in the flow.
- Feature behavior for each relevant Flipt decision and the unavailable-service default.
- PostgreSQL writes, reads, and transaction behavior through the Axum API.
- WebSocket authentication, subscription authorization, delivery, reconnect, and disconnect handling.
- Flutter loading, success, empty, validation, server-error, and offline states.

## Commands

```bash
cargo test --workspace
flutter test integration_test
```

Do not use production credentials or data. State explicitly when an external service is unavailable and its behavior could not be verified.

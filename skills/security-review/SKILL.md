---
name: security-review
description: Security checklist for Rust/Tokio/Axum services, PostgreSQL/SQLx, Flutter, Keycloak, Kong, and Flipt.
---

# Security Review

Use this checklist for features that handle identity, user input, persistence, external services, live connections, or sensitive data. Review actual trust boundaries, not only framework defaults.

## Secrets and Configuration

- Keep server credentials in the deployment secret manager or protected environment configuration.
- Fail startup when required secrets or issuer configuration are missing or invalid.
- Never embed database, gateway-admin, identity-provider, or Flipt service credentials in the client.
- Rotate exposed credentials and review logs and repository history for accidental disclosure.

## Input and Output Boundaries

- Validate request and message payloads at Axum HTTP and WebSocket boundaries.
- Enforce body/message size limits, timeouts, and rate limits.
- Treat client-side validation as usability only; repeat security and domain validation on the server.
- Return stable error codes without SQL details, stack traces, tokens, or sensitive upstream responses.
- Encode or sanitize any user-provided content before rendering it in the Flutter client.

## PostgreSQL and SQLx

- Bind all user-controlled SQL values; allow-list dynamic table/column identifiers.
- Use least-privilege database roles and server-side connection pools.
- Use transactions for atomic changes and idempotency for retryable requests.
- Review migrations, access patterns, constraints, and indexes for data isolation.
- Redact personal data and credentials from logs; encrypt backups and test restoration.

```rust
let record = sqlx::query_as::<_, RecordRow>(
    "SELECT id, owner_id, state FROM records WHERE id = $1"
)
.bind(record_id)
.fetch_optional(&pool)
.await?;
```

## Keycloak JWTs

- Use Keycloak as the initial issuer; validate the signature against trusted published keys.
- Verify issuer, audience, expiry, and required scopes/roles; support key rotation.
- Enforce resource authorization in each service even when Kong performs gateway checks.
- Do not accept identity or permission claims supplied outside the verified token.

## Kong Gateway

- Require TLS and review route exposure, allowed methods, request-size limits, and rate limits.
- Restrict direct access to services intended to be reachable only through Kong.
- Keep service-level authentication and authorization enabled; the gateway is not the only security boundary.

## Flipt Feature Flags

- Use Flipt for flag evaluation and the user-designated Git repository for flag configuration.
- Ask for the GitHub repository whenever requirements include a feature flag.
- Restrict who can change flag configuration and audit changes.
- Define a safe default for Flipt outages and test both flag outcomes.

## Axum WebSockets

- Authenticate the handshake and authorize every resource or event-stream subscription.
- Validate message type, payload size, and rate.
- Handle token expiry, permission changes, backpressure, reconnects, and cleanup.

## Flutter Client

- Store session material using platform-secure storage and never log credentials or sensitive payloads.
- Validate deep links and imported files before processing.
- Assume client state can be modified; make all security decisions on the backend.

## Release Checks

```bash
cargo audit
cargo deny check
cargo clippy --workspace --all-targets -- -D warnings
flutter analyze
flutter test
```

Before release, confirm authentication, authorization, injection, rate limiting, secret handling, dependency advisories, backup protection, and monitoring for security events.

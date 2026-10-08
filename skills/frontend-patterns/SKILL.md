---
name: frontend-patterns
description: Flutter frontend patterns and the likely future Rust/Dioxus direction.
---

# Frontend Development Patterns

Flutter is the current client framework. Rust with Dioxus is the likely future direction, pending validation of target platforms and ecosystem maturity. Do not introduce another frontend framework unless the user explicitly requests it.

## Feature-Oriented Structure

Organize code around user-visible capabilities rather than creating broad utility layers prematurely:

```text
lib/
  app/                 # Application startup, routing, and dependency wiring
  features/
    account/
      data/            # API adapters and serialization
      domain/          # Feature-specific models and rules
      presentation/    # Screens, widgets, and view state
  shared/              # Small, genuinely cross-feature components
```

Keep feature APIs narrow. Share a widget or model only when its meaning and lifecycle are truly shared.

## Widget Composition

- Compose screens from small widgets with one clear visual responsibility.
- Pass immutable data and explicit callbacks across presentation boundaries.
- Keep layout and rendering separate from network access and business decisions.
- Use `const` constructors for immutable widgets where applicable.
- Provide stable keys for repeated dynamic children.

```dart
class AccountSummary extends StatelessWidget {
  const AccountSummary({required this.account, super.key});

  final AccountViewData account;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(account.displayName),
        Text(account.formattedBalance),
      ],
    );
  }
}
```

## State and Effects

- Keep ephemeral view state close to the widget that owns it.
- Put feature state in an explicit state holder when multiple widgets share it or when it coordinates asynchronous work.
- Represent loading, success, empty, and failure states explicitly; avoid unrelated booleans that permit contradictory states.
- Do not perform network or database work inside `build`.
- Cancel subscriptions and dispose controllers with the owning lifecycle.
- Keep state immutable where practical and make transitions testable.

## API and WebSocket Boundaries

- Route all data access through a typed client boundary; do not embed transport details in widgets.
- Use stable HTTP contracts for requests and initial data, and authenticated Axum WebSockets for live updates.
- Treat client-side validation as usability support only; the Rust backend remains authoritative.
- Model reconnecting, stale data, permission failures, and offline states explicitly.
- Never embed server credentials, Keycloak client secrets, or privileged Flipt credentials in the client.

## Accessibility and Layout

- Use semantic controls, labels, focus behavior, and sufficient contrast.
- Support text scaling, keyboard navigation, and narrow screens.
- Avoid fixed sizes for text-bearing content; test long labels, empty states, and validation errors.
- Keep interactive targets large enough and expose loading and error states to assistive technology.

## Testing

- Unit-test pure presentation logic and state transitions.
- Widget-test loading, empty, success, error, and accessibility-relevant states.
- Integration-test critical navigation and API/WebSocket workflows against controlled services.
- Run `flutter analyze`, `dart format --output=none --set-exit-if-changed .`, and `flutter test` before completion.

## Future Dioxus Direction

If Dioxus is selected, preserve feature boundaries and backend contracts. Keep Rust UI state and rendering separate from application/domain rules, avoid coupling the client directly to server infrastructure, and validate the framework's supported deployment targets before migration.

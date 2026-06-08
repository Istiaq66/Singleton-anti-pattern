# Singleton-anti-pattern
Singleton is often called an anti-pattern because it can create more problems than it solves, especially in large applications. However, it's not always bad—there are valid use cases.


# 🚫 Why Singleton is Often an Anti-Pattern

> **TL;DR** — Singletons feel convenient but silently make your code harder to test, reason about, and maintain.

---

## What is a Singleton?

A Singleton ensures **only one instance** of a class exists globally, accessible from anywhere.

```dart
class AuthService {
  static final AuthService _instance = AuthService._internal();
  factory AuthService() => _instance;
  AuthService._internal();

  bool isLoggedIn = false;
}

// Usage anywhere in the codebase:
AuthService().isLoggedIn = true;
```

Looks handy. So what's the problem?

---

## ⚠️ The Problems

### 1. Hidden Dependencies
Classes that use a Singleton don't declare it in their constructor — the dependency is **invisible**.

```dart
// ❌ No way to tell this widget depends on AuthService
class ProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    if (AuthService().isLoggedIn) { ... } // sneaky global call
  }
}
```
You can't understand a class in isolation — you must hunt for hidden globals.

---

### 2. Testing Difficulties
You **can't swap** a Singleton for a fake/mock in tests. The global instance is always baked in.

```dart
// ❌ Can't inject a MockAuthService here — it's hardcoded
testWidgets('shows profile when logged in', (tester) async {
  AuthService().isLoggedIn = true; // mutating shared global state in tests = 💥
  await tester.pumpWidget(ProfilePage());
});
```
Tests can bleed state into each other, causing flaky, order-dependent failures.

---

### 3. Global Mutable State
Singletons are typically mutable and shared across the entire app. **Any code anywhere can change the state**, making bugs hard to trace.

```dart
// Widget A sets it
AuthService().isLoggedIn = true;

// Widget B resets it unexpectedly
AuthService().isLoggedIn = false;

// Widget C is now broken — but why?
```

---

### 4. Tight Coupling
Code that calls `AuthService()` directly is **glued** to that exact class. Swapping implementations (e.g., for a different auth provider) requires finding and changing every call site.

---

## ✅ Prefer Dependency Injection (DI)

Pass dependencies **explicitly** through constructors or `Provider`/`Riverpod`. This makes dependencies visible, swappable, and testable.

```dart
// ✅ Dependency is declared openly
class ProfilePage extends StatelessWidget {
  final AuthService authService;
  const ProfilePage({required this.authService});

  @override
  Widget build(BuildContext context) {
    if (authService.isLoggedIn) { ... }
  }
}

// In tests, inject a mock
testWidgets('shows profile when logged in', (tester) async {
  await tester.pumpWidget(ProfilePage(authService: MockAuthService()));
});
```

With `Provider` / `Riverpod`, DI scales cleanly across the widget tree without prop-drilling.

---

## 📌 Practical Takeaways

| | Singleton | Dependency Injection |
|---|---|---|
| Dependencies visible? | ❌ Hidden | ✅ Explicit |
| Testable? | ❌ Hard to mock | ✅ Inject mocks freely |
| State predictable? | ❌ Global mutation | ✅ Scoped & controlled |
| Swappable impl? | ❌ Tightly coupled | ✅ Program to interface |

- **Avoid** reaching for `static` instances or `ServiceLocator.instance` as a reflex.
- **Do** inject dependencies via constructors, `Provider`, or `Riverpod`.
- A Singleton **can** be acceptable for truly stateless utilities (e.g., a logger, a date formatter) — but even then, prefer injecting it.

> **Rule of thumb:** If a Singleton holds mutable state or is called in business logic — reach for DI instead.
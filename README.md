# Flutter Cheat sheet
---

## 🔧 Flutter Fundamentals

### 1. Stateless vs Stateful Widgets

* **StatelessWidget**:

  * Immutable.
  * UI doesn't depend on state changes.
  * Rebuilds only when parent changes.
  * Example: Static content like splash screen or about page.

* **StatefulWidget**:

  * Mutable via `State` class.
  * UI depends on dynamic data (user input, timers).
  * Lifecycle methods available: `initState`, `didUpdateWidget`, `dispose`.
  * Example: Form inputs, toggles, dynamic content.

**Choose based on dynamicity of content and interaction requirements.**

---

### 2. Keys

* **Purpose**: Retain widget identity across rebuilds.
* **Types**:

  * `Key` (base), `ValueKey`, `ObjectKey`, `UniqueKey`, `GlobalKey`.

**Use Cases**:

* Optimizing ListView performance.
* Maintaining state during item reordering.
* `GlobalKey` for accessing widget state (e.g., forms).

---

### 3. Extension Methods

* **Add new functionality to existing classes** without subclassing.

```dart
extension CapExtension on String {
  String capitalize() => this[0].toUpperCase() + substring(1);
}
```

* **Benefits**:

  * Improves readability.
  * Keeps utility functions near relevant types.

---

### 4. `isMounted`

* Property in `State` objects to check if widget is in tree.
* **Avoids**: `setState() called after dispose()` exception.
* **Best Practice**: Check `if (mounted)` before calling `setState` in async callbacks.

---

### 5. Localization (`l10n`)

* **Tools**: `flutter_localizations`, `intl`, `.arb` files.
* **Command**: `flutter gen-l10n`
* **Pluralization**: Use `Intl.plural(...)`

**Challenges**:

* Keeping .arb files in sync.
* Misnamed keys or malformed JSON.

---

## ⚖️ State Management

### Approaches:

| Approach     | Pros                                  | Cons                          |
| ------------ | ------------------------------------- | ----------------------------- |
| **Provider** | Simple, official, great for beginners | Not scalable alone            |
| **Riverpod** | Compile-safe, declarative, testable   | Learning curve                |
| **BLoC**     | Structured, scalable, widely used     | Boilerplate-heavy             |
| **Redux**    | Predictable, unidirectional           | Verbose, outdated for Flutter |

**Decision Factors**:

* Team experience
* App complexity
* Developer familiarity

**For junior teams**: Prefer Provider or Riverpod.
**Avoid**: BLoC/Redux unless training is given.

**When to avoid your preferred method?**

* When team lacks experience with it.
* When rapid prototyping is needed.
* If debugging complexity outweighs benefits.

---

## 🎨 UI/UX Design

### Responsive Layouts

* `LayoutBuilder`, `MediaQuery`, `FractionallySizedBox`, `Flexible`.
* `flutter_screenutil`, `responsive_builder`, `sizer`.
* Adaptive layouts for tablets and phones.

### Animations

* **Implicit**: `AnimatedContainer`, `AnimatedOpacity`
* **Explicit**: `AnimationController`, `Tween`, `AnimationBuilder`
* **Transitions**: `PageRouteBuilder`, `Hero`, `FadeTransition`, `SlideTransition`
* **Debug Performance**: DevTools timeline, `TickerMode`, frame budget awareness.

---

## 🔌 API Integration

### Networking

* **Libraries**: `http`, `Dio`, `Chopper`
* **Layering**:

  * Data Layer (API services)
  * Domain Layer (Use Cases)
  * Presentation Layer (Widgets/BLoC)
* **Error Handling**: Dio interceptors, sealed result classes

---

## 📈 Performance Optimization

### Strategies:

* Use `const` widgets where possible.
* Avoid rebuilding large trees unnecessarily.
* Use `RepaintBoundary` to isolate re-renders.
* Debounce/throttle user input.

### Detection:

* Use `Flutter DevTools` (Rebuild profiler, Timeline).
* Check widget rebuilds with `debugPrintBuild()`.

### Startup Time:

* Minimize `initState` work.
* Use deferred loading for heavy widgets.
* Optimize asset loading.

---

## ⚙️ Flutter Architecture

### InheritedWidget

* Low-level dependency injection.
* Used by `Provider`, `Theme.of()`, `MediaQuery.of()`.
* Used when child widgets need access to ancestor data.

### Patterns Used:

* **MVVM**: With Provider or Riverpod.
* **Clean Architecture**: Domain-driven separation.
* **MVC**: Rare in modern Flutter.

### Modularization

* Feature-first or layer-based packages.
* Shared module for design system/constants.
* Handle dependencies using DI frameworks.

### Navigation

* **Navigator 1.0**: Imperative.
* **Navigator 2.0**: Declarative (used by `go_router`).
* **go\_router**:

```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/home', builder: (context, state) => HomePage()),
  ],
);
```

* Benefits: Deep linking, guards, declarative structure.

---

## ⏳ Async Programming

### Dart Constructs:

* `Future`: For one-time async results.
* `Stream`: For multiple async events (like WebSockets).
* `FutureBuilder`, `StreamBuilder`: For UI binding.

---

## 📱 Native Integration

* **MethodChannels**: Bridge between Dart and platform code.
* **PlatformView**: Embed native views.
* **FFI**: For native C/C++ interop.
* **Challenges**: Permissions, testing, maintenance.

---

## ❌ Error Handling

* Wrap async code in `try/catch`.
* Use `Either` / `Result` types for functional error propagation.
* Show errors with `SnackBar`, `Dialogs`, or error screens.

---

## 🧪 Dependency Injection

### Tools:

* **GetIt**: Service locator.
* **Provider**: Scoped injection.
* **Riverpod**: Declarative, better scoping and testing.

### Scoped DI:

* Provide different instances using `MultiProvider`, `ScopedProvider`.
* Example: AuthService at root, user-specific service deeper.

### Refactoring for DI:

* Extract dependencies to constructor params.
* Replace singletons with injectable services.
* Test by mocking dependencies.

---

## 🌍 Localization & Accessibility

* **Localization**: `.arb` files, `flutter_localizations`, `intl`.
* **Accessibility**:

  * `Semantics` widget.
  * Label images with `alt`.
  * Avoid hardcoded sizes.
  * Test with screen readers.

---

## 🧪 Testing & Quality

### Testing Pyramid:

* **Unit Tests**: Logic, use cases.
* **Widget Tests**: UI widgets, user interactions.
* **Integration Tests**: End-to-end scenarios.

### Tools:

* `flutter_test`, `mockito`, `bloc_test`, `integration_test`

### CI/CD:

* **GitHub Actions**, **Bitrise**, **Codemagic**.
* Stages: Lint -> Build -> Test -> Release.

### Static Analysis:

* `flutter analyze`, `dart analyze`, `very_good_analysis`, `custom_lint`

### Linting:

* Use `.analysis_options.yaml`.
* Auto-formatting via IDE or CLI.

### Code Quality

* Code reviews: Check for patterns, state separation, testability.
* Static analysis: Lint rules and warnings.
* Tools: `dart_code_metrics`, `SonarQube`, `Danger`.

### Accessibility Testing

* Test with screen readers and accessibility audit tools.
* Ensure contrast, tap targets, focusable widgets.

---

## 📦 Flutter Packages

* `flutter_svg`, `fl_chart`, `cached_network_image`, `riverpod`, `hooks_riverpod`, `lottie`, `go_router`, `get_it`

**Evaluate**:

* Maintenance status
* Community support
* License
* Complexity vs benefit

---

## 📚 Version Control & Environments

### Git Best Practices

* **Branching**:

  * GitFlow: `main`, `develop`, `feature/*`
  * Trunk-based: single main branch + short-lived branches

* **Commits**: Use semantic messages (`feat:`, `fix:`)

* **PRs**: Short, reviewed, linked to tasks

### Environments

* Use `.env`, `flutter_dotenv`, or `flavors` (dev, staging, prod).
* Feature flags via remote config or in-code switches.

---

## 🧠 Dart Language Proficiency

### Language Features:

* Null safety, type inference, extension methods.
* `late`, `required`, `typedef`, mixins.

### Error Handling

* Use `try/catch`, custom exceptions.
* Propagate or wrap with Either/Result.

### Performance Optimization:

* Avoid unnecessary object allocation.
* Prefer unmodifiable collections for data safety.
* Understand AOT/JIT and memory profiling.

### Anti-patterns:

* Overusing `setState`.
* Business logic in widgets.
* Ignoring async errors.

---

## ✅ Problem Solving

### Live Coding Practice:

* [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)
* [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
* [LRU Cache](https://leetcode.com/problems/lru-cache/)

---

## 🔍 Flutter Architecture & Framework

* **State Management Decisions**:

  * Use Provider for simple state needs and for teams newer to Flutter.
  * Use Riverpod for safe, modular, testable, and scalable apps.
  * BLoC is suitable for enterprise-grade apps requiring strict state separation.
  * Avoid Redux unless working with developers familiar with web/state mutation debugging.
  * In junior teams, prefer Provider or Riverpod to reduce complexity.

* **Dependency Injection (DI)**:

  * Prefer `get_it` for simple service locator pattern.
  * Use Riverpod's `ProviderScope` for context-aware DI.
  * Handle scoped dependencies with nested providers or DI containers.
  * Testing is easier when using dependency injection—mock services in tests easily.
  * To refactor legacy code: extract services from constructors and centralize logic into injectable classes.

* **Performance Optimization**:

  * Use `const` constructors.
  * Avoid unnecessary rebuilds by breaking widgets down and using `shouldRebuild` and `ValueKey`s.
  * Use DevTools, track jank frames, and diagnose layout overdraws.
  * Place expensive widgets in `RepaintBoundary`.
  * Profile build methods that re-run too often.

* **Animations**:

  * Use implicit animations (`AnimatedContainer`, etc.) for simple use cases.
  * Use `AnimationController`, `Tween`, `AnimationBuilder` for explicit control.
  * Custom transitions: `PageRouteBuilder`, `SharedAxisTransition` from animations package.
  * Debug animations using timeline in Flutter DevTools.

---

## 💡 Dart Proficiency

* **Key Dart Features**:

  * Null safety, extension methods, `sealed`, `late`, type inference.
  * Extension methods improve modularity, e.g., `.px`, `.capitalize()` on common types.

* **Error Handling**:

  * Use `try/catch`, `Result` classes, or sealed failure models for robust APIs.
  * Consider wrapping exceptions in failure models for service layers.

* **Performance**:

  * Profile memory allocations.
  * Use lazy initializations (`late`) and avoid unintentional list mutations.
  * Watch for leaks via retained closures or open streams.

---

## 🧪 Testing & Quality

* **Testing Strategy**:

  * Follow pyramid: 70% unit, 20% widget, 10% integration.
  * Mock APIs with `mockito`, `mocktail`, or `http/testing`.
  * Use golden tests for visual regressions.

* **Code Quality**:

  * Enforce `very_good_analysis` or custom `analysis_options.yaml`.
  * Lint on CI with GitHub Actions, Codemagic, etc.

* **Accessibility**:

  * Use `Semantics`, `ExcludeSemantics`, `MergeSemantics`.
  * Test with screen readers on Android/iOS.

---

## 🚀 Project Experience

* **Complex Projects**:

  * Modularized architecture using feature-based packages.
  * Used GoRouter for unified routing with nested paths and deep linking.
  * Isolated business logic in domain layer using clean architecture.

* **Environment Configs**:

  * Used `flutter_dotenv`, `--dart-define`, build flavors.
  * Feature flagging via Firebase Remote Config or local toggles.

* **Navigation**:

  * Prefer `go_router` for declarative navigation with nested routes.
  * Support deep linking via `GoRoute` with `redirect`, `extra`, and `params`.
  * Modular navigation by defining `routes.dart` per module.

---

## 📦 CI/CD & Deployment

* **CI/CD**:

  * Codemagic / GitHub Actions pipelines with build, test, and deploy jobs.
  * Use `fastlane` for iOS and Android builds and Play/App Store submission.

* **Versioning & Releases**:

  * Semantic versioning via `pubspec.yaml`, Fastlane for tagging.
  * Migrations handled with DB scripts and feature toggles.

* **Store Submissions**:

  * Used staged rollout, monitored via Crashlytics.
  * Prepared metadata and store listing automation.

---

## 🧠 Leadership & Management

* **Team Structure**:

  * Organize by features or layers (UI, Domain, Data).
  * Onboard new devs with starter tasks, code walkthroughs, architecture docs.

* **Disagreements**:

  * Use RFCs or ADRs (Architecture Decision Records).
  * Encourage pair programming for resolving architectural choices.

* **Team Growth**:

  * Weekly code-along or POC sessions.
  * Shared internal knowledge base / wiki.

---

## 🔄 Dev Practices

* **Code Reviews**:

  * Check readability, performance, test coverage.
  * Avoid nitpicking; focus on design patterns, data flow, and testability.

* **Standardization**:

  * Use formatter (`dart format`) and custom lint rules.
  * Enforce using pre-commit hooks (`pre-commit`, `husky`, etc.).

* **Managing Tech Debt**:

  * Label TODOs with issue references.
  * Prioritize by impact, and communicate clearly with stakeholders.

---

## 🧩 Problem Solving

* **Debugging Performance**:

  * Use `debugProfileBuildsEnabled`, `debugRepaintRainbowEnabled`.
  * DevTools memory and CPU profiler to find leaks and bottlenecks.

* **Working with Limited Docs**:

  * Dive into source code, GitHub, and community forums.
  * Build POCs and validate via tests before adoption.

---

## 🧱 Architecture Decisions

* **Key Decisions**:

  * Migrated from deep Provider trees to scoped Riverpod + hooks.
  * Modularized routing with GoRouter.

* **Multi-Platform Strategy**:

  * UI abstraction: Platform-aware widgets or builders.
  * Common domain and data logic in core/shared packages.

---

## 🔍 Technical Deep Dives

* **Shopping Cart**:

  * State via Riverpod Notifier, persisted in SharedPrefs or SQLite.
  * UI watches discount state and recomputes total dynamically.

* **Platform Integration**:

  * Used `MethodChannel` for battery, connectivity, permissions.
  * Validated via integration tests + mocks.

* **Component Libraries**:

  * Shared design system with `ThemeData` overrides.
  * Custom widgets with flexible APIs, named constructors, and documentation.

---

## 🔮 Trends & Vision

* **Staying Current**:

  * Follow Flutter Dev YouTube, Flutter Weekly, Medium blogs.
  * Try beta/dev releases in sandbox apps.

* **Future of Flutter**:

  * Dart Macros and static metaprogramming.
  * Better Wasm, Impeller engine improvements.

---

## 🎭 Situational Handling

* **Quality vs Speed**:

  * Balance MVP delivery with clear refactor plans.
  * Communicate debt trade-offs to product.

* **Legacy Codebase**:

  * Snapshot current state, gradually replace modules.
  * Add tests as refactoring progresses.

---

## ✅ Bonus Practice Links

* [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)
* [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
* [LRU Cache](https://leetcode.com/problems/lru-cache/)
* [Grade Calculator](https://www.rapidtables.com/calc/grade/test-calculator.html)



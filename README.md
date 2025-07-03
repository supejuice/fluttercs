# Flutter Cheat Sheet

A comprehensive guide for Flutter development, covering fundamentals, architecture, state management, UI/UX, API integration, performance, testing, and best practices.

---

## 🔧 Flutter Fundamentals

### Stateless vs Stateful Widgets

- **StatelessWidget**: Immutable, UI does not depend on state changes. Rebuilds only when parent changes.  
  _Example_: Static content (splash screen, about page).
- **StatefulWidget**: Mutable via `State` class, UI depends on dynamic data. Lifecycle methods: `initState`, `didUpdateWidget`, `dispose`.  
  _Example_: Forms, toggles, dynamic content.

> **Tip:** Choose based on content dynamicity and interaction needs.

---

### Keys

- **Purpose**: Retain widget identity across rebuilds.
- **Types**: `Key`, `ValueKey`, `ObjectKey`, `UniqueKey`, `GlobalKey`.
- **Use Cases**:  
  - Optimizing `ListView` performance  
  - Maintaining state during item reordering  
  - Accessing widget state (`GlobalKey` for forms)

---

### Extension Methods

Add new functionality to existing classes without subclassing.

```dart
extension CapExtension on String {
  String capitalize() => this[0].toUpperCase() + substring(1);
}
```

- **Benefits**: Improves readability, keeps utilities near relevant types.

---

### `isMounted`

- Property in `State` objects to check if widget is in the tree.
- Prevents `setState()` after `dispose()`.
- **Best Practice**: Check `if (mounted)` before calling `setState` in async callbacks.

---

### Localization (`l10n`)

- **Tools**: `flutter_localizations`, `intl`, `.arb` files.
- **Command**: `flutter gen-l10n`
- **Pluralization**: `Intl.plural(...)`
- **Challenges**: Syncing `.arb` files, key naming, malformed JSON.

---

## ⚖️ State Management

| Approach     | Pros                                  | Cons                          |
| ------------ | ------------------------------------- | ----------------------------- |
| Provider     | Simple, official, beginner-friendly   | Not scalable alone            |
| Riverpod     | Compile-safe, declarative, testable   | Learning curve                |
| BLoC         | Structured, scalable, widely used     | Boilerplate-heavy             |
| Redux        | Predictable, unidirectional           | Verbose, outdated for Flutter |

- **Decision Factors**: Team experience, app complexity, familiarity.
- **For junior teams**: Prefer Provider or Riverpod.
- **Avoid**: BLoC/Redux unless team is trained.

---

## 🎨 UI/UX Design

### Responsive Layouts

- Use: `LayoutBuilder`, `MediaQuery`, `FractionallySizedBox`, `Flexible`.
- Packages: `flutter_screenutil`, `responsive_builder`, `sizer`.
- Design adaptive layouts for tablets and phones.

**MediaQuery rebuilds:**  
Widgets rebuild only if they read `MediaQuery` and the data changes (e.g., orientation).

**Responsive Grid Example:**

```dart
LayoutBuilder(
  builder: (context, constraints) {
    int columns = 1;
    if (constraints.maxWidth >= 1200) columns = 4;
    else if (constraints.maxWidth >= 800) columns = 3;
    else if (constraints.maxWidth >= 600) columns = 2;
    return GridView.count(
      crossAxisCount: columns,
      children: List.generate(20, (index) => Card(child: Text('Item $index'))),
    );
  },
)
```

---

**ConstrainedBox**: Set min/max constraints.

```dart
ConstrainedBox(
  constraints: BoxConstraints(
    minWidth: 100, maxWidth: 200, minHeight: 50, maxHeight: 100,
  ),
  child: YourWidget(),
)
```

**SizedBox**: Fixes width/height or acts as spacing.

---

**Flexible vs Expanded**

- `Expanded(child: ...)` = `Flexible(fit: FlexFit.tight, child: ...)`
- `Flexible(flex: 2, child: ...)` uses `FlexFit.loose` by default.

```dart
Row(
  children: [
    Flexible(flex: 1, child: Text("Short")),
    Flexible(flex: 2, child: Container(width: 50, color: Colors.red)),
  ],
);
```

- To force expansion: `Flexible(flex: 2, fit: FlexFit.tight, child: ...)` or `Expanded(flex: 2, child: ...)`

---

**FractionallySizedBox**: Sizes child relative to parent.

```dart
FractionallySizedBox(
  widthFactor: 0.8,  // 80% of parent width
  heightFactor: 0.5, // 50% of parent height
  child: YourWidget(),
)
```

---

**Sizer**: Sizing widgets as a percentage of the screen.

```dart
Text('Hello', style: TextStyle(fontSize: 12.sp));
Container(height: 20.h, width: 50.w);
```

- **Limitations**: Not pixel-perfect, limited padding/margin scaling.

---

**flutter_screenutil**: Advanced scaling based on design size.

```dart
ScreenUtilInit(
  designSize: Size(360, 690),
  builder: () => MyApp(),
)
Container(width: 100.w, height: 50.h);
Text("Hello", style: TextStyle(fontSize: 14.sp));
```

- **Benefits**: Precise scaling, supports paddings/margins/border radii.

---

**Animations**

- **Implicit**: `AnimatedContainer`, `AnimatedOpacity`
- **Explicit**: `AnimationController`, `Tween`, `AnimationBuilder`
- **Transitions**: `PageRouteBuilder`, `Hero`, `FadeTransition`, `SlideTransition`
- **Debug**: DevTools timeline, `TickerMode`

---

## 🔌 API Integration

### Authorization

- **Access Token**: Short-lived, sent with every request, stored in memory/secure storage.
- **Refresh Token**: Long-lived, used to get new access tokens, stored securely.

| Token Type    | Analogy                      |
|---------------|-----------------------------|
| Access Token  | Room key card (limited time)|
| Refresh Token | Passport + reissue form     |

- **Separation**: Improves security, allows session rotation, scalable backend.

---

**Login Endpoint**

- Accepts credentials, issues tokens, limits attempts, hashes passwords, logs IP/user agent.

**Refresh Endpoint**

- Accepts only valid refresh tokens, rotates on use, blocks reuse, can bind to device/IP, expires after set period.

**Token Structure**

| Property         | Access Token        | Refresh Token                |
|------------------|--------------------|------------------------------|
| Format           | JWT or opaque      | Opaque (preferred)           |
| Storage          | Memory/secure      | Secure only                  |
| Contains         | Claims             | UUID only                    |
| Signed           | Yes                | Preferably yes               |

---

### Networking

**Base Configuration**

```dart
final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com/v1',
  connectTimeout: Duration(seconds: 10),
  receiveTimeout: Duration(seconds: 15),
  headers: {'Accept': 'application/json', 'Content-Type': 'application/json'},
));
```

**JWT Auth Interceptor**

```dart
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) {
    final token = yourTokenStorage.getAccessToken();
    if (token != null) options.headers['Authorization'] = 'Bearer $token';
    return handler.next(options);
  },
));
```

**Centralized Error Handling**

```dart
dio.interceptors.add(InterceptorsWrapper(
  onError: (DioError err, handler) async {
    if (err.response?.statusCode == 401) {
      await storage.clear();
      navigatorKey.currentState?.pushNamedAndRemoveUntil('/login', (r) => false);
    }
    return handler.next(err);
  },
));
```

**Retry on Failure**

```dart
dio.interceptors.add(
  RetryInterceptor(
    dio: dio,
    retries: 3,
    retryDelays: [Duration(seconds: 1), Duration(seconds: 2)],
    retryEvaluator: (error) => error.type != DioExceptionType.cancel,
  ),
);
```

**File Upload**

```dart
final formData = FormData.fromMap({
  'file': await MultipartFile.fromFile(filePath, filename: 'image.jpg'),
});
final response = await dio.post('/upload', data: formData);
```

**Caching**

```dart
options.headers['Cache-Control'] = 'public, max-age=3600';
```

**Request Cancellation**

```dart
final cancelToken = CancelToken();
dio.get('/longrequest', cancelToken: cancelToken);
cancelToken.cancel("User navigated away");
```

**Logging (Debug Only)**

```dart
dio.interceptors.add(LogInterceptor(requestBody: true, responseBody: true));
```

**Sample GET with Query Params**

```dart
final response = await dio.get('/users', queryParameters: {'page': 1, 'limit': 20});
```

**Timeout Settings**

```dart
dio.get('/slow', options: Options(receiveTimeout: Duration(seconds: 5)));
```
## Performance Optimization

- Prefer `const` constructors for widgets that do not change, to reduce rebuilds.
- Use `RepaintBoundary` to isolate parts of the widget tree that update frequently, minimizing unnecessary repaints.
- Debounce or throttle user input (e.g., search fields) to avoid excessive rebuilds or API calls.
- Profile and detect performance issues using Flutter DevTools and `debugPrintBuild()`.

**Example: Using const and RepaintBoundary**

```dart
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      child: const Text('Static content'),
    );
  }
}
```

- Minimize heavy work in `initState`. For expensive initialization, use `FutureBuilder` or deferred loading.
- Optimize asset sizes and use compressed images.

---

## Flutter Architecture

- Use `InheritedWidget` for low-level dependency injection. Most state management solutions (like Provider) build on this.
- Common patterns: MVVM (with Provider or Riverpod), Clean Architecture (separating domain, data, and presentation layers).
- Modularize code by features or layers. Extract shared UI components and utilities into separate packages.
- Navigation:
  - Navigator 1.0: Imperative, push/pop routes.
  - Navigator 2.0: Declarative, supports deep linking and complex flows. Use packages like `go_router`.

**Example: go_router setup**

```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/home',
      builder: (context, state) => HomePage(),
    ),
  ],
);
```

---

## Async Programming

- Use `Future` for single async results, `Stream` for multiple events.
- Bind async data to UI with `FutureBuilder` and `StreamBuilder`.

**Example: FutureBuilder**

```dart
FutureBuilder<int>(
  future: fetchValue(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return CircularProgressIndicator();
    } else if (snapshot.hasError) {
      return Text('Error: ${snapshot.error}');
    }
    return Text('Value: ${snapshot.data}');
  },
)
```

---

## Native Integration

- Use `MethodChannel` to call platform-specific code (Android/iOS) from Dart.
- Use `PlatformView` to embed native UI components.
- Use FFI (Foreign Function Interface) for calling native C/C++ code.
- Handle permissions and platform-specific testing carefully.

**Example: MethodChannel**

```dart
static const platform = MethodChannel('samples.flutter.dev/battery');

Future<void> getBatteryLevel() async {
  final int batteryLevel = await platform.invokeMethod('getBatteryLevel');
}
```

---

## Error Handling

- Wrap async code in `try/catch` blocks.
- Use types like `Either` or `Result` for functional error handling.
- Show errors to users with `SnackBar`, dialogs, or dedicated error screens.

**Example: try/catch**

```dart
try {
  final data = await fetchData();
} catch (e) {
  ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Error: $e')));
}
```

---

## Dependency Injection

- Use GetIt for service locator pattern, Provider or Riverpod for scoped dependency injection.
- Prefer passing dependencies via constructors for testability.
- Use `MultiProvider` or `ProviderScope` for grouping dependencies.

**Example: GetIt registration**

```dart
final getIt = GetIt.instance;
getIt.registerSingleton<ApiService>(ApiServiceImpl());
```

---

## Localization and Accessibility

- Use `.arb` files, `flutter_localizations`, and `intl` for localization.
- For accessibility, use the `Semantics` widget, label images, avoid hardcoded sizes, and test with screen readers.

**Example: Semantics**

```dart
Semantics(
  label: 'Play button',
  child: Icon(Icons.play_arrow),
)
```

---

## Testing and Quality

- Write unit, widget, and integration tests.
- Use `flutter_test`, `mockito`, `bloc_test`, and `integration_test` packages.
- Set up CI/CD with GitHub Actions or Codemagic.
- Enforce static analysis with `flutter analyze` and custom lint rules.

**Example: Widget test**

```dart
testWidgets('Counter increments', (tester) async {
  await tester.pumpWidget(MyApp());
  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();
  expect(find.text('1'), findsOneWidget);
});
```

---

## Flutter Packages

- Commonly used: `flutter_svg`, `fl_chart`, `cached_network_image`, `riverpod`, `go_router`, `get_it`.
- Evaluate packages for maintenance, community support, and license.

---

## Version Control and Environments

- Use Git with branching strategies (e.g., GitFlow, trunk-based).
- Use semantic commit messages (`feat:`, `fix:`).
- Manage environments with `.env` files, `flutter_dotenv`, and build flavors.

---

## Dart Language Proficiency

- Use null safety, type inference, extension methods, `late`, `required`, and mixins.
- Handle errors with `try/catch` and custom exceptions.
- Optimize performance by avoiding unnecessary allocations and using unmodifiable collections.

---

## Problem Solving

- Practice with algorithm problems (e.g., Valid Parentheses, LRU Cache) to improve coding skills.

---

## Architecture and Framework

- Choose state management based on team experience and app complexity.
- Use dependency injection for testability and modularity.
- Profile and optimize performance with DevTools.
- Use implicit animations for simple cases, explicit for complex.

---

## Testing and Quality (Summary)

- Aim for a mix of unit, widget, and integration tests.
- Use mocks for API testing.
- Enforce code quality with static analysis and CI.

---

## Project Experience

- Modularize large projects, use GoRouter for navigation, and manage environment configs with `flutter_dotenv` or build flavors.

---

## CI/CD and Deployment

- Automate builds and tests with CI/CD tools.
- Use semantic versioning and automate store submissions.

---

## Leadership and Management

- Structure teams by feature or layer.
- Use documentation and code reviews to onboard and grow team members.

---

## Dev Practices

- Focus code reviews on readability, performance, and test coverage.
- Use formatters, custom lint rules, and pre-commit hooks.
- Track and prioritize technical debt.

---

## Problem Solving (Debugging)

- Use DevTools and `debugProfileBuildsEnabled` for profiling.
- Read source code and build proof-of-concepts when documentation is limited.

---

## Architecture Decisions

- Prefer scoped state management (e.g., Riverpod with hooks).
- Modularize routing and share domain/data logic across platforms.

---

## Technical Deep Dives

- Example: Shopping cart with Riverpod Notifier, persisted in SharedPreferences or SQLite.
- Use `MethodChannel` for native features.
- Build shared component libraries for consistent UI.

---

## Trends and Vision

- Stay updated with Flutter releases and community resources.
- Watch for upcoming features like Dart macros and WebAssembly support.

---

## Situational Handling

- Balance quality and speed by planning for refactoring.
- Refactor legacy code incrementally and add tests.

---

## Bonus Practice Links

- [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)
- [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
- [LRU Cache](https://leetcode.com/problems/lru-cache/)
- [Grade Calculator](https://www.rapidtables.com/calc/grade/test-calculator.html)

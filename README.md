# Flutter Cheat Sheet

A comprehensive guide for Flutter development, covering fundamentals, architecture, state management, UI/UX, API integration, performance, testing, and best practices.

## Flutter Fundamentals

### Stateless vs Stateful Widgets
___
- **StatelessWidget**: Immutable, UI does not depend on state changes. Rebuilds only when parent changes.  
  _Example_: Static content (splash screen, about page).
- **StatefulWidget**: Mutable via `State` class, UI depends on dynamic data. Lifecycle methods: `initState`, `didUpdateWidget`, `dispose`.  
  _Example_: Forms, toggles, dynamic content.

#### Widget Lifecycles

- **StatelessWidget**:
  - `build()`: Called when the widget is inserted into the tree or when its parent rebuilds.

- **StatefulWidget**:
  - `createState()`: Creates the mutable state object.
  - `initState()`: Called once when the state is created. Initialize data, subscribe to streams.
    - **Do:** Initialize controllers, listeners, or fetch initial data here.
    - **Don't:** Call `BuildContext`-dependent methods (like `showDialog`) directly in `initState`.
  - `didChangeDependencies()`: Called after `initState` and when dependencies change (e.g., `InheritedWidget` updates).
    - **Do:** Use for actions that depend on inherited widgets (like `Theme.of(context)`).
    - **Don't:** Perform heavy work here; it may be called multiple times.
  - `build()`: Called whenever the widget needs to be rendered.
    - **Do:** Keep build methods fast and side-effect free.
    - **Don't:** Start async operations or mutate state directly in `build()`.
  - `didUpdateWidget()`: Called when the parent widget changes and needs to update the state.
    - **Do:** Compare old and new widget properties to react to changes.
    - **Don't:** Forget to call `super.didUpdateWidget(oldWidget)`.
  - `setState()`: Triggers a rebuild by marking the widget as dirty.
    - **Do:** Only call when the widget is mounted and you need to update the UI.
    - **Don't:** Call after `dispose()` or in synchronous loops.
  - `deactivate()`: Called when the widget is removed from the tree temporarily.
    - **Do:** Pause animations or listeners if needed.
    - **Don't:** Dispose resources here; use `dispose()` instead.
  - `dispose()`: Called when the state object is permanently removed. Clean up controllers, listeners, etc.
    - **Do:** Cancel timers, close streams, and dispose controllers.
    - **Don't:** Call `setState()` here.

> **Tip:** Always pair resource allocation (controllers, streams) in `initState` with cleanup in `dispose()` to prevent memory leaks.

> **Tip:** Choose based on content dynamicity and interaction needs.  
> Always dispose resources in `dispose()` to prevent memory leaks.




### Keys

- **Purpose**: Retain widget identity across rebuilds, ensuring correct state and UI updates.

- **Types & Usage**:
  - `Key`: Base class, rarely used directly.
  - `ValueKey`: Use when you have a unique value (like an ID) for each widget, e.g., in lists with stable identifiers.
    ```dart
    ListView(
      children: items.map((item) => ListTile(
        key: ValueKey(item.id),
        title: Text(item.name),
      )).toList(),
    )
    ```
  - `ObjectKey`: Use when the identity of an object instance matters (e.g., model objects).
    ```dart
    ListView(
      children: models.map((model) => MyWidget(key: ObjectKey(model), model: model)).toList(),
    )
    ```
  - `UniqueKey`: Generates a unique key every time; use for widgets that must always be treated as new.
    ```dart
    ListView(
      children: List.generate(3, (i) => Container(key: UniqueKey())),
    )
    ```
  - `GlobalKey`: Allows access to widget state and context across the widget tree; use for forms, scaffold, or when you need to call methods on a widget from outside its build context.
    ```dart
    final formKey = GlobalKey<FormState>();
    Form(key: formKey, child: ...);
    formKey.currentState?.validate();
    ```

- **Navigator Key**:  
  - A special `GlobalKey<NavigatorState>` used to control navigation from outside the widget tree (e.g., in services or BLoC).
    ```dart
    final navigatorKey = GlobalKey<NavigatorState>();
    MaterialApp(navigatorKey: navigatorKey, ...);
    navigatorKey.currentState?.pushNamed('/home');
    ```

- **When Not to Use Keys**:
  - Avoid keys unless you need to preserve state, reorder items, or access widget state.
  - Overusing keys (especially `GlobalKey`) can hurt performance and break widget optimizations.
  - Do not use `UniqueKey` in lists unless you want every item to be treated as new on each rebuild (loses state).

- **Use Cases**:
  - Optimizing `ListView` performance (with `ValueKey` or `ObjectKey`)
  - Maintaining state during item reordering
    ```dart
    ReorderableListView(
      onReorder: ...,
      children: items.map((item) => ListTile(
        key: ValueKey(item.id),
        title: Text(item.name),
      )).toList(),
    )
    ```
  - Accessing widget state (`GlobalKey` for forms, scaffold, or navigation)



### Extension Methods

Add new functionality to existing classes without subclassing.

```dart
extension CapExtension on String {
  String capitalize() => this[0].toUpperCase() + substring(1);
}
```

- **Benefits**: Improves readability, keeps utilities near relevant types.



### `isMounted`

- Property in `State` objects to check if widget is in the tree.
- Prevents `setState()` after `dispose()`.
- **Best Practice**: Check `if (mounted)` before calling `setState` in async callbacks.



### Localization (`l10n`)

- **Tools**: `flutter_localizations`, `intl`, `.arb` files.
- **Command**: `flutter gen-l10n`
- **Pluralization**: `Intl.plural(...)`
- **Challenges**: Syncing `.arb` files, key naming, malformed JSON.



## UI/UX Design

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



**FractionallySizedBox**: Sizes child relative to parent.

```dart
FractionallySizedBox(
  widthFactor: 0.8,  // 80% of parent width
  heightFactor: 0.5, // 50% of parent height
  child: YourWidget(),
)
```



**Sizer**: Sizing widgets as a percentage of the screen.

```dart
Text('Hello', style: TextStyle(fontSize: 12.sp));
Container(height: 20.h, width: 50.w);
```

- **Limitations**: Not pixel-perfect, limited padding/margin scaling.



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

### Animations
___

Flutter supports both **implicit** and **explicit** animations for smooth UI transitions.

- **Implicit Animations**: Animate property changes automatically.
  - Widgets: `AnimatedContainer`, `AnimatedOpacity`, `AnimatedAlign`, etc.
  - **Do:** Use for simple transitions (size, color, opacity).
  - **Don't:** Use for complex, multi-property, or sequenced animations.

- **Explicit Animations**: Full control over animation timing and values.
  - Widgets: `AnimationController`, `Tween`, `AnimatedBuilder`, `Animation`, etc.
  - **Do:** Use for complex, coordinated, or custom animations.
  - **Don't:** Forget to dispose your `AnimationController` to avoid memory leaks.

- **Transitions**: Animate route changes or widget appearance.
  - Widgets: `PageRouteBuilder`, `Hero`, `FadeTransition`, `SlideTransition`.

- **Debug**: Use Flutter DevTools timeline, `TickerMode` to enable/disable tickers in widget subtrees.

**Example: Implicit Animation with AnimatedContainer**
```dart
class AnimatedBox extends StatefulWidget {
  @override
  State<AnimatedBox> createState() => _AnimatedBoxState();
}

class _AnimatedBoxState extends State<AnimatedBox> {
  double _size = 100;

  void _toggleSize() {
    setState(() {
      _size = _size == 100 ? 200 : 100;
    });
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _toggleSize,
      child: AnimatedContainer(
        width: _size,
        height: _size,
        color: Colors.blue,
        duration: Duration(milliseconds: 500),
        curve: Curves.easeInOut,
        child: Center(child: Text('Tap me')),
      ),
    );
  }
}
```

**Do:**
- Use implicit widgets for simple property changes.
- Always dispose `AnimationController` in `dispose()` when using explicit animations.
- Use `Hero` for shared element transitions between routes.

**Don't:**
- Animate too many widgets at once; it can hurt performance.
- Block the main thread with heavy computations during animations.

> **Tip:** For advanced choreography, use `AnimationController` with `AnimatedBuilder` or `AnimatedWidget` for maximum flexibility.

## API Integration

### Authorization
___

- **Access Token**: Short-lived, sent with every request, stored in memory/secure storage.
- **Refresh Token**: Long-lived, used to get new access tokens, stored securely.

| Token Type    | Analogy                      |
||--|
| Access Token  | Room key card (limited time)|
| Refresh Token | Passport + reissue form     |

- **Separation**: Improves security, allows session rotation, scalable backend.



**Login Endpoint**

- Accepts credentials, issues tokens, limits attempts, hashes passwords, logs IP/user agent.

**Refresh Endpoint**

- Accepts only valid refresh tokens, rotates on use, blocks reuse, can bind to device/IP, expires after set period.

**Token Structure**

| Property         | Access Token        | Refresh Token                |
||--||
| Format           | JWT or opaque      | Opaque (preferred)           |
| Storage          | Memory/secure      | Secure only                  |
| Contains         | Claims             | UUID only                    |
| Signed           | Yes                | Preferably yes               |



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

## Flutter Architecture
### Scalability Patterns

- **Feature-First vs. Layer-First**
  - *Feature-First*: Organize code by features (e.g., `/features/auth/`, `/features/profile/`). Improves modularity, makes onboarding easier, and supports parallel development.
    - **Do:** Use feature-first for large or growing apps.
    - **Do:** Keep feature folders self-contained (UI, logic, models).
    - **Don't:** Mix unrelated features in the same folder.
  - *Layer-First*: Organize by technical layers (e.g., `/models/`, `/views/`, `/services/`). Can lead to cross-feature dependencies and harder navigation in large codebases.
    - **Do:** Use for small apps or when layers are minimal.
    - **Don't:** Let layers become dumping grounds for unrelated code.
  - **Recommendation:** Prefer feature-first for large apps; combine with layer separation inside each feature.

- **Modularization**
  - Split code into independent modules/packages (e.g., `core`, `shared`, `feature_x`).
    - **Do:** Extract shared UI components, utilities, and services into their own modules.
    - **Do:** Use Dart/Flutter packages for isolation and reusability.
    - **Don't:** Over-modularize prematurely; start with clear boundaries.
    - **Don't:** Create circular dependencies between modules.

- **Package Splitting**
  - Move reusable features or libraries into separate packages (local or published).
    - **Do:** Use `path` or `git` dependencies in `pubspec.yaml` for local development.
    - **Do:** Write independent tests for each package.
    - **Don't:** Duplicate code across packages.
    - **Don't:** Expose internal implementation details in package APIs.

- **Monorepo Strategies**
  - Store all app packages and modules in a single repository.
    - **Do:** Use tools like [Melos](https://melos.invertase.dev/) for managing packages, scripts, and versioning.
    - **Do:** Set up shared CI/CD pipelines for all packages.
    - **Don't:** Ignore dependency version mismatches between packages.
    - **Don't:** Commit breaking changes without coordinating across affected packages.
  - Benefits: Easier refactoring, atomic commits across packages, shared CI/CD.
  - Challenges: Requires tooling for dependency management and build orchestration.

> **Tip:** Start with a scalable structure early—even small apps can grow quickly. Use clear naming conventions and documentation for each module or package.
___

### InheritedWidget
  Use for low-level dependency injection and propagating data down the widget tree. Most higher-level solutions (like Provider, Riverpod) are built on top of it.

  **Example: Custom InheritedWidget**
  ```dart
  class CounterData extends InheritedWidget {
    final int counter;
    final Widget child;

    CounterData({required this.counter, required this.child}) : super(child: child);

    static CounterData? of(BuildContext context) =>
        context.dependOnInheritedWidgetOfExactType<CounterData>();

    @override
    bool updateShouldNotify(CounterData oldWidget) => counter != oldWidget.counter;
  }

  // Usage in widget tree:
  CounterData(
    counter: 5,
    child: MyHomePage(),
  );
  ```

  **Do:**
  - Use for propagating immutable data or callbacks.
  - Use `of(context)` pattern for access.

  **Don't:**
  - Use for complex or deeply nested state; prefer higher-level solutions.

- **Common Patterns:**
  - **MVVM**: Use with Provider or Riverpod for separating UI (View), logic (ViewModel), and data.
  - **Clean Architecture**: Separate domain, data, and presentation layers for testability and scalability.

  **Example: Provider (MVVM)**
  ```dart
  class CounterViewModel extends ChangeNotifier {
    int _count = 0;
    int get count => _count;

    void increment() {
      _count++;
      notifyListeners();
    }
  }

  // In main.dart
  ChangeNotifierProvider(
    create: (_) => CounterViewModel(),
    child: MyApp(),
  );

  // In widget
  Consumer<CounterViewModel>(
    builder: (context, vm, child) => Text('${vm.count}'),
  );
  ```

  **Do:**
  - Use Provider/Riverpod for scalable, testable state management.
  - Keep business logic out of widgets.

  **Don't:**
  - Mutate state directly in widgets.
  - Overuse global state; prefer scoped providers.

- **Modularization:**
  - Organize code by features or layers.
  - Extract shared UI components, utilities, and services into separate packages or modules.

  **Do:**
  - Use feature-first structure for large apps.
  - Keep shared code in `core` or `shared` packages.

  **Don't:**
  - Mix unrelated features in the same folder.
  - Duplicate code across modules.
___
### Navigation

- **Navigator 1.0 (Imperative):**
  - Use `Navigator.push`, `Navigator.pop` for simple navigation.
  - Good for small apps or simple flows.

  **Example:**
  ```dart
  Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => DetailsPage()),
  );
  Navigator.pop(context);
  ```

  **Do:**
  - Use for straightforward navigation stacks.

  **Don't:**
  - Use for deep linking or complex flows.

- **Navigator 2.0 (Declarative):**
  - Supports deep linking, browser history, and complex navigation.
  - Use packages like `go_router` or `auto_route` for easier setup.

  **Example: go_router Setup**
  ```dart
  final router = GoRouter(
    routes: [
      GoRoute(
        path: '/home',
        builder: (context, state) => HomePage(),
      ),
      GoRoute(
        path: '/details/:id',
        builder: (context, state) {
          final id = state.params['id'];
          return DetailsPage(id: id);
        },
      ),
    ],
    errorBuilder: (context, state) => NotFoundPage(),
    redirect: (context, state) {
      // Example: Redirect unauthenticated users
      final loggedIn = checkAuth();
      if (!loggedIn && state.location != '/login') return '/login';
      return null;
    },
  );

  // In MaterialApp.router
  MaterialApp.router(
    routerConfig: router,
  );
  ```

  **Do:**
  - Use for apps requiring deep linking, web support, or complex navigation.
  - Handle unknown routes and errors gracefully.
  - Use route guards/redirects for authentication flows.

  **Don't:**
  - Mix imperative and declarative navigation in the same flow.
  - Forget to test navigation on web and mobile.

> **Tip:** For most apps, start with Navigator 1.0 for simplicity. Migrate to Navigator 2.0 or a package like `go_router` as navigation needs grow.



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



### Futures

- **What:** Represents a computation that completes later with a value or error.
- **Usage:** Use `async`/`await` for readable, sequential async code.
- **Do:**
  - Always handle errors with `try/catch` or `.catchError`.
  - Use `await` only inside `async` functions.
  - Chain async calls with `await` for clarity.
- **Don't:**
  - Block the UI thread with synchronous heavy work.
  - Forget to handle exceptions—unhandled errors can crash the app.

**Example:**
```dart
Future<void> loadData() async {
  try {
    final data = await fetchData();
    // Use data
  } catch (e) {
    // Handle error
  }
}
```



### Streams

- **What:** Delivers a sequence of async events (data, error, done).
- **Usage:** Use for sockets, user input, or continuous data.
- **Do:**
  - Use `StreamBuilder` to bind streams to widgets.
  - Cancel subscriptions in `dispose()` to prevent memory leaks.
  - Use `async*` and `yield` for custom stream generation.
- **Don't:**
  - Forget to close streams or cancel subscriptions.
  - Use streams for one-off events—prefer `Future` instead.

**Example: StreamBuilder**
```dart
StreamBuilder<int>(
  stream: counterStream(),
  builder: (context, snapshot) {
    if (!snapshot.hasData) return CircularProgressIndicator();
    return Text('Count: ${snapshot.data}');
  },
)
```



### Isolates

- **What:** Separate memory/thread for heavy computation.
- **Usage:** Use `compute()` for simple background tasks, or `Isolate` for advanced use.
- **Do:**
  - Use for CPU-intensive work (parsing, encoding, image processing).
  - Pass only simple, serializable data to isolates.
- **Don't:**
  - Use isolates for I/O-bound tasks (network, file)—Dart's async model handles these efficiently.
  - Share complex objects or open connections between isolates.



### Dos and Don'ts

**Do:**
- Use `mounted` check before calling `setState` in async callbacks.
- Dispose controllers, subscriptions, and streams in `dispose()`.
- Use `Future.microtask` or `WidgetsBinding.instance.addPostFrameCallback` for post-build async work.

**Don't:**
- Call `setState` after a widget is disposed.
- Start async work in `build()`—use `initState` or callbacks.
- Ignore unawaited futures; use `unawaited()` from `package:pedantic` if intentional.



> **Tip:** Prefer `FutureBuilder`/`StreamBuilder` for simple async UI. For complex flows, consider state management solutions (Provider, Riverpod, Bloc) to handle async logic and state updates cleanly.

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

## Dependency Injection

- Use GetIt for service locator pattern, Provider or Riverpod for scoped dependency injection.
- Prefer passing dependencies via constructors for testability.
- Use `MultiProvider` or `ProviderScope` for grouping dependencies.

**Example: GetIt registration**
```dart
final getIt = GetIt.instance;
getIt.registerSingleton<ApiService>(ApiServiceImpl());
```

## Design System Integration

- **Do:** Centralize theme, use design tokens, and enforce consistency.
- **Don't:** Override theme locally unless necessary.
```dart
ThemeData(
  primaryColor: AppColors.primary,
  textTheme: AppTextStyles.all,
)
```

## Error Reporting Framework

A robust error reporting framework helps you detect, log, and respond to errors in production and development. Here’s a unified approach for Flutter:

### Error Capture, Logging, and Reporting

Set up global error handlers to catch uncaught exceptions in both Flutter and Dart zones, and centralize reporting via a singleton service. Integrate with third-party services (Sentry, Crashlytics), local logging, and user feedback.

**Naming:**  
`ErrorReporter` is a clear, descriptive name for a centralized error reporting service. Alternatives: `AppErrorReporter`, `ErrorHandler`, or `ErrorService`. Choose based on your project’s naming conventions.

**Unified Example:**

```dart
import 'dart:developer' as developer;
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';
import 'package:share_plus/share_plus.dart';
// Uncomment if using Sentry or Crashlytics
// import 'package:sentry_flutter/sentry_flutter.dart';
// import 'package:firebase_crashlytics/firebase_crashlytics.dart';

class ErrorReporter {
  ErrorReporter._();
  static final instance = ErrorReporter._();

  // Integrate with Sentry
  Future<void> reportToSentry(Object error, StackTrace stack) async {
    // await Sentry.captureException(error, stackTrace: stack);
  }

  // Integrate with Crashlytics
  Future<void> reportToCrashlytics(Object error, StackTrace stack) async {
    // await FirebaseCrashlytics.instance.recordError(error, stack);
  }

  // Log to console/DevTools
  void logError(Object error, StackTrace stack, {String message = ''}) {
    developer.log(
      message.isEmpty ? 'Unhandled error' : message,
      error: error,
      stackTrace: stack,
      level: 1000,
      name: 'app.error',
    );
  }

  // Log to local file
  Future<void> logErrorToFile(Object error, StackTrace stack, {String message = ''}) async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/error_logs.txt');
    final logEntry = '${DateTime.now()}: $message\n$error\n$stack\n\n';
    await file.writeAsString(logEntry, mode: FileMode.append);
  }

  // Unified error reporting
  Future<void> report(Object error, StackTrace stack, {String message = ''}) async {
    logError(error, stack, message: message);
    await logErrorToFile(error, stack, message: message);
    // await reportToSentry(error, stack);
    // await reportToCrashlytics(error, stack);
  }

  // Share logs with developers
  Future<void> shareLogs() async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/error_logs.txt');
    if (await file.exists()) {
      await Share.shareXFiles([XFile(file.path)], text: 'App error logs');
    }
  }
}

void main() {
  FlutterError.onError = (FlutterErrorDetails details) {
    FlutterError.presentError(details);
    ErrorReporter.instance.report(details.exception, details.stack ?? StackTrace.current, message: 'FlutterError');
  };

  runZonedGuarded(() {
    runApp(MyApp());
  }, (error, stack) {
    ErrorReporter.instance.report(error, stack, message: 'ZoneError');
  });
}

// Friendly error screen
ErrorWidget.builder = (FlutterErrorDetails details) {
  return Center(child: Text('Something went wrong.'));
};
```

**Best Practices:**
- Use `ErrorReporter` for all error capture, logging, and reporting.
- Integrate with third-party services as needed.
- Store logs locally and provide a UI for users to share logs.
- Show user-friendly error screens instead of crashes.


## Performance Profiling

Flutter provides a rich set of profiling tools to help you analyze, debug, and optimize your app’s performance. The primary tool is **Flutter DevTools**, which offers multiple tabs for different profiling aspects.

### 1. Flutter DevTools Overview

**How to launch:**  
- Run your app in debug or profile mode.
- Execute `flutter pub global activate devtools` (if not already).
- Start DevTools:  
  - From terminal: `flutter pub global run devtools`
  - Or via IDE (VS Code/Android Studio): Click the “Open DevTools” button.

**Connect:**  
- Use the link printed in the console, or connect via your IDE.



### 2. DevTools Tabs and Profiling Techniques

#### **a. Performance Tab**

- **Timeline View:**  
  - Visualizes frame rendering, UI thread, GPU thread, and async events.
  - Shows frame rendering times (target: <16ms for 60fps).
  - **Use:** Identify jank, slow frames, and expensive build/layout/paint phases.
- **Frame Analysis:**  
  - Inspect each frame for build, layout, paint, and raster times.
  - **Highlight:** Frames exceeding the budget are marked in red.
- **CPU Profiler:**  
  - Record and analyze CPU samples for Dart code.
  - **Use:** Find slow functions, heavy computations, and bottlenecks.
- **User Flows:**  
  - Record custom user journeys for targeted profiling.

**How to use:**  
- Click “Record” to capture timeline events.
- Interact with your app to reproduce issues.
- Stop recording and analyze the timeline.



#### **b. Memory Tab**

- **Heap Snapshots:**  
  - Take snapshots of memory usage at any point.
  - **Use:** Compare before/after states to detect leaks.
- **Memory Allocation:**  
  - Track live objects, allocations, and garbage collection.
- **Dart Objects:**  
  - Inspect instances by type, retainers, and references.
- **Leaks:**  
  - Identify objects that are not being disposed.

**How to use:**  
- Take snapshots before and after navigation or heavy operations.
- Look for unexpected growth in object counts.



#### **c. CPU Profiler Tab**

- **Call Tree & Bottom Up:**  
  - Visualize CPU time spent in functions.
  - **Use:** Identify hot spots and optimize slow code.
- **Profile Recording:**  
  - Start/stop CPU profiling for specific actions.



#### **d. Network Tab**

- **HTTP Traffic:**  
  - View all HTTP requests/responses, headers, payloads, and timings.
- **WebSocket:**  
  - Inspect WebSocket messages.
- **Use:**  
  - Debug slow or failed network calls.
  - Analyze payload sizes and response times.



#### **e. Inspector Tab**

- **Widget Tree:**  
  - Visualize the widget hierarchy in real time.
- **Layout Explorer:**  
  - Inspect constraints, sizes, paddings, and flex layouts.
- **Highlight Repaints:**  
  - Enable “Repaint Rainbow” to see which widgets repaint.
- **Use:**  
  - Debug layout issues, excessive rebuilds, and widget structure.



#### **f. Logging Tab**

- **Console Output:**  
  - View logs, errors, and debug prints.
- **Filtering:**  
  - Filter logs by level or content.



#### **g. App Size Tab**

- **Size Analysis:**  
  - Analyze the size of your app’s release build.
- **Treemap:**  
  - Visualize which packages, assets, or code contribute most to the binary size.
- **Use:**  
  - Identify and reduce bloat.


#### **h. Provider/Bloc/Riverpod Tabs (if using)**

- **State Management:**  
  - Inspect provider/bloc/riverpod state, dependencies, and updates.


### 3. Additional Profiling Techniques

- **Profile Mode:**  
  - Run `flutter run --profile` for near-release performance.
- **Release Mode:**  
  - For final performance, use `flutter run --release`.
- **Widget Rebuild Profiling:**  
  - Use `debugPrintRebuildDirtyWidgets = true;` to log widget rebuilds.
- **Repaint Rainbow:**  
  - Enable in DevTools Inspector to visualize repaints.
- **Performance Overlay:**  
  - Add `showPerformanceOverlay: true` in `MaterialApp` for in-app frame stats.
- **Tracing:**  
  - Use `Timeline` API for custom event tracing.



### 4. Best Practices

- **Do:**  
  - Profile on real devices, not just emulators.
  - Use DevTools regularly during development.
  - Investigate slow frames, memory leaks, and large app sizes.
- **Don't:**  
  - Rely solely on debug mode for performance metrics.
  - Ignore warnings about jank or memory growth.

> **Tip:** Always profile in both profile and release modes, as debug mode does not reflect real-world performance.



## Security

- **Do:** Store secrets in secure storage, use HTTPS, and obfuscate builds.
- **Don't:** Log sensitive data or hardcode secrets.
```dart
final storage = FlutterSecureStorage();
await storage.write(key: 'token', value: 'abc');
```

## Code Generation

Automate repetitive code using tools like `build_runner`, `Freezed`, and `JsonSerializable`.

- **Do:** Use for models, DI, and unions.
- **Don't:** Edit generated files manually.
```dart
@freezed
class User with _$User {
  const factory User({required String id, required String name}) = _User;
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
```

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

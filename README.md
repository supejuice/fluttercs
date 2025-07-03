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


---

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

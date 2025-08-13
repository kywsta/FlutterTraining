# Navigation

Flutter provides multiple approaches to handle navigation, from the traditional Navigator class to modern declarative routing solutions.

## Github

https://github.com/kywsta/flutter_navigation_sample

## Overview

- **MaterialApp**: The main widget for your app that manages the overall navigation and routing.
- **Navigator**: The traditional imperative navigation approach
- **Named Routes**: A more organized way to manage routes using string identifiers
- **Router**: The modern declarative routing system
- **GoRouter**: A popular third-party package that simplifies declarative routing

## Navigator

### Basic Navigation Between Screens

```dart
// Screen A navigate to Screen B
Navigator.push(context, MaterialPageRoute(builder: (context) => ScreenB()));

// Screen B navigate back to Screen A
Navigator.pop(context);

or

// Screen B navigate back to Screen A with some result
Navigator.pop(context, result);
```

### Named Routes (Not Recommended)

```dart
// Define routes in MaterialApp
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => const ScreenA(),
    '/b': (context) => const ScreenB(),
  },
)

// Navigate to Screen B
Navigator.pushNamed(context, '/b');

// Navigate back to Screen A
Navigator.pop(context);
```

### Navigator Functions

```dart
// Push a new route onto the stack
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => ScreenC()),
);

// Pop the current route
Navigator.pop(context);

// Pop the current route with a result
Navigator.pop(context, result);

// Push a new route and remove all previous routes from the navigation stack
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => ScreenC()),
  (route) => false,
);

// Push a new route and remove all previous routes from the navigation stack until a specific route is reached
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => ScreenB()),
  ModalRoute.withName('/'),
);

// Push a new route and replace the current route
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => ScreenC()),
);
```

## Go Router

GoRouter is a powerful third-party package that simplifies declarative routing in Flutter applications. It provides excellent support for deep linking, nested routing, and type-safe navigation.

### Installation

Add GoRouter to your `pubspec.yaml`:

```yaml
dependencies:
  go_router: latest
```

### Basic GoRouter Setup

```dart
import 'package:go_router/go_router.dart';
import 'package:flutter/material.dart';

// Define the router
final GoRouter _router = GoRouter(
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) {
        return const ScreenA();
      },
      routes: <RouteBase>[
        GoRoute(
          path: '/b',
          builder: (BuildContext context, GoRouterState state) {
            return const ScreenB();
          },
        ),
      ],
    ),
  ],
);

// Configure the app to use the router
class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: appRouter,
    );
  }
}
```

### Advanced GoRouter Features

#### Route Parameters

```dart
GoRoute(
  path: '/user/:userId',
  builder: (context, state) {
    final userId = state.pathParameters['userId']!;
    return UserScreen(userId: userId);
  },
),
```

#### Query Parameters

```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'] ?? '';
    return SearchScreen(query: query);
  },
),
```

#### Nested Routes

```dart
GoRoute(
  path: '/family/:familyId',
  builder: (context, state) {
    return FamilyScreen(familyId: state.pathParameters['familyId']!);
  },
  routes: [
    GoRoute(
      path: 'person/:personId',
      builder: (context, state) {
        return PersonScreen(
          familyId: state.pathParameters['familyId']!,
          personId: state.pathParameters['personId']!,
        );
      },
    ),
  ],
),
```

## Go, Push, Pop

### Navigation Methods with GoRouter

#### Go (Replace Current Route)

The `go` method replaces the current route with a new one:

```dart
// Navigate to a new route, replacing the current one
context.go('/profile');

// Navigate with parameters
context.go('/user/123');

// Navigate with query parameters
context.go('/search?q=flutter');
```

#### Push (Add Route to Stack)

The `push` method adds a new route to the navigation stack:

```dart
// Push a new route onto the stack
context.push('/settings');

// Push with parameters
context.push('/user/456');

// Push and wait for result
final result = await context.push('/edit-profile');
if (result != null) {
  // Handle the returned result
  print('Profile updated: $result');
}
```

#### Pop (Remove Current Route)

The `pop` method removes the current route from the stack:

```dart
// Simple pop
context.pop();

// Pop with result
context.pop('Profile saved successfully');

// Pop until a specific route
context.go('/');
```

#### Passing Data Forward

```dart
// Using extra parameter
context.push('/details', extra: {'userId': 123, 'userName': 'John'});

// In the destination route
GoRoute(
  path: '/details',
  builder: (context, state) {
    final data = state.extra as Map<String, dynamic>?;
    return DetailsScreen(
      userId: data?['userId'],
      userName: data?['userName'],
    );
  },
),
```

### Navigation Guards

```dart
final GoRouter _router = GoRouter(
  redirect: (context, state) {
    final isLoggedIn = AuthService.instance.isLoggedIn;
    final isGoingToLogin = state.matchedLocation == '/login';

    // Redirect to login if not authenticated
    if (!isLoggedIn && !isGoingToLogin) {
      return '/login';
    }

    // Redirect to home if already logged in and going to login
    if (isLoggedIn && isGoingToLogin) {
      return '/';
    }

    return null; // No redirect needed
  },
  routes: [
    // Your routes here
  ],
);
```

## Deeplink

Deep linking allows users to navigate directly to specific screens in your app through URLs. This is essential for web applications and provides a better user experience on mobile platforms.

### Configuring Deep Links

### Set up hosting

Follow the steps in the [official documentation](https://firebase.google.com/docs/hosting/quickstart?_gl=1*88u658*_up*MQ..*_ga*MTM1MzcyODgwMS4xNzU1MDcyODk2*_ga_CW55HF8NVT*czE3NTUwNzI4OTUkbzEkZzAkdDE3NTUwNzI4OTUkajYwJGwwJGgw) to setup firebase hosting to host the `assetlinks.json` and `apple-app-site-association` files inside the `public/.well-known` folder.

#### Android Configuration

Get the SHA256 fingerprint of the your app signing key

```bash
# If you are using a custom keystore
keytool -list -v -keystore "your-path-to/keystore.jks" -alias debug -storepass your-store-passs -keypass your-key-pass

# If you are using the default debug keystore
keytool -list -v -alias androiddebugkey -keystore ~/.android/debug.keystore -storepass android
```

Follow the steps in the [official documentation](https://docs.flutter.dev/cookbook/navigation/set-up-app-links) to configure deep links for Android.


#### iOS Configuration

Follow the steps in the [official documentation](https://docs.flutter.dev/cookbook/navigation/set-up-universal-links) to configure deep links for iOS.

**Note**: It might take 24 hours for the changes to take effect.

### Testing Deep Links

#### Testing on Android

```bash
# Test with ADB
adb shell 'am start -a android.intent.action.VIEW \
    -c android.intent.category.BROWSABLE \
    -d "https://flutter-deeplink-example-85985.web.app/b"' \
    com.example.flutterNavigationSample

adb shell 'am start -a android.intent.action.VIEW \
    -c android.intent.category.BROWSABLE \
    -d "https://flutter-deeplink-example-85985.web.app/c/12"' \
    com.example.flutter_navigation_sample
```

#### Testing on iOS Simulator

```bash
# Test with Simulator
xcrun simctl openurl booted "https://yourapp.com/product/123"

# Test custom scheme
xcrun simctl openurl booted "yourapp://product/123"
```

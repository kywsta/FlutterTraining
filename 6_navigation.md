# Navigation

Navigation is a fundamental aspect of mobile application development that allows users to move between different screens and sections of your app. Flutter provides multiple approaches to handle navigation, from the traditional Navigator class to modern declarative routing solutions.

## Overview

Flutter offers several navigation paradigms:
- **Navigator**: The traditional imperative navigation approach
- **Named Routes**: A more organized way to manage routes using string identifiers
- **Router**: The modern declarative routing system
- **GoRouter**: A popular third-party package that simplifies declarative routing

This section will guide you through each approach, starting with the basics and progressing to more advanced concepts.

## Navigator

### Basic Navigator

```dart

```

## Router

### Understanding Flutter's Router System

Flutter's Router is a declarative navigation system that provides more control over the navigation stack and URL handling. Unlike the imperative Navigator approach, the Router system treats navigation as a state management problem.

#### Key Concepts

**RouteInformation**: Contains information about the current route, including the URL path and state.

**RouteInformationParser**: Converts RouteInformation to a configuration object that your app understands.

**RouterDelegate**: Builds the Navigator widget and handles navigation logic based on the current configuration.

#### Basic Router Implementation

```dart
import 'package:flutter/material.dart';

class AppRouterDelegate extends RouterDelegate<String>
    with ChangeNotifier, PopNavigatorRouterDelegateMixin<String> {
  
  @override
  final GlobalKey<NavigatorState> navigatorKey;
  
  String _currentPath = '/';
  
  AppRouterDelegate() : navigatorKey = GlobalKey<NavigatorState>();
  
  @override
  String get currentConfiguration => _currentPath;
  
  @override
  Widget build(BuildContext context) {
    return Navigator(
      key: navigatorKey,
      pages: [
        MaterialPage(
          key: ValueKey('/'),
          child: HomeScreen(),
        ),
        if (_currentPath == '/profile')
          MaterialPage(
            key: ValueKey('/profile'),
            child: ProfileScreen(),
          ),
      ],
      onPopPage: (route, result) {
        if (!route.didPop(result)) return false;
        _currentPath = '/';
        notifyListeners();
        return true;
      },
    );
  }
  
  @override
  Future<void> setNewRoutePath(String path) async {
    _currentPath = path;
    notifyListeners();
  }
  
  void navigateToProfile() {
    _currentPath = '/profile';
    notifyListeners();
  }
}

class AppRouteInformationParser extends RouteInformationParser<String> {
  @override
  Future<String> parseRouteInformation(RouteInformation routeInformation) async {
    return routeInformation.location ?? '/';
  }
  
  @override
  RouteInformation restoreRouteInformation(String path) {
    return RouteInformation(location: path);
  }
}
```

#### Using Router in MaterialApp

```dart
class MyApp extends StatelessWidget {
  final AppRouterDelegate _routerDelegate = AppRouterDelegate();
  final AppRouteInformationParser _routeInformationParser = 
      AppRouteInformationParser();
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Flutter Router Demo',
      routerDelegate: _routerDelegate,
      routeInformationParser: _routeInformationParser,
    );
  }
}
```

## Go Router

GoRouter is a powerful third-party package that simplifies declarative routing in Flutter applications. It provides excellent support for deep linking, nested routing, and type-safe navigation.

### Installation

Add GoRouter to your `pubspec.yaml`:

```yaml
dependencies:
  go_router: ^12.1.3
```

### Basic GoRouter Setup

```dart
import 'package:go_router/go_router.dart';
import 'package:flutter/material.dart';

final GoRouter _router = GoRouter(
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) {
        return const HomeScreen();
      },
      routes: <RouteBase>[
        GoRoute(
          path: '/details',
          builder: (BuildContext context, GoRouterState state) {
            return const DetailsScreen();
          },
        ),
      ],
    ),
  ],
);

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
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
context.go('/'); // This will clear the stack and go to home
```

### Navigation with Data

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

#### Receiving Data Back

```dart
// Push and wait for result
final result = await context.push('/edit-item', extra: item);

if (result != null) {
  final updatedItem = result as Item;
  // Update your state with the returned data
  setState(() {
    items[index] = updatedItem;
  });
}

// In the edit screen, return data when popping
ElevatedButton(
  onPressed: () {
    context.pop(updatedItem); // Return the updated item
  },
  child: Text('Save'),
)
```

### Conditional Navigation

```dart
void navigateBasedOnUserRole(String role) {
  switch (role) {
    case 'admin':
      context.go('/admin-dashboard');
      break;
    case 'user':
      context.go('/user-dashboard');
      break;
    default:
      context.go('/login');
  }
}
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

#### Android Configuration

Add the following to your `android/app/src/main/AndroidManifest.xml`:

```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:launchMode="singleTop"
    android:theme="@style/LaunchTheme">
    
    <!-- Standard App Launch -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    
    <!-- Deep Link Intent Filter -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https"
              android:host="yourapp.com" />
    </intent-filter>
    
    <!-- Custom Scheme Deep Link -->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="yourapp" />
    </intent-filter>
</activity>
```

#### iOS Configuration

Add the following to your `ios/Runner/Info.plist`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>yourapp.com</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>https</string>
        </array>
    </dict>
    <dict>
        <key>CFBundleURLName</key>
        <string>yourapp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>yourapp</string>
        </array>
    </dict>
</array>
```

### Handling Deep Links with GoRouter

GoRouter automatically handles deep links when properly configured:

```dart
final GoRouter _router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/product/:productId',
      builder: (context, state) {
        final productId = state.pathParameters['productId']!;
        return ProductScreen(productId: productId);
      },
    ),
    GoRoute(
      path: '/user/:userId/profile',
      builder: (context, state) {
        final userId = state.pathParameters['userId']!;
        final tab = state.uri.queryParameters['tab'];
        return UserProfileScreen(
          userId: userId,
          initialTab: tab,
        );
      },
    ),
  ],
);
```

### Testing Deep Links

#### Testing on Android

```bash
# Test with ADB
adb shell am start \
  -W -a android.intent.action.VIEW \
  -d "https://yourapp.com/product/123" \
  com.example.yourapp

# Test custom scheme
adb shell am start \
  -W -a android.intent.action.VIEW \
  -d "yourapp://product/123" \
  com.example.yourapp
```

#### Testing on iOS Simulator

```bash
# Test with Simulator
xcrun simctl openurl booted "https://yourapp.com/product/123"

# Test custom scheme
xcrun simctl openurl booted "yourapp://product/123"
```

### Dynamic Deep Link Handling

```dart
class DeepLinkHandler {
  static Future<void> handleIncomingLink(String link) async {
    final uri = Uri.parse(link);
    
    // Extract path and parameters
    final path = uri.path;
    final queryParams = uri.queryParameters;
    
    // Handle different deep link patterns
    if (path.startsWith('/product/')) {
      final productId = path.split('/')[2];
      GoRouter.of(context).go('/product/$productId');
    } else if (path.startsWith('/user/')) {
      final userId = path.split('/')[2];
      final tab = queryParams['tab'];
      GoRouter.of(context).go('/user/$userId/profile?tab=$tab');
    } else {
      // Default fallback
      GoRouter.of(context).go('/');
    }
  }
}
```

### Best Practices for Deep Linking

1. **Validate Parameters**: Always validate deep link parameters to prevent crashes
2. **Graceful Fallbacks**: Provide fallback navigation for invalid deep links
3. **User Authentication**: Handle authentication state when processing deep links
4. **Analytics**: Track deep link usage for better insights

```dart
GoRoute(
  path: '/shared/:contentId',
  builder: (context, state) {
    final contentId = state.pathParameters['contentId'];
    
    // Validate content ID
    if (contentId == null || contentId.isEmpty) {
      return const NotFoundScreen();
    }
    
    // Check authentication if required
    if (!AuthService.instance.isLoggedIn) {
      // Store intended destination and redirect to login
      AuthService.instance.setIntendedDestination(state.uri.toString());
      return const LoginScreen();
    }
    
    return SharedContentScreen(contentId: contentId);
  },
),
```

## Summary

Navigation in Flutter has evolved from simple imperative approaches to sophisticated declarative systems. The Router system provides powerful control over navigation state, while GoRouter offers a developer-friendly API with excellent deep linking support. Understanding these navigation patterns is crucial for building professional Flutter applications that provide seamless user experiences across different platforms and entry points.

Key takeaways:
- Use Router for complex navigation requirements with custom logic
- GoRouter simplifies declarative routing with excellent deep linking support
- Proper parameter handling and validation are essential for robust navigation
- Deep linking configuration requires platform-specific setup but provides significant user experience benefits
# 12. Error Handling

This section covers three fundamental approaches to error handling in Flutter: FlutterError class, the Dartz package for functional error handling, and global error handling strategies.

## 12.1 Flutter Error

The `FlutterError` class is Flutter's built-in mechanism for handling framework-level errors. It provides a structured way to capture, report, and handle errors that occur within the Flutter framework.

### Basic Usage of FlutterError

FlutterError is primarily used for reporting errors that occur during widget building, rendering, or other framework operations. Here is how to use it effectively:

```dart
import 'package:flutter/foundation.dart';

void main() {
  // Set up custom error handling
  FlutterError.onError = (FlutterErrorDetails details) {
    // Log error details
    print('Flutter Error: ${details.exception}');
    print('Stack Trace: ${details.stack}');
    
    // In debug mode, show the error
    if (kDebugMode) {
      FlutterError.presentError(details);
    }
  };
  
  runApp(MyApp());
}
```

### Creating Custom FlutterError

You can create custom FlutterError instances to report application-specific errors:

```dart
void reportCustomError() {
  try {
    // Some operation that might fail
    performRiskyOperation();
  } catch (error, stackTrace) {
    // Report the error using FlutterError
    FlutterError.reportError(FlutterErrorDetails(
      exception: error,
      stack: stackTrace,
      library: 'my_app',
      context: ErrorDescription('Error occurred in custom operation'),
    ));
  }
}

void performRiskyOperation() {
  throw Exception('Something went wrong');
}
```

## 12.2 Dartz Package

Dartz is a functional programming package for Dart that provides powerful error handling capabilities through the `Either` type, enabling elegant error handling without exceptions.

### Installation

Add Dartz to your `pubspec.yaml`:

```yaml
dependencies:
  dartz: ^0.10.1
```

### Basic Either Usage

The `Either` type represents a value that can be either a `Left` (error) or `Right` (success):

```dart
import 'package:dartz/dartz.dart';

// Define a function that returns Either
void _performDivision(int a, int b) {
  final result = _divide(a, b);
  result.fold(
    (error) => ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        backgroundColor: Colors.red,
        content: Text('Failed to divide: $error'),
      ),
    ),
    (value) => ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        backgroundColor: Colors.green,
        content: Text('Success: $value'),
      ),
    ),
  );
}

Either<String, int> _divide(int a, int b) {
  if (b == 0) {
    return const Left('Division by zero error');
  }
  return Right(a ~/ b);
}
```

### Chaining Operations with Either

Chain multiple operations that might fail using `flatMap` or `bind`:

```dart
Either<String, String> validateEmail(String email) {
  if (email.isEmpty) {
    return const Left('Email cannot be empty');
  }
  if (!email.contains('@')) {
    return const Left('Invalid email format');
  }
  return Right(email);
}

Either<String, String> validatePassword(String password) {
  if (password.length < 6) {
    return const Left('Password must be at least 6 characters');
  }
  return Right(password);
}

Either<String, Map<String, String>> validateUser(String email, String password) {
  return validateEmail(email).flatMap((validEmail) =>
    validatePassword(password).map((validPassword) => {
      'email': validEmail,
      'password': validPassword,
    })
  );
}

void main() {
  final result = validateUser('user@example.com', 'password123');
  
  result.fold(
    (error) => print('Validation error: $error'),
    (userData) => print('Valid user data: $userData'),
  );
}
```

## 12.3 Global Error Handling

Global error handling ensures that all unhandled errors in your Flutter application are caught and processed appropriately, providing a consistent error handling strategy across the entire application.

### Setting Up Global Error Handling

Configure global error handlers in your main function:

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/services.dart';
import 'dart:async';

void main() {
  runZonedGuarded<Future<void>>(() async {
    WidgetsFlutterBinding.ensureInitialized();
    
    // Handle Flutter framework errors
    FlutterError.onError = (FlutterErrorDetails details) {
      FlutterError.presentError(details);
      GlobalErrorHandler.handleError(details.exception, details.stack);
    };
    
    // Handle errors outside of Flutter framework
    PlatformDispatcher.instance.onError = (error, stack) {
      GlobalErrorHandler.handleError(error, stack);
      return true;
    };
    
    runApp(MyApp());
  }, (error, stackTrace) {
    GlobalErrorHandler.handleError(error, stackTrace);
  });
}
```

### Global Error Handler Implementation

Create a centralized error handler class:

```dart
import 'package:logging/logging.dart';

class GlobalErrorHandler {
  static final Logger _logger = Logger('GlobalErrorHandler');
  
  static void initialize() {
    Logger.root.level = Level.ALL;
    Logger.root.onRecord.listen((record) {
      print('${record.level.name}: ${record.time}: ${record.message}');
    });
  }
  
  static void handleError(Object error, StackTrace? stackTrace) {
    _logger.severe('Unhandled error occurred', error, stackTrace);
    
    // Send error to crash reporting service
    _sendToCrashReporting(error, stackTrace);
    
    // Show user-friendly error message
    _showErrorToUser(error);
  }
  
  static void _sendToCrashReporting(Object error, StackTrace? stackTrace) {
    // Integration with Firebase Crashlytics or similar service
    // FirebaseCrashlytics.instance.recordError(error, stackTrace);
    print('Sending error to crash reporting: $error');
  }
  
  static void _showErrorToUser(Object error) {
    // Show a user-friendly error message
    // This could be a toast, dialog, or error page
    print('Showing error to user: Something went wrong');
  }
}
```

### Error Interceptor for HTTP Requests

Implement a global error interceptor for network requests:

```dart
import 'package:dio/dio.dart';

class ErrorInterceptor extends Interceptor {
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    String errorMessage = _handleError(err);
    
    // Log the error globally
    GlobalErrorHandler.handleError(
      Exception(errorMessage), 
      err.stackTrace,
    );
    
    super.onError(err, handler);
  }
  
  String _handleError(DioException error) {
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
        return 'Connection timeout';
      case DioExceptionType.sendTimeout:
        return 'Send timeout';
      case DioExceptionType.receiveTimeout:
        return 'Receive timeout';
      case DioExceptionType.badResponse:
        return 'Bad response: ${error.response?.statusCode}';
      case DioExceptionType.cancel:
        return 'Request cancelled';
      case DioExceptionType.unknown:
      default:
        return 'Network error occurred';
    }
  }
}

// Usage
final dio = Dio();
dio.interceptors.add(ErrorInterceptor());
```

### Error State Management

Create a global error state manager:

```dart
import 'package:flutter/material.dart';

class ErrorState extends ChangeNotifier {
  String? _currentError;
  bool _hasError = false;
  
  String? get currentError => _currentError;
  bool get hasError => _hasError;
  
  void setError(String error) {
    _currentError = error;
    _hasError = true;
    notifyListeners();
  }
  
  void clearError() {
    _currentError = null;
    _hasError = false;
    notifyListeners();
  }
}

// Global error display widget
class GlobalErrorDisplay extends StatelessWidget {
  final Widget child;
  
  const GlobalErrorDisplay({Key? key, required this.child}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Consumer<ErrorState>(
      builder: (context, errorState, child) {
        return Stack(
          children: [
            child!,
            if (errorState.hasError)
              Positioned(
                top: 0,
                left: 0,
                right: 0,
                child: Material(
                  color: Colors.red,
                  child: Padding(
                    padding: const EdgeInsets.all(16.0),
                    child: Row(
                      children: [
                        Expanded(
                          child: Text(
                            errorState.currentError ?? 'Unknown error',
                            style: const TextStyle(color: Colors.white),
                          ),
                        ),
                        IconButton(
                          icon: const Icon(Icons.close, color: Colors.white),
                          onPressed: () => errorState.clearError(),
                        ),
                      ],
                    ),
                  ),
                ),
              ),
          ],
        );
      },
      child: child,
    );
  }
}
```

### Complete Example: Integrated Error Handling

Here is a complete example that demonstrates all three error handling approaches:

```dart
import 'package:flutter/material.dart';
import 'package:dartz/dartz.dart';
import 'package:provider/provider.dart';

void main() {
  GlobalErrorHandler.initialize();
  
  runZonedGuarded<Future<void>>(() async {
    FlutterError.onError = (FlutterErrorDetails details) {
      GlobalErrorHandler.handleError(details.exception, details.stack);
    };
    
    runApp(MyApp());
  }, (error, stackTrace) {
    GlobalErrorHandler.handleError(error, stackTrace);
  });
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => ErrorState(),
      child: MaterialApp(
        home: GlobalErrorDisplay(
          child: HomeScreen(),
        ),
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Error Handling Demo')),
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () => _simulateFlutterError(context),
            child: const Text('Simulate Flutter Error'),
          ),
          ElevatedButton(
            onPressed: () => _simulateDartzError(context),
            child: const Text('Simulate Dartz Error'),
          ),
          ElevatedButton(
            onPressed: () => _simulateGlobalError(context),
            child: const Text('Simulate Global Error'),
          ),
        ],
      ),
    );
  }
  
  void _simulateFlutterError(BuildContext context) {
    throw FlutterError('Simulated Flutter error');
  }
  
  void _simulateDartzError(BuildContext context) {
    final result = divide(10, 0);
    result.fold(
      (error) => Provider.of<ErrorState>(context, listen: false).setError(error),
      (value) => print('Success: $value'),
    );
  }
  
  void _simulateGlobalError(BuildContext context) {
    throw Exception('Simulated global error');
  }
}
```

This comprehensive error handling system provides multiple layers of error detection and handling, ensuring that your Flutter application remains stable and provides meaningful feedback to users when errors occur.
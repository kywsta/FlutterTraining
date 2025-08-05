# Theme

Flutter theming provides a centralized way to define the visual appearance of your application. The `ThemeData` class allows you to specify colors, typography, component styles, and other visual properties that will be consistently applied throughout your app.

### ThemeData

The `ThemeData` class is the foundation of Flutter's theming system. It contains all the styling information for your app's visual elements.

### Material Design 3 (Material You)

Flutter supports Material Design 3, which introduces dynamic color schemes and improved accessibility features.

## Basic Theme Implementation

### Setting Up a Basic Theme

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData.light(useMaterial3: true),
      darkTheme: ThemeData.dark(useMaterial3: true),
      themeMode: ThemeMode.system,
      home: const MyHomePage(),
    );
  }
}
```

### Accessing Theme Data

```dart
@override
Widget build(BuildContext context) {
  final theme = Theme.of(context);

  final titleTextStyle = theme.textTheme.titleMedium;

  final titleTextColor = theme.colorScheme.onPrimary;

  return Text(
    title,
    style: titleTextStyle?.copyWith(color: titleTextColor),
  );
}
```

## Color Schemes

### Using ColorScheme

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
          brightness: Brightness.light,
        ),
        useMaterial3: true,
      ),
      home: MyHomePage(),
    );
  }
}
```

### Custom Color Scheme

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: const ColorScheme(
          brightness: Brightness.light,
          primary: Color(0xFF6750A4),
          onPrimary: Color(0xFFFFFFFF),
          secondary: Color(0xFF625B71),
          onSecondary: Color(0xFFFFFFFF),
          error: Color(0xFFBA1A1A),
          onError: Color(0xFFFFFFFF),
          background: Color(0xFFFFFBFE),
          onBackground: Color(0xFF1C1B1F),
          surface: Color(0xFFFFFBFE),
          onSurface: Color(0xFF1C1B1F),
        ),
        useMaterial3: true,
      ),
      home: MyHomePage(),
    );
  }
}
```

## Typography

### Custom Text Theme

```dart
TextTheme customTextTheme = TextTheme(
  displayLarge: TextStyle(
    fontSize: 57,
    fontWeight: FontWeight.w400,
    letterSpacing: -0.25,
  ),
  displayMedium: TextStyle(
    fontSize: 45,
    fontWeight: FontWeight.w400,
  ),
  // ... other text styles
);

// Apply to theme
ThemeData(
  textTheme: customTextTheme,
  // ... other theme properties
)
```

### Using Google Fonts

```dart
// Add google_fonts dependency to pubspec.yaml
// > flutter pub add google_fonts

import 'package:google_fonts/google_fonts.dart';

ThemeData(
  textTheme: GoogleFonts.robotoTextTheme(
    Theme.of(context).textTheme,
  ),
  // Or for specific text styles
  // textTheme: TextTheme(
  //   headlineLarge: GoogleFonts.roboto(
  //     fontSize: 32,
  //     fontWeight: FontWeight.bold,
  //   ),
  // ),
)
```

## Component Themes

### Button Themes

```dart
ThemeData(
  elevatedButtonTheme: ElevatedButtonThemeData(
    style: ElevatedButton.styleFrom(
      backgroundColor: Colors.blue,
      foregroundColor: Colors.white,
      // ... other properties
    ),
  ),

  textButtonTheme: TextButtonThemeData(
    style: TextButton.styleFrom(
      foregroundColor: Colors.blue,
      // ... other properties
    ),
  ),

  outlinedButtonTheme: OutlinedButtonThemeData(
    style: OutlinedButton.styleFrom(
      foregroundColor: Colors.blue,
      side: BorderSide(color: Colors.blue, width: 1),
      // ... other properties
    ),
  ),
)
```

### Input Decoration Theme

```dart
ThemeData(
  inputDecorationTheme: InputDecorationTheme(
    filled: true,
    fillColor: Colors.grey[100],
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
      borderSide: BorderSide(color: Colors.grey[300]!),
    ),
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
      borderSide: BorderSide(color: Colors.grey[300]!),
    ),
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
      borderSide: BorderSide(color: Colors.blue, width: 2),
    ),
    errorBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(8),
      borderSide: BorderSide(color: Colors.red),
    ),
    labelStyle: TextStyle(color: Colors.grey[600]),
    hintStyle: TextStyle(color: Colors.grey[400]),
    contentPadding: EdgeInsets.symmetric(horizontal: 16, vertical: 12),
  ),
)
```

### Card Theme

```dart
ThemeData(
  cardTheme: CardTheme(
    elevation: 4,
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(12),
    ),
    margin: EdgeInsets.all(8),
    color: Colors.white,
    shadowColor: Colors.black26,
  ),
)
```

## Dark Theme

### Implementing Dark Theme

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Theme Demo',

      // Light theme
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.blue,
          brightness: Brightness.light,
        ),
        useMaterial3: true,
      ),

      // Dark theme
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.blue,
          brightness: Brightness.dark,
        ),
        useMaterial3: true,
      ),

      // Theme mode (system, light, dark)
      themeMode: ThemeMode.system,

      home: MyHomePage(),
    );
  }
}
```

## Best Practices

### 1. Use Material 3 Design System

```dart
ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
)
```

### 2. Consistent Color Usage

```dart
// Access colors through theme
Container(
  color: Theme.of(context).colorScheme.primary,
  // Instead of hardcoded colors
  // color: Colors.blue,
)
```

### 3. Typography Consistency

```dart
Text(
  'Title',
  style: Theme.of(context).textTheme.headlineMedium,
)
```

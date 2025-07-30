# Localization

Flutter provides built-in support for internationalization (i18n) through the `flutter_localizations` package and the `intl` package.

## Github

https://github.com/kywsta/flutter_localization_sample.git

## Setting Up Localization

### Dependencies

Add the following dependencies to your `pubspec.yaml`:

By using the following command,

```bash
flutter pub add flutter_localizations --sdk=flutter
flutter pub add intl:any
```

will add the following to your `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.20.2
```

### Enabling Localization in MaterialApp

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Localization Demo',

      // Localization delegates
      localizationsDelegates: [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],

      // Supported locales
      supportedLocales: [
        Locale('en', 'US'), // English (United States)
        Locale('my', 'MM'), // Myanmar (Myanmar)
      ],

      home: MyHomePage(),
    );
  }
}
```

## Generating Localization Files

### Creating ARB Files

Create the following directory structure:

```
lib/
  l10n/
    app_en.arb
    app_my.arb
```

### English ARB File (app_en.arb)

```json
{
  "homePageTitle": "Localization Sample",
  "@homePageTitle": {
    "description": "The title of the home page"
  },
  "helloWorld": "Hello World!",
  "@helloWorld": {
    "description": "The conventional newborn programmer greeting"
  }
}
```

### Myanmar ARB File (app_my.arb)

```json
{
  "homePageTitle": "ဘာသာစကား စမ်းသပ်ခြင်း",
  "helloWorld": "မင်္ဂလာပါ ကမ္ဘာကြီးရေ"
}
```

### Configuration File (l10n.yaml)

Create `l10n.yaml` in the root directory:

```yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
```

### Generating Localization Files

Run the following command to generate the localization files in `lib/l10n`:

```bash
flutter pub get

or

flutter run
```

## Using Generated `AppLocalizations`

### Basic Usage

```dart
class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      supportedLocales: AppLocalizations.supportedLocales,
      localizationsDelegates: AppLocalizations.localizationsDelegates,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;

    return Scaffold(
      appBar: AppBar(title: Text(l10n.homePageTitle)),
      body: Center(child: Text(l10n.helloWorld)),
    );
  }
}
```

### Form Validation with Localization

```dart
class HomePage extends StatelessWidget {
  final _emailRegex = r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$';

  // ...

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;

    return TextFormField(
      decoration: InputDecoration(
        labelText: l10n.emailInputLabel,
      ),
      validator: (value) {
        if (value == null || value.isEmpty) {
          return l10n.invalidValue;
        }

        if (!RegExp(_emailRegex).hasMatch(value)) {
          return l10n.invalidValue;
        }

        return null;
      },
      autovalidateMode: AutovalidateMode.always,
    );
  }
}
```

## Dynamic Locale Switching

### Locale Provider

First, we need to create `ChangeNotifier` to provide the current locale and to notify the UI when the locale is changed.

```dart
class LocaleProvider extends ChangeNotifier {
  Locale _locale = AppLocalizations.supportedLocales.first;

  Locale get locale => _locale;

  void setLocale(Locale locale) {
    if (!supportedLocales.contains(locale)) return;

    _locale = locale;
    notifyListeners();
  }

  void clearLocale() {
    _locale = AppLocalizations.supportedLocales.first;
    notifyListeners();
  }

  static const List<Locale> supportedLocales = AppLocalizations.supportedLocales;
}
```

### Add `provider` package

Second, we need to add `provider` package to our project to provide the `LocaleProvider` to the whole app.

```bash
flutter pub add provider
```

### Provide `LocaleProvider` using `ChangeNotifierProvider`

Third, we need to provide the `LocaleProvider` to the whole app using `ChangeNotifierProvider` by passing it to `MultiProvider`, and then we can use `context.select`|`context.watch` to get the current locale, which will trigger the UI to rebuild when the locale is changed.

```dart
class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => LocaleProvider()),
      ],
      builder: (context, child) {
        final locale = context.select<LocaleProvider, Locale>(
          (localeProvider) => localeProvider.locale,
        );

        return MaterialApp(
          locale: locale,
          supportedLocales: AppLocalizations.supportedLocales,
          localizationsDelegates: AppLocalizations.localizationsDelegates,
          home: const HomePage(),
        );
      },
    );
  }
}
```

### Language Selection Widget

Fourth, we need to create a language selection widget to change the locale.

```dart
class LanguageSelector extends StatelessWidget {
  const LanguageSelector({super.key});

  @override
  Widget build(BuildContext context) {
    // Get the current locale from the LocaleProvider
    final localeProvider = context.watch<LocaleProvider>();

    return PopupMenuButton(
    icon: const Icon(Icons.language),
    itemBuilder: (context) => AppLocalizations.supportedLocales
        .map(
          (locale) => PopupMenuItem(
            value: locale,
            child: Text(locale.languageCode.toUpperCase()),
          ),
        )
        .toList(),
    onSelected: (locale) {
      // Set the new locale and notify the UI to rebuild
      localeProvider.setLocale(locale);
    },
  );
  }
}
```

## Number and Date Formatting

### Number Formatting

```dart
@override
Widget build(BuildContext context) {
  final locale = Localizations.localeOf(context);

  final numberFormat = NumberFormat.decimalPattern(locale.toString());
  final currencyFormat = NumberFormat.currency(locale: locale.toString());
  final percentFormat = NumberFormat.percentPattern(locale.toString());
  final compactFormat = NumberFormat.compact(locale: locale.toString());

  return Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    mainAxisSize: MainAxisSize.min,
    children: [
      Text('Locale: ${locale.toString()}'),
      SizedBox(height: 16),
      Text('Number: ${numberFormat.format(1234567.89)}'),
      Text('Currency: ${currencyFormat.format(1234.56)}'),
      Text('Percent: ${percentFormat.format(0.75)}'),
      Text('Compact: ${compactFormat.format(1234567)}'),
    ],
  );
}
```

### Date and Time Formatting

```dart
@override
Widget build(BuildContext context) {
  final locale = Localizations.localeOf(context);
  final now = DateTime.now();

  final dateFormat = DateFormat.yMd(locale.toString());
  final timeFormat = DateFormat.jm(locale.toString());
  final dateTimeFormat = DateFormat.yMd(locale.toString()).add_jm();
  final longDateFormat = DateFormat.yMMMMEEEEd(locale.toString());

  return Column(
    crossAxisAlignment: CrossAxisAlignment.stretch,
    mainAxisSize: MainAxisSize.min,
    children: [
      Text('Current DateTime: $now'),
      SizedBox(height: 16),

      Text('Date: ${dateFormat.format(now)}'),
      Text('Time: ${timeFormat.format(now)}'),
      Text('DateTime: ${dateTimeFormat.format(now)}'),
      Text('Long Date: ${longDateFormat.format(now)}'),

      SizedBox(height: 16),

      // Relative time
      Text(
        'Yesterday: ${dateFormat.format(now.subtract(Duration(days: 1)))}',
      ),
      Text('Next week: ${dateFormat.format(now.add(Duration(days: 7)))}'),
    ],
  );
}
```

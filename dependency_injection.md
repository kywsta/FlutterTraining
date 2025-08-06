# 7. Dependency Injection

## Introduction

Dependency Injection (DI) is a design pattern that allows objects to receive their dependencies from external sources rather than creating them internally. In Flutter development, proper dependency injection leads to more testable, maintainable, and loosely coupled code.

## Why Use Dependency Injection?

- **Testability**: Easy to mock dependencies for unit testing
- **Maintainability**: Changes to dependencies don't affect dependent classes
- **Flexibility**: Easy to swap implementations
- **Separation of Concerns**: Classes focus on their primary responsibilities

## Provider Package

Provider is Flutter's recommended state management solution that also serves as a lightweight dependency injection container.

### Installation

Add to your `pubspec.yaml`:

```yaml
dependencies:
  provider: latest
```

### Basic Usage

#### 1. Creating a Service and a Provider

In this example, there is a `ItemService` that fetches items from an API and a `ItemList` that fetches and stores the items to display in the UI. The `ItemList` is a `ChangeNotifier` that fetches the items from the `ItemService` and updates the UI when the items are fetched.

Another example is a `Counter` that stores the count and a `CounterTextTranslation` that translates the count to a text. The `CounterTextTranslation` is a `ChangeNotifier` that depends on the `Counter` which change over time, so the `CounterTextTranslation` must be updated when the `Counter` is updated.

```dart
// item_service.dart
class ItemService {
  final _items = ['Item 1', 'Item 2', 'Item 3'];

  ItemService();

  Future<List<String>> fetchItems() async {
    await Future.delayed(const Duration(seconds: 3));
    return _items;
  }
}
```

```dart
// item_list.dart
class ItemList extends ChangeNotifier {
  final ItemService itemService;

  final List<String> _items = [];

  bool _isLoading = false;

  ItemList(this.itemService) {
    _fetchItems();
  }

  List<String> get items => _items;

  bool get isLoading => _isLoading;

  void _fetchItems() async {
    _isLoading = true;
    notifyListeners();

    final items = await itemService.fetchItems();
    _items.addAll(items);
    _isLoading = false;
    notifyListeners();
  }
}
```

```dart
// counter.dart
class Counter extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}
```

```dart
// counter_text_translation.dart
class CounterTextTranslation {
  final Counter counter;

  CounterTextTranslation(this.counter);

  String get text => 'You have clicked ${counter.count} times';
}
```

Now, we implemented all of the classes that we need for the example.

#### 2. Providing Dependencies

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        // ItemService is a singleton provider
        Provider(create: (_) => ItemService()),
        // ItemList is a notifier provider that depends on ItemService
        ChangeNotifierProvider(
          create: (context) => ItemList(context.read<ItemService>()),
        ),
        // Counter is a notifier provider
        ChangeNotifierProvider(create: (_) => Counter()),
        // CounterTextTranslation is a proxy provider that depends on Counter
        ProxyProvider<Counter, CounterTextTranslation>(
          update: (_, value, __) => CounterTextTranslation(value),
        ),
      ],
      child: MaterialApp(
        home: HomePage(),
      ),
    ),
  );
}
```

#### 3. Consuming Dependencies

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    // We can use the context.watch to watch the changes
    final counterText = context.watch<CounterTextTranslation>().text;
    // We can use the context.select to select the specific property
    final isLoading = context.select<ItemList, bool>(
      (itemList) => itemList.isLoading,
    );
    final items = context.select<ItemList, List<String>>(
      (itemList) => itemList.items,
    );

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(counterText),
          Expanded(
            child: (isLoading)
                ? Center(child: CircularProgressIndicator())
                : ListView.builder(
                    itemCount: items.length,
                    itemBuilder: (context, index) {
                      return Text(items[index]);
                    },
                  ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // We can use the context.read to read the provider
          context.read<Counter>().increment();
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

#### 4. Using Consumer Widget

```dart
class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Consumer<CounterTextTranslation>(
        builder: (context, value, child) {
          return Center(child: Text(value.text));
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          context.read<Counter>().increment();
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

## "get_it" Package

GetIt is a simple service locator for Dart and Flutter projects that provides global access to registered dependencies.

### Installation

Add to your `pubspec.yaml`:

```yaml
dependencies:
  get_it: latest
```

### Basic Setup

In this example, we will use the "get_it" package to implement the dependency injection.

We will use the same example as the Provider package. First, we will create the same classes as the Provider package.

```dart
// item_data_source.dart
abstract class ItemDataSource {
  Future<List<String>> fetchItems();
}

class MockItemDataSourceImpl implements ItemDataSource {
  final List<String> items = ['Item 1', 'Item 2', 'Item 3'];

  @override
  Future<List<String>> fetchItems() async {
    await Future.delayed(const Duration(seconds: 3));
    return items;
  }
}
```

```dart
// item_repository.dart
abstract class ItemRepository {
  final ItemDataSource itemDataSource;

  ItemRepository({required this.itemDataSource});

  Future<List<String>> fetchItems();
}

class ItemRepositoryImpl extends ItemRepository {
  ItemRepositoryImpl({required super.itemDataSource});

  @override
  Future<List<String>> fetchItems() async {
    return itemDataSource.fetchItems();
  }
}
```

```dart
// item_list.dart
class ItemList extends ChangeNotifier {
  final ItemRepository itemRepository;

  final List<String> _items = [];

  bool _isLoading = false;

  ItemList(this.itemRepository) {
    _fetchItems();
  }

  List<String> get items => _items;

  bool get isLoading => _isLoading;

  void _fetchItems() async {
    _isLoading = true;
    notifyListeners();

    final items = await itemRepository.fetchItems();
    _items.addAll(items);
    _isLoading = false;
    notifyListeners();
  }
}
```

#### 1. Service Locator Setup

After the required classes are created, we can setup the service locator.

```dart
// service_locator.dart
final serviceLocator = GetIt.instance;

void setupServiceLocator() {
  serviceLocator.registerLazySingleton<ItemDataSource>(
    () => MockItemDataSourceImpl(),
  );

  serviceLocator.registerLazySingleton<ItemRepository>(
    () => ItemRepositoryImpl(itemDataSource: serviceLocator<ItemDataSource>()),
  );
}
```

#### 2. Main Application Setup

```dart
void main() {
  setupServiceLocator();

  runApp(const App());
}

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(
          create: (context) => ItemList(serviceLocator<ItemRepository>()),
        ),
      ],
      builder: (_, _) {
        return MaterialApp(home: ItemListPage());
      },
    );
  }
}
```

### Advanced GetIt Usage

#### 1. Conditional Registration

```dart
void setupLocator() {
  if (kDebugMode) {
    locator.registerSingleton<LoggingService>(DebugLoggingService());
  } else {
    locator.registerSingleton<LoggingService>(ProductionLoggingService());
  }
}
```

#### 2. Named Registration

```dart
void setupLocator() {
  locator.registerSingleton<ApiService>(
    ApiService(baseUrl: 'https://api.dev.com'),
    instanceName: 'dev'
  );

  locator.registerSingleton<ApiService>(
    ApiService(baseUrl: 'https://api.prod.com'),
    instanceName: 'prod'
  );
}

// Usage
final devApi = locator<ApiService>(instanceName: 'dev');
final prodApi = locator<ApiService>(instanceName: 'prod');
```

#### 3. Async Registration

```dart
void setupLocator() {
  locator.registerSingletonAsync<SharedPreferences>(
    () async => await SharedPreferences.getInstance(),
  );
}

// Wait for async dependencies
await locator.allReady();
```

## Provider vs GetIt Comparison

| Feature                | Provider           | GetIt                    |
| ---------------------- | ------------------ | ------------------------ |
| **Learning Curve**     | Moderate           | Easy                     |
| **Widget Integration** | Excellent          | Manual                   |
| **Global Access**      | Through context    | Direct access            |
| **Testing**            | Good with context  | Excellent                |
| **Performance**        | Widget rebuilds    | No rebuilds              |
| **Scoping**            | Widget tree scoped | Manual scoping           |
| **State Management**   | Built-in           | Separate solution needed |

## Best Practices

### 1. Interface Segregation

```dart
abstract class DataSource {
  Future<List<User>> getUsers();
}

class ApiDataSource implements DataSource {
  @override
  Future<List<User>> getUsers() async {
    // API implementation
  }
}

class CacheDataSource implements DataSource {
  @override
  Future<List<User>> getUsers() async {
    // Cache implementation
  }
}
```

### 2. Environment-Based Registration

```dart
void setupLocator() {
  if (Environment.isDevelopment) {
    locator.registerSingleton<DataSource>(MockDataSource());
  } else {
    locator.registerSingleton<DataSource>(ApiDataSource());
  }
}
```

### 3. Disposal Management

```dart
void setupLocator() {
  locator.registerSingleton<DatabaseService>(
    DatabaseService(),
    dispose: (service) => service.close(),
  );
}
```

## Testing with Dependency Injection

### Unit Testing with GetIt

```dart
void main() {
  setUp(() {
    serviceLocator.reset();
    setupServiceLocator();
  });

  test('should get items from repository', () async {
    final itemRepository = serviceLocator<ItemRepository>();
    expect(itemRepository, isA<ItemRepository>());

    final items = await itemRepository.fetchItems();
    expect(items, isA<List<String>>());
    expect(items.length, greaterThan(0));
  });
}
```

### Widget Testing with Provider

```dart
testWidgets('should shown progress indicator during loading and shown items in the list after loading', (tester) async {
  await tester.pumpWidget(const App());

  expect(find.byType(ItemListPage), findsOneWidget);

  BuildContext context = tester.element(find.byType(ItemListPage));

  final itemList = Provider.of<ItemList>(
    context,
    listen: false,
  );

  expect(itemList, isA<ItemList>());
  expect(itemList.isLoading, isTrue);
  expect(find.byType(CircularProgressIndicator), findsOneWidget);

  await tester.pump(Duration(seconds: 3));

  expect(itemList.isLoading, isFalse);
  expect(find.byType(CircularProgressIndicator), findsNothing);
  expect(find.byType(ListView), findsOneWidget);
  expect(find.text('Item 1'), findsOneWidget);
  expect(find.text('Item 2'), findsOneWidget);
  expect(find.text('Item 3'), findsOneWidget);
});
```

## Conclusion

Dependency injection is crucial for building maintainable Flutter applications. Provider excels when you need tight widget integration and state management, while GetIt offers simplicity and global access. Choose based on your project's specific requirements and team preferences.

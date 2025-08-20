# 9. State Management

Over the development with simple state management using 'setState' and 'Provider' package, we will learn two popular state management solutions: BLoC and Riverpod.

## Github

https://github.com/kywsta/flutter_state_manage_sample

## 9.1 BLoC Pattern

The BLoC pattern separates business logic from the UI layer, making your code more testable, reusable, and maintainable. It uses streams to handle state changes and events.

### 9.1.1 Core Concepts

- **Events**: Input to the BLoC (user actions, API calls)
- **States**: Output from the BLoC (UI states)
- **BLoC**: Contains business logic and transforms events into states

### 9.1.2 Dependencies

Add the following dependencies to your `pubspec.yaml`:

```yaml
dependencies:
  flutter_bloc: ^8.1.3
  equatable: ^2.0.5
```

### 9.1.3 Basic BLoC Implementation

```dart
// counter_event.dart
abstract class CounterEvent extends Equatable {
  const CounterEvent();

  @override
  List<Object> get props => [];
}

class CounterIncremented extends CounterEvent {}

class CounterDecremented extends CounterEvent {}
```

```dart
// counter_state.dart
class CounterState extends Equatable {
  const CounterState({this.count = 0});

  final int count;

  @override
  List<Object> get props => [count];
}
```

```dart
// counter_bloc.dart
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterState()) {
    on<CounterIncremented>(_onIncrement);
    on<CounterDecremented>(_onDecrement);
  }

  void _onIncrement(CounterIncremented event, Emitter<CounterState> emit) {
    emit(CounterState(count: state.count + 1));
  }

  void _onDecrement(CounterDecremented event, Emitter<CounterState> emit) {
    emit(CounterState(count: state.count - 1));
  }
}
```

### 9.1.4 Using BLoC in UI

```dart
class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => CounterBloc(),
      child: CounterView(),
    );
  }
}

class CounterView extends StatelessWidget {
  const CounterView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: _buildAppBar(),
      body: _buildCounterText(),
      floatingActionButton: _buildCounterButtons(context),
    );
  }

  AppBar _buildAppBar() => AppBar(title: Text('Counter'));

  Widget _buildCounterText() {
    return Center(
      child: BlocBuilder<CounterBloc, CounterState>(
        builder: (context, state) {
          return Text(
            '${state.count}',
            style: Theme.of(context).textTheme.headlineMedium,
          );
        },
      ),
    );
  }

  Widget _buildCounterButtons(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.end,
      children: [
        FloatingActionButton(
          onPressed: () => _onIncrementPressed(context),
          child: Icon(Icons.add),
        ),
        SizedBox(height: 8),
        FloatingActionButton(
          onPressed: () => _onDecrementPressed(context),
          child: Icon(Icons.remove),
        ),
      ],
    );
  }

  void _onIncrementPressed(BuildContext context) {
    context.read<CounterBloc>().add(CounterIncremented());
  }

  void _onDecrementPressed(BuildContext context) {
    context.read<CounterBloc>().add(CounterDecremented());
  }
}
```

## 9.2 Cubit

Cubit is a simplified version of BLoC that doesn't use events. It directly exposes methods to trigger state changes.

### 9.2.1 Basic Cubit Implementation

```dart
// counter_cubit.dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
  void reset() => emit(0);
}
```

### 9.2.2 Using Cubit in UI

```dart
class CounterCubitPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => CounterCubit(),
      child: Scaffold(
        appBar: AppBar(title: Text('Counter Cubit')),
        body: Center(
          child: BlocBuilder<CounterCubit, int>(
            builder: (context, count) {
              return Text(
                '$count',
                style: Theme.of(context).textTheme.headlineMedium,
              );
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => context.read<CounterCubit>().increment(),
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}
```

## 9.3 Authentication State Example using BLoC

### 9.3.1 Authentication Events

```dart
// auth_event.dart
abstract class AuthEvent extends Equatable {
  const AuthEvent();

  @override
  List<Object> get props => [];
}

class LoginEvent extends AuthEvent {
  const LoginEvent({required this.email, required this.password});

  final String email;
  final String password;

  @override
  List<Object> get props => [email, password];
}

class LogoutEvent extends AuthEvent {}
```

### 9.3.2 Authentication States

```dart
// auth_state.dart
enum AuthStatus { unknown, authenticated, unauthenticated }

class AuthState extends Equatable {
  const AuthState._({
    this.status = AuthStatus.unknown,
    this.user,
    this.errorMessage,
  });

  const AuthState.unknown() : this._();

  const AuthState.authenticated({required User user})
    : this._(status: AuthStatus.authenticated, user: user);

  const AuthState.unauthenticated({String? errorMessage})
    : this._(status: AuthStatus.unauthenticated, errorMessage: errorMessage);

  final AuthStatus status;
  final User? user;
  final String? errorMessage;

  @override
  List<Object?> get props => [status, user, errorMessage];
}

class User extends Equatable {
  const User({required this.id, required this.email, required this.name});

  final String id;
  final String email;
  final String name;

  @override
  List<Object> get props => [id, email, name];
}
```

### 9.3.3 Authentication BLoC

```dart
// auth_bloc.dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(const AuthState.unknown()) {
    on<LoginEvent>(_onLoginRequested);
    on<LogoutEvent>(_onLogoutRequested);
  }

  void _onLoginRequested(LoginEvent event, Emitter<AuthState> emit) async {
    try {
      final user = User(id: '1', email: 'john@test.com', name: 'John Doe');
      await Future.delayed(const Duration(milliseconds: 500));
      emit(AuthState.authenticated(user: user));
    } catch (error) {
      emit(AuthState.unauthenticated(errorMessage: error.toString()));
    }
  }

  void _onLogoutRequested(LogoutEvent event, Emitter<AuthState> emit) async {
    await Future.delayed(const Duration(milliseconds: 500));
    emit(const AuthState.unauthenticated());
  }
}
```

### 9.3.4 Using Authentication BLoC

```dart
class AuthStatusPage extends StatefulWidget {
  const AuthStatusPage({super.key});

  @override
  State<AuthStatusPage> createState() => _AuthStatusPageState();
}

class _AuthStatusPageState extends State<AuthStatusPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Builder(
          builder: (context) {
            final isAuthenticated = context.select(
              (AuthBloc bloc) => bloc.state.status == AuthStatus.authenticated,
            );
            return isAuthenticated
                ? _buildAuthenticatedContent()
                : _buildUnauthenticatedContent();
          },
        ),
      ),
    );
  }

  Widget _buildAuthenticatedContent() {
    return Builder(
      builder: (context) {
        final user = context.select((AuthBloc bloc) => bloc.state.user);

        return Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(Icons.check_circle),
              Text(
                'Authenticated',
                style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
              ),
              Text("${user?.name}\n${user?.email}"),
              ElevatedButton(onPressed: _logout, child: Text('Logout')),
            ],
          ),
        );
      },
    );
  }

  Widget _buildUnauthenticatedContent() {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(Icons.lock),
          Text(
            'Unauthenticated',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          ElevatedButton(onPressed: _login, child: Text('Login')),
        ],
      ),
    );
  }

  void _login() {
    context.read<AuthBloc>().add(
      LoginEvent(email: 'test@test.com', password: '123456'),
    );
  }

  void _logout() {
    context.read<AuthBloc>().add(LogoutEvent());
  }
}
```

## 9.4 Riverpod

Riverpod is an upgraded version of Provider which itself is a reactive caching framework. It provides multiple useful providers for different use cases and can be used with or without riverpod_generator for code generation.

### 9.4.1 Dependencies

Add Riverpod to your `pubspec.yaml`:

```yaml
dependencies:
  flutter_riverpod: ^2.4.9
```

### 9.4.2 Setting up Riverpod

Wrap your app with `ProviderScope`:

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

### 9.4.3 Using Providers

Reading providers can be done using `ConsumerWidget`, `Consumer`, and `ConsumerStatefulWidget`.

## 9.5 Riverpod Providers

### 9.5.1 Provider

Used for providing immutable objects or constants:

```dart
final stringProvider = Provider<String>((ref) => 'Hello World');
final configProvider = Provider<AppConfig>((ref) => AppConfig.dev());

// Usage
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final greeting = ref.watch(stringProvider);
    final config = ref.watch(configProvider);

    return Text(greeting);
  }
}
```

### 9.5.2 StateProvider

For simple state that can be modified:

```dart
final counterProvider = StateProvider<int>((ref) => 0);

// Usage
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).state++,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

### 9.5.3 StateNotifierProvider

StateNotifierProvider and StateNotifier are ideal for managing state that may change in reaction to an event or user interaction.

```dart
import 'dart:async';

class Clock extends StateNotifier<DateTime> {
  // 1. initialize with current time
  Clock() : super(DateTime.now()) {
    // 2. create a timer that fires every second
    _timer = Timer.periodic(Duration(seconds: 1), (_) {
      // 3. update the state with the current time
      state = DateTime.now();
    });
  }

  late final Timer _timer;

  // 4. cancel the timer when finished
  @override
  void dispose() {
    _timer.cancel();
    super.dispose();
  }
}
```

This class sets the initial state by calling super(DateTime.now()) in the constructor, and updates it every second using a periodic timer.

Once we have this, we can create a new provider:

```dart
final clockProvider = StateNotifierProvider<Clock, DateTime>((ref) {
  return Clock();
});
```

Then, we can watch the clockProvider inside a ConsumerWidget to get the current time and show it inside a Text widget:

```dart
class ClockWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // watch the StateNotifierProvider to return a DateTime (the state)
    final currentTime = ref.watch(clockProvider);
    // format the time as `hh:mm:ss`
    final timeFormatted = DateFormat.Hms().format(currentTime);
    return Text(timeFormatted);
  }
}
```

### 9.5.4 FutureProvider

For asynchronous operations:

```dart
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
});

// Usage
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
      data: (user) => Text('Welcome, ${user.name}!'),
    );
  }
}
```

### 9.5.4 StreamProvider

For reactive data streams:

```dart
final authStateChangesProvider = StreamProvider.autoDispose<User?>((ref) {
  // get FirebaseAuth from the provider below
  final firebaseAuth = ref.watch(firebaseAuthProvider);
  // call a method that returns a Stream<User?>
  return firebaseAuth.authStateChanges();
});

// provider to access the FirebaseAuth instance
final firebaseAuthProvider = Provider<FirebaseAuth>((ref) {
  return FirebaseAuth.instance;
});
```

And we can use it in a ConsumerWidget to get the current user:

```dart
Widget build(BuildContext context, WidgetRef ref) {
  // watch the StreamProvider and get an AsyncValue<User?>
  final authStateAsync = ref.watch(authStateChangesProvider);
  // use pattern matching to map the state to the UI
  return authStateAsync.when(
    data: (user) => user != null ? HomePage() : SignInPage(),
    loading: () => const CircularProgressIndicator(),
    error: (err, stack) => Text('Error: $err'),
  );
}
```

### 9.5.5 ChangeNotifierProvider

For using ChangeNotifier classes:

```dart
class TodoNotifier extends ChangeNotifier {
  final List<Todo> _todos = [];

  List<Todo> get todos => _todos;

  void addTodo(String title) {
    _todos.add(Todo(title: title));
    notifyListeners();
  }

  void removeTodo(String id) {
    _todos.removeWhere((todo) => todo.id == id);
    notifyListeners();
  }
}

final todoProvider = ChangeNotifierProvider<TodoNotifier>((ref) {
  return TodoNotifier();
});

// Usage
class TodoList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todoNotifier = ref.watch(todoProvider);

    return ListView.builder(
      itemCount: todoNotifier.todos.length,
      itemBuilder: (context, index) {
        final todo = todoNotifier.todos[index];
        return ListTile(
          title: Text(todo.title),
          trailing: IconButton(
            icon: Icon(Icons.delete),
            onPressed: () => todoNotifier.removeTodo(todo.id),
          ),
        );
      },
    );
  }
}
```

### 9.6 Riverpod Generator

For complex state management with `NotifierProvider`, we can use riverpod_generator to generate the provider and the notifier class.

To use riverpod_generator, we need to add the following dependencies:

```bash
flutter pub add flutter_riverpod
flutter pub add riverpod_annotation
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner
flutter pub add dev:custom_lint
flutter pub add dev:riverpod_lint
```

Then, we will create a simple state management example with riverpod_generator. In this example, we will create a shopping cart with the following features:

- Fetch products from the API
- Display products in a list
- Add a product to the cart
- Remove a product from the cart
- Calculate the total price of the cart
- Display the cart summary

Now let's create the models for the products and the cart.

```dart
// product.dart
@JsonSerializable()
class Product {
  final int id;
  final String title;
  final String description;
  final double price;
  final String category;
  final String image;

  Product({
    required this.id,
    required this.title,
    required this.description,
    required this.price,
    required this.category,
    required this.image,
  });

  factory Product.fromJson(Map<String, dynamic> json) =>
      _$ProductFromJson(json);

  Map<String, dynamic> toJson() => _$ProductToJson(this);

  bool get isBudgetFriendly => price <= 100;
}

// cart.dart
@JsonSerializable()
class Cart {
  final List<Product> products;

  Cart({required this.products});

  double get subTotal =>
      products.fold(0, (sum, product) => sum + product.price);

  Cart copyWith({List<Product>? products}) {
    return Cart(products: products ?? this.products);
  }
}
```

After creating the models, we will create the required providers.

```dart
// product_repository.dart
abstract class ProductRepository {
  Future<List<Product>> loadProducts();
}

class ProductRepositoryImpl implements ProductRepository {
  @override
  Future<List<Product>> loadProducts() async {
    final response = await http.get(
      Uri.parse('https://fakestoreapi.com/products'),
    );
    if (response.statusCode == 200) {
      final body = jsonDecode(response.body);
      return (body as List).map((e) => Product.fromJson(e)).toList();
    } else {
      throw Exception('Failed to load products: ${response.statusCode}');
    }
  }
}

@Riverpod(dependencies: [])
ProductRepository productRepository(Ref ref) {
  return ProductRepositoryImpl();
}
```

```dart
// product_provider.dart
@Riverpod(dependencies: [productRepository])
class ProductList extends _$ProductList {
  @override
  Future<List<Product>> build() async {
    final productRepository = ref.watch(productRepositoryProvider);
    return await productRepository.loadProducts();
  }
}
```

```dart
// cart_provider.dart
@riverpod
class ActiveCart extends _$ActiveCart {
  @override
  Cart build() {
    return Cart(products: []);
  }

  void addProduct(Product product) {
    state = state.copyWith(products: [...state.products, product]);
  }

  void removeProduct(Product product) {
    state = state.copyWith(
      products: state.products.where((p) => p.id != product.id).toList(),
    );
  }

  void checkout() {
    state = state.copyWith(products: []);
  }
}
```

After creating the providers, run build_runner to generate the providers.

```bash
dart run build_runner build -d
```

And then, we can use the providers in the UI.

```dart
class StorePage extends ConsumerWidget {
  const StorePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(title: const Text('Store')),
      body: SafeArea(
        child: Stack(
          children: [
            Positioned.fill(child: _buildProductList(ref)),
            Positioned(bottom: 0, left: 8, right: 8, child: _buildCart(ref)),
          ],
        ),
      ),
    );
  }

  Widget _buildProductList(WidgetRef ref) {
    final productList = ref.watch(productListProvider);
    return productList.when(
      data: (productList) => ListView.builder(
        itemCount: productList.length,
        itemBuilder: (context, index) {
          return _buildProductItem(ref, productList[index]);
        },
      ),
      error: (error, stackTrace) => Text('Error: $error'),
      loading: () => const Center(child: CircularProgressIndicator()),
    );
  }

  Widget _buildProductItem(WidgetRef ref, Product product) {
    return ListTile(
      leading: AspectRatio(aspectRatio: 1, child: Image.network(product.image)),
      title: Text(product.title),
      subtitle: Text(
        '\$${product.price}',
        style: TextStyle(fontWeight: FontWeight.bold, color: Colors.red),
      ),
      trailing: IconButton(
        onPressed: () {
          ref.read(activeCartProvider.notifier).addProduct(product);
        },
        icon: const Icon(Icons.add_shopping_cart),
      ),
    );
  }

  Widget _buildCart(WidgetRef ref) {
    final cart = ref.watch(activeCartProvider);
    return Card(
      margin: EdgeInsets.zero,
      color: Colors.grey[200],
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.all(Radius.circular(16)),
      ),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            Expanded(
              child: Text(
                '${cart.products.length} Items',
                style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
              ),
            ),
            Expanded(
              child: Text(
                'Total: ${cart.subTotal}',
                style: TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
            ElevatedButton(
              onPressed: () {
                ref.read(activeCartProvider.notifier).checkout();
              },
              child: const Text('Checkout'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 9.7 Consumer Widgets

### 9.7.1 ConsumerWidget

Replaces StatelessWidget when you need to read providers:

```dart
class MyConsumerWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(someProvider);
    return Text('$value');
  }
}
```

### 9.7.2 ConsumerStatefulWidget

Replaces StatefulWidget when you need to read providers:

```dart
class MyConsumerStatefulWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyConsumerStatefulWidget> createState() =>
      _MyConsumerStatefulWidgetState();
}

class _MyConsumerStatefulWidgetState
    extends ConsumerState<MyConsumerStatefulWidget> {
  @override
  Widget build(BuildContext context) {
    final value = ref.watch(someProvider);
    return Text('$value');
  }
}
```

### 9.7.3 Consumer

For listening to providers in specific parts of the widget tree:

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Static content'),
        Consumer(
          builder: (context, ref, child) {
            final value = ref.watch(someProvider);
            return Text('Dynamic: $value');
          },
        ),
      ],
    );
  }
}
```

## 9.8 Best Practices

1. **Choose the right state management solution**: Use BLoC for complex business logic, Cubit for simpler cases, and Riverpod for reactive programming
2. **Separate business logic from UI**: Keep your widgets focused on presentation
3. **Use proper provider types**: Choose the appropriate Riverpod provider based on your data type
4. **Handle loading and error states**: Always provide feedback for asynchronous operations
5. **Test your state management**: Both BLoC and Riverpod provide excellent testing capabilities
6. **Avoid overusing global state**: Keep state as local as possible

# 8. Architecture

## Overview

Architecture in Flutter development refers to the structural organization of code that defines how different components of an application interact with each other. A well-designed architecture promotes maintainability, testability, and scalability. This section covers two fundamental architectural patterns: Clean Architecture and Model-View-ViewModel (MVVM).

## 8.1 MVVM (Model-View-ViewModel)

MVVM is an architectural pattern that separates the development of graphical user interfaces from business logic. In Flutter, MVVM helps organize code by separating UI logic from business logic through a ViewModel intermediary.

### 8.1.1 Components

#### Model

- Represents data and business logic
- Independent of UI components
- Handles data validation and business rules

#### View

- Represents the UI layer (Flutter widgets)
- Displays data and captures user input
- Communicates with ViewModel

#### ViewModel

- Acts as a bridge between View and Model
- Handles presentation logic
- Manages UI state and user interactions

### 8.1.2 Folder Structure Example

```
lib/
├── models/
│   ├── user.dart
│   └── product.dart
├── viewmodels/
│   ├── user_viewmodel.dart
│   └── product_viewmodel.dart
├── views/
│   ├── pages/
│   │   ├── user_page.dart
│   │   └── product_page.dart
│   └── widgets/
│       ├── user_card.dart
│       └── product_item.dart
├── services/
│   ├── api_service.dart
│   └── local_storage_service.dart
└── main.dart
```

### 8.1.3 Implementation Example

#### Model

We will start with a simple model for a product. In this example, we will use the `json_serializable` package to generate the model from a JSON object.

```dart
// lib/models/product.dart
part 'product.g.dart';

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

  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);

  Map<String, dynamic> toJson() => _$ProductToJson(this);

  bool get isBudgetFriendly => price <= 100;
}
```

#### Service

And then, we will create a service to access the product data from the API. In this example, we will use the `http` package to make the API calls.

**Note:**

At this point, the repository and data source can be implemented for better SOC (Separation of Concerns) and testability. But for the sake of simplicity, we will just use the service.

```dart
class ProductService {
  Future<List<Product>> getProducts() async {
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

  Future<Product> createProduct(Product product) async {
    final response = await http.post(
      Uri.parse('https://fakestoreapi.com/products'),
      body: jsonEncode(product.toJson()),
    );
    if (response.statusCode == 201) {
      return Product.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to create product: ${response.statusCode}');
    }
  }
}
```

#### ViewModel

After implementing the service, that's where the ViewModel comes in. The ViewModel is responsible for managing the state of the view and the business logic related to the view.

```dart
// lib/viewmodels/product_list_viewmodel.dart
class ProductListViewModel extends ChangeNotifier {
  final ProductService _productService;

  ProductListViewModel({required ProductService productService})
    : _productService = productService;

  List<Product> _products = [];
  bool _isLoading = false;
  String? _error;

  List<Product> get products => _products;
  bool get isLoading => _isLoading;
  String? get error => _error;

  void loadProducts() async {
    _setLoading(true);
    _setError(null);

    try {
      _products = await _productService.getProducts();
      notifyListeners();
    } catch (e, s) {
      debugPrint("$e $s");
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }

  void _setLoading(bool value) {
    _isLoading = value;
    notifyListeners();
  }

  void _setError(String? value) {
    _error = value;
    notifyListeners();
  }
}
```

#### View

Now, we will create a view to display the product list which reflects the state of the ViewModel. In this example, we will use the `provider` package to manage the state of the view.

```dart
// lib/views/pages/product_list_page.dart
class ProductListPage extends StatefulWidget {
  const ProductListPage({super.key});

  @override
  State<ProductListPage> createState() => _ProductListPageState();
}

class _ProductListPageState extends State<ProductListPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(appBar: _buildAppBar(), body: _buildBody());
  }

  AppBar _buildAppBar() {
    return AppBar(title: const Text('Products'));
  }

  Widget _buildBody() {
    final isLoading = context.select<ProductListViewModel, bool>(
      (viewModel) => viewModel.isLoading,
    );
    final productsCount = context.select<ProductListViewModel, int>(
      (viewModel) => viewModel.products.length,
    );
    return isLoading && productsCount == 0
        ? _buildLoadingIndicator()
        : _buildContent();
  }

  Widget _buildLoadingIndicator() {
    return const Center(child: CircularProgressIndicator());
  }

  Widget _buildContent() {
    final error = context.select<ProductListViewModel, String?>(
      (viewModel) => viewModel.error,
    );

    return error != null ? _buildErrorIndicator(error) : _buildProductGrids();
  }

  Widget _buildErrorIndicator(String error) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32.0),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(Icons.error, size: 48),
            const SizedBox(height: 8),
            Text("Error", style: Theme.of(context).textTheme.titleLarge),
            const SizedBox(height: 8),
            Text(error, textAlign: TextAlign.center),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: _reloadProducts,
              child: const Text('Retry'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildProductGrids() {
    final products = context.select<ProductListViewModel, List<Product>>(
      (viewModel) => viewModel.products,
    );

    return RefreshIndicator(
      onRefresh: _reloadProducts,
      child: CustomScrollView(
        slivers: [
          SliverPadding(
            padding: const EdgeInsets.all(16.0),
            sliver: SliverMasonryGrid.count(
              crossAxisCount: 2,
              mainAxisSpacing: 16,
              crossAxisSpacing: 16,
              childCount: products.length,
              itemBuilder: (context, index) {
                return ProductCard(product: products[index]);
              },
            ),
          ),
        ],
      ),
    );
  }

  Future<void> _reloadProducts() async {
    context.read<ProductListViewModel>().loadProducts();
    return Future.delayed(const Duration(seconds: 2));
  }
}
```

After implementing all of the required components, we can now put it all together in the main function.

```dart
// lib/main.dart
return MaterialApp(
  title: 'MVVM Example',
  theme: ThemeData.light(useMaterial3: true),
  home: MultiProvider(
    providers: [
      Provider(create: (context) => ProductService()),
      ChangeNotifierProvider<ProductListViewModel>(
        create: (context) => ProductListViewModel(
          productService: context.read<ProductService>(),
        )..loadProducts(),
      ),
    ],
    child: const ProductListPage(),
  ),
);
```

## 8.2 Clean Architecture

Clean Architecture, is a software design philosophy that emphasizes separation of concerns and dependency inversion. In Flutter applications, Clean Architecture helps organize code into distinct layers with clear responsibilities.

### 8.2.1 Core Principles

1. **Independence of Frameworks**: The architecture doesn't depend on the existence of Flutter widgets or external libraries
2. **Testability**: Business rules can be tested without UI, databases, or external dependencies
3. **Independence of UI**: The UI can change without affecting business logic
4. **Independence of Database**: Business rules are not bound to any specific database

### 8.2.2 Layers

Clean Architecture in Flutter typically consists of three main layers:

#### Data Layer

- Responsible for data retrieval and storage
- Contains repositories, data sources, and models
- Handles API calls and local storage operations

#### Domain Layer

- Contains business logic and rules
- Defines entities, use cases, and repository contracts
- Independent of external frameworks

#### Presentation Layer

- Manages UI components and user interactions
- Contains widgets, pages, and presentation logic
- Depends on the domain layer

### 8.2.3 Folder Structure Example

```
lib/
├── core/
│   ├── error/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── usecases/
│   │   └── usecase.dart
│   └── utils/
│       └── constants.dart
├── features/
│   └── recipes/
│       ├── data/
│       │   ├── datasources/
│       │   │   ├── recipe_local_data_source.dart
│       │   │   └── recipe_remote_data_source.dart
│       │   └── repositories/
│       │       └── recipe_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── recipe.dart
│       │   ├── repositories/
│       │   │   └── recipe_repository.dart
│       │   └── usecases/
│       │       ├── get_recipes.dart
│       │       └── create_recipe.dart
│       └── presentation/
│           ├── pages/
│           │   └── recipe_list_page.dart
│           └── widgets/
│               └── recipe_card.dart
└── main.dart
```

### 8.2.4 Domain Layer Implementation

#### Entity

```dart
// lib/features/recipes/domain/entities/recipe.dart
class Recipe {
  final int id;

  final String name;
  final List<String> ingredients;
  final List<String> instructions;
  final int prepTimeMinutes;
  final int cookTimeMinutes;
  final int caloriesPerServing;
  // ...

  Recipe({
    required this.id,
    required this.name,
    required this.ingredients,
    required this.instructions,
    required this.prepTimeMinutes,
    required this.cookTimeMinutes,
    required this.caloriesPerServing,
    // ...
  });

  factory Recipe.fromJson(Map<String, dynamic> json) => _$RecipeFromJson(json);

  Map<String, dynamic> toJson() => _$RecipeToJson(this);

  bool get isHealthy => caloriesPerServing < 300;
  bool get isQuick => prepTimeMinutes + cookTimeMinutes < 30;
}
```

#### Repository Interface

```dart
// lib/features/recipes/domain/repositories/recipe_repository.dart
abstract class RecipeRepository {
  Future<List<Recipe>> getRecipes();
  Future<Recipe> getRecipeById(int id);
  Future<Recipe> createRecipe(Recipe recipe);
  Future<Recipe> updateRecipe(Recipe recipe);
  Future<void> deleteRecipe(int id);
}
```

#### Use Case

```dart
// lib/features/recipes/domain/usecases/get_recipes.dart
class GetRecipesUseCase {
  final RecipeRepository recipeRepository;

  GetRecipesUseCase(this.recipeRepository);

  Future<List<Recipe>> call() async {
    return await postRepository.getRecipes();
  }
}
```

#### Data Layer Implementation

#### Repository Implementation

```dart
// lib/features/recipes/data/repositories/recipe_repository_impl.dart
class RecipeRepositoryImpl extends RecipeRepository {
  final RecipeRemoteDataSource remoteDataSource;

  RecipeRepositoryImpl({required this.remoteDataSource});

  @override
  Future<Recipe> createRecipe(Recipe recipe) async {
    return await remoteDataSource.createRecipe(recipe);
  }

  @override
  Future<List<Recipe>> getRecipes() async {
    return await remoteDataSource.getRecipes();
  }

  // ...
}
```

#### Data Source

```dart
// lib/features/recipes/data/datasources/recipe_remote_data_source.dart
abstract class RecipeRemoteDataSource {
  Future<List<Recipe>> getRecipes();
  Future<Recipe> createRecipe(Recipe recipe);
  // ...
}

class RecipeRemoteDataSourceImpl implements RecipeRemoteDataSource {
  final http.Client _client;

  RecipeRemoteDataSourceImpl({required http.Client client}) : _client = client;

  @override
  Future<Recipe> createRecipe(Recipe recipe) async {
    final response = await _client.post(
      Uri.parse('$baseUrl/recipes'),
      body: jsonEncode(recipe.toJson()),
    );
    if (response.statusCode == 201) {
      return Recipe.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to create Recipe: ${response.statusCode}');
    }
  }

  @override
  Future<List<Recipe>> getRecipes() async {
    final response = await _client.get(Uri.parse('$baseUrl/recipes'));
    if (response.statusCode == 200) {
      final body = jsonDecode(response.body);
      return (body['recipes'] as List)
          .map((e) => Recipe.fromJson(e))
          .toList();
    } else {
      throw Exception('Failed to get Recipes: ${response.statusCode}');
    }
  }
}
```

### 8.2.5 Presentation Layer Implementation

#### State Management

```dart
// lib/features/recipes/presentation/providers/recipe_list.dart
class RecipeList extends ChangeNotifier {
  final GerRecipesUseCase getRecipesUseCase;

  RecipeList({required this.getRecipesUseCase});

  List<Recipe> _recipes = [];
  bool _isLoading = false;
  String? _error;

  List<Recipe> get recipes => _recipes;
  bool get isLoading => _isLoading;
  String? get error => _error;

  void loadRecipes() async {
    _setLoading(true);
    _setError(null);

    try {
      _recipes = await getRecipesUseCase.call();
      notifyListeners();
    } catch (e, s) {
      debugPrint("$e $s");
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }

  void _setLoading(bool value) {
    _isLoading = value;
    notifyListeners();
  }

  void _setError(String? value) {
    _error = value;
    notifyListeners();
  }
}
```

#### View

```dart
// lib/features/recipes/presentation/pages/recipes_page.dart
class RecipesPage extends StatefulWidget {
  const RecipesPage({super.key});

  @override
  State<RecipesPage> createState() => _RecipesPageState();
}

class _RecipesPageState extends State<RecipesPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(appBar: _buildAppBar(), body: _buildBody());
  }

  AppBar _buildAppBar() {
    return AppBar(title: const Text('Recipes'));
  }

  Widget _buildBody() {
    final isLoading = context.select<RecipeList, bool>(
      (viewModel) => viewModel.isLoading,
    );
    final recipeCount = context.select<RecipeList, int>(
      (viewModel) => viewModel.recipes.length,
    );
    return isLoading && recipeCount == 0
        ? _buildLoadingIndicator()
        : _buildContent();
  }

  Widget _buildLoadingIndicator() {
    return const Center(child: CircularProgressIndicator());
  }

  Widget _buildContent() {
    final error = context.select<RecipeList, String?>(
      (viewModel) => viewModel.error,
    );

    return error != null ? _buildErrorIndicator(error) : _buildRecipeList();
  }

  Widget _buildErrorIndicator(String error) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32.0),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(Icons.error, size: 48),
            const SizedBox(height: 8),
            Text("Error", style: Theme.of(context).textTheme.titleLarge),
            const SizedBox(height: 8),
            Text(error, textAlign: TextAlign.center),
            const SizedBox(height: 16),
            ElevatedButton(onPressed: _reload, child: const Text('Retry')),
          ],
        ),
      ),
    );
  }

  Widget _buildRecipeList() {
    final recipes = context.select<RecipeList, List<Recipe>>(
      (viewModel) => viewModel.recipes,
    );

    return RefreshIndicator(
      onRefresh: _reload,
      child: ListView.separated(
        padding: const EdgeInsets.all(16),
        itemCount: recipes.length,

        itemBuilder: (context, index) {
          return RecipeListItem(recipe: recipes[index]);
        },
        separatorBuilder: (context, index) => const SizedBox(height: 8),
      ),
    );
  }

  Future<void> _reload() async {
    context.read<RecipeList>().loadRecipes();
    return Future.delayed(const Duration(seconds: 2));
  }
}
```

## 8.3 Architecture Comparison

### Clean Architecture

**Advantages:**

- High testability and maintainability
- Clear separation of concerns
- Framework independence
- Suitable for complex, large-scale applications

**Disadvantages:**

- Higher initial complexity
- More boilerplate code
- Steeper learning curve

### MVVM

**Advantages:**

- Simpler to implement and understand
- Good separation between UI and business logic
- Suitable for medium-scale applications
- Less boilerplate compared to Clean Architecture

**Disadvantages:**

- Less framework independence
- Can become complex with bidirectional data binding
- ViewModels may become bloated in large applications

## 8.4 Choosing the Right Architecture

Consider **Clean Architecture** when:

- Building large, complex applications
- Multiple developers are working on the project
- High testability requirements
- Long-term maintenance is crucial

Consider **MVVM** when:

- Building medium-scale applications
- Quick development is prioritized
- The team has limited experience with complex architectures
- Simple separation of UI and business logic is sufficient

Both architectures can be combined with state management solutions to handle complex state requirements effectively.

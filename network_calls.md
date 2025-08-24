# 11. Consuming Data

## Overview

In this section, we will cover the following topics:

- HTTP Client with Dio
- GraphQL Client

## Github

https://github.com/kywsta/flutter_network_client_sample

## 11.1 HTTP Client with Dio

### Introduction to Dio

Dio is a powerful HTTP client for Dart that provides interceptors, request cancellation, file downloading, timeout handling, and many other advanced features. It is preferred over the basic `http` package for production applications due to its robust feature set.

### Installation

Add the following dependencies to your `pubspec.yaml`:

```yaml
dependencies:
  dio: ^5.4.0
  # Optional: For logging requests and responses
  pretty_dio_logger: ^1.3.1
```

### Basic Dio Setup

Create a singleton service class to store the configured Dio instance:

```dart
class DioService {
  static final DioService _instance = DioService._internal();
  factory DioService() => _instance;
  DioService._internal();

  late final Dio _dio;

  Dio get dio => _dio;

  void initialize() {
    _dio = Dio(
      BaseOptions(
        baseUrl: 'https://dummyjson.com',
        connectTimeout: const Duration(seconds: 30),
        receiveTimeout: const Duration(seconds: 30),
      ),
    );

    // Add logging interceptor for development
    _dio.interceptors.add(
      PrettyDioLogger(
        requestHeader: true,
        requestBody: true,
        responseBody: true,
        responseHeader: false,
        compact: false,
      ),
    );

    _dio.interceptors.add(AuthInterceptor());
  }
}
```

### Authentication Token Interceptor

Implement an interceptor to automatically add authentication tokens to requests and
handle token expiration and refresh.

```dart
class AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    // Get token from secure storage or shared preferences
    final token = await _getAuthToken();

    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }

    super.onRequest(options, handler);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    // Handle token refresh on 401 errors
    if (err.response?.statusCode == 401) {
      try {
        final newToken = await _refreshToken();
        if (newToken != null) {
          // Retry the original request with new token
          final requestOptions = err.requestOptions;
          requestOptions.headers['Authorization'] = 'Bearer $newToken';

          final response = await ApiService().dio.fetch(requestOptions);
          return handler.resolve(response);
        }
      } catch (e) {
        // Redirect to login if refresh fails
        _redirectToLogin();
      }
    }

    super.onError(err, handler);
  }

  Future<String?> _getAuthToken() async {
    // Implementation depends on your storage solution
    // Example: return await SecureStorage.read('auth_token');
    return null;
  }

  Future<String?> _refreshToken() async {
    // Implement token refresh logic
    return null;
  }

  void _redirectToLogin() {
    // Implement navigation to login screen
  }
}
```

### API Service Implementation

```dart
class RecipeService {
  final DioService _dioService = DioService();

  Future<List<Recipe>> getRecipes({int limit = 30, int skip = 0}) async {
    try {
      final response = await _dioService.dio.get(
        '/recipes',
        queryParameters: {'limit': limit, 'skip': skip},
      );

      if (response.statusCode == 200) {
        final List<dynamic> recipesJson = response.data['recipes'];
        return recipesJson.map((json) => Recipe.fromJson(json)).toList();
      } else {
        throw ApiException('Failed to fetch recipes: ${response.statusCode}');
      }
    } on DioException catch (e) {
      throw _handleDioError(e);
    } catch (e) {
      throw ApiException('Unexpected error: $e');
    }
  }

  Future<Recipe> getRecipeById(int id) async {
    try {
      final response = await _dioService.dio.get('/recipes/$id');

      if (response.statusCode == 200) {
        return Recipe.fromJson(response.data);
      } else {
        throw ApiException('Recipe not found');
      }
    } on DioException catch (e) {
      throw _handleDioError(e);
    }
  }

  ApiException _handleDioError(DioException error) {
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.sendTimeout:
      case DioExceptionType.receiveTimeout:
        return ApiException('Connection timeout');
      case DioExceptionType.badResponse:
        return ApiException('Server error: ${error.response?.statusCode}');
      case DioExceptionType.cancel:
        return ApiException('Request cancelled');
      case DioExceptionType.connectionError:
        return ApiException('No internet connection');
      default:
        return ApiException('Unknown error occurred');
    }
  }
}

class ApiException implements Exception {
  final String message;
  ApiException(this.message);

  @override
  String toString() => 'ApiException: $message';
}
```

### Usage Example

Implement the service in a widget:

```dart
class RecipeListScreen extends StatefulWidget {
  const RecipeListScreen({super.key});

  @override
  State<RecipeListScreen> createState() => _RecipeListScreenState();
}

class _RecipeListScreenState extends State<RecipeListScreen> {
  final RecipeService _recipeService = RecipeService();
  List<Recipe> _recipes = [];
  bool _isLoading = false;
  String? _error;

  @override
  void initState() {
    super.initState();
    _loadRecipes();
  }

  Future<void> _loadRecipes() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });

    try {
      final recipes = await _recipeService.getRecipes();
      setState(() {
        _recipes = recipes;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }

    if (_error != null) {
      return Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Error: $_error'),
              ElevatedButton(
                onPressed: _loadRecipes,
                child: const Text('Retry'),
              ),
            ],
          ),
        ),
      );
    }

    return Scaffold(
      appBar: AppBar(title: const Text('Recipes')),
      body: ListView.builder(
        itemCount: _recipes.length,
        itemBuilder: (context, index) {
          final recipe = _recipes[index];
          return ListTile(
            leading: Image.network(recipe.image, width: 50, height: 50),
            title: Text(recipe.name),
            subtitle: Text('${recipe.cuisine} • ${recipe.difficulty}'),
            trailing: Text('⭐ ${recipe.rating}'),
          );
        },
      ),
    );
  }
}
```

## 11.2 GraphQL Client

### Introduction to GraphQL

GraphQL is a query language and runtime for APIs that allows clients to request exactly the data they need. It provides a more efficient alternative to REST APIs by enabling single requests to fetch multiple related resources.

### Installation

Add the required dependencies to your `pubspec.yaml`:

```yaml
dependencies:
  graphql: ^5.2.1
  graphql_flutter: ^5.2.0

dev_dependencies:
  graphql_codegen: ^2.0.0
  build_runner: ^2.4.7
```

### GraphQL Client Setup

Create a GraphQL client configuration:

```dart
import 'package:graphql_flutter/graphql_flutter.dart';

class GraphQLService {
  static final GraphQLService _instance = GraphQLService._internal();
  factory GraphQLService() => _instance;
  GraphQLService._internal();

  late GraphQLClient _client;

  void initialize() {
    final HttpLink httpLink = HttpLink(
      'https://swapi-graphql.netlify.app/graphql'
    );

    final AuthLink authLink = AuthLink(
      getToken: () async {
        // Return your authentication token
        return 'Bearer YOUR_TOKEN_HERE';
      },
    );

    final Link link = authLink.concat(httpLink);

    _client = GraphQLClient(
      link: link,
      cache: GraphQLCache(store: InMemoryStore()),
    );
  }

  GraphQLClient get client => _client;
}
```

### GraphQL Code Generation Setup

Create a `build.yaml` file in your project root:

```yaml
targets:
  $default:
    builders:
      graphql_codegen:
        options:
          clients:
            - graphql
          addTypename: true
          addTypenameExcludedPaths: 
            - subscription
```

Copy or download the schema file from `https://studio.apollographql.com/public/star-wars-swapi/variant/current/schema/sdl` and save it as `lib/graphql/schema.graphql`

After we have the schema file, we can create a GraphQL query file `lib/graphql/queries.graphql`:

```graphql
query GetAllFilms {
  allFilms {
    films {
      id
      title
      director
      releaseDate
      episodeID
      openingCrawl
    }
  }
}

query GetFilmById($id: ID!) {
  film(id: $id) {
    id
    title
    director
    releaseDate
    episodeID
    openingCrawl
    characterConnection {
      characters {
        name
        birthYear
        eyeColor
        gender
      }
    }
    planetConnection {
      planets {
        name
        population
        climates
      }
    }
  }
}

query GetAllPeople($first: Int, $after: String) {
  allPeople(first: $first, after: $after) {
    edges {
      node {
        id
        name
        birthYear
        eyeColor
        gender
        height
        mass
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

Run code generation to generate the queries:

```bash
dart run build_runner build -d
```

### GraphQL Service Implementation

Create a service class using the generated code:

```dart
class StarWarsService {
  final GraphQLService _graphQLService = GraphQLService();

  Future<Query$GetAllFilms> getAllFilms() async {
    final options = Options$Query$GetAllFilms();

    final result = await _graphQLService.client.query$GetAllFilms(options);

    if (result.hasException) {
      throw GraphQLException(result.exception.toString());
    }

    return result.parsedData!;
  }

  Future<Query$GetFilmById> getFilmById(String id) async {
    final options = Options$Query$GetFilmById(
      variables: Variables$Query$GetFilmById(id: id),
    );

    final result = await _graphQLService.client.query$GetFilmById(options);

    if (result.hasException) {
      throw GraphQLException(result.exception.toString());
    }

    return result.parsedData!;
  }

  Future<Query$GetAllPeople> getAllPeople({
    int first = 10,
    String? after,
  }) async {
    final options = Options$Query$GetAllPeople(
      variables: Variables$Query$GetAllPeople(first: first, after: after),
    );

    final result = await _graphQLService.client.query$GetAllPeople(options);

    if (result.hasException) {
      throw GraphQLException(result.exception.toString());
    }

    return result.parsedData!;
  }
}

class GraphQLException implements Exception {
  final String message;
  GraphQLException(this.message);

  @override
  String toString() => 'GraphQLException: $message';
}
```

### GraphQL Widget Implementation

Use the GraphQL service with proper state management:

```dart
class FilmsScreen extends StatefulWidget {
  const FilmsScreen({super.key});

  @override
  State<FilmsScreen> createState() => _FilmsScreenState();
}

class _FilmsScreenState extends State<FilmsScreen> {
  final StarWarsService _starWarsService = StarWarsService();
  List<Query$GetAllFilms$allFilms$films> _films = [];
  bool _isLoading = false;
  String? _error;

  @override
  void initState() {
    super.initState();
    _loadFilms();
  }

  Future<void> _loadFilms() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });

    try {
      final result = await _starWarsService.getAllFilms();
      setState(() {
        _films = result.allFilms!.films!.nonNulls.toList();
        _isLoading = false;
      });
    } catch (e) {
      FlutterError.reportError(FlutterErrorDetails(
        exception: e,
        stack: StackTrace.current,
      ));
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Star Wars Films'),
        actions: [
          IconButton(icon: const Icon(Icons.refresh), onPressed: _loadFilms),
        ],
      ),
      body: _buildBody(),
    );
  }

  Widget _buildBody() {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_error != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error, size: 64, color: Colors.red),
            const SizedBox(height: 16),
            Text(
              'Error: $_error',
              textAlign: TextAlign.center,
              style: const TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 16),
            ElevatedButton(onPressed: _loadFilms, child: const Text('Retry')),
          ],
        ),
      );
    }

    return ListView.builder(
      itemCount: _films.length,
      itemBuilder: (context, index) {
        final film = _films[index];
        return Card(
          margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
          child: ListTile(
            title: Text(
              film.title ?? 'Unknown Title',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
            subtitle: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Director: ${film.director ?? 'Unknown'}'),
                Text('Episode: ${film.episodeID ?? 'Unknown'}'),
                Text('Release Date: ${film.releaseDate ?? 'Unknown'}'),
              ],
            ),
            onTap: () => _navigateToFilmDetail(film.id!),
          ),
        );
      },
    );
  }

  void _navigateToFilmDetail(String filmId) {
    Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => FilmDetailScreen(filmId: filmId)),
    );
  }
}
```

### Advanced GraphQL Features

#### Query with Variables Example

```dart
class FilmDetailScreen extends StatefulWidget {
  final String filmId;

  const FilmDetailScreen({super.key, required this.filmId});

  @override
  State<FilmDetailScreen> createState() => _FilmDetailScreenState();
}

class _FilmDetailScreenState extends State<FilmDetailScreen> {
  final StarWarsService _starWarsService = StarWarsService();
  Query$GetFilmById$film? _film;
  bool _isLoading = false;
  String? _error;

  @override
  void initState() {
    super.initState();
    _loadFilm();
  }

  Future<void> _loadFilm() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });

    try {
      final result = await _starWarsService.getFilmById(widget.filmId);
      setState(() {
        _film = result.film;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_film?.title ?? 'Loading...'),
      ),
      body: _buildBody(),
    );
  }

  Widget _buildBody() {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_error != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Error: $_error'),
            ElevatedButton(
              onPressed: _loadFilm,
              child: const Text('Retry'),
            ),
          ],
        ),
      );
    }

    if (_film == null) {
      return const Center(child: Text('Film not found'));
    }

    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            _film!.title ?? 'Unknown Title',
            style: Theme.of(context).textTheme.headlineMedium,
          ),
          const SizedBox(height: 8),
          Text('Director: ${_film!.director}'),
          Text('Episode: ${_film!.episodeID}'),
          Text('Release Date: ${_film!.releaseDate}'),
          const SizedBox(height: 16),
          const Text(
            'Opening Crawl:',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Text(_film!.openingCrawl ?? ''),
        ],
      ),
    );
  }
}
```
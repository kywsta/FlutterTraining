# Data Persistent

Flutter provides several options for local data storage, each suited for different use cases and requirements.

## Github

https://github.com/kywsta/flutter_local_storage_examples.git

https://github.com/kywsta/flutter_objectbox_sample.git

## Shared Preferences

Shared Preferences is the simplest form of local storage in Flutter, designed for storing small pieces of primitive data such as user settings, preferences, and simple configuration values.

### Basic Usage

First, add the dependency to your `pubspec.yaml`:

```yaml
dependencies:
  shared_preferences: ^2.2.2
```

and create a service to interact with the shared preferences

```dart
import 'package:shared_preferences/shared_preferences.dart';

class PreferencesService {
  static SharedPreferences? _preferences;

  // Initialize SharedPreferences
  static Future<void> init() async {
    _preferences = await SharedPreferences.getInstance();
  }

  // Save data
  static Future<void> saveString(String key, String value) async {
    await _preferences?.setString(key, value);
  }

  static Future<void> saveInt(String key, int value) async {
    await _preferences?.setInt(key, value);
  }

  static Future<void> saveBool(String key, bool value) async {
    await _preferences?.setBool(key, value);
  }

  // Retrieve data
  static String? getString(String key) {
    return _preferences?.getString(key);
  }

  static int? getInt(String key) {
    return _preferences?.getInt(key);
  }

  static bool? getBool(String key) {
    return _preferences?.getBool(key);
  }

  // Remove data
  static Future<void> remove(String key) async {
    await _preferences?.remove(key);
  }

  // Clear all data
  static Future<void> clear() async {
    await _preferences?.clear();
  }
}
```

### Usage in Widget

```dart
class SettingsScreen extends StatefulWidget {
  const SettingsScreen({super.key});

  @override
  State<SettingsScreen> createState() => _SettingsScreenState();
}

class _SettingsScreenState extends State<SettingsScreen> {
  bool _isDarkMode = false;

  @override
  void initState() {
    super.initState();
    _loadPreferences();
  }

  void _loadPreferences() {
    setState(() {
      _isDarkMode = PreferencesService.getBool('isDarkMode') ?? false;
    });
    debugPrint('isDarkMode: $_isDarkMode');
  }

  void _savePreferences() {
    debugPrint('savePreferences: isDarkMode: $_isDarkMode');
    PreferencesService.saveBool('isDarkMode', _isDarkMode);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Settings')),
      body: Column(
        children: [
          SwitchListTile(
            title: Text('Dark Mode'),
            value: _isDarkMode,
            onChanged: (value) {
              setState(() {
                _isDarkMode = value;
              });
              _savePreferences();
            },
          ),
          Expanded(
            child: Center(
              child: CardTheme(
                data: CardThemeData(
                  color: _isDarkMode
                      ? ThemeData.dark().colorScheme.surfaceContainer
                      : ThemeData.light().colorScheme.surfaceContainer,
                ),
                child: Card(
                  child: Padding(
                    padding: const EdgeInsets.all(16.0),
                    child: Text(
                      _isDarkMode ? "Dark Mode" : "Light Mode",
                      style: TextStyle(
                        color: _isDarkMode
                            ? ThemeData.dark().colorScheme.onSurface
                            : ThemeData.light().colorScheme.onSurface,
                      ),
                    ),
                  ),
                ),
              ),
            ),
          ),
          // Additional settings widgets...
        ],
      ),
    );
  }
}
```

## 4.2 Secure Storage

For sensitive data such as authentication tokens, passwords, or personal information, Flutter provides secure storage options that encrypt data before storing it locally.

**Note:**

- Keychain is used for iOS
- AES encryption is used for Android. AES secret key is encrypted with RSA and RSA key is stored in KeyStore

### Flutter Secure Storage

Add the dependency:

```yaml
dependencies:
  flutter_secure_storage: latest
```

### Code Example

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureStorageService {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
    ),
    iOptions: IOSOptions(
      accessibility: IOSAccessibility.first_unlock_this_device,
    ),
  );

  // Save secure data
  static Future<void> saveSecureData(String key, String value) async {
    await _storage.write(key: key, value: value);
  }

  // Retrieve secure data
  static Future<String?> getSecureData(String key) async {
    return await _storage.read(key: key);
  }

  // Delete secure data
  static Future<void> deleteSecureData(String key) async {
    await _storage.delete(key: key);
  }

  // Clear all secure data
  static Future<void> clearAll() async {
    await _storage.deleteAll();
  }

  // Check if key exists
  static Future<bool> containsKey(String key) async {
    return await _storage.containsKey(key: key);
  }
}
```

```dart
class TokenStorage {
  static const String _tokenKey = 'auth_token';

  // Save authentication tokens
  static Future<void> saveTokens(String token) async {
    await SecureStorageService.saveSecureData(_tokenKey, token);
  }

  // Get authentication token
  static Future<String?> getToken() async {
    return await SecureStorageService.getSecureData(_tokenKey);
  }

  // Logout (clear tokens)
  static Future<void> clear() async {
    await SecureStorageService.deleteSecureData(_tokenKey);
  }
}
```

```dart
class TokenStorageScreen extends StatefulWidget {
  const TokenStorageScreen({super.key});

  @override
  State<TokenStorageScreen> createState() => _TokenStorageScreenState();
}

class _TokenStorageScreenState extends State<TokenStorageScreen> {
  final _tokenController = TextEditingController();

  String? _token;

  @override
  void initState() {
    super.initState();
    _loadToken();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Secure Storage Sample')),
      body: SafeArea(
        child: Column(
          children: [
            Expanded(child: Center(child: Text('Token: $_token'))),
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: TextField(
                controller: _tokenController,
                decoration: InputDecoration(
                  border: OutlineInputBorder(),
                  label: Text("Enter text to save"),
                ),
              ),
            ),
            Column(
              children: [
                TextButton(
                  onPressed: _saveToken,
                  child: const Text('Save Token'),
                ),
                TextButton(
                  onPressed: _loadToken,
                  child: const Text('Load Token'),
                ),
                TextButton(
                  onPressed: _clearToken,
                  child: const Text('Clear Token'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  void _saveToken() async {
    await TokenStorage.saveTokens(_tokenController.text);
    setState(() {
      _token = "Token changed, please reload";
    });
  }

  void _loadToken() async {
    final token = await TokenStorage.getToken();
    setState(() {
      _token = token;
    });
  }

  void _clearToken() async {
    await TokenStorage.clear();
    setState(() {
      _token = null;
    });
  }
}
```

## 4.3 Local Database Options

For more complex data storage requirements, Flutter supports several local database solutions. Each option has its own strengths and is suitable for different use cases.

### Database Comparison Table

| Name             | Type       | Language | Supported Platforms                      | Package   |
| ---------------- | ---------- | -------- | ---------------------------------------- | --------- |
| SQLite (sqflite) | Relational | SQL/Dart | Android, iOS, macOS                      | sqflite   |
| Drift            | Relational | SQL/Dart | Android, iOS, Web, Desktop, macOS, Linux | drift     |
| Hive CE          | NoSQL      | Dart     | Android, iOS, Web, Desktop, macOS, Linux | hive_ce   |
| ObjectBox        | NoSQL      | Dart     | Android, iOS, macOS, Linux, Windows      | objectbox |
| Realm            | NoSQL      | Dart     | Android, iOS, macOS, Linux, Windows      | realm     |

### 4.3.1 SQLite (sqflite)

SQLite is a lightweight, relational database that's widely used in mobile applications. The `sqflite` package provides Flutter integration for SQLite.

#### Dependencies

```yaml
dependencies:
  sqflite: latest
  path: latest
```

#### Code Example

```dart
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

class SqfliteDatabaseHelper {
  static Database? _database;
  static const String _tableName = 'notes';

  // Singleton pattern
  static Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  static Future<Database> _initDatabase() async {
    String path = join(await getDatabasesPath(), 'notes.db');
    return await openDatabase(path, version: 1, onCreate: _createTable);
  }

  static Future<void> _createTable(Database db, int version) async {
    await db.execute('''
      CREATE TABLE $_tableName(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        content TEXT NOT NULL,
        createdAt TEXT NOT NULL
      )
    ''');
  }

  // CRUD Operations
  static Future<int> insertNote(Map<String, dynamic> note) async {
    final db = await database;
    return await db.insert(_tableName, note);
  }

  static Future<List<Map<String, dynamic>>> getAllNotes() async {
    final db = await database;
    return await db.query(_tableName, orderBy: 'createdAt DESC');
  }

  static Future<int> updateNote(int id, Map<String, dynamic> note) async {
    final db = await database;
    return await db.update(_tableName, note, where: 'id = ?', whereArgs: [id]);
  }

  static Future<int> deleteNote(int id) async {
    final db = await database;
    return await db.delete(_tableName, where: 'id = ?', whereArgs: [id]);
  }
}
```

#### Usage

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  SqfliteDatabaseHelper.insertNote({
    'title': 'Sample Note',
    'content': 'This is a sample note',
    'createdAt': DateTime.now().toIso8601String(),
  });

  List<Map<String, dynamic>> allNotes =
      await SqfliteDatabaseHelper.getAllNotes();
  print(allNotes);
}
```

#### Pros and Cons

**Pros:**

- Support transactions and batches
- Automatic version managment during open
- Helpers for insert/query/update/delete queries
- DB operation executed in a background thread on iOS and Android

**Cons:**

- Requires SQL knowledge
- More verbose code compared to NoSQL alternatives
- Need more work to make type safe queries
- No native support for web, linux, windows

### 4.3.2 Drift

Drift (formerly Moor) is a reactive persistence library built on top of SQLite, providing type-safe database access with Dart code generation.

#### Dependencies

```yaml
dependencies:
  drift: latest
  drift_flutter: latest
  path_provider: latest

dev_dependencies:
  drift_dev: latest
  build_runner: latest
```

#### Code Example

```dart
// database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:path_provider/path_provider.dart';

part 'drift_database.g.dart';

class TodoItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 6, max: 32)();
  TextColumn get content => text().named('body')();
  DateTimeColumn get createdAt => dateTime().nullable()();
}

@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  // After generating code, this class needs to define a `schemaVersion` getter
  // and a constructor telling drift where the database should be stored.
  // These are described in the getting started guide: https://drift.simonbinder.eu/setup/
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'my_database',
      native: const DriftNativeOptions(
        // By default, `driftDatabase` from `package:drift_flutter` stores the
        // database files in `getApplicationDocumentsDirectory()`.
        databaseDirectory: getApplicationSupportDirectory,
      ),
      // If you need web support, see https://drift.simonbinder.eu/platforms/web/
    );
  }
}
```

and run build_runner to generate the database class

```bash
dart run build_runner build -d
```

#### Usage

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final database = AppDatabase();

  await database
      .into(database.todoItems)
      .insert(
        TodoItemsCompanion.insert(
          title: 'todo: finish drift setup',
          content: 'We can now write queries and define our own tables.',
        ),
      );

  List<TodoItem> allItems = await database.select(database.todoItems).get();

  print('items in database: $allItems');

  List<TodoItem> item = await (database.select(database.todoItems)..where((table) => table.id.equals(2))).get();
  print('item with id 2: $item');
}
```

#### Pros and Cons

**Pros:**

- Type-safe database access with code generation
- Query using both SQL and Dart
- Reactive streams for real-time updates
- Cross-platform support including web
- Built-in threading support

**Cons:**

- Steeper learning curve because of both SQL and Dart
- Still requires SQL knowledge for complex queries
- Complex migration steps

### 4.3.3 Hive CE (Community Edition of Hive)

Hive CE is a lightweight, NoSQL database written in pure Dart, designed for Flutter applications with excellent performance characteristics.

#### Dependencies

```yaml
dependencies:
  hive_ce: latest
  hive_ce_flutter: latest

dev_dependencies:
  hive_ce_generator: latest
  build_runner: latest
```

#### Code Example

```dart
import 'package:hive_ce/hive.dart';

class Note extends HiveObject {
  String id;
  String title;
  String content;
  DateTime createdAt;

  Note({
    required this.id,
    required this.title,
    required this.content,
    required this.createdAt,
  });
}
```

and create a file for adapter generation

```dart
import 'package:hive_ce/hive.dart';

@GenerateAdapters([AdapterSpec<Note>()])
part 'hive_adapters.g.dart';
```

and run build_runner to generate the adapters

```bash
dart run build_runner build -d
```

#### Usage in Widget

```dart
void main() async {
  // 1. Initialize Hive
  await Hive.initFlutter();

  // 2. Register adapters
  Hive.registerAdapters();

  // 3. Open box
  final box = await Hive.openBox<Note>('notes');

  await box.add(Note(
    id: (box.length + 1).toString(),
    title: 'Sample Note',
    content: 'This is a sample note',
    createdAt: DateTime.now(),
  ));

  List<Note> allNotes = box.values.toList();
  print('all notes: $allNotes');
}
```

#### Pros and Cons

**Pros:**

- Pure Dart implementation (no native dependencies)
- Excellent performance and low memory footprint
- Simple and fast to setup
- Support for all platforms
- Built-in encryption support
- Isolation support

**Cons:**

- NoSQL limitations for complex relational queries
- Manual data modeling required
- Limited querying capabilities compared to SQL
- No support for transactions
- No reactive stream support

### 4.3.4 ObjectBox

ObjectBox is a high-performance NoSQL database designed for mobile and IoT applications, offering fast object persistence with minimal setup.

#### Dependencies

```yaml
dependencies:
  objectbox: latest
  objectbox_flutter_libs: latest
  path_provider: latest
  path: latest

dev_dependencies:
  objectbox_generator: latest
  build_runner: latest
```

#### Code Example

```dart
// note_model.dart
import 'package:objectbox/objectbox.dart';

@Entity()
class NoteEntity {
  @Id()
  int id = 0;
  final String title;
  final String content;
  final DateTime createdAt;

  NoteEntity({
    required this.title,
    required this.content,
    required this.createdAt,
  });

  @override
  String toString() {
    return 'NoteEntity(id: $id, title: $title, content: $content, createdAt: $createdAt)';
  }
}
```

and run build_runner to generate the `objectbox.g.dart` and `objectbox-model.json` files

```bash
dart run build_runner build -d
```

and create a service to interact with the database

```dart
import 'package:flutter_objectbox_sample/note_entity.dart';
import 'package:flutter_objectbox_sample/objectbox.g.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

class NoteService {
  static late Store _store;
  static late Box<NoteEntity> _noteBox;

  static Future<void> init() async {
    final directory = await getApplicationDocumentsDirectory();
    _store = await openStore(directory: p.join(directory.path, 'note-db'));
    _noteBox = _store.box<NoteEntity>();
  }

  static int get lastIndex => _noteBox.count();

  // CRUD operations
  static int addNote(NoteEntity note) {
    return _noteBox.put(note);
  }

  static List<NoteEntity> getAllNotes() {
    return _noteBox.getAll();
  }

  static List<NoteEntity> searchNotes(String keyword) {
    final query = _noteBox
        .query(
          NoteEntity_.title
              .contains(keyword)
              .and(NoteEntity_.content.contains(keyword)),
        )
        .build();
    final results = query.find();
    query.close();
    return results;
  }

  static bool updateNote(NoteEntity note) {
    return _noteBox.put(note) != 0;
  }

  static bool deleteNote(int id) {
    return _noteBox.remove(id);
  }

  static Stream<List<NoteEntity>> watchAllNotes() {
    return _noteBox
        .query()
        .watch(triggerImmediately: true)
        .map((query) => query.find());
  }

  static void dispose() {
    _store.close();
  }
}
```

#### Usage

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await NoteService.init();

  final recipeTitle = RecipeGenerator.generateRecipeTitle();
  final recipeContent = RecipeGenerator.generateRecipeContent(recipeTitle);

  final noteId = NoteService.addNote(
    NoteEntity(
      title: recipeTitle,
      content: recipeContent,
      createdAt: DateTime.now(),
    ),
  );

  print('Note added with id: $noteId');

  final notes = NoteService.getAllNotes();
  print('All notes: ${notes.length}');

  final searchResults = NoteService.searchNotes('Chicken');
  print('Search result:');
  for (var note in searchResults) {
    print('\n$note');
  }
}

class RecipeGenerator {
  static const _recipeTitles = [
    "Chicken",
    "Beef",
    "Pork",
    "Fish",
    "Vegetable",
    "Fruit",
    "Dessert",
    "Snack",
    "Drink",
    "Other",
  ];

  static String generateRecipeTitle() {
    return _recipeTitles[Random().nextInt(_recipeTitles.length)];
  }

  static String generateRecipeContent(String title) {
    return 'This is the recipe for $title';
  }
}
```

#### Pros and Cons

**Pros:**

- Built-in vector store and on device ANN vector search
- 10x faster than SQLite (This is one of the author's claims)
- Built-in support for relationships
- Simple schema migration
- Reactive queries with streams
- Minimal boilerplate code
- Offline-first Data Sync (Proprietary)

**Cons:**

- No web support

## 4.4 Choosing the Right Storage Solution

The choice of storage solution depends on your specific requirements:

### Use Shared Preferences when:

- Storing simple key-value pairs
- Managing user preferences and settings
- Data size is small (< 1MB)
- No complex queries needed

### Use Secure Storage when:

- Storing sensitive information (tokens, passwords)
- Data security is paramount
- Simple key-value storage is sufficient

### Use SQLite/sqflite when:

- Complex relational data structures
- Need for complex SQL queries
- ACID compliance is required
- Working with existing SQL knowledge

### Use Drift when:

- Want type-safety with SQL databases
- Need reactive UI updates
- Cross-platform support including web
- Prefer Dart over raw SQL

### Use Hive when:

- Simple object persistence
- Excellent performance is needed
- Cross-platform support including web
- Minimal setup and configuration
- NoSQL approach fits your data model

### Use ObjectBox when:

- Maximum performance is critical
- Object-oriented database approach preferred
- Need for complex relationships between objects
- Real-time reactive queries required
- Need for vector search
- Need for offline-first data sync

## 4.5 Best Practices

1. **Initialize storage services early**: Initialize your chosen storage solution in the `main()` function or during app startup.

2. **Use dependency injection**: Abstract your storage layer behind interfaces to make testing and switching between storage solutions easier.

3. **Handle errors gracefully**: Always wrap storage operations in try-catch blocks and provide fallback mechanisms.

4. **Implement proper data migration**: Plan for schema changes and data migrations, especially for database solutions.

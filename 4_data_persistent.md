# 4. Data Persistent

Flutter provides several options for local data storage, each suited for different use cases and requirements.

## 4.1 Shared Preferences

Shared Preferences is the simplest form of local storage in Flutter, designed for storing small pieces of primitive data such as user settings, preferences, and simple configuration values.

### Basic Usage

First, add the dependency to your `pubspec.yaml`:

```yaml
dependencies:
  shared_preferences: ^2.2.2
```

### Code Example

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
| SQLite (sqflite) | Relational | SQL      | Android, iOS, macOS                      | sqflite   |
| Drift            | Relational | SQL/Dart | Android, iOS, Web, Desktop, macOS, Linux | drift     |
| Hive CE          | NoSQL      | Dart     | Android, iOS, Web, Desktop, macOS, Linux | hive_ce   |
| ObjectBox        | NoSQL      | Dart     | Android, iOS, macOS, Linux, Windows      | objectbox |

### 4.3.1 SQLite (sqflite)

SQLite is a lightweight, serverless relational database that's widely used in mobile applications. The `sqflite` package provides Flutter integration for SQLite.

#### Dependencies

```yaml
dependencies:
  sqflite: ^2.3.0
  path: ^1.8.3
```

#### Code Example

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
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
    return await openDatabase(
      path,
      version: 1,
      onCreate: _createTable,
    );
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

#### Pros and Cons

**Pros:**

- Mature and stable technology
- SQL queries provide powerful data manipulation
- Excellent performance for complex queries
- Wide community support and documentation
- ACID compliance ensures data integrity

**Cons:**

- Requires SQL knowledge
- More verbose code compared to NoSQL alternatives
- Manual schema management and migrations
- Platform-specific limitations (not available on web)

### 4.3.2 Drift

Drift (formerly Moor) is a reactive persistence library built on top of SQLite, providing type-safe database access with Dart code generation.

#### Dependencies

```yaml
dependencies:
  drift: ^2.14.1
  sqlite3_flutter_libs: ^0.5.15
  path_provider: ^2.1.1
  path: ^1.8.3

dev_dependencies:
  drift_dev: ^2.14.1
  build_runner: ^2.4.7
```

#### Code Example

```dart
// database.dart
import 'dart:io';
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

part 'database.g.dart';

class Notes extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 100)();
  TextColumn get content => text()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

@DriftDatabase(tables: [Notes])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  // CRUD operations
  Future<int> insertNote(NotesCompanion note) => into(notes).insert(note);

  Future<List<Note>> getAllNotes() => select(notes).get();

  Future<bool> updateNote(NotesCompanion note) => update(notes).replace(note);

  Future<int> deleteNote(int id) =>
    (delete(notes)..where((n) => n.id.equals(id))).go();

  Stream<List<Note>> watchAllNotes() => select(notes).watch();
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'db.sqlite'));
    return NativeDatabase.createInBackground(file);
  });
}
```

#### Pros and Cons

**Pros:**

- Type-safe database access with code generation
- Reactive streams for real-time updates
- Cross-platform support including web
- Built-in migration support
- Excellent IDE support and error checking

**Cons:**

- Steeper learning curve
- Code generation adds build complexity
- Larger app size due to code generation
- Still requires SQL knowledge for complex queries

### 4.3.3 Hive

Hive is a lightweight, NoSQL database written in pure Dart, designed for Flutter applications with excellent performance characteristics.

#### Dependencies

```yaml
dependencies:
  hive_ce: ^2.6.0
  hive_ce_flutter: ^2.1.0
  path_provider: ^2.1.1

dev_dependencies:
  hive_ce_generator: ^1.6.0
  build_runner: ^2.4.7
```

#### Code Example

```dart
// note_model.dart
import 'package:hive_ce/hive.dart';

part 'note_model.g.dart';

@HiveType(typeId: 0)
class Note extends HiveObject {
  @HiveField(0)
  String title;

  @HiveField(1)
  String content;

  @HiveField(2)
  DateTime createdAt;

  Note({
    required this.title,
    required this.content,
    required this.createdAt,
  });
}

// hive_service.dart
import 'package:hive_ce_flutter/hive_flutter.dart';
import 'package:path_provider/path_provider.dart';

class HiveService {
  static late Box<Note> _notesBox;

  static Future<void> init() async {
    // Initialize Hive
    await Hive.initFlutter();

    // Register adapters
    Hive.registerAdapter(NoteAdapter());

    // Open boxes
    _notesBox = await Hive.openBox<Note>('notes');
  }

  // CRUD operations
  static Future<void> addNote(Note note) async {
    await _notesBox.add(note);
  }

  static List<Note> getAllNotes() {
    return _notesBox.values.toList();
  }

  static Future<void> updateNote(int index, Note note) async {
    await _notesBox.putAt(index, note);
  }

  static Future<void> deleteNote(int index) async {
    await _notesBox.deleteAt(index);
  }

  static ValueListenable<Box<Note>> getNotesListenable() {
    return _notesBox.listenable();
  }

  static Future<void> clearAll() async {
    await _notesBox.clear();
  }
}
```

#### Usage in Widget

```dart
class NotesScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Notes')),
      body: ValueListenableBuilder<Box<Note>>(
        valueListenable: HiveService.getNotesListenable(),
        builder: (context, box, _) {
          final notes = box.values.toList();

          if (notes.isEmpty) {
            return Center(child: Text('No notes yet'));
          }

          return ListView.builder(
            itemCount: notes.length,
            itemBuilder: (context, index) {
              final note = notes[index];
              return ListTile(
                title: Text(note.title),
                subtitle: Text(note.content),
                trailing: IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: () => HiveService.deleteNote(index),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

#### Pros and Cons

**Pros:**

- Pure Dart implementation (no native dependencies)
- Excellent performance and low memory footprint
- Simple and intuitive API
- Cross-platform support (including web)
- Built-in encryption support
- Reactive UI updates with ValueListenable

**Cons:**

- NoSQL limitations for complex relational queries
- Less mature ecosystem compared to SQLite
- Manual data modeling required
- Limited querying capabilities compared to SQL

### 4.3.4 ObjectBox

ObjectBox is a high-performance NoSQL database designed for mobile and IoT applications, offering fast object persistence with minimal setup.

#### Dependencies

```yaml
dependencies:
  objectbox: ^2.1.0
  objectbox_flutter_libs: any

dev_dependencies:
  objectbox_generator: any
  build_runner: ^2.4.7
```

#### Code Example

```dart
// note_model.dart
import 'package:objectbox/objectbox.dart';

@Entity()
class Note {
  @Id()
  int id = 0;

  String title;
  String content;
  DateTime createdAt;

  Note({
    required this.title,
    required this.content,
    required this.createdAt,
  });
}

// objectbox_service.dart
import 'package:objectbox/objectbox.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

class ObjectBoxService {
  static late Store _store;
  static late Box<Note> _noteBox;

  static Future<void> init() async {
    final docsDir = await getApplicationDocumentsDirectory();
    _store = await openStore(directory: p.join(docsDir.path, 'objectbox'));
    _noteBox = _store.box<Note>();
  }

  // CRUD operations
  static int addNote(Note note) {
    return _noteBox.put(note);
  }

  static List<Note> getAllNotes() {
    return _noteBox.getAll();
  }

  static bool updateNote(Note note) {
    return _noteBox.put(note) != 0;
  }

  static bool deleteNote(int id) {
    return _noteBox.remove(id);
  }

  static Stream<List<Note>> watchAllNotes() {
    return _noteBox.query().watch(triggerImmediately: true).map((query) => query.find());
  }

  static void dispose() {
    _store.close();
  }
}
```

#### Pros and Cons

**Pros:**

- Exceptional performance (10x faster than SQLite in many cases)
- Simple object-oriented API
- Built-in support for relationships and indexes
- Reactive queries with streams
- Cross-platform support
- Minimal boilerplate code

**Cons:**

- Larger binary size impact
- Less mature than SQLite/Hive
- Limited community resources
- Proprietary technology (commercial license required for some use cases)
- Platform-specific native dependencies

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

## 4.5 Best Practices

1. **Initialize storage services early**: Initialize your chosen storage solution in the `main()` function or during app startup.

2. **Use dependency injection**: Abstract your storage layer behind interfaces to make testing and switching between storage solutions easier.

3. **Handle errors gracefully**: Always wrap storage operations in try-catch blocks and provide fallback mechanisms.

4. **Implement proper data migration**: Plan for schema changes and data migrations, especially for database solutions.

5. **Consider data encryption**: For sensitive data, always use secure storage or implement encryption at the application level.

6. **Optimize for performance**: Use appropriate indexing for databases and consider lazy loading for large datasets.

7. **Test thoroughly**: Write unit tests for your storage layer and test with various data scenarios.

By understanding these different storage options and their appropriate use cases, you can make informed decisions about data persistence in your Flutter applications.

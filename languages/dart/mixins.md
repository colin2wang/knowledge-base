# Dart Mixins

Mixins are a powerful feature in Dart that allow classes to share code without using traditional inheritance. They provide a flexible way to add functionality to classes, enabling multiple inheritance-like behavior while avoiding the complexity of deep inheritance hierarchies.

## What are Mixins?

Mixins are a way of reusing a class's code in multiple class hierarchies. They allow you to add methods and properties to classes without affecting the inheritance chain.

### Basic Mixin Syntax

```dart
// Define a mixin
mixin Flyable {
  void fly() {
    print('Flying...');
  }
  
  bool get canFly => true;
}

mixin Swimmable {
  void swim() {
    print('Swimming...');
  }
  
  bool get canSwim => true;
}

mixin Walkable {
  void walk() {
    print('Walking...');
  }
  
  bool get canWalk => true;
}

// Using mixins with a class
class Bird with Flyable, Walkable {
  String name;
  
  Bird(this.name);
  
  void chirp() {
    print('$name is chirping');
  }
}

class Fish with Swimmable {
  String species;
  
  Fish(this.species);
  
  void bubble() {
    print('$species is making bubbles');
  }
}

class Duck with Flyable, Swimmable, Walkable {
  String name;
  
  Duck(this.name);
  
  void quack() {
    print('$name says: Quack!');
  }
}

void demonstrateBasicMixins() {
  print('=== Basic Mixins Demo ===');
  
  Bird bird = Bird('Robin');
  Fish fish = Fish('Goldfish');
  Duck duck = Duck('Donald');
  
  print('${bird.name}:');
  bird.fly();
  bird.walk();
  bird.chirp();
  print('Can fly: ${bird.canFly}');
  print('Can walk: ${bird.canWalk}');
  print('');
  
  print('${fish.species}:');
  fish.swim();
  fish.bubble();
  print('Can swim: ${fish.canSwim}');
  print('');
  
  print('${duck.name}:');
  duck.fly();
  duck.swim();
  duck.walk();
  duck.quack();
  print('Can fly: ${duck.canFly}');
  print('Can swim: ${duck.canSwim}');
  print('Can walk: ${duck.canWalk}');
}
```

### Mixins vs Traditional Inheritance

```dart
// Traditional inheritance approach (limited)
class Animal {
  void breathe() => print('Breathing');
}

class Mammal extends Animal {
  void nurse() => print('Nursing young');
}

class Bird2 extends Animal {
  void fly() => print('Flying');
}

// This creates problems - what about flying mammals?
class Bat extends Mammal {
  // We have to duplicate flying functionality or use complex inheritance
  void fly() => print('Flying with wings');
}

// Mixin approach (flexible)
mixin Breathing {
  void breathe() => print('Breathing through lungs');
}

mixin Nursing {
  void nurse() => print('Producing milk for offspring');
}

mixin Flying {
  void fly() => print('Flying with wings');
}

mixin Echolocation {
  void echolocate() => print('Using sonar for navigation');
}

class Dog with Breathing, Nursing {
  String name;
  Dog(this.name);
}

class Eagle with Breathing, Flying {
  String name;
  Eagle(this.name);
}

class Bat2 with Breathing, Nursing, Flying, Echolocation {
  String species;
  Bat2(this.species);
}

void demonstrateMixinAdvantages() {
  print('\n=== Mixin Advantages Demo ===');
  
  Dog dog = Dog('Buddy');
  Eagle eagle = Eagle('Freedom');
  Bat2 bat = Bat2('Vampire Bat');
  
  print('${dog.name} (Dog):');
  dog.breathe();
  dog.nurse();
  print('');
  
  print('${eagle.name} (Eagle):');
  eagle.breathe();
  eagle.fly();
  print('');
  
  print('${bat.species} (Bat):');
  bat.breathe();
  bat.nurse();
  bat.fly();
  bat.echolocate();
}
```

## Advanced Mixin Features

### Mixin Constraints

```dart
// Mixin that can only be applied to specific types
mixin Musical on Instrument {
  void playMusic() {
    print('Playing beautiful music');
  }
  
  void tune() {
    print('Tuning the instrument');
  }
}

mixin Electronic on Device {
  void charge() {
    print('Charging device');
  }
  
  bool get isCharged => true;
}

abstract class Instrument {
  String name;
  Instrument(this.name);
  
  void play() => print('Playing $name');
}

abstract class Device {
  String model;
  Device(this.model);
  
  void powerOn() => print('$model powered on');
}

class Piano extends Instrument with Musical {
  Piano(String name) : super(name);
  
  @override
  void play() {
    super.play();
    playMusic();
  }
}

class Smartphone extends Device with Electronic {
  Smartphone(String model) : super(model);
  
  void makeCall() {
    if (isCharged) {
      print('Making phone call');
    } else {
      charge();
    }
  }
}

class SmartPiano extends Instrument with Musical, Electronic {
  SmartPiano(String name) : super(name);
  
  @override
  void play() {
    if (isCharged) {
      super.play();
      playMusic();
    } else {
      charge();
      print('Please wait for charging to complete');
    }
  }
}

void demonstrateMixinConstraints() {
  print('\n=== Mixin Constraints Demo ===');
  
  Piano piano = Piano('Grand Piano');
  Smartphone phone = Smartphone('iPhone');
  SmartPiano smartPiano = SmartPiano('Digital Grand');
  
  print('Piano:');
  piano.tune();
  piano.play();
  print('');
  
  print('Smartphone:');
  phone.powerOn();
  phone.charge();
  phone.makeCall();
  print('');
  
  print('Smart Piano:');
  smartPiano.powerOn();
  smartPiano.tune();
  smartPiano.play();
}
```

### Mixin Constructor Behavior

```dart
mixin TimestampMixin {
  DateTime createdAt = DateTime.now();
  DateTime? updatedAt;
  
  void updateTimestamp() {
    updatedAt = DateTime.now();
    print('Updated at: $updatedAt');
  }
  
  @override
  String toString() => 'Created: $createdAt, Updated: $updatedAt';
}

mixin ValidationMixin {
  bool isValid = false;
  
  void validate() {
    isValid = true;
    print('Object validated');
  }
  
  void invalidate() {
    isValid = false;
    print('Object invalidated');
  }
}

class DataRecord with TimestampMixin, ValidationMixin {
  String data;
  
  DataRecord(this.data) {
    print('DataRecord created with: $data');
  }
  
  void processData() {
    if (!isValid) {
      validate();
    }
    updateTimestamp();
    print('Processing data: $data');
  }
}

class User with TimestampMixin, ValidationMixin {
  String username;
  String email;
  
  User(this.username, this.email) {
    print('User created: $username <$email>');
  }
  
  void updateUser(String newEmail) {
    email = newEmail;
    updateTimestamp();
    invalidate(); // Needs revalidation
    print('User updated: $username <$email>');
  }
}

void demonstrateMixinConstructors() {
  print('\n=== Mixin Constructor Behavior Demo ===');
  
  DataRecord record = DataRecord('Sample data');
  User user = User('john_doe', 'john@example.com');
  
  print('\nInitial states:');
  print('Record: $record');
  print('User: $user');
  
  print('\nAfter processing:');
  record.processData();
  user.updateUser('john.doe@company.com');
  
  print('\nFinal states:');
  print('Record: $record');
  print('User: $user');
}
```

## Practical Mixin Patterns

### 1. Notification System

```dart
mixin Observable {
  final List<Function> _observers = [];
  
  void addObserver(Function observer) {
    _observers.add(observer);
    print('Observer added. Total observers: ${_observers.length}');
  }
  
  void removeObserver(Function observer) {
    _observers.remove(observer);
    print('Observer removed. Total observers: ${_observers.length}');
  }
  
  void notifyObservers(dynamic data) {
    for (var observer in _observers) {
      observer(data);
    }
  }
}

mixin Loggable {
  String get logPrefix => runtimeType.toString();
  
  void log(String message) {
    print('[$logPrefix] $message');
  }
  
  void logError(String error) {
    print('[$logPrefix] ERROR: $error');
  }
}

class UserService with Observable, Loggable {
  Map<String, String> _users = {};
  
  void addUser(String id, String name) {
    _users[id] = name;
    log('Added user: $id -> $name');
    notifyObservers({'action': 'add', 'id': id, 'name': name});
  }
  
  void removeUser(String id) {
    if (_users.containsKey(id)) {
      String name = _users.remove(id)!;
      log('Removed user: $id -> $name');
      notifyObservers({'action': 'remove', 'id': id, 'name': name});
    } else {
      logError('User not found: $id');
    }
  }
  
  String? getUser(String id) {
    return _users[id];
  }
  
  List<String> getAllUsers() {
    return _users.values.toList();
  }
}

void demonstrateNotificationSystem() {
  print('\n=== Notification System Demo ===');
  
  UserService userService = UserService();
  
  // Add observers
  userService.addObserver((data) {
    print('Observer 1 received: $data');
  });
  
  userService.addObserver((data) {
    print('Observer 2 received: $data');
  });
  
  // Perform operations
  userService.addUser('001', 'Alice');
  userService.addUser('002', 'Bob');
  userService.removeUser('001');
  userService.removeUser('999'); // Non-existent user
  
  print('All users: ${userService.getAllUsers()}');
}
```

### 2. State Management Mixin

```dart
mixin Stateful<T> {
  T _state;
  final List<Function(T)> _listeners = [];
  
  Stateful(this._state);
  
  T get state => _state;
  
  set state(T newState) {
    if (_state != newState) {
      T oldState = _state;
      _state = newState;
      _notifyListeners(newState, oldState);
    }
  }
  
  void addListener(Function(T) listener) {
    _listeners.add(listener);
  }
  
  void removeListener(Function(T) listener) {
    _listeners.remove(listener);
  }
  
  void _notifyListeners(T newState, T oldState) {
    for (var listener in _listeners) {
      listener(newState);
    }
  }
  
  void updateState(T Function(T) updater) {
    state = updater(state);
  }
}

class Counter with Stateful<int> {
  Counter() : super(0);
  
  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
  
  void incrementBy(int amount) => state += amount;
}

class TodoList with Stateful<List<Map<String, dynamic>>> {
  TodoList() : super([]);
  
  void addTodo(String title) {
    updateState((todos) => [
      ...todos,
      {'title': title, 'completed': false, 'id': DateTime.now().millisecondsSinceEpoch}
    ]);
  }
  
  void toggleTodo(int id) {
    updateState((todos) => todos.map((todo) {
      if (todo['id'] == id) {
        return {...todo, 'completed': !todo['completed']};
      }
      return todo;
    }).toList());
  }
  
  void removeTodo(int id) {
    updateState((todos) => todos.where((todo) => todo['id'] != id).toList());
  }
  
  List<Map<String, dynamic>> get completedTodos => 
      state.where((todo) => todo['completed'] as bool).toList();
  
  List<Map<String, dynamic>> get pendingTodos => 
      state.where((todo) => !(todo['completed'] as bool)).toList();
}

void demonstrateStateManager() {
  print('\n=== State Management Demo ===');
  
  // Counter example
  Counter counter = Counter();
  
  counter.addListener((newState) {
    print('Counter changed to: $newState');
  });
  
  counter.increment();
  counter.increment();
  counter.incrementBy(5);
  counter.decrement();
  print('Final counter value: ${counter.state}');
  
  print('\n--- Todo List Example ---');
  
  // Todo list example
  TodoList todoList = TodoList();
  
  todoList.addListener((todos) {
    print('Todo list updated. Total items: ${todos.length}');
  });
  
  todoList.addTodo('Learn Dart');
  todoList.addTodo('Build Flutter app');
  todoList.addTodo('Deploy to production');
  
  print('Pending todos: ${todoList.pendingTodos.length}');
  print('Completed todos: ${todoList.completedTodos.length}');
  
  // Toggle first todo
  if (todoList.state.isNotEmpty) {
    int firstId = todoList.state[0]['id'];
    todoList.toggleTodo(firstId);
  }
  
  print('After toggling first todo:');
  print('Pending todos: ${todoList.pendingTodos.length}');
  print('Completed todos: ${todoList.completedTodos.length}');
}
```

### 3. Caching Mixin

```dart
mixin Cacheable<K, V> {
  final Map<K, V> _cache = {};
  final Map<K, DateTime> _timestamps = {};
  Duration cacheDuration = const Duration(minutes: 5);
  
  V? getCached(K key) {
    if (_cache.containsKey(key)) {
      DateTime timestamp = _timestamps[key]!;
      if (DateTime.now().difference(timestamp) < cacheDuration) {
        print('Cache hit for key: $key');
        return _cache[key];
      } else {
        print('Cache expired for key: $key');
        _invalidate(key);
      }
    }
    print('Cache miss for key: $key');
    return null;
  }
  
  void cache(K key, V value) {
    _cache[key] = value;
    _timestamps[key] = DateTime.now();
    print('Cached value for key: $key');
  }
  
  void _invalidate(K key) {
    _cache.remove(key);
    _timestamps.remove(key);
  }
  
  void clearCache() {
    _cache.clear();
    _timestamps.clear();
    print('Cache cleared');
  }
  
  int get cacheSize => _cache.length;
  
  List<K> get cachedKeys => _cache.keys.toList();
}

class ApiClient with Cacheable<String, dynamic> {
  // Simulate API calls
  Future<dynamic> fetchUserData(String userId) async {
    // Check cache first
    var cached = getCached(userId);
    if (cached != null) {
      return cached;
    }
    
    print('Fetching user data from API for: $userId');
    // Simulate network delay
    await Future.delayed(Duration(milliseconds: 500));
    
    // Simulate API response
    var userData = {
      'id': userId,
      'name': 'User $userId',
      'email': '$userId@example.com',
      'lastUpdated': DateTime.now().toIso8601String()
    };
    
    // Cache the result
    cache(userId, userData);
    return userData;
  }
  
  Future<dynamic> fetchProductData(String productId) async {
    var cached = getCached(productId);
    if (cached != null) {
      return cached;
    }
    
    print('Fetching product data from API for: $productId');
    await Future.delayed(Duration(milliseconds: 300));
    
    var productData = {
      'id': productId,
      'name': 'Product $productId',
      'price': (100 + (productId.hashCode % 900)).toDouble(),
      'inStock': productId.hashCode % 2 == 0
    };
    
    cache(productId, productData);
    return productData;
  }
}

void demonstrateCaching() async {
  print('\n=== Caching Mixin Demo ===');
  
  ApiClient client = ApiClient();
  
  print('First calls (cache misses):');
  var user1 = await client.fetchUserData('user123');
  var product1 = await client.fetchProductData('prod456');
  
  print('\nSecond calls (should be cache hits):');
  var user1Again = await client.fetchUserData('user123');
  var product1Again = await client.fetchProductData('prod456');
  
  print('\nCache statistics:');
  print('Cache size: ${client.cacheSize}');
  print('Cached keys: ${client.cachedKeys}');
  
  print('\nClearing cache:');
  client.clearCache();
  print('Cache size after clear: ${client.cacheSize}');
  
  print('\nCalls after cache clear (cache misses again):');
  var user1Third = await client.fetchUserData('user123');
}
```

## Mixin Composition and Order

### Mixin Application Order

```dart
mixin First {
  void method() {
    print('First mixin method');
  }
  
  void sharedMethod() {
    print('First mixin shared method');
  }
}

mixin Second {
  void method() {
    print('Second mixin method');
  }
  
  void sharedMethod() {
    print('Second mixin shared method');
  }
}

mixin Third {
  void method() {
    print('Third mixin method');
  }
  
  void anotherMethod() {
    print('Third mixin another method');
  }
}

// Order matters - rightmost mixin takes precedence
class MyClass1 with First, Second, Third {
  // Third.method() will be used
  // Third.sharedMethod() will be used
}

class MyClass2 with Third, Second, First {
  // First.method() will be used
  // First.sharedMethod() will be used
}

void demonstrateMixinOrder() {
  print('\n=== Mixin Order Demo ===');
  
  print('MyClass1 (First, Second, Third):');
  MyClass1 obj1 = MyClass1();
  obj1.method();        // Third's method
  obj1.sharedMethod();  // Second's sharedMethod
  obj1.anotherMethod(); // Third's anotherMethod
  
  print('\nMyClass2 (Third, Second, First):');
  MyClass2 obj2 = MyClass2();
  obj2.method();        // First's method
  obj2.sharedMethod();  // First's sharedMethod
  obj2.anotherMethod(); // Third's anotherMethod
}
```

### Diamond Problem Solution

```dart
mixin Base {
  void commonMethod() {
    print('Base common method');
  }
}

mixin A on Base {
  void methodA() {
    print('Method A');
    commonMethod(); // Calls Base's implementation
  }
}

mixin B on Base {
  void methodB() {
    print('Method B');
    commonMethod(); // Calls Base's implementation
  }
}

// No diamond problem - each mixin has its own path to Base
class Combined with Base, A, B {
  void combinedMethod() {
    print('Combined method');
    methodA();
    methodB();
    commonMethod(); // Calls Base's implementation
  }
}

void demonstrateDiamondSolution() {
  print('\n=== Diamond Problem Solution Demo ===');
  
  Combined combined = Combined();
  combined.combinedMethod();
}
```

## Advanced Mixin Techniques

### 1. Generic Mixins

```dart
mixin Repository<T> {
  final List<T> _items = [];
  int _nextId = 1;
  
  T add(T item) {
    _items.add(item);
    print('Added item: $item');
    return item;
  }
  
  List<T> getAll() => List.unmodifiable(_items);
  
  T? findById(int id) {
    return _items.cast<dynamic>().firstWhere(
      (item) => item is HasId && item.id == id,
      orElse: () => null,
    ) as T?;
  }
  
  void remove(T item) {
    _items.remove(item);
    print('Removed item: $item');
  }
  
  int get count => _items.length;
}

mixin Timestamped {
  DateTime createdAt = DateTime.now();
  DateTime updatedAt = DateTime.now();
  
  void updateTimestamp() {
    updatedAt = DateTime.now();
  }
}

abstract class HasId {
  int get id;
}

class User2 implements HasId {
  @override
  final int id;
  final String name;
  final String email;
  
  User2(this.id, this.name, this.email);
  
  @override
  String toString() => 'User(id: $id, name: $name, email: $email)';
}

class Product implements HasId {
  @override
  final int id;
  final String name;
  final double price;
  
  Product(this.id, this.name, this.price);
  
  @override
  String toString() => 'Product(id: $id, name: $name, price: \$${price.toStringAsFixed(2)})';
}

class UserRepository with Repository<User2>, Timestamped {
  UserRepository() {
    // Initialize with some data
    add(User2(1, 'Alice', 'alice@example.com'));
    add(User2(2, 'Bob', 'bob@example.com'));
  }
  
  User2 createUser(String name, String email) {
    updateTimestamp();
    return add(User2(_nextId++, name, email));
  }
}

class ProductRepository with Repository<Product>, Timestamped {
  ProductRepository() {
    add(Product(1, 'Laptop', 999.99));
    add(Product(2, 'Mouse', 29.99));
  }
  
  Product createProduct(String name, double price) {
    updateTimestamp();
    return add(Product(_nextId++, name, price));
  }
}

void demonstrateGenericMixins() {
  print('\n=== Generic Mixins Demo ===');
  
  UserRepository userRepo = UserRepository();
  ProductRepository productRepo = ProductRepository();
  
  print('Users:');
  userRepo.getAll().forEach(print);
  print('User count: ${userRepo.count}');
  
  print('\nAdding new user:');
  User2 newUser = userRepo.createUser('Charlie', 'charlie@example.com');
  print('Created: $newUser');
  print('Updated at: ${userRepo.updatedAt}');
  
  print('\nProducts:');
  productRepo.getAll().forEach(print);
  print('Product count: ${productRepo.count}');
  
  print('\nAdding new product:');
  Product newProduct = productRepo.createProduct('Keyboard', 79.99);
  print('Created: $newProduct');
  print('Updated at: ${productRepo.updatedAt}');
}
```

### 2. Mixin with Static Members

```dart
mixin Configurable {
  static const String defaultEnvironment = 'development';
  static Map<String, String> _configs = {};
  
  static void setConfig(String key, String value) {
    _configs[key] = value;
    print('Config set: $key = $value');
  }
  
  static String? getConfig(String key) {
    return _configs[key];
  }
  
  static void loadFromFile(String filename) {
    print('Loading config from $filename');
    // Simulate loading
    _configs['database_url'] = 'localhost:5432';
    _configs['api_key'] = 'secret123';
    _configs['debug_mode'] = 'true';
  }
  
  String get environment => getConfig('environment') ?? defaultEnvironment;
  
  bool get isDebugMode => getConfig('debug_mode') == 'true';
  
  void printConfig() {
    print('Environment: $environment');
    print('Debug mode: $isDebugMode');
    print('All configs: $_configs');
  }
}

class Application with Configurable {
  String name;
  
  Application(this.name) {
    print('Application $name initialized');
  }
  
  void start() {
    print('Starting $name in $environment mode');
    if (isDebugMode) {
      print('Debug logging enabled');
    }
  }
}

class Service with Configurable {
  String serviceName;
  
  Service(this.serviceName);
  
  void connect() {
    String? dbUrl = getConfig('database_url');
    if (dbUrl != null) {
      print('$serviceName connecting to database: $dbUrl');
    } else {
      print('$serviceName: No database URL configured');
    }
  }
}

void demonstrateStaticMixins() {
  print('\n=== Static Mixin Members Demo ===');
  
  // Load configuration
  Configurable.loadFromFile('config.json');
  Configurable.setConfig('environment', 'production');
  Configurable.setConfig('debug_mode', 'false');
  
  Application app = Application('MyApp');
  Service service = Service('UserService');
  
  print('\nApplication config:');
  app.printConfig();
  app.start();
  
  print('\nService config:');
  service.printConfig();
  service.connect();
  
  print('\nStatic config access:');
  print('Default environment: ${Configurable.defaultEnvironment}');
  print('Database URL: ${Configurable.getConfig('database_url')}');
}
```

## Mixin Best Practices

### 1. Proper Mixin Design

```dart
// Good: Well-defined, focused mixins
mixin Validatable {
  bool validate();
  List<String> get validationErrors;
}

mixin Serializable {
  Map<String, dynamic> toJson();
  void fromJson(Map<String, dynamic> json);
}

mixin ComparableMixin<T> on Comparable<T> {
  bool operator <(T other) => compareTo(other) < 0;
  bool operator <=(T other) => compareTo(other) <= 0;
  bool operator >(T other) => compareTo(other) > 0;
  bool operator >=(T other) => compareTo(other) >= 0;
}

// Bad: Too broad or overlapping functionality
mixin KitchenSink {
  void validate() { /* ... */ }
  Map<String, dynamic> toJson() { /* ... */ }
  void connect() { /* ... */ }
  void disconnect() { /* ... */ }
  // Too many responsibilities!
}

// Good: Specific purpose mixins
mixin DatabaseConnection {
  bool isConnected = false;
  
  void connect() {
    isConnected = true;
    print('Database connected');
  }
  
  void disconnect() {
    isConnected = false;
    print('Database disconnected');
  }
}

mixin NetworkAware {
  bool get isOnline => true;
  
  void checkConnectivity() {
    print('Checking network connectivity');
  }
}
```

### 2. Mixin Conflict Resolution

```dart
mixin Logger {
  void log(String message) {
    print('[LOGGER] $message');
  }
}

mixin Auditor {
  void log(String message) {
    print('[AUDITOR] $message');
  }
}

// Solution 1: Override conflicting method
class AuditLogger with Logger, Auditor {
  @override
  void log(String message) {
    // Call both implementations
    super.log(message); // Auditor's log
    (super as Logger).log(message); // Logger's log
  }
}

// Solution 2: Use different method names
mixin NamedLogger {
  void logInfo(String message) {
    print('[INFO] $message');
  }
}

mixin NamedAuditor {
  void logAudit(String message) {
    print('[AUDIT] $message');
  }
}

class SeparateLogger with NamedLogger, NamedAuditor {
  void logBoth(String message) {
    logInfo(message);
    logAudit(message);
  }
}

void demonstrateConflictResolution() {
  print('\n=== Mixin Conflict Resolution Demo ===');
  
  AuditLogger auditLogger = AuditLogger();
  print('Combined logger:');
  auditLogger.log('System started');
  
  SeparateLogger separateLogger = SeparateLogger();
  print('\nSeparate loggers:');
  separateLogger.logBoth('User logged in');
}
```

## Summary

Mixins provide powerful code reuse capabilities in Dart:

✅ **Key Benefits:**
- Multiple inheritance-like behavior without complex hierarchies
- Flexible composition of functionality
- Clean separation of concerns
- No diamond problem issues

⚠️ **Best Practices:**
- Keep mixins focused on single responsibilities
- Use `on` clause for mixin constraints when needed
- Be mindful of method name conflicts
- Consider order of mixin application
- Use mixins for cross-cutting concerns

Mixins are essential for building flexible, maintainable Dart applications, especially in Flutter development where they're commonly used for widget composition and feature mixing.
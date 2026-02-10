# Dart Classes and Objects

In Dart, everything is an object, and every object is an instance of a class. Classes are the fundamental building blocks for object-oriented programming, defining the structure and behavior of objects.

## What are Classes and Objects?

### Basic Class Definition

```dart
// Basic class definition
class Person {
  // Fields (instance variables)
  String name;
  int age;
  String email;
  
  // Constructor
  Person(String name, int age, String email) {
    this.name = name;
    this.age = age;
    this.email = email;
  }
  
  // Method
  void introduce() {
    print('Hi, I\'m $name, $age years old. Contact me at $email');
  }
  
  // Getter
  bool get isAdult => age >= 18;
  
  // Setter
  set age(int value) {
    if (value >= 0) {
      this.age = value;
    }
  }
}

// Object creation and usage
void main() {
  Person person = Person('Alice', 25, 'alice@example.com');
  person.introduce();
  print('Is adult: ${person.isAdult}');
}
```

### Constructors

#### Default Constructor
```dart
class Student {
  String name;
  int grade;
  
  // Default constructor
  Student(String name, int grade) {
    this.name = name;
    this.grade = grade;
  }
}

// Usage
Student student = Student('Bob', 10);
```

#### Constructor Syntax Sugar
```dart
class Student {
  String name;
  int grade;
  
  // Constructor with field initialization
  Student(this.name, this.grade);
  
  // Named constructor
  Student.withNameOnly(this.name) : grade = 0;
  
  // Forwarding constructor
  Student.createSenior(String name) : this(name, 12);
}

// Usage
Student student1 = Student('Charlie', 8);
Student student2 = Student.withNameOnly('David');
Student student3 = Student.createSenior('Eve');
```

#### Initializer Lists
```dart
class Rectangle {
  final double width;
  final double height;
  final double area;
  
  Rectangle(double width, double height)
      : width = width,
        height = height,
        area = width * height {
    print('Rectangle created with area: $area');
  }
  
  // Assert in initializer list
  Rectangle.square(double side)
      : assert(side > 0),
        width = side,
        height = side,
        area = side * side;
}

// Usage
Rectangle rect1 = Rectangle(5, 3);
Rectangle square = Rectangle.square(4);
```

#### Factory Constructors
```dart
class Logger {
  final String name;
  static final Map<String, Logger> _cache = {};
  
  // Private constructor
  Logger._internal(this.name);
  
  // Factory constructor
  factory Logger(String name) {
    if (_cache.containsKey(name)) {
      return _cache[name]!;
    } else {
      final logger = Logger._internal(name);
      _cache[name] = logger;
      return logger;
    }
  }
  
  void log(String message) {
    print('[$name] $message');
  }
}

// Usage - same name returns same instance
Logger logger1 = Logger('App');
Logger logger2 = Logger('App');
print(identical(logger1, logger2)); // true
```

### Named Constructors
```dart
class Point {
  double x, y;
  
  Point(this.x, this.y);
  
  // Named constructors for common patterns
  Point.origin() : x = 0, y = 0;
  Point.fromJson(Map<String, dynamic> json) 
      : x = json['x'], 
        y = json['y'];
  
  // Named constructor with computation
  Point.polar(double radius, double angle) 
      : x = radius * Math.cos(angle),
        y = radius * Math.sin(angle);
}

// Usage
Point origin = Point.origin();
Point fromData = Point.fromJson({'x': 10, 'y': 20});
Point polar = Point.polar(5, Math.pi / 4);
```

## Object Instantiation and Lifecycle

### Object Creation Process
```dart
class DatabaseConnection {
  final String connectionString;
  bool isConnected = false;
  
  DatabaseConnection(this.connectionString) {
    print('Constructor: Initializing connection to $connectionString');
  }
  
  // Initialization method
  void connect() {
    isConnected = true;
    print('Connected to database');
  }
  
  void disconnect() {
    isConnected = false;
    print('Disconnected from database');
  }
  
  // Destructor-like cleanup
  void dispose() {
    if (isConnected) {
      disconnect();
    }
    print('Connection disposed');
  }
}

void demonstrateLifecycle() {
  print('=== Object Lifecycle Demo ===');
  
  // 1. Object creation
  DatabaseConnection db = DatabaseConnection('localhost:5432');
  
  // 2. Object usage
  db.connect();
  print('Is connected: ${db.isConnected}');
  
  // 3. Object cleanup
  db.dispose();
}
```

### Constant Constructors
```dart
class Color {
  final int red;
  final int green;
  final int blue;
  
  // Constant constructor - all fields must be final
  const Color(this.red, this.green, this.blue);
  
  // Named constant constructors
  const Color.black() : red = 0, green = 0, blue = 0;
  const Color.white() : red = 255, green = 255, blue = 255;
  const Color.red() : red = 255, green = 0, blue = 0;
  
  @override
  String toString() => 'RGB($red, $green, $blue)';
}

void demonstrateConstants() {
  // Compile-time constants
  const black = Color.black();
  const white = Color.white();
  const red = Color.red();
  
  // Same instances due to canonicalization
  const color1 = Color(255, 0, 0);
  const color2 = Color(255, 0, 0);
  print(identical(color1, color2)); // true
  
  print('Colors: $black, $white, $red');
}
```

## Instance Members

### Fields and Properties
```dart
class BankAccount {
  // Private fields (library private)
  String _accountNumber;
  double _balance;
  
  // Public fields
  final String owner;
  final DateTime createdAt;
  
  // Getters and setters
  String get accountNumber => '****${_accountNumber.substring(_accountNumber.length - 4)}';
  
  double get balance => _balance;
  
  set balance(double value) {
    if (value >= 0) {
      _balance = value;
    } else {
      throw ArgumentError('Balance cannot be negative');
    }
  }
  
  BankAccount(this.owner, String accountNumber, double initialBalance)
      : _accountNumber = accountNumber,
        _balance = initialBalance,
        createdAt = DateTime.now();
  
  void deposit(double amount) {
    if (amount > 0) {
      _balance += amount;
      print('Deposited \$${amount}. New balance: \$${_balance}');
    }
  }
  
  bool withdraw(double amount) {
    if (amount > 0 && amount <= _balance) {
      _balance -= amount;
      print('Withdrew \$${amount}. New balance: \$${_balance}');
      return true;
    }
    print('Insufficient funds or invalid amount');
    return false;
  }
}

void demonstrateFields() {
  BankAccount account = BankAccount('John Doe', 'ACC123456789', 1000.0);
  print('Account: ${account.accountNumber}');
  print('Owner: ${account.owner}');
  print('Balance: \$${account.balance}');
  
  account.deposit(500);
  account.withdraw(200);
}
```

### Methods
```dart
class Calculator {
  // Instance methods
  double add(double a, double b) => a + b;
  double subtract(double a, double b) => a - b;
  double multiply(double a, double b) => a * b;
  double divide(double a, double b) {
    if (b == 0) throw ArgumentError('Division by zero');
    return a / b;
  }
  
  // Method with optional parameters
  double calculate(String operation, double a, double b, {int precision = 2}) {
    double result;
    switch (operation.toLowerCase()) {
      case 'add':
        result = add(a, b);
        break;
      case 'subtract':
        result = subtract(a, b);
        break;
      case 'multiply':
        result = multiply(a, b);
        break;
      case 'divide':
        result = divide(a, b);
        break;
      default:
        throw ArgumentError('Unknown operation: $operation');
    }
    return double.parse(result.toStringAsFixed(precision));
  }
  
  // Static method
  static bool isValidNumber(String input) {
    return double.tryParse(input) != null;
  }
}

void demonstrateMethods() {
  Calculator calc = Calculator();
  
  print('Addition: ${calc.add(10, 5)}');
  print('Division: ${calc.divide(10, 3)}');
  print('Precise calculation: ${calc.calculate('divide', 10, 3, precision: 4)}');
  
  print('Is valid number: ${Calculator.isValidNumber('123.45')}');
}
```

## Inheritance and Object Relationships

### Basic Inheritance
```dart
class Animal {
  String name;
  int age;
  
  Animal(this.name, this.age);
  
  void eat() {
    print('$name is eating');
  }
  
  void sleep() {
    print('$name is sleeping');
  }
  
  @override
  String toString() => 'Animal(name: $name, age: $age)';
}

class Dog extends Animal {
  String breed;
  
  Dog(String name, int age, this.breed) : super(name, age);
  
  void bark() {
    print('$name says woof!');
  }
  
  // Override parent method
  @override
  void eat() {
    super.eat(); // Call parent method
    print('$name is eating dog food');
  }
  
  @override
  String toString() => '${super.toString()}, breed: $breed';
}

void demonstrateInheritance() {
  Dog dog = Dog('Buddy', 3, 'Golden Retriever');
  print(dog);
  dog.eat();
  dog.bark();
  dog.sleep();
}
```

### Object Identity and Equality
```dart
class Person {
  final String name;
  final int age;
  
  Person(this.name, this.age);
  
  // Override equality
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is Person && other.name == name && other.age == age;
  }
  
  @override
  int get hashCode => Object.hash(name, age);
  
  @override
  String toString() => 'Person($name, $age)';
}

void demonstrateEquality() {
  Person person1 = Person('Alice', 25);
  Person person2 = Person('Alice', 25);
  Person person3 = Person('Bob', 30);
  
  print('person1 == person2: ${person1 == person2}'); // true
  print('person1 == person3: ${person1 == person3}'); // false
  print('Identical: ${identical(person1, person2)}'); // false
  
  // Using in collections
  Set<Person> people = {person1, person2, person3};
  print('Unique people: ${people.length}'); // 2
}
```

## Advanced Class Features

### Cascades and Method Chaining
```dart
class StringBuilder {
  final List<String> _parts = [];
  
  StringBuilder append(String text) {
    _parts.add(text);
    return this; // Enable method chaining
  }
  
  StringBuilder appendLine(String text) {
    _parts.add('$text\n');
    return this;
  }
  
  String build() => _parts.join();
  
  @override
  String toString() => build();
}

void demonstrateCascades() {
  // Method chaining
  StringBuilder builder = StringBuilder()
    ..append('Hello')
    ..append(' ')
    ..append('World')
    ..appendLine('!')
    ..append('How are you?');
  
  print(builder);
  
  // Object configuration with cascades
  class Config {
    String host = 'localhost';
    int port = 8080;
    bool ssl = false;
    String apiKey = '';
  }
  
  Config config = Config()
    ..host = 'api.example.com'
    ..port = 443
    ..ssl = true
    ..apiKey = 'secret123';
  
  print('Config: ${config.host}:${config.port}, SSL: ${config.ssl}');
}
```

### Operators Overloading
```dart
class Vector2D {
  final double x, y;
  
  Vector2D(this.x, this.y);
  
  // Arithmetic operators
  Vector2D operator +(Vector2D other) => Vector2D(x + other.x, y + other.y);
  Vector2D operator -(Vector2D other) => Vector2D(x - other.x, y - other.y);
  Vector2D operator *(double scalar) => Vector2D(x * scalar, y * scalar);
  
  // Comparison operators
  bool operator ==(Object other) =>
      other is Vector2D && x == other.x && y == other.y;
  
  bool operator <(Vector2D other) => magnitude < other.magnitude;
  bool operator >(Vector2D other) => magnitude > other.magnitude;
  
  // Unary operators
  Vector2D operator -() => Vector2D(-x, -y);
  
  // Getters
  double get magnitude => Math.sqrt(x * x + y * y);
  double get angle => Math.atan2(y, x);
  
  @override
  String toString() => 'Vector2D($x, $y)';
  
  @override
  int get hashCode => Object.hash(x, y);
}

void demonstrateOperators() {
  Vector2D v1 = Vector2D(3, 4);
  Vector2D v2 = Vector2D(1, 2);
  
  print('v1 + v2 = ${v1 + v2}');
  print('v1 - v2 = ${v1 - v2}');
  print('v1 * 2 = ${v1 * 2}');
  print('Magnitude of v1: ${v1.magnitude}');
  print('v1 == v2: ${v1 == v2}');
  print('v1 > v2: ${v1 > v2}');
}
```

## Best Practices

### 1. Immutability
```dart
class ImmutablePoint {
  final double x;
  final double y;
  
  const ImmutablePoint(this.x, this.y);
  
  // Return new instance instead of modifying
  ImmutablePoint translate(double dx, double dy) {
    return ImmutablePoint(x + dx, y + dy);
  }
  
  @override
  String toString() => 'Point($x, $y)';
}

void demonstrateImmutability() {
  const point1 = ImmutablePoint(1, 2);
  // point1.x = 5; // Error: final field
  
  var point2 = point1.translate(3, 4);
  print('Original: $point1');
  print('Translated: $point2');
}
```

### 2. Proper Encapsulation
```dart
class Temperature {
  double _celsius = 0;
  
  // Public getter
  double get celsius => _celsius;
  
  // Controlled setter
  set celsius(double value) {
    if (value < -273.15) {
      throw ArgumentError('Temperature cannot be below absolute zero');
    }
    _celsius = value;
  }
  
  // Computed properties
  double get fahrenheit => _celsius * 9/5 + 32;
  set fahrenheit(double value) => celsius = (value - 32) * 5/9;
  
  double get kelvin => _celsius + 273.15;
  set kelvin(double value) => celsius = value - 273.15;
}

void demonstrateEncapsulation() {
  Temperature temp = Temperature();
  temp.celsius = 25;
  print('Celsius: ${temp.celsius}°C');
  print('Fahrenheit: ${temp.fahrenheit}°F');
  print('Kelvin: ${temp.kelvin}K');
  
  temp.fahrenheit = 100;
  print('After setting Fahrenheit: ${temp.celsius}°C');
}
```

### 3. Factory Pattern Implementation
```dart
abstract class Shape {
  double get area;
  double get perimeter;
  String get type;
  
  @override
  String toString() => '$type: area=${area.toStringAsFixed(2)}, perimeter=${perimeter.toStringAsFixed(2)}';
}

class Circle implements Shape {
  final double radius;
  
  Circle(this.radius);
  
  @override
  double get area => Math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * Math.pi * radius;
  
  @override
  String get type => 'Circle';
}

class Rectangle implements Shape {
  final double width;
  final double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
  
  @override
  String get type => 'Rectangle';
}

class ShapeFactory {
  static Shape createCircle(double radius) => Circle(radius);
  static Shape createRectangle(double width, double height) => Rectangle(width, height);
  static Shape createSquare(double side) => Rectangle(side, side);
  
  static Shape fromJson(Map<String, dynamic> json) {
    switch (json['type']) {
      case 'circle':
        return Circle(json['radius']);
      case 'rectangle':
        return Rectangle(json['width'], json['height']);
      case 'square':
        return Rectangle(json['side'], json['side']);
      default:
        throw ArgumentError('Unknown shape type: ${json['type']}');
    }
  }
}

void demonstrateFactory() {
  Shape circle = ShapeFactory.createCircle(5);
  Shape rectangle = ShapeFactory.createRectangle(4, 6);
  Shape square = ShapeFactory.createSquare(3);
  
  print(circle);
  print(rectangle);
  print(square);
  
  // From JSON
  Shape jsonShape = ShapeFactory.fromJson({'type': 'circle', 'radius': 7});
  print(jsonShape);
}
```

## Summary

Classes and objects form the foundation of Dart's object-oriented programming:

✅ **Key Concepts:**
- Everything in Dart is an object
- Classes define object structure and behavior
- Constructors control object creation
- Encapsulation protects object state
- Methods define object behavior

⚠️ **Best Practices:**
- Prefer immutability when possible
- Use proper encapsulation with getters/setters
- Implement meaningful toString() methods
- Override == and hashCode consistently
- Use factory constructors for complex instantiation

Understanding classes and objects thoroughly is essential for building robust Dart applications and leveraging the full power of object-oriented programming.
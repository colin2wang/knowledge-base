# Dart Syntax Guide

Dart is a client-optimized language for fast apps on any platform. Developed by Google, it's the primary language for Flutter development and offers modern programming features with clean, readable syntax.

## 🌟 Key Features

- **Object-Oriented**: Everything is an object, including primitives
- **Strongly Typed**: Type safety with optional type annotations
- **Null Safety**: Built-in null safety since Dart 2.12
- **Asynchronous Programming**: Native async/await support
- **Cross-Platform**: Compiles to native code, JavaScript, or WebAssembly
- **Hot Reload**: Fast development cycle for Flutter apps

## Basic Syntax

### Hello World
```dart
void main() {
  print('Hello, World!');
}
```

### Comments
```dart
// Single line comment

/*
  Multi-line comment
  Can span multiple lines
*/

/// Documentation comment
/// Used for generating documentation
```

## Variables and Data Types

### Variable Declaration
```dart
// Explicit type declaration
String name = 'John';
int age = 25;
double height = 5.9;
bool isStudent = true;

// Type inference (var)
var city = 'New York';  // String
var score = 95;         // int
var price = 19.99;      // double

// Dynamic typing
dynamic value = 'text';
value = 123;  // Can change type
value = true; // Can change type again
```

### Final and Const
```dart
// Final - runtime constant (can be calculated at runtime)
final DateTime now = DateTime.now();
final String greeting = 'Hello';

// Const - compile-time constant (must be known at compile time)
const int maxUsers = 100;
const double pi = 3.14159;
const List<String> weekdays = ['Monday', 'Tuesday', 'Wednesday'];
```

### Nullable Types (Null Safety)
```dart
// Non-nullable types (cannot be null)
String name = 'Alice';  // Must have a value

// Nullable types (can be null)
String? middleName;     // Can be null
int? age;               // Can be null

// Null-aware operators
String? message;
String result = message ?? 'Default message';  // Null coalescing
message ??= 'Fallback';                        // Assignment if null
int length = message?.length ?? 0;             // Conditional property access
```

## Data Types

### Numbers
```dart
// Integers
int count = 42;
int hex = 0xDEADBEEF;
int binary = 0b101010;
int octal = 0o755;

// Doubles (64-bit floating point)
double price = 19.99;
double scientific = 1.4e5;

// Number utilities
int.parse('42');           // String to int
double.parse('3.14');      // String to double
42.toString();             // int to String
3.14159.toStringAsFixed(2); // Format to 2 decimal places
```

### Strings
```dart
// String literals
String single = 'Single quotes';
String doubleQuote = "Double quotes";
String multiline = '''
This is a
multi-line string
''';

String raw = r'Raw string \n no escape';

// String interpolation
String name = 'Alice';
int age = 30;
String message = 'Hello, $name! You are ${age + 1} years old.';

// String operations
String text = 'Hello World';
print(text.length);              // 11
print(text.toUpperCase());       // HELLO WORLD
print(text.contains('World'));   // true
print(text.substring(0, 5));     // Hello
print(text.split(' '));          // [Hello, World]
```

### Lists (Arrays)
```dart
// Creating lists
List<String> fruits = ['apple', 'banana', 'orange'];
var numbers = [1, 2, 3, 4, 5];
List<int> emptyList = [];

// List operations
fruits.add('grape');
fruits.insert(1, 'kiwi');
fruits.remove('banana');
print(fruits.length);
print(fruits[0]);
print(fruits.contains('apple'));

// List iteration
for (String fruit in fruits) {
  print(fruit);
}

fruits.forEach((fruit) => print(fruit));

// List comprehension equivalent
var squares = [for (int i = 1; i <= 5; i++) i * i]; // [1, 4, 9, 16, 25]

// Spread operator
List<String> moreFruits = ['mango', 'pineapple'];
List<String> allFruits = [...fruits, ...moreFruits];

// Collection if/for
var activeUsers = [
  'Alice',
  if (showAdmin) 'Admin',
  for (var user in users) if (user.isActive) user.name,
];
```

### Maps (Dictionaries)
```dart
// Creating maps
Map<String, int> ages = {
  'Alice': 30,
  'Bob': 25,
  'Charlie': 35
};

var person = {
  'name': 'John',
  'age': 30,
  'city': 'New York'
};

// Map operations
ages['David'] = 28;        // Add/update
ages.remove('Bob');        // Remove
print(ages.length);        // Size
print(ages.keys);          // Keys iterable
print(ages.values);        // Values iterable
print(ages.containsKey('Alice'));  // Check key existence

// Map iteration
ages.forEach((name, age) {
  print('$name is $age years old');
});

for (var entry in ages.entries) {
  print('${entry.key}: ${entry.value}');
}
```

### Sets
```dart
// Creating sets
Set<String> uniqueNames = {'Alice', 'Bob', 'Charlie'};
var numbers = <int>{1, 2, 3, 4, 5};

// Set operations
uniqueNames.add('David');
uniqueNames.remove('Bob');
print(uniqueNames.length);
print(uniqueNames.contains('Alice'));

// Set mathematics
Set<int> evens = {2, 4, 6, 8};
Set<int> primes = {2, 3, 5, 7};

var union = evens.union(primes);        // {2, 3, 4, 5, 6, 7, 8}
var intersection = evens.intersection(primes); // {2}
var difference = evens.difference(primes);     // {4, 6, 8}
```

## Control Flow

### Conditional Statements
```dart
// If statement
if (age >= 18) {
  print('Adult');
} else if (age >= 13) {
  print('Teenager');
} else {
  print('Child');
}

// Ternary operator
String status = age >= 18 ? 'Adult' : 'Minor';

// Switch statement
switch (grade) {
  case 'A':
    print('Excellent');
    break;
  case 'B':
    print('Good');
    break;
  case 'C':
    print('Average');
    break;
  default:
    print('Needs improvement');
}

// Switch with pattern matching (Dart 3.0+)
switch (shape) {
  case Square(size: var s):
    print('Square with size $s');
  case Circle(radius: var r):
    print('Circle with radius $r');
}
```

### Loops
```dart
// For loop
for (int i = 0; i < 5; i++) {
  print(i);
}

// For-in loop (collections)
List<String> fruits = ['apple', 'banana', 'orange'];
for (String fruit in fruits) {
  print(fruit);
}

// While loop
int i = 0;
while (i < 5) {
  print(i);
  i++;
}

// Do-while loop
int j = 0;
do {
  print(j);
  j++;
} while (j < 5);

// Break and continue
for (int i = 0; i < 10; i++) {
  if (i == 3) continue;  // Skip iteration
  if (i == 7) break;     // Exit loop
  print(i);
}
```

## Functions

### Basic Functions
```dart
// Function with explicit types
int add(int a, int b) {
  return a + b;
}

// Function with inferred return type
multiply(int a, int b) {
  return a * b;
}

// Arrow function (single expression)
int square(int x) => x * x;
String greet(String name) => 'Hello, $name!';

// Function call
int sum = add(5, 3);
int product = multiply(4, 6);
```

### Optional Parameters
```dart
// Optional positional parameters
String greetPerson(String firstName, [String? lastName]) {
  if (lastName != null) {
    return 'Hello, $firstName $lastName!';
  }
  return 'Hello, $firstName!';
}

// Optional named parameters
void createUser(String name, {int? age, String? email}) {
  print('Creating user: $name');
  if (age != null) print('Age: $age');
  if (email != null) print('Email: $email');
}

// Usage
greetPerson('John');           // Hello, John!
greetPerson('John', 'Doe');    // Hello, John Doe!

createUser('Alice', age: 25);
createUser('Bob', email: 'bob@example.com', age: 30);
```

### Default Parameter Values
```dart
// Default values for optional parameters
String greetWithDefault(String name, [String greeting = 'Hello']) {
  return '$greeting, $name!';
}

void connect({String host = 'localhost', int port = 8080}) {
  print('Connecting to $host:$port');
}

// Usage
greetWithDefault('Alice');     // Hello, Alice!
greetWithDefault('Bob', 'Hi'); // Hi, Bob!

connect();                     // Connecting to localhost:8080
connect(host: '192.168.1.1');  // Connecting to 192.168.1.1:8080
```

### Function Types
```dart
// Function as variable
Function addNumbers = (int a, int b) => a + b;
var result = addNumbers(5, 3);

// Function type annotation
int Function(int, int) mathOperation = (a, b) => a * b;

// Higher-order functions
List<int> numbers = [1, 2, 3, 4, 5];
List<int> doubled = numbers.map((n) => n * 2).toList();
List<int> evens = numbers.where((n) => n.isEven).toList();
int sum = numbers.reduce((a, b) => a + b);
```

## Classes and Objects

### Basic Class
```dart
class Person {
  String name;
  int age;
  
  // Constructor
  Person(this.name, this.age);
  
  // Method
  void introduce() {
    print('Hi, I\'m $name and I\'m $age years old.');
  }
  
  // Getter
  bool get isAdult => age >= 18;
  
  // Setter
  set age(int value) {
    if (value >= 0) {
      age = value;
    }
  }
}

// Object creation
Person person = Person('Alice', 25);
person.introduce();
print(person.isAdult);
```

### Named Constructors
```dart
class Rectangle {
  double width;
  double height;
  
  // Default constructor
  Rectangle(this.width, this.height);
  
  // Named constructors
  Rectangle.square(double side) : width = side, height = side;
  Rectangle.fromJson(Map<String, dynamic> json)
      : width = json['width'],
        height = json['height'];
  
  double get area => width * height;
}

// Usage
var rect1 = Rectangle(10, 20);
var square = Rectangle.square(15);
var rect2 = Rectangle.fromJson({'width': 30, 'height': 40});
```

### Inheritance
```dart
class Animal {
  String name;
  
  Animal(this.name);
  
  void makeSound() {
    print('Some generic sound');
  }
}

class Dog extends Animal {
  String breed;
  
  Dog(String name, this.breed) : super(name);
  
  @override
  void makeSound() {
    print('Woof! Woof!');
  }
  
  void fetch() {
    print('$name is fetching the ball!');
  }
}

// Usage
Dog dog = Dog('Buddy', 'Golden Retriever');
dog.makeSound();  // Woof! Woof!
dog.fetch();      // Buddy is fetching the ball!
```

### Abstract Classes
```dart
abstract class Shape {
  // Abstract method (no implementation)
  double getArea();
  
  // Concrete method
  void describe() {
    print('This is a shape with area ${getArea()}');
  }
}

class Circle extends Shape {
  double radius;
  
  Circle(this.radius);
  
  @override
  double getArea() => 3.14159 * radius * radius;
}

class Square extends Shape {
  double side;
  
  Square(this.side);
  
  @override
  double getArea() => side * side;
}

// Usage
Shape circle = Circle(5);
Shape square = Square(4);
circle.describe();  // This is a shape with area 78.53975
square.describe();  // This is a shape with area 16
```

### Mixins
```dart
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
}

class Duck with Flyable, Swimmable {
  String name;
  
  Duck(this.name);
  
  void quack() {
    print('$name says: Quack!');
  }
}

// Usage
Duck duck = Duck('Donald');
duck.fly();    // Flying...
duck.swim();   // Swimming...
duck.quack();  // Donald says: Quack!
```

### Static Members
```dart
class MathUtils {
  // Static constant
  static const double pi = 3.14159;
  
  // Static method
  static double circleArea(double radius) {
    return pi * radius * radius;
  }
  
  // Static variable
  static int counter = 0;
  
  // Instance method accessing static member
  void incrementCounter() {
    MathUtils.counter++;
  }
}

// Usage
print(MathUtils.pi);                           // 3.14159
print(MathUtils.circleArea(5));                // 78.53975
MathUtils.counter = 10;
print(MathUtils.counter);                      // 10
```

## Asynchronous Programming

### Futures
```dart
// Future creation
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Data loaded';
}

// Using futures
void handleFuture() {
  fetchData().then((data) {
    print(data);  // Data loaded
  }).catchError((error) {
    print('Error: $error');
  });
}

// Async/await syntax
Future<void> processData() async {
  try {
    String data = await fetchData();
    print('Processing: $data');
  } catch (error) {
    print('Failed to process: $error');
  }
}
```

### Async/Await
```dart
Future<String> getUserProfile(int userId) async {
  // Simulate API call
  await Future.delayed(Duration(milliseconds: 500));
  return 'Profile for user $userId';
}

Future<String> getUserPosts(int userId) async {
  await Future.delayed(Duration(milliseconds: 300));
  return 'Posts for user $userId';
}

// Sequential async operations
Future<void> loadUserDataSequential(int userId) async {
  String profile = await getUserProfile(userId);
  String posts = await getUserPosts(userId);
  print('Loaded: $profile and $posts');
}

// Parallel async operations
Future<void> loadUserDataParallel(int userId) async {
  var profileFuture = getUserProfile(userId);
  var postsFuture = getUserPosts(userId);
  
  var results = await Future.wait([profileFuture, postsFuture]);
  print('Loaded: ${results[0]} and ${results[1]}');
}
```

### Streams
```dart
// Creating streams
Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

// Using streams
void listenToStream() {
  Stream<int> counter = countStream(5);
  
  counter.listen(
    (value) => print('Count: $value'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Stream completed'),
  );
}

// Stream transformations
Future<void> transformStream() async {
  Stream<int> numbers = Stream.fromIterable([1, 2, 3, 4, 5]);
  
  var doubled = numbers.map((n) => n * 2);
  var evens = doubled.where((n) => n.isEven);
  var sum = await evens.reduce((a, b) => a + b);
  
  print('Sum of doubled evens: $sum');  // 12
}
```

## Generics

### Generic Classes
```dart
class Box<T> {
  T content;
  
  Box(this.content);
  
  T getContent() => content;
  void setContent(T value) => content = value;
}

class Pair<T, U> {
  T first;
  U second;
  
  Pair(this.first, this.second);
  
  @override
  String toString() => '($first, $second)';
}

// Usage
Box<String> stringBox = Box('Hello');
Box<int> intBox = Box(42);
Pair<String, int> user = Pair('Alice', 25);
```

### Generic Methods
```dart
T firstElement<T>(List<T> list) {
  if (list.isEmpty) throw Exception('List is empty');
  return list[0];
}

List<T> reverseList<T>(List<T> list) {
  return list.reversed.toList();
}

// Usage
List<String> names = ['Alice', 'Bob', 'Charlie'];
String first = firstElement(names);  // Alice
List<String> reversed = reverseList(names);  // [Charlie, Bob, Alice]
```

### Bounded Generics
```dart
class NumberBox<T extends num> {
  T value;
  
  NumberBox(this.value);
  
  T add(T other) => value + other as T;
  T multiply(T other) => value * other as T;
}

// Usage
NumberBox<int> intBox = NumberBox(10);
NumberBox<double> doubleBox = NumberBox(3.14);
// NumberBox<String> stringBox = NumberBox('text'); // Error!
```

## Error Handling

### Try-Catch-Finally
```dart
void divideNumbers(int a, int b) {
  try {
    if (b == 0) {
      throw Exception('Division by zero');
    }
    print('Result: ${a / b}');
  } on IntegerDivisionByZeroException {
    print('Cannot divide by zero');
  } on Exception catch (e) {
    print('General exception: $e');
  } catch (e) {
    print('Unexpected error: $e');
  } finally {
    print('Cleanup operations');
  }
}

// Custom exceptions
class ValidationError extends Exception {
  final String message;
  
  ValidationError(this.message);
  
  @override
  String toString() => 'ValidationError: $message';
}

void validateAge(int age) {
  if (age < 0) {
    throw ValidationError('Age cannot be negative');
  }
  if (age > 150) {
    throw ValidationError('Age seems unrealistic');
  }
}
```

## Operators

### Arithmetic Operators
```dart
int a = 10;
int b = 3;

print(a + b);   // 13
print(a - b);   // 7
print(a * b);   // 30
print(a / b);   // 3.333...
print(a ~/ b);  // 3 (integer division)
print(a % b);   // 1 (modulo)
print(a ^ b);   // 1000 (bitwise XOR)
```

### Comparison Operators
```dart
int x = 5;
int y = 10;

print(x == y);  // false
print(x != y);  // true
print(x < y);   // true
print(x > y);   // false
print(x <= y);  // true
print(x >= y);  // false
```

### Logical Operators
```dart
bool isAdult = true;
bool hasLicense = false;

print(isAdult && hasLicense);  // false
print(isAdult || hasLicense);  // true
print(!isAdult);               // false
```

### Cascade Notation
```dart
class Person {
  String? name;
  int? age;
  String? city;
}

// Without cascade
Person person = Person();
person.name = 'Alice';
person.age = 25;
person.city = 'New York';

// With cascade
Person person = Person()
  ..name = 'Alice'
  ..age = 25
  ..city = 'New York';
```

## Enums

### Basic Enums
```dart
enum Color { red, green, blue }

void describeColor(Color color) {
  switch (color) {
    case Color.red:
      print('Red like fire');
      break;
    case Color.green:
      print('Green like grass');
      break;
    case Color.blue:
      print('Blue like sky');
      break;
  }
}

// Usage
Color favoriteColor = Color.blue;
print(favoriteColor.name);  // blue
print(Color.values);        // [Color.red, Color.green, Color.blue]
```

### Enhanced Enums (Dart 2.17+)
```dart
enum HttpStatus {
  ok(200),
  notFound(404),
  internalServerError(500);
  
  const HttpStatus(this.code);
  
  final int code;
  
  bool get isSuccess => code >= 200 && code < 300;
  bool get isError => code >= 400;
  
  @override
  String toString() => '$name ($code)';
}

// Usage
HttpStatus status = HttpStatus.ok;
print(status.code);       // 200
print(status.isSuccess);  // true
print(status);            // ok (200)
```

## Extensions

### Extension Methods
```dart
extension StringExtensions on String {
  String capitalize() {
    if (isEmpty) return this;
    return '${this[0].toUpperCase()}${substring(1).toLowerCase()}';
  }
  
  bool get isNumeric {
    return RegExp(r'^-?\d+$').hasMatch(this);
  }
  
  String truncate(int maxLength) {
    if (length <= maxLength) return this;
    return '${substring(0, maxLength)}...';
  }
}

extension IterableExtensions<T> on Iterable<T> {
  T? get second {
    if (length < 2) return null;
    return elementAt(1);
  }
  
  List<List<T>> chunk(int size) {
    List<List<T>> chunks = [];
    for (int i = 0; i < length; i += size) {
      int end = i + size < length ? i + size : length;
      chunks.add(take(end).skip(i).toList());
    }
    return chunks;
  }
}

// Usage
print('hello'.capitalize());           // Hello
print('123'.isNumeric);                // true
print('very long text'.truncate(8));   // very lon...

List<int> numbers = [1, 2, 3, 4, 5, 6];
print(numbers.second);                 // 2
print(numbers.chunk(2));               // [[1, 2], [3, 4], [5, 6]]
```

## Metadata and Annotations

### Built-in Annotations
```dart
import 'dart:math';

// Deprecated annotation
@deprecated
void oldMethod() {
  print('This method is deprecated');
}

// Override annotation
class Animal {
  void makeSound() {}
}

class Dog extends Animal {
  @override
  void makeSound() {
    print('Woof!');
  }
}

// Required parameters
class User {
  final String name;
  final int age;
  
  const User({
    required this.name,
    required this.age,
  });
}
```

## Libraries and Imports

### Importing Libraries
```dart
// Dart core libraries
import 'dart:math';
import 'dart:io';
import 'dart:convert';

// External packages
import 'package:http/http.dart' as http;
import 'package:path/path.dart' as path;

// Local files
import 'utils.dart';
import '../models/user.dart';

// Selective imports
import 'dart:math' show Random, pi;
import 'dart:math' hide min, max;

// Library prefixes
import 'package:json_annotation/json_annotation.dart' as json;

// Deferred loading
import 'heavy_library.dart' deferred as heavy;

Future<void> useHeavyLibrary() async {
  await heavy.loadLibrary();
  heavy.someFunction();
}
```

## Useful Resources

- [Dart Documentation](https://dart.dev/guides)
- [Dart by Example](https://dart.dev/samples)
- [Effective Dart](https://dart.dev/guides/language/effective-dart)
- [DartPad - Online Editor](https://dartpad.dev/)
- [Flutter Documentation](https://flutter.dev/docs)
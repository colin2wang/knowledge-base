# Dart Abstract Classes

Abstract classes in Dart provide a blueprint for other classes to implement. They define a contract that subclasses must fulfill, enabling polymorphic behavior while ensuring consistent interfaces across related classes.

## What are Abstract Classes?

Abstract classes are classes that cannot be instantiated directly and typically contain one or more abstract methods (methods without implementation) that must be implemented by concrete subclasses.

### Basic Abstract Class Syntax

```dart
// Abstract class - cannot be instantiated
abstract class Shape {
  // Abstract methods - no implementation
  double get area;
  double get perimeter;
  
  // Concrete method - has implementation
  void describe() {
    print('This is a ${runtimeType} with area ${area.toStringAsFixed(2)} and perimeter ${perimeter.toStringAsFixed(2)}');
  }
  
  // Abstract method
  void draw();
  
  // Another concrete method
  bool isLargerThan(Shape other) => area > other.area;
}

// Concrete implementation
class Circle extends Shape {
  final double radius;
  
  Circle(this.radius);
  
  @override
  double get area => 3.14159 * radius * radius;
  
  @override
  double get perimeter => 2 * 3.14159 * radius;
  
  @override
  void draw() {
    print('Drawing a circle with radius $radius');
  }
}

class Rectangle extends Shape {
  final double width;
  final double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
  
  @override
  void draw() {
    print('Drawing a rectangle ${width}x$height');
  }
}

void demonstrateBasicAbstract() {
  print('=== Basic Abstract Class Demo ===');
  
  // Shape shape = Shape(); // Error: Cannot instantiate abstract class
  
  Circle circle = Circle(5);
  Rectangle rectangle = Rectangle(4, 6);
  
  List<Shape> shapes = [circle, rectangle];
  
  for (Shape shape in shapes) {
    shape.describe();
    shape.draw();
    print('---');
  }
  
  print('Circle is larger than rectangle: ${circle.isLargerThan(rectangle)}');
}
```

### Abstract Methods vs Concrete Methods

```dart
abstract class Vehicle {
  String brand;
  int year;
  
  Vehicle(this.brand, this.year);
  
  // Abstract methods - must be implemented
  void start();
  void stop();
  double calculateFuelEfficiency();
  
  // Concrete methods - provide default implementation
  void honk() {
    print('Beep beep!');
  }
  
  void displayInfo() {
    print('Vehicle: $brand ($year)');
  }
  
  // Concrete getter with calculation
  bool get isVintage => year < 1990;
  
  // Abstract getter - must be implemented
  String get fuelType;
}

class Car extends Vehicle {
  int numberOfDoors;
  
  Car(String brand, int year, this.numberOfDoors) : super(brand, year);
  
  @override
  void start() {
    print('$brand car engine started');
  }
  
  @override
  void stop() {
    print('$brand car engine stopped');
  }
  
  @override
  double calculateFuelEfficiency() {
    return 25.0; // MPG
  }
  
  @override
  String get fuelType => 'Gasoline';
  
  // Override concrete method
  @override
  void honk() {
    print('Car horn: Honk honk!');
  }
  
  void openTrunk() {
    print('Trunk opened');
  }
}

class ElectricCar extends Vehicle {
  double batteryCapacity; // kWh
  
  ElectricCar(String brand, int year, this.batteryCapacity) : super(brand, year);
  
  @override
  void start() {
    print('$brand electric motor activated');
  }
  
  @override
  void stop() {
    print('$brand electric motor deactivated');
  }
  
  @override
  double calculateFuelEfficiency() {
    return batteryCapacity * 3.5; // Miles per full charge approximation
  }
  
  @override
  String get fuelType => 'Electricity';
  
  @override
  void honk() {
    print('Electric horn: Beep beep!');
  }
  
  void charge() {
    print('Charging $brand with $batteryCapacity kWh battery');
  }
}

void demonstrateMethodTypes() {
  print('\n=== Abstract vs Concrete Methods Demo ===');
  
  Car car = Car('Toyota', 2020, 4);
  ElectricCar tesla = ElectricCar('Tesla', 2022, 75);
  
  List<Vehicle> vehicles = [car, tesla];
  
  for (Vehicle vehicle in vehicles) {
    vehicle.displayInfo();
    print('Fuel type: ${vehicle.fuelType}');
    print('Vintage: ${vehicle.isVintage}');
    vehicle.start();
    vehicle.honk();
    vehicle.stop();
    print('Fuel efficiency: ${vehicle.calculateFuelEfficiency()}');
    print('---');
  }
  
  // Car-specific method
  car.openTrunk();
  
  // Electric car-specific method
  tesla.charge();
}
```

## Abstract Class Patterns

### Factory Method Pattern

```dart
abstract class DatabaseConnection {
  String connectionString;
  
  DatabaseConnection(this.connectionString);
  
  // Factory method - subclasses implement connection logic
  void connect();
  void disconnect();
  void executeQuery(String query);
  
  // Template method using abstract methods
  void runQuery(String query) {
    connect();
    executeQuery(query);
    disconnect();
  }
  
  // Static factory method
  static DatabaseConnection createConnection(String type, String connectionString) {
    switch (type.toLowerCase()) {
      case 'mysql':
        return MySQLConnection(connectionString);
      case 'postgresql':
        return PostgreSQLConnection(connectionString);
      case 'sqlite':
        return SQLiteConnection(connectionString);
      default:
        throw ArgumentError('Unsupported database type: $type');
    }
  }
}

class MySQLConnection extends DatabaseConnection {
  MySQLConnection(String connectionString) : super(connectionString);
  
  @override
  void connect() {
    print('Connecting to MySQL database: $connectionString');
  }
  
  @override
  void disconnect() {
    print('Disconnecting from MySQL database');
  }
  
  @override
  void executeQuery(String query) {
    print('Executing MySQL query: $query');
  }
}

class PostgreSQLConnection extends DatabaseConnection {
  PostgreSQLConnection(String connectionString) : super(connectionString);
  
  @override
  void connect() {
    print('Connecting to PostgreSQL database: $connectionString');
  }
  
  @override
  void disconnect() {
    print('Disconnecting from PostgreSQL database');
  }
  
  @override
  void executeQuery(String query) {
    print('Executing PostgreSQL query: $query');
  }
}

class SQLiteConnection extends DatabaseConnection {
  SQLiteConnection(String connectionString) : super(connectionString);
  
  @override
  void connect() {
    print('Opening SQLite database: $connectionString');
  }
  
  @override
  void disconnect() {
    print('Closing SQLite database');
  }
  
  @override
  void executeQuery(String query) {
    print('Executing SQLite query: $query');
  }
}

void demonstrateFactoryPattern() {
  print('\n=== Factory Method Pattern Demo ===');
  
  List<String> dbTypes = ['mysql', 'postgresql', 'sqlite'];
  String connectionString = 'localhost:5432/mydb';
  
  for (String type in dbTypes) {
    try {
      DatabaseConnection db = DatabaseConnection.createConnection(type, connectionString);
      print('\n--- $type Database ---');
      db.runQuery('SELECT * FROM users');
    } catch (e) {
      print('Error: $e');
    }
  }
}
```

### Template Method Pattern

```dart
abstract class DataProcessor {
  // Template method - defines algorithm structure
  void processData() {
    print('=== Starting data processing ===');
    loadData();
    validateData();
    transformData();
    saveData();
    cleanup();
    print('=== Data processing completed ===\n');
  }
  
  // Abstract methods - must be implemented by subclasses
  void loadData();
  void transformData();
  void saveData();
  
  // Hook methods - can be overridden
  void validateData() {
    print('Performing basic data validation');
  }
  
  void cleanup() {
    print('Cleaning up resources');
  }
  
  // Concrete method
  void logProgress(String message) {
    print('[PROGRESS] $message');
  }
}

class CSVProcessor extends DataProcessor {
  @override
  void loadData() {
    logProgress('Loading data from CSV file');
    print('Parsing CSV content');
  }
  
  @override
  void transformData() {
    logProgress('Transforming CSV data');
    print('Converting CSV rows to objects');
  }
  
  @override
  void saveData() {
    logProgress('Saving processed data');
    print('Writing to database');
  }
  
  @override
  void validateData() {
    super.validateData();
    print('Performing CSV-specific validation');
    print('Checking column headers and data types');
  }
}

class JSONProcessor extends DataProcessor {
  @override
  void loadData() {
    logProgress('Loading data from JSON source');
    print('Parsing JSON structure');
  }
  
  @override
  void transformData() {
    logProgress('Transforming JSON data');
    print('Mapping JSON objects to domain models');
  }
  
  @override
  void saveData() {
    logProgress('Saving processed data');
    print('Exporting to file system');
  }
  
  @override
  void cleanup() {
    super.cleanup();
    print('Sending processing completion notification');
  }
}

class XMLProcessor extends DataProcessor {
  @override
  void loadData() {
    logProgress('Loading data from XML document');
    print('Parsing XML nodes');
  }
  
  @override
  void transformData() {
    logProgress('Transforming XML data');
    print('Converting XML elements to structured data');
  }
  
  @override
  void saveData() {
    logProgress('Saving processed data');
    print('Storing in cloud storage');
  }
}

void demonstrateTemplateMethod() {
  print('\n=== Template Method Pattern Demo ===');
  
  List<DataProcessor> processors = [
    CSVProcessor(),
    JSONProcessor(),
    XMLProcessor()
  ];
  
  for (var processor in processors) {
    print('--- Processing with ${processor.runtimeType} ---');
    processor.processData();
  }
}
```

## Interface-like Abstract Classes

### Defining Contracts

```dart
// Abstract class serving as interface
abstract class Drawable {
  void draw();
  void resize(double factor);
  
  // Default implementation
  void rotate(double degrees) {
    print('Rotating by $degrees degrees');
  }
  
  // Property that subclasses must implement
  String get shapeType;
}

abstract class Clickable {
  void onClick();
  
  // Optional hook method
  void onDoubleClick() {
    print('Double click detected');
  }
  
  void onMouseEnter() {
    print('Mouse entered');
  }
  
  void onMouseLeave() {
    print('Mouse left');
  }
}

// Implementation combining multiple "interfaces"
class UIButton implements Drawable, Clickable {
  String text;
  double width;
  double height;
  bool isEnabled = true;
  
  UIButton(this.text, this.width, this.height);
  
  @override
  void draw() {
    print('Drawing button: "$text" (${width}x$height)');
    if (!isEnabled) {
      print('Button is disabled');
    }
  }
  
  @override
  void resize(double factor) {
    width *= factor;
    height *= factor;
    print('Resized button to ${width}x$height');
  }
  
  @override
  String get shapeType => 'Rectangle';
  
  @override
  void onClick() {
    if (isEnabled) {
      print('Button "$text" clicked');
    } else {
      print('Cannot click disabled button');
    }
  }
  
  @override
  void onDoubleClick() {
    super.onDoubleClick();
    if (isEnabled) {
      print('Button "$text" double-clicked');
    }
  }
  
  void disable() {
    isEnabled = false;
    print('Button disabled');
  }
  
  void enable() {
    isEnabled = true;
    print('Button enabled');
  }
}

class Icon implements Drawable {
  String iconName;
  double size;
  
  Icon(this.iconName, this.size);
  
  @override
  void draw() {
    print('Drawing icon: $iconName (${size}x$size)');
  }
  
  @override
  void resize(double factor) {
    size *= factor;
    print('Resized icon to ${size}x$size');
  }
  
  @override
  String get shapeType => 'Square';
  
  void changeIcon(String newIconName) {
    iconName = newIconName;
    print('Icon changed to: $iconName');
  }
}

void demonstrateInterfaces() {
  print('\n=== Interface-like Abstract Classes Demo ===');
  
  UIButton button = UIButton('Submit', 100, 40);
  Icon icon = Icon('home', 24);
  
  print('=== Button Operations ===');
  button.draw();
  button.resize(1.2);
  button.onClick();
  button.onDoubleClick();
  button.disable();
  button.onClick();
  
  print('\n=== Icon Operations ===');
  icon.draw();
  icon.resize(1.5);
  icon.changeIcon('settings');
  icon.draw();
  icon.rotate(45);
  
  print('\n=== Polymorphic Usage ===');
  List<Drawable> drawables = [button, icon];
  
  for (var drawable in drawables) {
    print('\nDrawing ${drawable.shapeType}:');
    drawable.draw();
    drawable.resize(0.8);
  }
}
```

### Multiple Interface Implementation

```dart
abstract class Serializable {
  Map<String, dynamic> toJson();
  
  void fromJson(Map<String, dynamic> json) {
    print('Loading from JSON: $json');
  }
  
  String serialize() {
    return toJson().toString();
  }
}

abstract class Validatable {
  bool validate();
  
  List<String> get validationErrors;
  
  bool get isValid => validationErrors.isEmpty;
}

abstract class Cloneable<T> {
  T clone();
}

class User implements Serializable, Validatable, Cloneable<User> {
  String id;
  String name;
  String email;
  int age;
  
  User(this.id, this.name, this.email, this.age);
  
  @override
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'age': age
    };
  }
  
  @override
  void fromJson(Map<String, dynamic> json) {
    super.fromJson(json);
    id = json['id'];
    name = json['name'];
    email = json['email'];
    age = json['age'];
  }
  
  @override
  bool validate() {
    return validationErrors.isEmpty;
  }
  
  @override
  List<String> get validationErrors {
    List<String> errors = [];
    
    if (name.isEmpty) errors.add('Name is required');
    if (email.isEmpty || !email.contains('@')) errors.add('Valid email is required');
    if (age < 0 || age > 150) errors.add('Age must be between 0 and 150');
    
    return errors;
  }
  
  @override
  User clone() {
    return User(id, name, email, age);
  }
  
  @override
  String toString() => 'User(id: $id, name: $name, email: $email, age: $age)';
}

void demonstrateMultipleInterfaces() {
  print('\n=== Multiple Interface Implementation Demo ===');
  
  User user = User('001', 'Alice Johnson', 'alice@example.com', 28);
  
  print('User: $user');
  print('Valid: ${user.isValid}');
  print('Validation errors: ${user.validationErrors}');
  
  print('\nSerializing user:');
  print('JSON: ${user.toJson()}');
  print('Serialized: ${user.serialize()}');
  
  print('\nCloning user:');
  User clonedUser = user.clone();
  print('Original: $user');
  print('Clone: $clonedUser');
  print('Are equal: ${user.toJson() == clonedUser.toJson()}');
  print('Are identical: ${identical(user, clonedUser)}');
  
  print('\nTesting invalid user:');
  User invalidUser = User('', 'Bob', 'invalid-email', -5);
  print('Invalid user: $invalidUser');
  print('Valid: ${invalidUser.isValid}');
  print('Validation errors: ${invalidUser.validationErrors}');
}
```

## Abstract Class Best Practices

### 1. Proper Abstraction Levels

```dart
// Good: Clear abstraction with well-defined responsibilities
abstract class PaymentProcessor {
  String get processorName;
  
  // Abstract methods for core functionality
  void processPayment(double amount);
  bool validatePaymentDetails(Map<String, dynamic> details);
  
  // Concrete utility methods
  String formatAmount(double amount) {
    return '\$${amount.toStringAsFixed(2)}';
  }
  
  void logTransaction(String message) {
    print('[$processorName] $message');
  }
}

abstract class CreditCardProcessor extends PaymentProcessor {
  // Additional credit card specific methods
  bool validateCardNumber(String cardNumber);
  bool validateCVV(String cvv);
  bool validateExpiry(String expiryDate);
  
  @override
  bool validatePaymentDetails(Map<String, dynamic> details) {
    return validateCardNumber(details['cardNumber']) &&
           validateCVV(details['cvv']) &&
           validateExpiry(details['expiryDate']);
  }
}

class StripeProcessor extends CreditCardProcessor {
  @override
  String get processorName => 'Stripe';
  
  @override
  void processPayment(double amount) {
    logTransaction('Processing payment of ${formatAmount(amount)}');
    // Stripe-specific implementation
  }
  
  @override
  bool validateCardNumber(String cardNumber) {
    // Luhn algorithm validation
    return cardNumber.length >= 13 && cardNumber.length <= 19;
  }
  
  @override
  bool validateCVV(String cvv) {
    return cvv.length == 3 || cvv.length == 4;
  }
  
  @override
  bool validateExpiry(String expiryDate) {
    // MM/YY format validation
    return RegExp(r'^(0[1-9]|1[0-2])\/\d{2}$').hasMatch(expiryDate);
  }
}

// Bad: Too many responsibilities in abstract class
abstract class KitchenSinkProcessor {
  void processPayment(double amount);
  void validateCreditCard(String cardNumber);
  void validateBankAccount(String accountNumber);
  void validatePayPal(String email);
  void sendEmailReceipt(String email);
  void generateInvoice();
  void updateInventory();
  void notifyShipping();
  // Too broad and unfocused!
}

void demonstrateGoodAbstraction() {
  print('\n=== Good Abstraction Demo ===');
  
  PaymentProcessor processor = StripeProcessor();
  
  print('Processor: ${processor.processorName}');
  print('Formatted amount: ${processor.formatAmount(123.45)}');
  
  Map<String, dynamic> paymentDetails = {
    'cardNumber': '4242424242424242',
    'cvv': '123',
    'expiryDate': '12/25'
  };
  
  if (processor.validatePaymentDetails(paymentDetails)) {
    processor.processPayment(99.99);
  } else {
    print('Invalid payment details');
  }
}
```

### 2. Abstract Class Hierarchy Design

```dart
// Base abstract class
abstract class FileSystemItem {
  String name;
  DateTime createdAt;
  
  FileSystemItem(this.name) : createdAt = DateTime.now();
  
  // Abstract methods
  String get fullPath;
  int get size;
  void delete();
  
  // Concrete methods
  bool get exists => true;
  
  void rename(String newName) {
    print('Renaming $name to $newName');
    name = newName;
  }
  
  @override
  String toString() => '$runtimeType: $name';
}

// Intermediate abstract class
abstract class DirectoryItem extends FileSystemItem {
  DirectoryItem(String name) : super(name);
  
  @override
  String get fullPath => '/$name';
  
  @override
  void delete() {
    print('Deleting directory item: $name');
  }
}

// Concrete implementations
class File extends DirectoryItem {
  final String content;
  final String extension;
  
  File(String name, this.content, this.extension) : super(name);
  
  @override
  int get size => content.length;
  
  String get fileName => '$name.$extension';
  
  @override
  String get fullPath => super.fullPath + '.$extension';
  
  void editContent(String newContent) {
    print('Editing file content');
    // In real implementation, would modify content
  }
}

class ImageFile extends File {
  final int width;
  final int height;
  final String format;
  
  ImageFile(String name, String content, this.width, this.height, this.format)
      : super(name, content, format.toLowerCase());
  
  @override
  int get size => super.size + (width * height * 3); // Approximate size
  
  void resize(int newWidth, int newHeight) {
    print('Resizing image from ${width}x$height to ${newWidth}x$newHeight');
    // In real implementation, would resize image
  }
}

class Folder extends DirectoryItem {
  final List<FileSystemItem> children = [];
  
  Folder(String name) : super(name);
  
  @override
  int get size => children.fold(0, (sum, item) => sum + item.size);
  
  void addItem(FileSystemItem item) {
    children.add(item);
    print('Added ${item.name} to folder $name');
  }
  
  void removeItem(FileSystemItem item) {
    children.remove(item);
    print('Removed ${item.name} from folder $name');
  }
  
  FileSystemItem? findItem(String itemName) {
    return children.firstWhere(
      (item) => item.name == itemName,
      orElse: () => throw Exception('Item not found: $itemName'),
    );
  }
}

void demonstrateHierarchy() {
  print('\n=== Abstract Class Hierarchy Demo ===');
  
  Folder root = Folder('root');
  Folder documents = Folder('documents');
  Folder images = Folder('images');
  
  File readme = File('README', 'Project documentation', 'md');
  ImageFile logo = ImageFile('logo', 'image_data', 100, 100, 'PNG');
  ImageFile banner = ImageFile('banner', 'large_image_data', 800, 200, 'JPG');
  
  documents.addItem(readme);
  images.addItem(logo);
  images.addItem(banner);
  
  root.addItem(documents);
  root.addItem(images);
  
  print('Root folder size: ${root.size} bytes');
  print('Documents folder size: ${documents.size} bytes');
  print('Images folder size: ${images.size} bytes');
  
  print('\nFolder structure:');
  print(root);
  for (var child in root.children) {
    print('  └── $child (size: ${child.size})');
    if (child is Folder) {
      for (var grandchild in child.children) {
        print('      └── $grandchild (size: ${grandchild.size})');
      }
    }
  }
}
```

### 3. Abstract Factory Pattern

```dart
abstract class UIFactory {
  Button createButton();
  TextField createTextField();
  Dialog createDialog();
  
  Widget createWidget(String type) {
    switch (type.toLowerCase()) {
      case 'button':
        return createButton();
      case 'textfield':
        return createTextField();
      case 'dialog':
        return createDialog();
      default:
        throw ArgumentError('Unknown widget type: $type');
    }
  }
}

abstract class Widget {
  String id;
  double width;
  double height;
  
  Widget(this.id, this.width, this.height);
  
  void render();
  void dispose();
  
  @override
  String toString() => '${runtimeType}(id: $id, ${width}x$height)';
}

abstract class Button extends Widget {
  String text;
  bool isEnabled;
  
  Button(String id, double width, double height, this.text, this.isEnabled)
      : super(id, width, height);
  
  void click();
}

abstract class TextField extends Widget {
  String placeholder;
  String value;
  
  TextField(String id, double width, double height, this.placeholder, this.value)
      : super(id, width, height);
  
  void onFocus();
  void onBlur();
}

abstract class Dialog extends Widget {
  String title;
  bool isVisible;
  
  Dialog(String id, double width, double height, this.title, this.isVisible)
      : super(id, width, height);
  
  void show();
  void hide();
}

// Material Design implementation
class MaterialUIFactory extends UIFactory {
  @override
  Button createButton() => MaterialButton('mat-btn-1', 120, 40, 'Click Me', true);
  
  @override
  TextField createTextField() => MaterialTextField('mat-tf-1', 200, 40, 'Enter text', '');
  
  @override
  Dialog createDialog() => MaterialDialog('mat-dlg-1', 300, 200, 'Confirm', false);
}

class MaterialButton extends Button {
  MaterialButton(String id, double width, double height, String text, bool isEnabled)
      : super(id, width, height, text, isEnabled);
  
  @override
  void render() {
    print('Rendering Material button: $text');
  }
  
  @override
  void click() {
    if (isEnabled) {
      print('Material button clicked: $text');
    } else {
      print('Material button is disabled');
    }
  }
  
  @override
  void dispose() {
    print('Disposing Material button: $id');
  }
}

class MaterialTextField extends TextField {
  MaterialTextField(String id, double width, double height, String placeholder, String value)
      : super(id, width, height, placeholder, value);
  
  @override
  void render() {
    print('Rendering Material text field: $placeholder');
  }
  
  @override
  void onFocus() {
    print('Material text field focused');
  }
  
  @override
  void onBlur() {
    print('Material text field blurred');
  }
  
  @override
  void dispose() {
    print('Disposing Material text field: $id');
  }
}

class MaterialDialog extends Dialog {
  MaterialDialog(String id, double width, double height, String title, bool isVisible)
      : super(id, width, height, title, isVisible);
  
  @override
  void render() {
    print('Rendering Material dialog: $title');
  }
  
  @override
  void show() {
    isVisible = true;
    print('Showing Material dialog: $title');
  }
  
  @override
  void hide() {
    isVisible = false;
    print('Hiding Material dialog: $title');
  }
  
  @override
  void dispose() {
    print('Disposing Material dialog: $id');
  }
}

void demonstrateAbstractFactory() {
  print('\n=== Abstract Factory Pattern Demo ===');
  
  UIFactory factory = MaterialUIFactory();
  
  // Create widgets using factory methods
  Button button = factory.createButton();
  TextField textField = factory.createTextField();
  Dialog dialog = factory.createDialog();
  
  print('Created widgets:');
  print(button);
  print(textField);
  print(dialog);
  
  print('\nRendering widgets:');
  button.render();
  textField.render();
  dialog.render();
  
  print('\nInteracting with widgets:');
  button.click();
  textField.onFocus();
  dialog.show();
  
  print('\nUsing generic factory method:');
  Widget genericButton = factory.createWidget('button');
  Widget genericDialog = factory.createWidget('dialog');
  
  print('Generic creations: $genericButton, $genericDialog');
}
```

## Summary

Abstract classes are fundamental to object-oriented design in Dart:

✅ **Key Benefits:**
- Define contracts that subclasses must implement
- Enable polymorphic behavior
- Provide partial implementations through concrete methods
- Support template method patterns
- Serve as interfaces in Dart's single-inheritance model

⚠️ **Best Practices:**
- Keep abstract classes focused on specific domains
- Use abstract methods for required functionality
- Provide sensible defaults with concrete methods
- Design clear inheritance hierarchies
- Consider using factory methods for object creation

Abstract classes form the backbone of flexible, maintainable Dart applications, particularly in Flutter development where they're extensively used for widget frameworks and architectural patterns.
# Dart Inheritance and Polymorphism

Inheritance and polymorphism are fundamental concepts in object-oriented programming that enable code reusability, extensibility, and flexible design patterns. Dart supports single inheritance with powerful polymorphic capabilities.

## Inheritance Fundamentals

### Basic Inheritance Syntax

```dart
// Parent class (Superclass)
class Vehicle {
  String brand;
  int year;
  bool isRunning = false;
  
  Vehicle(this.brand, this.year);
  
  void start() {
    isRunning = true;
    print('$brand vehicle started');
  }
  
  void stop() {
    isRunning = false;
    print('$brand vehicle stopped');
  }
  
  void honk() {
    print('Beep beep!');
  }
  
  @override
  String toString() => 'Vehicle(brand: $brand, year: $year, running: $isRunning)';
}

// Child class (Subclass)
class Car extends Vehicle {
  int numberOfDoors;
  
  // Constructor calling superclass constructor
  Car(String brand, int year, this.numberOfDoors) : super(brand, year);
  
  // Override parent method
  @override
  void honk() {
    print('Car horn: Honk honk!');
  }
  
  // Add new method
  void openTrunk() {
    print('Trunk opened');
  }
  
  @override
  String toString() => '${super.toString()}, doors: $numberOfDoors';
}

// Another child class
class Motorcycle extends Vehicle {
  bool hasSidecar;
  
  Motorcycle(String brand, int year, this.hasSidecar) : super(brand, year);
  
  @override
  void honk() {
    print('Motorcycle horn: Vroom!');
  }
  
  void wheelie() {
    if (isRunning) {
      print('Performing wheelie!');
    } else {
      print('Start the motorcycle first!');
    }
  }
  
  @override
  String toString() => '${super.toString()}, sidecar: $hasSidecar';
}

void demonstrateBasicInheritance() {
  print('=== Basic Inheritance Demo ===');
  
  Car car = Car('Toyota', 2020, 4);
  Motorcycle bike = Motorcycle('Harley', 2019, false);
  
  print(car);
  print(bike);
  
  car.start();
  car.honk();
  car.openTrunk();
  
  bike.start();
  bike.honk();
  bike.wheelie();
}
```

### Constructor Chaining and Super Calls

```dart
class Animal {
  String name;
  int age;
  
  Animal(this.name, this.age) {
    print('Animal constructor: $name, age $age');
  }
  
  Animal.named(String name) : this(name, 0) {
    print('Named Animal constructor: $name');
  }
  
  void makeSound() {
    print('Some generic animal sound');
  }
}

class Dog extends Animal {
  String breed;
  String color;
  
  // Full constructor chaining
  Dog(String name, int age, this.breed, this.color) : super(name, age) {
    print('Dog constructor: $name, breed $breed');
  }
  
  // Named constructor with super call
  Dog.puppy(String name, String breed) 
      : breed = breed,
        color = 'Various',
        super.named(name) {
    age = 0; // Can modify inherited fields
    print('Puppy constructor: $name');
  }
  
  // Redirecting constructor
  Dog.stray(String name) : this(name, 1, 'Mixed', 'Brown');
  
  @override
  void makeSound() {
    print('Woof woof!');
  }
  
  void fetch() {
    print('$name is fetching the ball');
  }
}

void demonstrateConstructorChaining() {
  print('\n=== Constructor Chaining Demo ===');
  
  Dog dog1 = Dog('Buddy', 3, 'Golden Retriever', 'Golden');
  Dog puppy = Dog.puppy('Max', 'Labrador');
  Dog stray = Dog.stray('Rocky');
  
  dog1.makeSound();
  puppy.makeSound();
  stray.makeSound();
}
```

## Method Overriding and Polymorphism

### Runtime Polymorphism

```dart
class Shape {
  double get area => 0;
  double get perimeter => 0;
  
  void draw() {
    print('Drawing a generic shape');
  }
  
  @override
  String toString() => '${runtimeType}: area=${area.toStringAsFixed(2)}';
}

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

class Triangle extends Shape {
  final double base;
  final double height;
  final double sideA;
  final double sideB;
  
  Triangle(this.base, this.height, this.sideA, this.sideB);
  
  @override
  double get area => 0.5 * base * height;
  
  @override
  double get perimeter => base + sideA + sideB;
  
  @override
  void draw() {
    print('Drawing a triangle with base $base and height $height');
  }
}

void demonstratePolymorphism() {
  print('\n=== Polymorphism Demo ===');
  
  // Polymorphic collection
  List<Shape> shapes = [
    Circle(5),
    Rectangle(4, 6),
    Triangle(3, 4, 3, 5)
  ];
  
  // Runtime polymorphic behavior
  for (Shape shape in shapes) {
    print(shape);
    shape.draw();
    print('Area: ${shape.area.toStringAsFixed(2)}');
    print('Perimeter: ${shape.perimeter.toStringAsFixed(2)}');
    print('---');
  }
}

// Polymorphic function
void processShape(Shape shape) {
  print('Processing ${shape.runtimeType}');
  shape.draw();
  print('Calculated area: ${shape.area}');
}

void demonstratePolymorphicFunctions() {
  print('\n=== Polymorphic Functions Demo ===');
  
  Circle circle = Circle(3);
  Rectangle rect = Rectangle(2, 4);
  
  processShape(circle);
  print('---');
  processShape(rect);
}
```

### The `super` Keyword

```dart
class Employee {
  String name;
  double baseSalary;
  
  Employee(this.name, this.baseSalary);
  
  double calculateSalary() {
    return baseSalary;
  }
  
  void displayInfo() {
    print('Employee: $name, Base Salary: \$${baseSalary.toStringAsFixed(2)}');
  }
  
  String getEmployeeType() => 'Regular Employee';
}

class Manager extends Employee {
  double bonus;
  List<String> teamMembers;
  
  Manager(String name, double baseSalary, this.bonus, this.teamMembers) 
      : super(name, baseSalary);
  
  // Override with super call
  @override
  double calculateSalary() {
    double total = super.calculateSalary() + bonus;
    print('Manager salary calculation: Base (\$${baseSalary}) + Bonus (\$${bonus}) = \$${total}');
    return total;
  }
  
  @override
  void displayInfo() {
    super.displayInfo(); // Call parent method
    print('Bonus: \$${bonus.toStringAsFixed(2)}');
    print('Team size: ${teamMembers.length} members');
    print('Team members: ${teamMembers.join(', ')}');
  }
  
  @override
  String getEmployeeType() => '${super.getEmployeeType()} - Manager';
  
  // New method using inherited fields
  void conductMeeting() {
    print('$name is conducting a meeting with ${teamMembers.length} team members');
  }
}

class Developer extends Employee {
  String programmingLanguage;
  int experienceYears;
  
  Developer(String name, double baseSalary, this.programmingLanguage, this.experienceYears)
      : super(name, baseSalary);
  
  @override
  double calculateSalary() {
    // Adjust salary based on experience
    double experienceMultiplier = 1.0 + (experienceYears * 0.05);
    double adjustedSalary = super.calculateSalary() * experienceMultiplier;
    print('Developer salary: Base (\$${baseSalary}) × Experience (${experienceMultiplier.toStringAsFixed(2)}) = \$${adjustedSalary.toStringAsFixed(2)}');
    return adjustedSalary;
  }
  
  @override
  String getEmployeeType() => '${super.getEmployeeType()} - Developer';
  
  void code() {
    print('$name is coding in $programmingLanguage');
  }
}

void demonstrateSuperKeyword() {
  print('\n=== Super Keyword Demo ===');
  
  Manager manager = Manager('Alice Johnson', 80000, 20000, ['Bob', 'Charlie', 'Diana']);
  Developer developer = Developer('Bob Smith', 70000, 'Dart', 3);
  
  print('=== Manager Info ===');
  manager.displayInfo();
  print('Total Salary: \$${manager.calculateSalary().toStringAsFixed(2)}');
  print('Type: ${manager.getEmployeeType()}');
  manager.conductMeeting();
  
  print('\n=== Developer Info ===');
  developer.displayInfo();
  print('Total Salary: \$${developer.calculateSalary().toStringAsFixed(2)}');
  print('Type: ${developer.getEmployeeType()}');
  developer.code();
}
```

## Abstract Classes and Interfaces

### Abstract Classes

```dart
// Abstract class - cannot be instantiated
abstract class Animal {
  String name;
  int age;
  
  Animal(this.name, this.age);
  
  // Abstract method - must be implemented by subclasses
  void makeSound();
  
  // Concrete method - can be inherited
  void sleep() {
    print('$name is sleeping');
  }
  
  // Another concrete method
  void eat(String food) {
    print('$name is eating $food');
  }
  
  // Abstract getter
  String get species;
  
  // Concrete getter
  bool get isAdult => age >= 2;
}

class Dog extends Animal {
  String breed;
  
  Dog(String name, int age, this.breed) : super(name, age);
  
  @override
  void makeSound() {
    print('Woof! Woof!');
  }
  
  @override
  String get species => 'Canis lupus familiaris';
  
  void fetch() {
    print('$name is fetching the ball');
  }
}

class Cat extends Animal {
  bool isIndoor;
  
  Cat(String name, int age, this.isIndoor) : super(name, age);
  
  @override
  void makeSound() {
    print('Meow!');
  }
  
  @override
  String get species => 'Felis catus';
  
  void purr() {
    print('$name is purring contentedly');
  }
}

void demonstrateAbstractClasses() {
  print('\n=== Abstract Classes Demo ===');
  
  // Animal animal = Animal('Generic', 1); // Error: Cannot instantiate abstract class
  
  Dog dog = Dog('Buddy', 3, 'Golden Retriever');
  Cat cat = Cat('Whiskers', 2, true);
  
  List<Animal> animals = [dog, cat];
  
  for (Animal animal in animals) {
    print('${animal.name} (${animal.species}):');
    animal.makeSound();
    animal.eat('food');
    animal.sleep();
    print('Is adult: ${animal.isAdult}');
    print('---');
  }
}
```

### Interface-like Behavior

```dart
// Dart doesn't have explicit interfaces, but classes can serve as interfaces
class Drawable {
  void draw() {
    throw UnimplementedError('draw() must be implemented');
  }
  
  void resize(double factor) {
    throw UnimplementedError('resize() must be implemented');
  }
}

class Clickable {
  void onClick() {
    throw UnimplementedError('onClick() must be implemented');
  }
  
  void onDoubleClick() {
    print('Double click detected');
  }
}

class UIComponent implements Drawable, Clickable {
  String id;
  double width;
  double height;
  
  UIComponent(this.id, this.width, this.height);
  
  @override
  void draw() {
    print('Drawing component $id (${width}x$height)');
  }
  
  @override
  void resize(double factor) {
    width *= factor;
    height *= factor;
    print('Resized component $id to ${width}x$height');
  }
  
  @override
  void onClick() {
    print('Component $id clicked');
  }
  
  @override
  void onDoubleClick() {
    super.onDoubleClick(); // Call default implementation
    print('Component $id double-clicked');
  }
}

// Extending functionality through interfaces
class Button extends UIComponent {
  String text;
  bool isEnabled = true;
  
  Button(String id, double width, double height, this.text) 
      : super(id, width, height);
  
  @override
  void draw() {
    super.draw();
    print('Button text: "$text"');
    print('Enabled: $isEnabled');
  }
  
  @override
  void onClick() {
    if (isEnabled) {
      super.onClick();
      print('Button action executed');
    } else {
      print('Button is disabled');
    }
  }
}

void demonstrateInterfaces() {
  print('\n=== Interface-like Behavior Demo ===');
  
  UIComponent component = UIComponent('comp1', 100, 50);
  Button button = Button('btn1', 120, 40, 'Click Me');
  
  print('=== Generic Component ===');
  component.draw();
  component.resize(1.5);
  component.onClick();
  component.onDoubleClick();
  
  print('\n=== Button Component ===');
  button.draw();
  button.resize(0.8);
  button.onClick();
  button.onDoubleClick();
}
```

## Advanced Inheritance Patterns

### Template Method Pattern

```dart
abstract class DataProcessor {
  // Template method - defines algorithm structure
  void processData() {
    loadData();
    validateData();
    transformData();
    saveData();
    notifyCompletion();
  }
  
  // Abstract methods to be implemented by subclasses
  void loadData();
  void transformData();
  void saveData();
  
  // Hook methods with default implementations
  void validateData() {
    print('Performing basic data validation');
  }
  
  void notifyCompletion() {
    print('Data processing completed');
  }
}

class CSVProcessor extends DataProcessor {
  @override
  void loadData() {
    print('Loading data from CSV file');
  }
  
  @override
  void transformData() {
    print('Transforming CSV data');
  }
  
  @override
  void saveData() {
    print('Saving processed data to database');
  }
  
  // Override hook method
  @override
  void validateData() {
    super.validateData();
    print('Performing CSV-specific validation');
  }
}

class JSONProcessor extends DataProcessor {
  @override
  void loadData() {
    print('Loading data from JSON API');
  }
  
  @override
  void transformData() {
    print('Transforming JSON data');
  }
  
  @override
  void saveData() {
    print('Saving processed data to file');
  }
  
  @override
  void notifyCompletion() {
    super.notifyCompletion();
    print('Sending notification email');
  }
}

void demonstrateTemplateMethod() {
  print('\n=== Template Method Pattern Demo ===');
  
  DataProcessor csvProcessor = CSVProcessor();
  DataProcessor jsonProcessor = JSONProcessor();
  
  print('Processing CSV data:');
  csvProcessor.processData();
  
  print('\nProcessing JSON data:');
  jsonProcessor.processData();
}
```

### Strategy Pattern with Inheritance

```dart
abstract class SortStrategy {
  void sort(List<int> data);
  
  String get name;
}

class BubbleSort extends SortStrategy {
  @override
  void sort(List<int> data) {
    print('Sorting using Bubble Sort');
    // Simplified bubble sort implementation
    for (int i = 0; i < data.length - 1; i++) {
      for (int j = 0; j < data.length - i - 1; j++) {
        if (data[j] > data[j + 1]) {
          int temp = data[j];
          data[j] = data[j + 1];
          data[j + 1] = temp;
        }
      }
    }
  }
  
  @override
  String get name => 'Bubble Sort';
}

class QuickSort extends SortStrategy {
  @override
  void sort(List<int> data) {
    print('Sorting using Quick Sort');
    _quickSort(data, 0, data.length - 1);
  }
  
  void _quickSort(List<int> arr, int low, int high) {
    if (low < high) {
      int pi = _partition(arr, low, high);
      _quickSort(arr, low, pi - 1);
      _quickSort(arr, pi + 1, high);
    }
  }
  
  int _partition(List<int> arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
      if (arr[j] < pivot) {
        i++;
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
      }
    }
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    return i + 1;
  }
  
  @override
  String get name => 'Quick Sort';
}

class Sorter {
  SortStrategy strategy;
  
  Sorter(this.strategy);
  
  void setStrategy(SortStrategy newStrategy) {
    strategy = newStrategy;
  }
  
  void sortData(List<int> data) {
    print('Using ${strategy.name}');
    List<int> copy = List.from(data);
    strategy.sort(copy);
    print('Sorted data: $copy');
  }
}

void demonstrateStrategyPattern() {
  print('\n=== Strategy Pattern Demo ===');
  
  List<int> data = [64, 34, 25, 12, 22, 11, 90];
  print('Original data: $data');
  
  Sorter sorter = Sorter(BubbleSort());
  sorter.sortData(data);
  
  print('\nSwitching to Quick Sort:');
  sorter.setStrategy(QuickSort());
  sorter.sortData(data);
}
```

## Inheritance Best Practices

### 1. Favor Composition Over Inheritance

```dart
// Bad: Deep inheritance hierarchy
class Bird {
  void fly() => print('Flying');
  void eat() => print('Eating');
}

class Eagle extends Bird {
  void hunt() => print('Hunting prey');
}

class Penguin extends Bird {
  @override
  void fly() {
    // Penguins can't fly - violates Liskov substitution
    print('Cannot fly');
  }
  
  void swim() => print('Swimming');
}

// Good: Composition approach
class Flyable {
  void fly() => print('Flying');
}

class Swimmable {
  void swim() => print('Swimming');
}

class Edible {
  void eat() => print('Eating');
}

class Hunter {
  void hunt() => print('Hunting prey');
}

class Eagle2 {
  final Flyable flyable = Flyable();
  final Edible edible = Edible();
  final Hunter hunter = Hunter();
  
  void fly() => flyable.fly();
  void eat() => edible.eat();
  void hunt() => hunter.hunt();
}

class Penguin2 {
  final Swimmable swimmable = Swimmable();
  final Edible edible = Edible();
  
  void swim() => swimmable.swim();
  void eat() => edible.eat();
  // No fly method - clearer design
}

void demonstrateComposition() {
  print('\n=== Composition vs Inheritance Demo ===');
  
  print('Eagle (composition):');
  Eagle2 eagle = Eagle2();
  eagle.fly();
  eagle.eat();
  eagle.hunt();
  
  print('\nPenguin (composition):');
  Penguin2 penguin = Penguin2();
  penguin.swim();
  penguin.eat();
  // penguin.fly(); // Compile error - no such method
}
```

### 2. Proper Use of `@override` Annotation

```dart
class BaseClass {
  void existingMethod() => print('Base method');
  
  String get property => 'Base property';
  
  void deprecatedMethod() => print('Deprecated');
}

class DerivedClass extends BaseClass {
  // Good: Override existing method
  @override
  void existingMethod() {
    super.existingMethod();
    print('Extended functionality');
  }
  
  // Good: Override getter
  @override
  String get property => '${super.property} (modified)';
  
  // Warning: No @override for new method
  void newMethod() => print('New method');
  
  // Error: @override for non-existent method
  // @override
  // void nonExistentMethod() => print('This will cause warning');
}

void demonstrateOverrideAnnotation() {
  print('\n=== Override Annotation Demo ===');
  
  DerivedClass obj = DerivedClass();
  obj.existingMethod();
  print('Property: ${obj.property}');
  obj.newMethod();
}
```

### 3. Liskov Substitution Principle

```dart
abstract class Rectangle {
  double width;
  double height;
  
  Rectangle(this.width, this.height);
  
  double get area => width * height;
  double get perimeter => 2 * (width + height);
  
  void setWidth(double value) => width = value;
  void setHeight(double value) => height = value;
}

class Square extends Rectangle {
  Square(double side) : super(side, side);
  
  // Violates LSP - changes behavior of parent methods
  @override
  void setWidth(double value) {
    super.setWidth(value);
    super.setHeight(value); // Forces square property
  }
  
  @override
  void setHeight(double value) {
    super.setHeight(value);
    super.setWidth(value); // Forces square property
  }
}

// Better approach - separate classes
class ProperSquare {
  double side;
  
  ProperSquare(this.side);
  
  double get area => side * side;
  double get perimeter => 4 * side;
  
  void setSide(double value) => side = value;
}

void demonstrateLSP() {
  print('\n=== Liskov Substitution Principle Demo ===');
  
  // Problematic inheritance
  Rectangle rect = Square(5);
  rect.setWidth(10);
  print('Square area after setWidth(10): ${rect.area}'); // Expected 100, but gets 100 (correct in this case)
  
  // Better approach
  ProperSquare square = ProperSquare(5);
  square.setSide(10);
  print('ProperSquare area: ${square.area}'); // Clearly 100
}
```

## Type Checking and Casting

### Runtime Type Information

```dart
class Animal {
  String name;
  Animal(this.name);
  void speak() => print('Animal sound');
}

class Dog extends Animal {
  String breed;
  Dog(String name, this.breed) : super(name);
  @override
  void speak() => print('Woof!');
  void fetch() => print('$name is fetching');
}

class Cat extends Animal {
  bool isIndoor;
  Cat(String name, this.isIndoor) : super(name);
  @override
  void speak() => print('Meow!');
  void purr() => print('$name is purring');
}

void demonstrateTypeChecking(Animal animal) {
  print('\n=== Type Checking Demo ===');
  print('Animal: ${animal.name}');
  
  // Type checking
  if (animal is Dog) {
    print('This is a dog');
    animal.speak();
    animal.fetch(); // Safe to call Dog-specific method
  } else if (animal is Cat) {
    print('This is a cat');
    animal.speak();
    animal.purr(); // Safe to call Cat-specific method
  } else {
    print('This is a generic animal');
    animal.speak();
  }
  
  // Type casting
  if (animal.runtimeType == Dog) {
    Dog dog = animal as Dog; // Explicit cast
    print('Dog breed: ${dog.breed}');
  }
  
  // Safe casting with check
  if (animal is Dog) {
    Dog dog = animal; // Implicit cast due to type check
    print('Accessing breed: ${dog.breed}');
  }
}

void demonstrateCollectionTyping() {
  print('\n=== Collection Type Checking Demo ===');
  
  List<Animal> animals = [
    Dog('Buddy', 'Golden Retriever'),
    Cat('Whiskers', true),
    Animal('Generic Animal')
  ];
  
  for (var animal in animals) {
    print('\nProcessing: ${animal.runtimeType}');
    
    // Pattern matching approach
    switch (animal.runtimeType) {
      case Dog:
        (animal as Dog).fetch();
        break;
      case Cat:
        (animal as Cat).purr();
        break;
      default:
        print('Generic animal behavior');
    }
  }
}

void main() {
  demonstrateBasicInheritance();
  demonstrateConstructorChaining();
  demonstratePolymorphism();
  demonstratePolymorphicFunctions();
  demonstrateSuperKeyword();
  demonstrateAbstractClasses();
  demonstrateInterfaces();
  demonstrateTemplateMethod();
  demonstrateStrategyPattern();
  demonstrateComposition();
  demonstrateOverrideAnnotation();
  demonstrateLSP();
  
  // Type checking demos
  demonstrateTypeChecking(Dog('Rex', 'German Shepherd'));
  demonstrateTypeChecking(Cat('Fluffy', false));
  demonstrateTypeChecking(Animal('Generic'));
  demonstrateCollectionTyping();
}
```

## Summary

Inheritance and polymorphism are powerful OOP concepts in Dart:

✅ **Key Benefits:**
- Code reusability through inheritance hierarchies
- Runtime polymorphic behavior
- Flexible and extensible designs
- Clear separation of concerns

⚠️ **Important Considerations:**
- Dart supports single inheritance only
- Use composition over deep inheritance hierarchies
- Properly override methods with `@override` annotation
- Follow the Liskov Substitution Principle
- Leverage runtime type checking when needed

Mastering inheritance and polymorphism enables developers to create maintainable, scalable Dart applications with clean architectural patterns.
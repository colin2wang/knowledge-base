# Inheritance

Inheritance is a fundamental object-oriented programming concept that allows one class to acquire the properties and methods of another class. It promotes code reusability and establishes hierarchical relationships between classes.

## Basic Inheritance

### extends Keyword

```java
// Parent class (Superclass)
public class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

// Child class (Subclass)
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age); // Call parent constructor
        this.breed = breed;
    }
    
    public void bark() {
        System.out.println(name + " is barking");
    }
    
    // Method overriding
    @Override
    public void eat() {
        System.out.println(name + " the " + breed + " is eating dog food");
    }
}

// Usage
Dog myDog = new Dog("Buddy", 3, "Golden Retriever");
myDog.eat();   // Output: Buddy the Golden Retriever is eating dog food
myDog.sleep(); // Output: Buddy is sleeping
myDog.bark();  // Output: Buddy is barking
```

## Types of Inheritance

### 1. Single Inheritance
```java
public class Vehicle {
    protected String brand;
    
    public void start() {
        System.out.println("Vehicle started");
    }
}

public class Car extends Vehicle {
    private int doors;
    
    public void honk() {
        System.out.println("Car honking");
    }
}
```

### 2. Multilevel Inheritance
```java
public class Animal {
    public void breathe() {
        System.out.println("Breathing...");
    }
}

public class Mammal extends Animal {
    public void feedMilk() {
        System.out.println("Feeding milk...");
    }
}

public class Dog extends Mammal {
    public void bark() {
        System.out.println("Woof!");
    }
}

// Usage chain: Dog -> Mammal -> Animal
Dog dog = new Dog();
dog.breathe();    // From Animal
dog.feedMilk();   // From Mammal
dog.bark();       // From Dog
```

### 3. Hierarchical Inheritance
```java
public class Shape {
    protected String color;
    
    public void draw() {
        System.out.println("Drawing shape");
    }
}

public class Circle extends Shape {
    private double radius;
    
    @Override
    public void draw() {
        System.out.println("Drawing circle");
    }
}

public class Rectangle extends Shape {
    private double width, height;
    
    @Override
    public void draw() {
        System.out.println("Drawing rectangle");
    }
}
```

## Method Overriding

### Rules for Method Overriding

```java
public class Parent {
    public void display() {
        System.out.println("Parent display");
    }
    
    public final void cannotOverride() {
        System.out.println("This cannot be overridden");
    }
    
    protected void protectedMethod() {
        System.out.println("Protected method");
    }
}

public class Child extends Parent {
    // Valid override
    @Override
    public void display() {
        System.out.println("Child display");
        super.display(); // Call parent method
    }
    
    // Error: Cannot override final method
    // public void cannotOverride() { }
    
    // Valid: Increasing access level
    @Override
    public void protectedMethod() {
        System.out.println("Child protected method");
    }
}
```

### The super Keyword

```java
public class Vehicle {
    protected String model;
    
    public Vehicle(String model) {
        this.model = model;
    }
    
    public void start() {
        System.out.println("Vehicle " + model + " starting");
    }
}

public class Car extends Vehicle {
    private int doors;
    
    public Car(String model, int doors) {
        super(model); // Call parent constructor
        this.doors = doors;
    }
    
    @Override
    public void start() {
        super.start(); // Call parent method
        System.out.println("Car with " + doors + " doors ready");
    }
}
```

## Abstract Classes and Methods

### Abstract Classes

```java
// Abstract class - cannot be instantiated
public abstract class Animal {
    protected String name;
    
    public Animal(String name) {
        this.name = name;
    }
    
    // Concrete method
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    // Abstract method - must be implemented by subclasses
    public abstract void makeSound();
    
    // Abstract method with implementation
    public abstract void move();
}

public class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " says Woof!");
    }
    
    @Override
    public void move() {
        System.out.println(name + " is running");
    }
}

// Usage
Animal dog = new Dog("Buddy"); // Polymorphism
dog.makeSound(); // Output: Buddy says Woof!
dog.sleep();     // Output: Buddy is sleeping
```

## Interface Inheritance

### Extending Interfaces

```java
public interface Drawable {
    void draw();
}

public interface Movable {
    void move();
}

// Interface extending multiple interfaces
public interface Animated extends Drawable, Movable {
    void animate();
}

public class Sprite implements Animated {
    @Override
    public void draw() {
        System.out.println("Drawing sprite");
    }
    
    @Override
    public void move() {
        System.out.println("Moving sprite");
    }
    
    @Override
    public void animate() {
        System.out.println("Animating sprite");
    }
}
```

## The Object Class

All classes in Java implicitly extend `java.lang.Object`.

### Key Methods from Object Class

```java
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // equals() method - for object comparison
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        Person person = (Person) obj;
        return age == person.age && 
               Objects.equals(name, person.name);
    }
    
    // hashCode() method - for hash-based collections
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    // toString() method - string representation
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}

// Usage
Person p1 = new Person("Alice", 25);
Person p2 = new Person("Alice", 25);
System.out.println(p1.toString()); // Person{name='Alice', age=25}
System.out.println(p1.equals(p2)); // true
```

## Inheritance vs Composition

### Inheritance (IS-A relationship)
```java
public class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

public class Car extends Engine { // Car IS-A Engine
    private String model;
    
    public Car(String model) {
        this.model = model;
    }
    
    public void drive() {
        start(); // Inherited method
        System.out.println("Driving " + model);
    }
}
```

### Composition (HAS-A relationship)
```java
public class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

public class Car {
    private Engine engine; // Car HAS-A Engine
    private String model;
    
    public Car(String model) {
        this.engine = new Engine();
        this.model = model;
    }
    
    public void drive() {
        engine.start(); // Delegated call
        System.out.println("Driving " + model);
    }
}
```

## Best Practices

### 1. Favor Composition Over Inheritance
```java
// Better approach - composition
public class NotificationService {
    private EmailSender emailSender;
    private SmsSender smsSender;
    
    public void notifyUser(String message, String method) {
        if ("email".equals(method)) {
            emailSender.send(message);
        } else if ("sms".equals(method)) {
            smsSender.send(message);
        }
    }
}
```

### 2. Use Abstract Classes for Common Behavior
```java
public abstract class PaymentProcessor {
    protected abstract void processPayment(double amount);
    
    // Common validation logic
    public final boolean validateAmount(double amount) {
        return amount > 0 && amount <= 10000;
    }
    
    // Template method pattern
    public final void executePayment(double amount) {
        if (validateAmount(amount)) {
            processPayment(amount);
            logTransaction(amount);
        }
    }
    
    protected abstract void logTransaction(double amount);
}
```

### 3. Proper Use of Access Modifiers
```java
public class Vehicle {
    protected String brand;    // Accessible to subclasses
    private int speed;         // Not accessible to subclasses
    
    protected void accelerate() { // Accessible to subclasses
        speed += 10;
    }
}
```

## Advanced Inheritance Patterns

### Template Method Pattern
```java
public abstract class DataProcessor {
    // Template method
    public final void processData() {
        readData();
        transformData();
        saveData();
    }
    
    protected abstract void readData();
    protected abstract void transformData();
    protected abstract void saveData();
}

public class CsvProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading CSV data");
    }
    
    @Override
    protected void transformData() {
        System.out.println("Transforming CSV data");
    }
    
    @Override
    protected void saveData() {
        System.out.println("Saving to database");
    }
}
```

### Factory Method Pattern
```java
public abstract class DatabaseConnection {
    public abstract void connect();
    
    // Factory method
    public static DatabaseConnection getConnection(String type) {
        switch (type.toLowerCase()) {
            case "mysql":
                return new MySQLConnection();
            case "postgresql":
                return new PostgreSQLConnection();
            default:
                throw new IllegalArgumentException("Unknown database type");
        }
    }
}

public class MySQLConnection extends DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL");
    }
}
```

Inheritance is a powerful feature but should be used judiciously. Prefer composition when possible and ensure that inheritance hierarchies follow the "is-a" relationship principle.
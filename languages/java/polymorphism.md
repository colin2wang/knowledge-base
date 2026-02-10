# Polymorphism

Polymorphism is one of the four fundamental principles of object-oriented programming. It allows objects of different types to be treated as objects of a common superclass, enabling methods to behave differently based on the actual object type at runtime.

## What is Polymorphism?

Polymorphism means "many forms." In Java, it allows a single interface to represent different underlying forms (data types).

### Basic Example

```java
// Parent class
public class Animal {
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

// Subclasses
public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof! Woof!");
    }
}

public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

public class Bird extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Tweet! Tweet!");
    }
}

// Polymorphic usage
public class Zoo {
    public static void makeAnimalsSpeak(Animal[] animals) {
        for (Animal animal : animals) {
            animal.makeSound(); // Different behavior based on actual type
        }
    }
    
    public static void main(String[] args) {
        Animal[] animals = {
            new Dog(),
            new Cat(),
            new Bird(),
            new Animal() // Generic animal
        };
        
        makeAnimalsSpeak(animals);
        // Output:
        // Woof! Woof!
        // Meow!
        // Tweet! Tweet!
        // Some generic animal sound
    }
}
```

## Types of Polymorphism

### 1. Compile-time Polymorphism (Method Overloading)

```java
public class Calculator {
    // Method overloading - same name, different parameters
    public int add(int a, int b) {
        return a + b;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    public String add(String a, String b) {
        return a + b;
    }
}

// Usage
Calculator calc = new Calculator();
System.out.println(calc.add(5, 3));        // 8
System.out.println(calc.add(5.5, 3.2));    // 8.7
System.out.println(calc.add(1, 2, 3));     // 6
System.out.println(calc.add("Hello", " World")); // Hello World
```

### 2. Runtime Polymorphism (Method Overriding)

```java
public class Shape {
    public double calculateArea() {
        return 0; // Default implementation
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    private double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

// Runtime polymorphism in action
public class Geometry {
    public static void printAreas(Shape[] shapes) {
        for (Shape shape : shapes) {
            System.out.println("Area: " + shape.calculateArea());
            // Actual method called depends on runtime type
        }
    }
    
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle(5),
            new Rectangle(4, 6),
            new Shape() // Anonymous Shape
        };
        
        printAreas(shapes);
        // Output:
        // Area: 78.53981633974483
        // Area: 24.0
        // Area: 0.0
    }
}
```

## Dynamic Method Dispatch

Java uses dynamic method dispatch to determine which method to call at runtime.

### How It Works

```java
public class Vehicle {
    public void start() {
        System.out.println("Vehicle starting");
    }
}

public class Car extends Vehicle {
    @Override
    public void start() {
        System.out.println("Car engine starting");
    }
    
    public void openTrunk() {
        System.out.println("Trunk opened");
    }
}

public class Motorcycle extends Vehicle {
    @Override
    public void start() {
        System.out.println("Motorcycle engine roaring");
    }
}

// Demonstration
public class TestPolymorphism {
    public static void demonstrate(Vehicle vehicle) {
        vehicle.start(); // Dynamic method dispatch
        
        // Type checking and casting
        if (vehicle instanceof Car) {
            Car car = (Car) vehicle;
            car.openTrunk();
        }
    }
    
    public static void main(String[] args) {
        Vehicle v1 = new Car();
        Vehicle v2 = new Motorcycle();
        Vehicle v3 = new Vehicle();
        
        demonstrate(v1); // Car engine starting + Trunk opened
        demonstrate(v2); // Motorcycle engine roaring
        demonstrate(v3); // Vehicle starting
    }
}
```

## Polymorphism with Interfaces

### Interface-based Polymorphism

```java
public interface Drawable {
    void draw();
}

public class Circle implements Drawable {
    @Override
    public void draw() {
        System.out.println("Drawing a circle");
    }
}

public class Square implements Drawable {
    @Override
    public void draw() {
        System.out.println("Drawing a square");
    }
}

public class Triangle implements Drawable {
    @Override
    public void draw() {
        System.out.println("Drawing a triangle");
    }
}

// Polymorphic method
public class Artist {
    public static void createArt(Drawable[] shapes) {
        for (Drawable shape : shapes) {
            shape.draw(); // Polymorphic call
        }
    }
    
    public static void main(String[] args) {
        Drawable[] shapes = {
            new Circle(),
            new Square(),
            new Triangle()
        };
        
        createArt(shapes);
        // Output:
        // Drawing a circle
        // Drawing a square
        // Drawing a triangle
    }
}
```

## Abstract Classes and Polymorphism

```java
public abstract class PaymentMethod {
    protected double amount;
    
    public PaymentMethod(double amount) {
        this.amount = amount;
    }
    
    public abstract void processPayment();
    public abstract String getPaymentDetails();
}

public class CreditCardPayment extends PaymentMethod {
    private String cardNumber;
    
    public CreditCardPayment(double amount, String cardNumber) {
        super(amount);
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void processPayment() {
        System.out.println("Processing credit card payment of $" + amount);
    }
    
    @Override
    public String getPaymentDetails() {
        return "Credit Card ending in " + cardNumber.substring(cardNumber.length() - 4);
    }
}

public class PayPalPayment extends PaymentMethod {
    private String email;
    
    public PayPalPayment(double amount, String email) {
        super(amount);
        this.email = email;
    }
    
    @Override
    public void processPayment() {
        System.out.println("Processing PayPal payment of $" + amount);
    }
    
    @Override
    public String getPaymentDetails() {
        return "PayPal account: " + email;
    }
}

// Polymorphic payment processor
public class PaymentProcessor {
    public static void processPayments(PaymentMethod[] payments) {
        for (PaymentMethod payment : payments) {
            System.out.println("Payment Details: " + payment.getPaymentDetails());
            payment.processPayment();
            System.out.println("---");
        }
    }
    
    public static void main(String[] args) {
        PaymentMethod[] payments = {
            new CreditCardPayment(100.0, "1234567890123456"),
            new PayPalPayment(50.0, "user@example.com")
        };
        
        processPayments(payments);
        // Output:
        // Payment Details: Credit Card ending in 3456
        // Processing credit card payment of $100.0
        // ---
        // Payment Details: PayPal account: user@example.com
        // Processing PayPal payment of $50.0
        // ---
    }
}
```

## Covariant Return Types

```java
public class Animal {
    public Animal reproduce() {
        return new Animal();
    }
}

public class Dog extends Animal {
    @Override
    public Dog reproduce() { // More specific return type
        return new Dog();
    }
    
    public void bark() {
        System.out.println("Woof!");
    }
}

public class BreedingProgram {
    public static void breedAnimals(Animal[] animals) {
        for (Animal animal : animals) {
            Animal offspring = animal.reproduce();
            
            // Type-specific behavior
            if (offspring instanceof Dog) {
                ((Dog) offspring).bark();
            }
        }
    }
}
```

## Polymorphism in Collections

### Heterogeneous Collections

```java
import java.util.*;

public class Game {
    public static void main(String[] args) {
        List<Shape> shapes = new ArrayList<>();
        shapes.add(new Circle(5));
        shapes.add(new Rectangle(4, 6));
        shapes.add(new Triangle(3, 4, 5));
        
        // Polymorphic processing
        for (Shape shape : shapes) {
            System.out.println(shape.getClass().getSimpleName() + 
                             " area: " + shape.calculateArea());
        }
    }
}

abstract class Shape {
    public abstract double calculateArea();
}

class Circle extends Shape {
    private double radius;
    public Circle(double radius) { this.radius = radius; }
    @Override public double calculateArea() { 
        return Math.PI * radius * radius; 
    }
}

class Rectangle extends Shape {
    private double width, height;
    public Rectangle(double width, double height) { 
        this.width = width; 
        this.height = height; 
    }
    @Override public double calculateArea() { 
        return width * height; 
    }
}

class Triangle extends Shape {
    private double a, b, c;
    public Triangle(double a, double b, double c) { 
        this.a = a; 
        this.b = b; 
        this.c = c; 
    }
    @Override public double calculateArea() {
        double s = (a + b + c) / 2;
        return Math.sqrt(s * (s - a) * (s - b) * (s - c));
    }
}
```

## instanceof Operator and Downcasting

```java
public class AnimalShelter {
    public static void careForAnimals(Animal[] animals) {
        for (Animal animal : animals) {
            animal.makeSound();
            
            // Downcasting with instanceof check
            if (animal instanceof Dog) {
                Dog dog = (Dog) animal;
                dog.fetchStick();
            } else if (animal instanceof Cat) {
                Cat cat = (Cat) animal;
                cat.purr();
            } else if (animal instanceof Bird) {
                Bird bird = (Bird) animal;
                bird.fly();
            }
        }
    }
}

class Animal {
    public void makeSound() {
        System.out.println("Some animal sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
    
    public void fetchStick() {
        System.out.println("Fetching stick!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
    
    public void purr() {
        System.out.println("Purring contentedly");
    }
}

class Bird extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Tweet!");
    }
    
    public void fly() {
        System.out.println("Flying away!");
    }
}
```

## Benefits of Polymorphism

### 1. Code Reusability
```java
public class ReportGenerator {
    // Single method works with all Report types
    public static void generateReports(List<Report> reports) {
        for (Report report : reports) {
            report.generate();
        }
    }
}

interface Report {
    void generate();
}

class SalesReport implements Report {
    public void generate() {
        System.out.println("Generating sales report");
    }
}

class InventoryReport implements Report {
    public void generate() {
        System.out.println("Generating inventory report");
    }
}
```

### 2. Maintainability
```java
// Easy to add new types without changing existing code
class FinancialReport implements Report {
    public void generate() {
        System.out.println("Generating financial report");
    }
}

// Existing code still works
List<Report> reports = Arrays.asList(
    new SalesReport(),
    new InventoryReport(),
    new FinancialReport() // New type
);
ReportGenerator.generateReports(reports);
```

### 3. Flexibility
```java
public class PluginSystem {
    private List<Plugin> plugins = new ArrayList<>();
    
    public void addPlugin(Plugin plugin) {
        plugins.add(plugin);
    }
    
    public void executePlugins() {
        for (Plugin plugin : plugins) {
            plugin.execute();
        }
    }
}

interface Plugin {
    void execute();
}

// Different plugins can be added dynamically
class DatabasePlugin implements Plugin {
    public void execute() {
        System.out.println("Executing database operations");
    }
}

class LoggingPlugin implements Plugin {
    public void execute() {
        System.out.println("Logging system events");
    }
}
```

Polymorphism enables flexible, extensible code that can adapt to new requirements without major modifications to existing code.
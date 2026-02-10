# Classes and Objects

Classes and objects are the fundamental building blocks of object-oriented programming in Java. Everything in Java is associated with classes and objects, along with their attributes and methods.

## What is a Class?

A class is a blueprint or template for creating objects. It defines the properties (fields) and behaviors (methods) that objects of that class will have.

### Basic Class Structure

```java
public class Person {
    // Fields (attributes)
    private String name;
    private int age;
    
    // Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Methods (behaviors)
    public void introduce() {
        System.out.println("Hi, I'm " + name + " and I'm " + age + " years old.");
    }
    
    // Getter and Setter methods
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
}
```

## What is an Object?

An object is an instance of a class. It's a concrete entity based on the class blueprint that contains actual values rather than variables.

### Creating Objects

```java
// Creating objects using the 'new' keyword
Person person1 = new Person("Alice", 25);
Person person2 = new Person("Bob", 30);

// Using objects
person1.introduce(); // Output: Hi, I'm Alice and I'm 25 years old.
person2.introduce(); // Output: Hi, I'm Bob and I'm 30 years old.
```

## Constructors

Constructors are special methods used to initialize objects when they are created.

### Types of Constructors

#### 1. Default Constructor
```java
public class Student {
    private String name;
    
    // Default constructor (no parameters)
    public Student() {
        this.name = "Unknown";
    }
}
```

#### 2. Parameterized Constructor
```java
public class Student {
    private String name;
    private int id;
    
    // Parameterized constructor
    public Student(String name, int id) {
        this.name = name;
        this.id = id;
    }
}
```

#### 3. Constructor Overloading
```java
public class Rectangle {
    private double width;
    private double height;
    
    // Multiple constructors
    public Rectangle() {
        this(1.0, 1.0); // Calling another constructor
    }
    
    public Rectangle(double side) {
        this(side, side);
    }
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
}
```

## Access Modifiers

Access modifiers control the visibility of classes, fields, and methods.

### Public
```java
public class Calculator {
    public int add(int a, int b) {
        return a + b; // Accessible from anywhere
    }
}
```

### Private
```java
public class BankAccount {
    private double balance; // Only accessible within this class
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public double getBalance() {
        return balance; // Controlled access
    }
}
```

### Protected
```java
public class Vehicle {
    protected String brand; // Accessible in same package and subclasses
}
```

### Package-private (Default)
```java
class Helper { // No access modifier - package private
    void utilityMethod() {
        // Only accessible within the same package
    }
}
```

## Static Members

Static members belong to the class rather than instances.

### Static Variables
```java
public class Counter {
    private static int count = 0; // Shared by all instances
    
    public Counter() {
        count++;
    }
    
    public static int getCount() {
        return count;
    }
}

// Usage
Counter c1 = new Counter();
Counter c2 = new Counter();
System.out.println(Counter.getCount()); // Output: 2
```

### Static Methods
```java
public class MathUtils {
    public static int factorial(int n) {
        if (n <= 1) return 1;
        return n * factorial(n - 1);
    }
}

// Usage - no object creation needed
int result = MathUtils.factorial(5); // Result: 120
```

## this Keyword

The `this` keyword refers to the current object instance.

### Common Uses

#### 1. Disambiguating Field Names
```java
public class Employee {
    private String name;
    
    public Employee(String name) {
        this.name = name; // 'this.name' refers to field, 'name' refers to parameter
    }
}
```

#### 2. Calling Another Constructor
```java
public class Point {
    private int x, y;
    
    public Point() {
        this(0, 0); // Calls the parameterized constructor
    }
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

#### 3. Returning Current Object
```java
public class StringBuilder {
    private String text = "";
    
    public StringBuilder append(String str) {
        this.text += str;
        return this; // Method chaining
    }
}

// Usage
StringBuilder sb = new StringBuilder()
    .append("Hello")
    .append(" ")
    .append("World");
```

## Object Lifecycle

### Creation
1. Memory allocation
2. Constructor execution
3. Object initialization

### Usage
- Calling methods
- Accessing fields
- Passing as parameters

### Destruction
- Eligible for garbage collection when no references exist
- Finalizers (deprecated) or try-with-resources for cleanup

## Best Practices

### 1. Proper Encapsulation
```java
public class BankAccount {
    private double balance;
    
    // Good: Controlled access through methods
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public double getBalance() {
        return balance;
    }
}
```

### 2. Meaningful Class Names
```java
// Good
public class CustomerService { }
public class DatabaseConnection { }

// Avoid
public class CS { }
public class DBConn { }
```

### 3. Single Responsibility Principle
```java
// Good: Each class has one clear purpose
public class UserService { }
public class EmailService { }

// Avoid: Mixed responsibilities
public class UserManager { 
    // Handles user operations AND email sending
}
```

## Advanced Concepts

### Anonymous Classes
```java
interface Greeting {
    void sayHello();
}

Greeting greeting = new Greeting() {
    @Override
    public void sayHello() {
        System.out.println("Hello from anonymous class!");
    }
};
greeting.sayHello();
```

### Inner Classes
```java
public class OuterClass {
    private String outerField = "Outer";
    
    class InnerClass {
        public void display() {
            System.out.println("Accessing: " + outerField);
        }
    }
    
    public InnerClass getInner() {
        return new InnerClass();
    }
}
```

### Immutable Objects
```java
public final class ImmutablePerson {
    private final String name;
    private final int age;
    
    public ImmutablePerson(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    // No setter methods - object cannot be modified after creation
}
```

Understanding classes and objects thoroughly is crucial for mastering Java OOP concepts and building robust, maintainable applications.
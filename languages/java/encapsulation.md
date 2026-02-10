# Encapsulation

Encapsulation is the bundling of data (attributes) and methods that operate on that data within a single unit (class), and restricting direct access to some of the object's components. It's about hiding internal state and requiring all interaction to be performed through an object's methods.

## What is Encapsulation?

Encapsulation provides several key benefits:
- **Data Hiding**: Internal representation is hidden from outside world
- **Controlled Access**: Access to data is controlled through methods
- **Maintainability**: Changes to internal implementation don't affect external code
- **Security**: Prevents unauthorized access to sensitive data

### Basic Encapsulation Example

```java
// Poor encapsulation - direct field access
public class BadBankAccount {
    public double balance; // Direct access - dangerous!
    
    public BadBankAccount(double initialBalance) {
        this.balance = initialBalance;
    }
}

// Good encapsulation - controlled access
public class GoodBankAccount {
    private double balance; // Hidden from outside
    
    public GoodBankAccount(double initialBalance) {
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        } else {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
    }
    
    // Controlled access methods
    public double getBalance() {
        return balance;
    }
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited: $" + amount);
        } else {
            System.out.println("Invalid deposit amount");
        }
    }
    
    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
            return true;
        } else {
            System.out.println("Insufficient funds or invalid amount");
            return false;
        }
    }
}
```

## Access Modifiers

Java provides four access levels to control encapsulation:

### 1. Private
```java
public class Student {
    private String name;        // Only accessible within this class
    private int studentId;      // Only accessible within this class
    private double gpa;         // Only accessible within this class
    
    public Student(String name, int studentId) {
        this.name = name;
        this.studentId = studentId;
        this.gpa = 0.0;
    }
    
    // Public methods to control access
    public String getName() {
        return name;
    }
    
    public int getStudentId() {
        return studentId;
    }
    
    public double getGpa() {
        return gpa;
    }
    
    public void setGpa(double gpa) {
        if (gpa >= 0.0 && gpa <= 4.0) {
            this.gpa = gpa;
        }
    }
}
```

### 2. Default (Package-private)
```java
class UtilityHelper {
    // No access modifier - package private
    void helperMethod() {
        // Only accessible within the same package
        System.out.println("Helper method called");
    }
    
    // Private method for internal use
    private void internalLogic() {
        System.out.println("Internal logic");
    }
}
```

### 3. Protected
```java
public class Vehicle {
    protected String brand;     // Accessible in same package and subclasses
    protected int year;         // Accessible in same package and subclasses
    
    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }
    
    protected void startEngine() {
        System.out.println("Engine started");
    }
}

public class Car extends Vehicle {
    private int doors;
    
    public Car(String brand, int year, int doors) {
        super(brand, year);
        this.doors = doors;
    }
    
    public void displayInfo() {
        // Can access protected fields from parent class
        System.out.println("Brand: " + brand + ", Year: " + year + ", Doors: " + doors);
        startEngine(); // Can call protected method
    }
}
```

### 4. Public
```java
public class Calculator {
    // Public methods - accessible from anywhere
    public int add(int a, int b) {
        return a + b;
    }
    
    public int subtract(int a, int b) {
        return a - b;
    }
    
    public int multiply(int a, int b) {
        return a * b;
    }
    
    public double divide(int a, int b) {
        if (b != 0) {
            return (double) a / b;
        }
        throw new ArithmeticException("Division by zero");
    }
}
```

## Getters and Setters

Proper getter and setter methods are essential for encapsulation:

### Validation in Setters

```java
public class Employee {
    private String name;
    private int age;
    private double salary;
    private String department;
    
    // Constructor with validation
    public Employee(String name, int age, double salary, String department) {
        setName(name);
        setAge(age);
        setSalary(salary);
        setDepartment(department);
    }
    
    // Getter methods
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public double getSalary() {
        return salary;
    }
    
    public String getDepartment() {
        return department;
    }
    
    // Setter methods with validation
    public void setName(String name) {
        if (name != null && !name.trim().isEmpty()) {
            this.name = name.trim();
        } else {
            throw new IllegalArgumentException("Name cannot be null or empty");
        }
    }
    
    public void setAge(int age) {
        if (age >= 16 && age <= 65) {
            this.age = age;
        } else {
            throw new IllegalArgumentException("Age must be between 16 and 65");
        }
    }
    
    public void setSalary(double salary) {
        if (salary >= 0) {
            this.salary = salary;
        } else {
            throw new IllegalArgumentException("Salary cannot be negative");
        }
    }
    
    public void setDepartment(String department) {
        if (department != null && !department.trim().isEmpty()) {
            this.department = department.trim();
        } else {
            throw new IllegalArgumentException("Department cannot be null or empty");
        }
    }
}
```

## Immutable Objects

Creating truly encapsulated immutable objects:

```java
public final class ImmutablePerson {
    private final String name;
    private final int age;
    private final Address address;
    
    public ImmutablePerson(String name, int age, Address address) {
        this.name = validateName(name);
        this.age = validateAge(age);
        this.address = address != null ? new Address(address) : null; // Defensive copy
    }
    
    // Only getters, no setters
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public Address getAddress() {
        return address != null ? new Address(address) : null; // Defensive copy
    }
    
    // Validation methods
    private String validateName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be null or empty");
        }
        return name.trim();
    }
    
    private int validateAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Age must be between 0 and 150");
        }
        return age;
    }
    
    // No setter methods - object is immutable
    
    @Override
    public String toString() {
        return "ImmutablePerson{name='" + name + "', age=" + age + 
               ", address=" + address + "}";
    }
}

// Immutable Address class
final class Address {
    private final String street;
    private final String city;
    private final String zipCode;
    
    public Address(String street, String city, String zipCode) {
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
    }
    
    // Copy constructor for defensive copying
    public Address(Address other) {
        this(other.street, other.city, other.zipCode);
    }
    
    // Getters only
    public String getStreet() { return street; }
    public String getCity() { return city; }
    public String getZipCode() { return zipCode; }
    
    @Override
    public String toString() {
        return "Address{street='" + street + "', city='" + city + 
               "', zipCode='" + zipCode + "'}";
    }
}
```

## Builder Pattern for Complex Objects

```java
public class User {
    private final String username;
    private final String email;
    private final String firstName;
    private final String lastName;
    private final int age;
    private final boolean isActive;
    private final List<String> roles;
    
    private User(Builder builder) {
        this.username = builder.username;
        this.email = builder.email;
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.isActive = builder.isActive;
        this.roles = new ArrayList<>(builder.roles);
    }
    
    // Getters only - no setters
    public String getUsername() { return username; }
    public String getEmail() { return email; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
    public boolean isActive() { return isActive; }
    public List<String> getRoles() { return new ArrayList<>(roles); }
    
    // Builder class
    public static class Builder {
        private String username;
        private String email;
        private String firstName;
        private String lastName;
        private int age;
        private boolean isActive = true;
        private List<String> roles = new ArrayList<>();
        
        public Builder(String username, String email) {
            this.username = username;
            this.email = email;
        }
        
        public Builder firstName(String firstName) {
            this.firstName = firstName;
            return this;
        }
        
        public Builder lastName(String lastName) {
            this.lastName = lastName;
            return this;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder active(boolean isActive) {
            this.isActive = isActive;
            return this;
        }
        
        public Builder addRole(String role) {
            this.roles.add(role);
            return this;
        }
        
        public User build() {
            // Validation
            if (username == null || username.isEmpty()) {
                throw new IllegalStateException("Username is required");
            }
            if (email == null || !email.contains("@")) {
                throw new IllegalStateException("Valid email is required");
            }
            
            return new User(this);
        }
    }
}

// Usage
User user = new User.Builder("john_doe", "john@example.com")
    .firstName("John")
    .lastName("Doe")
    .age(30)
    .active(true)
    .addRole("USER")
    .addRole("ADMIN")
    .build();
```

## Encapsulation in Practice

### Bank Account Example

```java
public class BankAccount {
    private String accountNumber;
    private String accountHolder;
    private double balance;
    private String pin;
    private boolean isLocked;
    private int failedAttempts;
    
    public BankAccount(String accountNumber, String accountHolder, String pin) {
        this.accountNumber = accountNumber;
        this.accountHolder = accountHolder;
        this.pin = pin;
        this.balance = 0.0;
        this.isLocked = false;
        this.failedAttempts = 0;
    }
    
    // Public methods with proper encapsulation
    public boolean authenticate(String enteredPin) {
        if (isLocked) {
            System.out.println("Account is locked");
            return false;
        }
        
        if (pin.equals(enteredPin)) {
            failedAttempts = 0;
            return true;
        } else {
            failedAttempts++;
            if (failedAttempts >= 3) {
                isLocked = true;
                System.out.println("Account locked due to too many failed attempts");
            }
            return false;
        }
    }
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited: $" + amount);
            System.out.println("New balance: $" + balance);
        }
    }
    
    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
            System.out.println("Remaining balance: $" + balance);
            return true;
        }
        System.out.println("Insufficient funds or invalid amount");
        return false;
    }
    
    public double getBalance() {
        return balance;
    }
    
    public String getAccountNumber() {
        // Mask sensitive information
        return "****" + accountNumber.substring(accountNumber.length() - 4);
    }
    
    public String getAccountHolder() {
        return accountHolder;
    }
    
    public boolean isAccountLocked() {
        return isLocked;
    }
    
    public void unlockAccount(String managerPin) {
        if ("MANAGER123".equals(managerPin)) {
            isLocked = false;
            failedAttempts = 0;
            System.out.println("Account unlocked");
        }
    }
}
```

## Benefits of Encapsulation

### 1. Data Integrity
```java
public class Temperature {
    private double celsius;
    
    public Temperature(double celsius) {
        setCelsius(celsius);
    }
    
    public double getCelsius() {
        return celsius;
    }
    
    public void setCelsius(double celsius) {
        if (celsius < -273.15) {
            throw new IllegalArgumentException("Temperature cannot be below absolute zero");
        }
        this.celsius = celsius;
    }
    
    public double getFahrenheit() {
        return (celsius * 9/5) + 32;
    }
    
    public void setFahrenheit(double fahrenheit) {
        setCelsius((fahrenheit - 32) * 5/9);
    }
}
```

### 2. Implementation Flexibility
```java
public class DataStorage {
    private List<String> data;
    
    public DataStorage() {
        // Could easily change to HashSet, ArrayList, etc.
        this.data = new ArrayList<>();
    }
    
    public void addItem(String item) {
        data.add(item);
    }
    
    public boolean removeItem(String item) {
        return data.remove(item);
    }
    
    public List<String> getAllItems() {
        // Return copy to prevent external modification
        return new ArrayList<>(data);
    }
    
    // Internal implementation can change without affecting clients
}
```

### 3. Debugging and Logging
```java
public class Counter {
    private int count = 0;
    
    public int getCount() {
        System.out.println("Getting count: " + count);
        return count;
    }
    
    public void increment() {
        count++;
        System.out.println("Incremented to: " + count);
    }
    
    public void decrement() {
        count--;
        System.out.println("Decremented to: " + count);
    }
    
    public void reset() {
        count = 0;
        System.out.println("Counter reset");
    }
}
```

Encapsulation is crucial for building robust, maintainable, and secure Java applications. It ensures that object state remains consistent and that changes to internal implementation don't break client code.
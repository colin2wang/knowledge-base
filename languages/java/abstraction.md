# Abstraction

Abstraction is the process of hiding complex implementation details and showing only the essential features of an object. It focuses on what an object does rather than how it does it, allowing developers to work with high-level concepts without getting bogged down in implementation specifics.

## What is Abstraction?

Abstraction involves:
- **Hiding implementation details**
- **Exposing only necessary functionality**
- **Reducing complexity**
- **Improving maintainability**
- **Enabling loose coupling**

### Real-world Analogy

Think of driving a car - you don't need to understand how the engine works internally. You just need to know that pressing the gas pedal makes the car go faster. The complex mechanics are abstracted away behind a simple interface.

## Abstract Classes

Abstract classes provide partial abstraction by defining common behavior while leaving some methods to be implemented by subclasses.

### Basic Abstract Class Example

```java
// Abstract class - cannot be instantiated
public abstract class Vehicle {
    protected String brand;
    protected int year;
    
    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }
    
    // Concrete method - shared implementation
    public void start() {
        System.out.println("Starting " + brand + " vehicle");
    }
    
    // Concrete method
    public void stop() {
        System.out.println("Stopping vehicle");
    }
    
    // Abstract method - must be implemented by subclasses
    public abstract void move();
    
    // Abstract method
    public abstract double calculateFuelEfficiency();
    
    // Abstract method with parameters
    public abstract void refuel(String fuelType, double amount);
}

// Concrete implementation
public class Car extends Vehicle {
    private int numberOfDoors;
    private double fuelConsumption;
    
    public Car(String brand, int year, int numberOfDoors) {
        super(brand, year);
        this.numberOfDoors = numberOfDoors;
        this.fuelConsumption = 8.5; // liters per 100km
    }
    
    @Override
    public void move() {
        System.out.println("Car is driving on roads");
    }
    
    @Override
    public double calculateFuelEfficiency() {
        return 100.0 / fuelConsumption; // km per liter
    }
    
    @Override
    public void refuel(String fuelType, double amount) {
        System.out.println("Refueling car with " + amount + " liters of " + fuelType);
    }
    
    public int getNumberOfDoors() {
        return numberOfDoors;
    }
}

// Another concrete implementation
public class Motorcycle extends Vehicle {
    private boolean hasSidecar;
    
    public Motorcycle(String brand, int year, boolean hasSidecar) {
        super(brand, year);
        this.hasSidecar = hasSidecar;
    }
    
    @Override
    public void move() {
        System.out.println("Motorcycle is riding on roads");
    }
    
    @Override
    public double calculateFuelEfficiency() {
        return 25.0; // km per liter (more efficient)
    }
    
    @Override
    public void refuel(String fuelType, double amount) {
        System.out.println("Refueling motorcycle with " + amount + " liters of " + fuelType);
    }
}
```

## Interfaces

Interfaces provide complete abstraction by defining contracts without any implementation.

### Basic Interface Example

```java
// Interface - 100% abstraction
public interface PaymentProcessor {
    boolean processPayment(double amount);
    void refund(double amount);
    String getTransactionId();
    PaymentStatus getStatus();
}

public enum PaymentStatus {
    PENDING, SUCCESS, FAILED, REFUNDED
}

// Implementation
public class CreditCardProcessor implements PaymentProcessor {
    private String transactionId;
    private PaymentStatus status;
    private String cardNumber;
    
    public CreditCardProcessor(String cardNumber) {
        this.cardNumber = cardNumber;
        this.status = PaymentStatus.PENDING;
    }
    
    @Override
    public boolean processPayment(double amount) {
        // Simulate payment processing
        if (amount > 0 && amount <= 10000) {
            this.transactionId = "CC-" + System.currentTimeMillis();
            this.status = PaymentStatus.SUCCESS;
            System.out.println("Credit card payment processed: $" + amount);
            return true;
        }
        this.status = PaymentStatus.FAILED;
        return false;
    }
    
    @Override
    public void refund(double amount) {
        if (status == PaymentStatus.SUCCESS) {
            this.status = PaymentStatus.REFUNDED;
            System.out.println("Refund processed: $" + amount);
        }
    }
    
    @Override
    public String getTransactionId() {
        return transactionId;
    }
    
    @Override
    public PaymentStatus getStatus() {
        return status;
    }
}

// Another implementation
public class PayPalProcessor implements PaymentProcessor {
    private String transactionId;
    private PaymentStatus status;
    private String emailAddress;
    
    public PayPalProcessor(String emailAddress) {
        this.emailAddress = emailAddress;
        this.status = PaymentStatus.PENDING;
    }
    
    @Override
    public boolean processPayment(double amount) {
        // Simulate PayPal payment processing
        if (amount > 0) {
            this.transactionId = "PP-" + System.currentTimeMillis();
            this.status = PaymentStatus.SUCCESS;
            System.out.println("PayPal payment processed: $" + amount);
            return true;
        }
        this.status = PaymentStatus.FAILED;
        return false;
    }
    
    @Override
    public void refund(double amount) {
        if (status == PaymentStatus.SUCCESS) {
            this.status = PaymentStatus.REFUNDED;
            System.out.println("PayPal refund processed: $" + amount);
        }
    }
    
    @Override
    public String getTransactionId() {
        return transactionId;
    }
    
    @Override
    public PaymentStatus getStatus() {
        return status;
    }
}
```

## Abstract vs Interface

### When to Use Abstract Classes

```java
// Use abstract class when you have common implementation
public abstract class DatabaseConnection {
    protected String connectionString;
    protected boolean isConnected;
    
    public DatabaseConnection(String connectionString) {
        this.connectionString = connectionString;
        this.isConnected = false;
    }
    
    // Common implementation
    public void connect() {
        System.out.println("Connecting to: " + connectionString);
        // Common connection logic
        isConnected = true;
    }
    
    public void disconnect() {
        System.out.println("Disconnecting from database");
        isConnected = false;
    }
    
    // Abstract methods for specific implementations
    public abstract void executeQuery(String sql);
    public abstract void executeUpdate(String sql);
}

public class MySQLConnection extends DatabaseConnection {
    public MySQLConnection(String connectionString) {
        super(connectionString);
    }
    
    @Override
    public void executeQuery(String sql) {
        if (isConnected) {
            System.out.println("MySQL executing query: " + sql);
        }
    }
    
    @Override
    public void executeUpdate(String sql) {
        if (isConnected) {
            System.out.println("MySQL executing update: " + sql);
        }
    }
}
```

### When to Use Interfaces

```java
// Use interface when you want to define a contract
public interface Drawable {
    void draw();
    void resize(double factor);
    Point getPosition();
}

public interface Movable {
    void moveTo(Point position);
    void moveBy(double deltaX, double deltaY);
}

// Class can implement multiple interfaces
public class GameSprite implements Drawable, Movable {
    private Point position;
    private double size;
    
    public GameSprite(Point initialPosition) {
        this.position = initialPosition;
        this.size = 1.0;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing sprite at " + position);
    }
    
    @Override
    public void resize(double factor) {
        this.size *= factor;
        System.out.println("Resized to: " + size);
    }
    
    @Override
    public Point getPosition() {
        return new Point(position.getX(), position.getY());
    }
    
    @Override
    public void moveTo(Point position) {
        this.position = position;
        System.out.println("Moved to: " + position);
    }
    
    @Override
    public void moveBy(double deltaX, double deltaY) {
        this.position = new Point(
            position.getX() + deltaX,
            position.getY() + deltaY
        );
        System.out.println("Moved by (" + deltaX + ", " + deltaY + ")");
    }
}
```

## Layered Abstraction

### Multi-layer Architecture Example

```java
// Service Layer - Business Logic Abstraction
public interface UserService {
    User createUser(UserRegistrationRequest request);
    User getUserById(Long userId);
    List<User> getAllUsers();
    User updateUser(Long userId, UserUpdateRequest request);
    void deleteUser(Long userId);
}

// Implementation with abstraction
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    private final PasswordEncoder passwordEncoder;
    
    public UserServiceImpl(UserRepository userRepository, 
                          EmailService emailService,
                          PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.emailService = emailService;
        this.passwordEncoder = passwordEncoder;
    }
    
    @Override
    public User createUser(UserRegistrationRequest request) {
        // Business logic abstraction
        validateRegistrationRequest(request);
        String encodedPassword = passwordEncoder.encode(request.getPassword());
        User user = new User(request.getUsername(), encodedPassword, request.getEmail());
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser.getEmail());
        return savedUser;
    }
    
    private void validateRegistrationRequest(UserRegistrationRequest request) {
        // Validation logic abstracted
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new IllegalArgumentException("Username already exists");
        }
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new IllegalArgumentException("Email already registered");
        }
    }
    
    // Other method implementations...
}

// Repository Layer - Data Access Abstraction
public interface UserRepository {
    User save(User user);
    Optional<User> findById(Long userId);
    List<User> findAll();
    User update(User user);
    void deleteById(Long userId);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}

// Controller Layer - API Abstraction
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody UserRegistrationRequest request) {
        // API abstraction - hides business logic complexity
        User user = userService.createUser(request);
        return ResponseEntity.ok(user);
    }
    
    @GetMapping("/{userId}")
    public ResponseEntity<User> getUser(@PathVariable Long userId) {
        User user = userService.getUserById(userId);
        return ResponseEntity.ok(user);
    }
}
```

## Functional Abstraction

### Higher-order Functions

```java
// Functional interface - abstraction of behavior
@FunctionalInterface
public interface Operation<T> {
    T execute(T a, T b);
}

public class Calculator {
    // High-level abstraction
    public static <T> T calculate(T a, T b, Operation<T> operation) {
        return operation.execute(a, b);
    }
    
    // Specific implementations abstracted
    public static void demonstrate() {
        // Integer addition
        Integer sum = calculate(5, 3, (x, y) -> x + y);
        System.out.println("Sum: " + sum);
        
        // String concatenation
        String result = calculate("Hello", " World", (s1, s2) -> s1 + s2);
        System.out.println("Concatenated: " + result);
        
        // Custom operation
        Double power = calculate(2.0, 3.0, (x, y) -> Math.pow(x, y));
        System.out.println("Power: " + power);
    }
}
```

## Template Method Pattern

```java
// Abstract class providing template method
public abstract class DataProcessor {
    // Template method - defines algorithm structure
    public final void processData() {
        readData();
        validateData();
        transformData();
        saveData();
        cleanup();
    }
    
    // Abstract methods - to be implemented by subclasses
    protected abstract void readData();
    protected abstract void transformData();
    protected abstract void saveData();
    
    // Hook methods - optional customization
    protected void validateData() {
        System.out.println("Performing basic validation");
    }
    
    protected void cleanup() {
        System.out.println("Cleaning up resources");
    }
}

// Concrete implementation
public class CsvDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading CSV file");
    }
    
    @Override
    protected void transformData() {
        System.out.println("Transforming CSV data");
    }
    
    @Override
    protected void saveData() {
        System.out.println("Saving to database");
    }
    
    // Override hook method for customization
    @Override
    protected void validateData() {
        System.out.println("Performing CSV-specific validation");
        super.validateData();
    }
}
```

## Strategy Pattern

```java
// Strategy interface - abstraction of algorithms
public interface SortingStrategy {
    void sort(int[] array);
}

// Concrete strategies
public class BubbleSortStrategy implements SortingStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using Bubble Sort");
        // Implementation hidden
    }
}

public class QuickSortStrategy implements SortingStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using Quick Sort");
        // Implementation hidden
    }
}

public class MergeSortStrategy implements SortingStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using Merge Sort");
        // Implementation hidden
    }
}

// Context class using abstraction
public class Sorter {
    private SortingStrategy strategy;
    
    public Sorter(SortingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void setStrategy(SortingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sortArray(int[] array) {
        // Client doesn't need to know sorting details
        strategy.sort(array);
    }
}

// Usage
public class SortingDemo {
    public static void demonstrate() {
        int[] data = {64, 34, 25, 12, 22, 11, 90};
        
        Sorter sorter = new Sorter(new QuickSortStrategy());
        sorter.sortArray(data); // Uses Quick Sort
        
        sorter.setStrategy(new MergeSortStrategy());
        sorter.sortArray(data); // Uses Merge Sort
    }
}
```

## Benefits of Abstraction

### 1. Reduced Complexity
```java
// Without abstraction - complex and tightly coupled
public class ComplexSystem {
    public void processOrder(String orderId, String customerId, 
                           double amount, String paymentMethod) {
        // Complex implementation details exposed
        DatabaseConnection db = new MySQLConnection("jdbc:mysql://localhost:3306/store");
        db.connect();
        
        String sql = "SELECT * FROM customers WHERE id = '" + customerId + "'";
        ResultSet rs = db.executeQuery(sql);
        // ... lots of complex code
        
        PaymentProcessor processor = new CreditCardProcessor();
        processor.processPayment(amount);
        // ... more complex code
    }
}

// With abstraction - simplified interface
public class OrderService {
    private final CustomerService customerService;
    private final PaymentService paymentService;
    
    public void processOrder(OrderRequest request) {
        // Simple, abstracted interface
        Customer customer = customerService.getCustomer(request.getCustomerId());
        boolean paymentSuccess = paymentService.processPayment(
            request.getAmount(), 
            request.getPaymentMethod()
        );
        // Clean, simple logic
    }
}
```

### 2. Improved Maintainability
```java
// Easy to modify implementations without affecting clients
public interface NotificationService {
    void sendNotification(String message, String recipient);
}

public class EmailNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending email to " + recipient + ": " + message);
    }
}

// Can easily swap implementations
public class SMSService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
    }
}
```

### 3. Enhanced Testability
```java
// Easy to mock abstractions for testing
public class OrderProcessor {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    
    // Dependencies injected - easy to mock
    public OrderProcessor(PaymentService paymentService, 
                         InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
    
    public boolean processOrder(Order order) {
        // Simple logic using abstractions
        if (paymentService.charge(order.getAmount(), order.getPaymentMethod()) &&
            inventoryService.reserveItems(order.getItems())) {
            return true;
        }
        return false;
    }
}
```

Abstraction is essential for creating clean, maintainable, and scalable Java applications. It allows developers to focus on high-level design while hiding implementation complexities.
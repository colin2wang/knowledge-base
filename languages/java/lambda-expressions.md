# Java Lambda Expressions

Lambda expressions are a key feature introduced in Java 8 that enable functional programming paradigms in Java. They provide a concise way to represent anonymous functions and are widely used with the Stream API and functional interfaces.

## Introduction

Lambda expressions allow you to treat functionality as method arguments, or code as data. They are essentially anonymous functions that can be passed around as parameters or stored in variables.

### Basic Syntax

```java
(parameter_list) -> { body }
```

### Simple Examples

```java
// Zero parameter lambda
Runnable runnable = () -> System.out.println("Hello World");

// Single parameter lambda (parentheses optional)
Consumer<String> printer = message -> System.out.println(message);
Consumer<String> printer2 = (message) -> System.out.println(message);

// Multiple parameters
BiFunction<Integer, Integer, Integer> adder = (a, b) -> a + b;

// With block body and return statement
Function<String, Integer> stringLength = (str) -> {
    return str.length();
};

// Without return statement (expression body)
Function<String, Integer> stringLength2 = str -> str.length();
```

## Functional Interfaces

Lambda expressions work with functional interfaces - interfaces that contain exactly one abstract method.

### Built-in Functional Interfaces

#### Consumer<T>
Consumes a single argument and returns no result.

```java
Consumer<String> consumer = s -> System.out.println(s);
consumer.accept("Hello"); // prints: Hello
```

#### Supplier<T>
Supplies a result of type T with no input.

```java
Supplier<String> supplier = () -> "Hello World";
String result = supplier.get(); // returns: Hello World
```

#### Function<T,R>
Takes an argument of type T and returns a result of type R.

```java
Function<String, Integer> func = s -> s.length();
Integer length = func.apply("Hello"); // returns: 5
```

#### Predicate<T>
Takes an argument of type T and returns a boolean.

```java
Predicate<String> predicate = s -> s.length() > 5;
boolean result = predicate.test("Hello World"); // returns: true
```

#### BiFunction<T,U,R>
Takes two arguments of types T and U, returns a result of type R.

```java
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
Integer result = multiply.apply(5, 3); // returns: 15
```

## Method References

Method references are shorthand for lambda expressions that call existing methods.

### Types of Method References

```java
// Static method reference
Function<String, Integer> parseInt = Integer::parseInt;

// Instance method reference on particular object
List<String> list = Arrays.asList("a", "b", "c");
list.forEach(System.out::println);

// Instance method reference on arbitrary object
Function<String, String> upperCase = String::toUpperCase;

// Constructor reference
Supplier<List<String>> listSupplier = ArrayList::new;
Function<Integer, List<String>> sizedList = ArrayList::new;
```

## Practical Applications

### Collections Operations

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// Filtering
List<String> longNames = names.stream()
    .filter(name -> name.length() > 4)
    .collect(Collectors.toList());

// Mapping
List<Integer> nameLengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());

// Sorting
names.sort((a, b) -> a.compareTo(b));
// Or with method reference
names.sort(String::compareTo);
```

### Event Handling

```java
button.addActionListener(e -> System.out.println("Button clicked"));

// Multiple statements
button.addActionListener(e -> {
    System.out.println("Button clicked");
    counter++;
});
```

### Custom Functional Interfaces

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
}

// Usage
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

int sum = add.calculate(5, 3);      // returns: 8
int product = multiply.calculate(5, 3); // returns: 15
```

## Variable Scoping

Lambda expressions can access variables from the enclosing scope, but with restrictions.

### Effectively Final Variables

```java
public void processData() {
    String prefix = "Processing: "; // effectively final
    
    Consumer<String> processor = data -> {
        System.out.println(prefix + data); // legal
        // prefix = "Changed"; // illegal - would cause compilation error
    };
    
    processor.accept("item1");
}
```

### Restrictions

- Lambda expressions can only access local variables that are final or effectively final
- Instance variables and static variables can be accessed freely
- Cannot shadow local variables from enclosing scope

## Best Practices

### When to Use Lambdas

✅ **Good use cases:**
- Simple operations that fit on one line
- Event handlers and callbacks
- Stream operations (filter, map, reduce)
- Comparators for sorting

❌ **Avoid lambdas when:**
- Logic is complex (more than 2-3 lines)
- Requires multiple parameters with unclear purpose
- Readability suffers significantly

### Naming Conventions

```java
// Good - descriptive names
Function<String, Boolean> isValidEmail = email -> email.contains("@");
Predicate<User> isActiveUser = user -> user.isActive();

// Avoid - cryptic single letters
Function<String, Boolean> f = s -> s.contains("@");
```

## Advanced Features

### Exception Handling

```java
// Lambda throwing checked exception
Function<String, Integer> parseWithException = str -> {
    try {
        return Integer.parseInt(str);
    } catch (NumberFormatException e) {
        throw new RuntimeException("Invalid number: " + str, e);
    }
};
```

### Generic Lambdas

```java
public class GenericProcessor<T> {
    private Function<T, String> processor;
    
    public GenericProcessor(Function<T, String> processor) {
        this.processor = processor;
    }
    
    public String process(T item) {
        return processor.apply(item);
    }
}

// Usage
GenericProcessor<Integer> intProcessor = new GenericProcessor<>(i -> "Number: " + i);
GenericProcessor<String> stringProcessor = new GenericProcessor<>(s -> "Text: " + s);
```

## Common Pitfalls

### Type Inference Issues

```java
// May cause compilation issues due to ambiguous types
Function<Object, String> func = obj -> obj.toString(); // Clear typing

// Instead of
var func = obj -> obj.toString(); // Less clear
```

### Performance Considerations

- Lambdas are objects and create overhead
- Method references are generally more efficient than lambda expressions
- Avoid creating lambdas in loops or frequently called methods

## Integration with Other Features

### Streams API

```java
List<Person> people = getPersonList();

// Complex stream operation with lambda
Map<String, List<Person>> groupedByCity = people.stream()
    .filter(person -> person.getAge() >= 18)
    .collect(Collectors.groupingBy(Person::getCity));

// Parallel processing
people.parallelStream()
    .filter(person -> person.isActive())
    .forEach(person -> processPerson(person));
```

### Optional Class

```java
Optional<String> result = Optional.of("Hello")
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase);
```

This comprehensive guide covers the essential aspects of Java lambda expressions, from basic syntax to advanced applications and best practices.
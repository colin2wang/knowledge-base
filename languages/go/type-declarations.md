# Go Type Declarations

Go provides powerful type declaration mechanisms that allow you to create new types, define type aliases, and establish type relationships. Understanding these concepts is crucial for writing type-safe and maintainable Go code.

## Type Definitions vs Type Aliases

Go distinguishes between type definitions (creating new types) and type aliases (alternative names for existing types).

```go
package main

import "fmt"

// Type alias - same underlying type
type Kilometers = float64
type Miles = float64

// Type definition - creates new distinct type
type Celsius float64
type Fahrenheit float64

func demonstrateTypeDefinitionsVsAliases() {
    // Type aliases can be used interchangeably
    var distance Kilometers = 100.0
    var miles Miles = distance  // This works because they're aliases
    
    fmt.Printf("Distance: %f km\n", distance)
    fmt.Printf("Miles: %f mi\n", miles)
    
    // Type definitions cannot be used interchangeably
    var tempC Celsius = 25.0
    // var tempF Fahrenheit = tempC  // Compile error!
    
    // Explicit conversion required for type definitions
    tempF := Fahrenheit(tempC * 9/5 + 32)
    fmt.Printf("Temperature: %f°C = %f°F\n", tempC, tempF)
    
    // Underlying type operations still work
    fmt.Printf("Celsius + 5: %f\n", tempC + 5.0)  // Can use float64 operations
}
```

## Struct Type Declarations

```go
// Basic struct definition
type Person struct {
    FirstName string
    LastName  string
    Age       int
    Email     string
}

// Struct with tags (used for reflection, JSON marshaling, etc.)
type User struct {
    ID        int    `json:"id" db:"id"`
    Username  string `json:"username" db:"username"`
    Password  string `json:"-" db:"password"`        // "-" omits from JSON
    CreatedAt string `json:"created_at" db:"created_at"`
}

// Anonymous struct fields (embedding)
type ContactInfo struct {
    Phone string
    Email string
}

type Employee struct {
    Person      // Embedded struct
    EmployeeID  int
    Department  string
    ContactInfo // Anonymous embedding
}

func demonstrateStructTypes() {
    // Creating instances
    person := Person{
        FirstName: "John",
        LastName:  "Doe",
        Age:       30,
        Email:     "john.doe@example.com",
    }
    
    user := User{
        ID:        1,
        Username:  "johndoe",
        Password:  "secret123",
        CreatedAt: "2023-01-01",
    }
    
    employee := Employee{
        Person: Person{
            FirstName: "Alice",
            LastName:  "Smith",
            Age:       28,
            Email:     "alice.smith@company.com",
        },
        EmployeeID: 1001,
        Department: "Engineering",
        ContactInfo: ContactInfo{
            Phone: "555-1234",
            Email: "alice.smith@company.com",
        },
    }
    
    fmt.Printf("Person: %+v\n", person)
    fmt.Printf("User: %+v\n", user)
    fmt.Printf("Employee: %+v\n", employee)
    
    // Accessing embedded fields
    fmt.Printf("Employee name: %s %s\n", employee.FirstName, employee.LastName)
    fmt.Printf("Employee phone: %s\n", employee.Phone)  // Direct access due to anonymous embedding
}
```

## Interface Type Declarations

```go
// Basic interface definition
type Writer interface {
    Write([]byte) (int, error)
}

type Reader interface {
    Read([]byte) (int, error)
}

// Interface composition
type ReadWriter interface {
    Reader
    Writer
}

// Interface with method sets
type Shape interface {
    Area() float64
    Perimeter() float64
}

type Drawable interface {
    Draw()
    String() string
}

// Empty interface (can hold any type)
type Any interface{}

func demonstrateInterfaceTypes() {
    // Interface implementation is implicit
    var w Writer = &File{}  // File implements Writer interface
    var rw ReadWriter = &Buffer{}  // Buffer implements both Reader and Writer
    
    fmt.Printf("Writer: %T\n", w)
    fmt.Printf("ReadWriter: %T\n", rw)
    
    // Empty interface can hold any value
    var anything Any
    anything = 42
    fmt.Printf("Anything as int: %v\n", anything)
    
    anything = "Hello"
    fmt.Printf("Anything as string: %v\n", anything)
    
    anything = Person{FirstName: "Test", LastName: "User"}
    fmt.Printf("Anything as struct: %v\n", anything)
}

// Implementing interfaces
type File struct {
    name string
}

func (f *File) Write(data []byte) (int, error) {
    fmt.Printf("Writing %d bytes to file %s\n", len(data), f.name)
    return len(data), nil
}

func (f *File) Read(data []byte) (int, error) {
    fmt.Printf("Reading from file %s\n", f.name)
    return 0, nil
}

type Buffer struct {
    data []byte
}

func (b *Buffer) Write(data []byte) (int, error) {
    b.data = append(b.data, data...)
    fmt.Printf("Buffer wrote %d bytes\n", len(data))
    return len(data), nil
}

func (b *Buffer) Read(data []byte) (int, error) {
    n := copy(data, b.data)
    b.data = b.data[n:]
    fmt.Printf("Buffer read %d bytes\n", n)
    return n, nil
}
```

## Function Type Declarations

```go
// Function type definition
type Operation func(int, int) int
type Predicate func(interface{}) bool
type Handler func(string) error

// Function types with multiple return values
type Calculator func(float64, float64) (float64, error)
type Converter func(interface{}) (interface{}, error)

func demonstrateFunctionTypes() {
    // Using function types
    var add Operation = func(a, b int) int {
        return a + b
    }
    
    var multiply Operation = func(a, b int) int {
        return a * b
    }
    
    fmt.Printf("Add result: %d\n", add(5, 3))
    fmt.Printf("Multiply result: %d\n", multiply(5, 3))
    
    // Function type in struct
    type MathProcessor struct {
        operations map[string]Operation
    }
    
    processor := MathProcessor{
        operations: map[string]Operation{
            "add":      add,
            "multiply": multiply,
            "subtract": func(a, b int) int { return a - b },
            "divide":   func(a, b int) int { 
                if b != 0 { 
                    return a / b 
                }
                return 0
            },
        },
    }
    
    fmt.Printf("Processor add: %d\n", processor.operations["add"](10, 5))
    fmt.Printf("Processor multiply: %d\n", processor.operations["multiply"](10, 5))
}

// Higher-order functions
func ApplyOperation(op Operation, a, b int) int {
    return op(a, b)
}

func CreateMultiplier(factor int) Operation {
    return func(x int) int {
        return x * factor
    }
}
```

## Generic Type Declarations (Go 1.18+)

```go
// Generic type constraint interfaces
type Number interface {
    int | int32 | int64 | float32 | float64
}

type Comparable interface {
    comparable
}

// Generic types
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

func (s *Stack[T]) Len() int {
    return len(s.items)
}

// Generic functions
func Map[T any, R any](slice []T, fn func(T) R) []R {
    result := make([]R, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

func Filter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

func demonstrateGenericTypes() {
    // Using generic stack
    var intStack Stack[int]
    intStack.Push(1)
    intStack.Push(2)
    intStack.Push(3)
    
    if item, ok := intStack.Pop(); ok {
        fmt.Printf("Popped: %d\n", item)
    }
    fmt.Printf("Stack length: %d\n", intStack.Len())
    
    // Using generic functions
    numbers := []int{1, 2, 3, 4, 5}
    doubled := Map(numbers, func(x int) int { return x * 2 })
    fmt.Printf("Doubled: %v\n", doubled)
    
    evens := Filter(numbers, func(x int) bool { return x%2 == 0 })
    fmt.Printf("Even numbers: %v\n", evens)
    
    // Generic stack with strings
    var stringStack Stack[string]
    stringStack.Push("Hello")
    stringStack.Push("World")
    fmt.Printf("String stack length: %d\n", stringStack.Len())
}
```

## Type Assertion and Reflection

```go
import (
    "reflect"
)

func demonstrateTypeAssertion() {
    // Type assertion with interface{}
    var val interface{} = "Hello, Go!"
    
    // Safe type assertion
    if str, ok := val.(string); ok {
        fmt.Printf("String value: %s\n", str)
    }
    
    // Unsafe type assertion (panics if wrong type)
    str := val.(string)
    fmt.Printf("Unsafe assertion: %s\n", str)
    
    // Type switching
    describeType(val)
    describeType(42)
    describeType(true)
    describeType(Person{FirstName: "John"})
}

func describeType(val interface{}) {
    switch v := val.(type) {
    case string:
        fmt.Printf("String: %s (length: %d)\n", v, len(v))
    case int:
        fmt.Printf("Integer: %d\n", v)
    case bool:
        fmt.Printf("Boolean: %t\n", v)
    case Person:
        fmt.Printf("Person: %s %s\n", v.FirstName, v.LastName)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}

func demonstrateReflection() {
    person := Person{
        FirstName: "Alice",
        LastName:  "Johnson",
        Age:       30,
        Email:     "alice@example.com",
    }
    
    // Using reflection to examine type
    t := reflect.TypeOf(person)
    v := reflect.ValueOf(person)
    
    fmt.Printf("Type: %s\n", t.Name())
    fmt.Printf("Kind: %s\n", t.Kind())
    fmt.Printf("Number of fields: %d\n", t.NumField())
    
    // Iterate through fields
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("Field %s: %v (tag: %s)\n", 
                   field.Name, value.Interface(), field.Tag)
    }
    
    // Modify struct using reflection (requires pointer and settable value)
    ptr := reflect.ValueOf(&person)
    elem := ptr.Elem()
    if elem.Kind() == reflect.Struct {
        nameField := elem.FieldByName("FirstName")
        if nameField.CanSet() {
            nameField.SetString("Bob")
        }
    }
    fmt.Printf("Modified person: %+v\n", person)
}
```

## Type Declaration Best Practices

```go
// Good naming conventions
type UserID int64
type UserName string
type EmailAddress string

// Domain-specific types
type TemperatureCelsius float64
type DistanceMeters float64
type CurrencyUSD float64

// Method sets for domain types
func (t TemperatureCelsius) Fahrenheit() float64 {
    return float64(t)*9/5 + 32
}

func (d DistanceMeters) Kilometers() float64 {
    return float64(d) / 1000
}

func (c CurrencyUSD) String() string {
    return fmt.Sprintf("$%.2f", c)
}

func demonstrateBestPractices() {
    var temp TemperatureCelsius = 25.0
    var dist DistanceMeters = 5000
    var price CurrencyUSD = 29.99
    
    fmt.Printf("Temperature: %f°C = %f°F\n", temp, temp.Fahrenheit())
    fmt.Printf("Distance: %f meters = %f kilometers\n", dist, dist.Kilometers())
    fmt.Printf("Price: %s\n", price.String())
    
    // Type safety benefits
    // processTemperature(dist)  // Compile error - type mismatch
    // processDistance(temp)     // Compile error - type mismatch
}

// Avoid these anti-patterns
/*
// Don't create unnecessary type aliases
type MyInt = int  // Usually unnecessary

// Don't overuse empty interfaces
func ProcessAnything(data interface{}) {
    // Hard to maintain and type-unsafe
}

// Don't create deeply nested generic types without clear purpose
type ComplexGeneric[T any, U any, V any] struct {
    // Usually indicates poor design
}
*/
```

Type declarations are fundamental to Go's type system, enabling type safety, code organization, and domain modeling. Proper use of type definitions, interfaces, and generics leads to more maintainable and robust Go applications.
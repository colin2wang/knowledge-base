# Go Functions and Methods

Functions and methods are fundamental building blocks in Go, providing the means to organize code, implement logic, and define behavior. Go supports first-class functions, closures, variadic functions, and methods with both value and pointer receivers.

## Function Basics

```go
package main

import (
    "fmt"
    "math"
)

// Simple function declaration
func add(a, b int) int {
    return a + b
}

// Function with multiple return values
func divide(dividend, divisor int) (int, error) {
    if divisor == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    return dividend / divisor, nil
}

// Function with named return values
func calculateStats(numbers []float64) (min, max, average float64) {
    if len(numbers) == 0 {
        return 0, 0, 0
    }
    
    min, max = numbers[0], numbers[0]
    var sum float64
    
    for _, num := range numbers {
        if num < min {
            min = num
        }
        if num > max {
            max = num
        }
        sum += num
    }
    
    average = sum / float64(len(numbers))
    return  // Named returns automatically returned
}

func demonstrateBasicFunctions() {
    // Function calls
    result := add(5, 3)
    fmt.Printf("5 + 3 = %d\n", result)
    
    // Multiple return values
    quotient, err := divide(10, 2)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("10 / 2 = %d\n", quotient)
    }
    
    // Handling division by zero
    _, err = divide(10, 0)
    if err != nil {
        fmt.Printf("Expected error: %v\n", err)
    }
    
    // Named return values
    numbers := []float64{1.5, 2.3, 0.7, 4.2, 3.1}
    min, max, avg := calculateStats(numbers)
    fmt.Printf("Stats - Min: %.2f, Max: %.2f, Average: %.2f\n", min, max, avg)
}
```

## Function Parameters and Return Types

```go
// Function with different parameter types
func processUserData(name string, age int, isActive bool) string {
    status := "inactive"
    if isActive {
        status = "active"
    }
    return fmt.Sprintf("User %s (age %d) is %s", name, age, status)
}

// Variadic functions (variable number of arguments)
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

// Function with slice parameters
func findMax(numbers []int) (int, bool) {
    if len(numbers) == 0 {
        return 0, false
    }
    
    max := numbers[0]
    for _, num := range numbers[1:] {
        if num > max {
            max = num
        }
    }
    return max, true
}

// Function returning function
func createMultiplier(factor int) func(int) int {
    return func(x int) int {
        return x * factor
    }
}

func demonstrateFunctionParameters() {
    // Different parameter types
    userInfo := processUserData("Alice", 28, true)
    fmt.Println(userInfo)
    
    // Variadic function calls
    fmt.Printf("Sum of 1,2,3,4,5: %d\n", sum(1, 2, 3, 4, 5))
    fmt.Printf("Sum of 10,20: %d\n", sum(10, 20))
    fmt.Printf("Sum of no args: %d\n", sum())  // Empty slice
    
    // Passing slice to variadic function
    nums := []int{1, 2, 3, 4, 5}
    fmt.Printf("Sum of slice: %d\n", sum(nums...))  // Unpacking slice
    
    // Function returning function
    doubler := createMultiplier(2)
    tripler := createMultiplier(3)
    
    fmt.Printf("Double of 5: %d\n", doubler(5))
    fmt.Printf("Triple of 5: %d\n", tripler(5))
}
```

## Anonymous Functions and Closures

```go
func demonstrateAnonymousFunctions() {
    // Anonymous function assigned to variable
    square := func(x int) int {
        return x * x
    }
    
    fmt.Printf("Square of 4: %d\n", square(4))
    
    // Immediate function execution
    result := func(a, b int) int {
        return a * b
    }(6, 7)
    fmt.Printf("Immediate execution result: %d\n", result)
    
    // Closure example - function capturing variables from enclosing scope
    counter := 0
    increment := func() int {
        counter++
        return counter
    }
    
    fmt.Printf("First call: %d\n", increment())
    fmt.Printf("Second call: %d\n", increment())
    fmt.Printf("Third call: %d\n", increment())
    
    // Closure with captured variables
    createCounter := func(initial int) func() int {
        count := initial
        return func() int {
            count++
            return count
        }
    }
    
    counter1 := createCounter(0)
    counter2 := createCounter(100)
    
    fmt.Printf("Counter1: %d, %d, %d\n", counter1(), counter1(), counter1())
    fmt.Printf("Counter2: %d, %d, %d\n", counter2(), counter2(), counter2())
}

func demonstrateClosureUseCases() {
    // Configurable function using closure
    createValidator := func(minLength int) func(string) bool {
        return func(s string) bool {
            return len(s) >= minLength
        }
    }
    
    validateUsername := createValidator(3)
    validatePassword := createValidator(8)
    
    fmt.Printf("Username 'ab' valid: %t\n", validateUsername("ab"))
    fmt.Printf("Username 'alice' valid: %t\n", validateUsername("alice"))
    fmt.Printf("Password '123' valid: %t\n", validatePassword("123"))
    fmt.Printf("Password 'secretpass' valid: %t\n", validatePassword("secretpass"))
    
    // Stateful processing
    createAverager := func() func(float64) float64 {
        var sum float64
        var count int
        return func(value float64) float64 {
            sum += value
            count++
            return sum / float64(count)
        }
    }
    
    averager := createAverager()
    fmt.Printf("Running average: %.2f\n", averager(10))
    fmt.Printf("Running average: %.2f\n", averager(20))
    fmt.Printf("Running average: %.2f\n", averager(30))
}
```

## Method Declarations

```go
// Struct for method examples
type Rectangle struct {
    Width, Height float64
}

type Circle struct {
    Radius float64
}

// Method with value receiver
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Method with pointer receiver
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// Method with value receiver for Circle
func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

// Method with pointer receiver for Circle
func (c *Circle) SetRadius(radius float64) {
    c.Radius = radius
}

func demonstrateMethods() {
    // Value receiver methods
    rect := Rectangle{Width: 10, Height: 5}
    fmt.Printf("Rectangle area: %.2f\n", rect.Area())
    
    circle := Circle{Radius: 3}
    fmt.Printf("Circle area: %.2f\n", circle.Area())
    
    // Pointer receiver methods
    rect.Scale(2.0)
    fmt.Printf("Scaled rectangle dimensions: %.2fx%.2f\n", rect.Width, rect.Height)
    
    circle.SetRadius(5)
    fmt.Printf("Circle new area: %.2f\n", circle.Area())
    
    // Method calls on pointers
    rectPtr := &rect
    fmt.Printf("Area via pointer: %.2f\n", rectPtr.Area())  // Automatic dereferencing
    
    circlePtr := &circle
    circlePtr.SetRadius(7)
    fmt.Printf("Area via pointer: %.2f\n", circlePtr.Area())
}

// Interface for method demonstration
type Shape interface {
    Area() float64
}

func demonstrateInterfaceMethods() {
    // Polymorphic behavior
    shapes := []Shape{
        Rectangle{Width: 5, Height: 3},
        Circle{Radius: 2},
        Rectangle{Width: 4, Height: 4},
    }
    
    fmt.Println("Shape areas:")
    for i, shape := range shapes {
        fmt.Printf("Shape %d: %.2f\n", i+1, shape.Area())
    }
}
```

## Method Sets and Receiver Types

```go
type Counter struct {
    count int
}

// Method with pointer receiver
func (c *Counter) Increment() {
    c.count++
}

// Method with value receiver
func (c Counter) GetValue() int {
    return c.count
}

// Method with pointer receiver
func (c *Counter) Reset() {
    c.count = 0
}

func demonstrateMethodSets() {
    // Variable of type Counter
    var c1 Counter
    
    // Methods available on value
    fmt.Printf("Initial value: %d\n", c1.GetValue())
    // c1.Increment()  // This won't work - Increment has pointer receiver
    
    // Methods available on pointer
    c1Ptr := &c1
    c1Ptr.Increment()
    c1Ptr.Increment()
    fmt.Printf("After incrementing: %d\n", c1.GetValue())
    c1Ptr.Reset()
    fmt.Printf("After resetting: %d\n", c1.GetValue())
    
    // Automatic address taking
    c1.Increment()  // Works because Go automatically takes address
    fmt.Printf("Auto-address taking: %d\n", c1.GetValue())
}

// Interface satisfaction demonstration
type Stringer interface {
    String() string
}

type Person struct {
    FirstName string
    LastName  string
}

// Value receiver method satisfying interface
func (p Person) String() string {
    return fmt.Sprintf("%s %s", p.FirstName, p.LastName)
}

func demonstrateInterfaceSatisfaction() {
    person := Person{FirstName: "John", LastName: "Doe"}
    
    // Person satisfies Stringer interface
    var s Stringer = person
    fmt.Printf("Stringer output: %s\n", s.String())
    
    // Interface method calls
    printStringer(person)
    printStringer(Rectangle{Width: 3, Height: 4})
    printStringer(Circle{Radius: 2})
}

func printStringer(s Stringer) {
    fmt.Printf("Printed: %s\n", s.String())
}

// String method for Rectangle
func (r Rectangle) String() string {
    return fmt.Sprintf("Rectangle(%.2fx%.2f)", r.Width, r.Height)
}

// String method for Circle
func (c Circle) String() string {
    return fmt.Sprintf("Circle(radius=%.2f)", c.Radius)
}
```

## Function Literals and Higher-Order Functions

```go
// Higher-order function - takes function as parameter
func applyOperation(numbers []int, operation func(int) int) []int {
    result := make([]int, len(numbers))
    for i, num := range numbers {
        result[i] = operation(num)
    }
    return result
}

// Higher-order function - returns function
func createPowerFunction(exponent int) func(float64) float64 {
    return func(base float64) float64 {
        return math.Pow(base, float64(exponent))
    }
}

func demonstrateHigherOrderFunctions() {
    numbers := []int{1, 2, 3, 4, 5}
    
    // Using higher-order function with anonymous function
    squared := applyOperation(numbers, func(x int) int {
        return x * x
    })
    fmt.Printf("Squared: %v\n", squared)
    
    // Using higher-order function with named function
    doubled := applyOperation(numbers, func(x int) int {
        return x * 2
    })
    fmt.Printf("Doubled: %v\n", doubled)
    
    // Function returning function
    square := createPowerFunction(2)
    cube := createPowerFunction(3)
    
    fmt.Printf("Square of 4: %.2f\n", square(4))
    fmt.Printf("Cube of 4: %.2f\n", cube(4))
    
    // Function composition
    compose := func(f, g func(int) int) func(int) int {
        return func(x int) int {
            return f(g(x))
        }
    }
    
    addOne := func(x int) int { return x + 1 }
    double := func(x int) int { return x * 2 }
    
    addOneThenDouble := compose(double, addOne)
    fmt.Printf("(5 + 1) * 2 = %d\n", addOneThenDouble(5))
}
```

## Deferred Functions and Panic/Recover

```go
func demonstrateDefer() {
    // Basic defer usage
    defer fmt.Println("This will execute last")
    defer fmt.Println("This will execute second")
    fmt.Println("This executes first")
    
    // Deferred function with parameters evaluated immediately
    x := 10
    defer fmt.Printf("Deferred x value: %d\n", x)  // x is 10 when deferred
    x = 20
    fmt.Printf("Current x value: %d\n", x)  // x is 20 here
}

func demonstratePanicRecover() {
    // Function that panics
    mightPanic := func(shouldPanic bool) {
        if shouldPanic {
            panic("Something went wrong!")
        }
        fmt.Println("Everything is fine")
    }
    
    // Recovery function
    safeCall := func() {
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Recovered from panic: %v\n", r)
            }
        }()
        
        mightPanic(true)  // This will panic
        fmt.Println("This won't execute")
    }
    
    fmt.Println("Calling safe function:")
    safeCall()
    fmt.Println("Program continues after recovery")
    
    // Panic with different types
    panicWithRecovery(func() {
        panic(42)  // Panic with integer
    })
    
    panicWithRecovery(func() {
        panic(fmt.Errorf("error message"))  // Panic with error
    })
}

func panicWithRecovery(panicker func()) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("Recovered from panic of type %T: %v\n", r, r)
        }
    }()
    
    panicker()
}

// Resource management with defer
func demonstrateResourceManagement() {
    // File handling with defer
    processFile := func(filename string) error {
        file, err := os.Open(filename)
        if err != nil {
            return fmt.Errorf("failed to open file: %w", err)
        }
        defer file.Close()  // Ensure file is closed
        
        // Process file content
        fmt.Printf("Processing file: %s\n", filename)
        return nil
    }
    
    // Mutex protection with defer
    type SafeCounter struct {
        mu    sync.Mutex
        count int
    }
    
    incrementSafely := func(sc *SafeCounter) {
        sc.mu.Lock()
        defer sc.mu.Unlock()  // Ensure unlock
        
        sc.count++
        fmt.Printf("Count incremented to: %d\n", sc.count)
    }
    
    counter := &SafeCounter{}
    incrementSafely(counter)
    incrementSafely(counter)
}
```

## Function Best Practices

```go
import (
    "context"
    "sync"
    "time"
)

// Good function naming and structure
func CalculateMonthlyInterest(principal float64, rate float64, months int) float64 {
    if principal <= 0 || rate < 0 || months <= 0 {
        return 0
    }
    return principal * rate * float64(months) / 12
}

// Function with context for cancellation
func ProcessDataWithContext(ctx context.Context, data []int) ([]int, error) {
    result := make([]int, len(data))
    
    for i, value := range data {
        // Check for cancellation
        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        default:
            // Simulate processing
            time.Sleep(10 * time.Millisecond)
            result[i] = value * 2
        }
    }
    
    return result, nil
}

// Concurrent function with wait group
func ProcessBatchConcurrently(data [][]int, processor func([]int) []int) [][]int {
    var wg sync.WaitGroup
    results := make([][]int, len(data))
    
    for i, batch := range data {
        wg.Add(1)
        go func(index int, batchData []int) {
            defer wg.Done()
            results[index] = processor(batchData)
        }(i, batch)
    }
    
    wg.Wait()
    return results
}

func demonstrateBestPractices() {
    // Clear function names
    interest := CalculateMonthlyInterest(10000, 0.05, 12)
    fmt.Printf("Monthly interest: %.2f\n", interest)
    
    // Context usage
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()
    
    largeData := make([]int, 1000)
    for i := range largeData {
        largeData[i] = i
    }
    
    result, err := ProcessDataWithContext(ctx, largeData)
    if err != nil {
        fmt.Printf("Processing error: %v\n", err)
    } else {
        fmt.Printf("Processed %d items\n", len(result))
    }
    
    // Concurrent processing
    batches := [][]int{
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
    }
    
    processed := ProcessBatchConcurrently(batches, func(data []int) []int {
        result := make([]int, len(data))
        for i, v := range data {
            result[i] = v * v
        }
        return result
    })
    
    fmt.Printf("Concurrent results: %v\n", processed)
}

/*
// Anti-patterns to avoid:

// Don't use too many parameters
func BadFunction(a, b, c, d, e, f, g, h int) int {  // Too many parameters
    // Complex logic
    return a + b + c + d + e + f + g + h
}

// Better approach - use struct
type FunctionParams struct {
    A, B, C, D, E, F, G, H int
}

func GoodFunction(params FunctionParams) int {
    return params.A + params.B + params.C + params.D + 
           params.E + params.F + params.G + params.H
}

// Don't ignore errors
func BadWithError() int {
    result, _ := divide(10, 0)  // Ignoring error
    return result
}

// Always handle errors properly
func GoodWithError() (int, error) {
    result, err := divide(10, 0)
    if err != nil {
        return 0, fmt.Errorf("calculation failed: %w", err)
    }
    return result, nil
}
*/
```

Functions and methods in Go provide powerful mechanisms for code organization, reuse, and abstraction. Understanding their various forms, from simple functions to complex closures and methods with different receiver types, is essential for writing idiomatic Go code.
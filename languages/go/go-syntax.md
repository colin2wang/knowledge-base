# Go Syntax Guide

This guide covers the basic syntax and fundamental concepts of the Go programming language.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Variables and Data Types](#variables-and-data-types)
- [Control Flow](#control-flow)
- [Functions](#functions)
- [Structs](#structs)
- [Interfaces](#interfaces)
- [Concurrency](#concurrency)
- [Error Handling](#error-handling)
- [Packages](#packages)
- [Memory Management](#memory-management)

## Basic Structure

```go
package main  // Package declaration

import "fmt"  // Import statement

// Main function - entry point of the program
func main() {
    fmt.Println("Hello, World!")
}
```

## Variables and Data Types

### Variable Declaration
```go
// Using var keyword
var age int
var name string = "Alice"
var isActive bool = true

// Multiple variables
var x, y int = 1, 2
var a, b = 3.14, "hello"

// Short variable declaration (inside functions only)
age := 25
name := "Bob"
isActive := false
```

### Basic Data Types
```go
// Numeric types
int     // Integer (platform-dependent size)
int8    // 8-bit signed integer
int16   // 16-bit signed integer
int32   // 32-bit signed integer
int64   // 64-bit signed integer
uint    // Unsigned integer (platform-dependent size)
uint8   // 8-bit unsigned integer
uint16  // 16-bit unsigned integer
uint32  // 32-bit unsigned integer
uint64  // 64-bit unsigned integer
uintptr // Unsigned integer large enough to hold a pointer

// Floating point types
float32 // 32-bit floating point
float64 // 64-bit floating point (default)

// Complex types
complex64  // Complex number with float32 real and imaginary parts
complex128 // Complex number with float64 real and imaginary parts (default)

// Boolean type
bool // true or false

// String type
string // UTF-8 encoded string

// Character type
byte   // Alias for uint8
rune   // Alias for int32 (represents a Unicode code point)
```

### Constants
```go
// Using const keyword
const Pi = 3.14159
const MaxAge int = 100
const (
    Monday = 1
    Tuesday = 2
    Wednesday = 3
)

// iota for incrementing constants
const (
    Zero = iota // 0
    One         // 1
    Two         // 2
)
```

## Control Flow

### Conditional Statements
```go
// If statement
if age >= 18 {
    fmt.Println("Adult")
} else if age >= 13 {
    fmt.Println("Teenager")
} else {
    fmt.Println("Child")
}

// If with initialization
if name := getName(); name != "" {
    fmt.Println("Hello, " + name)
}

// Switch statement
switch day {
case Monday:
    fmt.Println("Start of week")
case Friday:
    fmt.Println("End of week")
default:
    fmt.Println("Midweek")
}

// Switch with no condition (like if-else chain)
switch {
case age < 13:
    fmt.Println("Child")
case age < 18:
    fmt.Println("Teenager")
default:
    fmt.Println("Adult")
}
```

### Loops
```go
// For loop (only loop type in Go)
for i := 0; i < 5; i++ {
    fmt.Println(i)
}

// While-like loop
for x < 10 {
    x++
    fmt.Println(x)
}

// Infinite loop
for {
    fmt.Println("Running...")
    // break to exit
}

// For range (iterating over collections)
numbers := []int{1, 2, 3, 4, 5}
for index, value := range numbers {
    fmt.Printf("Index: %d, Value: %d\n", index, value)
}

// Iterating over strings (returns index and rune)
for i, r := range "Hello" {
    fmt.Printf("Index: %d, Rune: %c\n", i, r)
}
```

## Functions

### Basic Function
```go
// Function definition
func add(a int, b int) int {
    return a + b
}

// Function call
result := add(5, 3)

// Multiple return values
func swap(a, b string) (string, string) {
    return b, a
}

x, y := swap("hello", "world")
```

### Named Return Values
```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return // Naked return - returns x and y
}
```

### Variadic Functions
```go
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

// Usage
total := sum(1, 2, 3, 4, 5)
slice := []int{1, 2, 3}
total := sum(slice...)
```

### Anonymous Functions (Closures)
```go
add := func(a, b int) int {
    return a + b
}

result := add(5, 3)

// Closure that captures variables
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

c := counter()
fmt.Println(c()) // 1
fmt.Println(c()) // 2
```

## Structs

### Struct Definition
```go
type Person struct {
    Name string
    Age  int
    City string
}

// Creating struct instances
p1 := Person{"Alice", 25, "New York"}
p2 := Person{Name: "Bob", Age: 30, City: "London"}
p3 := Person{Name: "Charlie"} // Age and City are zero values

// Accessing fields
fmt.Println(p1.Name)
p2.Age = 31
```

### Struct Methods
```go
// Method with value receiver
func (p Person) GetInfo() string {
    return fmt.Sprintf("%s, %d years old from %s", p.Name, p.Age, p.City)
}

// Method with pointer receiver (modifies the struct)
func (p *Person) UpdateAge(newAge int) {
    p.Age = newAge
}

// Usage
p := Person{"Alice", 25, "New York"}
fmt.Println(p.GetInfo())
p.UpdateAge(26)
fmt.Println(p.Age)
```

## Interfaces

### Interface Definition
```go
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Implementing interface
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

// Using interface
func PrintShapeInfo(s Shape) {
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", s.Area(), s.Perimeter())
}

// Usage
r := Rectangle{Width: 5, Height: 3}
c := Circle{Radius: 2}
PrintShapeInfo(r)
PrintShapeInfo(c)
```

## Concurrency

### Goroutines
```go
// Starting a goroutine
func sayHello() {
    fmt.Println("Hello from goroutine")
}

func main() {
    go sayHello() // Starts new goroutine
    fmt.Println("Hello from main")
    time.Sleep(1 * time.Second) // Wait for goroutine to complete
}
```

### Channels
```go
// Creating a channel
ch := make(chan int)

// Sending to channel
ch <- 42

// Receiving from channel
value := <-ch

// Buffered channel
ch := make(chan int, 3) // Buffer size of 3

// Channel with range
func sendNumbers(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch) // Close channel to signal completion
}

func main() {
    ch := make(chan int)
    go sendNumbers(ch)
    
    for num := range ch {
        fmt.Println(num)
    }
}
```

### Select Statement
```go
func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "one"
    }()
    
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "two"
    }()
    
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Received", msg1)
        case msg2 := <-ch2:
            fmt.Println("Received", msg2)
        }
    }
}
```

## Error Handling

### Basic Error Handling
```go
// Returning errors
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Using errors
result, err := divide(10, 2)
if err != nil {
    fmt.Println("Error:", err)
    return
}
fmt.Println("Result:", result)
```

### Custom Errors
```go
type MyError struct {
    Message string
    Code    int
}

func (e *MyError) Error() string {
    return fmt.Sprintf("%s (code: %d)", e.Message, e.Code)
}

func doSomething() error {
    return &MyError{"Operation failed", 500}
}
```

## Packages

### Creating and Using Packages
```go
// In greetings/greetings.go
package greetings

import "fmt"

func Hello(name string) string {
    return fmt.Sprintf("Hello, %s!", name)
}

// In main.go
package main

import (
    "fmt"
    "example.com/greetings"
)

func main() {
    message := greetings.Hello("Alice")
    fmt.Println(message)
}
```

## Memory Management

### Pointers
```go
// Creating a pointer
var x int = 10
var ptr *int = &x // Pointer to x

// Dereferencing a pointer
fmt.Println(*ptr) // Prints 10
*ptr = 20         // Changes x to 20

// Zero value of a pointer is nil
var p *int
if p == nil {
    fmt.Println("Pointer is nil")
}

// New function
ptr := new(int) // Allocates memory for an int
*ptr = 100
```

### Slice
```go
// Creating slices
var s []int                // Zero value is nil
empty := make([]int, 0)    // Empty slice
numbers := []int{1, 2, 3}  // Slice with elements
slice := make([]int, 3, 5) // Slice with length 3 and capacity 5

// Slice operations
appendSlice := append(numbers, 4, 5)
subSlice := numbers[1:3]   // Elements from index 1 to 2 (exclusive of 3)

// Slice length and capacity
len(numbers)  // Number of elements
cap(numbers)  // Capacity (how much it can grow without reallocating)
```

### Map
```go
// Creating maps
var m map[string]int                // Zero value is nil
m = make(map[string]int)            // Initialize map
ages := map[string]int{             // Map with initial values
    "Alice": 25,
    "Bob":   30,
}

// Map operations
ages["Charlie"] = 35               // Add key-value pair
value, exists := ages["Alice"]      // Check if key exists
if exists {
    fmt.Println("Alice's age:", value)
}

// Delete from map
delete(ages, "Bob")

// Iterate over map
for name, age := range ages {
    fmt.Printf("%s: %d\n", name, age)
}
```

## Useful Resources
- [Go Documentation](https://golang.org/doc/)
- [Go by Example](https://gobyexample.com/)
- [Effective Go](https://golang.org/doc/effective_go.html)
# Go Primitive Types

Go provides a rich set of primitive data types that form the foundation of the language. Understanding these types is essential for writing efficient and type-safe Go code.

## Numeric Types

### Integer Types

Go offers both signed and unsigned integer types with different sizes:

```go
package main

import (
    "fmt"
    "math"
)

func demonstrateIntegers() {
    // Signed integers
    var int8Var int8 = 127           // Range: -128 to 127
    var int16Var int16 = 32767       // Range: -32,768 to 32,767
    var int32Var int32 = 2147483647  // Range: -2,147,483,648 to 2,147,483,647
    var int64Var int64 = 9223372036854775807 // Range: -2^63 to 2^63-1
    
    // Unsigned integers
    var uint8Var uint8 = 255         // Range: 0 to 255
    var uint16Var uint16 = 65535     // Range: 0 to 65,535
    var uint32Var uint32 = 4294967295 // Range: 0 to 4,294,967,295
    var uint64Var uint64 = 18446744073709551615 // Range: 0 to 2^64-1
    
    // Platform-dependent sizes
    var intVar int = 42              // Either 32 or 64 bits depending on platform
    var uintVar uint = 42            // Either 32 or 64 bits depending on platform
    var uintptrVar uintptr = 42      // Large enough to store pointer values
    
    fmt.Println("Signed integers:")
    fmt.Printf("int8: %d\n", int8Var)
    fmt.Printf("int16: %d\n", int16Var)
    fmt.Printf("int32: %d\n", int32Var)
    fmt.Printf("int64: %d\n", int64Var)
    
    fmt.Println("\nUnsigned integers:")
    fmt.Printf("uint8: %d\n", uint8Var)
    fmt.Printf("uint16: %d\n", uint16Var)
    fmt.Printf("uint32: %d\n", uint32Var)
    fmt.Printf("uint64: %d\n", uint64Var)
    
    fmt.Println("\nPlatform-dependent:")
    fmt.Printf("int: %d\n", intVar)
    fmt.Printf("uint: %d\n", uintVar)
    fmt.Printf("uintptr: %d\n", uintptrVar)
}

func demonstrateIntegerLimits() {
    fmt.Println("Integer type limits:")
    fmt.Printf("int8 min: %d, max: %d\n", math.MinInt8, math.MaxInt8)
    fmt.Printf("int16 min: %d, max: %d\n", math.MinInt16, math.MaxInt16)
    fmt.Printf("int32 min: %d, max: %d\n", math.MinInt32, math.MaxInt32)
    fmt.Printf("int64 min: %d, max: %d\n", math.MinInt64, math.MaxInt64)
    
    fmt.Printf("uint8 max: %d\n", math.MaxUint8)
    fmt.Printf("uint16 max: %d\n", math.MaxUint16)
    fmt.Printf("uint32 max: %d\n", math.MaxUint32)
    // Note: No math.MaxUint64 constant, but can calculate: ^uint64(0)
    fmt.Printf("uint64 max: %d\n", ^uint64(0))
}
```

### Floating-Point Types

```go
func demonstrateFloats() {
    // 32-bit floating-point
    var float32Var float32 = 3.14159
    // 64-bit floating-point (default)
    var float64Var float64 = 3.141592653589793
    
    fmt.Println("Floating-point numbers:")
    fmt.Printf("float32: %f\n", float32Var)
    fmt.Printf("float64: %f\n", float64Var)
    
    // Special values
    fmt.Printf("Positive infinity: %f\n", math.Inf(1))
    fmt.Printf("Negative infinity: %f\n", math.Inf(-1))
    fmt.Printf("Not a Number (NaN): %f\n", math.NaN())
    
    // Checking for special values
    fmt.Printf("Is NaN: %t\n", math.IsNaN(float64Var))
    fmt.Printf("Is Inf: %t\n", math.IsInf(float64Var, 0))
}

func demonstrateFloatPrecision() {
    // Precision differences
    f32 := float32(0.1)
    f64 := float64(0.1)
    
    fmt.Printf("float32(0.1): %.20f\n", f32)
    fmt.Printf("float64(0.1): %.20f\n", f64)
    
    // Arithmetic precision
    sum32 := float32(0.1) + float32(0.2)
    sum64 := float64(0.1) + float64(0.2)
    
    fmt.Printf("float32 0.1 + 0.2 = %.20f\n", sum32)
    fmt.Printf("float64 0.1 + 0.2 = %.20f\n", sum64)
}
```

### Complex Types

```go
func demonstrateComplex() {
    // Complex numbers
    var complex64Var complex64 = 3 + 4i
    var complex128Var complex128 = 3.14 + 2.71i
    
    fmt.Println("Complex numbers:")
    fmt.Printf("complex64: %v\n", complex64Var)
    fmt.Printf("complex128: %v\n", complex128Var)
    
    // Accessing real and imaginary parts
    fmt.Printf("Real part of complex64: %f\n", real(complex64Var))
    fmt.Printf("Imaginary part of complex64: %f\n", imag(complex64Var))
    
    // Complex arithmetic
    c1 := 2 + 3i
    c2 := 1 + 4i
    fmt.Printf("Addition: %v + %v = %v\n", c1, c2, c1+c2)
    fmt.Printf("Multiplication: %v * %v = %v\n", c1, c2, c1*c2)
    
    // Complex functions
    fmt.Printf("Magnitude of %v: %f\n", c1, cmplx.Abs(c1))
    fmt.Printf("Phase of %v: %f\n", c1, cmplx.Phase(c1))
}
```

## Boolean Type

```go
func demonstrateBoolean() {
    // Boolean values
    var boolVar bool = true
    var falseBool bool = false
    
    fmt.Println("Boolean values:")
    fmt.Printf("true: %t\n", boolVar)
    fmt.Printf("false: %t\n", falseBool)
    
    // Boolean operations
    fmt.Printf("NOT true: %t\n", !boolVar)
    fmt.Printf("true AND false: %t\n", boolVar && falseBool)
    fmt.Printf("true OR false: %t\n", boolVar || falseBool)
    
    // Boolean expressions
    x, y := 10, 5
    fmt.Printf("%d > %d: %t\n", x, y, x > y)
    fmt.Printf("%d == %d: %t\n", x, y, x == y)
    fmt.Printf("%d != %d: %t\n", x, y, x != y)
    
    // Zero value
    var defaultBool bool // Zero value is false
    fmt.Printf("Zero value of bool: %t\n", defaultBool)
}
```

## String Type

```go
func demonstrateStrings() {
    // String literals
    var str1 string = "Hello, World!"
    str2 := `Raw string literal
with newlines and "quotes"`
    
    fmt.Println("String examples:")
    fmt.Printf("Regular string: %s\n", str1)
    fmt.Printf("Raw string: %s\n", str2)
    
    // String operations
    fmt.Printf("Length of '%s': %d\n", str1, len(str1))
    fmt.Printf("First character: %c\n", str1[0])
    fmt.Printf("Substring: %s\n", str1[0:5])
    
    // String concatenation
    greeting := "Hello"
    name := "Go"
    message := greeting + ", " + name + "!"
    fmt.Printf("Concatenated: %s\n", message)
    
    // String comparison
    fmt.Printf("'hello' == 'Hello': %t\n", "hello" == "Hello")
    fmt.Printf("'abc' < 'abd': %t\n", "abc" < "abd")
    
    // Unicode support
    unicodeStr := "Hello, 世界! 🌍"
    fmt.Printf("Unicode string: %s\n", unicodeStr)
    fmt.Printf("Length in bytes: %d\n", len(unicodeStr))
    fmt.Printf("Runes count: %d\n", len([]rune(unicodeStr)))
}

func demonstrateStringManipulation() {
    // Converting between string and other types
    str := "123"
    num, err := strconv.Atoi(str)
    if err != nil {
        fmt.Printf("Error converting string to int: %v\n", err)
    } else {
        fmt.Printf("Converted '%s' to int: %d\n", str, num)
    }
    
    // Formatting strings
    name := "Alice"
    age := 30
    formatted := fmt.Sprintf("Name: %s, Age: %d", name, age)
    fmt.Printf("Formatted string: %s\n", formatted)
    
    // String builder for efficient concatenation
    var builder strings.Builder
    builder.WriteString("Hello")
    builder.WriteString(" ")
    builder.WriteString("World")
    result := builder.String()
    fmt.Printf("Built string: %s\n", result)
}
```

## Byte and Rune Types

```go
func demonstrateByteAndRune() {
    // byte is alias for uint8
    var b byte = 'A'  // ASCII value 65
    fmt.Printf("Byte 'A': %d (decimal), %c (character)\n", b, b)
    
    // rune is alias for int32, represents Unicode code point
    var r rune = '世'  // Unicode character
    fmt.Printf("Rune '世': %d (decimal), %c (character)\n", r, r)
    
    // Converting strings to bytes and runes
    str := "Hello, 世界!"
    bytes := []byte(str)
    runes := []rune(str)
    
    fmt.Printf("Original string: %s\n", str)
    fmt.Printf("As bytes: %v\n", bytes)
    fmt.Printf("As runes: %v\n", runes)
    fmt.Printf("Byte length: %d\n", len(bytes))
    fmt.Printf("Rune length: %d\n", len(runes))
    
    // Iterating over strings
    fmt.Println("Iterating over string:")
    for i, r := range str {
        fmt.Printf("Index %d: %c (rune %d)\n", i, r, r)
    }
}
```

## Type Conversion and Compatibility

```go
func demonstrateTypeConversion() {
    // Explicit conversions required between numeric types
    var intVar int = 42
    var int64Var int64 = int64(intVar)  // Explicit conversion
    var float64Var float64 = float64(intVar)
    
    fmt.Printf("int to int64: %d -> %d\n", intVar, int64Var)
    fmt.Printf("int to float64: %d -> %f\n", intVar, float64Var)
    
    // Converting between float and int (truncates decimal part)
    var floatVar float64 = 3.14159
    var intFromFloat int = int(floatVar)  // Truncates to 3
    fmt.Printf("float64 to int: %f -> %d\n", floatVar, intFromFloat)
    
    // Converting string to numeric types
    str := "123"
    if num, err := strconv.Atoi(str); err == nil {
        fmt.Printf("String '%s' to int: %d\n", str, num)
    }
    
    if num, err := strconv.ParseFloat("3.14", 64); err == nil {
        fmt.Printf("String '3.14' to float64: %f\n", num)
    }
    
    // Converting numeric types to string
    num := 42
    strFromInt := strconv.Itoa(num)
    fmt.Printf("Int %d to string: '%s'\n", num, strFromInt)
    
    floatNum := 3.14159
    strFromFloat := strconv.FormatFloat(floatNum, 'f', 2, 64)
    fmt.Printf("Float %f to string: '%s'\n", floatNum, strFromFloat)
}

func demonstrateUnsafeConversions() {
    // Using unsafe package for direct memory conversion (use with caution)
    var intVal int64 = 42
    var floatVal float64
    
    // This is unsafe and should be avoided in most cases
    // floatVal = *(*float64)(unsafe.Pointer(&intVal))
    
    fmt.Println("Unsafe conversions should be used sparingly and with extreme caution")
    fmt.Printf("Safe conversion example: int64(%d) to float64: %f\n", 
               intVal, float64(intVal))
}
```

## Zero Values

```go
func demonstrateZeroValues() {
    // Every type has a zero value
    var intZero int
    var floatZero float64
    var boolZero bool
    var stringZero string
    var complexZero complex128
    
    fmt.Println("Zero values:")
    fmt.Printf("int: %d\n", intZero)
    fmt.Printf("float64: %f\n", floatZero)
    fmt.Printf("bool: %t\n", boolZero)
    fmt.Printf("string: '%s'\n", stringZero)
    fmt.Printf("complex128: %v\n", complexZero)
    
    // Zero values for numeric types are 0
    // Zero value for bool is false
    // Zero value for string is empty string ""
    // Zero value for complex types has real and imaginary parts as 0
}
```

Understanding Go's primitive types and their characteristics is fundamental to writing efficient, type-safe Go programs. Each type has specific use cases and performance characteristics that should be considered when designing your applications.
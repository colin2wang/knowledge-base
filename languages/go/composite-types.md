# Go Composite Types

Go provides several composite types that allow you to group and organize data in structured ways. These types include arrays, slices, maps, and structs, each serving different purposes and offering unique capabilities.

## Arrays

Arrays are fixed-size sequences of elements of the same type.

```go
package main

import "fmt"

func demonstrateArrays() {
    // Array declaration and initialization
    var arr1 [5]int                    // Array of 5 integers, zero-valued
    arr2 := [5]int{1, 2, 3, 4, 5}     // Array with initial values
    arr3 := [...]int{10, 20, 30}      // Compiler determines size
    arr4 := [3]string{"a", "b", "c"}  // String array
    
    fmt.Println("Array examples:")
    fmt.Printf("arr1: %v\n", arr1)
    fmt.Printf("arr2: %v\n", arr2)
    fmt.Printf("arr3: %v (length: %d)\n", arr3, len(arr3))
    fmt.Printf("arr4: %v\n", arr4)
    
    // Array access and modification
    fmt.Printf("First element of arr2: %d\n", arr2[0])
    arr2[0] = 100
    fmt.Printf("Modified arr2: %v\n", arr2)
    
    // Array iteration
    fmt.Println("Iterating over arr2:")
    for i, value := range arr2 {
        fmt.Printf("Index %d: %d\n", i, value)
    }
    
    // Multi-dimensional arrays
    var matrix [3][3]int
    matrix[0][0] = 1
    matrix[1][1] = 5
    matrix[2][2] = 9
    
    fmt.Println("Matrix:")
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            fmt.Printf("%d ", matrix[i][j])
        }
        fmt.Println()
    }
}

func demonstrateArrayOperations() {
    // Array copying
    arr1 := [3]int{1, 2, 3}
    arr2 := arr1  // Creates a copy
    arr2[0] = 100
    
    fmt.Printf("Original array: %v\n", arr1)
    fmt.Printf("Copied array: %v\n", arr2)
    
    // Array comparison
    arr3 := [3]int{1, 2, 3}
    arr4 := [3]int{1, 2, 3}
    arr5 := [3]int{1, 2, 4}
    
    fmt.Printf("arr3 == arr4: %t\n", arr3 == arr4)
    fmt.Printf("arr3 == arr5: %t\n", arr3 == arr5)
    
    // Arrays as function parameters (passed by value)
    modifyArray(arr1)
    fmt.Printf("After function call: %v\n", arr1)  // Original unchanged
}

func modifyArray(arr [3]int) {
    arr[0] = 999  // Only modifies local copy
    fmt.Printf("Inside function: %v\n", arr)
}
```

## Slices

Slices are dynamic, flexible views into arrays. They are more commonly used than arrays in Go.

```go
func demonstrateSlices() {
    // Slice creation
    var slice1 []int                    // Nil slice
    slice2 := []int{1, 2, 3, 4, 5}     // Slice literal
    slice3 := make([]int, 5)           // Make with length
    slice4 := make([]int, 3, 10)       // Make with length and capacity
    
    fmt.Println("Slice examples:")
    fmt.Printf("slice1: %v (nil: %t)\n", slice1, slice1 == nil)
    fmt.Printf("slice2: %v (len: %d, cap: %d)\n", slice2, len(slice2), cap(slice2))
    fmt.Printf("slice3: %v (len: %d, cap: %d)\n", slice3, len(slice3), cap(slice3))
    fmt.Printf("slice4: %v (len: %d, cap: %d)\n", slice4, len(slice4), cap(slice4))
    
    // Creating slices from arrays
    arr := [5]int{10, 20, 30, 40, 50}
    slice5 := arr[1:4]    // Elements from index 1 to 3
    slice6 := arr[:3]     // Elements from start to index 2
    slice7 := arr[2:]     // Elements from index 2 to end
    
    fmt.Printf("Original array: %v\n", arr)
    fmt.Printf("slice5 [1:4]: %v\n", slice5)
    fmt.Printf("slice6 [:3]: %v\n", slice6)
    fmt.Printf("slice7 [2:]: %v\n", slice7)
    
    // Slice modification affects underlying array
    slice5[0] = 999
    fmt.Printf("Modified slice5: %v\n", slice5)
    fmt.Printf("Original array after modification: %v\n", arr)
}

func demonstrateSliceOperations() {
    // Appending to slices
    slice := []int{1, 2, 3}
    fmt.Printf("Original slice: %v\n", slice)
    
    slice = append(slice, 4)
    fmt.Printf("After append(4): %v\n", slice)
    
    slice = append(slice, 5, 6, 7)
    fmt.Printf("After append(5,6,7): %v\n", slice)
    
    // Appending another slice
    moreNumbers := []int{8, 9, 10}
    slice = append(slice, moreNumbers...)
    fmt.Printf("After appending slice: %v\n", slice)
    
    // Copying slices
    src := []int{1, 2, 3, 4, 5}
    dst := make([]int, 3)
    n := copy(dst, src)
    fmt.Printf("Copied %d elements: %v\n", n, dst)
    
    // Slice tricks
    // Removing element at index
    slice = []int{1, 2, 3, 4, 5}
    index := 2
    slice = append(slice[:index], slice[index+1:]...)
    fmt.Printf("After removing index %d: %v\n", index, slice)
    
    // Inserting element at index
    slice = []int{1, 2, 4, 5}
    index = 2
    value := 3
    slice = append(slice[:index], append([]int{value}, slice[index:]...)...)
    fmt.Printf("After inserting %d at index %d: %v\n", value, index, slice)
}

func demonstrateSliceInternals() {
    // Understanding slice header: pointer, length, capacity
    slice := make([]int, 5, 10)
    fmt.Printf("Slice: %v\n", slice)
    fmt.Printf("Length: %d\n", len(slice))
    fmt.Printf("Capacity: %d\n", cap(slice))
    
    // Reslicing without allocation
    subslice := slice[2:4]
    fmt.Printf("Subslice [2:4]: %v (len: %d, cap: %d)\n", 
               subslice, len(subslice), cap(subslice))
    
    // When capacity is exceeded, new array is allocated
    fmt.Println("Growing slice beyond capacity:")
    for i := 0; i < 15; i++ {
        slice = append(slice, i)
        fmt.Printf("Length: %d, Capacity: %d\n", len(slice), cap(slice))
    }
}
```

## Maps

Maps are key-value data structures that provide fast lookup operations.

```go
func demonstrateMaps() {
    // Map creation
    var map1 map[string]int              // Nil map
    map2 := make(map[string]int)         // Empty map
    map3 := map[string]int{              // Map literal
        "apple":  5,
        "banana": 3,
        "orange": 8,
    }
    
    fmt.Println("Map examples:")
    fmt.Printf("map1 (nil): %v\n", map1)
    fmt.Printf("map2 (empty): %v\n", map2)
    fmt.Printf("map3: %v\n", map3)
    
    // Map operations
    // Adding elements
    map2["grape"] = 12
    map2["kiwi"] = 7
    
    // Retrieving elements
    appleCount := map3["apple"]
    fmt.Printf("Apple count: %d\n", appleCount)
    
    // Checking if key exists
    if count, exists := map3["banana"]; exists {
        fmt.Printf("Banana count: %d\n", count)
    }
    
    // Key that doesn't exist returns zero value
    mangoCount := map3["mango"]
    fmt.Printf("Mango count (doesn't exist): %d\n", mangoCount)
    
    // Deleting elements
    delete(map3, "orange")
    fmt.Printf("After deleting orange: %v\n", map3)
    
    // Map iteration
    fmt.Println("Iterating over map3:")
    for key, value := range map3 {
        fmt.Printf("%s: %d\n", key, value)
    }
}

func demonstrateMapOperations() {
    // Map of different types
    stringMap := map[string]string{
        "en": "English",
        "fr": "French",
        "es": "Spanish",
    }
    
    intMap := map[int]bool{
        1: true,
        2: false,
        3: true,
    }
    
    // Nested maps
    nestedMap := map[string]map[string]int{
        "fruits": {
            "apple":  5,
            "banana": 3,
        },
        "vegetables": {
            "carrot": 10,
            "broccoli": 4,
        },
    }
    
    fmt.Printf("String map: %v\n", stringMap)
    fmt.Printf("Int map: %v\n", intMap)
    fmt.Printf("Nested map: %v\n", nestedMap)
    
    // Map equality (maps cannot be compared with ==)
    // Must iterate and compare manually
    mapA := map[string]int{"a": 1, "b": 2}
    mapB := map[string]int{"a": 1, "b": 2}
    
    equal := mapsEqual(mapA, mapB)
    fmt.Printf("Maps equal: %t\n", equal)
    
    // Map as function parameter (passed by reference)
    modifyMap(mapA)
    fmt.Printf("After function call: %v\n", mapA)
}

func mapsEqual(a, b map[string]int) bool {
    if len(a) != len(b) {
        return false
    }
    for key, value := range a {
        if b[key] != value {
            return false
        }
    }
    return true
}

func modifyMap(m map[string]int) {
    m["new"] = 42  // Modifies original map
    fmt.Printf("Inside function: %v\n", m)
}
```

## Structs

Structs are composite types that group together zero or more named fields.

```go
// Struct definition
type Person struct {
    FirstName string
    LastName  string
    Age       int
    Email     string
}

type Address struct {
    Street  string
    City    string
    Country string
    ZipCode string
}

type Employee struct {
    Person        // Embedded struct
    EmployeeID    int
    Department    string
    Salary        float64
    WorkAddress   Address  // Nested struct
    Skills        []string // Slice field
}

func demonstrateStructs() {
    // Struct instantiation
    var person1 Person  // Zero-valued struct
    person2 := Person{
        FirstName: "John",
        LastName:  "Doe",
        Age:       30,
        Email:     "john.doe@example.com",
    }
    
    // Short struct literal (when all fields are specified in order)
    person3 := Person{"Jane", "Smith", 25, "jane.smith@example.com"}
    
    fmt.Println("Person examples:")
    fmt.Printf("person1 (zero value): %+v\n", person1)
    fmt.Printf("person2: %+v\n", person2)
    fmt.Printf("person3: %+v\n", person3)
    
    // Field access
    fmt.Printf("First name: %s\n", person2.FirstName)
    fmt.Printf("Age: %d\n", person2.Age)
    
    // Field modification
    person2.Age = 31
    person2.Email = "john.doe.updated@example.com"
    fmt.Printf("Updated person2: %+v\n", person2)
    
    // Pointer to struct
    personPtr := &person3
    personPtr.Age = 26  // Automatically dereferenced
    fmt.Printf("Modified through pointer: %+v\n", person3)
}

func demonstrateEmbeddedStructs() {
    // Creating employee with embedded struct
    emp := Employee{
        Person: Person{
            FirstName: "Alice",
            LastName:  "Johnson",
            Age:       28,
            Email:     "alice.johnson@company.com",
        },
        EmployeeID:  1001,
        Department:  "Engineering",
        Salary:      75000.0,
        WorkAddress: Address{
            Street:  "123 Tech Street",
            City:    "San Francisco",
            Country: "USA",
            ZipCode: "94105",
        },
        Skills: []string{"Go", "Python", "Docker"},
    }
    
    fmt.Printf("Employee: %+v\n", emp)
    
    // Accessing embedded fields directly
    fmt.Printf("Name: %s %s\n", emp.FirstName, emp.LastName)  // Direct access
    fmt.Printf("Department: %s\n", emp.Department)
    fmt.Printf("Work City: %s\n", emp.WorkAddress.City)
    
    // Iterating over slice field
    fmt.Println("Skills:")
    for _, skill := range emp.Skills {
        fmt.Printf("- %s\n", skill)
    }
}

func demonstrateStructMethods() {
    // Methods can be defined on struct types
    type Rectangle struct {
        Width, Height float64
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
    
    rect := Rectangle{Width: 10, Height: 5}
    fmt.Printf("Original rectangle: %+v\n", rect)
    fmt.Printf("Area: %.2f\n", rect.Area())
    
    rect.Scale(2.0)
    fmt.Printf("After scaling by 2: %+v\n", rect)
    fmt.Printf("New area: %.2f\n", rect.Area())
}
```

## Type Aliases and Definitions

```go
// Type alias (same underlying type)
type Celsius float64
type Fahrenheit float64

// Type definition (creates new distinct type)
type Temperature float64

func demonstrateTypeAliases() {
    var c Celsius = 25.0
    var f Fahrenheit = 77.0
    var t Temperature = 20.0
    
    fmt.Printf("Celsius: %f°C\n", c)
    fmt.Printf("Fahrenheit: %f°F\n", f)
    fmt.Printf("Temperature: %f°T\n", t)
    
    // Type aliases can be used interchangeably with underlying type
    var temp float64 = float64(c)  // Explicit conversion required
    fmt.Printf("Converted to float64: %f\n", temp)
    
    // Type definitions require explicit conversion
    // var temp2 float64 = t  // This would cause compile error
    var temp2 float64 = float64(t)  // Explicit conversion required
    fmt.Printf("Temperature converted to float64: %f\n", temp2)
}
```

Composite types are fundamental to Go programming, providing the building blocks for complex data structures and applications. Understanding their behavior, performance characteristics, and appropriate use cases is essential for writing effective Go code.
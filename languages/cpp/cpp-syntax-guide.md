# C++ Syntax Guide

This guide covers the basic syntax and fundamental concepts of C++ programming language.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Variables and Data Types](#variables-and-data-types)
- [Control Flow](#control-flow)
- [Functions](#functions)
- [Classes and Objects](#classes-and-objects)
- [Templates](#templates)
- [Memory Management](#memory-management)

## Basic Structure

```cpp
#include <iostream>  // Preprocessor directive

using namespace std;  // Optional: Avoid typing std:: everywhere

// Main function - entry point of the program
int main() {
    cout << "Hello, World!" << endl;
    return 0;  // Return 0 to indicate successful execution
}
```

## Variables and Data Types

### Basic Data Types
```cpp
int age = 25;                // Integer
float salary = 50000.50f;    // Floating point (single precision)
double pi = 3.14159265359;   // Floating point (double precision)
char grade = 'A';            // Character
bool isActive = true;        // Boolean
```

### Type Modifiers
```cpp
unsigned int positiveNumber = 100;
long long largeNumber = 9223372036854775807LL;
const int MAX_SIZE = 100;    // Constant (cannot be modified)
```

### User-Defined Data Types
```cpp
enum Color { RED, GREEN, BLUE };
Color favoriteColor = GREEN;

struct Person {
    string name;
    int age;
};

Person person1 = {"John", 30};
```

## Control Flow

### Conditional Statements
```cpp
if (age >= 18) {
    cout << "Adult" << endl;
} else if (age >= 13) {
    cout << "Teenager" << endl;
} else {
    cout << "Child" << endl;
}

// Switch statement
switch (favoriteColor) {
    case RED:
        cout << "Red" << endl;
        break;
    case GREEN:
        cout << "Green" << endl;
        break;
    case BLUE:
        cout << "Blue" << endl;
        break;
    default:
        cout << "Unknown color" << endl;
}
```

### Loops
```cpp
// For loop
for (int i = 0; i < 5; i++) {
    cout << i << endl;
}

// While loop
int i = 0;
while (i < 5) {
    cout << i << endl;
    i++;
}

// Do-while loop
int j = 0;
do {
    cout << j << endl;
    j++;
} while (j < 5);
```

## Functions

### Basic Function
```cpp
int add(int a, int b) {
    return a + b;
}

// Function call
int sum = add(5, 3);
```

### Function Overloading
```cpp
int multiply(int a, int b) {
    return a * b;
}

double multiply(double a, double b) {
    return a * b;
}
```

### Default Parameters
```cpp
void greet(string name, string message = "Hello") {
    cout << message << ", " << name << "!" << endl;
}

// Usage
greet("Alice");  // Uses default message
greet("Bob", "Hi");  // Uses custom message
```

## Classes and Objects

### Class Definition
```cpp
class Car {
private:
    string brand;
    string model;
    int year;

public:
    // Constructor
    Car(string b, string m, int y) {
        brand = b;
        model = m;
        year = y;
    }

    // Method to display information
    void displayInfo() {
        cout << year << " " << brand << " " << model << endl;
    }

    // Getter
    string getBrand() {
        return brand;
    }

    // Setter
    void setYear(int y) {
        if (y > 1885) {  // First car invented in 1885
            year = y;
        }
    }
};

// Object creation and usage
Car myCar("Toyota", "Camry", 2020);
myCar.displayInfo();
myCar.setYear(2022);
cout << "Brand: " << myCar.getBrand() << endl;
```

## Templates

### Function Template
```cpp
template <typename T>
T maximum(T a, T b) {
    return (a > b) ? a : b;
}

// Usage
int maxInt = maximum(5, 10);
double maxDouble = maximum(3.14, 2.71);
```

### Class Template
```cpp
template <typename T>
class Pair {
private:
    T first;
    T second;

public:
    Pair(T a, T b) : first(a), second(b) {}

    T getMax() {
        return (first > second) ? first : second;
    }
};

// Usage
Pair<int> intPair(10, 20);
Pair<double> doublePair(3.14, 2.71);
```

## Memory Management

### Dynamic Memory Allocation
```cpp
// Allocate single variable
int* ptr = new int;
*ptr = 10;

// Allocate array
int* arr = new int[5];
for (int i = 0; i < 5; i++) {
    arr[i] = i * 2;
}

// Deallocate memory
delete ptr;
delete[] arr;  // Use [] for arrays
```

### Smart Pointers (C++11 and later)
```cpp
#include <memory>

// Unique pointer - exclusive ownership
unique_ptr<int> uptr(new int(5));

// Shared pointer - shared ownership
shared_ptr<int> sptr1(new int(10));
shared_ptr<int> sptr2 = sptr1;  // Reference count increases to 2

// Weak pointer - doesn't affect reference count
weak_ptr<int> wptr = sptr1;
```

## Input/Output

```cpp
#include <iostream>
#include <string>

using namespace std;

int main() {
    int age;
    string name;
    
    cout << "Enter your name: ";
    getline(cin, name);  // Read entire line including spaces
    
    cout << "Enter your age: ";
    cin >> age;
    
    cout << "Hello " << name << ", you are " << age << " years old." << endl;
    
    return 0;
}
```

## Useful Resources
- [C++ Standard Library Reference](https://en.cppreference.com/w/)
- [Learn C++](https://www.learncpp.com/)
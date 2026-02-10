# Goroutines

Goroutines are lightweight threads managed by the Go runtime that enable concurrent execution of functions. They are one of Go's key features for building concurrent and parallel applications efficiently.

## What are Goroutines?

A goroutine is a lightweight thread of execution that runs concurrently with other goroutines within the same address space. Unlike traditional OS threads, goroutines are:

- **Lightweight**: Initial stack size is only a few KB
- **Cheap to create**: Thousands or millions can run simultaneously
- **Managed by Go runtime**: Automatic scheduling and load balancing
- **Cooperative**: Yield control voluntarily during blocking operations

## Creating Goroutines

### Basic Syntax

Use the `go` keyword followed by a function call to create a goroutine:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create a goroutine
    go sayHello()
    
    // Main function continues immediately
    fmt.Println("Main function continues...")
    
    // Give goroutine time to execute
    time.Sleep(100 * time.Millisecond)
}

func sayHello() {
    fmt.Println("Hello from goroutine!")
}
```

### Anonymous Goroutines

You can also create goroutines with anonymous functions:

```go
func main() {
    // Anonymous goroutine
    go func() {
        fmt.Println("Anonymous goroutine executing")
    }()
    
    // With parameters
    name := "Alice"
    go func(n string) {
        fmt.Printf("Hello %s from anonymous goroutine\n", n)
    }(name)
    
    time.Sleep(100 * time.Millisecond)
}
```

## Goroutine Lifecycle

### Creation and Scheduling

```go
func demonstrateLifecycle() {
    fmt.Println("Creating goroutine...")
    
    go func() {
        fmt.Println("Goroutine started")
        time.Sleep(50 * time.Millisecond)
        fmt.Println("Goroutine finished")
    }()
    
    fmt.Println("Main continues immediately")
    time.Sleep(100 * time.Millisecond)
}
```

### Goroutine States

Goroutines go through several states during their lifecycle:

1. **Runnable**: Ready to execute
2. **Running**: Currently executing
3. **Waiting**: Blocked on I/O, channels, or synchronization
4. **Dead**: Completed execution

## Communication Between Goroutines

### Shared Memory Approach

```go
func sharedMemoryExample() {
    var counter int
    
    // Multiple goroutines accessing shared variable
    for i := 0; i < 5; i++ {
        go func(id int) {
            for j := 0; j < 1000; j++ {
                counter++ // Race condition!
            }
            fmt.Printf("Goroutine %d finished\n", id)
        }(i)
    }
    
    time.Sleep(time.Second)
    fmt.Printf("Final counter value: %d\n", counter) // Unpredictable result
}
```

### Proper Synchronization (Using Mutex)

```go
import "sync"

func synchronizedCounter() {
    var counter int
    var mutex sync.Mutex
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                mutex.Lock()
                counter++
                mutex.Unlock()
            }
            fmt.Printf("Goroutine %d finished\n", id)
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("Final counter value: %d\n", counter) // Always 5000
}
```

## Goroutine Patterns

### Producer-Consumer Pattern

```go
func producerConsumer() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // Producer
    go func() {
        for i := 1; i <= 10; i++ {
            jobs <- i
        }
        close(jobs)
    }()
    
    // Consumer/Worker
    go func() {
        for job := range jobs {
            results <- job * job
        }
        close(results)
    }()
    
    // Collect results
    for result := range results {
        fmt.Printf("Result: %d\n", result)
    }
}
```

### Worker Pool Pattern

```go
func workerPool() {
    const numWorkers = 3
    const numJobs = 10
    
    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)
    
    // Start workers
    for w := 1; w <= numWorkers; w++ {
        go worker(w, jobs, results)
    }
    
    // Send jobs
    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs)
    
    // Collect results
    for a := 1; a <= numJobs; a++ {
        <-results
    }
}

func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, j)
        time.Sleep(time.Second)
        results <- j * 2
    }
}
```

### Fan-out/Fan-in Pattern

```go
func fanOutFanIn() {
    // Fan-out: distribute work to multiple workers
    work := make(chan int)
    results := make(chan int)
    
    // Start multiple workers
    for i := 0; i < 3; i++ {
        go worker(i, work, results)
    }
    
    // Send work
    go func() {
        for i := 1; i <= 10; i++ {
            work <- i
        }
        close(work)
    }()
    
    // Fan-in: collect all results
    go func() {
        for i := 0; i < 10; i++ {
            result := <-results
            fmt.Printf("Received result: %d\n", result)
        }
    }()
}
```

## Goroutine Best Practices

### 1. Always Handle Goroutine Lifecycle

```go
// Good: Use WaitGroup to wait for completion
func properGoroutineHandling() {
    var wg sync.WaitGroup
    
    wg.Add(1)
    go func() {
        defer wg.Done()
        // Do work
        fmt.Println("Working...")
    }()
    
    wg.Wait() // Wait for goroutine to finish
}
```

### 2. Avoid Goroutine Leaks

```go
// Bad: Goroutine leak
func leakingGoroutine() {
    ch := make(chan int)
    go func() {
        // This goroutine will block forever
        val := <-ch
        fmt.Println(val)
    }()
    // Channel never gets closed
}

// Good: Proper cleanup
func properCleanup() {
    ch := make(chan int)
    go func() {
        val, ok := <-ch
        if ok {
            fmt.Println(val)
        }
    }()
    
    ch <- 42
    close(ch) // Close channel to unblock goroutine
}
```

### 3. Use Context for Cancellation

```go
import "context"

func contextCancellation() {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    go func() {
        select {
        case <-time.After(3 * time.Second):
            fmt.Println("Work completed")
        case <-ctx.Done():
            fmt.Println("Work cancelled:", ctx.Err())
        }
    }()
    
    time.Sleep(3 * time.Second)
}
```

## Performance Considerations

### Goroutine Overhead

```go
func measureGoroutineOverhead() {
    const numGoroutines = 1000000
    
    start := time.Now()
    
    var wg sync.WaitGroup
    wg.Add(numGoroutines)
    
    for i := 0; i < numGoroutines; i++ {
        go func() {
            defer wg.Done()
            // Minimal work
        }()
    }
    
    wg.Wait()
    duration := time.Since(start)
    
    fmt.Printf("Created %d goroutines in %v\n", numGoroutines, duration)
    fmt.Printf("Average creation time: %v per goroutine\n", 
               duration/time.Duration(numGoroutines))
}
```

### GOMAXPROCS Setting

```go
import "runtime"

func configureMaxProcs() {
    // Set maximum number of OS threads
    runtime.GOMAXPROCS(4)
    
    // Or let Go automatically determine optimal value
    optimal := runtime.GOMAXPROCS(0)
    fmt.Printf("Optimal GOMAXPROCS: %d\n", optimal)
}
```

## Debugging Goroutines

### Using runtime package

```go
func debugGoroutines() {
    // Print number of goroutines
    fmt.Printf("Number of goroutines: %d\n", runtime.NumGoroutine())
    
    // Print stack traces of all goroutines
    buf := make([]byte, 1<<20)
    runtime.Stack(buf, true)
    fmt.Printf("Stack trace:\n%s\n", buf)
}
```

### Profiling Goroutines

```bash
# Command line profiling
go tool pprof http://localhost:6060/debug/pprof/goroutine

# In code
import _ "net/http/pprof"

func startProfilingServer() {
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
}
```

## Common Pitfalls

### 1. Race Conditions

```go
// Race condition example
func raceCondition() {
    var data int
    
    go func() {
        data = 42 // Write
    }()
    
    fmt.Println(data) // Read - might be 0 or 42
}
```

### 2. Closing Channels Prematurely

```go
// Wrong way
func wrongChannelClosing() {
    ch := make(chan int)
    
    go func() {
        for i := 0; i < 5; i++ {
            ch <- i
        }
    }()
    
    close(ch) // Closed before goroutine finishes!
    
    for val := range ch {
        fmt.Println(val)
    }
}
```

### 3. Ignoring Return Values

```go
// Bad: Ignoring errors in goroutines
func ignoreErrors() {
    go func() {
        _, err := someOperation()
        if err != nil {
            // Error is lost!
            return
        }
    }()
}
```

## Advanced Topics

### Goroutine Local Storage

```go
import "context"

func goroutineLocalStorage() {
    type keyType string
    const userIDKey keyType = "userID"
    
    ctx := context.WithValue(context.Background(), userIDKey, "user123")
    
    go func(ctx context.Context) {
        userID := ctx.Value(userIDKey).(string)
        fmt.Printf("Processing for user: %s\n", userID)
    }(ctx)
}
```

### Goroutine Scheduling Internals

The Go scheduler uses a M:N threading model:
- **M**: OS threads
- **P**: Processors (logical CPUs)
- **G**: Goroutines

The scheduler automatically distributes goroutines across available processors for optimal performance.

## Summary

Goroutines provide an elegant way to achieve concurrency in Go:

✅ **Advantages:**
- Extremely lightweight compared to OS threads
- Simple syntax with the `go` keyword
- Automatic scheduling and load balancing
- Excellent integration with channels for communication

⚠️ **Considerations:**
- Need proper synchronization to avoid race conditions
- Context cancellation for proper cleanup
- Monitor goroutine leaks in long-running applications
- Understand channel semantics for communication

Goroutines form the foundation of Go's concurrency model and work seamlessly with channels to create robust concurrent applications.
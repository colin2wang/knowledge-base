# Sync Package

The `sync` package provides essential synchronization primitives for coordinating goroutines and protecting shared resources in concurrent Go programs. It includes mutexes, wait groups, condition variables, and other tools for safe concurrent programming.

## Overview of Sync Package

The `sync` package offers fundamental building blocks for concurrent programming:

- **Mutex**: Mutual exclusion locks for protecting shared data
- **RWMutex**: Reader-writer mutex for optimized read-heavy scenarios
- **WaitGroup**: Synchronization for waiting on multiple goroutines
- **Cond**: Condition variables for complex signaling
- **Once**: Ensuring one-time initialization
- **Pool**: Object pooling for reducing allocation overhead

## Mutex

### Basic Mutex Usage

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var counter int
    var mutex sync.Mutex
    var wg sync.WaitGroup
    
    // Launch multiple goroutines
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
    fmt.Printf("Final counter: %d\n", counter)
}
```

### Mutex Methods

```go
func mutexMethods() {
    var mu sync.Mutex
    
    // Lock acquires the mutex
    mu.Lock()
    
    // Critical section - only one goroutine can be here
    fmt.Println("Critical section")
    
    // Unlock releases the mutex
    mu.Unlock()
    
    // TryLock (Go 1.18+) - non-blocking attempt to acquire lock
    if mu.TryLock() {
        fmt.Println("Lock acquired")
        mu.Unlock()
    } else {
        fmt.Println("Could not acquire lock")
    }
}
```

### Mutex Best Practices

```go
func mutexBestPractices() {
    type Counter struct {
        mu    sync.Mutex
        value int
    }
    
    c := &Counter{}
    
    // Good: Lock only critical sections
    c.mu.Lock()
    c.value++
    c.mu.Unlock()
    
    // Good: Use defer for unlock
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value += 10
    
    // Bad: Holding lock too long
    c.mu.Lock()
    time.Sleep(time.Second) // Don't do this in critical section
    c.value++
    c.mu.Unlock()
}
```

## RWMutex (Reader-Writer Mutex)

### Read-Write Separation

```go
func rwMutexExample() {
    var rwMu sync.RWMutex
    var data map[string]int = make(map[string]int)
    
    // Writer goroutine
    go func() {
        for i := 0; i < 5; i++ {
            rwMu.Lock()
            key := fmt.Sprintf("key%d", i)
            data[key] = i * 10
            fmt.Printf("Wrote %s: %d\n", key, data[key])
            rwMu.Unlock()
            time.Sleep(100 * time.Millisecond)
        }
    }()
    
    // Reader goroutines
    for i := 0; i < 3; i++ {
        go func(readerID int) {
            for j := 0; j < 10; j++ {
                rwMu.RLock()
                // Multiple readers can access simultaneously
                for k, v := range data {
                    fmt.Printf("Reader %d sees %s: %d\n", readerID, k, v)
                }
                rwMu.RUnlock()
                time.Sleep(50 * time.Millisecond)
            }
        }(i)
    }
    
    time.Sleep(2 * time.Second)
}
```

### RWMutex Methods

```go
func rwMutexMethods() {
    var rwMu sync.RWMutex
    
    // RLock - acquire read lock
    rwMu.RLock()
    // Multiple goroutines can hold read locks
    fmt.Println("Reading data...")
    rwMu.RUnlock()
    
    // Lock - acquire write lock (exclusive)
    rwMu.Lock()
    // Only one goroutine can hold write lock
    fmt.Println("Writing data...")
    rwMu.Unlock()
    
    // RLocker - get Locker interface for read operations
    readLocker := rwMu.RLocker()
    readLocker.Lock()   // Same as RLock()
    readLocker.Unlock() // Same as RUnlock()
}
```

## WaitGroup

### Basic WaitGroup Usage

```go
func waitGroupBasic() {
    var wg sync.WaitGroup
    
    // Add the number of goroutines to wait for
    wg.Add(3)
    
    go func() {
        defer wg.Done() // Decrement counter when done
        fmt.Println("Goroutine 1 working...")
        time.Sleep(100 * time.Millisecond)
        fmt.Println("Goroutine 1 finished")
    }()
    
    go func() {
        defer wg.Done()
        fmt.Println("Goroutine 2 working...")
        time.Sleep(200 * time.Millisecond)
        fmt.Println("Goroutine 2 finished")
    }()
    
    go func() {
        defer wg.Done()
        fmt.Println("Goroutine 3 working...")
        time.Sleep(150 * time.Millisecond)
        fmt.Println("Goroutine 3 finished")
    }()
    
    // Wait blocks until counter reaches zero
    fmt.Println("Waiting for goroutines...")
    wg.Wait()
    fmt.Println("All goroutines completed")
}
```

### WaitGroup Patterns

```go
func waitGroupPatterns() {
    // Pattern 1: Worker pool with WaitGroup
    func workerPool() {
        var wg sync.WaitGroup
        jobs := make(chan int, 10)
        
        // Start workers
        for i := 0; i < 3; i++ {
            wg.Add(1)
            go func(workerID int) {
                defer wg.Done()
                for job := range jobs {
                    fmt.Printf("Worker %d processing job %d\n", workerID, job)
                    time.Sleep(50 * time.Millisecond)
                }
            }(i)
        }
        
        // Send jobs
        for i := 1; i <= 10; i++ {
            jobs <- i
        }
        close(jobs)
        
        wg.Wait() // Wait for all workers
        fmt.Println("All jobs completed")
    }
    
    // Pattern 2: Error collection
    func errorHandling() {
        var wg sync.WaitGroup
        var mu sync.Mutex
        var errors []error
        
        tasks := []func() error{
            func() error { return nil },
            func() error { return fmt.Errorf("task 2 failed") },
            func() error { return nil },
        }
        
        for i, task := range tasks {
            wg.Add(1)
            go func(id int, t func() error) {
                defer wg.Done()
                if err := t(); err != nil {
                    mu.Lock()
                    errors = append(errors, fmt.Errorf("task %d: %w", id, err))
                    mu.Unlock()
                }
            }(i, task)
        }
        
        wg.Wait()
        
        if len(errors) > 0 {
            fmt.Printf("Encountered %d errors:\n", len(errors))
            for _, err := range errors {
                fmt.Printf("  %v\n", err)
            }
        } else {
            fmt.Println("All tasks completed successfully")
        }
    }
    
    workerPool()
    fmt.Println("---")
    errorHandling()
}
```

## Cond (Condition Variables)

### Basic Condition Variable Usage

```go
func condBasic() {
    var mu sync.Mutex
    cond := sync.NewCond(&mu)
    ready := false
    
    // Waiter goroutine
    go func() {
        mu.Lock()
        for !ready {
            fmt.Println("Waiting for signal...")
            cond.Wait() // Releases lock and waits for signal
        }
        fmt.Println("Received signal, continuing...")
        mu.Unlock()
    }()
    
    // Signaler goroutine
    time.Sleep(500 * time.Millisecond)
    mu.Lock()
    ready = true
    fmt.Println("Sending signal...")
    cond.Signal() // Wake up one waiting goroutine
    mu.Unlock()
    
    time.Sleep(100 * time.Millisecond)
}
```

### Broadcast vs Signal

```go
func condBroadcast() {
    var mu sync.Mutex
    cond := sync.NewCond(&mu)
    var ready bool
    
    // Multiple waiter goroutines
    for i := 0; i < 3; i++ {
        go func(id int) {
            mu.Lock()
            for !ready {
                fmt.Printf("Goroutine %d waiting...\n", id)
                cond.Wait()
            }
            fmt.Printf("Goroutine %d proceeding...\n", id)
            mu.Unlock()
        }(i)
    }
    
    // Let waiters get ready
    time.Sleep(100 * time.Millisecond)
    
    // Broadcast wakes all waiting goroutines
    mu.Lock()
    ready = true
    fmt.Println("Broadcasting signal...")
    cond.Broadcast()
    mu.Unlock()
    
    time.Sleep(100 * time.Millisecond)
}
```

## Once

### One-Time Initialization

```go
func onceExample() {
    var once sync.Once
    var initialized bool
    
    // Function to initialize something expensive
    initFunc := func() {
        fmt.Println("Initializing...")
        time.Sleep(100 * time.Millisecond) // Simulate expensive operation
        initialized = true
        fmt.Println("Initialization complete")
    }
    
    // Multiple goroutines trying to initialize
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            fmt.Printf("Goroutine %d attempting initialization\n", id)
            once.Do(initFunc)
            fmt.Printf("Goroutine %d sees initialized=%v\n", id, initialized)
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("Final state: initialized=%v\n", initialized)
}
```

### Once with Closures

```go
func onceWithClosures() {
    var once sync.Once
    var config *Config
    
    getConfig := func() *Config {
        once.Do(func() {
            fmt.Println("Loading configuration...")
            config = &Config{
                Host: "localhost",
                Port: 8080,
            }
            fmt.Println("Configuration loaded")
        })
        return config
    }
    
    // Multiple concurrent requests for config
    var wg sync.WaitGroup
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            cfg := getConfig()
            fmt.Printf("Goroutine %d got config: %+v\n", id, cfg)
        }(i)
    }
    
    wg.Wait()
}

type Config struct {
    Host string
    Port int
}
```

## Pool

### Object Pooling

```go
func poolExample() {
    // Create pool of byte slices
    pool := sync.Pool{
        New: func() interface{} {
            fmt.Println("Creating new buffer")
            return make([]byte, 1024)
        },
    }
    
    // Get object from pool
    buf1 := pool.Get().([]byte)
    fmt.Printf("Got buffer of length %d\n", len(buf1))
    
    // Use the buffer
    copy(buf1, []byte("Hello, World!"))
    fmt.Printf("Buffer content: %s\n", string(buf1[:13]))
    
    // Put object back in pool
    pool.Put(buf1)
    
    // Get another object (might reuse the previous one)
    buf2 := pool.Get().([]byte)
    fmt.Printf("Got buffer of length %d (same object: %v)\n", 
               len(buf2), &buf1[0] == &buf2[0])
    
    // Put it back
    pool.Put(buf2)
}
```

### Pool Best Practices

```go
func poolBestPractices() {
    type Worker struct {
        id   int
        data []byte
    }
    
    pool := sync.Pool{
        New: func() interface{} {
            return &Worker{
                data: make([]byte, 100),
            }
        },
    }
    
    // Worker function that uses pooled objects
    processWork := func(workID int) {
        // Get worker from pool
        worker := pool.Get().(*Worker)
        worker.id = workID
        
        // Do work with the worker
        fmt.Printf("Processing work %d with worker %d\n", workID, worker.id)
        time.Sleep(10 * time.Millisecond)
        
        // Reset worker state before returning to pool
        worker.id = 0
        // Clear sensitive data if needed
        // worker.data = nil
        
        // Return to pool
        pool.Put(worker)
    }
    
    // Process multiple work items
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            processWork(id)
        }(i)
    }
    
    wg.Wait()
}
```

## Advanced Synchronization Patterns

### Double-Checked Locking

```go
func doubleCheckedLocking() {
    var instance *Singleton
    var once sync.Once
    
    getInstance := func() *Singleton {
        if instance == nil {
            once.Do(func() {
                instance = &Singleton{name: "MySingleton"}
            })
        }
        return instance
    }
    
    // Test concurrent access
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            inst := getInstance()
            fmt.Printf("Goroutine %d got instance: %p\n", id, inst)
        }(i)
    }
    
    wg.Wait()
}

type Singleton struct {
    name string
}
```

### Semaphore Pattern

```go
func semaphorePattern() {
    type Semaphore struct {
        ch chan struct{}
    }
    
    newSemaphore := func(maxConcurrency int) *Semaphore {
        return &Semaphore{
            ch: make(chan struct{}, maxConcurrency),
        }
    }
    
    (s *Semaphore) Acquire() {
        s.ch <- struct{}{}
    }
    
    (s *Semaphore) Release() {
        <-s.ch
    }
    
    // Usage
    sem := newSemaphore(3) // Allow max 3 concurrent operations
    
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            sem.Acquire()
            defer sem.Release()
            
            fmt.Printf("Task %d running\n", id)
            time.Sleep(200 * time.Millisecond)
            fmt.Printf("Task %d completed\n", id)
        }(i)
    }
    
    wg.Wait()
}
```

### Barrier Pattern

```go
func barrierPattern() {
    type Barrier struct {
        count     int
        threshold int
        cond      *sync.Cond
        mu        sync.Mutex
    }
    
    newBarrier := func(threshold int) *Barrier {
        return &Barrier{
            threshold: threshold,
            cond:      sync.NewCond(&sync.Mutex{}),
        }
    }
    
    (b *Barrier) Wait() {
        b.cond.L.Lock()
        defer b.cond.L.Unlock()
        
        b.count++
        if b.count < b.threshold {
            b.cond.Wait()
        } else {
            b.cond.Broadcast()
            b.count = 0 // Reset for next use
        }
    }
    
    // Usage: All goroutines wait at barrier
    barrier := newBarrier(3)
    
    var wg sync.WaitGroup
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            fmt.Printf("Goroutine %d approaching barrier\n", id)
            time.Sleep(time.Duration(id*100) * time.Millisecond)
            
            fmt.Printf("Goroutine %d waiting at barrier\n", id)
            barrier.Wait()
            
            fmt.Printf("Goroutine %d passed barrier\n", id)
        }(i)
    }
    
    wg.Wait()
}
```

## Map (Concurrent Map)

### Sync.Map Usage

```go
func syncMapExample() {
    var m sync.Map
    
    // Store values
    m.Store("key1", "value1")
    m.Store("key2", 42)
    m.Store(123, "numeric_key")
    
    // Load values
    if val, ok := m.Load("key1"); ok {
        fmt.Printf("key1: %v\n", val)
    }
    
    // Load or store
    if val, loaded := m.LoadOrStore("key3", "default_value"); loaded {
        fmt.Printf("Existing value: %v\n", val)
    } else {
        fmt.Printf("Stored new value: %v\n", val)
    }
    
    // Delete
    m.Delete("key2")
    
    // Range over all entries
    m.Range(func(key, value interface{}) bool {
        fmt.Printf("Key: %v, Value: %v\n", key, value)
        return true // Continue iteration
    })
}
```

### Sync.Map vs Regular Map with Mutex

```go
func compareMaps() {
    // Regular map with mutex
    type SafeMap struct {
        mu   sync.RWMutex
        data map[string]int
    }
    
    safeMap := &SafeMap{
        data: make(map[string]int),
    }
    
    // Sync.Map
    var syncMap sync.Map
    
    // Benchmark writes
    const iterations = 100000
    
    start := time.Now()
    for i := 0; i < iterations; i++ {
        safeMap.mu.Lock()
        safeMap.data[fmt.Sprintf("key%d", i)] = i
        safeMap.mu.Unlock()
    }
    safeMapTime := time.Since(start)
    
    start = time.Now()
    for i := 0; i < iterations; i++ {
        syncMap.Store(fmt.Sprintf("key%d", i), i)
    }
    syncMapTime := time.Since(start)
    
    fmt.Printf("SafeMap write time: %v\n", safeMapTime)
    fmt.Printf("Sync.Map write time: %v\n", syncMapTime)
    fmt.Printf("Sync.Map is %.2fx faster for writes\n", 
               float64(safeMapTime)/float64(syncMapTime))
}
```

## Performance Considerations

### Mutex vs RWMutex Benchmark

```go
func mutexVsRwMutex() {
    const readers = 5
    const writers = 2
    const operations = 100000
    
    // Test with regular mutex
    var mu sync.Mutex
    var data int
    
    start := time.Now()
    var wg sync.WaitGroup
    
    // Writers
    for i := 0; i < writers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < operations/writers; j++ {
                mu.Lock()
                data++
                mu.Unlock()
            }
        }()
    }
    
    // Readers
    for i := 0; i < readers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < operations/readers; j++ {
                mu.Lock()
                _ = data
                mu.Unlock()
            }
        }()
    }
    
    wg.Wait()
    mutexTime := time.Since(start)
    
    // Test with RWMutex
    var rwMu sync.RWMutex
    data = 0
    
    start = time.Now()
    wg = sync.WaitGroup{}
    
    // Writers
    for i := 0; i < writers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < operations/writers; j++ {
                rwMu.Lock()
                data++
                rwMu.Unlock()
            }
        }()
    }
    
    // Readers
    for i := 0; i < readers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < operations/readers; j++ {
                rwMu.RLock()
                _ = data
                rwMu.RUnlock()
            }
        }()
    }
    
    wg.Wait()
    rwMutexTime := time.Since(start)
    
    fmt.Printf("Mutex time: %v\n", mutexTime)
    fmt.Printf("RWMutex time: %v\n", rwMutexTime)
    fmt.Printf("RWMutex improvement: %.2fx faster\n", 
               float64(mutexTime)/float64(rwMutexTime))
}
```

### Pool Performance Benefits

```go
func poolPerformance() {
    const iterations = 1000000
    
    // Without pool
    start := time.Now()
    for i := 0; i < iterations; i++ {
        buf := make([]byte, 1024)
        // Use buffer...
        _ = len(buf)
    }
    withoutPoolTime := time.Since(start)
    
    // With pool
    pool := sync.Pool{
        New: func() interface{} {
            return make([]byte, 1024)
        },
    }
    
    start = time.Now()
    for i := 0; i < iterations; i++ {
        buf := pool.Get().([]byte)
        // Use buffer...
        _ = len(buf)
        pool.Put(buf)
    }
    withPoolTime := time.Since(start)
    
    fmt.Printf("Without pool: %v\n", withoutPoolTime)
    fmt.Printf("With pool: %v\n", withPoolTime)
    fmt.Printf("Pool improvement: %.2fx faster\n", 
               float64(withoutPoolTime)/float64(withPoolTime))
}
```

## Debugging Synchronization Issues

### Detecting Deadlocks

```go
func detectDeadlocks() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("Recovered from panic: %v\n", r)
        }
    }()
    
    var mu1, mu2 sync.Mutex
    
    // Potential deadlock scenario
    go func() {
        mu1.Lock()
        time.Sleep(100 * time.Millisecond)
        mu2.Lock() // May deadlock if other goroutine holds mu2
        mu2.Unlock()
        mu1.Unlock()
    }()
    
    mu2.Lock()
    time.Sleep(100 * time.Millisecond)
    mu1.Lock() // May deadlock
    mu1.Unlock()
    mu2.Unlock()
    
    fmt.Println("No deadlock occurred")
}
```

### Race Detection

```go
func raceDetection() {
    var data int
    var mu sync.Mutex
    
    // Run with: go run -race sync_debug.go
    for i := 0; i < 2; i++ {
        go func() {
            mu.Lock()
            data++ // Protected access
            mu.Unlock()
            
            // Uncomment below to trigger race detector
            // data++ // Unprotected access - race condition!
        }()
    }
    
    time.Sleep(100 * time.Millisecond)
    fmt.Printf("Final data: %d\n", data)
}
```

## Common Pitfalls

### 1. Nested Locks

```go
// Bad: Nested locks can cause deadlocks
func nestedLocksBad() {
    var mu1, mu2 sync.Mutex
    
    mu1.Lock()
    mu2.Lock() // Risk of deadlock if another goroutine locks in reverse order
    // ... work ...
    mu2.Unlock()
    mu1.Unlock()
}

// Good: Consistent lock ordering
func nestedLocksGood() {
    var mu1, mu2 sync.Mutex
    
    // Always acquire locks in consistent order
    mu1.Lock()
    mu2.Lock()
    // ... work ...
    mu2.Unlock()
    mu1.Unlock()
}
```

### 2. Returning Locked Mutex

```go
// Bad: Returning locked mutex
func badLockedReturn() *sync.Mutex {
    var mu sync.Mutex
    mu.Lock()
    return &mu // Caller might never unlock!
}

// Good: Return unlocked resources
func goodUnlockedReturn() *sync.Mutex {
    var mu sync.Mutex
    // Work with mu...
    return &mu // Safe to use
}
```

### 3. Copying Sync Types

```go
// Bad: Copying mutex (creates invalid state)
func copyMutex() {
    var original sync.Mutex
    original.Lock()
    
    copied := original // Don't copy sync primitives!
    copied.Unlock()    // Undefined behavior
}

// Good: Use pointers or don't copy
func noCopyMutex() {
    var mu sync.Mutex
    mu.Lock()
    defer mu.Unlock()
    
    // Pass pointer instead of copying
    useMutex(&mu)
}

func useMutex(mu *sync.Mutex) {
    mu.Lock()
    defer mu.Unlock()
    // ... work ...
}
```

## Summary

The `sync` package provides essential tools for concurrent programming:

✅ **Key Components:**
- **Mutex/RWMutex**: Protect shared data with mutual exclusion
- **WaitGroup**: Coordinate goroutine completion
- **Cond**: Complex signaling between goroutines
- **Once**: Ensure one-time initialization
- **Pool**: Reduce allocation overhead with object pooling
- **Map**: Thread-safe concurrent map implementation

⚠️ **Best Practices:**
- Keep critical sections small and fast
- Always unlock mutexes (preferably with defer)
- Use RWMutex for read-heavy workloads
- Handle errors properly in concurrent contexts
- Test with race detector (`go run -race`)

The sync package forms the foundation for safe concurrent programming in Go, working seamlessly with goroutines and channels to build robust concurrent applications.
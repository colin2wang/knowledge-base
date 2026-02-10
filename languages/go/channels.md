# Channels

Channels are Go's built-in communication mechanism that enables safe data exchange between goroutines. They provide a type-safe way to synchronize and communicate between concurrent operations, following the CSP (Communicating Sequential Processes) model.

## What are Channels?

Channels are typed conduits through which you can send and receive values with the channel operator `<-`. They provide:

- **Type Safety**: Compile-time type checking
- **Synchronization**: Blocking operations for coordination
- **Communication**: Safe data transfer between goroutines
- **Backpressure**: Natural flow control mechanism

## Channel Basics

### Creating Channels

```go
package main

import "fmt"

func main() {
    // Create unbuffered channel
    ch := make(chan int)
    
    // Create buffered channel
    bufferedCh := make(chan string, 5)
    
    // Create channel with zero value (nil channel)
    var nilCh chan bool
    
    fmt.Printf("Unbuffered channel: %T\n", ch)
    fmt.Printf("Buffered channel: %T\n", bufferedCh)
    fmt.Printf("Nil channel: %T\n", nilCh)
}
```

### Sending and Receiving

```go
func basicChannelOperations() {
    ch := make(chan string)
    
    // Sender goroutine
    go func() {
        ch <- "Hello, World!" // Send operation
        fmt.Println("Message sent")
    }()
    
    // Receiver
    message := <-ch // Receive operation
    fmt.Printf("Received: %s\n", message)
}
```

## Channel Types

### Unbuffered Channels

Unbuffered channels block both sender and receiver until both are ready:

```go
func unbufferedChannelDemo() {
    ch := make(chan int)
    
    go func() {
        fmt.Println("Goroutine: About to send")
        ch <- 42 // Blocks until receiver is ready
        fmt.Println("Goroutine: Message sent")
    }()
    
    fmt.Println("Main: Waiting for message...")
    value := <-ch // Blocks until sender is ready
    fmt.Printf("Main: Received %d\n", value)
}
```

### Buffered Channels

Buffered channels can hold a specified number of values without blocking:

```go
func bufferedChannelDemo() {
    ch := make(chan int, 2) // Buffer size of 2
    
    // These won't block due to buffer space
    ch <- 1
    ch <- 2
    fmt.Println("Sent two values")
    
    // This would block (buffer full)
    // ch <- 3
    
    // Receive values
    fmt.Printf("Received: %d\n", <-ch)
    fmt.Printf("Received: %d\n", <-ch)
    
    // Now we can send again
    ch <- 3
    fmt.Printf("Received: %d\n", <-ch)
}
```

### Channel Direction

Channels can be restricted to send-only or receive-only:

```go
func directionalChannels() {
    // Send-only channel
    func sendData(sendCh chan<- int) {
        sendCh <- 100
        // sendCh <- 200 // This would cause compile error
        // <-sendCh      // This would cause compile error
    }
    
    // Receive-only channel
    func receiveData(recvCh <-chan int) {
        value := <-recvCh
        fmt.Printf("Received: %d\n", value)
        // recvCh <- 50  // This would cause compile error
    }
    
    ch := make(chan int)
    
    go sendData(ch)
    receiveData(ch)
}
```

## Channel Operations

### Basic Send/Receive

```go
func channelOperations() {
    ch := make(chan int, 3)
    
    // Send operations
    ch <- 1
    ch <- 2
    ch <- 3
    
    // Receive operations
    fmt.Printf("First: %d\n", <-ch)
    fmt.Printf("Second: %d\n", <-ch)
    fmt.Printf("Third: %d\n", <-ch)
}
```

### Range Over Channels

```go
func rangeOverChannels() {
    ch := make(chan int, 5)
    
    // Send multiple values
    go func() {
        for i := 1; i <= 5; i++ {
            ch <- i * 10
        }
        close(ch) // Must close to terminate range loop
    }()
    
    // Receive all values
    for value := range ch {
        fmt.Printf("Received: %d\n", value)
    }
}
```

### Select with Default Case

```go
func selectWithDefault() {
    ch := make(chan int, 1)
    
    select {
    case value := <-ch:
        fmt.Printf("Received: %d\n", value)
    default:
        fmt.Println("No value available")
    }
    
    // Send a value
    ch <- 42
    
    select {
    case value := <-ch:
        fmt.Printf("Now received: %d\n", value)
    default:
        fmt.Println("This won't print")
    }
}
```

## Channel Patterns

### Pipeline Pattern

```go
func pipelinePattern() {
    // Stage 1: Generator
    generator := func(done <-chan struct{}) <-chan int {
        out := make(chan int)
        go func() {
            defer close(out)
            for i := 1; i <= 5; i++ {
                select {
                case out <- i:
                case <-done:
                    return
                }
            }
        }()
        return out
    }
    
    // Stage 2: Processor
    processor := func(done <-chan struct{}, in <-chan int) <-chan int {
        out := make(chan int)
        go func() {
            defer close(out)
            for num := range in {
                select {
                case out <- num * num:
                case <-done:
                    return
                }
            }
        }()
        return out
    }
    
    // Stage 3: Consumer
    consumer := func(done <-chan struct{}, in <-chan int) {
        for value := range in {
            fmt.Printf("Processed: %d\n", value)
        }
    }
    
    done := make(chan struct{})
    defer close(done)
    
    // Create pipeline
    numbers := generator(done)
    squares := processor(done, numbers)
    consumer(done, squares)
}
```

### Fan-in Pattern

```go
func fanInPattern() {
    // Multiple input channels -> Single output channel
    fanIn := func(channels ...<-chan int) <-chan int {
        out := make(chan int)
        
        multiplex := func(ch <-chan int) {
            for val := range ch {
                out <- val
            }
        }
        
        for _, ch := range channels {
            go multiplex(ch)
        }
        
        go func() {
            // Wait for all input channels to close
            // This is simplified - in practice you'd use sync.WaitGroup
            close(out)
        }()
        
        return out
    }
    
    // Create input channels
    ch1 := make(chan int, 2)
    ch2 := make(chan int, 2)
    
    ch1 <- 1
    ch1 <- 2
    close(ch1)
    
    ch2 <- 3
    ch2 <- 4
    close(ch2)
    
    // Fan in
    merged := fanIn(ch1, ch2)
    
    for val := range merged {
        fmt.Printf("Received: %d\n", val)
    }
}
```

### Fan-out Pattern

```go
func fanOutPattern() {
    // Single input channel -> Multiple output channels
    fanOut := func(in <-chan int, numWorkers int) []<-chan int {
        outs := make([]<-chan int, numWorkers)
        
        for i := 0; i < numWorkers; i++ {
            out := make(chan int)
            outs[i] = out
            
            go func(workerID int, out chan<- int) {
                defer close(out)
                for val := range in {
                    processed := val * workerID
                    out <- processed
                }
            }(i+1, out)
        }
        
        return outs
    }
    
    // Input channel
    input := make(chan int, 5)
    for i := 1; i <= 5; i++ {
        input <- i
    }
    close(input)
    
    // Fan out to 3 workers
    outputs := fanOut(input, 3)
    
    // Collect results from all workers
    for i, output := range outputs {
        fmt.Printf("Worker %d results:\n", i+1)
        for val := range output {
            fmt.Printf("  %d\n", val)
        }
    }
}
```

### Timeout Pattern

```go
func timeoutPattern() {
    doWork := func() <-chan string {
        ch := make(chan string)
        go func() {
            time.Sleep(2 * time.Second)
            ch <- "Work completed"
        }()
        return ch
    }
    
    timeout := time.After(1 * time.Second)
    
    select {
    case result := <-doWork():
        fmt.Printf("Success: %s\n", result)
    case <-timeout:
        fmt.Println("Timeout occurred")
    }
}
```

### Heartbeat Pattern

```go
func heartbeatPattern() {
    doWork := func(done <-chan struct{}) (<-chan int, <-chan time.Time) {
        intStream := make(chan int)
        heartbeat := make(chan time.Time)
        
        go func() {
            defer close(intStream)
            defer close(heartbeat)
            
            for {
                select {
                case <-done:
                    return
                case <-time.After(100 * time.Millisecond):
                    heartbeat <- time.Now()
                case intStream <- rand.Intn(10):
                }
            }
        }()
        
        return intStream, heartbeat
    }
    
    done := make(chan struct{})
    defer close(done)
    
    intStream, heartbeat := doWork(done)
    
    for {
        select {
        case <-heartbeat:
            fmt.Println("Heartbeat received")
        case result := <-intStream:
            fmt.Printf("Result: %d\n", result)
            if result == 9 {
                fmt.Println("Found target, stopping")
                return
            }
        case <-time.After(2 * time.Second):
            fmt.Println("No activity, stopping")
            return
        }
    }
}
```

## Channel Synchronization

### Synchronizing Goroutines

```go
func synchronizationExample() {
    done := make(chan bool)
    
    go func() {
        fmt.Println("Working...")
        time.Sleep(1 * time.Second)
        fmt.Println("Work completed")
        done <- true // Signal completion
    }()
    
    <-done // Wait for completion
    fmt.Println("Main continues")
}
```

### Signaling Multiple Events

```go
func multipleSignals() {
    ready := make(chan bool)
    done := make(chan bool)
    
    go func() {
        fmt.Println("Worker starting...")
        time.Sleep(500 * time.Millisecond)
        ready <- true
        
        fmt.Println("Worker processing...")
        time.Sleep(1 * time.Second)
        done <- true
    }()
    
    <-ready
    fmt.Println("Worker is ready")
    
    <-done
    fmt.Println("Worker finished")
}
```

## Channel Best Practices

### 1. Always Close Channels

```go
// Good: Proper channel closing
func properClosing() {
    ch := make(chan int, 3)
    
    go func() {
        for i := 0; i < 3; i++ {
            ch <- i
        }
        close(ch) // Signal no more values
    }()
    
    for value := range ch {
        fmt.Printf("Value: %d\n", value)
    }
}
```

### 2. Only Close Channels Once

```go
// Bad: Multiple closes cause panic
func multipleClose() {
    ch := make(chan int)
    close(ch)
    // close(ch) // This would panic!
}

// Good: Check if channel should be closed
func safeClose(ch chan int, shouldClose bool) {
    if shouldClose {
        close(ch)
    }
}
```

### 3. Only Sender Should Close

```go
// Bad: Receiver closing channel
func badReceiverClose() {
    ch := make(chan int)
    go func() {
        ch <- 42
    }()
    value := <-ch
    close(ch) // Receiver shouldn't close!
}

// Good: Sender closes channel
func goodSenderClose() {
    ch := make(chan int)
    go func() {
        ch <- 42
        close(ch) // Sender closes
    }()
    value := <-ch
}
```

### 4. Handle Closed Channels Gracefully

```go
func handleClosedChannels() {
    ch := make(chan int, 2)
    ch <- 1
    ch <- 2
    close(ch)
    
    // Check if channel is closed
    for {
        value, ok := <-ch
        if !ok {
            fmt.Println("Channel closed")
            break
        }
        fmt.Printf("Value: %d\n", value)
    }
}
```

## Advanced Channel Techniques

### Channel of Channels

```go
func channelOfChannels() {
    // Channel that carries other channels
    chOfCh := make(chan chan int, 2)
    
    // Create worker channels
    worker1 := make(chan int)
    worker2 := make(chan int)
    
    chOfCh <- worker1
    chOfCh <- worker2
    close(chOfCh)
    
    // Process worker channels
    go func() {
        worker1 <- 100
        close(worker1)
    }()
    
    go func() {
        worker2 <- 200
        close(worker2)
    }()
    
    for ch := range chOfCh {
        if value, ok := <-ch; ok {
            fmt.Printf("Received from worker channel: %d\n", value)
        }
    }
}
```

### Buffered Channel with Semaphore Pattern

```go
func semaphorePattern() {
    const maxWorkers = 3
    semaphore := make(chan struct{}, maxWorkers)
    
    var wg sync.WaitGroup
    
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            semaphore <- struct{}{} // Acquire semaphore
            defer func() { <-semaphore }() // Release semaphore
            
            fmt.Printf("Worker %d starting\n", id)
            time.Sleep(500 * time.Millisecond)
            fmt.Printf("Worker %d finished\n", id)
        }(i)
    }
    
    wg.Wait()
}
```

### Rate Limiting with Channels

```go
func rateLimiting() {
    requests := make(chan int, 5)
    limiter := time.Tick(200 * time.Millisecond) // 5 requests per second
    
    // Send requests
    for i := 1; i <= 5; i++ {
        requests <- i
    }
    close(requests)
    
    // Process with rate limiting
    for req := range requests {
        <-limiter // Wait for rate limiter
        fmt.Printf("Processing request %d at %v\n", req, time.Now())
    }
}
```

## Channel Anti-patterns

### 1. Using Channels for Shared State

```go
// Bad: Using channels as mutex replacement
func badSharedState() {
    var counter int
    increment := make(chan bool)
    
    for i := 0; i < 1000; i++ {
        go func() {
            increment <- true
            counter++
            <-increment
        }()
    }
    // This is inefficient and error-prone
}

// Good: Use proper synchronization
func goodSharedState() {
    var counter int
    var mutex sync.Mutex
    
    for i := 0; i < 1000; i++ {
        go func() {
            mutex.Lock()
            counter++
            mutex.Unlock()
        }()
    }
}
```

### 2. Infinite Channel Growth

```go
// Bad: Unbounded buffering
func infiniteGrowth() {
    ch := make(chan int) // Unbuffered
    
    go func() {
        for i := 0; i < 1000000; i++ {
            ch <- i // May block indefinitely
        }
    }()
    
    // Slow consumer
    for value := range ch {
        time.Sleep(1 * time.Second)
        fmt.Println(value)
    }
}

// Good: Use bounded buffer or context cancellation
func boundedGrowth() {
    ch := make(chan int, 100) // Bounded buffer
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    go func() {
        for i := 0; i < 1000000; i++ {
            select {
            case ch <- i:
            case <-ctx.Done():
                return
            }
        }
    }()
}
```

## Performance Considerations

### Benchmarking Channel Operations

```go
func benchmarkChannels() {
    const iterations = 1000000
    
    // Unbuffered channel
    unbuffered := make(chan int)
    start := time.Now()
    
    go func() {
        for i := 0; i < iterations; i++ {
            unbuffered <- i
        }
        close(unbuffered)
    }()
    
    for range unbuffered {}
    unbufferedTime := time.Since(start)
    
    // Buffered channel
    buffered := make(chan int, 1000)
    start = time.Now()
    
    go func() {
        for i := 0; i < iterations; i++ {
            buffered <- i
        }
        close(buffered)
    }()
    
    for range buffered {}
    bufferedTime := time.Since(start)
    
    fmt.Printf("Unbuffered: %v\n", unbufferedTime)
    fmt.Printf("Buffered: %v\n", bufferedTime)
    fmt.Printf("Buffered is %.2fx faster\n", 
               float64(unbufferedTime)/float64(bufferedTime))
}
```

## Debugging Channels

### Detecting Deadlocks

```go
func detectDeadlock() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("Caught panic: %v\n", r)
        }
    }()
    
    ch := make(chan int)
    // ch <- 42 // This would cause deadlock!
    // <-ch     // This would also cause deadlock!
    
    // Proper usage:
    go func() {
        ch <- 42
    }()
    value := <-ch
    fmt.Printf("Value: %d\n", value)
}
```

### Channel Inspection

```go
func inspectChannels() {
    ch := make(chan int, 5)
    
    // Send some values
    ch <- 1
    ch <- 2
    
    // Note: Go doesn't provide direct channel inspection APIs
    // You need to track state manually or use debugging tools
    
    fmt.Printf("Channel capacity: %d\n", cap(ch))
    fmt.Printf("Channel length: %d\n", len(ch))
}
```

## Summary

Channels provide Go's elegant solution for concurrent communication:

✅ **Strengths:**
- Type-safe communication between goroutines
- Built-in synchronization mechanisms
- Natural backpressure and flow control
- Composable patterns for complex workflows

⚠️ **Considerations:**
- Understand blocking behavior of unbuffered channels
- Proper channel closing to avoid panics
- Choose appropriate buffer sizes
- Combine with context for cancellation

Channels work beautifully with goroutines to create robust, concurrent Go applications following the principle: "Don't communicate by sharing memory; share memory by communicating."
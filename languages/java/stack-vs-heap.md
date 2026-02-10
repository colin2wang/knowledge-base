# Stack vs Heap Memory

Understanding the differences between stack and heap memory is crucial for writing efficient Java applications. These two memory areas serve different purposes and have distinct characteristics that affect performance and memory management.

## Memory Area Comparison

| Aspect | Stack Memory | Heap Memory |
|--------|-------------|-------------|
| **Purpose** | Method execution and local variables | Object storage and dynamic allocation |
| **Access Speed** | Very fast | Slower than stack |
| **Memory Management** | Automatic (LIFO) | Garbage collected |
| **Size** | Fixed per thread | Dynamic, configurable |
| **Thread Safety** | Thread-local | Shared among threads |
| **Allocation** | Compile-time | Runtime |

## Stack Memory

Stack memory is allocated per thread and follows Last-In-First-Out (LIFO) principle.

### Stack Frame Structure

```java
public class StackFrameStructure {
    public static void main(String[] args) {
        int result = calculate(10, 20);
        System.out.println("Result: " + result);
    }
    
    public static int calculate(int a, int b) {
        int sum = add(a, b);
        int product = multiply(a, b);
        return sum + product;
    }
    
    public static int add(int x, int y) {
        int localResult = x + y;
        return localResult;
    }
    
    public static int multiply(int x, int y) {
        int localResult = x * y;
        return localResult;
    }
}
```

### Stack Memory Layout Visualization

```
Thread Stack (grows downward)
┌─────────────────────────────┐ ← High address
│ main() stack frame          │
│   args reference            │
│   result variable           │
├─────────────────────────────┤
│ calculate() stack frame     │
│   a = 10, b = 20            │
│   sum variable              │
│   product variable          │
├─────────────────────────────┤
│ add() stack frame           │
│   x = 10, y = 20            │
│   localResult = 30          │
├─────────────────────────────┤
│ multiply() stack frame      │
│   x = 10, y = 20            │
│   localResult = 200         │
└─────────────────────────────┘ ← Low address (stack pointer)
```

### Stack Memory Characteristics

```java
public class StackCharacteristics {
    // Fixed size allocation
    private static final int STACK_SIZE = 1024 * 1024; // 1MB default
    
    public static void demonstrateStackSize() {
        // Stack size can be configured per thread
        Thread defaultThread = new Thread(() -> {
            System.out.println("Default stack size thread running");
            processStackData();
        });
        
        Thread customThread = new Thread(null, // Runnable
                                       () -> processStackData(),
                                       "CustomStackThread",
                                       2 * 1024 * 1024); // 2MB stack
        
        defaultThread.start();
        customThread.start();
    }
    
    private static void processStackData() {
        // Local variables stored on stack
        int localInt = 42;
        double localDouble = 3.14159;
        String localString = "Stack data";
        
        // Method parameters also on stack
        processParameters(localInt, localDouble, localString);
    }
    
    private static void processParameters(int paramInt, double paramDouble, String paramString) {
        // Parameters are copies on the stack
        paramInt = 99; // Only affects local copy
        System.out.println("Parameter values: " + paramInt + ", " + paramDouble + ", " + paramString);
    }
}
```

### Stack Overflow Prevention

```java
public class StackOverflowHandling {
    private static int recursionDepth = 0;
    
    public static void demonstrateSafeRecursion() {
        try {
            safeRecursiveMethod(1000); // Controlled recursion depth
        } catch (StackOverflowError e) {
            System.err.println("Stack overflow prevented at depth: " + recursionDepth);
        }
    }
    
    private static void safeRecursiveMethod(int maxDepth) {
        if (recursionDepth >= maxDepth) {
            return; // Base case to prevent overflow
        }
        
        recursionDepth++;
        System.out.println("Recursion depth: " + recursionDepth);
        
        // Tail recursion optimization simulation
        if (recursionDepth < maxDepth) {
            safeRecursiveMethod(maxDepth);
        }
        
        recursionDepth--; // Cleanup on unwinding
    }
    
    public static void iterativeAlternative() {
        // Convert recursive algorithm to iterative
        for (int i = 0; i < 1000000; i++) {
            processIteration(i);
        }
    }
    
    private static void processIteration(int iteration) {
        // Iterative processing - no stack growth
        System.out.println("Processing iteration: " + iteration);
    }
}
```

## Heap Memory

Heap memory is used for dynamic object allocation and is managed by the garbage collector.

### Heap Memory Organization

```java
public class HeapOrganization {
    public static void demonstrateHeapStructure() {
        // Object allocation in heap
        Person person = new Person("John Doe", 30);
        List<String> hobbies = new ArrayList<>();
        hobbies.add("Reading");
        hobbies.add("Programming");
        
        // Array allocation
        int[] numbers = new int[1000];
        String[] names = new String[500];
        
        // Large object allocation
        LargeDataStructure largeData = new LargeDataStructure();
    }
    
    static class Person {
        private String name;
        private int age;
        private List<String> hobbies;
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
            this.hobbies = new ArrayList<>();
        }
    }
    
    static class LargeDataStructure {
        private byte[] data = new byte[1024 * 1024]; // 1MB
        private String[] metadata = new String[10000];
    }
}
```

### Heap Memory Layout

```
Heap Memory Structure
┌─────────────────────────────────────────────┐
│ Young Generation                            │
│  ┌─────────────────────────────────────────┐ │
│  │ Eden Space                              │ │
│  │  New objects allocated here             │ │
│  └─────────────────────────────────────────┘ │
│  ┌─────────────┐  ┌─────────────┐           │
│  │ Survivor S0 │  │ Survivor S1 │           │
│  │  Surviving  │  │  objects    │           │
│  │  objects    │  │  alternate  │           │
│  └─────────────┘  └─────────────┘           │
├─────────────────────────────────────────────┤
│ Old Generation (Tenured Space)              │
│  Long-lived objects promoted here           │
│  ┌─────────────────────────────────────────┐ │
│  │ Mature objects that survived multiple   │ │
│  │ garbage collection cycles               │ │
│  └─────────────────────────────────────────┘ │
├─────────────────────────────────────────────┤
│ Metaspace (Java 8+)                         │
│  Class metadata, method bytecode            │
└─────────────────────────────────────────────┘
```

### Heap Memory Management

```java
public class HeapManagement {
    public static void demonstrateHeapAllocation() {
        // TLAB (Thread Local Allocation Buffer) allocation
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        for (int i = 0; i < 1000; i++) {
            final int taskId = i;
            executor.submit(() -> {
                // Each thread allocates in its own TLAB
                List<Object> threadObjects = new ArrayList<>();
                for (int j = 0; j < 1000; j++) {
                    threadObjects.add(new SmallObject(taskId, j));
                }
                // Objects eligible for GC when method completes
            });
        }
        
        executor.shutdown();
    }
    
    static class SmallObject {
        private int taskId;
        private int objectId;
        private String data;
        
        public SmallObject(int taskId, int objectId) {
            this.taskId = taskId;
            this.objectId = objectId;
            this.data = "Data_" + taskId + "_" + objectId;
        }
    }
    
    public static void demonstrateDirectMemory() {
        // Off-heap allocation using ByteBuffer
        ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024 * 1024); // 1MB direct buffer
        // Not managed by GC, must be explicitly cleaned up
        
        // Cleaner pattern for off-heap resources
        Cleaner cleaner = Cleaner.create();
        cleaner.register(directBuffer, () -> {
            System.out.println("Direct buffer cleaned up");
            // Cleanup native resources
        });
    }
}
```

## Performance Comparison

### Access Speed Differences

```java
public class MemoryAccessPerformance {
    private static final int ITERATIONS = 1000000;
    
    public static void compareAccessSpeed() {
        // Stack access performance
        long stackStartTime = System.nanoTime();
        stackAccessTest();
        long stackEndTime = System.nanoTime();
        
        // Heap access performance
        long heapStartTime = System.nanoTime();
        heapAccessTest();
        long heapEndTime = System.nanoTime();
        
        System.out.printf("Stack access time: %.2f ms%n", 
                         (stackEndTime - stackStartTime) / 1_000_000.0);
        System.out.printf("Heap access time: %.2f ms%n", 
                         (heapEndTime - heapStartTime) / 1_000_000.0);
    }
    
    private static void stackAccessTest() {
        int[] stackArray = new int[1000]; // Array reference on stack, elements on heap
        for (int i = 0; i < ITERATIONS; i++) {
            for (int j = 0; j < stackArray.length; j++) {
                stackArray[j] = i + j;
            }
        }
    }
    
    private static void heapAccessTest() {
        List<Integer> heapList = new ArrayList<>(1000);
        for (int i = 0; i < 1000; i++) {
            heapList.add(i);
        }
        
        for (int i = 0; i < ITERATIONS; i++) {
            for (int j = 0; j < heapList.size(); j++) {
                heapList.set(j, i + j);
            }
        }
    }
}
```

### Memory Allocation Benchmarks

```java
public class AllocationBenchmark {
    public static void benchmarkAllocations() {
        // Stack allocation benchmark
        long stackStart = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            processStackData(i);
        }
        long stackEnd = System.nanoTime();
        
        // Heap allocation benchmark
        long heapStart = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            processHeapData(i);
        }
        long heapEnd = System.nanoTime();
        
        System.out.printf("Stack allocation: %.2f ms%n", 
                         (stackEnd - stackStart) / 1_000_000.0);
        System.out.printf("Heap allocation: %.2f ms%n", 
                         (heapEnd - heapStart) / 1_000_000.0);
    }
    
    private static void processStackData(int value) {
        // Local variables on stack
        int local1 = value * 2;
        int local2 = value * 3;
        int result = local1 + local2;
        // Very fast allocation and deallocation
    }
    
    private static void processHeapData(int value) {
        // Object allocation on heap
        DataObject obj = new DataObject(value);
        obj.process();
        // Slower due to heap allocation and eventual GC
    }
    
    static class DataObject {
        private int value;
        private String data;
        
        public DataObject(int value) {
            this.value = value;
            this.data = "Data_" + value;
        }
        
        public void process() {
            // Processing logic
        }
    }
}
```

## Thread Safety Considerations

### Stack Thread Safety

```java
public class StackThreadSafety {
    public static void demonstrateStackSafety() {
        // Each thread has its own stack - naturally thread-safe
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        for (int i = 0; i < 10; i++) {
            final int threadId = i;
            executor.submit(() -> {
                // Thread-local stack variables
                int localVar = threadId * 100;
                String threadName = "Thread-" + threadId;
                
                processThreadLocalData(localVar, threadName);
                // No synchronization needed - each thread has separate stack
            });
        }
        
        executor.shutdown();
    }
    
    private static void processThreadLocalData(int value, String name) {
        // Stack variables are thread-local
        System.out.println(name + " processing value: " + value);
        // Safe concurrent execution
    }
}
```

### Heap Thread Safety

```java
public class HeapThreadSafety {
    private static int sharedCounter = 0; // Heap variable - requires synchronization
    private static final Object lock = new Object();
    
    public static void demonstrateHeapConcurrency() {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                // Unsafe increment - race condition
                unsafeIncrement();
                
                // Safe increment with synchronization
                safeIncrement();
                
                // Safe increment with atomic operations
                atomicIncrement();
            });
        }
        
        executor.shutdown();
        
        try {
            executor.awaitTermination(10, TimeUnit.SECONDS);
            System.out.println("Final counter value: " + sharedCounter);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void unsafeIncrement() {
        sharedCounter++; // Race condition - not thread-safe
    }
    
    private static void safeIncrement() {
        synchronized (lock) {
            sharedCounter++; // Thread-safe with explicit locking
        }
    }
    
    private static void atomicIncrement() {
        // Using AtomicInteger for thread-safe operations
        java.util.concurrent.atomic.AtomicInteger atomicCounter = 
            new java.util.concurrent.atomic.AtomicInteger(0);
        atomicCounter.incrementAndGet(); // Thread-safe atomic operation
    }
}
```

## Memory Leak Scenarios

### Stack Memory Leaks

```java
public class StackMemoryLeaks {
    public static void demonstrateStackIssues() {
        // Stack overflow due to deep recursion
        try {
            deepRecursion(0);
        } catch (StackOverflowError e) {
            System.err.println("Stack overflow occurred");
        }
        
        // Large local variables consuming stack space
        processLargeLocalData();
    }
    
    private static void deepRecursion(int depth) {
        if (depth > 10000) return;
        deepRecursion(depth + 1); // Eventually causes stack overflow
    }
    
    private static void processLargeLocalData() {
        // Large arrays on stack can cause issues
        long[] largeLocalArray = new long[100000]; // Elements on heap, but reference on stack
        // Process array...
    }
}
```

### Heap Memory Leaks

```java
public class HeapMemoryLeaks {
    // Static collection causing memory leak
    private static final List<Object> STATIC_LEAK = new ArrayList<>();
    
    public static void demonstrateHeapLeaks() {
        // Accumulating objects in static collection
        for (int i = 0; i < 1000000; i++) {
            STATIC_LEAK.add(new LeakingObject(i)); // Memory leak!
        }
        
        // Unclosed resources
        unclosedResourceLeak();
        
        // Listener leaks
        listenerLeak();
    }
    
    private static void unclosedResourceLeak() {
        try {
            FileInputStream fis = new FileInputStream("large-file.dat");
            // Forgot to close the stream - resource leak
            // Should use try-with-resources
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        // Stream remains open, holding heap references
    }
    
    private static void listenerLeak() {
        EventBus eventBus = new EventBus();
        LeakyListener listener = new LeakyListener();
        eventBus.register(listener);
        
        // Even after we're done with listener, it remains registered
        // and holds references, preventing GC
    }
    
    static class LeakingObject {
        private byte[] data = new byte[1024]; // 1KB each
        private int id;
        
        public LeakingObject(int id) {
            this.id = id;
        }
    }
    
    static class EventBus {
        private List<Object> listeners = new ArrayList<>();
        
        public void register(Object listener) {
            listeners.add(listener);
        }
    }
    
    static class LeakyListener {
        private List<String> capturedData = new ArrayList<>();
        
        public void handleEvent(String event) {
            capturedData.add(event); // Accumulates data
        }
    }
}
```

## Best Practices

### Stack Usage Guidelines

```java
public class StackBestPractices {
    // Keep local variables small and simple
    public static void efficientStackUsage() {
        // Good: Primitive types and small objects
        int count = 0;
        String name = "John";
        Point location = new Point(10, 20); // Small object
        
        // Avoid: Large local arrays
        // int[] hugeArray = new int[1000000]; // Better as instance variable
        
        processEfficiently(count, name, location);
    }
    
    // Use tail recursion when possible
    public static int tailRecursiveSum(int n, int accumulator) {
        if (n <= 0) return accumulator;
        return tailRecursiveSum(n - 1, accumulator + n); // Tail call optimization friendly
    }
    
    // Limit recursion depth
    public static void controlledRecursion() {
        performControlledOperation(1000); // Reasonable limit
    }
    
    private static void performControlledOperation(int maxDepth) {
        // Implementation with proper base cases
    }
}
```

### Heap Usage Guidelines

```java
public class HeapBestPractices {
    // Use object pooling for frequently created objects
    private static final ObjectPool<StringBuilder> STRING_BUILDER_POOL = 
        new ObjectPool<>(StringBuilder::new, 50);
    
    public static void efficientHeapUsage() {
        // Reuse objects from pool
        StringBuilder sb = STRING_BUILDER_POOL.acquire();
        try {
            sb.append("Efficient ").append("string ").append("building");
            String result = sb.toString();
            // Use result...
        } finally {
            STRING_BUILDER_POOL.release(sb); // Return to pool
        }
        
        // Use appropriate data structures
        demonstrateEfficientStructures();
    }
    
    private static void demonstrateEfficientStructures() {
        // For primitive collections, consider specialized libraries
        // Eclipse Collections, Trove, etc.
        
        // Use lazy initialization
        LazyInitializer<DataStructure> lazyData = 
            new LazyInitializer<>(DataStructure::new);
        
        // Only create expensive objects when actually needed
        if (needsExpensiveOperation()) {
            DataStructure data = lazyData.get();
            data.process();
        }
    }
    
    private static boolean needsExpensiveOperation() {
        return Math.random() > 0.5;
    }
    
    static class DataStructure {
        private byte[] largeBuffer = new byte[1024 * 1024]; // 1MB
        
        public void process() {
            // Processing logic
        }
    }
}

// Supporting classes
class ObjectPool<T> {
    private final Queue<T> pool = new ConcurrentLinkedQueue<>();
    private final Supplier<T> factory;
    private final int maxSize;
    
    public ObjectPool(Supplier<T> factory, int maxSize) {
        this.factory = factory;
        this.maxSize = maxSize;
    }
    
    public T acquire() {
        T object = pool.poll();
        return object != null ? object : factory.get();
    }
    
    public void release(T object) {
        if (pool.size() < maxSize) {
            pool.offer(object);
        }
    }
}

class LazyInitializer<T> {
    private volatile T instance;
    private final Supplier<T> supplier;
    
    public LazyInitializer(Supplier<T> supplier) {
        this.supplier = supplier;
    }
    
    public T get() {
        if (instance == null) {
            synchronized (this) {
                if (instance == null) {
                    instance = supplier.get();
                }
            }
        }
        return instance;
    }
}
```

Understanding the differences between stack and heap memory helps developers write more efficient, reliable Java applications while avoiding common memory-related issues.
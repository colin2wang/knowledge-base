# Java Memory Management

Java memory management is handled automatically by the JVM through garbage collection, but understanding the different memory areas and their characteristics is crucial for writing efficient and performant applications.

## JVM Memory Areas Overview

The JVM divides memory into several distinct areas, each serving specific purposes in the application lifecycle.

## 1. Metaspace

Metaspace replaced PermGen space starting from Java 8 and stores class metadata.

### Characteristics
- **Purpose**: Stores class definitions, method bytecode, constant pool, field/method data
- **Location**: Native memory (off-heap)
- **Size**: Dynamically expands based on application needs
- **Management**: Automatic growth and shrinkage

### Configuration Options
```bash
# Set initial metaspace size
-XX:MetaspaceSize=256m

# Set maximum metaspace size
-XX:MaxMetaspaceSize=512m

# Set compressed class space size
-XX:CompressedClassSpaceSize=64m
```

### Common Issues
```java
// Metaspace OutOfMemoryError
// Usually caused by:
// 1. Excessive class generation (dynamic proxies, bytecode manipulation)
// 2. Class loader leaks
// 3. Insufficient metaspace allocation

public class MetaspaceLeakExample {
    public static void main(String[] args) {
        List<ClassLoader> loaders = new ArrayList<>();
        while (true) {
            ClassLoader loader = new URLClassLoader(new URL[0]);
            loaders.add(loader);
            // Each loader retains metaspace memory
        }
    }
}
```

### Monitoring and Analysis
```java
// Monitor metaspace usage programmatically
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;
import java.lang.management.MemoryUsage;

public class MetaspaceMonitor {
    public static void monitorMetaspace() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        // Note: Metaspace is not directly accessible through standard beans
        // Use jstat or JMC for detailed metaspace monitoring
    }
}
```

## 2. Heap Memory

The heap is the runtime data area where objects are allocated and garbage collected.

### Heap Structure
```
Young Generation          Old Generation
┌─────────────┐          ┌─────────────┐
│   Eden      │          │             │
├─────────────┤          │             │
│ Survivor S0 │          │             │
├─────────────┤          │             │
│ Survivor S1 │          │             │
└─────────────┘          └─────────────┘
```

### Young Generation
**Eden Space**: Where new objects are initially allocated
**Survivor Spaces**: Two equal-sized spaces (S0, S1) that hold surviving objects

### Old Generation (Tenured Generation)
Holds long-lived objects that survive multiple garbage collection cycles

### Configuration Options
```bash
# Initial and maximum heap size
-Xms2g -Xmx4g

# Young generation size
-XX:NewSize=512m -XX:MaxNewSize=1g

# Ratio of young to old generation
-XX:NewRatio=2  # Old:Young = 2:1

# Survivor ratio within young generation
-XX:SurvivorRatio=8  # Eden:Survivor = 8:1
```

### Garbage Collection in Heap
```java
public class HeapGCExample {
    public static void demonstrateHeapBehavior() {
        // Objects created in Eden space
        List<String> shortLivedObjects = new ArrayList<>();
        
        // Fill Eden space
        for (int i = 0; i < 100000; i++) {
            shortLivedObjects.add("Object #" + i);
        }
        
        // Clear references - objects become eligible for GC
        shortLivedObjects.clear();
        
        // Long-lived objects move to old generation
        List<String> longLivedObjects = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            longLivedObjects.add(createLongLivedObject(i));
        }
        // These objects will eventually be promoted to old generation
    }
    
    private static String createLongLivedObject(int id) {
        return "Long-lived object #" + id;
    }
}
```

## 3. Stack Memory

Each thread has its own stack, which stores method call frames and local variables.

### Stack Frame Structure
```
Thread Stack
┌─────────────────┐ ← Stack pointer
│ Local Variables │
├─────────────────┤
│ Operand Stack   │
├─────────────────┤
│ Frame Data      │
└─────────────────┘
```

### Stack Characteristics
- **Per-thread allocation**: Each thread gets its own stack
- **Fixed size**: Typically 1MB per thread (configurable)
- **Fast allocation**: Stack allocation is very fast
- **Automatic cleanup**: Frames are popped when methods return

### Configuration Options
```bash
# Stack size per thread
-Xss512k  # 512KB per thread
-Xss1m    # 1MB per thread (default on most systems)
```

### Stack Overflow Example
```java
public class StackOverflowExample {
    private static int recursionDepth = 0;
    
    public static void recursiveMethod() {
        recursionDepth++;
        System.out.println("Recursion depth: " + recursionDepth);
        recursiveMethod(); // Infinite recursion
    }
    
    public static void demonstrateStackOverflow() {
        try {
            recursiveMethod();
        } catch (StackOverflowError e) {
            System.err.println("Stack overflow at depth: " + recursionDepth);
            // Handle stack overflow gracefully
        }
    }
}
```

### Thread Stack Management
```java
public class ThreadStackExample {
    public static void demonstrateThreadStacks() {
        // Create multiple threads with different stack sizes
        Thread thread1 = new Thread(() -> {
            // This thread uses default stack size
            processLargeData();
        });
        
        Thread thread2 = new Thread(null, // Runnable
                                  () -> processLargeData(),
                                  "CustomThread", // Thread name
                                  2 * 1024 * 1024); // 2MB stack size
        
        thread1.start();
        thread2.start();
    }
    
    private static void processLargeData() {
        // Method that might need larger stack
        int[] largeArray = new int[100000];
        // Process array...
    }
}
```

## Memory Allocation Strategies

### Object Allocation Process
```java
public class MemoryAllocationExample {
    public static void demonstrateAllocation() {
        // Fast allocation in TLAB (Thread Local Allocation Buffer)
        Object obj1 = new Object();  // Fast path - TLAB allocation
        
        // Large object allocation bypasses TLAB
        byte[] largeArray = new byte[1024 * 1024]; // 1MB array
        
        // Array allocation with specific sizing
        String[] stringArray = new String[1000];
        
        // Object with fields
        Person person = new Person("John", 30);
    }
}

class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

### Thread Local Allocation Buffers (TLABs)
```java
public class TLABExample {
    public static void demonstrateTLAB() {
        // Multiple threads allocating objects concurrently
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                // Each thread allocates in its own TLAB
                List<Object> objects = new ArrayList<>();
                for (int j = 0; j < 100; j++) {
                    objects.add(new Object());
                }
            });
        }
        
        executor.shutdown();
    }
}
```

## Memory Monitoring and Profiling

### Programmatic Memory Monitoring
```java
import java.lang.management.*;
import java.util.List;

public class MemoryMonitor {
    public static void monitorMemoryUsage() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        List<MemoryPoolMXBean> pools = ManagementFactory.getMemoryPoolMXBeans();
        
        // Get heap memory usage
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        System.out.println("Heap Used: " + heapUsage.getUsed() / (1024 * 1024) + " MB");
        System.out.println("Heap Max: " + heapUsage.getMax() / (1024 * 1024) + " MB");
        
        // Monitor specific memory pools
        for (MemoryPoolMXBean pool : pools) {
            MemoryUsage usage = pool.getUsage();
            System.out.printf("%s: %d MB used%n", 
                pool.getName(), 
                usage.getUsed() / (1024 * 1024));
        }
    }
    
    public static void setupMemoryAlerts() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        
        // Set threshold for heap usage
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        long threshold = (long) (heapUsage.getMax() * 0.8); // 80% threshold
        
        // Add notification listener for memory alerts
        NotificationEmitter emitter = (NotificationEmitter) memoryBean;
        emitter.addNotificationListener((notification, handback) -> {
            if (notification.getType().equals(MemoryNotificationInfo.MEMORY_THRESHOLD_EXCEEDED)) {
                System.err.println("Memory threshold exceeded!");
                performEmergencyCleanup();
            }
        }, null, null);
        
        memoryBean.setUsageThreshold(threshold);
    }
    
    private static void performEmergencyCleanup() {
        // Emergency memory cleanup procedures
        System.gc(); // Request garbage collection
        // Clear caches, release resources
    }
}
```

### GC Logging Configuration
```bash
# Enable GC logging (Java 9+)
-Xlog:gc*:gc.log:time,tags

# Detailed GC logging with specific options
-Xlog:gc*=debug:stdout:time,uptime,level,tags
-XX:+PrintGCDateStamps
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-XX:+PrintGCApplicationStoppedTime

# GC log rotation
-Xlog:gc*:gc.log.%p.%t:time,tags:filecount=5,filesize=10M
```

## Memory Optimization Techniques

### Object Pooling
```java
public class ObjectPool<T> {
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
        // Otherwise discard the object to allow GC
    }
}

// Usage example
ObjectPool<StringBuilder> stringBuilderPool = 
    new ObjectPool<>(StringBuilder::new, 100);
```

### Memory-Efficient Data Structures
```java
public class MemoryEfficientStructures {
    public static void demonstrateOptimizations() {
        // Use primitive collections when possible
        // Eclipse Collections or Trove for primitive collections
        
        // Compact string representations
        String[] compactStrings = new String[1000];
        // Consider using char[] arrays for large string collections
        
        // Efficient caching strategies
        Map<String, SoftReference<ExpensiveObject>> cache = 
            new ConcurrentHashMap<>();
    }
}
```

### Weak and Soft References
```java
import java.lang.ref.*;
import java.util.WeakHashMap;

public class ReferenceExamples {
    public static void demonstrateReferences() {
        // Weak references - eligible for GC when no strong references exist
        WeakReference<String> weakRef = new WeakReference<>("Weak Object");
        
        // Soft references - cleared only when memory is low
        SoftReference<byte[]> softRef = new SoftReference<>(new byte[1024 * 1024]);
        
        // WeakHashMap - entries automatically removed when keys are GC'd
        Map<Object, String> weakMap = new WeakHashMap<>();
        
        // Phantom references - for post-finalization cleanup
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        PhantomReference<Object> phantomRef = 
            new PhantomReference<>(new Object(), queue);
    }
}
```

## Common Memory Issues and Solutions

### Memory Leaks
```java
public class MemoryLeakExamples {
    // Static collection holding references
    private static final List<Object> STATIC_LIST = new ArrayList<>();
    
    public static void badPractice() {
        // Adding objects to static collection prevents GC
        for (int i = 0; i < 1000000; i++) {
            STATIC_LIST.add(new Object()); // Memory leak!
        }
    }
    
    public static void goodPractice() {
        // Use weak references for caches
        Map<String, WeakReference<Object>> cache = new HashMap<>();
        // Or clear collections periodically
        STATIC_LIST.clear(); // Proper cleanup
    }
}
```

### Class Loader Leaks
```java
public class ClassLoaderLeakPrevention {
    public static void preventClassLoaderLeaks() {
        // Properly clean up thread locals
        ThreadLocal<Object> threadLocal = new ThreadLocal<>();
        try {
            threadLocal.set(new Object());
            // Use the object
        } finally {
            threadLocal.remove(); // Prevent leak
        }
        
        // Clean up static references in web applications
        // Implement ServletContextListener to clean up static data
    }
}
```

Understanding Java memory management is essential for building robust, scalable applications. Proper memory configuration and monitoring can significantly improve application performance and stability.
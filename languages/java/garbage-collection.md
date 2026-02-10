# Garbage Collection

Garbage Collection (GC) is Java's automatic memory management system that reclaims memory occupied by objects that are no longer reachable or referenced by the application. This eliminates the need for manual memory deallocation and prevents memory leaks.

## Garbage Collection Fundamentals

### What is Garbage Collection?

Garbage collection automatically identifies and frees memory occupied by objects that are no longer in use by the application.

```java
public class GarbageCollectionBasics {
    public static void demonstrateGCConcept() {
        // Object becomes eligible for GC when no references exist
        String str = new String("Hello World");
        str = null; // Object is now unreachable - eligible for GC
        
        // Another example
        List<String> list = new ArrayList<>();
        list.add("Item 1");
        list.add("Item 2");
        list = null; // Entire list and its contents become eligible for GC
    }
    
    // Objects become eligible when they go out of scope
    public static void scopeBasedGC() {
        {
            String localString = new String("Local object");
            // localString goes out of scope at the end of this block
            // Object becomes eligible for GC
        }
        // localString no longer exists here
    }
}
```

### Reachability Analysis

The JVM determines object eligibility through reachability analysis, starting from GC Roots.

#### GC Roots

```java
public class GCRoots {
    private static Object staticObject = new Object(); // Static variable - GC Root
    private Object instanceObject = new Object();      // Instance variable
    
    public void demonstrateGCRoots() {
        Object localVariable = new Object();           // Local variable - GC Root
        Thread.currentThread();                        // Running threads - GC Roots
        
        // Objects reachable from GC roots are NOT eligible for GC
        Object reachable = localVariable;              // Still reachable
        
        localVariable = null;                          // Now only reachable through 'reachable'
        reachable = null;                              // Now eligible for GC
    }
    
    // Method parameters are GC roots during method execution
    public static void processObject(Object param) {
        // 'param' is a GC root during this method execution
        Object localCopy = param;
        // Both param and localCopy are GC roots
    }
}
```

## Types of Garbage Collectors

### 1. Serial Garbage Collector

Single-threaded collector suitable for small applications.

```java
public class SerialGC {
    public static void configureSerialGC() {
        // JVM options for Serial GC
        // -XX:+UseSerialGC
        
        System.out.println("Using Serial Garbage Collector");
        
        // Demonstrating GC behavior
        List<Object> objects = new ArrayList<>();
        for (int i = 0; i < 100000; i++) {
            objects.add(new byte[1024]); // 1KB objects
            if (i % 10000 == 0) {
                objects.clear(); // Trigger GC
                System.gc(); // Suggest garbage collection
            }
        }
    }
}
```

### 2. Parallel Garbage Collector (Throughput Collector)

Multi-threaded collector optimized for maximum application throughput.

```java
public class ParallelGC {
    public static void configureParallelGC() {
        // JVM options for Parallel GC
        // -XX:+UseParallelGC
        // -XX:ParallelGCThreads=4
        
        System.out.println("Using Parallel Garbage Collector");
        
        // High-throughput scenario
        ExecutorService executor = Executors.newFixedThreadPool(8);
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                // Create temporary objects
                List<String> temp = new ArrayList<>();
                for (int j = 0; j < 1000; j++) {
                    temp.add("Temp object " + j);
                }
                // Objects become eligible for GC when method ends
            });
        }
        executor.shutdown();
    }
}
```

### 3. G1 Garbage Collector (Garbage First)

Low-pause collector that meets pause time goals while maintaining good throughput.

```java
public class G1GC {
    public static void configureG1GC() {
        // JVM options for G1 GC
        // -XX:+UseG1GC
        // -XX:MaxGCPauseMillis=200
        // -XX:G1HeapRegionSize=16m
        
        System.out.println("Using G1 Garbage Collector");
        
        // Demonstrate region-based allocation
        List<Object> objects = new ArrayList<>();
        for (int i = 0; i < 500000; i++) {
            objects.add(new LargeObject()); // Objects distributed across regions
            if (i % 50000 == 0) {
                objects.subList(0, 25000).clear(); // Free some regions
            }
        }
    }
    
    static class LargeObject {
        private byte[] data = new byte[1024]; // 1KB object
    }
}
```

### 4. Z Garbage Collector (ZGC)

Ultra-low latency collector designed for applications requiring sub-millisecond pause times.

```java
public class ZGC {
    public static void configureZGC() {
        // JVM options for ZGC (Java 11+)
        // -XX:+UseZGC
        // -XX:+UnlockExperimentalVMOptions
        
        System.out.println("Using Z Garbage Collector");
        
        // Low-latency requirement scenario
        long startTime = System.nanoTime();
        processRealTimeData();
        long endTime = System.nanoTime();
        
        System.out.printf("Processing time: %.2f ms%n", 
                         (endTime - startTime) / 1_000_000.0);
    }
    
    private static void processRealTimeData() {
        // Continuous processing with strict latency requirements
        for (int i = 0; i < 1000000; i++) {
            RealTimeEvent event = new RealTimeEvent(i);
            processEvent(event);
            // ZGC ensures minimal pause times during processing
        }
    }
    
    private static void processEvent(RealTimeEvent event) {
        // Event processing logic
    }
    
    static class RealTimeEvent {
        private int id;
        private long timestamp;
        
        public RealTimeEvent(int id) {
            this.id = id;
            this.timestamp = System.currentTimeMillis();
        }
    }
}
```

### 5. Shenandoah Garbage Collector

Low-pause collector that performs evacuation concurrently with application threads.

```java
public class ShenandoahGC {
    public static void configureShenandoahGC() {
        // JVM options for Shenandoah GC (Java 12+)
        // -XX:+UseShenandoahGC
        // -XX:+UnlockExperimentalVMOptions
        
        System.out.println("Using Shenandoah Garbage Collector");
        
        // Concurrent processing scenario
        ScheduledExecutorService scheduler = 
            Executors.newScheduledThreadPool(4);
        
        scheduler.scheduleAtFixedRate(() -> {
            // Continuous object allocation
            List<TemporaryObject> batch = new ArrayList<>();
            for (int i = 0; i < 10000; i++) {
                batch.add(new TemporaryObject());
            }
            // Shenandoah handles concurrent GC during allocation
        }, 0, 100, TimeUnit.MILLISECONDS);
        
        // Let it run for demonstration
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        scheduler.shutdown();
    }
    
    static class TemporaryObject {
        private String data = "Temporary data";
        private long timestamp = System.currentTimeMillis();
    }
}
```

## GC Algorithms and Processes

### Mark and Sweep Algorithm

```java
public class MarkAndSweep {
    public static void demonstrateMarkSweep() {
        // Phase 1: Mark - Identify reachable objects
        // Phase 2: Sweep - Free unreferenced objects
        
        List<Object> liveObjects = new ArrayList<>();
        List<Object> deadObjects = new ArrayList<>();
        
        // Create mixed live and dead objects
        for (int i = 0; i < 1000; i++) {
            Object obj = new Object();
            if (i % 3 == 0) {
                liveObjects.add(obj); // Keep reference - live object
            } else {
                deadObjects.add(obj); // No reference - will be GC'd
            }
        }
        
        // Clear dead object references
        deadObjects.clear();
        
        // Suggest GC - triggers mark and sweep
        System.gc();
        
        System.out.println("Live objects retained: " + liveObjects.size());
    }
}
```

### Generational Collection

```java
public class GenerationalGC {
    public static void demonstrateGenerationalCollection() {
        // Young Generation GC (Minor GC)
        List<String> youngObjects = new ArrayList<>();
        for (int i = 0; i < 50000; i++) {
            youngObjects.add("Young object " + i);
        }
        youngObjects.clear(); // Triggers Minor GC
        
        // Old Generation GC (Major GC/Full GC)
        List<String> oldObjects = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            String obj = createLongLivedObject(i);
            oldObjects.add(obj);
        }
        // These objects will be promoted to old generation
        // Full GC occurs when old generation fills up
        
        System.gc(); // May trigger Full GC
    }
    
    private static String createLongLivedObject(int id) {
        return "Old generation object #" + id;
    }
}
```

### Stop-The-World Events

```java
public class StopTheWorldDemo {
    private static volatile boolean gcInProgress = false;
    
    public static void demonstrateSTW() {
        // Monitor GC pauses
        Thread monitoringThread = new Thread(() -> {
            long lastTime = System.nanoTime();
            while (!Thread.currentThread().isInterrupted()) {
                long currentTime = System.nanoTime();
                if (gcInProgress && (currentTime - lastTime) > 10_000_000) { // 10ms
                    System.out.printf("GC pause detected: %.2f ms%n", 
                                    (currentTime - lastTime) / 1_000_000.0);
                }
                lastTime = currentTime;
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        });
        
        monitoringThread.setDaemon(true);
        monitoringThread.start();
        
        // Trigger heavy GC activity
        List<Object> objects = new ArrayList<>();
        for (int i = 0; i < 1000000; i++) {
            objects.add(new byte[1024]); // Allocate 1GB of data
            if (i % 100000 == 0) {
                gcInProgress = true;
                objects.clear();
                System.gc(); // STW event
                gcInProgress = false;
            }
        }
    }
}
```

## GC Tuning and Configuration

### Heap Size Configuration

```bash
# Basic heap configuration
-Xms2g                    # Initial heap size: 2GB
-Xmx4g                    # Maximum heap size: 4GB

# Generation-specific settings
-XX:NewSize=512m          # Initial young generation size
-XX:MaxNewSize=1g         # Maximum young generation size
-XX:NewRatio=3            # Old:Young ratio = 3:1
-XX:SurvivorRatio=8       # Eden:Survivor ratio = 8:1
```

### GC-Specific Tuning

```bash
# Serial GC tuning
-XX:+UseSerialGC
-XX:MaxGCPauseMillis=100

# Parallel GC tuning
-XX:+UseParallelGC
-XX:ParallelGCThreads=4
-XX:MaxGCPauseMillis=200
-XX:GCTimeRatio=9         # Throughput goal: 90% application time

# G1 GC tuning
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=16m
-XX:G1NewSizePercent=20
-XX:G1MaxNewSizePercent=30

# ZGC tuning
-XX:+UseZGC
-XX:SoftMaxHeapSize=32g   # Soft maximum heap size
```

### Adaptive Sizing

```java
public class AdaptiveSizing {
    public static void demonstrateAdaptiveBehavior() {
        // JVM automatically adjusts heap sizes based on application behavior
        List<Object> workload = new ArrayList<>();
        
        // Simulate varying workload
        for (int phase = 0; phase < 3; phase++) {
            int objectCount = phase == 1 ? 100000 : 10000; // Varying intensity
            
            for (int i = 0; i < objectCount; i++) {
                workload.add(new WorkObject(phase, i));
            }
            
            // Clear and GC to observe adaptive behavior
            workload.clear();
            System.gc();
            
            try {
                Thread.sleep(1000); // Allow JVM to adapt
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    static class WorkObject {
        private int phase;
        private int id;
        private byte[] data;
        
        public WorkObject(int phase, int id) {
            this.phase = phase;
            this.id = id;
            this.data = new byte[1024]; // 1KB object
        }
    }
}
```

## GC Monitoring and Analysis

### Programmatic GC Monitoring

```java
import java.lang.management.*;
import java.util.List;

public class GCMonitoring {
    public static void monitorGCActivity() {
        List<GarbageCollectorMXBean> gcBeans = 
            ManagementFactory.getGarbageCollectorMXBeans();
        
        System.out.println("Available GC collectors:");
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.println("- " + gcBean.getName());
        }
        
        // Continuous monitoring
        Timer timer = new Timer(true);
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                reportGCCollectorStats();
            }
        }, 0, 5000); // Every 5 seconds
    }
    
    private static void reportGCCollectorStats() {
        List<GarbageCollectorMXBean> gcBeans = 
            ManagementFactory.getGarbageCollectorMXBeans();
        
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            long collections = gcBean.getCollectionCount();
            long time = gcBean.getCollectionTime();
            
            if (collections > 0) {
                System.out.printf("%s: %d collections, %d ms total, %.2f ms avg%n",
                    gcBean.getName(), collections, time, 
                    (double) time / collections);
            }
        }
    }
    
    public static void setupGCCollectionListener() {
        NotificationEmitter emitter = 
            (NotificationEmitter) ManagementFactory.getMemoryMXBean();
        
        emitter.addNotificationListener((notification, handback) -> {
            if (notification.getType().equals(GarbageCollectionNotificationInfo.GARBAGE_COLLECTION_NOTIFICATION)) {
                GarbageCollectionNotificationInfo info = 
                    GarbageCollectionNotificationInfo.from((CompositeData) notification.getUserData());
                
                System.out.printf("GC Event: %s - %s (%d ms)%n",
                    info.getGcName(), info.getGcAction(), info.getGcInfo().getDuration());
            }
        }, null, null);
    }
}
```

### GC Logging Configuration

```bash
# Unified GC logging (Java 9+)
-Xlog:gc*:gc.log:time,uptime,level,tags

# Detailed GC logging
-Xlog:gc*=debug:stdout:time,uptime,level,tags
-XX:+PrintGCDateStamps
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-XX:+PrintGCApplicationStoppedTime

# GC log rotation
-Xlog:gc*:gc-%t-%p.log:time,tags:filecount=5,filesize=100M

# Pre-Java 9 logging format
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-XX:+PrintGCDateStamps
-Xloggc:gc.log
```

### Analyzing GC Logs

```java
public class GCLogAnalyzer {
    public static void analyzeGCLogs() {
        // Example GC log analysis
        String sampleLogEntry = "[GC (Allocation Failure) [PSYoungGen: 262144K->43520K(307200K)] 262144K->45056K(1024000K), 0.0123456 secs]";
        
        parseGCLogEntry(sampleLogEntry);
        
        // Memory usage calculation
        long youngBefore = 262144;
        long youngAfter = 43520;
        long youngCapacity = 307200;
        
        System.out.printf("Young Generation: %dK -> %dK (capacity: %dK)%n",
            youngBefore, youngAfter, youngCapacity);
        System.out.printf("Reclaimed: %dK%n", youngBefore - youngAfter);
        System.out.printf("Utilization: %.1f%%%n", 
            (double) youngAfter / youngCapacity * 100);
    }
    
    private static void parseGCLogEntry(String logEntry) {
        // Parse GC log entry to extract metrics
        // This would typically use regex or specialized parsing libraries
        System.out.println("Parsing GC log: " + logEntry);
    }
}
```

## Common GC Issues and Solutions

### Memory Leaks

```java
public class MemoryLeakExamples {
    // Static collection causing memory leak
    private static final List<Object> STATIC_CACHE = new ArrayList<>();
    
    public static void demonstrateBadPractice() {
        // Anti-pattern: Adding objects to static collections
        for (int i = 0; i < 1000000; i++) {
            STATIC_CACHE.add(new LeakingObject(i)); // Memory leak!
        }
    }
    
    public static void demonstrateGoodPractice() {
        // Solution 1: Use weak references
        Map<String, WeakReference<Object>> weakCache = new HashMap<>();
        
        // Solution 2: Implement proper cache eviction
        Map<String, Object> boundedCache = new LinkedHashMap<String, Object>(100, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry<String, Object> eldest) {
                return size() > 1000; // Limit cache size
            }
        };
        
        // Solution 3: Clear collections periodically
        if (STATIC_CACHE.size() > 10000) {
            STATIC_CACHE.clear();
        }
    }
    
    static class LeakingObject {
        private byte[] data = new byte[1024]; // 1KB object
        private int id;
        
        public LeakingObject(int id) {
            this.id = id;
        }
    }
}
```

### GC Tuning for Different Applications

```java
public class GCTuningScenarios {
    // Batch processing application
    public static void batchProcessingGC() {
        // High throughput, occasional pauses acceptable
        // Recommended: Parallel GC
        System.setProperty("java.vm.args", "-XX:+UseParallelGC -XX:ParallelGCThreads=8");
    }
    
    // Web application
    public static void webApplicationGC() {
        // Balanced throughput and latency
        // Recommended: G1 GC
        System.setProperty("java.vm.args", "-XX:+UseG1GC -XX:MaxGCPauseMillis=200");
    }
    
    // Real-time application
    public static void realTimeGC() {
        // Ultra-low latency requirement
        // Recommended: ZGC or Shenandoah
        System.setProperty("java.vm.args", "-XX:+UseZGC -XX:+UnlockExperimentalVMOptions");
    }
    
    // Microservice
    public static void microserviceGC() {
        // Fast startup, moderate footprint
        // Recommended: G1 GC with smaller heap
        System.setProperty("java.vm.args", "-XX:+UseG1GC -Xmx1g -XX:MaxGCPauseMillis=100");
    }
}
```

## GC Performance Optimization

### Object Allocation Optimization

```java
public class AllocationOptimization {
    // Object pooling to reduce GC pressure
    public static class ObjectPool<T> {
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
            // Otherwise let GC handle it
        }
    }
    
    // Flyweight pattern for memory efficiency
    public static class CharacterFlyweight {
        private static final Map<Character, CharacterFlyweight> cache = new HashMap<>();
        
        private final char character;
        
        private CharacterFlyweight(char character) {
            this.character = character;
        }
        
        public static CharacterFlyweight valueOf(char c) {
            return cache.computeIfAbsent(c, CharacterFlyweight::new);
        }
        
        public char getCharacter() {
            return character;
        }
    }
}
```

### Memory-Efficient Coding Practices

```java
public class MemoryEfficientCoding {
    // Use primitive collections when possible
    public static void usePrimitiveCollections() {
        // Instead of List<Integer>, use int[] or specialized collections
        int[] primitiveArray = new int[1000000]; // Much more memory efficient
        
        // For maps with primitive keys/values, consider Eclipse Collections
        // org.eclipse.collections.api.map.primitive.IntObjectMap<String>
    }
    
    // String optimization
    public static void optimizeStringUsage() {
        // Use StringBuilder for concatenation
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 1000; i++) {
            sb.append("Item ").append(i).append("\n");
        }
        String result = sb.toString();
        
        // Use String.intern() judiciously for frequently used strings
        String commonValue = "STATUS_ACTIVE".intern();
    }
    
    // Lazy initialization
    public static class LazyInitialization {
        private ExpensiveObject expensiveObject;
        
        public ExpensiveObject getExpensiveObject() {
            if (expensiveObject == null) {
                expensiveObject = new ExpensiveObject();
            }
            return expensiveObject;
        }
    }
    
    static class ExpensiveObject {
        private byte[] largeData = new byte[1024 * 1024]; // 1MB
    }
}
```

Effective garbage collection tuning requires understanding your application's memory usage patterns and performance requirements. Regular monitoring and analysis of GC behavior is essential for maintaining optimal performance.
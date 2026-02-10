# Java Virtual Machine (JVM)

The Java Virtual Machine (JVM) is a virtual machine that enables a computer to run Java programs and other languages that compile to Java bytecode. It provides platform independence, automatic memory management, and robust security features.

## JVM Architecture Overview

The JVM consists of several key components that work together to execute Java applications:

```
┌─────────────────────────────────────────────┐
│              Java Application               │
├─────────────────────────────────────────────┤
│           Class Loader Subsystem            │
├─────────────────────────────────────────────┤
│         Runtime Data Areas                  │
│  ┌─────────┬─────────┬─────────┬─────────┐  │
│  │ Method  │  Heap   │  Stack  │  PC     │  │
│  │ Area    │         │         │ Register│  │
│  └─────────┴─────────┴─────────┴─────────┘  │
├─────────────────────────────────────────────┤
│           Execution Engine                  │
│  ┌─────────┬─────────┬─────────┐            │
│  │  JIT    │  GC     │  Native │            │
│  │Compiler │         │ Interface│            │
│  └─────────┴─────────┴─────────┘            │
├─────────────────────────────────────────────┤
│           Native Method Interface           │
└─────────────────────────────────────────────┘
```

## JVM Components

### 1. Class Loader Subsystem

The class loader is responsible for loading class files into the JVM.

#### Class Loading Process

```java
public class ClassLoadingProcess {
    public static void demonstrateClassLoading() {
        // Bootstrap ClassLoader - loads core Java classes
        ClassLoader bootstrap = String.class.getClassLoader();
        System.out.println("Bootstrap CL: " + bootstrap); // null
        
        // Extension ClassLoader - loads extension classes
        ClassLoader extension = sun.misc.Launcher.getLauncher()
            .getClassLoader().getParent();
        System.out.println("Extension CL: " + extension);
        
        // Application ClassLoader - loads application classes
        ClassLoader application = ClassLoadingProcess.class.getClassLoader();
        System.out.println("Application CL: " + application);
    }
}

// Custom ClassLoader example
public class CustomClassLoader extends ClassLoader {
    private String classPath;
    
    public CustomClassLoader(String classPath) {
        this.classPath = classPath;
    }
    
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        }
        return defineClass(name, classData, 0, classData.length);
    }
    
    private byte[] loadClassData(String className) {
        // Load class bytes from custom location
        // Implementation depends on storage mechanism
        return null;
    }
}
```

#### Class Loader Hierarchy

```java
public class ClassLoaderHierarchy {
    public static void showHierarchy() {
        ClassLoader current = ClassLoaderHierarchy.class.getClassLoader();
        
        System.out.println("Current ClassLoader: " + current);
        System.out.println("Parent: " + current.getParent());
        System.out.println("Grandparent: " + current.getParent().getParent());
        // Grandparent is typically null (Bootstrap ClassLoader)
    }
    
    // Delegation Model
    public static void demonstrateDelegation() {
        try {
            // When loading a class, JVM first delegates to parent
            Class<?> stringClass = ClassLoader.getSystemClassLoader()
                .loadClass("java.lang.String");
            // Goes through: App CL -> Ext CL -> Bootstrap CL
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 2. Runtime Data Areas

#### Method Area (Metaspace since Java 8)

```java
public class MethodAreaExample {
    // Class metadata stored here
    private static final String CONSTANT_STRING = "Constant Value";
    private static int staticCounter = 0;
    
    public static void demonstrateMethodArea() {
        // Runtime Constant Pool
        String s1 = "Hello";           // String literal
        String s2 = "Hello";           // Same reference due to string interning
        String s3 = new String("Hello"); // Different object
        
        System.out.println(s1 == s2);  // true - same reference
        System.out.println(s1 == s3);  // false - different objects
        
        // Class structure information
        Class<?> clazz = MethodAreaExample.class;
        System.out.println("Class name: " + clazz.getName());
        System.out.println("Methods count: " + clazz.getDeclaredMethods().length);
    }
}
```

#### Heap Memory Management

```java
public class HeapStructure {
    public static void demonstrateHeapOrganization() {
        // Young Generation
        List<String> youngObjects = new ArrayList<>();
        for (int i = 0; i < 10000; i++) {
            youngObjects.add("Object " + i); // Allocated in Eden space
        }
        
        // Promote objects to Old Generation
        List<String> oldObjects = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            String obj = createLongLivedObject(i);
            oldObjects.add(obj); // Will be promoted after surviving GCs
        }
        
        // Clear references to trigger GC
        youngObjects.clear();
        System.gc(); // Suggest garbage collection
    }
    
    private static String createLongLivedObject(int id) {
        return "Long-lived object #" + id;
    }
}
```

#### Stack Memory

```java
public class StackFrameExample {
    public static void main(String[] args) {
        int result = calculate(5, 3);
        System.out.println("Result: " + result);
    }
    
    public static int calculate(int a, int b) {
        int sum = add(a, b);
        int product = multiply(a, b);
        return sum + product;
    }
    
    public static int add(int x, int y) {
        return x + y; // Stack frame: [x=5, y=3, return_address]
    }
    
    public static int multiply(int x, int y) {
        return x * y; // Stack frame: [x=5, y=3, return_address]
    }
}
```

### 3. Execution Engine

#### Interpreter

```java
public class InterpreterExample {
    // Bytecode interpretation demonstration
    public static void interpretedMethod() {
        int a = 10;
        int b = 20;
        int c = a + b; // Bytecode: iload_1, iload_2, iadd, istore_3
        System.out.println("Sum: " + c);
    }
}
```

#### Just-In-Time (JIT) Compiler

```java
public class JITCompilation {
    public static void demonstrateJIT() {
        // HotSpot JVM identifies frequently executed code
        for (int i = 0; i < 100000; i++) {
            frequentlyCalledMethod(); // Will be JIT compiled
        }
    }
    
    // This method will likely be JIT compiled due to frequent calls
    public static void frequentlyCalledMethod() {
        String s = "HotSpot Optimization";
        s.length(); // Simple operation, good for optimization
    }
    
    // JIT compiler optimizations
    public static int optimizedCalculation(int[] array) {
        int sum = 0;
        // Loop unrolling and vectorization potential
        for (int i = 0; i < array.length; i++) {
            sum += array[i];
        }
        return sum;
    }
}
```

## JVM Startup Process

### JVM Initialization Steps

```java
public class JVMStartup {
    // Static initialization - happens during JVM startup
    static {
        System.out.println("Static block executed during class loading");
        initializeSystemProperties();
    }
    
    private static void initializeSystemProperties() {
        // JVM system properties set during startup
        System.setProperty("java.version", System.getProperty("java.version"));
        System.setProperty("java.home", System.getProperty("java.home"));
        System.setProperty("os.name", System.getProperty("os.name"));
    }
    
    public static void main(String[] args) {
        System.out.println("Main method executed");
        demonstrateStartupSequence();
    }
    
    public static void demonstrateStartupSequence() {
        // 1. Load and initialize main class
        // 2. Execute static initializers
        // 3. Call main method
        // 4. Application begins execution
        
        System.out.println("JVM startup sequence completed");
        System.out.println("Available processors: " + 
                          Runtime.getRuntime().availableProcessors());
        System.out.println("Max memory: " + 
                          Runtime.getRuntime().maxMemory() / (1024 * 1024) + " MB");
    }
}
```

## JVM Configuration Options

### Memory Settings

```bash
# Heap memory configuration
-Xms2g                    # Initial heap size: 2GB
-Xmx4g                    # Maximum heap size: 4GB
-XX:NewRatio=3            # Old:Young generation ratio = 3:1
-XX:SurvivorRatio=8       # Eden:Survivor space ratio = 8:1

# Metaspace configuration
-XX:MetaspaceSize=256m    # Initial metaspace size
-XX:MaxMetaspaceSize=512m # Maximum metaspace size
-XX:+UseCompressedOops    # Compressed object pointers (64-bit JVMs)

# Stack configuration
-Xss1m                    # Thread stack size: 1MB
```

### Garbage Collection Settings

```bash
# Serial GC (single-threaded)
-XX:+UseSerialGC

# Parallel GC (multi-threaded throughput)
-XX:+UseParallelGC
-XX:ParallelGCThreads=4

# G1 GC (low pause collector)
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200

# ZGC (ultra-low latency)
-XX:+UseZGC
-XX:+UnlockExperimentalVMOptions

# Shenandoah GC
-XX:+UseShenandoahGC
-XX:+UnlockExperimentalVMOptions
```

### JIT Compiler Settings

```bash
# Compilation thresholds
-XX:CompileThreshold=10000     # Methods compiled after 10000 invocations
-XX:TieredStopAtLevel=4        # Use tiered compilation (C1 + C2)

# Compiler optimizations
-XX:+AggressiveOpts           # Enable aggressive optimizations
-XX:+OptimizeStringConcat     # Optimize string concatenation
-XX:+UseStringDeduplication   # Deduplicate identical strings

# Compilation logging
-XX:+PrintCompilation         # Print compilation information
-XX:+LogCompilation           # Log detailed compilation data
```

## JVM Monitoring and Profiling

### Built-in Monitoring Tools

```java
import java.lang.management.*;
import javax.management.*;

public class JVMMonitoring {
    public static void monitorJVMMetrics() {
        // Memory management
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        System.out.printf("Heap: %d/%d MB%n", 
            heapUsage.getUsed() / (1024 * 1024),
            heapUsage.getMax() / (1024 * 1024));
        
        // Garbage collection
        List<GarbageCollectorMXBean> gcBeans = 
            ManagementFactory.getGarbageCollectorMXBeans();
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.printf("GC %s: %d collections, %d ms%n",
                gcBean.getName(),
                gcBean.getCollectionCount(),
                gcBean.getCollectionTime());
        }
        
        // Thread information
        ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();
        System.out.println("Active threads: " + threadBean.getThreadCount());
        System.out.println("Peak threads: " + threadBean.getPeakThreadCount());
        
        // Operating system metrics
        OperatingSystemMXBean osBean = 
            ManagementFactory.getOperatingSystemMXBean();
        System.out.printf("CPU load: %.2f%%%n", 
            osBean.getSystemLoadAverage() * 100);
        System.out.println("Available processors: " + 
                          osBean.getAvailableProcessors());
    }
    
    public static void setupJMXMonitoring() throws Exception {
        MBeanServer server = ManagementFactory.getPlatformMBeanServer();
        
        // Register custom MBeans for application monitoring
        ObjectName name = new ObjectName("com.example:type=ApplicationMonitor");
        ApplicationMonitor monitor = new ApplicationMonitor();
        server.registerMBean(monitor, name);
    }
}

// Custom MBean for application monitoring
public interface ApplicationMonitorMBean {
    int getActiveSessions();
    long getTotalRequests();
    double getAverageResponseTime();
}

public class ApplicationMonitor implements ApplicationMonitorMBean {
    private int activeSessions = 0;
    private long totalRequests = 0;
    private double avgResponseTime = 0.0;
    
    @Override
    public int getActiveSessions() { return activeSessions; }
    @Override
    public long getTotalRequests() { return totalRequests; }
    @Override
    public double getAverageResponseTime() { return avgResponseTime; }
    
    // Methods to update metrics
    public void incrementSessions() { activeSessions++; }
    public void addRequest(long responseTime) {
        totalRequests++;
        avgResponseTime = (avgResponseTime * (totalRequests - 1) + responseTime) / totalRequests;
    }
}
```

### External Monitoring Tools

```bash
# JConsole - built-in GUI monitoring tool
jconsole

# VisualVM - enhanced monitoring and profiling
jvisualvm

# JMC (Java Mission Control) - advanced profiling
jmc

# Command-line monitoring
jstat -gc <pid> 1s          # GC statistics every second
jstack <pid>               # Thread dump
jmap -heap <pid>           # Heap summary
jcmd <pid> VM.flags        # JVM flags
```

## JVM Performance Tuning

### Garbage Collection Optimization

```java
public class GCOptimization {
    public static void optimizeGC() {
        // Minimize object allocation
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 1000; i++) {
            sb.append("iteration ").append(i).append("\n");
        }
        String result = sb.toString();
        
        // Use object pools for frequently created objects
        ObjectPool<StringBuilder> pool = new ObjectPool<>(StringBuilder::new, 50);
        StringBuilder pooledBuilder = pool.acquire();
        // Use pooledBuilder
        pool.release(pooledBuilder);
        
        // Choose appropriate GC algorithm based on requirements
        // Low latency: ZGC, Shenandoah
        // High throughput: Parallel GC
        // Balanced: G1 GC
    }
}
```

### Memory Leak Prevention

```java
public class MemoryLeakPrevention {
    // Use weak references for caches
    private Map<String, WeakReference<ExpensiveObject>> cache = 
        new ConcurrentHashMap<>();
    
    // Properly close resources
    public void handleResources() {
        try (FileInputStream fis = new FileInputStream("data.txt");
             BufferedReader reader = new BufferedReader(
                 new InputStreamReader(fis))) {
            // Process file
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    // Clean up listeners and callbacks
    public void cleanupListeners() {
        // Remove listeners when no longer needed
        eventSource.removeListener(listener);
    }
}
```

The JVM is a sophisticated runtime environment that provides the foundation for Java applications. Understanding its architecture and configuration options is crucial for developing high-performance Java applications.
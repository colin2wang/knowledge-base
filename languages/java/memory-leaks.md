# Memory Leaks in Java

Memory leaks in Java occur when objects that are no longer needed remain referenced in memory, preventing the garbage collector from reclaiming them. Unlike languages with manual memory management, Java memory leaks are typically caused by unintentional object retention rather than failing to deallocate memory.

## What Are Memory Leaks?

A memory leak occurs when an application continuously consumes memory without releasing it, eventually leading to `OutOfMemoryError`. In Java, this happens when objects remain reachable from GC roots despite being logically unused.

### Basic Memory Leak Example

```java
public class BasicMemoryLeak {
    // Static collection that accumulates objects indefinitely
    private static final List<Object> LEAKY_LIST = new ArrayList<>();
    
    public static void demonstrateBasicLeak() {
        // Anti-pattern: Adding objects to static collections without removal
        for (int i = 0; i < 1000000; i++) {
            LEAKY_LIST.add(new LeakingObject(i)); // Memory leak!
        }
        
        System.out.println("Leaked " + LEAKY_LIST.size() + " objects");
        // These objects will never be garbage collected
    }
    
    static class LeakingObject {
        private byte[] data = new byte[1024]; // 1KB each
        private int id;
        
        public LeakingObject(int id) {
            this.id = id;
        }
    }
}
```

## Common Causes of Memory Leaks

### 1. Static Collections

```java
public class StaticCollectionLeaks {
    // Cache that grows indefinitely
    private static final Map<String, Object> CACHE = new HashMap<>();
    
    // Session storage that never cleans up
    private static final Map<String, UserSession> USER_SESSIONS = new HashMap<>();
    
    // Listener registry that accumulates registrations
    private static final List<EventListener> LISTENERS = new ArrayList<>();
    
    public static void demonstrateStaticLeaks() {
        // Growing cache without eviction policy
        for (int i = 0; i < 100000; i++) {
            CACHE.put("key" + i, new ExpensiveObject());
        }
        
        // Accumulating user sessions
        for (int i = 0; i < 50000; i++) {
            USER_SESSIONS.put("session" + i, new UserSession(i));
        }
        
        // Accumulating event listeners
        for (int i = 0; i < 10000; i++) {
            LISTENERS.add(new EventListener());
        }
    }
    
    static class ExpensiveObject {
        private byte[] data = new byte[1024 * 10]; // 10KB each
    }
    
    static class UserSession {
        private int userId;
        private long loginTime;
        private List<String> activities;
        
        public UserSession(int userId) {
            this.userId = userId;
            this.loginTime = System.currentTimeMillis();
            this.activities = new ArrayList<>();
        }
    }
    
    static class EventListener {
        private String handlerName;
        private Object context;
        
        public EventListener() {
            this.handlerName = "Handler_" + System.nanoTime();
            this.context = new Object(); // Captures context
        }
    }
}
```

### 2. Unclosed Resources

```java
public class ResourceLeak {
    public static void demonstrateResourceLeaks() {
        // File stream leak
        FileInputStream fileStream = null;
        try {
            fileStream = new FileInputStream("large-data-file.dat");
            // Process file...
            // Forgot to close - resource leak!
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        // fileStream remains open, holding system resources
        
        // Database connection leak
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test");
            // Execute queries...
            // Forgot to close connection - connection leak!
        } catch (SQLException e) {
            e.printStackTrace();
        }
        // Connection remains open in pool
        
        // Network socket leak
        Socket socket = null;
        try {
            socket = new Socket("example.com", 80);
            // Communicate...
            // Forgot to close socket
        } catch (IOException e) {
            e.printStackTrace();
        }
        // Socket remains open
    }
    
    // Proper resource management using try-with-resources
    public static void properResourceManagement() {
        // File handling with automatic resource management
        try (FileInputStream fis = new FileInputStream("data.txt");
             BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
            String line;
            while ((line = reader.readLine()) != null) {
                // Process line
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // Resources automatically closed
        
        // Database connection with try-with-resources
        String url = "jdbc:mysql://localhost:3306/test";
        try (Connection conn = DriverManager.getConnection(url);
             PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users")) {
            ResultSet rs = stmt.executeQuery();
            while (rs.next()) {
                // Process results
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        // Connection automatically returned to pool
    }
}
```

### 3. Listener and Callback Leaks

```java
public class ListenerLeak {
    // Event publisher that accumulates listeners
    private static final EventEmitter EVENT_EMITTER = new EventEmitter();
    
    public static void demonstrateListenerLeaks() {
        // Registering listeners without unregistering
        for (int i = 0; i < 10000; i++) {
            DataProcessor processor = new DataProcessor(i);
            EVENT_EMITTER.addListener(processor::process); // Lambda captures processor
            // processor reference held by event emitter
        }
        
        // Observer pattern leak
        Subject subject = new Subject();
        for (int i = 0; i < 5000; i++) {
            Observer observer = new Observer(i);
            subject.addObserver(observer);
            // observer reference held indefinitely
        }
    }
    
    static class EventEmitter {
        private final List<Runnable> listeners = new ArrayList<>();
        
        public void addListener(Runnable listener) {
            listeners.add(listener);
        }
        
        public void emit() {
            listeners.forEach(Runnable::run);
        }
        
        // Missing: removeListener method
    }
    
    static class DataProcessor {
        private int id;
        private List<String> processedData;
        
        public DataProcessor(int id) {
            this.id = id;
            this.processedData = new ArrayList<>();
        }
        
        public void process() {
            processedData.add("Processed by " + id);
        }
    }
    
    static class Subject {
        private final List<Observer> observers = new ArrayList<>();
        
        public void addObserver(Observer observer) {
            observers.add(observer);
        }
        
        public void notifyObservers() {
            observers.forEach(Observer::update);
        }
        
        // Missing: removeObserver method
    }
    
    static class Observer {
        private int id;
        private String state;
        
        public Observer(int id) {
            this.id = id;
        }
        
        public void update() {
            state = "Updated at " + System.currentTimeMillis();
        }
    }
    
    // Proper listener management
    public static void properListenerManagement() {
        EventEmitter emitter = new ProperEventEmitter();
        
        DataProcessor processor = new DataProcessor(1);
        Registration registration = emitter.addListener(processor::process);
        
        // Use the listener
        emitter.emit();
        
        // Properly remove when done
        registration.remove();
    }
    
    static class ProperEventEmitter {
        private final Map<Runnable, Registration> listeners = new HashMap<>();
        
        public Registration addListener(Runnable listener) {
            listeners.put(listener, new Registration(this, listener));
            return listeners.get(listener);
        }
        
        public void emit() {
            listeners.keySet().forEach(Runnable::run);
        }
        
        private void removeListener(Runnable listener) {
            listeners.remove(listener);
        }
    }
    
    static class Registration {
        private final ProperEventEmitter emitter;
        private final Runnable listener;
        private boolean removed = false;
        
        public Registration(ProperEventEmitter emitter, Runnable listener) {
            this.emitter = emitter;
            this.listener = listener;
        }
        
        public void remove() {
            if (!removed) {
                emitter.removeListener(listener);
                removed = true;
            }
        }
    }
}
```

### 4. Thread Local Leaks

```java
public class ThreadLocalLeak {
    // ThreadLocal that can cause class loader leaks
    private static final ThreadLocal<List<String>> THREAD_LOCAL_DATA = 
        new ThreadLocal<List<String>>() {
            @Override
            protected List<String> initialValue() {
                return new ArrayList<>();
            }
        };
    
    // In web applications, this can prevent class loader garbage collection
    private static final ThreadLocal<ApplicationContext> CONTEXT_HOLDER = 
        new ThreadLocal<>();
    
    public static void demonstrateThreadLocalLeaks() {
        // Worker threads accumulating data
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                // Each thread accumulates data in ThreadLocal
                List<String> data = THREAD_LOCAL_DATA.get();
                for (int j = 0; j < 1000; j++) {
                    data.add("Data_" + j);
                }
                // Data never cleaned up when thread completes
            });
        }
        
        executor.shutdown();
    }
    
    // Proper ThreadLocal usage with cleanup
    public static void properThreadLocalUsage() {
        ThreadLocal<List<String>> safeThreadLocal = new ThreadLocal<>();
        
        try {
            // Set thread local data
            safeThreadLocal.set(new ArrayList<>());
            
            // Use the data
            List<String> data = safeThreadLocal.get();
            data.add("Important data");
            
            // Process data...
            
        } finally {
            // Always clean up ThreadLocal
            safeThreadLocal.remove();
        }
    }
    
    // Web application context holder with proper cleanup
    public static class WebContextHolder {
        private static final ThreadLocal<HttpServletRequest> REQUEST_HOLDER = 
            new ThreadLocal<>();
        
        public static void setRequest(HttpServletRequest request) {
            REQUEST_HOLDER.set(request);
        }
        
        public static HttpServletRequest getRequest() {
            return REQUEST_HOLDER.get();
        }
        
        public static void clear() {
            REQUEST_HOLDER.remove(); // Critical for preventing leaks
        }
    }
}
```

## Detection and Monitoring

### Memory Leak Detection Tools

```java
import java.lang.management.*;
import java.util.List;

public class MemoryLeakDetector {
    private static final long MONITORING_INTERVAL = 5000; // 5 seconds
    
    public static void startMonitoring() {
        Timer timer = new Timer(true);
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                checkForMemoryLeaks();
            }
        }, 0, MONITORING_INTERVAL);
    }
    
    private static void checkForMemoryLeaks() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        
        long usedMemory = heapUsage.getUsed();
        long maxMemory = heapUsage.getMax();
        double usagePercentage = (double) usedMemory / maxMemory * 100;
        
        System.out.printf("Heap usage: %.1f%% (%d MB / %d MB)%n",
            usagePercentage, usedMemory / (1024 * 1024), maxMemory / (1024 * 1024));
        
        // Alert if memory usage is concerning
        if (usagePercentage > 85) {
            System.err.println("High memory usage detected!");
            analyzeMemoryUsage();
        }
    }
    
    private static void analyzeMemoryUsage() {
        // Get memory pool information
        List<MemoryPoolMXBean> pools = ManagementFactory.getMemoryPoolMXBeans();
        
        for (MemoryPoolMXBean pool : pools) {
            MemoryUsage usage = pool.getUsage();
            if (usage.getUsed() > 0) {
                System.out.printf("Pool %s: %d MB used%n",
                    pool.getName(), usage.getUsed() / (1024 * 1024));
            }
        }
        
        // Force garbage collection for analysis
        System.gc();
        
        // Check for potential leaks
        checkStaticCollections();
    }
    
    private static void checkStaticCollections() {
        // Analyze static collections for unusual growth
        // This would typically use reflection or specialized tools
        System.out.println("Analyzing static collections...");
    }
}
```

### Heap Dump Analysis

```java
public class HeapDumpAnalysis {
    public static void generateHeapDump() {
        try {
            // Generate heap dump programmatically
            HotSpotDiagnosticMXBean mxBean = ManagementFactory
                .newPlatformMXBeanProxy(ManagementFactory.getPlatformMBeanServer(),
                                      "com.sun.management:type=HotSpotDiagnostic",
                                      HotSpotDiagnosticMXBean.class);
            
            String fileName = "heap_dump_" + System.currentTimeMillis() + ".hprof";
            mxBean.dumpHeap(fileName, true);
            System.out.println("Heap dump generated: " + fileName);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    // Analyze heap dump programmatically (simplified example)
    public static void analyzeHeapDump(String dumpFile) {
        System.out.println("Analyzing heap dump: " + dumpFile);
        // In practice, use tools like Eclipse MAT, JVisualVM, or YourKit
        // This would involve parsing the HPROF format
    }
    
    // Memory leak detection in running application
    public static void detectLeaksInRuntime() {
        // Monitor for growing object counts
        Map<Class<?>, Integer> objectCounts = new HashMap<>();
        
        // Sample object counts periodically
        Timer timer = new Timer(true);
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                sampleObjectCounts(objectCounts);
            }
        }, 0, 30000); // Every 30 seconds
    }
    
    private static void sampleObjectCounts(Map<Class<?>, Integer> baseline) {
        // This is a simplified example - real implementation would use
        // instrumentation or profiling APIs
        System.out.println("Sampling object counts...");
    }
}
```

## Prevention Strategies

### 1. Proper Collection Management

```java
public class CollectionLeakPrevention {
    // Bounded cache with eviction policy
    private final Map<String, SoftReference<CachedObject>> boundedCache = 
        new LinkedHashMap<String, SoftReference<CachedObject>>(100, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(
                    Map.Entry<String, SoftReference<CachedObject>> eldest) {
                return size() > 1000; // Limit cache size
            }
        };
    
    // WeakHashMap for automatic cleanup
    private final Map<KeyObject, CachedValue> weakCache = new WeakHashMap<>();
    
    // Proper session management
    private final Map<String, UserSession> sessions = new ConcurrentHashMap<>();
    private final ScheduledExecutorService cleanupExecutor = 
        Executors.newScheduledThreadPool(1);
    
    public CollectionLeakPrevention() {
        // Schedule periodic cleanup
        cleanupExecutor.scheduleAtFixedRate(this::cleanupExpiredSessions, 
                                          30, 30, TimeUnit.MINUTES);
    }
    
    public void addToCache(String key, CachedObject value) {
        // Use soft references for cache entries
        boundedCache.put(key, new SoftReference<>(value));
    }
    
    public CachedObject getFromCache(String key) {
        SoftReference<CachedObject> ref = boundedCache.get(key);
        return ref != null ? ref.get() : null;
    }
    
    public void addUserSession(String sessionId, UserSession session) {
        sessions.put(sessionId, session);
    }
    
    public void removeUserSession(String sessionId) {
        sessions.remove(sessionId);
    }
    
    private void cleanupExpiredSessions() {
        long currentTime = System.currentTimeMillis();
        sessions.entrySet().removeIf(entry -> 
            entry.getValue().getLastAccessTime() < currentTime - 30 * 60 * 1000);
    }
    
    // Closeable cache implementation
    public static class ManagedCache<K, V> implements AutoCloseable {
        private final Map<K, V> cache = new ConcurrentHashMap<>();
        private final int maxSize;
        private final ScheduledExecutorService cleanupExecutor;
        
        public ManagedCache(int maxSize, long cleanupInterval, TimeUnit timeUnit) {
            this.maxSize = maxSize;
            this.cleanupExecutor = Executors.newScheduledThreadPool(1);
            this.cleanupExecutor.scheduleAtFixedRate(
                this::enforceSizeLimit, cleanupInterval, cleanupInterval, timeUnit);
        }
        
        public void put(K key, V value) {
            if (cache.size() >= maxSize) {
                evictOldestEntry();
            }
            cache.put(key, value);
        }
        
        public V get(K key) {
            return cache.get(key);
        }
        
        private void evictOldestEntry() {
            // Simple FIFO eviction - could be enhanced with LRU
            K oldestKey = cache.keySet().iterator().next();
            cache.remove(oldestKey);
        }
        
        private void enforceSizeLimit() {
            while (cache.size() > maxSize) {
                evictOldestEntry();
            }
        }
        
        @Override
        public void close() {
            cleanupExecutor.shutdown();
            cache.clear();
        }
    }
    
    static class CachedObject {
        private byte[] data;
        private long createTime;
        
        public CachedObject(int size) {
            this.data = new byte[size];
            this.createTime = System.currentTimeMillis();
        }
    }
    
    static class KeyObject {
        private String id;
        
        public KeyObject(String id) {
            this.id = id;
        }
        
        @Override
        public int hashCode() {
            return id.hashCode();
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            KeyObject keyObject = (KeyObject) obj;
            return id.equals(keyObject.id);
        }
    }
    
    static class CachedValue {
        private String data;
    }
    
    static class UserSession {
        private String userId;
        private long lastAccessTime;
        
        public UserSession(String userId) {
            this.userId = userId;
            this.lastAccessTime = System.currentTimeMillis();
        }
        
        public long getLastAccessTime() {
            return lastAccessTime;
        }
        
        public void updateLastAccess() {
            this.lastAccessTime = System.currentTimeMillis();
        }
    }
}
```

### 2. Resource Management Patterns

```java
public class ResourceManager {
    // Resource pool pattern
    public static class ResourcePool<T extends AutoCloseable> {
        private final Queue<T> available = new ConcurrentLinkedQueue<>();
        private final Queue<T> inUse = new ConcurrentLinkedQueue<>();
        private final Supplier<T> factory;
        private final int maxSize;
        
        public ResourcePool(Supplier<T> factory, int maxSize) {
            this.factory = factory;
            this.maxSize = maxSize;
        }
        
        public T acquire() throws Exception {
            T resource = available.poll();
            if (resource == null && inUse.size() < maxSize) {
                resource = factory.get();
            }
            if (resource != null) {
                inUse.offer(resource);
            }
            return resource;
        }
        
        public void release(T resource) {
            if (inUse.remove(resource)) {
                available.offer(resource);
            }
        }
        
        public void close() {
            available.forEach(this::closeResource);
            inUse.forEach(this::closeResource);
            available.clear();
            inUse.clear();
        }
        
        private void closeResource(T resource) {
            try {
                resource.close();
            } catch (Exception e) {
                System.err.println("Error closing resource: " + e.getMessage());
            }
        }
    }
    
    // Try-with-resources for automatic cleanup
    public static void automaticResourceManagement() {
        // Database connection with proper cleanup
        try (Connection conn = DriverManager.getConnection("jdbc:h2:mem:test");
             PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users")) {
            
            ResultSet rs = stmt.executeQuery();
            while (rs.next()) {
                // Process results
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        // File processing with automatic cleanup
        try (Stream<String> lines = Files.lines(Paths.get("data.txt"))) {
            lines.filter(line -> !line.isEmpty())
                 .map(String::trim)
                 .forEach(System.out::println);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    // Custom resource wrapper
    public static class ManagedResource<T> implements AutoCloseable {
        private final T resource;
        private final Consumer<T> cleanupAction;
        private boolean closed = false;
        
        public ManagedResource(T resource, Consumer<T> cleanupAction) {
            this.resource = resource;
            this.cleanupAction = cleanupAction;
        }
        
        public T getResource() {
            if (closed) {
                throw new IllegalStateException("Resource already closed");
            }
            return resource;
        }
        
        @Override
        public void close() {
            if (!closed) {
                cleanupAction.accept(resource);
                closed = true;
            }
        }
    }
    
    // Usage example
    public static void useManagedResource() {
        try (ManagedResource<FileInputStream> managedStream = 
                new ManagedResource<>(
                    new FileInputStream("data.txt"),
                    stream -> {
                        try {
                            stream.close();
                            System.out.println("Stream closed automatically");
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    })) {
            
            FileInputStream stream = managedStream.getResource();
            // Use the stream...
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 3. Listener Management Best Practices

```java
public class ListenerManagement {
    // Weak listener pattern
    public static class WeakListenerManager<T> {
        private final Map<Object, WeakReference<T>> listeners = new ConcurrentHashMap<>();
        
        public void addListener(Object key, T listener) {
            listeners.put(key, new WeakReference<>(listener));
        }
        
        public void removeListener(Object key) {
            listeners.remove(key);
        }
        
        public void notifyListeners(Consumer<T> action) {
            listeners.entrySet().removeIf(entry -> {
                T listener = entry.getValue().get();
                if (listener != null) {
                    try {
                        action.accept(listener);
                    } catch (Exception e) {
                        System.err.println("Error notifying listener: " + e.getMessage());
                    }
                    return false; // Keep valid listeners
                }
                return true; // Remove garbage collected listeners
            });
        }
    }
    
    // Registration-based listener management
    public static class RegistrationBasedEventManager {
        private final Map<EventListener, Registration> registrations = 
            new ConcurrentHashMap<>();
        
        public Registration addListener(EventListener listener) {
            Registration registration = new Registration(this, listener);
            registrations.put(listener, registration);
            return registration;
        }
        
        public void removeListener(EventListener listener) {
            Registration registration = registrations.remove(listener);
            if (registration != null) {
                registration.invalidate();
            }
        }
        
        public void fireEvent(Event event) {
            registrations.keySet().forEach(listener -> {
                try {
                    listener.handleEvent(event);
                } catch (Exception e) {
                    System.err.println("Error handling event: " + e.getMessage());
                }
            });
        }
        
        public void cleanup() {
            registrations.values().forEach(Registration::invalidate);
            registrations.clear();
        }
    }
    
    public interface EventListener {
        void handleEvent(Event event);
    }
    
    public static class Event {
        private final String type;
        private final Object data;
        
        public Event(String type, Object data) {
            this.type = type;
            this.data = data;
        }
        
        public String getType() { return type; }
        public Object getData() { return data; }
    }
    
    public static class Registration {
        private final RegistrationBasedEventManager manager;
        private final EventListener listener;
        private volatile boolean valid = true;
        
        public Registration(RegistrationBasedEventManager manager, EventListener listener) {
            this.manager = manager;
            this.listener = listener;
        }
        
        public void remove() {
            if (valid) {
                manager.removeListener(listener);
            }
        }
        
        private void invalidate() {
            valid = false;
        }
        
        public boolean isValid() {
            return valid;
        }
    }
    
    // Usage example
    public static void demonstrateProperListenerManagement() {
        RegistrationBasedEventManager eventManager = new RegistrationBasedEventManager();
        
        EventListener listener = event -> 
            System.out.println("Received event: " + event.getType());
        
        // Register listener
        Registration registration = eventManager.addListener(listener);
        
        // Fire events
        eventManager.fireEvent(new Event("test", "data"));
        
        // Proper cleanup
        registration.remove();
        
        // Or cleanup entire manager
        eventManager.cleanup();
    }
}
```

Memory leak prevention requires careful attention to object lifecycle management, proper resource handling, and implementing appropriate cleanup mechanisms. Regular monitoring and analysis are essential for early detection and resolution of memory issues.
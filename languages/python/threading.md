# Python Threading

Python's threading module provides a way to run multiple threads (lightweight processes) within a single program. Threading is particularly useful for I/O-bound tasks where operations spend time waiting for external resources.

## Introduction to Threading

Threading allows concurrent execution of code within the same process, sharing the same memory space. Unlike multiprocessing, threads share data more easily but are limited by Python's Global Interpreter Lock (GIL).

## Basic Thread Creation

### Using Thread Class

```python
import threading
import time

def worker_function(name, delay):
    """Worker function that simulates I/O-bound task"""
    print(f"Thread {name} starting")
    time.sleep(delay)
    print(f"Thread {name} finished")

# Create and start threads
thread1 = threading.Thread(target=worker_function, args=("Worker-1", 2))
thread2 = threading.Thread(target=worker_function, args=("Worker-2", 3))

thread1.start()
thread2.start()

# Wait for threads to complete
thread1.join()
thread2.join()

print("All threads completed")
```

### Using Thread Subclass

```python
import threading
import time

class WorkerThread(threading.Thread):
    def __init__(self, name, delay):
        super().__init__()
        self.name = name
        self.delay = delay
    
    def run(self):
        """Override run method with thread logic"""
        print(f"Thread {self.name} starting")
        time.sleep(self.delay)
        print(f"Thread {self.name} finished")

# Create and start worker threads
worker1 = WorkerThread("Custom-1", 1)
worker2 = WorkerThread("Custom-2", 2)

worker1.start()
worker2.start()

worker1.join()
worker2.join()
```

## Thread Synchronization

### Locks

```python
import threading
import time

# Shared resource
counter = 0
lock = threading.Lock()

def increment_counter(name):
    global counter
    for i in range(5):
        # Acquire lock before accessing shared resource
        with lock:
            temp = counter
            time.sleep(0.1)  # Simulate processing time
            counter = temp + 1
            print(f"{name}: counter = {counter}")

# Create threads that modify shared resource
thread1 = threading.Thread(target=increment_counter, args=("Thread-1",))
thread2 = threading.Thread(target=increment_counter, args=("Thread-2",))

thread1.start()
thread2.start()
thread1.join()
thread2.join()

print(f"Final counter value: {counter}")
```

### RLock (Reentrant Lock)

```python
import threading

class Counter:
    def __init__(self):
        self.value = 0
        self.lock = threading.RLock()
    
    def increment(self):
        with self.lock:
            self.value += 1
            self.double_increment()  # Can acquire same lock again
    
    def double_increment(self):
        with self.lock:  # RLock allows reacquisition
            self.value += 1

counter = Counter()
threads = []

for i in range(10):
    thread = threading.Thread(target=counter.increment)
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

print(f"Final value: {counter.value}")  # Should be 20
```

### Condition Variables

```python
import threading
import time
import random

class ProducerConsumer:
    def __init__(self):
        self.items = []
        self.condition = threading.Condition()
        self.max_items = 5
    
    def producer(self, name):
        for i in range(10):
            with self.condition:
                while len(self.items) >= self.max_items:
                    print(f"Producer {name} waiting...")
                    self.condition.wait()  # Wait for space
                
                item = f"Item-{i}"
                self.items.append(item)
                print(f"Producer {name} produced {item}")
                self.condition.notify_all()  # Notify consumers
            
            time.sleep(random.uniform(0.1, 0.5))
    
    def consumer(self, name):
        for i in range(10):
            with self.condition:
                while len(self.items) == 0:
                    print(f"Consumer {name} waiting...")
                    self.condition.wait()  # Wait for items
                
                item = self.items.pop(0)
                print(f"Consumer {name} consumed {item}")
                self.condition.notify_all()  # Notify producers
            
            time.sleep(random.uniform(0.1, 0.5))

# Create producer-consumer system
pc = ProducerConsumer()

# Start threads
producer_thread = threading.Thread(target=pc.producer, args=("P1",))
consumer_thread1 = threading.Thread(target=pc.consumer, args=("C1",))
consumer_thread2 = threading.Thread(target=pc.consumer, args=("C2",))

producer_thread.start()
consumer_thread1.start()
consumer_thread2.start()

producer_thread.join()
consumer_thread1.join()
consumer_thread2.join()
```

## Thread Communication

### Queues

```python
import threading
import queue
import time
import random

def producer(queue_obj, name):
    for i in range(5):
        item = f"Product-{i}"
        queue_obj.put(item)
        print(f"Producer {name} put {item}")
        time.sleep(random.uniform(0.1, 0.3))

def consumer(queue_obj, name):
    while True:
        try:
            item = queue_obj.get(timeout=1)
            if item is None:  # Poison pill
                queue_obj.task_done()
                break
            print(f"Consumer {name} got {item}")
            time.sleep(random.uniform(0.2, 0.4))
            queue_obj.task_done()
        except queue.Empty:
            break

# Create queue
work_queue = queue.Queue(maxsize=3)

# Start threads
producer_threads = [
    threading.Thread(target=producer, args=(work_queue, f"P{i}"))
    for i in range(2)
]

consumer_threads = [
    threading.Thread(target=consumer, args=(work_queue, f"C{i}"))
    for i in range(3)
]

# Start all threads
for t in producer_threads + consumer_threads:
    t.start()

# Wait for producers to finish
for t in producer_threads:
    t.join()

# Send poison pills to stop consumers
for i in range(len(consumer_threads)):
    work_queue.put(None)

# Wait for consumers to finish
for t in consumer_threads:
    t.join()

print("All work completed")
```

## Daemon Threads

```python
import threading
import time

def background_task():
    while True:
        print("Background task running...")
        time.sleep(1)

# Create daemon thread
daemon_thread = threading.Thread(target=background_task)
daemon_thread.daemon = True  # Dies when main program exits
daemon_thread.start()

# Main program continues
print("Main program doing other work...")
time.sleep(5)
print("Main program ending...")

# Daemon thread automatically terminates when main program ends
```

## Thread Local Storage

```python
import threading
import time

# Thread-local data
thread_local_data = threading.local()

def process_data(thread_name):
    # Each thread has its own copy of thread_local_data
    thread_local_data.value = f"Data for {thread_name}"
    time.sleep(1)
    print(f"{thread_name}: {thread_local_data.value}")

# Create threads
threads = []
for i in range(3):
    thread = threading.Thread(target=process_data, args=(f"Thread-{i}",))
    threads.append(thread)
    thread.start()

# Wait for all threads
for thread in threads:
    thread.join()
```

## ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor
import time
import requests

def fetch_url(url):
    """Simulate I/O-bound task"""
    response = requests.get(url)
    return f"{url}: {len(response.content)} bytes"

urls = [
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/2",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/3"
]

# Using ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=3) as executor:
    # Submit tasks
    futures = [executor.submit(fetch_url, url) for url in urls]
    
    # Get results
    for future in futures:
        try:
            result = future.result(timeout=5)
            print(result)
        except Exception as e:
            print(f"Task failed: {e}")

print("All tasks completed")
```

## Common Threading Patterns

### Worker Pool Pattern

```python
import threading
import queue
import time

class WorkerPool:
    def __init__(self, num_workers):
        self.tasks = queue.Queue()
        self.workers = []
        self.shutdown_flag = threading.Event()
        
        # Create worker threads
        for i in range(num_workers):
            worker = threading.Thread(target=self.worker, args=(i,))
            worker.daemon = True
            worker.start()
            self.workers.append(worker)
    
    def worker(self, worker_id):
        while not self.shutdown_flag.is_set():
            try:
                task = self.tasks.get(timeout=1)
                if task is None:
                    break
                # Execute task
                task()
                self.tasks.task_done()
            except queue.Empty:
                continue
    
    def submit(self, task):
        if not self.shutdown_flag.is_set():
            self.tasks.put(task)
    
    def shutdown(self):
        self.shutdown_flag.set()
        # Wait for all tasks to complete
        self.tasks.join()
        # Stop workers
        for _ in self.workers:
            self.tasks.put(None)
        for worker in self.workers:
            worker.join()

# Usage example
def sample_task():
    print(f"Executing task in {threading.current_thread().name}")
    time.sleep(0.5)

pool = WorkerPool(num_workers=3)

# Submit tasks
for i in range(10):
    pool.submit(sample_task)

# Shutdown gracefully
pool.shutdown()
print("Worker pool shutdown complete")
```

## Best Practices

### When to Use Threading

✅ **Good for:**
- I/O-bound operations (file operations, network requests)
- GUI applications (keeping UI responsive)
- Producer-consumer patterns
- Simple concurrent tasks

❌ **Avoid for:**
- CPU-intensive computations (due to GIL)
- Tasks requiring true parallelism
- Complex shared state management

### Performance Considerations

```python
import threading
import time
from functools import wraps

def timing_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timing_decorator
def io_bound_task():
    """Simulate I/O-bound operation"""
    time.sleep(1)
    return "IO task completed"

@timing_decorator  
def cpu_bound_task():
    """Simulate CPU-bound operation"""
    total = 0
    for i in range(1000000):
        total += i * i
    return total

# Compare sequential vs threaded execution
if __name__ == "__main__":
    # Sequential execution
    print("Sequential execution:")
    io_bound_task()
    io_bound_task()
    
    # Threaded execution
    print("\nThreaded execution:")
    thread1 = threading.Thread(target=io_bound_task)
    thread2 = threading.Thread(target=io_bound_task)
    
    start = time.time()
    thread1.start()
    thread2.start()
    thread1.join()
    thread2.join()
    end = time.time()
    print(f"Threaded took {end - start:.4f} seconds")
```

Threading is excellent for I/O-bound tasks in Python, but remember the limitations imposed by the GIL for CPU-bound operations.
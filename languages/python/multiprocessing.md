# Python Multiprocessing

Python's multiprocessing module provides true parallelism by creating separate processes, each with its own Python interpreter and memory space. This bypasses the Global Interpreter Lock (GIL) limitation and is ideal for CPU-bound tasks.

## Introduction to Multiprocessing

Unlike threading, multiprocessing creates separate processes that run in parallel on multiple CPU cores. Each process has its own memory space, eliminating the need for locks in many scenarios but requiring explicit communication mechanisms.

## Basic Process Creation

### Using Process Class

```python
import multiprocessing
import time
import os

def worker_function(name, duration):
    """Worker function for CPU-intensive task"""
    print(f"Process {name} (PID: {os.getpid()}) starting")
    # CPU-intensive calculation
    result = sum(i * i for i in range(1000000))
    time.sleep(duration)
    print(f"Process {name} finished with result: {result}")

if __name__ == "__main__":
    # Create processes
    process1 = multiprocessing.Process(target=worker_function, args=("Worker-1", 1))
    process2 = multiprocessing.Process(target=worker_function, args=("Worker-2", 2))
    
    # Start processes
    process1.start()
    process2.start()
    
    # Wait for completion
    process1.join()
    process2.join()
    
    print("All processes completed")
```

### Using Process Subclass

```python
import multiprocessing
import time
import math

class CalculationProcess(multiprocessing.Process):
    def __init__(self, name, numbers):
        super().__init__()
        self.name = name
        self.numbers = numbers
        self.result = None
    
    def run(self):
        """Override run method with process logic"""
        print(f"Process {self.name} starting calculation")
        self.result = sum(math.sqrt(x) for x in self.numbers)
        print(f"Process {self.name} finished with result: {self.result}")

if __name__ == "__main__":
    # Large dataset
    numbers1 = list(range(1, 1000000, 2))
    numbers2 = list(range(2, 1000000, 2))
    
    # Create and start processes
    calc1 = CalculationProcess("Calc-1", numbers1)
    calc2 = CalculationProcess("Calc-2", numbers2)
    
    calc1.start()
    calc2.start()
    
    calc1.join()
    calc2.join()
    
    total_result = calc1.result + calc2.result
    print(f"Total result: {total_result}")
```

## Inter-Process Communication

### Pipes

```python
import multiprocessing
import time

def sender(conn, data):
    """Send data through pipe"""
    for item in data:
        conn.send(item)
        print(f"Sent: {item}")
        time.sleep(0.5)
    conn.send(None)  # Signal end
    conn.close()

def receiver(conn):
    """Receive data from pipe"""
    while True:
        item = conn.recv()
        if item is None:
            break
        print(f"Received: {item}")
    conn.close()

if __name__ == "__main__":
    # Create bidirectional pipe
    parent_conn, child_conn = multiprocessing.Pipe()
    
    # Data to send
    data_to_send = ["message1", "message2", "message3", "message4"]
    
    # Create processes
    sender_process = multiprocessing.Process(target=sender, args=(parent_conn, data_to_send))
    receiver_process = multiprocessing.Process(target=receiver, args=(child_conn,))
    
    # Start processes
    receiver_process.start()
    sender_process.start()
    
    # Wait for completion
    sender_process.join()
    receiver_process.join()
```

### Queues

```python
import multiprocessing
import time
import random

def producer(queue, name):
    """Produce items and put them in queue"""
    for i in range(5):
        item = f"Product-{name}-{i}"
        queue.put(item)
        print(f"Producer {name} put {item}")
        time.sleep(random.uniform(0.1, 0.3))
    
    # Send sentinel values to signal completion
    queue.put(None)

def consumer(queue, name):
    """Consume items from queue"""
    while True:
        item = queue.get()
        if item is None:
            break
        print(f"Consumer {name} got {item}")
        time.sleep(random.uniform(0.2, 0.4))
    
    # Put sentinel back for other consumers
    queue.put(None)

if __name__ == "__main__":
    # Create shared queue
    shared_queue = multiprocessing.Queue()
    
    # Create processes
    producers = [
        multiprocessing.Process(target=producer, args=(shared_queue, f"P{i}"))
        for i in range(2)
    ]
    
    consumers = [
        multiprocessing.Process(target=consumer, args=(shared_queue, f"C{i}"))
        for i in range(3)
    ]
    
    # Start all processes
    for p in producers + consumers:
        p.start()
    
    # Wait for producers to finish
    for p in producers:
        p.join()
    
    # Wait for consumers to finish
    for c in consumers:
        c.join()
    
    print("All processes completed")
```

## Shared State

### Shared Variables

```python
import multiprocessing
import time

def increment_shared_value(shared_var, name, iterations):
    """Increment shared variable"""
    for i in range(iterations):
        with shared_var.get_lock():  # Acquire lock for atomic operation
            current_value = shared_var.value
            time.sleep(0.001)  # Simulate processing
            shared_var.value = current_value + 1
        print(f"{name}: {shared_var.value}")

if __name__ == "__main__":
    # Create shared integer
    shared_counter = multiprocessing.Value('i', 0)
    
    # Create processes
    process1 = multiprocessing.Process(
        target=increment_shared_value, 
        args=(shared_counter, "Process-1", 100)
    )
    process2 = multiprocessing.Process(
        target=increment_shared_value, 
        args=(shared_counter, "Process-2", 100)
    )
    
    # Start processes
    process1.start()
    process2.start()
    
    # Wait for completion
    process1.join()
    process2.join()
    
    print(f"Final counter value: {shared_counter.value}")
```

### Shared Arrays

```python
import multiprocessing
import numpy as np

def process_array_segment(shared_array, start_idx, end_idx, process_id):
    """Process segment of shared array"""
    print(f"Process {process_id} working on indices {start_idx}-{end_idx}")
    
    # Modify array segment
    for i in range(start_idx, end_idx):
        shared_array[i] = shared_array[i] ** 2 + process_id
    
    print(f"Process {process_id} completed")

if __name__ == "__main__":
    # Create large array
    size = 1000000
    original_data = np.random.randint(1, 100, size)
    
    # Create shared array
    shared_array = multiprocessing.Array('d', original_data.tolist())
    
    # Number of processes
    num_processes = 4
    chunk_size = size // num_processes
    
    # Create and start processes
    processes = []
    for i in range(num_processes):
        start_idx = i * chunk_size
        end_idx = start_idx + chunk_size if i < num_processes - 1 else size
        
        process = multiprocessing.Process(
            target=process_array_segment,
            args=(shared_array, start_idx, end_idx, i)
        )
        processes.append(process)
        process.start()
    
    # Wait for all processes
    for process in processes:
        process.join()
    
    # Convert back to numpy array for verification
    result_array = np.frombuffer(shared_array.get_obj())
    print(f"First 10 elements: {result_array[:10]}")
    print(f"Last 10 elements: {result_array[-10:]}")
```

## Process Pools

### Pool Class

```python
import multiprocessing
import time
import math

def cpu_intensive_task(n):
    """CPU-intensive calculation"""
    result = 0
    for i in range(n):
        result += math.sin(i) * math.cos(i)
    return result

def main():
    # Data to process
    tasks = [1000000, 2000000, 1500000, 3000000, 2500000]
    
    # Sequential execution
    print("Sequential execution:")
    start_time = time.time()
    sequential_results = [cpu_intensive_task(task) for task in tasks]
    sequential_time = time.time() - start_time
    print(f"Sequential time: {sequential_time:.4f} seconds")
    
    # Parallel execution with Pool
    print("\nParallel execution:")
    start_time = time.time()
    
    with multiprocessing.Pool(processes=4) as pool:
        # Map function to data
        parallel_results = pool.map(cpu_intensive_task, tasks)
        
        # Alternative: async execution
        # async_results = [pool.apply_async(cpu_intensive_task, (task,)) for task in tasks]
        # parallel_results = [ar.get() for ar in async_results]
    
    parallel_time = time.time() - start_time
    print(f"Parallel time: {parallel_time:.4f} seconds")
    print(f"Speedup: {sequential_time/parallel_time:.2f}x")
    
    # Verify results are the same
    print(f"Results match: {sequential_results == parallel_results}")

if __name__ == "__main__":
    multiprocessing.freeze_support()  # Required for Windows
    main()
```

### ProcessPoolExecutor

```python
from concurrent.futures import ProcessPoolExecutor
import time
import requests

def fetch_and_process(url):
    """Simulate CPU-intensive processing of web data"""
    try:
        response = requests.get(url, timeout=5)
        # Simulate CPU-intensive processing
        processed_data = sum(ord(c) for c in response.text[:1000])
        return f"{url}: {processed_data}"
    except Exception as e:
        return f"{url}: Error - {str(e)}"

urls = [
    "https://httpbin.org/json",
    "https://httpbin.org/xml",
    "https://httpbin.org/html",
    "https://httpbin.org/robots.txt"
]

def main():
    # Sequential approach
    print("Sequential execution:")
    start = time.time()
    sequential_results = [fetch_and_process(url) for url in urls]
    sequential_time = time.time() - start
    for result in sequential_results:
        print(result)
    print(f"Sequential time: {sequential_time:.4f}s\n")
    
    # Parallel approach
    print("Parallel execution:")
    start = time.time()
    
    with ProcessPoolExecutor(max_workers=3) as executor:
        # Submit all tasks
        futures = [executor.submit(fetch_and_process, url) for url in urls]
        
        # Collect results as they complete
        parallel_results = []
        for future in futures:
            try:
                result = future.result(timeout=10)
                parallel_results.append(result)
                print(result)
            except Exception as e:
                print(f"Task failed: {e}")
    
    parallel_time = time.time() - start
    print(f"Parallel time: {parallel_time:.4f}s")
    print(f"Performance improvement: {sequential_time/parallel_time:.2f}x")

if __name__ == "__main__":
    main()
```

## Advanced Patterns

### Process Manager

```python
import multiprocessing
import time
import queue

class ProcessManager:
    def __init__(self, max_workers=4):
        self.max_workers = max_workers
        self.task_queue = multiprocessing.Queue()
        self.result_queue = multiprocessing.Queue()
        self.workers = []
        self.running = multiprocessing.Value('b', True)
    
    def worker(self, worker_id):
        """Worker process function"""
        print(f"Worker {worker_id} started")
        while self.running.value:
            try:
                task = self.task_queue.get(timeout=1)
                if task is None:  # Shutdown signal
                    break
                
                func, args, kwargs = task
                result = func(*args, **kwargs)
                self.result_queue.put((worker_id, result))
                
            except queue.Empty:
                continue
            except Exception as e:
                self.result_queue.put((worker_id, f"Error: {e}"))
    
    def start_workers(self):
        """Start worker processes"""
        for i in range(self.max_workers):
            worker = multiprocessing.Process(target=self.worker, args=(i,))
            worker.start()
            self.workers.append(worker)
    
    def submit_task(self, func, *args, **kwargs):
        """Submit task to worker pool"""
        if self.running.value:
            self.task_queue.put((func, args, kwargs))
    
    def get_results(self, timeout=None):
        """Get available results"""
        results = []
        try:
            while True:
                result = self.result_queue.get(timeout=timeout)
                results.append(result)
        except queue.Empty:
            pass
        return results
    
    def shutdown(self):
        """Gracefully shutdown all workers"""
        self.running.value = False
        
        # Send shutdown signals
        for _ in self.workers:
            self.task_queue.put(None)
        
        # Wait for workers to finish
        for worker in self.workers:
            worker.join()
        
        print("All workers shutdown")

# Example usage
def expensive_calculation(data_size):
    """Simulate expensive calculation"""
    import numpy as np
    data = np.random.random(data_size)
    result = np.sum(np.sqrt(data))
    return result

def main():
    manager = ProcessManager(max_workers=3)
    manager.start_workers()
    
    # Submit various tasks
    tasks_data = [100000, 200000, 150000, 300000, 250000]
    
    for data_size in tasks_data:
        manager.submit_task(expensive_calculation, data_size)
    
    # Collect results
    time.sleep(2)  # Allow some processing time
    results = manager.get_results()
    
    for worker_id, result in results:
        print(f"Worker {worker_id}: {result}")
    
    manager.shutdown()

if __name__ == "__main__":
    main()
```

## Performance Comparison

```python
import multiprocessing
import threading
import time
import math

def cpu_bound_task(n):
    """CPU-intensive task"""
    result = 0
    for i in range(n):
        result += math.sin(i) * math.cos(i)
    return result

def benchmark_approaches():
    """Compare threading vs multiprocessing vs sequential"""
    task_size = 2000000
    num_tasks = 4
    
    # Sequential
    print("Sequential execution:")
    start = time.time()
    sequential_results = [cpu_bound_task(task_size) for _ in range(num_tasks)]
    sequential_time = time.time() - start
    print(f"Time: {sequential_time:.4f}s\n")
    
    # Threading (will be slow due to GIL)
    print("Threading execution:")
    start = time.time()
    threads = []
    thread_results = [None] * num_tasks
    
    def thread_wrapper(index):
        thread_results[index] = cpu_bound_task(task_size)
    
    for i in range(num_tasks):
        thread = threading.Thread(target=thread_wrapper, args=(i,))
        threads.append(thread)
        thread.start()
    
    for thread in threads:
        thread.join()
    
    threading_time = time.time() - start
    print(f"Time: {threading_time:.4f}s")
    print(f"Speedup vs sequential: {sequential_time/threading_time:.2f}x\n")
    
    # Multiprocessing
    print("Multiprocessing execution:")
    start = time.time()
    
    with multiprocessing.Pool(processes=num_tasks) as pool:
        mp_results = pool.map(cpu_bound_task, [task_size] * num_tasks)
    
    multiprocessing_time = time.time() - start
    print(f"Time: {multiprocessing_time:.4f}s")
    print(f"Speedup vs sequential: {sequential_time/multiprocessing_time:.2f}x")
    print(f"Speedup vs threading: {threading_time/multiprocessing_time:.2f}x")

if __name__ == "__main__":
    benchmark_approaches()
```

## Best Practices

### When to Use Multiprocessing

✅ **Good for:**
- CPU-intensive computations
- Mathematical calculations
- Image/data processing
- Parallel algorithm execution
- Bypassing GIL limitations

❌ **Avoid for:**
- I/O-bound operations (use threading instead)
- Simple tasks with low overhead
- When memory usage is critical
- Simple coordination requirements

### Memory Considerations

```python
import multiprocessing
import psutil
import time

def memory_intensive_task(size_mb):
    """Task that consumes significant memory"""
    # Allocate memory
    data = [0] * (size_mb * 1024 * 128)  # Approximately size_mb MB
    time.sleep(1)  # Hold memory
    return f"Processed {size_mb}MB"

def monitor_memory():
    """Monitor system memory usage"""
    process = psutil.Process()
    memory_info = process.memory_info()
    return memory_info.rss / 1024 / 1024  # MB

if __name__ == "__main__":
    print(f"Initial memory usage: {monitor_memory():.2f} MB")
    
    # Run tasks sequentially
    print("\nSequential execution:")
    start_memory = monitor_memory()
    for size in [50, 100, 75]:
        result = memory_intensive_task(size)
        current_memory = monitor_memory()
        print(f"{result}, Memory: {current_memory:.2f} MB")
    
    sequential_memory = monitor_memory() - start_memory
    print(f"Sequential memory increase: {sequential_memory:.2f} MB")
    
    # Run tasks in parallel
    print("\nParallel execution:")
    start_memory = monitor_memory()
    
    with multiprocessing.Pool(processes=3) as pool:
        sizes = [50, 100, 75]
        results = pool.map(memory_intensive_task, sizes)
        
        for result in results:
            current_memory = monitor_memory()
            print(f"{result}, Memory: {current_memory:.2f} MB")
    
    parallel_memory = monitor_memory() - start_memory
    print(f"Parallel memory increase: {parallel_memory:.2f} MB")
    print(f"Memory overhead: {parallel_memory - sequential_memory:.2f} MB")
```

Multiprocessing provides true parallelism for CPU-bound tasks but comes with memory overhead and communication complexity costs.
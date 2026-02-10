# Python Coroutines

Coroutines are special functions that can suspend and resume execution, making them perfect for cooperative multitasking. In Python, coroutines are the foundation of the async/await syntax and provide fine-grained control over asynchronous operations.

## Introduction to Coroutines

Coroutines are functions that can pause execution at specific points and later resume from where they left off. Unlike regular functions, coroutines maintain their state between calls and can communicate with other coroutines.

## Basic Coroutine Creation

### Using Generators as Coroutines

```python
import asyncio

def simple_coroutine():
    """Generator-based coroutine"""
    print("Coroutine started")
    value = yield "First yield"
    print(f"Received: {value}")
    value = yield "Second yield"
    print(f"Received: {value}")
    return "Coroutine finished"

# Using the coroutine
def use_generator_coroutine():
    coro = simple_coroutine()
    
    # Start coroutine
    result1 = next(coro)
    print(f"Got: {result1}")
    
    # Send value to coroutine
    try:
        result2 = coro.send("Hello")
        print(f"Got: {result2}")
        
        result3 = coro.send("World")
        print(f"Got: {result3}")
    except StopIteration as e:
        print(f"Coroutine returned: {e.value}")

# Modern async/await coroutine
async def modern_coroutine(name):
    """Async coroutine function"""
    print(f"Coroutine {name} started")
    await asyncio.sleep(0.1)
    print(f"Coroutine {name} yielding")
    value = await asyncio.sleep(0.1, result=f"Value from {name}")
    print(f"Coroutine {name} received: {value}")
    return f"Result from {name}"

async def main():
    # Run multiple coroutines
    coro1 = modern_coroutine("A")
    coro2 = modern_coroutine("B")
    
    # Gather results
    results = await asyncio.gather(coro1, coro2)
    print(f"Results: {results}")

if __name__ == "__main__":
    use_generator_coroutine()
    print("\n" + "="*50 + "\n")
    asyncio.run(main())
```

## Coroutine States and Lifecycle

### Coroutine States

```python
import asyncio
import inspect

async def state_demonstration():
    """Demonstrate coroutine states"""
    print("Coroutine created")
    
    # Created state
    coro = state_demonstration()
    print(f"State: {inspect.getcoroutinestate(coro)}")
    
    # Pending state
    print("About to await...")
    await asyncio.sleep(0.1)
    print("After await")
    
    # Cleanup
    coro.close()

async def coroutine_lifecycle():
    """Complete coroutine lifecycle"""
    print("1. Coroutine definition - not running yet")
    
    # Create coroutine object
    coro = state_demonstration()
    print("2. Coroutine object created")
    
    try:
        # Start execution
        print("3. Starting coroutine execution")
        await coro
        print("4. Coroutine completed normally")
    except Exception as e:
        print(f"4. Coroutine failed: {e}")
    finally:
        print("5. Coroutine cleaned up")

if __name__ == "__main__":
    asyncio.run(coroutine_lifecycle())
```

## Advanced Coroutine Patterns

### Coroutine Chaining

```python
import asyncio

async def fetch_data(source):
    """Simulate data fetching"""
    print(f"Fetching from {source}")
    await asyncio.sleep(0.2)
    return f"Data from {source}"

async def process_data(raw_data):
    """Process fetched data"""
    print(f"Processing: {raw_data}")
    await asyncio.sleep(0.1)
    return f"Processed {raw_data}"

async def save_data(processed_data):
    """Save processed data"""
    print(f"Saving: {processed_data}")
    await asyncio.sleep(0.1)
    return f"Saved {processed_data}"

async def data_pipeline():
    """Chain coroutines together"""
    # Sequential chaining
    raw = await fetch_data("API")
    processed = await process_data(raw)
    result = await save_data(processed)
    return result

async def concurrent_pipeline():
    """Run pipeline steps concurrently where possible"""
    # Fetch from multiple sources concurrently
    sources = ["API-1", "API-2", "API-3"]
    fetch_tasks = [fetch_data(source) for source in sources]
    raw_data_list = await asyncio.gather(*fetch_tasks)
    
    # Process all data concurrently
    process_tasks = [process_data(raw_data) for raw_data in raw_data_list]
    processed_data_list = await asyncio.gather(*process_tasks)
    
    # Save all results concurrently
    save_tasks = [save_data(processed) for processed in processed_data_list]
    results = await asyncio.gather(*save_tasks)
    
    return results

async def main():
    print("Sequential pipeline:")
    result1 = await data_pipeline()
    print(f"Result: {result1}\n")
    
    print("Concurrent pipeline:")
    results = await concurrent_pipeline()
    print(f"Results: {results}")

if __name__ == "__main__":
    asyncio.run(main())
```

### Coroutine Communication

```python
import asyncio
from asyncio import Queue

async def producer(queue, name):
    """Produce items and send to queue"""
    for i in range(5):
        item = f"{name}-item-{i}"
        await queue.put(item)
        print(f"Producer {name} produced {item}")
        await asyncio.sleep(0.1)
    
    # Send sentinel to signal completion
    await queue.put(None)

async def consumer(queue, name):
    """Consume items from queue"""
    while True:
        item = await queue.get()
        if item is None:
            # Put sentinel back for other consumers
            await queue.put(None)
            break
        
        print(f"Consumer {name} consuming {item}")
        await asyncio.sleep(0.2)
        queue.task_done()

async def producer_consumer_system():
    """Coordinator for producer-consumer pattern"""
    # Create shared queue
    queue = Queue(maxsize=3)
    
    # Create producers and consumers
    producers = [
        asyncio.create_task(producer(queue, f"P{i}"))
        for i in range(2)
    ]
    
    consumers = [
        asyncio.create_task(consumer(queue, f"C{i}"))
        for i in range(3)
    ]
    
    # Wait for producers to finish
    await asyncio.gather(*producers)
    
    # Wait for all items to be processed
    await queue.join()
    
    # Wait for consumers to finish
    await asyncio.gather(*consumers)

if __name__ == "__main__":
    asyncio.run(producer_consumer_system())
```

## Coroutine Scheduling and Control

### Custom Coroutine Scheduler

```python
import asyncio
import time
from collections import deque

class CoroutineScheduler:
    def __init__(self):
        self.ready = deque()
        self.sleeping = []
        self.sequence = 0
    
    async def sleep(self, delay):
        """Custom sleep coroutine"""
        deadline = time.time() + delay
        self.sequence += 1
        heapq.heappush(self.sleeping, (deadline, self.sequence, asyncio.current_task()))
        await asyncio.sleep(0)  # Yield control
    
    def call_soon(self, coro):
        """Schedule coroutine for immediate execution"""
        self.ready.append(coro)
    
    def call_later(self, delay, coro):
        """Schedule coroutine for later execution"""
        deadline = time.time() + delay
        self.sequence += 1
        heapq.heappush(self.sleeping, (deadline, self.sequence, coro))
    
    async def run(self):
        """Run the scheduler"""
        while self.ready or self.sleeping:
            # Run ready coroutines
            while self.ready:
                coro = self.ready.popleft()
                try:
                    await coro
                except StopIteration:
                    pass
            
            # Handle sleeping coroutines
            if self.sleeping:
                deadline, _, coro = heapq.heappop(self.sleeping)
                now = time.time()
                if deadline <= now:
                    self.ready.append(coro)
                else:
                    heapq.heappush(self.sleeping, (deadline, self.sequence, coro))
                    await asyncio.sleep(min(deadline - now, 0.001))

# Example usage
async def timed_task(name, scheduler):
    """Task that uses custom scheduler"""
    print(f"Task {name} starting")
    await scheduler.sleep(0.5)
    print(f"Task {name} middle")
    await scheduler.sleep(0.3)
    print(f"Task {name} ending")

async def main():
    scheduler = CoroutineScheduler()
    
    # Schedule tasks
    scheduler.call_soon(timed_task("A", scheduler))
    scheduler.call_soon(timed_task("B", scheduler))
    scheduler.call_later(0.2, timed_task("C", scheduler))
    
    await scheduler.run()

if __name__ == "__main__":
    asyncio.run(main())
```

### Priority-Based Coroutine Execution

```python
import asyncio
import heapq
from enum import IntEnum

class Priority(IntEnum):
    HIGH = 1
    MEDIUM = 2
    LOW = 3

class PriorityScheduler:
    def __init__(self):
        self.queue = []
        self.counter = 0
    
    def schedule(self, coro, priority=Priority.MEDIUM):
        """Schedule coroutine with priority"""
        self.counter += 1
        # Negative priority for min-heap behavior
        heapq.heappush(self.queue, (-priority, self.counter, coro))
    
    async def run(self):
        """Execute coroutines by priority"""
        while self.queue:
            priority, counter, coro = heapq.heappop(self.queue)
            try:
                await coro
            except Exception as e:
                print(f"Coroutine failed: {e}")

async def priority_task(name, priority):
    """Task with priority level"""
    print(f"Executing {name} with priority {priority}")
    await asyncio.sleep(0.1)
    print(f"Completed {name}")

async def priority_scheduling_demo():
    """Demonstrate priority-based scheduling"""
    scheduler = PriorityScheduler()
    
    # Schedule tasks with different priorities
    scheduler.schedule(priority_task("Low Priority", Priority.LOW), Priority.LOW)
    scheduler.schedule(priority_task("High Priority", Priority.HIGH), Priority.HIGH)
    scheduler.schedule(priority_task("Medium Priority", Priority.MEDIUM), Priority.MEDIUM)
    scheduler.schedule(priority_task("Another High", Priority.HIGH), Priority.HIGH)
    
    await scheduler.run()

if __name__ == "__main__":
    asyncio.run(priority_scheduling_demo())
```

## Coroutine Exception Handling

### Exception Propagation

```python
import asyncio

async def failing_coroutine(name):
    """Coroutine that raises exception"""
    print(f"{name} starting")
    await asyncio.sleep(0.1)
    if name == "Bad":
        raise ValueError(f"{name} encountered an error")
    print(f"{name} completing")
    return f"Success from {name}"

async def exception_handling_demo():
    """Demonstrate exception handling patterns"""
    
    # Individual exception handling
    print("=== Individual handling ===")
    tasks = [
        failing_coroutine("Good1"),
        failing_coroutine("Bad"),
        failing_coroutine("Good2")
    ]
    
    results = []
    for task in asyncio.as_completed(tasks):
        try:
            result = await task
            results.append(result)
        except ValueError as e:
            print(f"Caught exception: {e}")
            results.append("FAILED")
    
    print(f"Results: {results}\n")
    
    # Group exception handling
    print("=== Group handling ===")
    try:
        group_results = await asyncio.gather(
            failing_coroutine("Good1"),
            failing_coroutine("Bad"),
            failing_coroutine("Good2"),
            return_exceptions=True  # Continue on exception
        )
        print(f"Group results: {group_results}")
    except Exception as e:
        print(f"Group failed: {e}")

async def supervisor_pattern():
    """Supervisor pattern for error recovery"""
    max_retries = 3
    
    for attempt in range(max_retries):
        try:
            print(f"Attempt {attempt + 1}")
            result = await failing_coroutine("Retryable")
            print(f"Success: {result}")
            break
        except ValueError as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt < max_retries - 1:
                await asyncio.sleep(0.2)  # Backoff
            else:
                print("All attempts failed")
                raise

if __name__ == "__main__":
    asyncio.run(exception_handling_demo())
    print("\n" + "="*50 + "\n")
    asyncio.run(supervisor_pattern())
```

## Advanced Coroutine Techniques

### Coroutine Context Management

```python
import asyncio
import contextvars

# Context variable for request tracking
request_id = contextvars.ContextVar('request_id')

class RequestContext:
    def __init__(self, req_id):
        self.req_id = req_id
    
    async def __aenter__(self):
        self.token = request_id.set(self.req_id)
        print(f"Request context entered: {self.req_id}")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        request_id.reset(self.token)
        print(f"Request context exited: {self.req_id}")

async def process_request(data):
    """Process request with context"""
    current_id = request_id.get()
    print(f"Processing request {current_id}: {data}")
    await asyncio.sleep(0.1)
    return f"Processed {data} for request {current_id}"

async def request_handler(req_id, data):
    """Handle request with proper context"""
    async with RequestContext(req_id):
        result = await process_request(data)
        return result

async def concurrent_requests():
    """Handle multiple requests concurrently"""
    requests = [
        request_handler("req-001", "data1"),
        request_handler("req-002", "data2"),
        request_handler("req-003", "data3")
    ]
    
    results = await asyncio.gather(*requests)
    print(f"All results: {results}")

if __name__ == "__main__":
    asyncio.run(concurrent_requests())
```

### Coroutine Decorators

```python
import asyncio
import functools
import time

def retry(max_attempts=3, delay=1.0):
    """Decorator for retrying coroutines"""
    def decorator(coro_func):
        @functools.wraps(coro_func)
        async def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return await coro_func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        print(f"Attempt {attempt + 1} failed, retrying in {delay}s...")
                        await asyncio.sleep(delay)
                    else:
                        print(f"All {max_attempts} attempts failed")
            
            raise last_exception
        return wrapper
    return decorator

def timeout(seconds):
    """Decorator for adding timeout to coroutines"""
    def decorator(coro_func):
        @functools.wraps(coro_func)
        async def wrapper(*args, **kwargs):
            try:
                return await asyncio.wait_for(
                    coro_func(*args, **kwargs), 
                    timeout=seconds
                )
            except asyncio.TimeoutError:
                raise TimeoutError(f"Function timed out after {seconds} seconds")
        return wrapper
    return decorator

@retry(max_attempts=3, delay=0.5)
async def unreliable_service(call_number):
    """Service that sometimes fails"""
    print(f"Calling service (attempt {call_number})")
    await asyncio.sleep(0.2)
    if call_number % 4 == 0:
        raise ConnectionError("Service temporarily unavailable")
    return f"Response {call_number}"

@timeout(2.0)
async def slow_operation(duration):
    """Operation that might be too slow"""
    print(f"Starting slow operation for {duration}s")
    await asyncio.sleep(duration)
    return f"Completed after {duration}s"

async def decorator_demo():
    """Demonstrate coroutine decorators"""
    # Retry decorator
    print("=== Retry demo ===")
    try:
        result = await unreliable_service(4)
        print(f"Success: {result}")
    except Exception as e:
        print(f"Final failure: {e}")
    
    # Timeout decorator
    print("\n=== Timeout demo ===")
    try:
        result = await slow_operation(1.5)
        print(f"Quick operation: {result}")
        
        result = await slow_operation(3.0)  # Will timeout
        print(f"Slow operation: {result}")
    except TimeoutError as e:
        print(f"Timeout occurred: {e}")

if __name__ == "__main__":
    asyncio.run(decorator_demo())
```

## Performance Monitoring

### Coroutine Performance Tracking

```python
import asyncio
import time
import statistics
from collections import defaultdict

class PerformanceMonitor:
    def __init__(self):
        self.metrics = defaultdict(list)
    
    def monitor(self, name):
        """Decorator to monitor coroutine performance"""
        def decorator(coro_func):
            async def wrapper(*args, **kwargs):
                start_time = time.perf_counter()
                try:
                    result = await coro_func(*args, **kwargs)
                    return result
                finally:
                    elapsed = time.perf_counter() - start_time
                    self.metrics[name].append(elapsed)
            return wrapper
        return decorator
    
    def report(self):
        """Generate performance report"""
        print("\n=== Performance Report ===")
        for name, times in self.metrics.items():
            avg_time = statistics.mean(times)
            min_time = min(times)
            max_time = max(times)
            print(f"{name}:")
            print(f"  Calls: {len(times)}")
            print(f"  Average: {avg_time:.4f}s")
            print(f"  Min: {min_time:.4f}s")
            print(f"  Max: {max_time:.4f}s")
            if len(times) > 1:
                std_dev = statistics.stdev(times)
                print(f"  Std Dev: {std_dev:.4f}s")

monitor = PerformanceMonitor()

@monitor.monitor("database_query")
async def database_query(query_id):
    """Simulate database query"""
    await asyncio.sleep(0.01 + (query_id % 3) * 0.01)
    return f"Result {query_id}"

@monitor.monitor("api_call")
async def api_call(endpoint):
    """Simulate API call"""
    await asyncio.sleep(0.02 + hash(endpoint) % 5 * 0.01)
    return f"Data from {endpoint}"

async def performance_test():
    """Test coroutine performance"""
    # Run mixed workload
    tasks = []
    
    # Database queries
    for i in range(20):
        tasks.append(database_query(i))
    
    # API calls
    endpoints = [f"/api/endpoint-{i}" for i in range(10)]
    for endpoint in endpoints:
        tasks.append(api_call(endpoint))
    
    # Execute all tasks
    results = await asyncio.gather(*tasks)
    print(f"Completed {len(results)} operations")
    
    # Generate report
    monitor.report()

if __name__ == "__main__":
    asyncio.run(performance_test())
```

Coroutines provide powerful mechanisms for cooperative multitasking and are essential for building efficient asynchronous Python applications. Understanding their behavior and patterns is crucial for modern Python development.
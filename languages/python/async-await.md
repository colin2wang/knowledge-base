# Python Async/Await

Python's async/await syntax provides a powerful way to write asynchronous code that can handle I/O-bound operations efficiently without blocking the main thread. This is particularly useful for web servers, API clients, and other I/O-heavy applications.

## Introduction to Async/Await

Async/await is built on top of coroutines and event loops, allowing you to write asynchronous code that looks synchronous. It's ideal for I/O-bound operations like network requests, file operations, and database queries.

## Basic Async Concepts

### Defining Async Functions

```python
import asyncio
import time

async def hello_world():
    """Basic async function"""
    print("Hello")
    await asyncio.sleep(1)  # Non-blocking sleep
    print("World")

# Running async function
async def main():
    await hello_world()

# Execute the async code
if __name__ == "__main__":
    asyncio.run(main())
```

### Running Multiple Coroutines

```python
import asyncio
import time

async def task(name, delay):
    """Async task with delay"""
    print(f"Task {name} starting")
    await asyncio.sleep(delay)
    print(f"Task {name} completed after {delay}s")
    return f"Result from {name}"

async def main():
    # Run tasks concurrently
    start_time = time.time()
    
    # Create tasks
    task1 = asyncio.create_task(task("A", 2))
    task2 = asyncio.create_task(task("B", 1))
    task3 = asyncio.create_task(task("C", 3))
    
    # Wait for all tasks
    results = await asyncio.gather(task1, task2, task3)
    
    end_time = time.time()
    print(f"All tasks completed in {end_time - start_time:.2f}s")
    print(f"Results: {results}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Event Loop Management

### Manual Event Loop Control

```python
import asyncio

async def background_task():
    """Task that runs in background"""
    while True:
        print("Background task running...")
        await asyncio.sleep(1)

async def main():
    # Create task but don't wait for it
    bg_task = asyncio.create_task(background_task())
    
    # Do other work
    for i in range(5):
        print(f"Main task iteration {i}")
        await asyncio.sleep(0.5)
    
    # Cancel background task
    bg_task.cancel()
    try:
        await bg_task
    except asyncio.CancelledError:
        print("Background task was cancelled")

if __name__ == "__main__":
    asyncio.run(main())
```

### Event Loop in Older Python Versions

```python
import asyncio

async def legacy_example():
    """Compatible with older Python versions"""
    print("Running async code")
    await asyncio.sleep(1)
    return "Completed"

# For Python < 3.7
def run_legacy():
    loop = asyncio.get_event_loop()
    try:
        result = loop.run_until_complete(legacy_example())
        print(result)
    finally:
        loop.close()

# For Python 3.7+
def run_modern():
    result = asyncio.run(legacy_example())
    print(result)
```

## Async Context Managers

### Async With Statement

```python
import asyncio
import aiofiles  # Async file operations

class AsyncDatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connected = False
    
    async def connect(self):
        """Simulate database connection"""
        print(f"Connecting to {self.connection_string}")
        await asyncio.sleep(0.5)
        self.connected = True
        print("Connected!")
    
    async def disconnect(self):
        """Simulate database disconnection"""
        print("Disconnecting...")
        await asyncio.sleep(0.3)
        self.connected = False
        print("Disconnected!")
    
    async def execute(self, query):
        """Execute database query"""
        if not self.connected:
            raise RuntimeError("Not connected to database")
        print(f"Executing: {query}")
        await asyncio.sleep(0.2)
        return f"Result for: {query}"
    
    async def __aenter__(self):
        """Async context manager entry"""
        await self.connect()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """Async context manager exit"""
        await self.disconnect()

async def database_operations():
    """Demonstrate async context manager usage"""
    # Using async with
    async with AsyncDatabaseConnection("postgresql://localhost/mydb") as db:
        result1 = await db.execute("SELECT * FROM users")
        result2 = await db.execute("INSERT INTO logs VALUES ('test')")
        print(f"Results: {result1}, {result2}")
    
    # Manual approach (not recommended)
    db = AsyncDatabaseConnection("postgresql://localhost/mydb")
    try:
        await db.connect()
        result = await db.execute("SELECT COUNT(*) FROM products")
        print(f"Count: {result}")
    finally:
        await db.disconnect()

if __name__ == "__main__":
    asyncio.run(database_operations())
```

## Async Iterators and Generators

### Async Iterators

```python
import asyncio

class AsyncCounter:
    def __init__(self, limit):
        self.limit = limit
        self.current = 0
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.current < self.limit:
            await asyncio.sleep(0.1)  # Simulate async operation
            self.current += 1
            return self.current - 1
        else:
            raise StopAsyncIteration

async def iterate_async_counter():
    """Demonstrate async iteration"""
    async for number in AsyncCounter(5):
        print(f"Got number: {number}")

# Async generator version
async def async_range(limit):
    """Async generator function"""
    for i in range(limit):
        await asyncio.sleep(0.1)
        yield i

async def use_async_generator():
    """Use async generator"""
    async for value in async_range(5):
        print(f"Generated: {value}")
```

### Async Comprehensions

```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    """Fetch URL asynchronously"""
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = [
        "https://httpbin.org/json",
        "https://httpbin.org/xml",
        "https://httpbin.org/html"
    ]
    
    async with aiohttp.ClientSession() as session:
        # Async list comprehension
        contents = [await fetch_url(session, url) for url in urls]
        print(f"Fetched {len(contents)} pages")
        
        # Async generator expression
        content_lengths = (len(content) for content in contents)
        total_length = sum(content_lengths)
        print(f"Total content length: {total_length}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Concurrent Execution Patterns

### Gathering Multiple Futures

```python
import asyncio
import aiohttp
import time

async def fetch_with_timeout(session, url, timeout=2):
    """Fetch URL with timeout"""
    try:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=timeout)) as response:
            return await response.text()
    except asyncio.TimeoutError:
        return f"Timeout for {url}"
    except Exception as e:
        return f"Error for {url}: {str(e)}"

async def concurrent_fetching():
    """Fetch multiple URLs concurrently"""
    urls = [
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2",
        "https://httpbin.org/delay/3",
        "https://httpbin.org/status/404",
        "https://invalid-url-that-will-fail.com"
    ]
    
    start_time = time.time()
    
    async with aiohttp.ClientSession() as session:
        # Create tasks for all URLs
        tasks = [fetch_with_timeout(session, url) for url in urls]
        
        # Wait for all tasks to complete
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Process results
        for url, result in zip(urls, results):
            if isinstance(result, Exception):
                print(f"{url}: Exception - {result}")
            else:
                print(f"{url}: {len(result)} characters")
    
    end_time = time.time()
    print(f"Total time: {end_time - start_time:.2f}s")

if __name__ == "__main__":
    asyncio.run(concurrent_fetching())
```

### Task Management and Cancellation

```python
import asyncio
import signal

class TaskManager:
    def __init__(self):
        self.tasks = set()
        self.shutdown_event = asyncio.Event()
    
    def create_task(self, coro, name=None):
        """Create and track task"""
        task = asyncio.create_task(coro, name=name)
        self.tasks.add(task)
        task.add_done_callback(self.tasks.discard)
        return task
    
    async def cancel_all_tasks(self):
        """Cancel all tracked tasks"""
        for task in self.tasks.copy():
            if not task.done():
                task.cancel()
        
        # Wait for tasks to finish cancellation
        if self.tasks:
            await asyncio.wait(self.tasks)
    
    async def graceful_shutdown(self):
        """Handle graceful shutdown"""
        print("Initiating graceful shutdown...")
        self.shutdown_event.set()
        await self.cancel_all_tasks()

async def long_running_task(name, task_manager):
    """Simulate long-running task"""
    try:
        counter = 0
        while not task_manager.shutdown_event.is_set():
            print(f"Task {name}: {counter}")
            counter += 1
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print(f"Task {name} was cancelled")
        raise

async def main():
    """Main application with signal handling"""
    task_manager = TaskManager()
    
    # Set up signal handlers
    def signal_handler():
        asyncio.create_task(task_manager.graceful_shutdown())
    
    # Handle Ctrl+C
    loop = asyncio.get_running_loop()
    for sig in (signal.SIGTERM, signal.SIGINT):
        loop.add_signal_handler(sig, signal_handler)
    
    # Create long-running tasks
    task_manager.create_task(long_running_task("Worker-1", task_manager))
    task_manager.create_task(long_running_task("Worker-2", task_manager))
    
    # Wait for shutdown event
    await task_manager.shutdown_event.wait()
    print("Shutdown complete")

if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("Interrupted by user")
```

## Real-World Examples

### Web Server with Async Views

```python
import asyncio
from aiohttp import web
import json
import time

async def hello_handler(request):
    """Simple async handler"""
    name = request.match_info.get('name', 'World')
    await asyncio.sleep(0.1)  # Simulate processing
    return web.Response(text=f"Hello, {name}!")

async def api_handler(request):
    """JSON API handler"""
    # Simulate database query
    await asyncio.sleep(0.2)
    data = {
        "timestamp": time.time(),
        "message": "Success",
        "data": [1, 2, 3, 4, 5]
    }
    return web.json_response(data)

async def concurrent_handler(request):
    """Handler that makes concurrent external calls"""
    async def fetch_external_data(source):
        # Simulate external API call
        await asyncio.sleep(0.3)
        return f"Data from {source}"
    
    # Make concurrent calls
    tasks = [
        fetch_external_data("API-1"),
        fetch_external_data("API-2"),
        fetch_external_data("API-3")
    ]
    
    results = await asyncio.gather(*tasks)
    
    return web.json_response({
        "results": results,
        "combined": " | ".join(results)
    })

def create_app():
    """Create web application"""
    app = web.Application()
    
    # Routes
    app.router.add_get('/', hello_handler)
    app.router.add_get('/hello/{name}', hello_handler)
    app.router.add_get('/api/data', api_handler)
    app.router.add_get('/api/concurrent', concurrent_handler)
    
    return app

if __name__ == "__main__":
    web.run_app(create_app(), host='localhost', port=8080)
```

### Async Database Operations

```python
import asyncio
import aiopg  # Async PostgreSQL adapter
import asyncpg  # Modern async PostgreSQL driver

class AsyncDatabase:
    def __init__(self, dsn):
        self.dsn = dsn
        self.pool = None
    
    async def connect(self):
        """Create connection pool"""
        self.pool = await asyncpg.create_pool(self.dsn)
        print("Database connected")
    
    async def disconnect(self):
        """Close connection pool"""
        if self.pool:
            await self.pool.close()
            print("Database disconnected")
    
    async def insert_user(self, name, email):
        """Insert user asynchronously"""
        async with self.pool.acquire() as connection:
            # Insert and return generated ID
            user_id = await connection.fetchval(
                "INSERT INTO users(name, email) VALUES($1, $2) RETURNING id",
                name, email
            )
            return user_id
    
    async def get_users(self):
        """Fetch all users"""
        async with self.pool.acquire() as connection:
            rows = await connection.fetch("SELECT id, name, email FROM users")
            return [dict(row) for row in rows]
    
    async def __aenter__(self):
        await self.connect()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.disconnect()

async def database_demo():
    """Demonstrate async database operations"""
    dsn = "postgresql://user:password@localhost/testdb"
    
    async with AsyncDatabase(dsn) as db:
        # Insert users concurrently
        insert_tasks = [
            db.insert_user(f"User{i}", f"user{i}@example.com")
            for i in range(1, 6)
        ]
        
        user_ids = await asyncio.gather(*insert_tasks)
        print(f"Inserted users with IDs: {user_ids}")
        
        # Fetch all users
        users = await db.get_users()
        print(f"Retrieved {len(users)} users:")
        for user in users:
            print(f"  {user}")

if __name__ == "__main__":
    asyncio.run(database_demo())
```

## Performance Comparison

```python
import asyncio
import aiohttp
import requests
import time
from concurrent.futures import ThreadPoolExecutor

# Sync version
def sync_fetch_urls(urls):
    """Synchronous URL fetching"""
    results = []
    for url in urls:
        response = requests.get(url)
        results.append(len(response.text))
    return results

# Async version
async def async_fetch_urls(urls):
    """Asynchronous URL fetching"""
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in urls:
            task = asyncio.create_task(fetch_single_url(session, url))
            tasks.append(task)
        
        results = await asyncio.gather(*tasks)
        return results

async def fetch_single_url(session, url):
    """Fetch single URL"""
    async with session.get(url) as response:
        text = await response.text()
        return len(text)

# Threaded version
def threaded_fetch_urls(urls):
    """Threaded URL fetching"""
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [executor.submit(requests.get, url) for url in urls]
        results = [len(future.result().text) for future in futures]
        return results

async def benchmark_comparison():
    """Compare different approaches"""
    urls = [f"https://httpbin.org/delay/{i%3+1}" for i in range(20)]
    
    # Synchronous
    print("Synchronous approach:")
    start = time.time()
    sync_results = sync_fetch_urls(urls)
    sync_time = time.time() - start
    print(f"Time: {sync_time:.2f}s\n")
    
    # Threaded
    print("Threaded approach:")
    start = time.time()
    thread_results = threaded_fetch_urls(urls)
    thread_time = time.time() - start
    print(f"Time: {thread_time:.2f}s")
    print(f"Speedup: {sync_time/thread_time:.2f}x\n")
    
    # Async
    print("Async approach:")
    start = time.time()
    async_results = await async_fetch_urls(urls)
    async_time = time.time() - start
    print(f"Time: {async_time:.2f}s")
    print(f"Speedup: {sync_time/async_time:.2f}x")
    print(f"vs Threaded: {thread_time/async_time:.2f}x")

if __name__ == "__main__":
    asyncio.run(benchmark_comparison())
```

## Best Practices

### Error Handling in Async Code

```python
import asyncio
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

async def risky_operation(operation_id):
    """Operation that might fail"""
    try:
        if operation_id % 3 == 0:
            raise ValueError(f"Operation {operation_id} failed")
        
        await asyncio.sleep(0.1)
        return f"Success {operation_id}"
    
    except ValueError as e:
        logger.error(f"ValueError in operation {operation_id}: {e}")
        raise
    except Exception as e:
        logger.error(f"Unexpected error in operation {operation_id}: {e}")
        return f"Failed {operation_id}"

async def handle_errors_gracefully():
    """Demonstrate proper error handling"""
    tasks = [risky_operation(i) for i in range(10)]
    
    # Handle exceptions individually
    results = []
    for task in asyncio.as_completed(tasks):
        try:
            result = await task
            results.append(result)
        except Exception as e:
            logger.warning(f"Task failed: {e}")
            results.append("FAILED")
    
    print(f"Results: {results}")

if __name__ == "__main__":
    asyncio.run(handle_errors_gracefully())
```

Async/await provides excellent performance for I/O-bound operations and is becoming the standard for modern Python applications requiring high concurrency.
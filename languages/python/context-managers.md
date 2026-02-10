# Python Context Managers

Context managers provide a clean and Pythonic way to manage resources that need proper setup and cleanup. The `with` statement and the `__enter__`/`__exit__` protocol make resource management automatic and exception-safe.

## Basic Context Manager Concepts

### Understanding the `with` Statement

```python
# Basic file handling without context manager
def file_handling_traditional():
    """Traditional file handling - manual resource management"""
    file_handle = None
    try:
        file_handle = open('example.txt', 'w')
        file_handle.write('Hello, World!')
        # If an exception occurs here, file might not be closed
    except IOError as e:
        print(f"File operation failed: {e}")
    finally:
        if file_handle:
            file_handle.close()  # Manual cleanup

# File handling with context manager
def file_handling_context_manager():
    """File handling with automatic resource management"""
    try:
        with open('example.txt', 'w') as file_handle:
            file_handle.write('Hello, World!')
            # File is automatically closed even if exception occurs
    except IOError as e:
        print(f"File operation failed: {e}")
    # No need for finally block - cleanup is automatic

# Both functions achieve the same result, but context manager is cleaner
file_handling_context_manager()
```

### Creating Custom Context Managers

```python
class DatabaseConnection:
    """Simple database connection context manager"""
    
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None
    
    def __enter__(self):
        """Enter the runtime context"""
        print(f"Connecting to database: {self.connection_string}")
        # Simulate database connection
        self.connection = f"Connection to {self.connection_string}"
        return self.connection
    
    def __exit__(self, exc_type, exc_value, traceback):
        """Exit the runtime context"""
        print("Closing database connection")
        self.connection = None
        # Return False to propagate exceptions, True to suppress them
        return False

# Usage example
def database_operations():
    """Demonstrate custom context manager usage"""
    with DatabaseConnection("postgresql://localhost/mydb") as conn:
        print(f"Connected: {conn}")
        # Perform database operations
        print("Executing queries...")
        # Connection automatically closed when exiting the with block

database_operations()
```

## Advanced Context Manager Implementation

### Context Manager with Exception Handling

```python
import time
from contextlib import contextmanager

class TimedOperation:
    """Context manager that times operations and handles exceptions"""
    
    def __init__(self, operation_name):
        self.operation_name = operation_name
        self.start_time = None
        self.end_time = None
    
    def __enter__(self):
        """Start timing the operation"""
        self.start_time = time.time()
        print(f"Starting {self.operation_name}...")
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        """End timing and handle exceptions"""
        self.end_time = time.time()
        duration = self.end_time - self.start_time
        
        if exc_type is None:
            print(f"{self.operation_name} completed successfully in {duration:.4f}s")
        else:
            print(f"{self.operation_name} failed after {duration:.4f}s")
            print(f"Exception type: {exc_type.__name__}")
            print(f"Exception value: {exc_value}")
            # Return True to suppress the exception, False to propagate it
            return False  # Propagate exceptions by default

# Usage with exception handling
def risky_operation():
    """Operation that might fail"""
    with TimedOperation("Risky Operation") as timer:
        time.sleep(0.1)
        # Uncomment the next line to see exception handling
        # raise ValueError("Something went wrong!")
        return "Success"

try:
    result = risky_operation()
    print(f"Result: {result}")
except ValueError as e:
    print(f"Caught exception: {e}")
```

### Using `contextmanager` Decorator

```python
from contextlib import contextmanager
import tempfile
import os

@contextmanager
def temporary_directory():
    """Context manager for temporary directories"""
    temp_dir = tempfile.mkdtemp()
    try:
        print(f"Created temporary directory: {temp_dir}")
        yield temp_dir  # Provide the directory path to the with block
    finally:
        # Cleanup code
        import shutil
        shutil.rmtree(temp_dir)
        print(f"Removed temporary directory: {temp_dir}")

@contextmanager
def database_transaction(connection):
    """Context manager for database transactions"""
    transaction = connection.begin_transaction()
    try:
        print("Starting transaction")
        yield transaction  # Provide transaction object
        transaction.commit()
        print("Transaction committed")
    except Exception as e:
        print(f"Rolling back transaction due to: {e}")
        transaction.rollback()
        raise  # Re-raise the exception

# Usage examples
def file_processing_example():
    """Demonstrate temporary directory context manager"""
    with temporary_directory() as temp_dir:
        # Work with temporary directory
        temp_file = os.path.join(temp_dir, "temp_file.txt")
        with open(temp_file, 'w') as f:
            f.write("Temporary data")
        print(f"Created temporary file: {temp_file}")
        # Directory and file automatically cleaned up

def database_example():
    """Simulate database transaction"""
    class MockConnection:
        def begin_transaction(self):
            return MockTransaction()
    
    class MockTransaction:
        def commit(self):
            pass
        def rollback(self):
            pass
    
    connection = MockConnection()
    try:
        with database_transaction(connection) as transaction:
            # Perform database operations
            print("Performing database operations...")
            # Uncomment next line to test rollback
            # raise Exception("Database error")
    except Exception as e:
        print(f"Database operation failed: {e}")

file_processing_example()
database_example()
```

## Nested and Multiple Context Managers

### Multiple Context Managers

```python
import threading
from contextlib import ExitStack

class ResourceLocker:
    """Context manager for acquiring and releasing locks"""
    
    def __init__(self, lock_name):
        self.lock_name = lock_name
        self.lock = threading.Lock()
    
    def __enter__(self):
        print(f"Acquiring lock: {self.lock_name}")
        self.lock.acquire()
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        print(f"Releasing lock: {self.lock_name}")
        self.lock.release()
        return False

def nested_context_managers():
    """Demonstrate nested context managers"""
    
    # Method 1: Nested with statements
    print("=== Nested with statements ===")
    with ResourceLocker("Database") as db_lock:
        with ResourceLocker("File") as file_lock:
            with ResourceLocker("Network") as net_lock:
                print("All resources acquired, performing operations...")
                # Operations requiring all three resources
    
    # Method 2: Multiple context managers in single with
    print("\n=== Multiple context managers ===")
    with ResourceLocker("Resource-A") as lock_a, \
         ResourceLocker("Resource-B") as lock_b, \
         ResourceLocker("Resource-C") as lock_c:
        print("All resources acquired simultaneously")
        # Operations with multiple resources
    
    # Method 3: Using ExitStack for dynamic context managers
    print("\n=== Dynamic context managers with ExitStack ===")
    with ExitStack() as stack:
        # Dynamically add context managers
        resources = ["Dynamic-1", "Dynamic-2", "Dynamic-3"]
        locks = []
        
        for resource in resources:
            lock = stack.enter_context(ResourceLocker(resource))
            locks.append(lock)
        
        print("All dynamic resources acquired")
        # Operations with dynamically managed resources

nested_context_managers()
```

## Real-World Context Manager Applications

### Database Connection Pool

```python
import sqlite3
from contextlib import contextmanager
from queue import Queue
import threading

class DatabasePool:
    """Database connection pool with context manager support"""
    
    def __init__(self, db_path, pool_size=5):
        self.db_path = db_path
        self.pool_size = pool_size
        self.pool = Queue(maxsize=pool_size)
        self.lock = threading.Lock()
        
        # Pre-populate pool
        for _ in range(pool_size):
            conn = sqlite3.connect(db_path, check_same_thread=False)
            self.pool.put(conn)
    
    @contextmanager
    def get_connection(self):
        """Get database connection from pool"""
        conn = None
        try:
            print("Acquiring database connection from pool")
            conn = self.pool.get(timeout=5)
            yield conn
        except Exception as e:
            print(f"Database operation failed: {e}")
            raise
        finally:
            if conn:
                print("Returning connection to pool")
                self.pool.put(conn)
    
    def close_all_connections(self):
        """Close all connections in the pool"""
        while not self.pool.empty():
            try:
                conn = self.pool.get_nowait()
                conn.close()
            except:
                pass

# Usage example
def database_pool_example():
    """Demonstrate database connection pooling"""
    # Create in-memory database for demo
    pool = DatabasePool(":memory:", pool_size=3)
    
    try:
        # Create table
        with pool.get_connection() as conn:
            conn.execute("""
                CREATE TABLE users (
                    id INTEGER PRIMARY KEY,
                    name TEXT NOT NULL,
                    email TEXT UNIQUE
                )
            """)
            print("Table created")
        
        # Insert users
        users = [("Alice", "alice@example.com"), ("Bob", "bob@example.com")]
        with pool.get_connection() as conn:
            conn.executemany(
                "INSERT INTO users (name, email) VALUES (?, ?)",
                users
            )
            conn.commit()
            print("Users inserted")
        
        # Query users
        with pool.get_connection() as conn:
            cursor = conn.execute("SELECT * FROM users")
            results = cursor.fetchall()
            print("Users in database:")
            for row in results:
                print(f"  {row}")
    
    finally:
        pool.close_all_connections()

database_pool_example()
```

### HTTP Session Manager

```python
import requests
from contextlib import contextmanager
import time

class HTTPSessionManager:
    """Context manager for HTTP sessions with retry logic"""
    
    def __init__(self, base_url, max_retries=3, timeout=30):
        self.base_url = base_url
        self.max_retries = max_retries
        self.timeout = timeout
        self.session = None
    
    def __enter__(self):
        """Initialize HTTP session"""
        print(f"Creating HTTP session for {self.base_url}")
        self.session = requests.Session()
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        """Close HTTP session"""
        if self.session:
            print("Closing HTTP session")
            self.session.close()
        return False
    
    def get(self, endpoint, **kwargs):
        """GET request with retry logic"""
        url = f"{self.base_url}{endpoint}"
        return self._make_request('GET', url, **kwargs)
    
    def post(self, endpoint, data=None, **kwargs):
        """POST request with retry logic"""
        url = f"{self.base_url}{endpoint}"
        return self._make_request('POST', url, data=data, **kwargs)
    
    def _make_request(self, method, url, **kwargs):
        """Make HTTP request with retry logic"""
        kwargs.setdefault('timeout', self.timeout)
        
        for attempt in range(self.max_retries):
            try:
                print(f"Attempt {attempt + 1}: {method} {url}")
                response = self.session.request(method, url, **kwargs)
                response.raise_for_status()  # Raise exception for bad status codes
                return response
            except (requests.RequestException, requests.HTTPError) as e:
                if attempt < self.max_retries - 1:
                    wait_time = 2 ** attempt  # Exponential backoff
                    print(f"Request failed, retrying in {wait_time}s: {e}")
                    time.sleep(wait_time)
                else:
                    print(f"All retries failed: {e}")
                    raise

# Usage example
@contextmanager
def api_client(base_url):
    """Context manager for API client"""
    with HTTPSessionManager(base_url) as session_manager:
        yield session_manager

def api_operations():
    """Demonstrate HTTP session management"""
    # Using httpbin.org for testing
    base_url = "https://httpbin.org"
    
    with api_client(base_url) as client:
        try:
            # GET request
            response = client.get("/json")
            print(f"GET Status: {response.status_code}")
            print(f"Response: {response.json()['slideshow']['title']}")
            
            # POST request
            data = {"name": "test", "value": "data"}
            response = client.post("/post", json=data)
            print(f"POST Status: {response.status_code}")
            print(f"Posted data: {response.json()['json']}")
            
        except Exception as e:
            print(f"API operation failed: {e}")

api_operations()
```

## Advanced Context Manager Patterns

### Context Manager Composition

```python
from contextlib import contextmanager
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@contextmanager
def performance_monitor(operation_name):
    """Context manager for performance monitoring"""
    import time
    start_time = time.time()
    logger.info(f"Starting {operation_name}")
    try:
        yield
    finally:
        end_time = time.time()
        duration = end_time - start_time
        logger.info(f"Completed {operation_name} in {duration:.4f}s")

@contextmanager
def error_handler(operation_name):
    """Context manager for centralized error handling"""
    try:
        yield
    except ValueError as e:
        logger.error(f"Validation error in {operation_name}: {e}")
        raise
    except IOError as e:
        logger.error(f"I/O error in {operation_name}: {e}")
        raise
    except Exception as e:
        logger.critical(f"Unexpected error in {operation_name}: {e}")
        raise

@contextmanager
def resource_tracker(resource_name):
    """Context manager for resource tracking"""
    logger.info(f"Acquiring resource: {resource_name}")
    try:
        yield resource_name
    finally:
        logger.info(f"Releasing resource: {resource_name}")

def composed_context_managers():
    """Demonstrate composition of multiple context managers"""
    
    def complex_operation():
        """Operation using multiple context managers"""
        with performance_monitor("Complex Operation"):
            with error_handler("Data Processing"):
                with resource_tracker("Database Connection"):
                    # Simulate complex operation
                    import time
                    time.sleep(0.1)
                    print("Performing complex database operation...")
                    
                    with resource_tracker("File Handler"):
                        # Nested resource acquisition
                        time.sleep(0.05)
                        print("Processing file data...")
                        
                        # Uncomment to test error handling
                        # raise ValueError("Invalid data format")
    
    try:
        complex_operation()
    except Exception as e:
        print(f"Operation failed: {e}")

composed_context_managers()
```

### Thread-Safe Context Managers

```python
import threading
from contextlib import contextmanager

class ThreadSafeCounter:
    """Thread-safe counter using context manager"""
    
    def __init__(self):
        self._value = 0
        self._lock = threading.RLock()
    
    @contextmanager
    def increment_context(self):
        """Context manager for safe increment operations"""
        with self._lock:
            old_value = self._value
            self._value += 1
            try:
                yield old_value, self._value
            finally:
                # Ensure cleanup if needed
                pass
    
    @property
    def value(self):
        with self._lock:
            return self._value

def threaded_counter_example():
    """Demonstrate thread-safe context manager"""
    counter = ThreadSafeCounter()
    
    def worker(worker_id):
        """Worker function that uses the counter"""
        for i in range(3):
            with counter.increment_context() as (old_val, new_val):
                print(f"Worker {worker_id}, Iteration {i}: {old_val} -> {new_val}")
    
    # Create and start threads
    threads = []
    for i in range(3):
        thread = threading.Thread(target=worker, args=(i,))
        threads.append(thread)
        thread.start()
    
    # Wait for all threads to complete
    for thread in threads:
        thread.join()
    
    print(f"Final counter value: {counter.value}")

threaded_counter_example()
```

Context managers are essential for writing clean, robust Python code. They ensure proper resource management, make code more readable, and provide automatic cleanup even when exceptions occur.
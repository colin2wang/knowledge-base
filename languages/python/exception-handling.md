# Python Exception Handling

Python's exception handling mechanism provides a structured way to deal with errors and exceptional situations in your code. Proper exception handling is crucial for building robust and maintainable applications.

## Basic Exception Handling

### Try-Except Blocks

```python
def divide_numbers(a, b):
    """Basic division with exception handling"""
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        print("Error: Division by zero!")
        return None
    except TypeError:
        print("Error: Invalid input types!")
        return None

# Usage examples
print(divide_numbers(10, 2))    # Normal case: 5.0
print(divide_numbers(10, 0))    # Zero division: Error message
print(divide_numbers(10, "2"))  # Type error: Error message
```

### Multiple Exception Types

```python
def process_data(data):
    """Handle multiple exception types"""
    try:
        # Various operations that might fail
        length = len(data)
        first_item = data[0]
        numeric_value = int(first_item)
        result = 100 / numeric_value
        return result
    
    except (TypeError, AttributeError) as e:
        print(f"Data structure error: {e}")
        return None
    except IndexError as e:
        print(f"Empty data: {e}")
        return None
    except ValueError as e:
        print(f"Invalid value: {e}")
        return None
    except ZeroDivisionError as e:
        print(f"Mathematical error: {e}")
        return None

# Test different scenarios
test_cases = [
    [1, 2, 3],           # Normal case
    [],                  # Empty list
    ["abc", "def"],      # Invalid conversion
    [0, 1, 2],           # Zero division
    None                 # Type error
]

for case in test_cases:
    result = process_data(case)
    print(f"Input: {case}, Result: {result}")
```

## Advanced Exception Handling

### Exception Hierarchy and Inheritance

```python
import sys

def demonstrate_exception_hierarchy():
    """Show how exceptions are organized in hierarchy"""
    
    # Base exception classes
    print("Exception hierarchy demonstration:")
    
    try:
        # This will raise various exceptions
        operations = [
            lambda: 1/0,                    # ZeroDivisionError
            lambda: int("not_a_number"),    # ValueError
            lambda: [1,2,3][10],            # IndexError
            lambda: {}["missing_key"],      # KeyError
            lambda: open("nonexistent.txt"), # FileNotFoundError
        ]
        
        for i, operation in enumerate(operations):
            try:
                operation()
            except ArithmeticError as e:
                print(f"ArithmeticError caught: {type(e).__name__}: {e}")
            except LookupError as e:
                print(f"LookupError caught: {type(e).__name__}: {e}")
            except OSError as e:
                print(f"OSError caught: {type(e).__name__}: {e}")
            except ValueError as e:
                print(f"ValueError caught: {type(e).__name__}: {e}")
            except Exception as e:
                print(f"Other exception: {type(e).__name__}: {e}")
    
    except Exception as e:
        print(f"Unexpected error: {e}")

# Show exception class hierarchy
def show_exception_tree():
    """Display exception class relationships"""
    import traceback
    
    # Common exception hierarchy
    exception_classes = [
        Exception,
        ArithmeticError,
        LookupError,
        OSError,
        ValueError,
        ZeroDivisionError,
        IndexError,
        KeyError,
        FileNotFoundError,
        TypeError
    ]
    
    print("Exception class hierarchy:")
    for cls in exception_classes:
        print(f"{cls.__name__}: {cls.__bases__[0].__name__}")

demonstrate_exception_hierarchy()
show_exception_tree()
```

### Else and Finally Clauses

```python
def file_processing_example(filename):
    """Demonstrate else and finally usage"""
    file_handle = None
    try:
        print(f"Attempting to open {filename}")
        file_handle = open(filename, 'r')
        content = file_handle.read()
        word_count = len(content.split())
        
    except FileNotFoundError:
        print(f"File {filename} not found")
        return None
    except PermissionError:
        print(f"Permission denied for {filename}")
        return None
    except Exception as e:
        print(f"Unexpected error: {e}")
        return None
    
    else:
        # Executes only if no exception occurred
        print(f"Successfully read {filename}")
        print(f"Word count: {word_count}")
        return word_count
    
    finally:
        # Always executes, regardless of exceptions
        if file_handle:
            file_handle.close()
            print(f"File {filename} closed")
        print("Cleanup completed")

# Usage
file_processing_example("existing_file.txt")
file_processing_example("nonexistent_file.txt")
```

## Exception Information and Debugging

### Accessing Exception Details

```python
import traceback
import sys

def detailed_exception_handling():
    """Show how to extract detailed exception information"""
    
    def problematic_function():
        """Function that raises an exception"""
        data = {"key": "value"}
        return data["missing_key"]  # KeyError
    
    try:
        result = problematic_function()
    except KeyError as e:
        # Extract exception information
        exc_type, exc_value, exc_traceback = sys.exc_info()
        
        print("=== Exception Details ===")
        print(f"Exception type: {exc_type.__name__}")
        print(f"Exception value: {exc_value}")
        print(f"Exception args: {e.args}")
        
        # Print traceback
        print("\n=== Traceback ===")
        traceback.print_exc()
        
        # Get formatted traceback
        tb_lines = traceback.format_exception(exc_type, exc_value, exc_traceback)
        print("\n=== Formatted Traceback ===")
        print(''.join(tb_lines))
        
        # Extract specific frame information
        print("\n=== Frame Information ===")
        tb = exc_traceback
        while tb is not None:
            frame = tb.tb_frame
            filename = frame.f_code.co_filename
            line_number = tb.tb_lineno
            function_name = frame.f_code.co_name
            print(f"File: {filename}, Line: {line_number}, Function: {function_name}")
            tb = tb.tb_next

detailed_exception_handling()
```

### Custom Exception Chaining

```python
def demonstrate_exception_chaining():
    """Show how exceptions can be chained"""
    
    def low_level_operation():
        """Simulate low-level operation that fails"""
        raise ConnectionError("Network connection failed")
    
    def high_level_operation():
        """Higher-level operation that handles low-level errors"""
        try:
            low_level_operation()
        except ConnectionError as e:
            # Chain exceptions to preserve context
            raise RuntimeError("High-level operation failed") from e
    
    try:
        high_level_operation()
    except RuntimeError as e:
        print(f"Caught: {e}")
        print(f"Caused by: {e.__cause__}")
        
        # Show full chain
        current = e
        while current.__cause__:
            print(f"  Caused by: {current.__cause__}")
            current = current.__cause__

demonstrate_exception_chaining()
```

## Context-Specific Exception Handling

### Database Operation Example

```python
import sqlite3
from contextlib import contextmanager

class DatabaseManager:
    def __init__(self, db_path):
        self.db_path = db_path
        self.connection = None
    
    def connect(self):
        """Connect to database with proper exception handling"""
        try:
            self.connection = sqlite3.connect(self.db_path)
            self.connection.row_factory = sqlite3.Row
            print("Database connected successfully")
        except sqlite3.Error as e:
            print(f"Database connection failed: {e}")
            raise
    
    def disconnect(self):
        """Safely close database connection"""
        if self.connection:
            try:
                self.connection.close()
                print("Database connection closed")
            except sqlite3.Error as e:
                print(f"Error closing connection: {e}")
    
    @contextmanager
    def get_cursor(self):
        """Context manager for database cursors"""
        cursor = None
        try:
            cursor = self.connection.cursor()
            yield cursor
        except sqlite3.Error as e:
            print(f"Database operation failed: {e}")
            self.connection.rollback()
            raise
        else:
            self.connection.commit()
        finally:
            if cursor:
                cursor.close()
    
    def create_table(self):
        """Create table with error handling"""
        try:
            with self.get_cursor() as cursor:
                cursor.execute("""
                    CREATE TABLE IF NOT EXISTS users (
                        id INTEGER PRIMARY KEY,
                        name TEXT NOT NULL,
                        email TEXT UNIQUE
                    )
                """)
                print("Table created successfully")
        except sqlite3.Error as e:
            print(f"Failed to create table: {e}")
            raise
    
    def insert_user(self, name, email):
        """Insert user with validation"""
        try:
            with self.get_cursor() as cursor:
                cursor.execute(
                    "INSERT INTO users (name, email) VALUES (?, ?)",
                    (name, email)
                )
                return cursor.lastrowid
        except sqlite3.IntegrityError as e:
            if "UNIQUE constraint failed" in str(e):
                raise ValueError(f"Email {email} already exists") from e
            else:
                raise
        except sqlite3.Error as e:
            print(f"Failed to insert user: {e}")
            raise

# Usage example
def database_operations():
    """Demonstrate database exception handling"""
    db = DatabaseManager(":memory:")  # In-memory database for demo
    
    try:
        db.connect()
        db.create_table()
        
        # Successful operations
        user_id = db.insert_user("John Doe", "john@example.com")
        print(f"Created user with ID: {user_id}")
        
        # This will fail due to duplicate email
        try:
            db.insert_user("Jane Doe", "john@example.com")
        except ValueError as e:
            print(f"Expected error: {e}")
        
    except Exception as e:
        print(f"Database operation failed: {e}")
    finally:
        db.disconnect()

database_operations()
```

## Best Practices

### Proper Exception Design

```python
# Custom exception hierarchy
class ApplicationError(Exception):
    """Base exception for our application"""
    pass

class ValidationError(ApplicationError):
    """Raised when data validation fails"""
    def __init__(self, field, message):
        self.field = field
        self.message = message
        super().__init__(f"Validation failed for {field}: {message}")

class BusinessLogicError(ApplicationError):
    """Raised when business rules are violated"""
    pass

class ExternalServiceError(ApplicationError):
    """Raised when external services fail"""
    def __init__(self, service, error_details):
        self.service = service
        self.error_details = error_details
        super().__init__(f"Service {service} failed: {error_details}")

def validate_user_data(user_data):
    """Validate user data with custom exceptions"""
    if not user_data.get('name'):
        raise ValidationError('name', 'Name is required')
    
    if not user_data.get('email'):
        raise ValidationError('email', 'Email is required')
    
    email = user_data['email']
    if '@' not in email:
        raise ValidationError('email', 'Invalid email format')
    
    if len(user_data.get('name', '')) < 2:
        raise ValidationError('name', 'Name must be at least 2 characters')

def process_user_registration(user_data):
    """Process user registration with proper error handling"""
    try:
        validate_user_data(user_data)
        
        # Simulate external service call
        if user_data.get('premium'):
            raise ExternalServiceError('PaymentService', 'Payment processing timeout')
        
        return {"status": "success", "user_id": 123}
        
    except ValidationError as e:
        return {"status": "error", "field": e.field, "message": e.message}
    except ExternalServiceError as e:
        return {"status": "error", "service": e.service, "details": e.error_details}
    except Exception as e:
        # Log unexpected errors
        print(f"Unexpected error in registration: {e}")
        return {"status": "error", "message": "Internal server error"}

# Test the validation
test_cases = [
    {"name": "John", "email": "john@example.com"},
    {"name": "", "email": "john@example.com"},
    {"name": "John", "email": "invalid-email"},
    {"name": "John", "email": "john@example.com", "premium": True}
]

for i, test_case in enumerate(test_cases, 1):
    print(f"\nTest case {i}: {test_case}")
    result = process_user_registration(test_case)
    print(f"Result: {result}")
```

### Logging and Monitoring

```python
import logging
import functools

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def exception_logger(func):
    """Decorator to log exceptions"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            logger.exception(f"Exception in {func.__name__}: {e}")
            raise
    return wrapper

@exception_logger
def risky_operation(data):
    """Function that might fail"""
    if not data:
        raise ValueError("Empty data provided")
    return sum(data) / len(data)

# Usage with logging
try:
    result = risky_operation([1, 2, 3, 4, 5])
    print(f"Result: {result}")
    
    risky_operation([])  # This will fail
except ValueError as e:
    print(f"Handled error: {e}")

# Exception handler with different log levels
def handle_exceptions(operation_name):
    """Context manager for consistent exception handling"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except ValueError as e:
                logger.warning(f"{operation_name}: Validation error - {e}")
                raise
            except IOError as e:
                logger.error(f"{operation_name}: I/O error - {e}")
                raise
            except Exception as e:
                logger.critical(f"{operation_name}: Unexpected error - {e}")
                raise
        return wrapper
    return decorator

@handle_exceptions("File Processor")
def process_file(filename):
    """Process file with proper error categorization"""
    with open(filename, 'r') as f:
        content = f.read()
        return len(content.split())

# This demonstrates proper exception categorization and logging
```

Exception handling is a fundamental aspect of robust Python programming. Proper implementation helps create more reliable applications and provides better user experience through meaningful error messages.
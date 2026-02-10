# Python Logging

Python's logging module provides a flexible framework for emitting log messages from Python programs. Proper logging is essential for debugging, monitoring, and maintaining production applications.

## Basic Logging Setup

### Simple Logging Configuration

```python
import logging

# Basic configuration
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

# Get logger for current module
logger = logging.getLogger(__name__)

# Different log levels
logger.debug("This is a debug message")
logger.info("This is an info message")
logger.warning("This is a warning message")
logger.error("This is an error message")
logger.critical("This is a critical message")

def basic_logging_example():
    """Demonstrate basic logging usage"""
    logger.info("Starting basic logging example")
    
    try:
        # Simulate some operations
        result = 10 / 2
        logger.info(f"Calculation successful: {result}")
        
        # This will cause an error
        risky_operation = 10 / 0
    except ZeroDivisionError as e:
        logger.error(f"Division by zero error: {e}")
        raise

basic_logging_example()
```

### Logger Hierarchy and Names

```python
import logging

# Configure root logger
logging.basicConfig(level=logging.WARNING)

# Create loggers with different names
root_logger = logging.getLogger()  # Root logger
app_logger = logging.getLogger('myapp')
module_logger = logging.getLogger('myapp.module')
submodule_logger = logging.getLogger('myapp.module.submodule')

def demonstrate_logger_hierarchy():
    """Show how logger hierarchy works"""
    
    print("Logger hierarchy demonstration:")
    print(f"Root logger level: {root_logger.level}")
    print(f"App logger level: {app_logger.level}")
    print(f"Module logger level: {module_logger.level}")
    print(f"Submodule logger level: {submodule_logger.level}")
    
    # By default, loggers inherit level from parent
    print(f"\nSetting app_logger to DEBUG level")
    app_logger.setLevel(logging.DEBUG)
    
    print(f"App logger level: {app_logger.level}")
    print(f"Module logger level: {module_logger.level}")  # Still inherits
    
    # Test logging at different levels
    print("\nLogging messages:")
    root_logger.info("Root logger - INFO message")
    app_logger.debug("App logger - DEBUG message")
    module_logger.info("Module logger - INFO message")
    submodule_logger.warning("Submodule logger - WARNING message")

demonstrate_logger_hierarchy()
```

## Advanced Logging Configuration

### File Configuration

```python
import logging
import logging.config
import yaml
import json
import os

def setup_logging_from_dict():
    """Configure logging using dictionary configuration"""
    
    # Dictionary-based configuration
    log_config = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'standard': {
                'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
            },
            'detailed': {
                'format': '%(asctime)s [%(levelname)s] %(name)s:%(lineno)d: %(message)s'
            }
        },
        'handlers': {
            'console': {
                'class': 'logging.StreamHandler',
                'level': 'INFO',
                'formatter': 'standard',
                'stream': 'ext://sys.stdout'
            },
            'file': {
                'class': 'logging.FileHandler',
                'level': 'DEBUG',
                'formatter': 'detailed',
                'filename': 'app.log',
                'mode': 'a'
            },
            'error_file': {
                'class': 'logging.FileHandler',
                'level': 'ERROR',
                'formatter': 'detailed',
                'filename': 'errors.log',
                'mode': 'a'
            }
        },
        'loggers': {
            'myapp': {
                'level': 'DEBUG',
                'handlers': ['console', 'file'],
                'propagate': False
            },
            'myapp.database': {
                'level': 'WARNING',
                'handlers': ['error_file'],
                'propagate': False
            }
        },
        'root': {
            'level': 'WARNING',
            'handlers': ['console']
        }
    }
    
    # Apply configuration
    logging.config.dictConfig(log_config)
    return logging.getLogger('myapp')

def setup_logging_from_file():
    """Configure logging from external configuration file"""
    
    # YAML configuration (you would typically load from a file)
    yaml_config = """
version: 1
disable_existing_loggers: False

formatters:
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
  json:
    format: '{"time": "%(asctime)s", "logger": "%(name)s", "level": "%(levelname)s", "message": "%(message)s"}'

handlers:
  console:
    class: logging.StreamHandler
    level: INFO
    formatter: simple
    stream: ext://sys.stdout
  
  rotating_file:
    class: logging.handlers.RotatingFileHandler
    level: DEBUG
    formatter: json
    filename: application.log
    maxBytes: 10485760  # 10MB
    backupCount: 5

loggers:
  app:
    level: DEBUG
    handlers: [console, rotating_file]
    propagate: false

root:
  level: WARNING
  handlers: [console]
    """
    
    # Parse YAML and configure
    import io
    config_dict = yaml.safe_load(io.StringIO(yaml_config))
    logging.config.dictConfig(config_dict)
    return logging.getLogger('app')

# Usage example
def advanced_logging_demo():
    """Demonstrate advanced logging features"""
    logger = setup_logging_from_dict()
    
    logger.debug("Debug message - goes to file only")
    logger.info("Info message - goes to console and file")
    logger.warning("Warning message - goes everywhere")
    
    # Logger for database operations
    db_logger = logging.getLogger('myapp.database')
    db_logger.info("Database info - won't appear due to WARNING level")
    db_logger.error("Database error - goes to error file")

advanced_logging_demo()
```

## Structured Logging

### JSON Logging for Better Parsing

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    """Custom formatter for JSON logging"""
    
    def format(self, record):
        """Format log record as JSON"""
        log_entry = {
            'timestamp': datetime.fromtimestamp(record.created).isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        
        # Add exception info if present
        if record.exc_info:
            log_entry['exception'] = self.formatException(record.exc_info)
        
        # Add extra fields
        if hasattr(record, 'extra_data'):
            log_entry['extra'] = record.extra_data
        
        return json.dumps(log_entry, ensure_ascii=False)

def setup_structured_logging():
    """Setup logging with structured JSON output"""
    
    # Create logger
    logger = logging.getLogger('structured_app')
    logger.setLevel(logging.DEBUG)
    
    # Create console handler with JSON formatter
    console_handler = logging.StreamHandler()
    json_formatter = JSONFormatter()
    console_handler.setFormatter(json_formatter)
    
    # Add handler to logger
    logger.addHandler(console_handler)
    
    return logger

def structured_logging_example():
    """Demonstrate structured logging"""
    logger = setup_structured_logging()
    
    # Basic structured logging
    logger.info("User login attempt", extra={'user_id': 12345, 'ip_address': '192.168.1.1'})
    
    # Business operation
    logger.info(
        "Order processed successfully",
        extra={
            'order_id': 'ORD-2024-001',
            'customer_id': 67890,
            'amount': 299.99,
            'items': 3
        }
    )
    
    # Error with context
    try:
        raise ValueError("Invalid payment method")
    except ValueError as e:
        logger.error(
            "Payment processing failed",
            extra={
                'order_id': 'ORD-2024-002',
                'payment_method': 'unsupported_crypto',
                'error_code': 'PAYMENT_METHOD_INVALID'
            },
            exc_info=True
        )

structured_logging_example()
```

## Context-Aware Logging

### Logger Adapters and Filters

```python
import logging
from contextlib import contextmanager

class ContextFilter(logging.Filter):
    """Filter to add contextual information to log records"""
    
    def __init__(self, request_id=None):
        super().__init__()
        self.request_id = request_id or 'N/A'
    
    def filter(self, record):
        """Add request ID to log record"""
        record.request_id = self.request_id
        return True

class ContextAdapter(logging.LoggerAdapter):
    """Logger adapter that adds context to log messages"""
    
    def process(self, msg, kwargs):
        """Process log message and add context"""
        extra = kwargs.get('extra', {})
        extra.update(self.extra)
        kwargs['extra'] = extra
        return msg, kwargs

# Global context storage
_request_context = {}

@contextmanager
def request_context(request_id, user_id=None):
    """Context manager for request-scoped logging context"""
    global _request_context
    old_context = _request_context.copy()
    _request_context.update({
        'request_id': request_id,
        'user_id': user_id
    })
    
    try:
        yield _request_context
    finally:
        _request_context = old_context

def get_current_context():
    """Get current logging context"""
    return _request_context.copy()

class ContextFormatter(logging.Formatter):
    """Formatter that includes context information"""
    
    def format(self, record):
        """Format log record with context"""
        # Add context to record
        context = get_current_context()
        for key, value in context.items():
            setattr(record, key, value)
        
        return super().format(record)

def setup_context_aware_logging():
    """Setup logging with context awareness"""
    
    # Configure logging
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s [%(levelname)s] [Req:%(request_id)s] [User:%(user_id)s] %(name)s: %(message)s'
    )
    
    # Set custom formatter
    root_logger = logging.getLogger()
    for handler in root_logger.handlers:
        handler.setFormatter(ContextFormatter(handler.formatter._fmt))
    
    return logging.getLogger('webapp')

def context_aware_logging_demo():
    """Demonstrate context-aware logging"""
    logger = setup_context_aware_logging()
    
    # Simulate different requests
    requests = [
        ('REQ-001', 12345, 'User login'),
        ('REQ-002', 67890, 'Data processing'),
        ('REQ-003', None, 'System maintenance')
    ]
    
    for req_id, user_id, operation in requests:
        with request_context(req_id, user_id):
            logger.info(f"Starting {operation}")
            
            # Simulate some work
            if user_id:
                logger.debug(f"Processing data for user {user_id}")
            
            # Simulate error in one case
            if req_id == 'REQ-002':
                try:
                    raise RuntimeError("Database connection failed")
                except RuntimeError as e:
                    logger.error(f"Operation failed: {e}")
            
            logger.info(f"Completed {operation}")

context_aware_logging_demo()
```

## Production Logging Setup

### Rotating File Handlers and Syslog

```python
import logging
import logging.handlers
import sys
import os
from logging.handlers import RotatingFileHandler, SysLogHandler

def setup_production_logging():
    """Production-ready logging configuration"""
    
    # Create main logger
    logger = logging.getLogger('production_app')
    logger.setLevel(logging.DEBUG)
    
    # Clear existing handlers
    logger.handlers.clear()
    
    # Formatter for detailed logging
    detailed_formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(funcName)s:%(lineno)d - %(message)s'
    )
    
    # Console handler (INFO and above)
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(detailed_formatter)
    logger.addHandler(console_handler)
    
    # File handler with rotation (DEBUG and above)
    if not os.path.exists('logs'):
        os.makedirs('logs')
    
    file_handler = RotatingFileHandler(
        'logs/application.log',
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5
    )
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(detailed_formatter)
    logger.addHandler(file_handler)
    
    # Error-only file handler
    error_handler = RotatingFileHandler(
        'logs/errors.log',
        maxBytes=5*1024*1024,  # 5MB
        backupCount=3
    )
    error_handler.setLevel(logging.ERROR)
    error_handler.setFormatter(detailed_formatter)
    logger.addHandler(error_handler)
    
    # Syslog handler (optional)
    try:
        syslog_handler = SysLogHandler(address='/dev/log')
        syslog_handler.setLevel(logging.WARNING)
        syslog_formatter = logging.Formatter('%(name)s: %(levelname)s %(message)s')
        syslog_handler.setFormatter(syslog_formatter)
        logger.addHandler(syslog_handler)
    except Exception:
        # Syslog might not be available
        pass
    
    return logger

class PerformanceLogger:
    """Utility class for performance logging"""
    
    def __init__(self, logger):
        self.logger = logger
        self.timers = {}
    
    def start_timer(self, operation_name):
        """Start timing an operation"""
        import time
        self.timers[operation_name] = time.time()
        self.logger.debug(f"Started {operation_name}")
    
    def end_timer(self, operation_name):
        """End timing and log duration"""
        import time
        if operation_name in self.timers:
            duration = time.time() - self.timers[operation_name]
            self.logger.info(f"{operation_name} completed in {duration:.4f}s")
            del self.timers[operation_name]
    
    @contextmanager
    def timed_operation(self, operation_name):
        """Context manager for timed operations"""
        self.start_timer(operation_name)
        try:
            yield
        finally:
            self.end_timer(operation_name)

def production_logging_demo():
    """Demonstrate production logging setup"""
    logger = setup_production_logging()
    perf_logger = PerformanceLogger(logger)
    
    # Simulate application operations
    logger.info("Application started")
    
    # Database operation
    with perf_logger.timed_operation("database_query"):
        logger.debug("Executing complex database query")
        # Simulate work
        import time
        time.sleep(0.1)
        logger.info("Database query completed successfully")
    
    # API call
    with perf_logger.timed_operation("external_api_call"):
        logger.debug("Calling external payment service")
        # Simulate API call
        time.sleep(0.05)
        
        # Simulate occasional error
        if hash("simulate_error") % 10 == 0:
            logger.error("Payment service timeout", extra={
                'service': 'payment_gateway',
                'timeout': '30s'
            })
        else:
            logger.info("Payment processed successfully")
    
    # Business logic
    logger.info("Processing user registration", extra={
        'user_id': 12345,
        'email': 'user@example.com',
        'registration_source': 'web'
    })
    
    logger.info("Application operations completed")

production_logging_demo()
```

## Best Practices and Anti-Patterns

### Logging Best Practices

```python
import logging
import functools
from typing import Callable, Any

# Configure logging for best practices demonstration
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

def log_execution_time(func: Callable) -> Callable:
    """Decorator to log function execution time"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logger = logging.getLogger(func.__module__)
        start_time = time.time()
        
        try:
            result = func(*args, **kwargs)
            execution_time = time.time() - start_time
            logger.info(f"{func.__name__} executed in {execution_time:.4f}s")
            return result
        except Exception as e:
            execution_time = time.time() - start_time
            logger.error(f"{func.__name__} failed after {execution_time:.4f}s: {e}")
            raise
    return wrapper

def avoid_common_logging_mistakes():
    """Demonstrate logging best practices and anti-patterns"""
    
    logger = logging.getLogger(__name__)
    
    # GOOD: Use appropriate log levels
    def good_log_levels():
        logger.debug("Detailed information for debugging")
        logger.info("General information about program execution")
        logger.warning("Something unexpected happened but can continue")
        logger.error("Serious problem that affects functionality")
        logger.critical("Very serious error that might terminate program")
    
    # BAD: String concatenation in logging
    user_id = 12345
    # Bad way - string is always evaluated
    logger.info("Processing user " + str(user_id))
    
    # Good way - string is only evaluated if log level permits
    logger.info("Processing user %s", user_id)
    logger.info("Processing user %(user_id)s", {'user_id': user_id})
    
    # GOOD: Include contextual information
    def process_order(order_id, customer_id):
        logger.info(
            "Processing order",
            extra={
                'order_id': order_id,
                'customer_id': customer_id,
                'timestamp': time.time()
            }
        )
    
    # BAD: Logging sensitive information
    password = "secret123"
    # Never log passwords or other sensitive data
    # logger.info(f"User login with password: {password}")  # DON'T DO THIS
    
    # GOOD: Log only necessary information
    logger.info("User login attempt", extra={'user_id': user_id})
    
    # GOOD: Use exception logging properly
    try:
        risky_operation()
    except Exception as e:
        logger.exception("Operation failed")  # Includes traceback
        # Or manually:
        # logger.error("Operation failed: %s", e, exc_info=True)

def performance_considerations():
    """Show performance-conscious logging"""
    import time
    
    logger = logging.getLogger(__name__)
    
    # Expensive operations should only run when needed
    def expensive_debug_info():
        # Simulate expensive operation
        time.sleep(0.01)
        return "expensive result"
    
    # Check log level before expensive operations
    if logger.isEnabledFor(logging.DEBUG):
        debug_info = expensive_debug_info()
        logger.debug("Expensive debug info: %s", debug_info)
    
    # Use lazy evaluation with % formatting
    logger.debug("Lazy evaluation: %s", expensive_debug_info())

# Demonstrate the concepts
avoid_common_logging_mistakes()
performance_considerations()
```

Logging is a critical aspect of professional Python development. Well-designed logging systems provide invaluable insights into application behavior, facilitate debugging, and enable effective monitoring of production systems.
# Python Custom Exceptions

Creating custom exceptions is essential for building well-structured Python applications. Custom exceptions provide meaningful error messages, enable better error categorization, and improve code maintainability.

## Basic Custom Exception Creation

### Simple Custom Exceptions

```python
# Basic custom exception
class CustomError(Exception):
    """A simple custom exception"""
    pass

# Custom exception with message
class ValidationError(Exception):
    """Raised when data validation fails"""
    def __init__(self, message):
        self.message = message
        super().__init__(self.message)

# Usage examples
def validate_age(age):
    """Validate age with custom exception"""
    if age < 0:
        raise ValidationError("Age cannot be negative")
    if age > 150:
        raise ValidationError("Age seems unrealistic")
    return True

# Test the validation
try:
    validate_age(-5)
except ValidationError as e:
    print(f"Validation failed: {e}")

try:
    validate_age(200)
except ValidationError as e:
    print(f"Validation failed: {e}")
```

### Exception with Additional Attributes

```python
class DatabaseError(Exception):
    """Custom database exception with additional context"""
    def __init__(self, operation, table, error_code=None, details=None):
        self.operation = operation
        self.table = table
        self.error_code = error_code
        self.details = details
        
        # Create informative message
        message_parts = [f"{operation} operation on table '{table}' failed"]
        if error_code:
            message_parts.append(f"(Error code: {error_code})")
        if details:
            message_parts.append(f"Details: {details}")
        
        super().__init__(" ".join(message_parts))

# Usage example
def database_operation(operation, table):
    """Simulate database operation that might fail"""
    if operation == "INSERT" and table == "users":
        raise DatabaseError(
            operation="INSERT", 
            table="users", 
            error_code=1062, 
            details="Duplicate entry for key 'email'"
        )
    
    return f"Successfully executed {operation} on {table}"

# Test with error handling
try:
    result = database_operation("INSERT", "users")
    print(result)
except DatabaseError as e:
    print(f"Database error occurred:")
    print(f"  Operation: {e.operation}")
    print(f"  Table: {e.table}")
    print(f"  Error code: {e.error_code}")
    print(f"  Details: {e.details}")
    print(f"  Message: {e}")
```

## Advanced Custom Exception Patterns

### Exception Hierarchy Design

```python
# Base application exception
class AppError(Exception):
    """Base exception for the entire application"""
    pass

# Category-specific base exceptions
class DataError(AppError):
    """Base for data-related exceptions"""
    pass

class BusinessError(AppError):
    """Base for business logic exceptions"""
    pass

class InfrastructureError(AppError):
    """Base for infrastructure-related exceptions"""
    pass

# Specific exceptions inheriting from category bases
class InvalidDataFormat(DataError):
    """Raised when data format is invalid"""
    def __init__(self, field, expected_format, actual_value):
        self.field = field
        self.expected_format = expected_format
        self.actual_value = actual_value
        super().__init__(
            f"Invalid format for field '{field}'. "
            f"Expected {expected_format}, got '{actual_value}'"
        )

class BusinessRuleViolation(BusinessError):
    """Raised when business rules are violated"""
    def __init__(self, rule_name, violation_details):
        self.rule_name = rule_name
        self.violation_details = violation_details
        super().__init__(f"Business rule '{rule_name}' violated: {violation_details}")

class ServiceUnavailable(InfrastructureError):
    """Raised when external service is unavailable"""
    def __init__(self, service_name, reason=None):
        self.service_name = service_name
        self.reason = reason
        message = f"Service '{service_name}' is unavailable"
        if reason:
            message += f" ({reason})"
        super().__init__(message)

# Demonstrate hierarchy usage
def process_user_data(user_data):
    """Process user data with hierarchical exceptions"""
    if not isinstance(user_data, dict):
        raise InvalidDataFormat("user_data", "dictionary", type(user_data).__name__)
    
    if 'age' in user_data:
        age = user_data['age']
        if not isinstance(age, int):
            raise InvalidDataFormat("age", "integer", type(age).__name__)
        if age < 0 or age > 150:
            raise BusinessRuleViolation("age_validation", f"Age {age} is out of valid range")
    
    if user_data.get('premium') and not user_data.get('payment_method'):
        raise BusinessRuleViolation(
            "premium_requirement", 
            "Premium users must provide payment method"
        )
    
    return "User data processed successfully"

# Test the hierarchy
test_cases = [
    "not_a_dict",                           # InvalidDataFormat
    {"age": "twenty-five"},                 # InvalidDataFormat
    {"age": -5},                            # BusinessRuleViolation
    {"premium": True},                      # BusinessRuleViolation
    {"age": 25, "premium": True, "payment_method": "credit_card"}  # Success
]

for i, test_case in enumerate(test_cases, 1):
    print(f"\nTest case {i}: {test_case}")
    try:
        result = process_user_data(test_case)
        print(f"✓ {result}")
    except DataError as e:
        print(f"✗ Data error: {e}")
    except BusinessError as e:
        print(f"✗ Business error: {e}")
    except AppError as e:
        print(f"✗ Application error: {e}")
```

### Context-Aware Custom Exceptions

```python
import traceback
import sys
from datetime import datetime

class ContextualError(Exception):
    """Exception that captures execution context"""
    def __init__(self, message, context=None):
        self.message = message
        self.context = context or {}
        self.timestamp = datetime.now()
        self.traceback_info = self._capture_traceback()
        super().__init__(self._format_message())
    
    def _capture_traceback(self):
        """Capture current traceback information"""
        exc_type, exc_value, exc_traceback = sys.exc_info()
        if exc_traceback:
            tb_lines = traceback.format_tb(exc_traceback)
            return ''.join(tb_lines)
        return None
    
    def _format_message(self):
        """Format complete error message with context"""
        parts = [self.message]
        if self.context:
            context_str = ', '.join([f"{k}={v}" for k, v in self.context.items()])
            parts.append(f"[Context: {context_str}]")
        parts.append(f"[Time: {self.timestamp}]")
        return ' '.join(parts)
    
    def to_dict(self):
        """Convert exception to dictionary for logging"""
        return {
            'message': self.message,
            'context': self.context,
            'timestamp': self.timestamp.isoformat(),
            'traceback': self.traceback_info
        }

# Specialized contextual exceptions
class APIError(ContextualError):
    """Contextual error for API operations"""
    def __init__(self, endpoint, status_code, response_body=None, **context):
        self.endpoint = endpoint
        self.status_code = status_code
        self.response_body = response_body
        context.update({
            'endpoint': endpoint,
            'status_code': status_code,
            'response_body': response_body
        })
        super().__init__(f"API call to {endpoint} failed with status {status_code}", context)

class ProcessingError(ContextualError):
    """Contextual error for data processing"""
    def __init__(self, data_source, record_id, error_type, **context):
        self.data_source = data_source
        self.record_id = record_id
        self.error_type = error_type
        context.update({
            'data_source': data_source,
            'record_id': record_id,
            'error_type': error_type
        })
        super().__init__(f"Processing failed for record {record_id} from {data_source}", context)

# Usage example
def api_call_simulation(endpoint, data):
    """Simulate API call with contextual errors"""
    if endpoint == "/users" and data.get('email') == 'duplicate@example.com':
        raise APIError(
            endpoint="/users",
            status_code=409,
            response_body={"error": "Email already exists"},
            request_data=data
        )
    
    if endpoint == "/orders" and not data.get('items'):
        raise ProcessingError(
            data_source="order_service",
            record_id=data.get('order_id', 'unknown'),
            error_type="empty_order",
            order_data=data
        )
    
    return {"status": "success", "data": data}

# Test contextual exceptions
test_scenarios = [
    ("/users", {"email": "duplicate@example.com", "name": "John"}),
    ("/orders", {"order_id": "ORD-001", "items": []}),
    ("/products", {"name": "Valid Product"})  # Should succeed
]

for endpoint, data in test_scenarios:
    print(f"\nTesting {endpoint} with {data}")
    try:
        result = api_call_simulation(endpoint, data)
        print(f"✓ Success: {result}")
    except ContextualError as e:
        error_dict = e.to_dict()
        print(f"✗ Error: {error_dict['message']}")
        print(f"  Context: {error_dict['context']}")
        print(f"  Time: {error_dict['timestamp']}")
```

## Exception Factory Pattern

### Dynamic Exception Creation

```python
class ExceptionFactory:
    """Factory for creating custom exceptions dynamically"""
    
    _registry = {}
    
    @classmethod
    def create_exception_class(cls, name, base_class=Exception, **attributes):
        """Create and register a new exception class"""
        if name in cls._registry:
            return cls._registry[name]
        
        # Create new exception class
        exception_class = type(name, (base_class,), {
            '__doc__': attributes.get('doc', f"Custom exception: {name}"),
            '__init__': cls._create_init_method(attributes.get('fields', [])),
            **attributes
        })
        
        cls._registry[name] = exception_class
        return exception_class
    
    @staticmethod
    def _create_init_method(fields):
        """Create __init__ method for exception class"""
        def __init__(self, *args, **kwargs):
            # Store field values
            for field in fields:
                setattr(self, field, kwargs.get(field))
            
            # Create message
            message_parts = list(args)
            if kwargs:
                field_parts = [f"{k}={v}" for k, v in kwargs.items() if k in fields]
                if field_parts:
                    message_parts.append(f"({', '.join(field_parts)})")
            
            super(type(self), self).__init__(' '.join(message_parts) if message_parts else '')
        
        return __init__

# Create various exception types
ValidationError = ExceptionFactory.create_exception_class(
    'ValidationError',
    base_class=ValueError,
    fields=['field', 'value', 'expected'],
    doc="Raised when data validation fails"
)

AuthenticationError = ExceptionFactory.create_exception_class(
    'AuthenticationError',
    base_class=Exception,
    fields=['username', 'reason'],
    doc="Raised when authentication fails"
)

RateLimitError = ExceptionFactory.create_exception_class(
    'RateLimitError',
    base_class=Exception,
    fields=['limit', 'reset_time'],
    doc="Raised when rate limit is exceeded"
)

# Usage examples
def validate_user_input(field, value, expected_type):
    """Validate user input with dynamic exceptions"""
    if not isinstance(value, expected_type):
        raise ValidationError(
            field=field,
            value=value,
            expected=expected_type.__name__
        )
    return True

def authenticate_user(username, password):
    """Authenticate user with custom exceptions"""
    if username == "locked_user":
        raise AuthenticationError(
            username=username,
            reason="Account is locked"
        )
    
    if len(password) < 8:
        raise ValidationError(
            field="password",
            value="***",
            expected="minimum 8 characters"
        )
    
    return True

# Test dynamic exceptions
test_validations = [
    ("age", "twenty", int),
    ("email", "user@example.com", str),
]

for field, value, expected in test_validations:
    try:
        validate_user_input(field, value, expected)
        print(f"✓ {field} validation passed")
    except ValidationError as e:
        print(f"✗ {field} validation failed: field={e.field}, value={e.value}, expected={e.expected}")

# Test authentication
auth_tests = [
    ("normal_user", "password123"),
    ("locked_user", "any_password"),
    ("user1", "short"),
]

for username, password in auth_tests:
    try:
        authenticate_user(username, password)
        print(f"✓ Authentication successful for {username}")
    except AuthenticationError as e:
        print(f"✗ Authentication failed for {e.username}: {e.reason}")
    except ValidationError as e:
        print(f"✗ Validation error: {e.field} {e.expected}")
```

## Best Practices for Custom Exceptions

### Exception Design Guidelines

```python
import logging
from abc import ABC, abstractmethod

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class BaseApplicationException(Exception, ABC):
    """Abstract base class for all application exceptions"""
    
    def __init__(self, message, error_code=None, **context):
        self.message = message
        self.error_code = error_code
        self.context = context
        self.timestamp = self._get_timestamp()
        super().__init__(self._format_message())
    
    @abstractmethod
    def _get_category(self):
        """Return exception category for logging"""
        pass
    
    def _get_timestamp(self):
        """Get current timestamp"""
        from datetime import datetime
        return datetime.now()
    
    def _format_message(self):
        """Format complete error message"""
        parts = [self.message]
        if self.error_code:
            parts.append(f"[Code: {self.error_code}]")
        if self.context:
            ctx_str = ', '.join([f"{k}={v}" for k, v in self.context.items()])
            parts.append(f"[Context: {ctx_str}]")
        return ' '.join(parts)
    
    def log_error(self):
        """Log the exception with appropriate level"""
        category = self._get_category()
        if category == "validation":
            logger.warning(str(self))
        elif category == "business":
            logger.error(str(self))
        else:
            logger.critical(str(self), extra={'exception': self})

# Concrete exception implementations
class ValidationError(BaseApplicationException):
    """Validation-related exceptions"""
    def _get_category(self):
        return "validation"

class BusinessLogicError(BaseApplicationException):
    """Business rule violations"""
    def _get_category(self):
        return "business"

class SystemError(BaseApplicationException):
    """System-level errors"""
    def _get_category(self):
        return "system"

# Usage in a real application
class UserService:
    def __init__(self):
        self.users = {}
    
    def create_user(self, user_data):
        """Create user with comprehensive error handling"""
        # Validation phase
        try:
            self._validate_user_data(user_data)
        except ValidationError as e:
            e.log_error()
            raise
        
        # Business logic phase
        try:
            self._check_business_rules(user_data)
        except BusinessLogicError as e:
            e.log_error()
            raise
        
        # System operation phase
        try:
            user_id = self._persist_user(user_data)
            return user_id
        except SystemError as e:
            e.log_error()
            raise
    
    def _validate_user_data(self, user_data):
        """Validate user input data"""
        required_fields = ['name', 'email', 'age']
        for field in required_fields:
            if field not in user_data:
                raise ValidationError(
                    f"Missing required field: {field}",
                    error_code="MISSING_FIELD",
                    field=field,
                    user_data=user_data
                )
        
        if not isinstance(user_data['age'], int) or user_data['age'] < 0:
            raise ValidationError(
                "Invalid age value",
                error_code="INVALID_AGE",
                age=user_data['age'],
                expected="positive integer"
            )
    
    def _check_business_rules(self, user_data):
        """Check business constraints"""
        email = user_data['email']
        if email in [user['email'] for user in self.users.values()]:
            raise BusinessLogicError(
                "Email already registered",
                error_code="EMAIL_EXISTS",
                email=email
            )
        
        if user_data['age'] < 13:
            raise BusinessLogicError(
                "Users must be at least 13 years old",
                error_code="AGE_RESTRICTION",
                age=user_data['age']
            )
    
    def _persist_user(self, user_data):
        """Persist user to storage"""
        try:
            # Simulate database operation
            user_id = len(self.users) + 1
            self.users[user_id] = user_data
            return user_id
        except Exception as e:
            raise SystemError(
                "Failed to persist user data",
                error_code="DB_ERROR",
                original_error=str(e)
            ) from e

# Demonstrate comprehensive error handling
def demonstrate_best_practices():
    """Show best practices in action"""
    user_service = UserService()
    
    test_cases = [
        {},  # Missing fields
        {'name': 'John', 'email': 'john@example.com', 'age': -5},  # Invalid age
        {'name': 'Jane', 'email': 'john@example.com', 'age': 25},  # Duplicate email
        {'name': 'Bob', 'email': 'bob@example.com', 'age': 10},    # Age restriction
        {'name': 'Alice', 'email': 'alice@example.com', 'age': 25}  # Valid case
    ]
    
    for i, user_data in enumerate(test_cases, 1):
        print(f"\nTest case {i}: {user_data}")
        try:
            user_id = user_service.create_user(user_data)
            print(f"✓ User created successfully with ID: {user_id}")
        except ValidationError as e:
            print(f"✗ Validation error: {e}")
        except BusinessLogicError as e:
            print(f"✗ Business rule violation: {e}")
        except SystemError as e:
            print(f"✗ System error: {e}")

demonstrate_best_practices()
```

Custom exceptions are powerful tools for creating maintainable and user-friendly Python applications. They enable clear error communication and facilitate proper error handling throughout your codebase.
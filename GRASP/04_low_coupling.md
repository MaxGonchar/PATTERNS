# Low Coupling

> **Category:** GRASP (General Responsibility Assignment Software Patterns)  
> **Type:** Evaluative Pattern (Design Quality Principle)  
> **Difficulty:** ⭐⭐ Intermediate

---

## 📖 Definition

**Low Coupling** is an evaluative principle that guides the assignment of responsibilities to minimize dependencies between classes. It states:

> **Assign responsibilities so that coupling remains low.**
>
> Use this principle to evaluate alternatives when choosing where to assign responsibility.

**Coupling** is a measure of how strongly one element is connected to, has knowledge of, or relies on other elements.

---

## 🔍 Explanation

Low Coupling answers the question: **"How do we reduce the impact of change and increase reusability?"**

### Core Principle
- **Minimal Dependencies**: Classes should depend on as few other classes as possible
- **Loose Connections**: Dependencies that exist should be weak and abstract
- **Independence**: Classes should be able to function with minimal knowledge of others
- **Change Isolation**: Changes in one class shouldn't ripple through the system

### Types of Coupling (Worst to Best)

#### 1. **Content Coupling** (Worst) ❌
One class modifies or relies on internal workings of another
```python
class ClassB:
    def modify_class_a(self, a):
        a._internal_state = "changed"  # Directly modifying internals
```

#### 2. **Common Coupling** ❌
Classes share global data
```python
global_config = {"setting": "value"}  # Global state

class ClassA:
    def update(self):
        global_config["setting"] = "new"  # Modifying global state
```

#### 3. **Control Coupling** ⚠️
One class controls the flow of another by passing control information
```python
def process(flag):
    if flag == "type_a":
        # Do type A processing
    elif flag == "type_b":
        # Do type B processing
```

#### 4. **Stamp Coupling** ⚠️
Classes share a complex data structure but only use part of it
```python
def calculate_total(order):  # Receives entire order
    return order.items[0].price  # Only uses items
```

#### 5. **Data Coupling** ✅
Classes communicate through simple parameters (Best)
```python
def calculate_total(items: List[OrderItem]) -> float:
    return sum(item.price for item in items)
```

#### 6. **Message Coupling** ✅ (Best)
Classes communicate through messages/interfaces with no direct dependency
```python
class EventBus:
    def publish(self, event): ...

publisher.publish(OrderPlacedEvent(order_id))
```

### What Creates Coupling?

1. **Type References**: `from module import SpecificClass`
2. **Method Calls**: `obj.specific_method()`
3. **Inheritance**: `class Child(Parent)`
4. **Global Variables**: Shared mutable state
5. **Hardcoded Dependencies**: `service = ConcreteService()`

---

## 🎯 Problem It Solves

### Problems with High Coupling:
1. **Ripple Effects**: Changes cascade through many classes
2. **Difficult Testing**: Must set up many dependencies to test one class
3. **Low Reusability**: Cannot reuse classes independently
4. **Hard to Understand**: Must understand many classes at once
5. **Fragile System**: Small changes break unrelated parts
6. **Parallel Development Difficult**: Teams can't work independently

### Example of High Coupling:
```python
# ❌ BAD: Highly coupled design

class EmailService:
    """Concrete implementation"""
    def send(self, to: str, subject: str, body: str):
        # Send email via SMTP
        print(f"Sending email to {to}")

class SMSService:
    """Another concrete implementation"""
    def send_sms(self, phone: str, message: str):
        # Send SMS
        print(f"Sending SMS to {phone}")

class OrderProcessor:
    """Tightly coupled to specific implementations"""
    def __init__(self):
        # Hardcoded dependencies
        self.email_service = EmailService()
        self.sms_service = SMSService()
        self.db_connection = MySQLDatabase()  # Specific database
    
    def process_order(self, order):
        # Directly calls concrete classes
        self.db_connection.execute("INSERT INTO orders ...")
        
        # Knows specific method names
        self.email_service.send(
            order.customer.email,
            "Order Confirmation",
            f"Your order {order.id} is confirmed"
        )
        
        # Different interface for SMS
        self.sms_service.send_sms(
            order.customer.phone,
            f"Order {order.id} confirmed"
        )

# Problems:
# 1. Cannot test without real EmailService, SMSService, MySQLDatabase
# 2. Cannot swap email provider without changing OrderProcessor
# 3. Cannot reuse OrderProcessor with different notification methods
# 4. Changes to EmailService/SMSService might break OrderProcessor
# 5. Must understand all concrete classes to understand OrderProcessor
```

---

## ✅ Solution With Low Coupling

```python
# ✅ GOOD: Low coupling design using abstraction

from abc import ABC, abstractmethod
from typing import Protocol

# Define interfaces/protocols (abstractions)
class NotificationService(Protocol):
    """Abstract interface - dependency on abstraction, not concretions"""
    def send(self, recipient: str, message: str) -> None: ...

class OrderRepository(ABC):
    """Abstract interface for data access"""
    @abstractmethod
    def save(self, order) -> None: ...

# Concrete implementations
class EmailNotificationService:
    """Implements the protocol"""
    def send(self, recipient: str, message: str) -> None:
        print(f"Email to {recipient}: {message}")

class SMSNotificationService:
    """Also implements the protocol"""
    def send(self, recipient: str, message: str) -> None:
        print(f"SMS to {recipient}: {message}")

class MySQLOrderRepository(OrderRepository):
    def save(self, order) -> None:
        print(f"Saving order {order.id} to MySQL")

class PostgreSQLOrderRepository(OrderRepository):
    def save(self, order) -> None:
        print(f"Saving order {order.id} to PostgreSQL")

# Low coupling through dependency injection
class OrderProcessor:
    """Depends on abstractions, not concrete classes"""
    def __init__(
        self,
        notification_service: NotificationService,
        order_repository: OrderRepository
    ):
        # Dependencies injected - not hardcoded
        self._notification_service = notification_service
        self._order_repository = order_repository
    
    def process_order(self, order):
        # Works with any implementation of the interfaces
        self._order_repository.save(order)
        
        self._notification_service.send(
            order.customer.email,
            f"Order {order.id} is confirmed"
        )

# Usage - easy to configure and test
email_notifier = EmailNotificationService()
mysql_repo = MySQLOrderRepository()

processor = OrderProcessor(email_notifier, mysql_repo)
processor.process_order(order)

# Easy to swap implementations
sms_notifier = SMSNotificationService()
postgres_repo = PostgreSQLOrderRepository()

processor2 = OrderProcessor(sms_notifier, postgres_repo)
processor2.process_order(order)

# Easy to test with mocks
class MockNotificationService:
    def __init__(self):
        self.sent_messages = []
    
    def send(self, recipient: str, message: str) -> None:
        self.sent_messages.append((recipient, message))

mock_notifier = MockNotificationService()
mock_repo = MockOrderRepository()

test_processor = OrderProcessor(mock_notifier, mock_repo)
test_processor.process_order(order)

# Verify behavior
assert len(mock_notifier.sent_messages) == 1
```

---

## 🐍 Practical Python Examples

### Example 1: Payment Processing - Reducing Coupling

```python
# High Coupling Version ❌
class StripePaymentGateway:
    def charge_card(self, amount: float, card_token: str) -> str:
        return "stripe_charge_id_123"

class OrderService:
    def __init__(self):
        # Tightly coupled to Stripe
        self.payment_gateway = StripePaymentGateway()
    
    def place_order(self, order, card_token):
        # Knows Stripe-specific method name
        charge_id = self.payment_gateway.charge_card(
            order.total,
            card_token
        )
        order.payment_id = charge_id


# Low Coupling Version ✅
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class PaymentResult:
    """Value object - decouples from specific gateway response"""
    success: bool
    transaction_id: str
    error_message: str = ""

class PaymentGateway(ABC):
    """Abstract interface - depend on abstraction"""
    @abstractmethod
    def process_payment(self, amount: float, payment_details: dict) -> PaymentResult:
        pass

class StripePaymentGateway(PaymentGateway):
    """Concrete implementation"""
    def process_payment(self, amount: float, payment_details: dict) -> PaymentResult:
        # Stripe-specific logic
        charge_id = self._call_stripe_api(amount, payment_details['card_token'])
        return PaymentResult(
            success=True,
            transaction_id=charge_id
        )
    
    def _call_stripe_api(self, amount, card_token):
        return "stripe_charge_123"

class PayPalPaymentGateway(PaymentGateway):
    """Different implementation - same interface"""
    def process_payment(self, amount: float, payment_details: dict) -> PaymentResult:
        # PayPal-specific logic
        transaction_id = self._call_paypal_api(amount, payment_details['paypal_email'])
        return PaymentResult(
            success=True,
            transaction_id=transaction_id
        )
    
    def _call_paypal_api(self, amount, email):
        return "paypal_txn_456"

class OrderService:
    """Low coupling - depends on abstraction"""
    def __init__(self, payment_gateway: PaymentGateway):
        self._payment_gateway = payment_gateway
    
    def place_order(self, order, payment_details: dict):
        result = self._payment_gateway.process_payment(
            order.total,
            payment_details
        )
        
        if result.success:
            order.mark_as_paid(result.transaction_id)
        else:
            raise PaymentError(result.error_message)

# Easy to switch gateways
stripe_gateway = StripePaymentGateway()
order_service = OrderService(stripe_gateway)

# Or use PayPal
paypal_gateway = PayPalPaymentGateway()
order_service = OrderService(paypal_gateway)

# Easy to test
class MockPaymentGateway(PaymentGateway):
    def process_payment(self, amount, payment_details):
        return PaymentResult(success=True, transaction_id="mock_123")

test_service = OrderService(MockPaymentGateway())
```

### Example 2: Logging - Event-Driven Low Coupling

```python
# High Coupling ❌
class FileLogger:
    def log(self, message):
        with open('app.log', 'a') as f:
            f.write(f"{message}\n")

class UserService:
    def __init__(self):
        self.logger = FileLogger()  # Tightly coupled
    
    def create_user(self, name, email):
        user = User(name, email)
        self.logger.log(f"User created: {email}")  # Direct dependency
        return user

class OrderService:
    def __init__(self):
        self.logger = FileLogger()  # Duplicate coupling
    
    def create_order(self, order):
        self.logger.log(f"Order created: {order.id}")
        return order


# Low Coupling with Events ✅
from typing import Callable, List
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Event:
    """Base event class"""
    timestamp: datetime = None
    
    def __post_init__(self):
        if self.timestamp is None:
            self.timestamp = datetime.now()

@dataclass
class UserCreatedEvent(Event):
    user_id: int
    email: str

@dataclass
class OrderCreatedEvent(Event):
    order_id: int
    total: float

class EventBus:
    """Mediator - reduces coupling between publishers and subscribers"""
    def __init__(self):
        self._subscribers: dict[type, List[Callable]] = {}
    
    def subscribe(self, event_type: type, handler: Callable):
        if event_type not in self._subscribers:
            self._subscribers[event_type] = []
        self._subscribers[event_type].append(handler)
    
    def publish(self, event: Event):
        event_type = type(event)
        if event_type in self._subscribers:
            for handler in self._subscribers[event_type]:
                handler(event)

# Services don't know about logging
class UserService:
    def __init__(self, event_bus: EventBus):
        self._event_bus = event_bus
    
    def create_user(self, name, email):
        user = User(name, email)
        # Just publishes event - doesn't know who handles it
        self._event_bus.publish(UserCreatedEvent(user.id, email))
        return user

class OrderService:
    def __init__(self, event_bus: EventBus):
        self._event_bus = event_bus
    
    def create_order(self, order):
        # Publishes event - low coupling
        self._event_bus.publish(OrderCreatedEvent(order.id, order.total))
        return order

# Logging is a separate concern
class FileLogger:
    def __init__(self, event_bus: EventBus):
        # Logger subscribes to events it cares about
        event_bus.subscribe(UserCreatedEvent, self._handle_user_created)
        event_bus.subscribe(OrderCreatedEvent, self._handle_order_created)
    
    def _handle_user_created(self, event: UserCreatedEvent):
        self._log(f"User created: {event.email}")
    
    def _handle_order_created(self, event: OrderCreatedEvent):
        self._log(f"Order created: {event.order_id}, total: ${event.total}")
    
    def _log(self, message):
        with open('app.log', 'a') as f:
            f.write(f"{message}\n")

# Easy to add more event handlers without changing services
class EmailNotifier:
    def __init__(self, event_bus: EventBus):
        event_bus.subscribe(UserCreatedEvent, self._send_welcome_email)
    
    def _send_welcome_email(self, event: UserCreatedEvent):
        print(f"Sending welcome email to {event.email}")

# Setup
event_bus = EventBus()
logger = FileLogger(event_bus)
notifier = EmailNotifier(event_bus)

user_service = UserService(event_bus)
order_service = OrderService(event_bus)

# Services don't know about logger or notifier
user_service.create_user("Alice", "alice@example.com")
```

### Example 3: Database Access - Repository Pattern

```python
# High Coupling ❌
import psycopg2

class UserService:
    def __init__(self):
        # Tightly coupled to PostgreSQL
        self.conn = psycopg2.connect(
            host="localhost",
            database="mydb",
            user="user",
            password="pass"
        )
    
    def get_user(self, user_id):
        # SQL in business logic
        cursor = self.conn.cursor()
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        row = cursor.fetchone()
        return User(row[0], row[1], row[2])
    
    def save_user(self, user):
        cursor = self.conn.cursor()
        cursor.execute(
            "INSERT INTO users (name, email) VALUES (%s, %s)",
            (user.name, user.email)
        )
        self.conn.commit()


# Low Coupling with Repository ✅
from abc import ABC, abstractmethod
from typing import Optional, List

class UserRepository(ABC):
    """Abstract interface - low coupling"""
    @abstractmethod
    def find_by_id(self, user_id: int) -> Optional['User']:
        pass
    
    @abstractmethod
    def find_by_email(self, email: str) -> Optional['User']:
        pass
    
    @abstractmethod
    def save(self, user: 'User') -> None:
        pass
    
    @abstractmethod
    def delete(self, user_id: int) -> None:
        pass

class PostgreSQLUserRepository(UserRepository):
    """Concrete implementation"""
    def __init__(self, connection):
        self._conn = connection
    
    def find_by_id(self, user_id: int) -> Optional['User']:
        cursor = self._conn.cursor()
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        row = cursor.fetchone()
        return User(row[0], row[1], row[2]) if row else None
    
    def find_by_email(self, email: str) -> Optional['User']:
        cursor = self._conn.cursor()
        cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
        row = cursor.fetchone()
        return User(row[0], row[1], row[2]) if row else None
    
    def save(self, user: 'User') -> None:
        cursor = self._conn.cursor()
        cursor.execute(
            "INSERT INTO users (name, email) VALUES (%s, %s)",
            (user.name, user.email)
        )
        self._conn.commit()
    
    def delete(self, user_id: int) -> None:
        cursor = self._conn.cursor()
        cursor.execute("DELETE FROM users WHERE id = %s", (user_id,))
        self._conn.commit()

class InMemoryUserRepository(UserRepository):
    """Different implementation - same interface"""
    def __init__(self):
        self._users: dict[int, 'User'] = {}
        self._next_id = 1
    
    def find_by_id(self, user_id: int) -> Optional['User']:
        return self._users.get(user_id)
    
    def find_by_email(self, email: str) -> Optional['User']:
        return next((u for u in self._users.values() if u.email == email), None)
    
    def save(self, user: 'User') -> None:
        if user.id is None:
            user.id = self._next_id
            self._next_id += 1
        self._users[user.id] = user
    
    def delete(self, user_id: int) -> None:
        self._users.pop(user_id, None)

class UserService:
    """Low coupling - depends on abstraction"""
    def __init__(self, user_repository: UserRepository):
        self._repository = user_repository
    
    def register_user(self, name: str, email: str) -> 'User':
        # Check if email exists
        existing = self._repository.find_by_email(email)
        if existing:
            raise ValueError("Email already registered")
        
        user = User(None, name, email)
        self._repository.save(user)
        return user
    
    def get_user(self, user_id: int) -> Optional['User']:
        return self._repository.find_by_id(user_id)

# Easy to switch databases
postgres_repo = PostgreSQLUserRepository(pg_connection)
service = UserService(postgres_repo)

# Or use in-memory for testing
memory_repo = InMemoryUserRepository()
test_service = UserService(memory_repo)

# Service code doesn't change!
```

---

## 🏢 Examples in Existing Frameworks

### 1. **Django - Loose Coupling through Settings**
```python
# settings.py - configuration decouples code from specific implementations
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',  # Can switch to MySQL
        'NAME': 'mydb',
    }
}

EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'  # Can switch to console

# Your code doesn't reference specific database or email implementation
from django.core.mail import send_mail

def notify_user(user):
    # Works with any EMAIL_BACKEND configured
    send_mail(
        'Subject',
        'Message',
        'from@example.com',
        [user.email],
    )
```

### 2. **Flask - Dependency Injection Pattern**
```python
from flask import Flask, g
from typing import Protocol

class Database(Protocol):
    def query(self, sql: str): ...

app = Flask(__name__)

def get_db() -> Database:
    """Factory function - decouples route from specific DB"""
    if 'db' not in g:
        g.db = connect_to_database()
    return g.db

@app.route('/users/<int:user_id>')
def get_user(user_id):
    # Depends on abstraction through factory
    db = get_db()
    user = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    return jsonify(user)
```

### 3. **SQLAlchemy - Abstract Interface**
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Engine abstraction - works with PostgreSQL, MySQL, SQLite
engine = create_engine('postgresql://user:pass@localhost/db')
# engine = create_engine('mysql://user:pass@localhost/db')  # Easy switch
# engine = create_engine('sqlite:///test.db')  # Testing

Session = sessionmaker(bind=engine)

# Code doesn't know which database it uses
session = Session()
users = session.query(User).all()
```

### 4. **Python's `logging` Module**
```python
import logging

# Code depends on abstract logger interface
logger = logging.getLogger(__name__)

def process_order(order):
    # Doesn't know if it logs to file, console, or network
    logger.info(f"Processing order {order.id}")
    
# Configuration decoupled from code
logging.basicConfig(
    level=logging.INFO,
    handlers=[
        logging.FileHandler('app.log'),      # Can add/remove handlers
        logging.StreamHandler(),             # without changing code
    ]
)
```

### 5. **Python's `typing` Module - Protocols**
```python
from typing import Protocol

class Drawable(Protocol):
    """Duck typing - loose coupling"""
    def draw(self) -> None: ...

def render_all(items: list[Drawable]):
    """Doesn't care about concrete types, only interface"""
    for item in items:
        item.draw()

class Circle:
    def draw(self): print("Drawing circle")

class Square:
    def draw(self): print("Drawing square")

# Works with any object that has draw()
render_all([Circle(), Square()])  # No inheritance needed
```

---

## ✅ Pros and Cons

### Pros ✅

1. **Easier Maintenance**
   - Changes don't ripple through system
   - Can modify one class without affecting others

2. **Better Testability**
   - Easy to mock dependencies
   - Can test classes in isolation

3. **Higher Reusability**
   - Classes can be used in different contexts
   - Less baggage to bring along

4. **Parallel Development**
   - Teams can work independently
   - Interfaces defined first, implementations later

5. **Flexibility**
   - Easy to swap implementations
   - Can adapt to changing requirements

6. **Better Understanding**
   - Each class is simpler
   - Don't need to understand entire system at once

### Cons ❌

1. **More Abstraction**
   - Additional interfaces/protocols
   - Can seem over-engineered for simple cases

2. **Indirection**
   - Harder to trace code flow
   - More layers to navigate

3. **Initial Complexity**
   - More upfront design work
   - Requires thinking about interfaces

4. **Performance Overhead**
   - Virtual method calls (minimal in practice)
   - Extra objects and indirection

5. **Premature Generalization**
   - Risk of over-abstracting too early
   - YAGNI (You Ain't Gonna Need It) violations

---

## 💡 When to Use

### ✅ Strive for Low Coupling When:

1. **Classes Will Change Independently**
   - Different teams own different parts
   - Example: Payment gateway might change

2. **Need Multiple Implementations**
   - Different databases, notification services
   - Example: Email, SMS, push notifications

3. **Testing is Important**
   - Need to mock external services
   - Example: Payment processing, email sending

4. **High Reusability Needed**
   - Classes used in multiple contexts
   - Example: Utility libraries, frameworks

5. **External Dependencies**
   - Third-party services, hardware
   - Example: Cloud services, external APIs

6. **Long-Term Maintainability**
   - System will evolve over time
   - Example: Enterprise applications

### ⚠️ High Coupling Acceptable When:

1. **Stable Relationships**
   - Classes always used together
   - Example: Point class with x, y coordinates

2. **Performance Critical**
   - Need direct access for speed
   - Example: Game engine inner loops

3. **Simple, Small Systems**
   - YAGNI - don't over-engineer
   - Example: Small scripts, prototypes

4. **Framework/Library Code**
   - Standard library classes
   - Example: Using built-in `datetime` directly

5. **Value Objects**
   - Immutable, simple data structures
   - Example: Using tuples, named tuples

---

## 🚫 Common Mistakes

### Mistake 1: Creating God Interfaces
```python
# ❌ BAD: Interface too large
class IUserService(ABC):
    @abstractmethod
    def create_user(self): pass
    
    @abstractmethod
    def delete_user(self): pass
    
    @abstractmethod
    def update_user(self): pass
    
    @abstractmethod
    def send_email(self): pass  # Unrelated!
    
    @abstractmethod
    def log_activity(self): pass  # Unrelated!
    
    @abstractmethod
    def charge_credit_card(self): pass  # Unrelated!

# ✅ GOOD: Small, focused interfaces (Interface Segregation)
class UserRepository(ABC):
    @abstractmethod
    def save(self, user): pass
    
    @abstractmethod
    def find_by_id(self, user_id): pass

class NotificationService(ABC):
    @abstractmethod
    def send(self, recipient, message): pass

class PaymentService(ABC):
    @abstractmethod
    def process_payment(self, amount): pass
```

### Mistake 2: Premature Abstraction
```python
# ❌ BAD: Over-engineering simple cases
class ILogger(ABC):
    @abstractmethod
    def log(self, message): pass

class ILoggerFactory(ABC):
    @abstractmethod
    def create_logger(self): pass

class FileLoggerFactory(ILoggerFactory):
    def create_logger(self):
        return FileLogger()

class AbstractLoggerDecorator(ILogger):
    def __init__(self, logger: ILogger):
        self._logger = logger

# For a simple script? Overkill!

# ✅ GOOD: Start simple, refactor when needed
import logging

logger = logging.getLogger(__name__)
logger.info("Simple message")

# Refactor to abstraction when you actually need multiple implementations
```

### Mistake 3: Passing Everything Through Constructors
```python
# ❌ BAD: Constructor injection overdone
class OrderService:
    def __init__(
        self,
        user_repo,
        product_repo,
        order_repo,
        payment_service,
        email_service,
        sms_service,
        logger,
        cache,
        metrics,
        config,
        event_bus,
        # ... 20 more dependencies
    ):
        # Too many dependencies = high coupling in disguise
        pass

# ✅ GOOD: Refactor to smaller services or use service locator pattern
class OrderService:
    def __init__(
        self,
        order_repo: OrderRepository,
        payment_service: PaymentService,
        notification_service: NotificationService
    ):
        self._order_repo = order_repo
        self._payment_service = payment_service
        self._notification_service = notification_service

# Or use facades to group related services
class OrderDependencies:
    def __init__(self):
        self.order_repo = OrderRepository()
        self.payment_service = PaymentService()
        self.notification_service = NotificationService()

class OrderService:
    def __init__(self, deps: OrderDependencies):
        self._deps = deps
```

### Mistake 4: Leaky Abstractions
```python
# ❌ BAD: Abstraction exposes implementation details
class Database(ABC):
    @abstractmethod
    def execute_sql(self, sql: str):  # SQL-specific!
        pass

class MongoDatabase(Database):
    def execute_sql(self, sql: str):
        # MongoDB doesn't use SQL - abstraction leaked!
        raise NotImplementedError("MongoDB doesn't support SQL")

# ✅ GOOD: Proper abstraction
class UserRepository(ABC):
    @abstractmethod
    def find_by_id(self, user_id: int) -> Optional[User]:
        pass
    
    @abstractmethod
    def save(self, user: User) -> None:
        pass

class SQLUserRepository(UserRepository):
    def find_by_id(self, user_id: int):
        # Uses SQL internally
        pass

class MongoUserRepository(UserRepository):
    def find_by_id(self, user_id: int):
        # Uses MongoDB internally
        pass
```

---

## 🔗 Related Patterns

### GRASP Patterns
- **Information Expert**: Helps achieve low coupling by keeping data and behavior together
- **Controller**: Separates UI from domain, reducing coupling
- **Polymorphism**: Enables low coupling through abstract interfaces
- **Indirection**: Introduces intermediary to reduce coupling
- **Protected Variations**: Shields from variations using stable interfaces

### GoF Patterns
- **Abstract Factory**: Creates families of objects without coupling to concrete classes
- **Adapter**: Reduces coupling by converting interfaces
- **Bridge**: Separates abstraction from implementation
- **Facade**: Provides simple interface, reducing coupling to subsystem
- **Mediator**: Reduces coupling between components
- **Observer**: Decouples subject from observers
- **Strategy**: Encapsulates algorithms, reducing coupling

### SOLID Principles
- **Dependency Inversion**: Depend on abstractions, not concretions (core to Low Coupling)
- **Interface Segregation**: Small interfaces reduce coupling
- **Open/Closed**: Achieved through low coupling to abstractions

---

## 🎓 Practice Exercises

### Exercise 1: Refactor High Coupling
```python
# Refactor this highly coupled code:
class ReportGenerator:
    def __init__(self):
        self.db = MySQLDatabase("localhost", "mydb")
        self.emailer = SMTPEmailer("smtp.gmail.com")
    
    def generate_and_send_report(self, user_id):
        user = self.db.query(f"SELECT * FROM users WHERE id={user_id}")
        report_data = self.db.query("SELECT * FROM sales")
        
        report_html = self._format_html(report_data)
        
        self.emailer.connect()
        self.emailer.send(user['email'], "Report", report_html)
        self.emailer.disconnect()
```

**Task**: Reduce coupling using abstraction and dependency injection.

### Exercise 2: Event-Driven Architecture
Create an event-driven system where:
- `OrderService` publishes `OrderPlacedEvent`
- `InventoryService`, `EmailService`, and `AnalyticsService` respond
- Services don't know about each other

### Exercise 3: Replace Hard Dependencies
```python
# Current code:
class WeatherApp:
    def __init__(self):
        self.api = OpenWeatherMapAPI(api_key="...")
    
    def get_weather(self, city):
        data = self.api.fetch_weather(city)
        return f"{data['temp']}°C, {data['condition']}"
```

**Task**: Create abstraction to support multiple weather APIs.

---

## ✅ Checklist

- [ ] I understand what coupling is and why it matters
- [ ] I can identify high coupling code smells
- [ ] I know how to use abstraction to reduce coupling
- [ ] I understand dependency injection
- [ ] I know when high coupling is acceptable
- [ ] I can distinguish between necessary and unnecessary coupling
- [ ] I avoid premature abstraction
- [ ] I've refactored at least 2 examples to reduce coupling

---

**Previous Pattern:** [Controller](03_controller.md)  
**Next Pattern:** [High Cohesion](05_high_cohesion.md)

**Status:** 🟢 Completed | 🟡 In Progress | 🔴 Not Started
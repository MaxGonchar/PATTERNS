# Polymorphism

> **Category:** GRASP (General Responsibility Assignment Software Patterns)  
> **Type:** Responsibility Assignment Pattern  
> **Difficulty:** ⭐⭐ Intermediate

---

## 📖 Definition

**Polymorphism** is a principle for assigning responsibilities when behavior varies by type. It states:

> **When related alternatives or behaviors vary by type (class), assign responsibility for the behavior using polymorphic operations to the types for which the behavior varies.**

In simpler terms: **When behavior changes based on type, let each type handle its own behavior instead of using conditional logic.**

---

## 🔍 Explanation

Polymorphism answers the question: **"How do we handle alternatives based on type in a way that is easy to extend and maintain?"**

### Core Principle
- **Type-Specific Behavior**: Each type implements its own variant of the behavior
- **Eliminate Conditionals**: Replace `if/switch` statements checking type with polymorphic calls
- **Open/Closed**: Easy to add new types without modifying existing code
- **Single Mechanism**: Use one interface for many implementations

### What is Polymorphism?

**Polymorphism** (Greek: "many forms") means the ability to treat different types uniformly while they behave differently:

```python
# One interface, many behaviors
shapes = [Circle(), Square(), Triangle()]

for shape in shapes:
    shape.draw()  # Each draws differently - polymorphism!
```

### Three Types of Polymorphism

#### 1. **Subtype Polymorphism** (Most Common)
Different classes inherit from a common base, override methods
```python
class Animal:
    def make_sound(self): pass

class Dog(Animal):
    def make_sound(self): return "Woof!"

class Cat(Animal):
    def make_sound(self): return "Meow!"
```

#### 2. **Duck Typing** (Python-style)
If it walks like a duck and quacks like a duck, it's a duck
```python
# No inheritance needed - just same interface
class FileWriter:
    def write(self, data): pass

class NetworkWriter:
    def write(self, data): pass

# Both work with anything expecting .write()
```

#### 3. **Parametric Polymorphism** (Generics)
Functions work with any type
```python
from typing import TypeVar, List

T = TypeVar('T')

def first(items: List[T]) -> T:
    return items[0]

# Works with any list type
```

---

## 🎯 Problem It Solves

### Problems Without Polymorphism:
1. **Type-Checking Conditionals**: Lots of `if isinstance()` or `type()` checks
2. **Hard to Extend**: Adding new types requires modifying existing code
3. **Scattered Logic**: Type-specific behavior spread across the codebase
4. **Violation of OCP**: Open/Closed Principle violated
5. **Fragile Code**: Easy to forget to update all conditional checks

### Example of Bad Design (Type Checking):
```python
# ❌ BAD: Using conditionals instead of polymorphism

class PaymentProcessor:
    def process_payment(self, payment_method, amount):
        # Type checking - anti-pattern!
        if payment_method.type == "credit_card":
            # Credit card specific logic
            self._validate_card_number(payment_method.card_number)
            self._validate_cvv(payment_method.cvv)
            result = self._charge_credit_card(payment_method, amount)
            
        elif payment_method.type == "paypal":
            # PayPal specific logic
            self._validate_paypal_account(payment_method.email)
            result = self._charge_paypal(payment_method, amount)
            
        elif payment_method.type == "bitcoin":
            # Bitcoin specific logic
            self._validate_wallet_address(payment_method.wallet)
            result = self._charge_bitcoin(payment_method, amount)
            
        elif payment_method.type == "bank_transfer":
            # Bank transfer logic
            self._validate_iban(payment_method.iban)
            result = self._charge_bank_transfer(payment_method, amount)
        
        else:
            raise ValueError(f"Unknown payment type: {payment_method.type}")
        
        return result

# Problems:
# 1. Adding new payment method requires changing this class
# 2. All payment logic mixed together
# 3. Easy to forget to add new type to conditional
# 4. Cannot reuse credit card logic separately
# 5. Violates Open/Closed Principle
```

---

## ✅ Solution With Polymorphism

```python
# ✅ GOOD: Using polymorphism

from abc import ABC, abstractmethod
from typing import Protocol

# Define interface (abstraction)
class PaymentMethod(ABC):
    """Base class defining polymorphic interface"""
    
    @abstractmethod
    def validate(self) -> bool:
        """Each type validates differently"""
        pass
    
    @abstractmethod
    def charge(self, amount: float) -> str:
        """Each type charges differently"""
        pass


# Each type implements its own behavior
class CreditCardPayment(PaymentMethod):
    def __init__(self, card_number, cvv, expiry):
        self.card_number = card_number
        self.cvv = cvv
        self.expiry = expiry
    
    def validate(self) -> bool:
        """Credit card specific validation"""
        if len(self.card_number) != 16:
            return False
        if len(self.cvv) != 3:
            return False
        return True
    
    def charge(self, amount: float) -> str:
        """Credit card specific charging"""
        if not self.validate():
            raise ValueError("Invalid credit card")
        
        # Process credit card charge
        transaction_id = self._process_card_charge(amount)
        return transaction_id
    
    def _process_card_charge(self, amount):
        return f"CC_TXN_{amount}"


class PayPalPayment(PaymentMethod):
    def __init__(self, email, oauth_token):
        self.email = email
        self.oauth_token = oauth_token
    
    def validate(self) -> bool:
        """PayPal specific validation"""
        return '@' in self.email and self.oauth_token
    
    def charge(self, amount: float) -> str:
        """PayPal specific charging"""
        if not self.validate():
            raise ValueError("Invalid PayPal account")
        
        # Process PayPal charge
        transaction_id = self._process_paypal_charge(amount)
        return transaction_id
    
    def _process_paypal_charge(self, amount):
        return f"PP_TXN_{amount}"


class BitcoinPayment(PaymentMethod):
    def __init__(self, wallet_address):
        self.wallet_address = wallet_address
    
    def validate(self) -> bool:
        """Bitcoin specific validation"""
        return self.wallet_address.startswith('bc1')
    
    def charge(self, amount: float) -> str:
        """Bitcoin specific charging"""
        if not self.validate():
            raise ValueError("Invalid Bitcoin wallet")
        
        # Process Bitcoin charge
        transaction_id = self._process_bitcoin_charge(amount)
        return transaction_id
    
    def _process_bitcoin_charge(self, amount):
        return f"BTC_TXN_{amount}"


# Processor uses polymorphic interface - no type checking!
class PaymentProcessor:
    """Works with any PaymentMethod - polymorphism!"""
    
    def process_payment(self, payment_method: PaymentMethod, amount: float) -> str:
        """No conditional logic - just polymorphic call"""
        try:
            # Polymorphic call - behavior depends on actual type
            transaction_id = payment_method.charge(amount)
            return transaction_id
        except ValueError as e:
            raise PaymentError(f"Payment failed: {e}")


# Usage - adding new payment methods is easy
processor = PaymentProcessor()

# Works with any payment method
cc = CreditCardPayment("1234567890123456", "123", "12/25")
processor.process_payment(cc, 99.99)

paypal = PayPalPayment("user@example.com", "oauth_token_123")
processor.process_payment(paypal, 49.99)

bitcoin = BitcoinPayment("bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh")
processor.process_payment(bitcoin, 199.99)

# Add new payment method without changing PaymentProcessor
class ApplePayPayment(PaymentMethod):
    def validate(self): return True
    def charge(self, amount): return f"APPLE_TXN_{amount}"

apple_pay = ApplePayPayment()
processor.process_payment(apple_pay, 79.99)  # Just works!
```

---

## 🐍 Practical Python Examples

### Example 1: Shape Rendering System

```python
from abc import ABC, abstractmethod
from typing import List
import math

# Without Polymorphism ❌
class ShapeRenderer:
    def draw_shape(self, shape):
        # Type checking - bad!
        if shape['type'] == 'circle':
            self._draw_circle(shape['x'], shape['y'], shape['radius'])
        elif shape['type'] == 'rectangle':
            self._draw_rectangle(shape['x'], shape['y'], shape['width'], shape['height'])
        elif shape['type'] == 'triangle':
            self._draw_triangle(shape['points'])
        else:
            raise ValueError(f"Unknown shape: {shape['type']}")
    
    def calculate_area(self, shape):
        # More type checking!
        if shape['type'] == 'circle':
            return math.pi * shape['radius'] ** 2
        elif shape['type'] == 'rectangle':
            return shape['width'] * shape['height']
        elif shape['type'] == 'triangle':
            # Heron's formula...
            pass


# With Polymorphism ✅
class Shape(ABC):
    """Polymorphic interface for all shapes"""
    
    @abstractmethod
    def draw(self) -> None:
        """Each shape draws itself"""
        pass
    
    @abstractmethod
    def calculate_area(self) -> float:
        """Each shape calculates its own area"""
        pass
    
    @abstractmethod
    def calculate_perimeter(self) -> float:
        """Each shape calculates its own perimeter"""
        pass


class Circle(Shape):
    """Circle-specific behavior"""
    def __init__(self, x: float, y: float, radius: float):
        self.x = x
        self.y = y
        self.radius = radius
    
    def draw(self) -> None:
        print(f"Drawing circle at ({self.x}, {self.y}) with radius {self.radius}")
    
    def calculate_area(self) -> float:
        return math.pi * self.radius ** 2
    
    def calculate_perimeter(self) -> float:
        return 2 * math.pi * self.radius


class Rectangle(Shape):
    """Rectangle-specific behavior"""
    def __init__(self, x: float, y: float, width: float, height: float):
        self.x = x
        self.y = y
        self.width = width
        self.height = height
    
    def draw(self) -> None:
        print(f"Drawing rectangle at ({self.x}, {self.y}) "
              f"with width {self.width} and height {self.height}")
    
    def calculate_area(self) -> float:
        return self.width * self.height
    
    def calculate_perimeter(self) -> float:
        return 2 * (self.width + self.height)


class Triangle(Shape):
    """Triangle-specific behavior"""
    def __init__(self, x1: float, y1: float, x2: float, y2: float, x3: float, y3: float):
        self.x1, self.y1 = x1, y1
        self.x2, self.y2 = x2, y2
        self.x3, self.y3 = x3, y3
    
    def draw(self) -> None:
        print(f"Drawing triangle with points "
              f"({self.x1}, {self.y1}), ({self.x2}, {self.y2}), ({self.x3}, {self.y3})")
    
    def calculate_area(self) -> float:
        # Using cross product formula
        return abs((self.x1 * (self.y2 - self.y3) + 
                   self.x2 * (self.y3 - self.y1) + 
                   self.x3 * (self.y1 - self.y2)) / 2.0)
    
    def calculate_perimeter(self) -> float:
        # Calculate distance between points
        def distance(x1, y1, x2, y2):
            return math.sqrt((x2 - x1)**2 + (y2 - y1)**2)
        
        return (distance(self.x1, self.y1, self.x2, self.y2) +
                distance(self.x2, self.y2, self.x3, self.y3) +
                distance(self.x3, self.y3, self.x1, self.y1))


# Canvas uses polymorphism - no type checking needed!
class Canvas:
    """Works with any Shape - polymorphism at work"""
    def __init__(self):
        self.shapes: List[Shape] = []
    
    def add_shape(self, shape: Shape):
        self.shapes.append(shape)
    
    def render_all(self):
        """Polymorphic call - each shape draws differently"""
        for shape in self.shapes:
            shape.draw()  # No if/elif needed!
    
    def calculate_total_area(self) -> float:
        """Polymorphic call - each shape calculates differently"""
        return sum(shape.calculate_area() for shape in self.shapes)
    
    def get_shapes_by_area_range(self, min_area: float, max_area: float) -> List[Shape]:
        """Works with any shape type"""
        return [s for s in self.shapes 
                if min_area <= s.calculate_area() <= max_area]


# Usage
canvas = Canvas()

# Add different shape types
canvas.add_shape(Circle(10, 10, 5))
canvas.add_shape(Rectangle(20, 20, 10, 15))
canvas.add_shape(Triangle(0, 0, 5, 0, 2.5, 4))

# Polymorphic operations
canvas.render_all()  # Each shape draws itself

total_area = canvas.calculate_total_area()  # Each calculates its area
print(f"Total area: {total_area:.2f}")

# Easy to add new shape types
class Hexagon(Shape):
    def __init__(self, x, y, side_length):
        self.x = x
        self.y = y
        self.side_length = side_length
    
    def draw(self):
        print(f"Drawing hexagon at ({self.x}, {self.y})")
    
    def calculate_area(self):
        return (3 * math.sqrt(3) / 2) * self.side_length ** 2
    
    def calculate_perimeter(self):
        return 6 * self.side_length

canvas.add_shape(Hexagon(30, 30, 3))
canvas.render_all()  # Hexagon works without changing Canvas!
```

### Example 2: File Export System

```python
from abc import ABC, abstractmethod
import json
import csv
from typing import List, Dict

# Polymorphic interface
class DataExporter(ABC):
    """Interface for different export formats"""
    
    @abstractmethod
    def export(self, data: List[Dict], filename: str) -> None:
        """Each format exports differently"""
        pass
    
    @abstractmethod
    def get_file_extension(self) -> str:
        """Each format has different extension"""
        pass


class JSONExporter(DataExporter):
    """JSON-specific export behavior"""
    
    def export(self, data: List[Dict], filename: str) -> None:
        with open(filename, 'w') as f:
            json.dump(data, f, indent=2)
        print(f"Exported {len(data)} records to {filename}")
    
    def get_file_extension(self) -> str:
        return ".json"


class CSVExporter(DataExporter):
    """CSV-specific export behavior"""
    
    def export(self, data: List[Dict], filename: str) -> None:
        if not data:
            return
        
        with open(filename, 'w', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=data[0].keys())
            writer.writeheader()
            writer.writerows(data)
        print(f"Exported {len(data)} records to {filename}")
    
    def get_file_extension(self) -> str:
        return ".csv"


class XMLExporter(DataExporter):
    """XML-specific export behavior"""
    
    def export(self, data: List[Dict], filename: str) -> None:
        xml_lines = ['<?xml version="1.0" encoding="UTF-8"?>']
        xml_lines.append('<records>')
        
        for record in data:
            xml_lines.append('  <record>')
            for key, value in record.items():
                xml_lines.append(f'    <{key}>{value}</{key}>')
            xml_lines.append('  </record>')
        
        xml_lines.append('</records>')
        
        with open(filename, 'w') as f:
            f.write('\n'.join(xml_lines))
        print(f"Exported {len(data)} records to {filename}")
    
    def get_file_extension(self) -> str:
        return ".xml"


class MarkdownExporter(DataExporter):
    """Markdown table export"""
    
    def export(self, data: List[Dict], filename: str) -> None:
        if not data:
            return
        
        lines = []
        
        # Header
        headers = list(data[0].keys())
        lines.append('| ' + ' | '.join(headers) + ' |')
        lines.append('| ' + ' | '.join(['---'] * len(headers)) + ' |')
        
        # Rows
        for record in data:
            values = [str(record[h]) for h in headers]
            lines.append('| ' + ' | '.join(values) + ' |')
        
        with open(filename, 'w') as f:
            f.write('\n'.join(lines))
        print(f"Exported {len(data)} records to {filename}")
    
    def get_file_extension(self) -> str:
        return ".md"


# Service uses polymorphism
class ReportGenerator:
    """Works with any DataExporter - polymorphism"""
    
    def __init__(self, exporter: DataExporter):
        self.exporter = exporter
    
    def generate_report(self, data: List[Dict], base_filename: str):
        """No conditional logic - polymorphic behavior"""
        filename = base_filename + self.exporter.get_file_extension()
        self.exporter.export(data, filename)
        return filename


# Usage
data = [
    {'name': 'Alice', 'age': 30, 'city': 'New York'},
    {'name': 'Bob', 'age': 25, 'city': 'Los Angeles'},
    {'name': 'Charlie', 'age': 35, 'city': 'Chicago'},
]

# Same code works with different exporters
json_gen = ReportGenerator(JSONExporter())
json_gen.generate_report(data, 'report')

csv_gen = ReportGenerator(CSVExporter())
csv_gen.generate_report(data, 'report')

xml_gen = ReportGenerator(XMLExporter())
xml_gen.generate_report(data, 'report')

md_gen = ReportGenerator(MarkdownExporter())
md_gen.generate_report(data, 'report')

# Easy to switch at runtime
def export_data(data, format_type):
    exporters = {
        'json': JSONExporter(),
        'csv': CSVExporter(),
        'xml': XMLExporter(),
        'markdown': MarkdownExporter()
    }
    
    exporter = exporters.get(format_type)
    if not exporter:
        raise ValueError(f"Unknown format: {format_type}")
    
    generator = ReportGenerator(exporter)
    return generator.generate_report(data, 'output')
```

### Example 3: Notification System with Duck Typing

```python
# Python's duck typing - no explicit inheritance needed!

class EmailNotifier:
    """Email-specific notification"""
    def send(self, recipient: str, message: str):
        print(f"📧 Sending email to {recipient}: {message}")
        # Email sending logic
        return {'status': 'sent', 'channel': 'email'}


class SMSNotifier:
    """SMS-specific notification"""
    def send(self, recipient: str, message: str):
        print(f"📱 Sending SMS to {recipient}: {message}")
        # SMS sending logic
        return {'status': 'sent', 'channel': 'sms'}


class PushNotifier:
    """Push notification-specific"""
    def send(self, recipient: str, message: str):
        print(f"🔔 Sending push notification to {recipient}: {message}")
        # Push notification logic
        return {'status': 'sent', 'channel': 'push'}


class SlackNotifier:
    """Slack-specific notification"""
    def send(self, recipient: str, message: str):
        print(f"💬 Sending Slack message to {recipient}: {message}")
        # Slack API logic
        return {'status': 'sent', 'channel': 'slack'}


# Using Protocol for type hints (Python 3.8+)
from typing import Protocol

class Notifier(Protocol):
    """Protocol defines the interface - duck typing with type hints"""
    def send(self, recipient: str, message: str) -> dict: ...


class NotificationService:
    """Works with any object that has send() method - polymorphism!"""
    
    def __init__(self):
        self.notifiers: List[Notifier] = []
    
    def add_notifier(self, notifier: Notifier):
        self.notifiers.append(notifier)
    
    def notify_all(self, recipient: str, message: str):
        """Polymorphic call - each notifier sends differently"""
        results = []
        for notifier in self.notifiers:
            result = notifier.send(recipient, message)
            results.append(result)
        return results
    
    def notify_with_fallback(self, recipient: str, message: str):
        """Try each notifier until one succeeds"""
        for notifier in self.notifiers:
            try:
                result = notifier.send(recipient, message)
                if result['status'] == 'sent':
                    return result
            except Exception as e:
                print(f"Failed with {notifier.__class__.__name__}: {e}")
                continue
        
        raise NotificationError("All notification methods failed")


# Usage - polymorphism through duck typing
service = NotificationService()

# Add different notifier types
service.add_notifier(EmailNotifier())
service.add_notifier(SMSNotifier())
service.add_notifier(PushNotifier())

# Polymorphic calls
service.notify_all("user123", "Your order has shipped!")

# Easy to add new notifier type
class DiscordNotifier:
    def send(self, recipient, message):
        print(f"🎮 Sending Discord message to {recipient}: {message}")
        return {'status': 'sent', 'channel': 'discord'}

service.add_notifier(DiscordNotifier())
service.notify_all("user456", "New message!")  # Just works!
```

---

## 🏢 Examples in Existing Frameworks

### 1. **Python's Built-in Functions (Duck Typing)**
```python
# len() works with any object that implements __len__()
len([1, 2, 3])           # List
len("hello")             # String
len({1, 2, 3})          # Set
len({'a': 1, 'b': 2})   # Dict

# Polymorphism! Each type implements __len__() differently


# for loop works with any iterable
for item in [1, 2, 3]:           # List
    print(item)

for char in "hello":              # String
    print(char)

for key in {'a': 1, 'b': 2}:     # Dict
    print(key)

# Polymorphism through __iter__() protocol
```

### 2. **Django Views**
```python
from django.views import View
from django.http import JsonResponse

# Polymorphic interface for HTTP methods
class UserView(View):
    """Each HTTP method has its own behavior"""
    
    def get(self, request, user_id):
        """GET-specific behavior"""
        user = User.objects.get(id=user_id)
        return JsonResponse({'user': user.name})
    
    def post(self, request):
        """POST-specific behavior"""
        user = User.objects.create(**request.POST)
        return JsonResponse({'user': user.id}, status=201)
    
    def delete(self, request, user_id):
        """DELETE-specific behavior"""
        User.objects.filter(id=user_id).delete()
        return JsonResponse({'status': 'deleted'})

# Django's View.dispatch() uses polymorphism internally
# It calls the appropriate method (get/post/delete) based on HTTP verb
```

### 3. **Python's `io` Module**
```python
import io

# Polymorphic interface for different I/O streams
def write_data(stream, data):
    """Works with any file-like object"""
    stream.write(data)
    stream.flush()

# Works with different stream types
file_stream = open('data.txt', 'w')
write_data(file_stream, 'Hello')

memory_stream = io.StringIO()
write_data(memory_stream, 'World')

bytes_stream = io.BytesIO()
write_data(bytes_stream, b'Binary')

# All implement same interface - polymorphism!
```

### 4. **SQLAlchemy Backends**
```python
from sqlalchemy import create_engine

# Same code works with different databases - polymorphism!
# PostgreSQL
engine = create_engine('postgresql://user:pass@localhost/db')

# MySQL
engine = create_engine('mysql://user:pass@localhost/db')

# SQLite
engine = create_engine('sqlite:///test.db')

# Same operations work with all backends
with engine.connect() as conn:
    result = conn.execute("SELECT * FROM users")
    # Polymorphism handles dialect differences
```

### 5. **Pytest Fixtures and Plugins**
```python
import pytest

# Polymorphic interface for different backends
class DatabaseBackend:
    def connect(self): pass
    def disconnect(self): pass
    def execute(self, query): pass

@pytest.fixture(params=['postgresql', 'mysql', 'sqlite'])
def db(request):
    """Pytest runs tests with each backend - polymorphism"""
    backends = {
        'postgresql': PostgreSQLBackend(),
        'mysql': MySQLBackend(),
        'sqlite': SQLiteBackend()
    }
    backend = backends[request.param]
    backend.connect()
    yield backend
    backend.disconnect()

def test_user_creation(db):
    """Works with any database backend"""
    db.execute("INSERT INTO users VALUES ('Alice')")
    # Polymorphism ensures it works with all backends
```

---

## ✅ Pros and Cons

### Pros ✅

1. **Extensibility (Open/Closed Principle)**
   - Add new types without modifying existing code
   - Easy to extend system

2. **Eliminates Conditionals**
   - No type-checking if/switch statements
   - Cleaner, more maintainable code

3. **Better Organization**
   - Type-specific behavior in type-specific classes
   - Clear responsibility assignment

4. **Easier Testing**
   - Test each type independently
   - Easy to mock/stub with polymorphic interface

5. **Runtime Flexibility**
   - Can swap implementations at runtime
   - Strategy pattern enabler

6. **Code Reuse**
   - Common interface promotes reusability
   - Clients work with any conforming type

### Cons ❌

1. **Indirection**
   - Less obvious what code executes
   - Harder to trace through debugger

2. **Learning Curve**
   - Requires understanding OOP concepts
   - Abstract thinking needed

3. **Over-Engineering Risk**
   - Can be overkill for simple cases
   - YAGNI violations possible

4. **Performance Overhead**
   - Virtual method calls (usually negligible)
   - Extra objects created

5. **Interface Stability Required**
   - Changing interface affects all implementations
   - Need to version carefully

---

## 💡 When to Use

### ✅ Use Polymorphism When:

1. **Behavior Varies by Type**
   - Different types need different behavior
   - Example: Different payment methods, shapes, file formats

2. **You Have Type-Checking Conditionals**
   - Lots of `if isinstance()` or `type()` checks
   - Example: Big switch statement on type

3. **Need to Extend with New Types**
   - System should be open to new types
   - Example: Plugin architecture

4. **Same Operation, Different Implementations**
   - Common interface, varied behavior
   - Example: Sort algorithms, compression algorithms

5. **Runtime Type Selection**
   - Don't know exact type until runtime
   - Example: User-selected export format

6. **Following Open/Closed Principle**
   - Want to extend without modification
   - Example: Framework design

### ❌ Don't Use Polymorphism When:

1. **Only One or Two Types**
   - Not worth the abstraction
   - Example: Simple flag: is_active = True/False

2. **Types Never Change**
   - Fixed set of types, never extending
   - Simple conditional may be clearer

3.**Behavior is Simple**
   - Just a simple conditional
   - Example: `if enabled: do_thing() else: skip()`

4. **Performance Critical**
   - Virtual calls have overhead (rare concern)
   - Tight loops in performance-critical code

5. **No Shared Interface Makes Sense**
   - Types are too different
   - Forced polymorphism is awkward

---

## 🚫 Common Mistakes

### Mistake 1: Forced Polymorphism
```python
# ❌ BAD: Forcing polymorphism when not needed
class User:
    def process(self): pass

class Product:
    def process(self): pass

class Order:
    def process(self): pass

# They're unrelated! 'process' means different things

# ✅ GOOD: Only use polymorphism when behavior is related
class PaymentMethod:
    def charge(self, amount): pass  # Same concept

class CreditCard(PaymentMethod):
    def charge(self, amount): pass  # Charging money

class PayPal(PaymentMethod):
    def charge(self, amount): pass  # Also charging money
```

### Mistake 2: Empty Parent Class
```python
# ❌ BAD: Parent exists just for inheritance
class Animal:
    pass  # No common behavior

class Dog(Animal):
    def bark(self): return "Woof"

class Cat(Animal):
    def meow(self): return "Meow"

# No polymorphism here - they don't share behavior!

# ✅ GOOD: Share actual behavior
class Animal:
    def make_sound(self):
        """Polymorphic interface"""
        raise NotImplementedError

class Dog(Animal):
    def make_sound(self): return "Woof"

class Cat(Animal):
    def make_sound(self): return "Meow"

# Now we can do: animals.make_sound() polymorphically
```

### Mistake 3: Type Checking in Polymorphic Code
```python
# ❌ BAD: Defeating polymorphism with type checks
class ShapeRenderer:
    def render(self, shapes):
        for shape in shapes:
            # Still type checking! Defeats polymorphism
            if isinstance(shape, Circle):
                self._render_circle_specially(shape)
            else:
                shape.draw()

# ✅ GOOD: Trust polymorphism
class ShapeRenderer:
    def render(self, shapes):
        for shape in shapes:
            shape.draw()  # Just call it - polymorphism handles it
```

### Mistake 4: Too Many Abstract Methods
```python
# ❌ BAD: Interface too complex
class Plugin(ABC):
    @abstractmethod
    def initialize(self): pass
    
    @abstractmethod
    def execute(self): pass
    
    @abstractmethod
    def validate(self): pass
    
    @abstractmethod
    def cleanup(self): pass
    
    @abstractmethod
    def configure(self, config): pass
    
    @abstractmethod
    def get_status(self): pass
    
    @abstractmethod
    def restart(self): pass
    
    # Too many required methods!

# ✅ GOOD: Minimal interface, optional extensions
class Plugin(ABC):
    @abstractmethod
    def execute(self): 
        """Only essential method required"""
        pass
    
    def initialize(self):
        """Optional - default implementation"""
        pass
    
    def cleanup(self):
        """Optional - default implementation"""
        pass
```

---

## 🔗 Related Patterns

### GRASP Patterns
- **Protected Variations**: Polymorphism is main technique for achieving this
- **Low Coupling**: Polymorphism reduces coupling to specific types
- **High Cohesion**: Each type's behavior is cohesive to that type

### GoF Patterns (Heavy Users of Polymorphism)
- **Strategy**: Family of algorithms using polymorphism
- **State**: Different states using polymorphism
- **Template Method**: Polymorphic operation hooks
- **Factory Method**: Returns polymorphic types
- **Abstract Factory**: Creates families of polymorphic types
- **Command**: Polymorphic command objects
- **Observer**: Polymorphic observers
- **Visitor**: Double dispatch polymorphism

### SOLID Principles
- **Open/Closed Principle**: Achieved through polymorphism
- **Liskov Substitution Principle**: Governs polymorphic inheritance
- **Dependency Inversion**: Depend on polymorphic abstractions

---

## 🎓 Practice Exercises

### Exercise 1: Refactor Type Checking
```python
# Refactor this to use polymorphism:
def calculate_shipping(order):
    if order.shipping_method == 'standard':
        return order.weight * 0.5
    elif order.shipping_method == 'express':
        return order.weight * 1.5
    elif order.shipping_method == 'overnight':
        return order.weight * 3.0
    elif order.shipping_method == 'pickup':
        return 0
```

### Exercise 2: Logger System
Create a logging system with polymorphism:
- FileLogger logs to file
- ConsoleLogger logs to console
- DatabaseLogger logs to database
- NetworkLogger sends logs to remote server

All should work through same interface.

### Exercise 3: Tax Calculator
Design a tax calculation system:
- Different countries have different tax rules
- Should handle: USA, UK, Germany, Japan
- Easy to add new countries
- No type checking in main code

---

## ✅ Checklist

- [ ] I understand what polymorphism means
- [ ] I can identify type-checking anti-patterns
- [ ] I know when to use inheritance vs. duck typing
- [ ] I can design polymorphic interfaces
- [ ] I understand the Open/Closed Principle
- [ ] I avoid forcing polymorphism when not needed
- [ ] I can refactor type-checking code to use polymorphism
- [ ] I've practiced with at least 2 examples

---

**Previous Pattern:** [High Cohesion](05_high_cohesion.md)  
**Next Pattern:** [Pure Fabrication](07_pure_fabrication.md)

**Status:** 🟢 Completed | 🟡 In Progress | 🔴 Not Started
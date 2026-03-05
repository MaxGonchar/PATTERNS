# High Cohesion

> **Category:** GRASP (General Responsibility Assignment Software Patterns)  
> **Type:** Evaluative Pattern (Design Quality Principle)  
> **Difficulty:** ⭐⭐ Intermediate

---

## 📖 Definition

**High Cohesion** is an evaluative principle that guides the assignment of responsibilities to keep classes focused and manageable. It states:

> **Assign responsibilities so that cohesion remains high.**
>
> Use this principle to evaluate alternatives when choosing where to assign responsibility.

**Cohesion** is a measure of how strongly related and focused the responsibilities of a class are.

---

## 🔍 Explanation

High Cohesion answers the question: **"How do we keep classes understandable, manageable, and maintain low coupling as a side effect?"**

### Core Principle
- **Focused Purpose**: Each class should have a clear, single purpose
- **Related Responsibilities**: All methods should work together toward that purpose
- **Manageable Complexity**: Classes should be easy to understand and maintain
- **Natural Grouping**: Related functionality naturally belongs together

### What is Cohesion?

Cohesion measures how well the elements inside a class belong together:
- **High Cohesion**: Class does one thing well, all methods are related
- **Low Cohesion**: Class does many unrelated things, methods are scattered

### Analogy
Think of a Swiss Army knife vs. specialized tools:
- **Low Cohesion**: Swiss Army knife (does many things, but not excellently)
- **High Cohesion**: Specialized screwdriver (does one thing very well)

---

## 🎯 Levels of Cohesion (Worst to Best)

### 1. **Coincidental Cohesion** (Worst) ❌
Elements grouped arbitrarily with no meaningful relationship
```python
class Utilities:
    """Random unrelated methods thrown together"""
    def calculate_tax(self, amount): pass
    def send_email(self, to, subject): pass
    def format_date(self, date): pass
    def save_to_database(self, data): pass
    def generate_random_number(self): pass
```

### 2. **Logical Cohesion** ❌
Elements that perform similar operations but on different data
```python
class InputOutput:
    """All I/O operations together, but different types"""
    def read_file(self, filename): pass
    def write_file(self, filename, data): pass
    def read_database(self, query): pass
    def write_database(self, query): pass
    def read_network(self, url): pass
```

### 3. **Temporal Cohesion** ⚠️
Elements grouped because they execute at the same time
```python
class SystemInitializer:
    """Things that happen at startup"""
    def init(self):
        self.open_database()
        self.load_config()
        self.initialize_cache()
        self.start_logger()
        self.connect_to_api()  # Unrelated concerns
```

### 4. **Procedural Cohesion** ⚠️
Elements grouped because they follow a sequence
```python
class OrderProcessor:
    def process(self):
        self.validate_input()
        self.check_inventory()
        self.calculate_price()
        self.charge_card()
        self.send_email()
        self.update_analytics()  # Sequential but mixed concerns
```

### 5. **Communicational Cohesion** ✅
Elements operate on same data or produce same output
```python
class CustomerReport:
    """All methods work with customer data"""
    def __init__(self, customer):
        self.customer = customer
    
    def get_total_purchases(self): pass
    def get_average_order_value(self): pass
    def get_customer_lifetime_value(self): pass
```

### 6. **Sequential Cohesion** ✅
Output from one element is input to another
```python
class DataPipeline:
    """Each step uses output from previous"""
    def extract_data(self): pass
    def transform_data(self, raw_data): pass
    def load_data(self, transformed_data): pass
```

### 7. **Functional Cohesion** (Best) ✅
All elements contribute to a single, well-defined task
```python
class Invoice:
    """Everything related to invoice behavior"""
    def __init__(self, items):
        self._items = items
    
    def calculate_subtotal(self): pass
    def calculate_tax(self): pass
    def calculate_total(self): pass
    def apply_discount(self, discount): pass
    def generate_pdf(self): pass
```

---

## 🎯 Problem It Solves

### Problems with Low Cohesion:
1. **Hard to Understand**: Too many unrelated responsibilities
2. **Hard to Maintain**: Changes affect unrelated functionality
3. **Hard to Reuse**: Cannot use one part without dragging along unrelated parts
4. **Fragile**: Changes break seemingly unrelated things
5. **Difficult to Test**: Too many things to test at once
6. **God Objects**: Classes that know/do too much

### Example of Low Cohesion:
```python
# ❌ BAD: Low cohesion - God object doing everything

class OrderManager:
    """This class does EVERYTHING - low cohesion"""
    
    def __init__(self):
        self.db_connection = None
        self.email_server = None
    
    # Database operations
    def connect_to_database(self):
        self.db_connection = connect("mysql://...")
    
    def save_order_to_db(self, order):
        self.db_connection.execute("INSERT INTO orders ...")
    
    def query_products(self, product_id):
        return self.db_connection.query("SELECT * FROM products ...")
    
    # Business logic
    def calculate_order_total(self, items):
        total = 0
        for item in items:
            total += item.price * item.quantity
        tax = total * 0.1
        return total + tax
    
    def validate_credit_card(self, card_number):
        # Credit card validation
        return len(card_number) == 16
    
    # Email operations
    def send_order_confirmation(self, email, order_id):
        self.email_server.connect()
        self.email_server.send(email, f"Order {order_id} confirmed")
    
    # UI formatting
    def format_order_summary(self, order):
        return f"""
        <html>
            <body>Order #{order.id}: ${order.total}</body>
        </html>
        """
    
    # File operations
    def export_to_csv(self, orders):
        with open('orders.csv', 'w') as f:
            for order in orders:
                f.write(f"{order.id},{order.total}\n")
    
    # Logging
    def log_error(self, message):
        with open('error.log', 'a') as f:
            f.write(f"{datetime.now()}: {message}\n")
    
    # Analytics
    def track_conversion(self, order):
        # Send to analytics service
        pass
    
    # Inventory management
    def update_stock(self, product_id, quantity):
        self.db_connection.execute("UPDATE products SET stock = ...")

# Problems:
# - Impossible to understand without reading all code
# - Changes to email affect order processing
# - Cannot reuse validation without database code
# - Single class has 10+ reasons to change
# - Testing requires mocking database, email, files, etc.
```

---

## ✅ Solution With High Cohesion

```python
# ✅ GOOD: High cohesion - focused classes

class Order:
    """Focused on order business logic - HIGH COHESION"""
    def __init__(self):
        self._items = []
        self._discount = None
    
    def add_item(self, product, quantity):
        """Related to order management"""
        self._items.append(OrderItem(product, quantity))
    
    def apply_discount(self, discount):
        """Related to order pricing"""
        self._discount = discount
    
    def calculate_subtotal(self):
        """Related to order calculation"""
        return sum(item.get_subtotal() for item in self._items)
    
    def calculate_tax(self):
        """Related to order calculation"""
        subtotal = self.calculate_subtotal()
        return subtotal * 0.1
    
    def calculate_total(self):
        """Related to order calculation"""
        total = self.calculate_subtotal() + self.calculate_tax()
        if self._discount:
            total = self._discount.apply(total)
        return total


class OrderRepository:
    """Focused on order persistence - HIGH COHESION"""
    def __init__(self, db_connection):
        self._db = db_connection
    
    def save(self, order):
        """Related to persistence"""
        self._db.execute("INSERT INTO orders ...")
    
    def find_by_id(self, order_id):
        """Related to persistence"""
        row = self._db.query("SELECT * FROM orders WHERE id = ?", order_id)
        return self._map_to_order(row)
    
    def find_by_customer(self, customer_id):
        """Related to persistence"""
        rows = self._db.query("SELECT * FROM orders WHERE customer_id = ?", customer_id)
        return [self._map_to_order(row) for row in rows]
    
    def _map_to_order(self, row):
        """Related to persistence - internal helper"""
        return Order(row['id'], row['items'])


class EmailNotificationService:
    """Focused on email notifications - HIGH COHESION"""
    def __init__(self, email_server):
        self._server = email_server
    
    def send_order_confirmation(self, order, customer_email):
        """Related to email notifications"""
        subject = "Order Confirmation"
        body = self._create_confirmation_body(order)
        self._server.send(customer_email, subject, body)
    
    def send_shipping_notification(self, order, customer_email):
        """Related to email notifications"""
        subject = "Your Order Has Shipped"
        body = self._create_shipping_body(order)
        self._server.send(customer_email, subject, body)
    
    def _create_confirmation_body(self, order):
        """Related to email content - internal helper"""
        return f"Your order #{order.id} for ${order.total} is confirmed"
    
    def _create_shipping_body(self, order):
        """Related to email content - internal helper"""
        return f"Your order #{order.id} has shipped"


class CreditCardValidator:
    """Focused on credit card validation - HIGH COHESION"""
    def validate_number(self, card_number):
        """Related to card validation"""
        return self._luhn_check(card_number)
    
    def validate_expiry(self, month, year):
        """Related to card validation"""
        expiry = datetime(year, month, 1)
        return expiry > datetime.now()
    
    def validate_cvv(self, cvv):
        """Related to card validation"""
        return len(cvv) == 3 and cvv.isdigit()
    
    def _luhn_check(self, card_number):
        """Related to card validation - internal algorithm"""
        # Luhn algorithm implementation
        pass


class OrderController:
    """Focused on coordinating order workflow - HIGH COHESION"""
    def __init__(self, order_repo, notification_service, payment_processor):
        self._order_repo = order_repo
        self._notification_service = notification_service
        self._payment_processor = payment_processor
    
    def place_order(self, order, payment_details):
        """Coordinates order placement"""
        # All methods related to coordinating order workflow
        self._validate_order(order)
        self._process_payment(order, payment_details)
        self._save_order(order)
        self._send_confirmation(order)
    
    def _validate_order(self, order):
        """Related to order coordination"""
        if order.get_total() <= 0:
            raise ValueError("Invalid order total")
    
    def _process_payment(self, order, payment_details):
        """Related to order coordination"""
        self._payment_processor.charge(order.get_total(), payment_details)
    
    def _save_order(self, order):
        """Related to order coordination"""
        self._order_repo.save(order)
    
    def _send_confirmation(self, order):
        """Related to order coordination"""
        self._notification_service.send_order_confirmation(
            order, 
            order.customer.email
        )
```

---

## 🐍 Practical Python Examples

### Example 1: User Management - High Cohesion

```python
# Low Cohesion ❌
class UserManager:
    """Does too many unrelated things"""
    def create_user(self, name, email): pass
    def send_welcome_email(self, email): pass
    def hash_password(self, password): pass
    def log_to_file(self, message): pass
    def export_users_csv(self): pass
    def validate_email_format(self, email): pass
    def generate_report(self): pass


# High Cohesion ✅
class User:
    """Focused on user entity and behavior"""
    def __init__(self, name, email, password_hash):
        self.name = name
        self.email = email
        self._password_hash = password_hash
        self.is_active = True
    
    def authenticate(self, password):
        """Related to user authentication"""
        return PasswordHasher.verify(password, self._password_hash)
    
    def update_profile(self, name=None, email=None):
        """Related to user data"""
        if name:
            self.name = name
        if email:
            self.email = email
    
    def deactivate(self):
        """Related to user status"""
        self.is_active = False


class UserRepository:
    """Focused on user persistence"""
    def __init__(self, db):
        self._db = db
    
    def save(self, user):
        """Persistence operation"""
        pass
    
    def find_by_id(self, user_id):
        """Persistence operation"""
        pass
    
    def find_by_email(self, email):
        """Persistence operation"""
        pass
    
    def delete(self, user_id):
        """Persistence operation"""
        pass


class PasswordHasher:
    """Focused on password hashing"""
    @staticmethod
    def hash(password):
        """Hashing operation"""
        import hashlib
        return hashlib.sha256(password.encode()).hexdigest()
    
    @staticmethod
    def verify(password, hash_value):
        """Hashing operation"""
        return PasswordHasher.hash(password) == hash_value


class EmailValidator:
    """Focused on email validation"""
    @staticmethod
    def is_valid(email):
        """Validation operation"""
        import re
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(pattern, email) is not None


class UserRegistrationService:
    """Focused on registration workflow"""
    def __init__(self, user_repo, email_service, email_validator):
        self._user_repo = user_repo
        self._email_service = email_service
        self._email_validator = email_validator
    
    def register(self, name, email, password):
        """Registration workflow coordination"""
        self._validate_input(name, email, password)
        user = self._create_user(name, email, password)
        self._save_user(user)
        self._send_welcome_email(user)
        return user
    
    def _validate_input(self, name, email, password):
        if not self._email_validator.is_valid(email):
            raise ValueError("Invalid email")
        if len(password) < 8:
            raise ValueError("Password too short")
    
    def _create_user(self, name, email, password):
        password_hash = PasswordHasher.hash(password)
        return User(name, email, password_hash)
    
    def _save_user(self, user):
        self._user_repo.save(user)
    
    def _send_welcome_email(self, user):
        self._email_service.send_welcome_email(user.email)
```

### Example 2: E-commerce Product - High Cohesion

```python
from decimal import Decimal
from typing import List
from dataclasses import dataclass

# High Cohesion Example
class Product:
    """Highly cohesive - all methods relate to product behavior"""
    def __init__(self, name, base_price: Decimal, category):
        self.name = name
        self._base_price = base_price
        self.category = category
        self._discount_percentage = 0
    
    def get_price(self) -> Decimal:
        """Price calculation - related to product pricing"""
        if self._discount_percentage > 0:
            discount = self._base_price * (self._discount_percentage / 100)
            return self._base_price - discount
        return self._base_price
    
    def apply_discount(self, percentage: int):
        """Discount management - related to product pricing"""
        if 0 <= percentage <= 100:
            self._discount_percentage = percentage
        else:
            raise ValueError("Discount must be between 0 and 100")
    
    def remove_discount(self):
        """Discount management - related to product pricing"""
        self._discount_percentage = 0
    
    def is_on_sale(self) -> bool:
        """Sale status - related to product pricing"""
        return self._discount_percentage > 0
    
    def get_discount_amount(self) -> Decimal:
        """Price calculation - related to product pricing"""
        return self._base_price - self.get_price()


class ProductInventory:
    """Highly cohesive - all methods relate to inventory management"""
    def __init__(self, product_id: int):
        self.product_id = product_id
        self._quantity = 0
        self._reserved = 0
        self._reorder_level = 10
    
    def add_stock(self, quantity: int):
        """Stock management"""
        self._quantity += quantity
    
    def remove_stock(self, quantity: int):
        """Stock management"""
        if self.get_available() < quantity:
            raise ValueError("Insufficient stock")
        self._quantity -= quantity
    
    def reserve(self, quantity: int):
        """Reservation management"""
        if self.get_available() < quantity:
            raise ValueError("Insufficient stock to reserve")
        self._reserved += quantity
    
    def release_reservation(self, quantity: int):
        """Reservation management"""
        self._reserved -= quantity
    
    def confirm_reservation(self, quantity: int):
        """Reservation management"""
        self._reserved -= quantity
        self._quantity -= quantity
    
    def get_available(self) -> int:
        """Stock inquiry"""
        return self._quantity - self._reserved
    
    def needs_reorder(self) -> bool:
        """Stock inquiry"""
        return self._quantity <= self._reorder_level
    
    def set_reorder_level(self, level: int):
        """Configuration"""
        self._reorder_level = level


class ProductReview:
    """Highly cohesive - all methods relate to product reviews"""
    def __init__(self, product_id: int):
        self.product_id = product_id
        self._reviews: List[dict] = []
    
    def add_review(self, user_id: int, rating: int, comment: str):
        """Review management"""
        if not 1 <= rating <= 5:
            raise ValueError("Rating must be between 1 and 5")
        
        review = {
            'user_id': user_id,
            'rating': rating,
            'comment': comment,
            'created_at': datetime.now()
        }
        self._reviews.append(review)
    
    def get_average_rating(self) -> float:
        """Review statistics"""
        if not self._reviews:
            return 0.0
        return sum(r['rating'] for r in self._reviews) / len(self._reviews)
    
    def get_rating_distribution(self) -> dict:
        """Review statistics"""
        distribution = {1: 0, 2: 0, 3: 0, 4: 0, 5: 0}
        for review in self._reviews:
            distribution[review['rating']] += 1
        return distribution
    
    def get_reviews_by_rating(self, rating: int) -> List[dict]:
        """Review filtering"""
        return [r for r in self._reviews if r['rating'] == rating]
    
    def get_recent_reviews(self, limit: int = 10) -> List[dict]:
        """Review filtering"""
        sorted_reviews = sorted(
            self._reviews,
            key=lambda r: r['created_at'],
            reverse=True
        )
        return sorted_reviews[:limit]


# Each class has high cohesion - focused on one aspect
product = Product("Laptop", Decimal("999.99"), "Electronics")
inventory = ProductInventory(product_id=1)
reviews = ProductReview(product_id=1)

# Each can evolve independently
product.apply_discount(10)
inventory.add_stock(50)
reviews.add_review(user_id=123, rating=5, comment="Great!")
```

### Example 3: File Processor - Refactoring for Cohesion

```python
# Low Cohesion ❌
class FileProcessor:
    """Does too many different things"""
    def read_csv(self, filename):
        # CSV reading logic
        pass
    
    def write_csv(self, filename, data):
        # CSV writing logic
        pass
    
    def validate_email(self, email):
        # Email validation
        pass
    
    def calculate_statistics(self, numbers):
        # Statistical calculations
        pass
    
    def send_error_notification(self, error):
        # Email notification
        pass
    
    def compress_file(self, filename):
        # File compression
        pass


# High Cohesion ✅
class CSVReader:
    """Focused on reading CSV files"""
    def read(self, filename: str) -> List[dict]:
        """Read CSV and return list of dicts"""
        import csv
        with open(filename, 'r') as f:
            reader = csv.DictReader(f)
            return list(reader)
    
    def read_filtered(self, filename: str, filter_fn) -> List[dict]:
        """Read and filter rows"""
        rows = self.read(filename)
        return [row for row in rows if filter_fn(row)]
    
    def get_column(self, filename: str, column_name: str) -> List:
        """Extract specific column"""
        rows = self.read(filename)
        return [row[column_name] for row in rows]


class CSVWriter:
    """Focused on writing CSV files"""
    def write(self, filename: str, data: List[dict], fieldnames: List[str]):
        """Write data to CSV"""
        import csv
        with open(filename, 'w', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=fieldnames)
            writer.writeheader()
            writer.writerows(data)
    
    def append(self, filename: str, row: dict):
        """Append single row"""
        import csv
        with open(filename, 'a', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=row.keys())
            writer.writerow(row)


class StatisticsCalculator:
    """Focused on statistical calculations"""
    @staticmethod
    def mean(numbers: List[float]) -> float:
        """Calculate mean"""
        return sum(numbers) / len(numbers)
    
    @staticmethod
    def median(numbers: List[float]) -> float:
        """Calculate median"""
        sorted_numbers = sorted(numbers)
        n = len(sorted_numbers)
        mid = n // 2
        if n % 2 == 0:
            return (sorted_numbers[mid-1] + sorted_numbers[mid]) / 2
        return sorted_numbers[mid]
    
    @staticmethod
    def standard_deviation(numbers: List[float]) -> float:
        """Calculate standard deviation"""
        mean = StatisticsCalculator.mean(numbers)
        variance = sum((x - mean) ** 2 for x in numbers) / len(numbers)
        return variance ** 0.5


class FileCompressor:
    """Focused on file compression"""
    def compress(self, input_file: str, output_file: str):
        """Compress file using gzip"""
        import gzip
        import shutil
        with open(input_file, 'rb') as f_in:
            with gzip.open(output_file, 'wb') as f_out:
                shutil.copyfileobj(f_in, f_out)
    
    def decompress(self, input_file: str, output_file: str):
        """Decompress gzip file"""
        import gzip
        import shutil
        with gzip.open(input_file, 'rb') as f_in:
            with open(output_file, 'wb') as f_out:
                shutil.copyfileobj(f_in, f_out)
    
    def get_compression_ratio(self, original_file: str, compressed_file: str) -> float:
        """Calculate compression ratio"""
        import os
        original_size = os.path.getsize(original_file)
        compressed_size = os.path.getsize(compressed_file)
        return compressed_size / original_size


# Each class is focused and cohesive
csv_reader = CSVReader()
statistics = StatisticsCalculator()
compressor = FileCompressor()

data = csv_reader.read('sales.csv')
prices = [float(row['price']) for row in data]
average = statistics.mean(prices)
```

---

## 🏢 Examples in Existing Frameworks

### 1. **Python's `pathlib.Path`**
```python
from pathlib import Path

# High cohesion - all methods relate to path operations
path = Path('/usr/local/bin')

# All path-related operations
path.exists()          # Path inquiry
path.is_file()         # Path inquiry
path.is_dir()          # Path inquiry
path.parent            # Path navigation
path.name              # Path component
path.suffix            # Path component
path.stem              # Path component
path.joinpath('file')  # Path manipulation
path.resolve()         # Path manipulation
```

### 2. **Django Models**
```python
from django.db import models

class Article(models.Model):
    """High cohesion - all methods relate to Article entity"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    published_date = models.DateTimeField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    
    # All methods relate to Article behavior
    def publish(self):
        """Article state management"""
        self.published_date = timezone.now()
        self.save()
    
    def is_published(self):
        """Article inquiry"""
        return self.published_date <= timezone.now()
    
    def get_word_count(self):
        """Article analysis"""
        return len(self.content.split())
    
    def get_read_time(self):
        """Article analysis"""
        words = self.get_word_count()
        return words // 200  # Assuming 200 words per minute
```

### 3. **Python's `datetime`**
```python
from datetime import datetime, timedelta

# High cohesion - all datetime operations in one class
now = datetime.now()

# All relate to datetime manipulation/inquiry
now.year               # Component access
now.month              # Component access
now.strftime('%Y-%m-%d')  # Formatting
now.isoformat()        # Formatting
now.replace(year=2025) # Manipulation
now + timedelta(days=1)  # Manipulation
now.weekday()          # Inquiry
```

### 4. **Requests Session**
```python
import requests

# High cohesion - all methods relate to HTTP session management
session = requests.Session()

# All HTTP session operations
session.get(url)           # HTTP request
session.post(url, data)    # HTTP request
session.put(url, data)     # HTTP request
session.delete(url)        # HTTP request
session.headers.update()   # Session configuration
session.cookies.set()      # Session state
session.close()            # Session lifecycle
```

### 5. **SQLAlchemy Query**
```python
from sqlalchemy.orm import Session

session = Session()

# High cohesion - all methods relate to building/executing queries
query = session.query(User)

# All query-related operations
query.filter(User.age > 18)    # Query building
query.order_by(User.name)      # Query building
query.limit(10)                # Query building
query.offset(20)               # Query building
query.all()                    # Query execution
query.first()                  # Query execution
query.count()                  # Query execution
```

---

## ✅ Pros and Cons

### Pros ✅

1. **Understandability**
   - Easy to understand what a class does
   - Clear, focused purpose

2. **Maintainability**
   - Changes localized to related functionality
   - Less risk of breaking unrelated things

3. **Reusability**
   - Focused classes easier to reuse
   - No unnecessary dependencies dragged along

4. **Testability**
   - Smaller, focused classes easier to test
   - Fewer test cases needed per class

5. **Low Coupling (Side Effect)**
   - High cohesion naturally leads to low coupling
   - Focused classes have fewer dependencies

6. **Robustness**
   - Fewer reasons to change
   - More stable interfaces

### Cons ❌

1. **More Classes**
   - Increased number of classes
   - More files to navigate

2. **Complexity Distribution**
   - Complexity moves from inside classes to relationships
   - Need to understand how classes collaborate

3. **Over-Granularity**
   - Too many tiny classes can be confusing
   - Balance needed

4. **Initial Design Effort**
   - Requires thinking about responsibilities
   - May need refactoring as understanding grows

5. **Performance Considerations**
   - More objects, more method calls
   - Usually negligible in practice

---

## 💡 When to Use

### ✅ Strive for High Cohesion When:

1. **Class Has Multiple Unrelated Responsibilities**
   - God object with many concerns
   - Example: Class handling UI, business logic, and database

2. **Methods Don't Use Same Data**
   - Methods work on different instance variables
   - Example: Half the methods use one field, half use another

3. **Frequent Changes to Unrelated Parts**
   - Changes in one area break unrelated functionality
   - Example: Email changes breaking order calculation

4. **Difficult to Name the Class**
   - Name includes "And", "Manager", "Handler", "Utility"
   - Example: `UserAndOrderAndPaymentManager`

5. **Large Classes (High LOC)**
   - Over 300-500 lines
   - Many methods (>15-20)

6. **Testing Requires Many Mocks**
   - Need to mock many unrelated dependencies
   - Hard to test one feature in isolation

### ⚠️ Lower Cohesion Acceptable When:

1. **Facade Pattern**
   - Intentionally simplifying complex subsystem
   - Example: `SystemFacade` coordinating many subsystems

2. **Utility Classes**
   - Static helper methods (use sparingly)
   - Example: `StringUtils`, `MathUtils`

3. **Framework Requirements**
   - Framework imposes structure
   - Example: Django view methods in one class

4. **Small, Simple Classes**
   - Very small classes (< 100 lines)
   - Few responsibilities

---

## 🚫 Common Mistakes

### Mistake 1: God Objects
```python
# ❌ BAD: One class does everything
class Application:
    def __init__(self):
        self.users = []
        self.products = []
        self.orders = []
        self.config = {}
    
    # User management (20 methods)
    def create_user(self): pass
    def delete_user(self): pass
    def authenticate_user(self): pass
    # ...
    
    # Product management (20 methods)
    def add_product(self): pass
    def update_inventory(self): pass
    # ...
    
    # Order processing (20 methods)
    def place_order(self): pass
    def process_payment(self): pass
    # ...
    
    # Email (10 methods)
    def send_email(self): pass
    def send_sms(self): pass
    # ...
    
    # Reporting (15 methods)
    def generate_sales_report(self): pass
    # ...
    
    # 85+ methods total!

# ✅ GOOD: Separate specialized classes
class UserService: pass
class ProductCatalog: pass
class OrderProcessor: pass
class NotificationService: pass
class ReportGenerator: pass
```

### Mistake 2: Utility Classes Dumping Ground
```python
# ❌ BAD: Random unrelated utility methods
class Utils:
    @staticmethod
    def format_date(date): pass
    
    @staticmethod
    def calculate_tax(amount): pass
    
    @staticmethod
    def send_email(to, subject): pass
    
    @staticmethod
    def hash_password(password): pass
    
    @staticmethod
    def validate_credit_card(number): pass

# ✅ GOOD: Separate focused utilities
class DateFormatter:
    @staticmethod
    def format(date, format_string): pass

class TaxCalculator:
    @staticmethod
    def calculate(amount, rate): pass

class PasswordHasher:
    @staticmethod
    def hash(password): pass
    @staticmethod
    def verify(password, hash): pass
```

### Mistake 3: Methods Not Using Instance State
```python
# ❌ BAD: Methods don't use instance variables
class OrderProcessor:
    def __init__(self, db):
        self.db = db
    
    def validate_email(self, email):
        # Doesn't use self.db or any instance variable
        return '@' in email
    
    def format_currency(self, amount):
        # Doesn't use any instance variable
        return f"${amount:.2f}"
    
    def calculate_percentage(self, value, percentage):
        # Doesn't use any instance variable
        return value * (percentage / 100)

# ✅ GOOD: Extract to appropriate places
class EmailValidator:
    @staticmethod
    def is_valid(email):
        return '@' in email

class CurrencyFormatter:
    @staticmethod
    def format(amount):
        return f"${amount:.2f}"

class OrderProcessor:
    def __init__(self, db, email_validator, formatter):
        self.db = db
        self._email_validator = email_validator
        self._formatter = formatter
    
    def process_order(self, order):
        # Now all methods use instance state
        if not self._email_validator.is_valid(order.email):
            raise ValueError()
        
        self.db.save(order)
        formatted = self._formatter.format(order.total)
```

### Mistake 4: Over-Cohesion (Too Granular)
```python
# ❌ BAD: Over-engineered, too granular
class UserFirstName:
    def __init__(self, first_name):
        self.value = first_name
    
    def get(self):
        return self.value

class UserLastName:
    def __init__(self, last_name):
        self.value = last_name
    
    def get(self):
        return self.value

class UserEmail:
    def __init__(self, email):
        self.value = email
    
    def get(self):
        return self.value

# Too many tiny classes!

# ✅ GOOD: Appropriate granularity
class User:
    def __init__(self, first_name, last_name, email):
        self.first_name = first_name
        self.last_name = last_name
        self.email = email
    
    def get_full_name(self):
        return f"{self.first_name} {self.last_name}"
```

---

## 🔗 Related Patterns

### GRASP Patterns
- **Low Coupling**: High cohesion naturally leads to low coupling
- **Information Expert**: Helps achieve cohesion by keeping data and behavior together
- **Pure Fabrication**: Creates cohesive service classes when domain objects can't handle it

### SOLID Principles
- **Single Responsibility Principle (SRP)**: Very similar to High Cohesion
  - SRP: "A class should have only one reason to change"
  - High Cohesion: "A class should have focused, related responsibilities"

### GoF Patterns
- **Facade**: Intentionally lower cohesion to simplify interface
- **Adapter**: Keeps adapters cohesive to one adaptation concern
- **Strategy**: Each strategy is highly cohesive to one algorithm

---

## 📊 Measuring Cohesion

### LCOM (Lack of Cohesion of Methods)
Measures how related methods are based on shared instance variables:

```python
# High cohesion (low LCOM) - all methods use same variables
class Rectangle:
    def __init__(self, width, height):
        self.width = width    # Used by all methods
        self.height = height  # Used by all methods
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)
    
    def scale(self, factor):
        self.width *= factor
        self.height *= factor


# Low cohesion (high LCOM) - methods use different variables
class UserAndProduct:
    def __init__(self, user_name, product_name):
        self.user_name = user_name      # Only used by user methods
        self.product_name = product_name  # Only used by product methods
    
    def get_user_info(self):
        return self.user_name  # Uses user_name only
    
    def get_product_info(self):
        return self.product_name  # Uses product_name only
    
    # Two separate concerns! Should be two classes.
```

---

## 🎓 Practice Exercises

### Exercise 1: Refactor God Object
```python
# Refactor this low cohesion class:
class WebShop:
    def create_user(self, name, email): pass
    def authenticate(self, email, password): pass
    def add_product_to_catalog(self, product): pass
    def get_product_price(self, product_id): pass
    def place_order(self, user_id, items): pass
    def calculate_shipping(self, order): pass
    def send_confirmation_email(self, order): pass
    def generate_invoice_pdf(self, order): pass
    def process_refund(self, order_id): pass
    def update_inventory(self, product_id, quantity): pass
```

**Task**: Split into 4-6 cohesive classes.

### Exercise 2: Identify Cohesion Violations
Review each class and identify cohesion issues:
```python
class ReportManager:
    def fetch_data_from_database(self): pass
    def calculate_statistics(self, data): pass
    def format_as_html(self, stats): pass
    def send_via_email(self, html): pass
    def log_to_file(self, message): pass
```

### Exercise 3: Design Cohesive Classes
Design a library system with high cohesion:
- Books, Members, Loans
- Ensure each class has focused responsibilities
- Identify potential cohesion problems

---

## ✅ Checklist

- [ ] I understand what cohesion is and why it matters
- [ ] I can identify low cohesion code smells (God objects, utility dumping grounds)
- [ ] I know the different levels of cohesion
- [ ] I understand the relationship between cohesion and coupling
- [ ] I can refactor low-cohesion classes into focused ones
- [ ] I know when lower cohesion is acceptable (facades, utilities)
- [ ] I avoid over-granularity (too many tiny classes)
- [ ] I've refactored at least 2 examples for higher cohesion

---

**Previous Pattern:** [Low Coupling](04_low_coupling.md)  
**Next Pattern:** [Polymorphism](06_polymorphism.md)

**Status:** 🟢 Completed | 🟡 In Progress | 🔴 Not Started
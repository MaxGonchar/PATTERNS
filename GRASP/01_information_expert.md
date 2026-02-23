# Information Expert

> **Category:** GRASP (General Responsibility Assignment Software Patterns)  
> **Type:** Responsibility Assignment Pattern  
> **Difficulty:** ⭐ Beginner

---

## 📖 Definition

**Information Expert** is a principle for assigning responsibilities to classes. It states:

> **Assign a responsibility to the class that has the information necessary to fulfill it.**

In other words: the class that has the data should have the behavior that works with that data.

---

## 🔍 Explanation

The Information Expert pattern is about **placing methods where the data lives**. When you need to perform an operation, assign that responsibility to the class that possesses the information required to carry it out.

### Core Principle
- **Encapsulation**: Keep data and behavior together
- **Cohesion**: Classes should be responsible for their own data
- **Natural Design**: Follows intuitive object-oriented thinking

### The Question to Ask
When assigning a responsibility, ask yourself:
> "Which class has the information needed to fulfill this responsibility?"

The answer points to your Information Expert.

---

## 🎯 Problem It Solves

### Problems Without Information Expert:
1. **Scattered Logic**: Business logic spread across multiple unrelated classes
2. **Poor Encapsulation**: External classes accessing internal data
3. **High Coupling**: Classes depend on implementation details of others
4. **Difficult Maintenance**: Changes require updates in many places
5. **Violation of Tell, Don't Ask**: Classes asking for data instead of asking for behavior

### Example of Bad Design:
```python
# ❌ BAD: Order class exposes data, calculation done elsewhere
class Order:
    def __init__(self):
        self.items = []
    
    def get_items(self):
        return self.items

# Logic scattered in another class
class OrderService:
    def calculate_total(self, order):
        total = 0
        for item in order.get_items():  # Asking for data
            total += item.price * item.quantity
        return total
```

---

## ✅ Solution With Information Expert

```python
# ✅ GOOD: Order has the information, so it calculates the total
class Order:
    def __init__(self):
        self.items = []
    
    def add_item(self, item):
        self.items.append(item)
    
    def get_total(self):
        """Order is the Information Expert for calculating its own total"""
        return sum(item.get_subtotal() for item in self.items)


class OrderItem:
    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity
    
    def get_subtotal(self):
        """OrderItem is the Information Expert for calculating its subtotal"""
        return self.product.price * self.quantity


class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
```

---

## 🐍 Practical Python Examples

### Example 1: Shopping Cart

```python
class ShoppingCart:
    def __init__(self):
        self._items = []
    
    def add_product(self, product, quantity):
        """Add product to cart"""
        self._items.append({'product': product, 'quantity': quantity})
    
    def get_total_price(self):
        """ShoppingCart is the expert - it has all items"""
        return sum(
            item['product'].price * item['quantity'] 
            for item in self._items
        )
    
    def get_item_count(self):
        """ShoppingCart knows how many items it contains"""
        return sum(item['quantity'] for item in self._items)
    
    def has_product(self, product):
        """ShoppingCart knows if it contains a product"""
        return any(
            item['product'] == product 
            for item in self._items
        )


class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price


# Usage
cart = ShoppingCart()
laptop = Product("Laptop", 1200)
mouse = Product("Mouse", 25)

cart.add_product(laptop, 1)
cart.add_product(mouse, 2)

print(f"Total: ${cart.get_total_price()}")  # Cart calculates its own total
print(f"Items: {cart.get_item_count()}")     # Cart counts its own items
```

### Example 2: Student Grade Management

```python
class Student:
    def __init__(self, name, student_id):
        self.name = name
        self.student_id = student_id
        self._grades = []
    
    def add_grade(self, grade):
        """Student manages their own grades"""
        self._grades.append(grade)
    
    def get_average_grade(self):
        """Student is the expert - knows all their grades"""
        if not self._grades:
            return 0
        return sum(self._grades) / len(self._grades)
    
    def is_passing(self, passing_grade=60):
        """Student knows if they're passing"""
        return self.get_average_grade() >= passing_grade
    
    def get_letter_grade(self):
        """Student calculates their own letter grade"""
        avg = self.get_average_grade()
        if avg >= 90: return 'A'
        if avg >= 80: return 'B'
        if avg >= 70: return 'C'
        if avg >= 60: return 'D'
        return 'F'


# Usage
student = Student("Alice", "S001")
student.add_grade(85)
student.add_grade(92)
student.add_grade(88)

print(f"Average: {student.get_average_grade():.1f}")
print(f"Letter Grade: {student.get_letter_grade()}")
print(f"Passing: {student.is_passing()}")
```

### Example 3: Bank Account

```python
from datetime import datetime
from enum import Enum

class TransactionType(Enum):
    DEPOSIT = "deposit"
    WITHDRAWAL = "withdrawal"

class Transaction:
    def __init__(self, amount, transaction_type, description=""):
        self.amount = amount
        self.type = transaction_type
        self.timestamp = datetime.now()
        self.description = description
    
    def get_signed_amount(self):
        """Transaction knows its impact on balance"""
        if self.type == TransactionType.WITHDRAWAL:
            return -self.amount
        return self.amount


class BankAccount:
    def __init__(self, account_number, owner):
        self.account_number = account_number
        self.owner = owner
        self._transactions = []
    
    def deposit(self, amount, description=""):
        """BankAccount manages its own transactions"""
        transaction = Transaction(amount, TransactionType.DEPOSIT, description)
        self._transactions.append(transaction)
    
    def withdraw(self, amount, description=""):
        """BankAccount decides if withdrawal is allowed"""
        if self.get_balance() >= amount:
            transaction = Transaction(amount, TransactionType.WITHDRAWAL, description)
            self._transactions.append(transaction)
            return True
        return False
    
    def get_balance(self):
        """BankAccount is the expert - knows all transactions"""
        return sum(t.get_signed_amount() for t in self._transactions)
    
    def get_transaction_history(self):
        """BankAccount provides its own history"""
        return [
            f"{t.timestamp}: {t.type.value} ${t.amount} - {t.description}"
            for t in self._transactions
        ]


# Usage
account = BankAccount("ACC123", "John Doe")
account.deposit(1000, "Initial deposit")
account.withdraw(250, "ATM withdrawal")
account.deposit(500, "Salary")

print(f"Balance: ${account.get_balance()}")
for record in account.get_transaction_history():
    print(record)
```

---

## 🏢 Examples in Existing Frameworks

### 1. **Django ORM (Python)**
```python
# Django models are Information Experts for their own data

from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    published_date = models.DateTimeField()
    views = models.IntegerField(default=0)
    
    def is_published(self):
        """Article knows if it's published"""
        return self.published_date <= timezone.now()
    
    def increment_views(self):
        """Article manages its own view count"""
        self.views += 1
        self.save()
    
    class Meta:
        ordering = ['-published_date']
```

### 2. **Python's datetime**
```python
from datetime import datetime

# datetime is the Information Expert for time calculations
now = datetime.now()

# It knows how to format itself
formatted = now.strftime("%Y-%m-%d")

# It knows how to compare with other dates
is_future = now > datetime(2020, 1, 1)

# It knows its own components
year = now.year
month = now.month
```

### 3. **Python's list/dict**
```python
# Built-in collections are experts about their contents

my_list = [1, 2, 3, 4, 5]

# List knows its length
length = my_list.len()  # or len(my_list)

# List knows if it contains an item
has_three = 3 in my_list

# List knows how to sort itself
my_list.sort()

# Dict knows its keys
my_dict = {'a': 1, 'b': 2}
keys = my_dict.keys()
```

### 4. **Flask Request Object**
```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/api/data', methods=['POST'])
def handle_data():
    # Request object is the expert about the incoming request
    
    # It knows the HTTP method
    if request.method == 'POST':
        pass
    
    # It knows the JSON data
    data = request.get_json()
    
    # It knows the headers
    auth_header = request.headers.get('Authorization')
    
    # It knows the client's IP
    client_ip = request.remote_addr
```

### 5. **Pandas DataFrame**
```python
import pandas as pd

df = pd.DataFrame({
    'age': [25, 30, 35, 40],
    'salary': [50000, 60000, 70000, 80000]
})

# DataFrame is the expert about its data
mean_age = df['age'].mean()
max_salary = df['salary'].max()
sorted_df = df.sort_values('age')
filtered = df[df['age'] > 30]
```

---

## ✅ Pros and Cons

### Pros ✅

1. **High Cohesion**
   - Related data and behavior stay together
   - Classes are more focused and understandable

2. **Better Encapsulation**
   - Internal data structures are hidden
   - Implementation can change without affecting clients

3. **Reduced Coupling**
   - Other classes don't need to know internal details
   - Changes are localized

4. **Easier Maintenance**
   - Logic is in one place
   - Bugs are easier to find and fix

5. **More Intuitive Design**
   - Follows object-oriented principles naturally
   - Code reads more naturally

6. **Reusability**
   - Expert classes can be used in different contexts
   - Behavior moves with the data

### Cons ❌

1. **May Violate Single Responsibility**
   - If taken too far, classes can become bloated
   - Need to balance with other patterns

2. **Can Lead to God Objects**
   - Classes that know "too much" become hard to maintain
   - Need to watch for classes that grow too large

3. **May Require Multiple Experts**
   - Complex operations might need collaboration
   - Information might be distributed across objects

4. **Database Operations**
   - Domain objects shouldn't handle persistence
   - Need to balance with Repository or Data Mapper patterns

5. **Cross-Cutting Concerns**
   - Logging, security, etc., shouldn't be in every expert
   - Use other patterns (Decorator, Aspect) for these

---

## 💡 When to Use

### ✅ Use Information Expert When:

1. **Clear Data Ownership**
   - One class clearly owns the data needed for an operation
   - Example: `Order` calculating its own total

2. **Encapsulation is Important**
   - You want to hide internal data structures
   - Example: `BankAccount` managing its balance

3. **Business Logic is Data-Specific**
   - Logic directly relates to the data it operates on
   - Example: `Student` calculating their GPA

4. **Building Domain Models**
   - Creating rich domain objects with behavior
   - Example: E-commerce entities with business logic

5. **Single Object Operations**
   - Operation only needs data from one object
   - Example: `Rectangle` calculating its own area

### ❌ Don't Use Information Expert When:

1. **Cross-Cutting Concerns**
   - Logging, authentication, caching
   - Use: Aspect-Oriented Programming or Decorators

2. **Complex Calculations Requiring Multiple Sources**
   - Data is distributed across many objects
   - Use: Service classes or Command pattern

3. **Persistence Logic**
   - Saving/loading from database
   - Use: Repository or Data Mapper pattern

4. **External System Integration**
   - Calling APIs, sending emails
   - Use: Service classes or Gateway pattern

5. **UI Logic**
   - Rendering, validation for specific UI
   - Use: Presenter or ViewModel pattern

6. **When It Creates God Objects**
   - Class becomes too large and complex
   - Use: Split into multiple classes, use Strategy pattern

---

## 🚫 Common Mistakes

### Mistake 1: Asking Instead of Telling
```python
# ❌ BAD: Violating Information Expert
class Invoice:
    def __init__(self):
        self.items = []

# External class asks for data and does calculation
total = 0
for item in invoice.items:
    total += item.price * item.quantity

# ✅ GOOD: Tell the expert to do it
total = invoice.calculate_total()
```

### Mistake 2: Creating Anemic Domain Models
```python
# ❌ BAD: Just a data container
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
        self.is_active = True

# Logic elsewhere
def activate_user(user):
    user.is_active = True

# ✅ GOOD: Rich domain model
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
        self._is_active = False
    
    def activate(self):
        """User is the expert about activation"""
        self._is_active = True
    
    def deactivate(self):
        self._is_active = False
    
    def is_active(self):
        return self._is_active
```

### Mistake 3: Ignoring Separation of Concerns
```python
# ❌ BAD: Mixing concerns
class User:
    def save_to_database(self):
        # Database logic in domain object
        connection = get_db_connection()
        connection.execute("INSERT INTO users...")

# ✅ GOOD: Separate persistence
class User:
    def get_full_name(self):
        return f"{self.first_name} {self.last_name}"

class UserRepository:
    def save(self, user):
        # Repository handles persistence
        pass
```

---

## 🔗 Related GRASP Patterns

- **Low Coupling**: Information Expert reduces coupling by keeping data and behavior together
- **High Cohesion**: Promotes cohesive classes with focused responsibilities
- **Creator**: Often works with Information Expert to decide who creates objects
- **Controller**: Complements Information Expert for system operations

---

## 📊 Before and After Comparison

### Before (Procedural Style)
```python
# Data and behavior separated
class Customer:
    def __init__(self, name):
        self.name = name
        self.orders = []

class Order:
    def __init__(self):
        self.items = []

class OrderItem:
    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity

# Logic scattered in functions
def calculate_order_total(order):
    total = 0
    for item in order.items:
        total += item.product.price * item.quantity
    return total

def calculate_customer_lifetime_value(customer):
    total = 0
    for order in customer.orders:
        total += calculate_order_total(order)
    return total
```

### After (Information Expert)
```python
class Customer:
    def __init__(self, name):
        self.name = name
        self._orders = []
    
    def add_order(self, order):
        self._orders.append(order)
    
    def get_lifetime_value(self):
        """Customer is expert about their orders"""
        return sum(order.get_total() for order in self._orders)


class Order:
    def __init__(self):
        self._items = []
    
    def add_item(self, item):
        self._items.append(item)
    
    def get_total(self):
        """Order is expert about its items"""
        return sum(item.get_subtotal() for item in self._items)


class OrderItem:
    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity
    
    def get_subtotal(self):
        """OrderItem is expert about its calculation"""
        return self.product.price * self.quantity
```

---

## 🎓 Practice Exercises

### Exercise 1: Library System
Create a library system where:
- `Book` knows if it's available
- `Library` knows how many books it has
- `Member` knows how many books they've borrowed

### Exercise 2: Temperature Converter
Create a `Temperature` class that:
- Stores a value and unit (Celsius/Fahrenheit)
- Converts to the other unit
- Compares with other temperatures

### Exercise 3: Rectangle Area Calculator
Refactor this code using Information Expert:
```python
# Current code
def calculate_area(width, height):
    return width * height

def calculate_perimeter(width, height):
    return 2 * (width + height)
```

**Hint:** Create a `Rectangle` class as the Information Expert.

---

## 📚 Further Reading

- "Applying UML and Patterns" by Craig Larman (Origin of GRASP)
- "Domain-Driven Design" by Eric Evans
- "Clean Code" by Robert C. Martin
- Martin Fowler's "Anemic Domain Model" article

---

## ✅ Checklist

- [ ] I understand what Information Expert means
- [ ] I can identify which class has the information needed
- [ ] I can avoid anemic domain models
- [ ] I know when NOT to use Information Expert
- [ ] I've practiced with at least 2 examples

---

**Next Pattern:** [Creator](02_creator.md)

**Status:** 🟢 Completed | 🟡 In Progress | 🔴 Not Started
# Creator

> **Category:** GRASP (General Responsibility Assignment Software Patterns)  
> **Type:** Responsibility Assignment Pattern  
> **Difficulty:** ⭐ Beginner

---

## 📖 Definition

**Creator** is a principle for assigning object creation responsibilities. It states:

> **Assign the responsibility to create an instance of class A to class B if one of these conditions is true:**
> 1. B contains or compositely aggregates A
> 2. B records instances of A
> 3. B closely uses A
> 4. B has the initializing data for A (B is an Information Expert for creating A)

In simpler terms: **The class that should create an object is the one that has the closest relationship with it.**

---

## 🔍 Explanation

The Creator pattern answers the fundamental question: **"Who should be responsible for creating a new instance of a class?"**

### Core Principle
- **Natural Responsibility**: Creation responsibility follows natural object relationships
- **Low Coupling**: Creator already knows about the created object
- **Cohesion**: Related objects are created where they're used

### The Decision Tree
When deciding who should create an object, ask:
1. Does anyone **contain** this object? → They should create it
2. Does anyone **aggregate** instances of this object? → They should create it
3. Does anyone **closely use** this object? → They should create it
4. Does anyone have the **data** needed to initialize it? → They should create it

### Priority Order
1. Container/Aggregator (strongest relationship)
2. Recorder (maintains collection)
3. Heavy user (frequent usage)
4. Information Expert (has initialization data)

---

## 🎯 Problem It Solves

### Problems Without Creator Pattern:
1. **Unclear Responsibilities**: Unclear who should create objects
2. **High Coupling**: Random classes creating objects they don't relate to
3. **Scattered Creation Logic**: Object creation spread across the codebase
4. **Difficult Testing**: Hard to mock or test creation logic
5. **Initialization Issues**: Wrong class has initialization data

### Example of Bad Design:
```python
# ❌ BAD: Unnatural creation responsibility
class ShoppingCartController:
    def process_order(self):
        # Controller creating domain objects it doesn't contain or use
        order = Order()
        item1 = OrderItem(product1, 2)
        item2 = OrderItem(product2, 1)
        order.add_item(item1)
        order.add_item(item2)
        
        # Scattered creation logic
        customer = Customer()
        customer.add_order(order)

# Problems:
# - Controller shouldn't create domain objects
# - Creation logic scattered across UI layer
# - High coupling to domain classes
```

---

## ✅ Solution With Creator Pattern

```python
# ✅ GOOD: Natural creation responsibility

class ShoppingCart:
    """ShoppingCart creates OrderItems (it contains them)"""
    def __init__(self):
        self._items = []
    
    def add_product(self, product, quantity):
        """Cart creates OrderItem because it contains/aggregates them"""
        item = OrderItem(product, quantity)
        self._items.append(item)
        return item
    
    def create_order(self):
        """Cart creates Order because it has the data and contains items"""
        order = Order()
        for item in self._items:
            order.add_item(item)
        self._items.clear()
        return order


class Customer:
    """Customer creates Orders (it records/aggregates them)"""
    def __init__(self, name, email):
        self.name = name
        self.email = email
        self._orders = []
    
    def place_order(self, shopping_cart):
        """Customer creates Order because it records them"""
        order = shopping_cart.create_order()
        order.set_customer(self)
        self._orders.append(order)
        return order


class Order:
    def __init__(self):
        self._items = []
        self._customer = None
    
    def set_customer(self, customer):
        self._customer = customer
    
    def add_item(self, item):
        self._items.append(item)


class OrderItem:
    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity
```

---

## 🐍 Practical Python Examples

### Example 1: Library System

```python
class Library:
    """Library creates Books (it contains/aggregates them)"""
    def __init__(self, name):
        self.name = name
        self._books = []
        self._members = []
    
    def add_book(self, title, author, isbn):
        """Library creates Book - it's the natural container"""
        book = Book(title, author, isbn)
        self._books.append(book)
        return book
    
    def register_member(self, name, email):
        """Library creates Member - it records members"""
        member = Member(name, email)
        self._members.append(member)
        return member
    
    def find_book(self, isbn):
        return next((b for b in self._books if b.isbn == isbn), None)


class Member:
    """Member creates Loans (it records them)"""
    def __init__(self, name, email):
        self.name = name
        self.email = email
        self._loans = []
    
    def borrow_book(self, book):
        """Member creates Loan - it records/tracks loans"""
        if book.is_available():
            loan = Loan(self, book)
            self._loans.append(loan)
            book.mark_as_borrowed()
            return loan
        return None
    
    def get_active_loans(self):
        return [loan for loan in self._loans if not loan.is_returned]


class Loan:
    """Simple class - created by Member"""
    def __init__(self, member, book):
        self.member = member
        self.book = book
        self.is_returned = False
        self.borrowed_date = datetime.now()
    
    def return_book(self):
        self.is_returned = True
        self.book.mark_as_available()


class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
        self._is_available = True
    
    def is_available(self):
        return self._is_available
    
    def mark_as_borrowed(self):
        self._is_available = False
    
    def mark_as_available(self):
        self._is_available = True


# Usage demonstrates natural creation flow
library = Library("City Library")

# Library creates books it contains
book1 = library.add_book("Clean Code", "Robert Martin", "123")
book2 = library.add_book("Design Patterns", "Gang of Four", "456")

# Library creates members it records
member = library.register_member("Alice", "alice@example.com")

# Member creates loans it tracks
loan = member.borrow_book(book1)
```

### Example 2: Company Organization Structure

```python
from datetime import datetime
from typing import List, Optional

class Company:
    """Company creates Departments (it contains them)"""
    def __init__(self, name):
        self.name = name
        self._departments = []
    
    def create_department(self, name, budget):
        """Company creates Department - natural container"""
        department = Department(name, budget)
        self._departments.append(department)
        return department
    
    def get_all_employees(self):
        employees = []
        for dept in self._departments:
            employees.extend(dept.get_employees())
        return employees


class Department:
    """Department creates Employees (it contains/manages them)"""
    def __init__(self, name, budget):
        self.name = name
        self.budget = budget
        self._employees = []
    
    def hire_employee(self, name, position, salary):
        """Department creates Employee - it manages employees"""
        if salary <= self.budget:
            employee = Employee(name, position, salary, self)
            self._employees.append(employee)
            self.budget -= salary
            return employee
        raise ValueError("Insufficient budget")
    
    def get_employees(self):
        return self._employees.copy()


class Employee:
    """Employee creates Tasks (it's assigned to them)"""
    def __init__(self, name, position, salary, department):
        self.name = name
        self.position = position
        self.salary = salary
        self.department = department
        self._tasks = []
    
    def assign_task(self, description, deadline):
        """Employee creates Task - they own/track their tasks"""
        task = Task(description, deadline, self)
        self._tasks.append(task)
        return task
    
    def get_pending_tasks(self):
        return [t for t in self._tasks if not t.is_completed]


class Task:
    def __init__(self, description, deadline, assignee):
        self.description = description
        self.deadline = deadline
        self.assignee = assignee
        self.is_completed = False
        self.created_at = datetime.now()
    
    def complete(self):
        self.is_completed = True


# Usage
company = Company("Tech Corp")

# Company creates departments
engineering = company.create_department("Engineering", 500000)
marketing = company.create_department("Marketing", 300000)

# Department creates employees
alice = engineering.hire_employee("Alice", "Senior Dev", 120000)
bob = engineering.hire_employee("Bob", "Junior Dev", 80000)

# Employee creates tasks
task1 = alice.assign_task("Review pull request", "2025-12-20")
task2 = alice.assign_task("Deploy to production", "2025-12-18")
```

### Example 3: Game World with Entities

```python
class Game:
    """Game creates GameWorld (it owns it)"""
    def __init__(self):
        self.world = None
    
    def start_new_game(self, world_name):
        """Game creates GameWorld - it's the top-level container"""
        self.world = GameWorld(world_name)
        return self.world
    
    def run(self):
        if self.world:
            self.world.update()


class GameWorld:
    """GameWorld creates Levels and manages entities"""
    def __init__(self, name):
        self.name = name
        self._levels = []
        self._current_level = None
    
    def create_level(self, level_id, difficulty):
        """GameWorld creates Level - it contains levels"""
        level = Level(level_id, difficulty)
        self._levels.append(level)
        return level
    
    def load_level(self, level_id):
        level = next((l for l in self._levels if l.id == level_id), None)
        if level:
            self._current_level = level
        return level
    
    def update(self):
        if self._current_level:
            self._current_level.update()


class Level:
    """Level creates Enemies and Items (it contains them)"""
    def __init__(self, level_id, difficulty):
        self.id = level_id
        self.difficulty = difficulty
        self._enemies = []
        self._items = []
        self._player = None
    
    def spawn_enemy(self, enemy_type, x, y):
        """Level creates Enemy - it manages enemies in the level"""
        health = 100 * self.difficulty
        enemy = Enemy(enemy_type, x, y, health)
        self._enemies.append(enemy)
        return enemy
    
    def spawn_item(self, item_type, x, y):
        """Level creates Item - it contains items"""
        item = Item(item_type, x, y)
        self._items.append(item)
        return item
    
    def set_player(self, player):
        self._player = player
    
    def update(self):
        for enemy in self._enemies:
            enemy.update()


class Enemy:
    """Enemy creates Projectiles when attacking"""
    def __init__(self, enemy_type, x, y, health):
        self.type = enemy_type
        self.x = x
        self.y = y
        self.health = health
        self._projectiles = []
    
    def attack(self, target_x, target_y):
        """Enemy creates Projectile - it shoots them"""
        projectile = Projectile(self.x, self.y, target_x, target_y)
        self._projectiles.append(projectile)
        return projectile
    
    def update(self):
        # Update projectiles
        for proj in self._projectiles:
            proj.update()


class Projectile:
    def __init__(self, start_x, start_y, target_x, target_y):
        self.x = start_x
        self.y = start_y
        self.target_x = target_x
        self.target_y = target_y
        self.speed = 5
    
    def update(self):
        # Move towards target
        pass


class Item:
    def __init__(self, item_type, x, y):
        self.type = item_type
        self.x = x
        self.y = y


# Usage
game = Game()
world = game.start_new_game("Dungeon World")

# World creates levels
level1 = world.create_level("level_1", difficulty=1)
level2 = world.create_level("level_2", difficulty=2)

# Level creates enemies and items
world.load_level("level_1")
enemy1 = level1.spawn_enemy("goblin", 100, 200)
enemy2 = level1.spawn_enemy("orc", 150, 250)
sword = level1.spawn_item("sword", 50, 50)

# Enemy creates projectiles when attacking
projectile = enemy1.attack(target_x=200, target_y=300)
```

---

## 🏢 Examples in Existing Frameworks

### 1. **Django ORM - QuerySet Creates Model Instances**
```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    
    def create_book(self, title, isbn):
        """Author creates Book - has initialization data and relationship"""
        return Book.objects.create(
            title=title,
            isbn=isbn,
            author=self  # Author has the relationship data
        )

class Book(models.Model):
    title = models.CharField(max_length=200)
    isbn = models.CharField(max_length=13)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

# Manager (aggregator) creates instances
author = Author.objects.create(name="Martin Fowler")

# Author creates books (has relationship data)
book = author.create_book("Refactoring", "1234567890")

# Or QuerySet creates (it manages collection)
books = Book.objects.create(title="Clean Code", isbn="123", author=author)
```

### 2. **Flask - Application Creates Routes/Blueprints**
```python
from flask import Flask, Blueprint

# Application creates and contains blueprints
app = Flask(__name__)

# App creates blueprints (it registers/contains them)
api_blueprint = Blueprint('api', __name__)
admin_blueprint = Blueprint('admin', __name__)

app.register_blueprint(api_blueprint, url_prefix='/api')
app.register_blueprint(admin_blueprint, url_prefix='/admin')
```

### 3. **SQLAlchemy - Session Creates Instances**
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

# Session is the creator - it tracks/manages instances
engine = create_engine('sqlite:///example.db')
SessionFactory = sessionmaker(bind=engine)
session = SessionFactory()

# Session creates and tracks objects
user = User(name="Alice", email="alice@example.com")
session.add(user)  # Session manages the instance

# Session creates queries (it has the data and context)
users = session.query(User).filter_by(name="Alice").all()
```

### 4. **Python Collections - Lists/Dicts Creating Elements**
```python
# List comprehension - list creates its elements
squares = [x**2 for x in range(10)]

# Dict creating entries
config = {
    'host': 'localhost',
    'port': 5432,
    'database': 'mydb'
}

# Factory methods
my_list = list()  # list() creates list instance
my_dict = dict()  # dict() creates dict instance
my_set = set()    # set() creates set instance
```

### 5. **unittest - TestCase Creates Test Methods**
```python
import unittest

class MyTestCase(unittest.TestCase):
    """TestCase creates test fixtures (it contains them)"""
    
    def setUp(self):
        """TestCase creates test objects - it has initialization data"""
        self.calculator = Calculator()
        self.test_data = [1, 2, 3, 4, 5]
    
    def test_addition(self):
        result = self.calculator.add(2, 3)
        self.assertEqual(result, 5)
```

### 6. **Pandas - DataFrame Creates Series**
```python
import pandas as pd

# DataFrame creates Series (it contains them)
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['NY', 'LA', 'SF']
})

# DataFrame creates new columns (Series)
df['adult'] = df['age'] >= 18

# DataFrame creates filtered views
young_people = df[df['age'] < 30]
```

---

## ✅ Pros and Cons

### Pros ✅

1. **Low Coupling**
   - Creator already knows about the created class
   - No new dependencies introduced

2. **Clear Responsibilities**
   - Obvious who should create what
   - Follows natural object relationships

3. **Better Cohesion**
   - Creation happens where objects are used
   - Related responsibilities stay together

4. **Easier Maintenance**
   - Creation logic in predictable places
   - Changes localized to natural owner

5. **Testability**
   - Easy to mock or stub creation
   - Clear boundaries for testing

6. **Follows Natural Design**
   - Intuitive and easy to understand
   - Matches real-world relationships

### Cons ❌

1. **May Not Support Complex Creation**
   - Simple creation only
   - Complex initialization needs Factory patterns

2. **Can Increase Class Responsibilities**
   - Classes do creation + their main job
   - May violate Single Responsibility if overdone

3. **Tight Coupling in Some Cases**
   - Creator knows concrete class
   - Hard to substitute different implementations

4. **Not Ideal for Configurable Creation**
   - When creation logic varies significantly
   - Better to use Factory Method or Abstract Factory

5. **Database/Persistence Issues**
   - Domain objects shouldn't create persistence objects
   - Need Repository or Data Mapper patterns

---

## 💡 When to Use

### ✅ Use Creator When:

1. **Clear Containment Relationship**
   - One object naturally contains another
   - Example: `Order` creates `OrderItem`

2. **Aggregation Relationship**
   - One object maintains a collection of others
   - Example: `Library` creates `Book` instances

3. **Close Usage**
   - One class heavily uses another
   - Example: `ShoppingCart` uses `OrderItem` extensively

4. **Has Initialization Data**
   - One class has all data needed to create another
   - Example: `Customer` has data to create `Order`

5. **Simple Object Creation**
   - Direct instantiation with `new` or `__init__`
   - No complex configuration needed

6. **Natural Ownership**
   - Clear "parent-child" relationship
   - Example: `Game` creates `GameWorld`

### ❌ Don't Use Creator When:

1. **Complex Creation Logic**
   - Multiple steps, validation, or configuration
   - Use: Builder or Factory Method pattern

2. **Multiple Creation Strategies**
   - Different ways to create same object
   - Use: Factory Method or Abstract Factory

3. **Runtime Type Selection**
   - Don't know exact class until runtime
   - Use: Factory patterns

4. **Cross-Cutting Concerns**
   - Creation involves logging, caching, pooling
   - Use: Factory with additional logic

5. **Persistence Layer**
   - Creating objects from database
   - Use: Repository or Data Mapper pattern

6. **Dependency Injection Needed**
   - Need to inject dependencies
   - Use: DI Container or Factory

---

## 🚫 Common Mistakes

### Mistake 1: Random Class Creating Objects
```python
# ❌ BAD: Controller creating domain objects
class UserController:
    def register(self, data):
        # Controller shouldn't create domain objects
        user = User(data['name'], data['email'])
        address = Address(data['street'], data['city'])
        user.set_address(address)
        return user

# ✅ GOOD: Use appropriate creator or factory
class UserService:
    def register_user(self, data):
        # Service coordinates, but delegates creation
        user = User.create_from_registration(data)
        return user

class User:
    @classmethod
    def create_from_registration(cls, data):
        """User has initialization data - creates itself"""
        user = cls(data['name'], data['email'])
        if 'address' in data:
            user.set_address(
                Address(data['street'], data['city'])
            )
        return user
```

### Mistake 2: Creating When You Don't Have the Relationship
```python
# ❌ BAD: No relationship to created object
class ReportGenerator:
    def generate_sales_report(self):
        # Why is ReportGenerator creating Customer?
        customer = Customer("John", "john@example.com")
        # This makes no sense
        
# ✅ GOOD: Report uses existing customer
class ReportGenerator:
    def generate_sales_report(self, customer):
        # Uses customer passed in, doesn't create it
        sales = customer.get_orders()
        return self.format_report(sales)
```

### Mistake 3: God Class Creating Everything
```python
# ❌ BAD: One factory creates everything
class ObjectFactory:
    def create_order(self): pass
    def create_customer(self): pass
    def create_product(self): pass
    def create_payment(self): pass
    def create_shipment(self): pass
    # Creates everything - poor cohesion

# ✅ GOOD: Each creator creates what it contains/uses
class Customer:
    def create_order(self):
        return Order(self)

class Order:
    def add_product(self, product, quantity):
        item = OrderItem(product, quantity)
        self.items.append(item)
```

### Mistake 4: Ignoring Information Expert for Creation
```python
# ❌ BAD: Creating without initialization data
class OrderService:
    def create_order_item(self, product_id, quantity):
        # Service doesn't have product object
        product = self.product_repo.find(product_id)
        return OrderItem(product, quantity)

# ✅ GOOD: Let the expert create
class ShoppingCart:
    def add_product(self, product, quantity):
        # Cart has the product and quantity - it's the expert
        item = OrderItem(product, quantity)
        self.items.append(item)
```

---

## 🔗 Related Patterns

### GRASP Patterns
- **Information Expert**: Often the creator is also the expert
- **Low Coupling**: Creator pattern maintains low coupling
- **High Cohesion**: Keeps creation with related functionality

### GoF Patterns
- **Factory Method**: For complex or variant creation
- **Abstract Factory**: For families of related objects
- **Builder**: For step-by-step complex creation
- **Prototype**: For cloning instead of creating

### When to Upgrade from Creator
```python
# Start with Creator (simple)
class Order:
    def add_item(self, product, quantity):
        item = OrderItem(product, quantity)
        self.items.append(item)

# Upgrade to Factory Method (variants)
class Order:
    def add_item(self, product, quantity):
        item = self._create_item(product, quantity)
        self.items.append(item)
    
    def _create_item(self, product, quantity):
        # Can be overridden for different item types
        return OrderItem(product, quantity)

# Upgrade to Builder (complex)
class OrderBuilder:
    def __init__(self):
        self.order = Order()
    
    def add_product(self, product, quantity):
        item = OrderItem(product, quantity)
        self.order.items.append(item)
        return self
    
    def set_customer(self, customer):
        self.order.customer = customer
        return self
    
    def build(self):
        return self.order
```

---

## 📊 Decision Guide

### Creator Pattern Decision Tree

```
Need to create an object?
│
├─ Is creation complex (validation, steps, config)?
│  ├─ YES → Use Factory/Builder Pattern
│  └─ NO → Continue
│
├─ Do you contain/aggregate the object?
│  ├─ YES → YOU are the Creator! ✅
│  └─ NO → Continue
│
├─ Do you record instances of this object?
│  ├─ YES → YOU are the Creator! ✅
│  └─ NO → Continue
│
├─ Do you closely use this object?
│  ├─ YES → YOU are the Creator! ✅
│  └─ NO → Continue
│
├─ Do you have initialization data?
│  ├─ YES → YOU are the Creator! ✅
│  └─ NO → Use Factory or reconsider design
```

---

## 🎓 Practice Exercises

### Exercise 1: Blog Platform
Design a blog system where:
- `Blog` creates `Post` instances
- `Post` creates `Comment` instances
- `Author` creates `Blog` instances

Implement using Creator pattern.

### Exercise 2: Restaurant Order System
Create a system where:
- `Restaurant` creates `Table` instances
- `Table` creates `Order` instances
- `Order` creates `OrderItem` instances
- `Kitchen` creates `Meal` instances

### Exercise 3: Refactor This Code
```python
# Current problematic code
class Application:
    def run(self):
        user = User("Alice")
        cart = ShoppingCart()
        product = Product("Laptop", 1000)
        order = Order()
        # Everything created in one place
```

Refactor to use Creator pattern properly.

---

## 📚 Further Reading

- "Applying UML and Patterns" by Craig Larman - Chapter on Creator
- "Design Patterns" (GoF) - Factory patterns (evolution of Creator)
- "Domain-Driven Design" by Eric Evans - Aggregate Roots as Creators
- Martin Fowler's blog on Object Creation patterns

---

## ✅ Checklist

- [ ] I understand the 4 Creator conditions
- [ ] I can identify natural creation relationships
- [ ] I know when Creator is insufficient (need Factory)
- [ ] I can avoid creating "God" objects
- [ ] I've practiced with at least 2 examples
- [ ] I understand Creator vs Factory patterns

---

**Previous Pattern:** [Information Expert](01_information_expert.md)  
**Next Pattern:** [Controller](03_controller.md)

**Status:** 🟢 Completed | 🟡 In Progress | 🔴 Not Started
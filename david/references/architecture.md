# DAVID — Architecture Audit Reference

Full pattern library for Scanner E. Load when performing architectural review.

---

## Table of Contents
1. [SOLID Principles — Detection Patterns](#solid-principles)
2. [Coupling & Cohesion Analysis](#coupling--cohesion)
3. [Layer Architecture Violations](#layer-violations)
4. [Common Anti-Patterns](#anti-patterns)
5. [Design Pattern Misuse](#design-pattern-misuse)

---

## SOLID Principles

### S — Single Responsibility Principle

**Violation signals:**
- Class/module name contains "And", "Or", "Manager", "Helper", "Utils", "Misc"
- More than one reason for the class to change
- Methods that don't share any data with each other
- Class has both business logic AND persistence logic AND formatting logic

```python
# ❌ SRP Violation — UserManager does everything
class UserManager:
    def validate_email(self, email): ...
    def save_to_db(self, user): ...
    def send_welcome_email(self, user): ...
    def format_for_display(self, user): ...
    def generate_report(self): ...

# ✅ Separated concerns
class UserValidator:
    def validate_email(self, email): ...

class UserRepository:
    def save(self, user): ...

class UserNotifier:
    def send_welcome_email(self, user): ...
```

**Threshold:** If you can't describe what the class does without using "and", it's violating SRP.

---

### O — Open/Closed Principle

**Violation signals:**
- Long `if/elif/switch` chains that check `type` of an object
- New requirements force editing existing working code (instead of extending)
- `isinstance()` checks scattered throughout codebase
- Strategy pattern absent where behavior needs to vary

```python
# ❌ OCP Violation — must modify for every new shape
def get_area(shape):
    if shape.type == 'circle':
        return math.pi * shape.radius ** 2
    elif shape.type == 'rectangle':
        return shape.width * shape.height
    elif shape.type == 'triangle':  # Adding a new type means editing this
        return 0.5 * shape.base * shape.height

# ✅ Open for extension, closed for modification
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

class Circle(Shape):
    def area(self): return math.pi * self.radius ** 2

class Rectangle(Shape):
    def area(self): return self.width * self.height
```

---

### L — Liskov Substitution Principle

**Violation signals:**
- Subclass throws `NotImplementedError` on inherited methods
- Subclass overrides method and narrows accepted input types
- Subclass overrides method and widens return types
- Code has `if isinstance(obj, SubclassX)` to handle special cases

```python
# ❌ LSP Violation
class Bird:
    def fly(self): ...

class Penguin(Bird):
    def fly(self):
        raise NotImplementedError("Penguins can't fly")  # Breaks substitutability

# ✅ Restructure hierarchy
class Bird: ...
class FlyingBird(Bird):
    def fly(self): ...
class Penguin(Bird): ...  # No fly method
```

---

### I — Interface Segregation Principle

**Violation signals:**
- Interface/abstract class with >7 methods
- Implementing class raises `NotImplementedError` on methods it doesn't need
- "Fat interface" where implementors only use half the methods
- All-or-nothing interface that forces unrelated method implementations

```python
# ❌ ISP Violation — Worker must implement eat() even if it's a robot
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...
    @abstractmethod
    def sleep(self): ...

class RobotWorker(Worker):
    def work(self): ...
    def eat(self): raise NotImplementedError  # Robots don't eat
    def sleep(self): raise NotImplementedError

# ✅ Segregated interfaces
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Biological(ABC):
    @abstractmethod
    def eat(self): ...
    @abstractmethod
    def sleep(self): ...
```

---

### D — Dependency Inversion Principle

**Violation signals:**
- High-level business logic directly instantiates low-level classes
- `import` of concrete implementation inside business logic class
- No dependency injection — dependencies hard-coded inside classes
- Unit tests require real DB / file system / network to function

```python
# ❌ DIP Violation — OrderService directly depends on MySQLDatabase
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()  # Hard dependency on concrete class
        self.emailer = SMTPEmailer()

    def place_order(self, order): ...

# ✅ Dependency Inversion — depends on abstractions
class OrderService:
    def __init__(self, db: Database, emailer: Emailer):
        self.db = db      # Injected — can be any Database implementation
        self.emailer = emailer

    def place_order(self, order): ...

# Test can inject mock:
# service = OrderService(db=MockDatabase(), emailer=MockEmailer())
```

---

## Coupling & Cohesion

### High Coupling (Bad) — Detection

| Signal | Example | Fix |
|--------|---------|-----|
| Accessing internals of another class | `user._private_field` | Expose via public method |
| Knowing concrete type of collaborator | `if isinstance(repo, MySQLRepo)` | Program to interface |
| Instantiating collaborators internally | `self.repo = MySQLRepo()` | Inject via constructor |
| Circular imports | A imports B, B imports A | Extract shared interface to third module |
| God class knows about 10+ other classes | `UserManager` imports every model | Decompose into smaller services |

**Coupling metric:** Count how many other modules a class directly imports. >5 is a yellow flag; >10 is a red flag.

### Low Cohesion (Bad) — Detection

| Signal | Example |
|--------|---------|
| Utility "grab bag" files | `utils.py`, `helpers.js`, `misc.rb` with 40 unrelated functions |
| Methods sharing no data | Half use `self.x`, half use `self.y`, none use both |
| Module does completely unrelated things | `user_service.py` also handles payment logic |
| Catch-all service | `ApiService` with methods for users, products, orders, billing |

**Cohesion metric:** What % of a class's methods use each field? If most methods use only their own subset of fields, the class should be split.

### God Class Detection

Flag any class where:
- More than 500 lines
- More than 20 public methods
- More than 10 instance variables
- Other classes reference it constantly

### Circular Dependency Analysis

```
Module A → imports → Module B
Module B → imports → Module A
     ↑                   ↑
        CIRCULAR — breaks at runtime in many languages

Fix options:
1. Extract shared types into Module C; A and B both import C
2. Move the shared logic to one module
3. Use dependency injection to break the cycle
```

---

## Layer Violations

### Clean Architecture / MVC Violations

```
CORRECT DEPENDENCY DIRECTION:
Presentation → Application → Domain ← Infrastructure

VIOLATIONS TO FLAG:
- Domain layer imports from Infrastructure (ORM model in domain logic)
- Presentation layer skips Application layer (Controller calls Repository directly)
- Domain layer imports HTTP request objects
- Infrastructure layer contains business rules
```

**Examples:**

```python
# ❌ Layer violation — Domain entity imports SQLAlchemy (infrastructure)
from sqlalchemy import Column, Integer, String
class User(Base):  # Domain entity knows about ORM
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    def calculate_discount(self): ...  # Business logic mixed with infrastructure

# ✅ Separate concerns
class User:  # Pure domain entity
    def __init__(self, id, name, email): ...
    def calculate_discount(self): ...

class UserModel(Base):  # Infrastructure model
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
```

```javascript
// ❌ Controller directly calls repository (skips service layer)
router.post('/orders', async (req, res) => {
    const order = await orderRepository.save(req.body);  // Business logic?
    await emailRepository.sendConfirmation(order);        // Controller handles email?
    res.json(order);
});

// ✅ Controller delegates to service
router.post('/orders', async (req, res) => {
    const order = await orderService.placeOrder(req.body);
    res.json(order);
});
```

---

## Anti-Patterns

### Anemic Domain Model
Business logic scattered in Service classes instead of being in domain objects. Domain objects are just data containers with no behavior.

```python
# ❌ Anemic — User is just a data bag
class User:
    def __init__(self, name, email, balance):
        self.name = name
        self.email = email
        self.balance = balance

class UserService:
    def can_withdraw(self, user, amount):  # Logic lives in service
        return user.balance >= amount

# ✅ Rich domain model
class User:
    def can_withdraw(self, amount) -> bool:  # Logic lives in domain object
        return self.balance >= amount

    def withdraw(self, amount):
        if not self.can_withdraw(amount):
            raise InsufficientFundsError()
        self.balance -= amount
```

### Service Locator (Anti-Pattern)
Global registry for finding services — makes dependencies invisible and untestable.

```python
# ❌ Service Locator
class ServiceLocator:
    _services = {}
    @classmethod
    def get(cls, name): return cls._services[name]

class OrderService:
    def place_order(self, order):
        db = ServiceLocator.get('database')  # Hidden dependency
        emailer = ServiceLocator.get('emailer')

# ✅ Explicit dependency injection (see DIP above)
```

### Premature Generalization
Over-engineered abstraction before there's a second use case ("YAGNI" violation).

```python
# ❌ Abstract factory for creating users — only one type of user exists
class AbstractUserFactory(ABC):
    @abstractmethod
    def create_user(self) -> AbstractUser: ...

class ConcreteUserFactory(AbstractUserFactory):
    def create_user(self): return User()

# ✅ Just create the user directly until a second type is needed
user = User()
```

### Primitive Obsession at Architecture Level
Using primitive types (string, int, dict) to pass around complex domain concepts.

```python
# ❌ User data passed as raw dict throughout the system
def process_order(user: dict, items: list): ...
def send_confirmation(email: str, name: str, order_id: int): ...

# ✅ Value objects / domain types
def process_order(customer: Customer, cart: ShoppingCart): ...
def send_confirmation(order: Order): ...
```

---

## Design Pattern Misuse

| Pattern | Misuse | Correct Use |
|---------|--------|-------------|
| Singleton | Used everywhere as global state store | Only for truly unique resources (config, logger) |
| Factory | Applied to single type with no variants | Use when creation logic varies or is complex |
| Observer | Used for sequential operations | Use only for decoupled event notification |
| Strategy | Applied where there's only one strategy | Use when algorithms need to be swappable |
| Repository | Bypassed by calling ORM directly in service | Always mediate DB access through repository |
| DTO | Domain objects returned directly from API | Always map to DTOs at API boundary |

---

## Refactor Advisor (Scanner RF)

> Activates when user says "refactor", "restructure", "clean this up", or when CC score (Scanner S)
> flags a function as High/Untestable (CC > 15).
> Works with Scanner E (Architecture) — E identifies structural violations, RF proposes the plan.

### When RF Fires vs Scanner E

| Scanner E | Scanner RF |
|-----------|-----------|
| "This class violates SRP" | "Here's the step-by-step extraction plan with safe sequence" |
| "These modules are tightly coupled" | "Extract this interface, update these N callers in this order" |
| "This is a God Class" | "Split into ServiceA, ServiceB, ServiceC — migrate this way" |

### What RF Produces

- **Extraction targets**: exact code blocks → named functions, classes, or modules
- **Safe refactor sequence**: order of changes that keeps tests green at every step
- **Risk classification**: flags any change touching public API surface, shared utilities, or DB access
- **Scope guarantee**: only proposes changes where improvement is concrete and measurable (CC reduction, duplication removal, cohesion improvement)
- Never refactors speculatively — no "while we're in here" opportunistic changes

### Refactor Sequence Protocol

```
STEP 1 — Snapshot: identify all callers of the target code
STEP 2 — Extract: create the new function/class with the extracted logic
STEP 3 — Redirect: update callers one by one
STEP 4 — Verify: run tests after each redirect (not just at the end)
STEP 5 — Cleanup: remove the old code only after all callers verified
```

### Finding Format

```
🔧 REFACTOR PROPOSAL [RF-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Target     : [function/class/module name]
File:Line  : [file:N–M]
Reason     : [CC too high / God class / DRY violation / layer violation]
Proposal   : [exact extraction plan — what gets moved where]
Risk       : [Low / Medium / High]
Fix Confidence : [✅ SAFE / ⚠️ REVIEW FIRST / 🔴 CONFIRM BEFORE APPLY]
Sequence   : [Step 1 → Step 2 → Step 3 — safe order of operations]
```

**Fix Confidence rules for RF:**
- `✅ SAFE` — pure extraction of private logic with no external callers
- `⚠️ REVIEW FIRST` — touches shared utilities or has 3+ callers
- `🔴 CONFIRM BEFORE APPLY` — public API surface, payment/auth/DB deletion logic, or DAVID confidence < 70% due to missing context

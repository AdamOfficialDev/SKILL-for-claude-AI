# DAVID — Code Review Reference

Full pattern library for Scanner D. Load when performing code review, style analysis, or best practice checks.

---

## Table of Contents
1. [Code Smell Catalog](#code-smell-catalog)
2. [Best Practice Rules by Category](#best-practice-rules)
3. [Naming Convention Guide](#naming-conventions)
4. [Comment & Documentation Standards](#comments--documentation)
5. [Framework-Specific Reviews](#framework-specific-reviews)

---

## Code Smell Catalog

### 🚩 Long Method (God Function)

**Threshold:** >50 lines OR doing more than one clearly distinct thing.

**Indicators:**
- Function name contains "and", "or", "also"
- Multiple levels of abstraction in same function
- Requires multiple scrolls to read in full
- Comments needed to separate "sections"

**Fix:** Extract into smaller, single-purpose functions. Each function should do one thing and its name should fully describe what it does.

---

### 🚩 Long Parameter List

**Threshold:** >4 parameters.

**Indicators:**
```python
# ❌ Boolean trap + too many params
def create_user(name, email, age, is_admin, send_email, activate_now, role_id):
    ...

# ✅ Options object / dataclass
@dataclass
class CreateUserOptions:
    name: str
    email: str
    age: int
    is_admin: bool = False
    send_welcome_email: bool = True
    activate_now: bool = True
    role_id: int = None

def create_user(opts: CreateUserOptions):
    ...
```

---

### 🚩 Boolean Trap

Parameters named with booleans where you can't tell what `True` means at the call site.

```javascript
// ❌ What do these booleans mean?
renderButton(label, true, false, true);

// ✅ Named options
renderButton(label, { isDisabled: true, isOutline: false, isFullWidth: true });
```

---

### 🚩 Duplicate Code (DRY Violation)

**Threshold:** Same logic repeated 2+ times.

**Detection:** Look for identical or near-identical blocks of code with only variable names changed. Extract into a shared function parameterized on what differs.

---

### 🚩 Dead Code

**Types:**
- Commented-out code (not documentation)
- Functions/methods never called from anywhere
- Variables assigned but never read
- Imports never used
- Conditions that are always true/false (constant folding)
- Code after `return`/`break`/`throw`

**Action:** Flag for user confirmation before removing (could be intentionally kept).

---

### 🚩 Magic Numbers & Strings

```python
# ❌ Magic numbers
if status == 3:
    ...
time.sleep(86400)
if score > 0.75:
    ...

# ✅ Named constants
STATUS_APPROVED = 3
SECONDS_PER_DAY = 86_400
PASSING_THRESHOLD = 0.75

if status == STATUS_APPROVED:
    ...
```

---

### 🚩 Deep Nesting (Pyramid of Doom)

**Threshold:** >3 levels of nesting.

**Fix:** Early returns, guard clauses, extract to functions.

```javascript
// ❌ Pyramid of doom
function process(data) {
    if (data) {
        if (data.user) {
            if (data.user.isActive) {
                if (data.user.permissions.includes('edit')) {
                    // actual logic buried here
                }
            }
        }
    }
}

// ✅ Guard clauses — fail fast
function process(data) {
    if (!data) return;
    if (!data.user) return;
    if (!data.user.isActive) return;
    if (!data.user.permissions.includes('edit')) return;
    // actual logic at top level
}
```

---

### 🚩 Feature Envy

A function that accesses another object's data more than its own — sign it belongs in the other class.

```python
# ❌ Feature envy — Order uses Customer's internals
class Order:
    def get_discount(self, customer):
        if customer.membership == 'gold' and customer.years >= 3:
            return customer.purchase_total * 0.15
        return 0

# ✅ Move it to Customer
class Customer:
    def calculate_discount(self, purchase_total):
        if self.membership == 'gold' and self.years >= 3:
            return purchase_total * 0.15
        return 0
```

---

### 🚩 Shotgun Surgery

A single logical change requires modifications to many different classes/files. Indicates poor encapsulation. Flag for architectural note but do not fix without user approval.

---

### 🚩 Primitive Obsession

Using primitive types (string, int) to represent domain concepts that deserve their own type.

```python
# ❌ Email is just a string — no validation, no type safety
def send_email(address: str):
    ...

# ✅ Email as a value object
@dataclass(frozen=True)
class Email:
    value: str
    def __post_init__(self):
        if '@' not in self.value:
            raise ValueError(f"Invalid email: {self.value}")
```

---

## Best Practice Rules

### Error Handling

```python
# ❌ Silent swallowing
try:
    result = risky_operation()
except Exception:
    pass  # Bug silently ignored

# ❌ Too broad catch
try:
    result = risky_operation()
except Exception as e:
    print(e)  # Logged but not handled

# ✅ Specific exception, proper handling
try:
    result = risky_operation()
except DatabaseConnectionError as e:
    logger.error("DB connection failed", exc_info=True)
    raise ServiceUnavailableError("Database unavailable") from e
except ValueError as e:
    raise InvalidInputError(str(e)) from e
```

### Logging Best Practices

```python
# ❌ Print statements (not capturable, no level)
print(f"User {user_id} logged in")
print(f"ERROR: {error}")

# ❌ Logging sensitive data
logger.info(f"Login attempt: email={email}, password={password}")

# ✅ Structured logging, appropriate levels, no PII
logger.info("User login", extra={"user_id": user_id, "ip": request.remote_addr})
logger.error("Login failed", extra={"reason": "invalid_password"})
# Never log: password, token, SSN, full credit card
```

### Function Design Rules

| Rule | Good | Bad |
|------|------|-----|
| Single responsibility | `validateEmail(email)` | `validateAndSaveAndSendEmail(email)` |
| Command-query separation | Either return value OR cause side effect | Return value AND modify state |
| Pure when possible | No side effects for same inputs → same output | Reads global state, modifies external |
| Fail fast | Validate at entry, throw early | Validate deep in logic after side effects |
| Obvious return types | Always returns same type | Returns `string` OR `null` OR `false` |

### Import Organization

```python
# ✅ Python: stdlib → third-party → local
import os
import sys
from typing import Optional

import requests
import sqlalchemy

from myapp.models import User
from myapp.utils import validate
```

```javascript
// ✅ JS/TS: react/framework → third-party → local → styles
import React, { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import axios from 'axios';

import { UserCard } from '@/components/UserCard';
import { formatDate } from '@/utils/date';
import styles from './Page.module.css';
```

---

## Naming Conventions

### Universal Rules

| Rule | Example |
|------|---------|
| Functions: verb + noun | `getUser()`, `validateEmail()`, `processOrder()` |
| Booleans: is/has/can/should prefix | `isActive`, `hasPermission`, `canEdit` |
| Constants: UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `API_BASE_URL` |
| Avoid abbreviations | `userRepository` not `usrRepo` |
| Be specific | `getUserById()` not `getUser()` |
| No generic names | Not `data`, `info`, `temp`, `obj`, `result` in outer scope |

### Language-Specific

| Language | Classes | Functions | Variables | Constants |
|----------|---------|-----------|-----------|-----------|
| Python | PascalCase | snake_case | snake_case | UPPER_SNAKE |
| JavaScript | PascalCase | camelCase | camelCase | UPPER_SNAKE |
| TypeScript | PascalCase | camelCase | camelCase | UPPER_SNAKE |
| Java | PascalCase | camelCase | camelCase | UPPER_SNAKE |
| Go | PascalCase (exported) | camelCase/PascalCase | camelCase | PascalCase |
| Rust | PascalCase | snake_case | snake_case | UPPER_SNAKE |

---

## Comments & Documentation

### When Comments Are Required

```python
# ✅ Explain WHY, not WHAT
# Using binary search here because this is called 10k/sec in hot path
index = binary_search(items, target)

# ✅ Document non-obvious behavior
# Returns None (not raises) if user not found — callers expect None check
def find_user(user_id: int) -> Optional[User]:
    ...

# ✅ TODO with context
# TODO(alice): Remove this workaround once API v2 is deployed (ticket: PROJ-123)
legacy_format = transform_for_v1(data)
```

```python
# ❌ Comment restates code — noise
# Increment i by 1
i += 1

# ❌ Commented-out code without explanation
# user.save()
# user.notify()

# ❌ Misleading comment (worse than no comment)
# Returns user email
def get_user_name(user_id):  # Actually returns name, not email
    ...
```

### Docstring Requirements (flag as missing if absent)

**Must have docstrings/JSDoc:**
- All exported functions / public methods
- All public classes
- Functions >20 lines
- Functions with non-obvious parameter types or behavior
- Any function that can raise/throw (document which exceptions)

**Template:**
```python
def process_payment(amount: Decimal, currency: str, user_id: int) -> PaymentResult:
    """
    Process a payment and return the result.

    Args:
        amount: Payment amount in the specified currency (must be > 0)
        currency: ISO 4217 currency code (e.g. 'USD', 'EUR')
        user_id: ID of the user initiating the payment

    Returns:
        PaymentResult with status and transaction_id

    Raises:
        InsufficientFundsError: If user balance < amount
        InvalidCurrencyError: If currency not supported
        PaymentGatewayError: If external payment service fails
    """
```

---

## Framework-Specific Reviews

### React

| Check | Good Pattern | Bad Pattern |
|-------|-------------|-------------|
| State shape | Minimal, flat state | Deeply nested, redundant state |
| Effect deps | Accurate dependency array | Missing deps or `// eslint-disable` |
| Component size | <200 lines, single concern | 500+ line components |
| Prop drilling | >2 levels → use Context | Passing props through 4 levels |
| Key prop | Stable unique key | Index as key in dynamic lists |
| Conditional rendering | Explicit conditions | Truthy short-circuit with `0 &&` |

### Django / Flask / FastAPI

| Check | Good | Bad |
|-------|------|-----|
| Business logic | In service layer / models | In views/routes |
| DB queries | In model methods or repositories | Scattered in views |
| Validation | Pydantic/DRF serializers | Manual `if request.data` checks |
| Secrets | `os.environ['SECRET']` | Hardcoded in settings.py |
| Migrations | Backward-compatible | Drops columns without deprecation period |

### Express / NestJS

| Check | Good | Bad |
|-------|------|-----|
| Route handlers | Thin — delegate to service | Contains business logic |
| Error handling | Centralized error middleware | Try-catch repeated in every route |
| Validation | `joi`, `zod`, or decorators | `if (!req.body.email)` checks |
| Async | `async/await` with try-catch | `.then().catch()` chains or uncaught |
| Response codes | Semantically correct 2xx/4xx/5xx | Always 200, error in body |

---

## Upgrade Advisor (Scanner UA)

> Activates when deprecated APIs, outdated patterns, or legacy syntax is detected.
> Also activates when user says "modernize", "upgrade this", or "use latest patterns".

### Deprecated Pattern Catalog

| Legacy Pattern | Modern Equivalent | Breaking? | Effort |
|---------------|------------------|-----------|--------|
| `var x =` | `const` / `let` | No | Drop-in |
| `React.Component` class | Functional + hooks | No (gradual) | Medium |
| `componentWillMount` | `useEffect(fn, [])` | No | Low |
| `componentDidMount` | `useEffect(fn, [])` | No | Low |
| `componentDidUpdate` | `useEffect(fn, [dep])` | No | Low |
| `moment(date)` | `dayjs(date)` or `date-fns` | Minor API diff | Medium |
| `Promise.then().catch()` chain | `async/await` + try/catch | No | Low |
| `_.get(obj, 'a.b')` | `obj?.a?.b` | No | Drop-in |
| `Object.assign({}, obj)` | `{ ...obj }` | No | Drop-in |
| `Array.from(new Set(arr))` | `[...new Set(arr)]` | No | Drop-in |
| `req.query.id \|\| ''` | `req.query.id ?? ''` | No | Drop-in |
| `height: 100vh` (mobile) | `height: 100dvh` | No | Drop-in |
| `getServerSideProps` for static content | `getStaticProps` | Perf improvement | Low |
| Callback hell (`fn(cb)`) | `async/await` | No | Medium |
| `new Promise(resolve => setTimeout(resolve, ms))` | `await new Promise(r => setTimeout(r, ms))` | No | Drop-in |
| CommonJS `require()` in ESM project | `import` | Yes — check ecosystem | High |
| `React.createRef()` in functional component | `useRef()` | No | Drop-in |
| Redux `connect()` HOC | `useSelector`, `useDispatch` hooks | No (gradual) | Medium |

### Node.js Specific

| Legacy | Modern | Notes |
|--------|--------|-------|
| `fs.readFile(path, cb)` | `fs.promises.readFile(path)` + await | No |
| `http.get(url, cb)` | `fetch(url)` (Node 18+) | Requires Node 18+ |
| `Buffer(size)` | `Buffer.alloc(size)` | Security — old form uninitialized memory |
| `util.promisify(fn)` + await | Native async API (most Node APIs now have them) | Check Node version |

### Python Specific

| Legacy | Modern | Notes |
|--------|--------|-------|
| `%s` string formatting | f-strings | No — Python 3.6+ |
| `str.format()` | f-strings | No |
| `open(f)` without context manager | `with open(f) as ...` | No |
| `dict.has_key(k)` | `k in dict` | Python 3 only |
| Class without `(object)` | Just `class Foo:` | Python 3 style |
| `print "..."` | `print(...)` | Python 3 only |

### Finding Format

```
⬆️ UPGRADE FINDING [UA-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Current    : [deprecated pattern]
File:Line  : [file:N]
Deprecated : [since version X / in favor of Y]
Upgrade To : [modern equivalent]
Effort     : [Low (drop-in) / Medium (refactor) / High (migration)]
Breaking   : [YES — describe impact / NO]
Fix        : [exact migration code]
```

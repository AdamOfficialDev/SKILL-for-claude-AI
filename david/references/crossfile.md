# DAVID — Cross-File Consistency Reference (Scanner AJ)

Scanner AJ detects pattern drift, naming inconsistencies, and structural divergence
across multiple files in the same codebase.

Load this file when the user submits 2+ files, when a multi-file session is active,
or when explicitly asked to check consistency across a codebase.

## Table of Contents
1. [When Scanner AJ Activates](#when-scanner-aj-activates)
2. [Pattern Drift Detection](#pattern-drift-detection)
3. [Naming Consistency Audit](#naming-consistency-audit)
4. [Type & Interface Drift](#type--interface-drift)
5. [Error Handling Consistency](#error-handling-consistency)
6. [API Response Envelope Consistency](#api-response-envelope-consistency)
7. [Import & Dependency Consistency](#import--dependency-consistency)
8. [Test Coverage Consistency](#test-coverage-consistency)
9. [Cross-File Impact Propagation](#cross-file-impact-propagation)
10. [AJ Report Output Format](#aj-report-output-format)

---

## When Scanner AJ Activates

| Trigger | Scope |
|---------|-------|
| 2+ files submitted in one message | All submitted files |
| Multi-turn session with 2+ files audited | All files seen in session |
| `"cek konsistensi"` / `"check consistency"` / `"are these consistent"` | All submitted files |
| `"pattern drift"` / `"naming inconsistency"` | All submitted files |
| After AH (PRDESC) on a multi-file diff | Changed files + their immediate dependents |

**Scope boundary:** AJ only analyzes files DAVID has seen. It never speculates about
files not provided. When relevant files are missing, DAVID flags them using the
Smart File Request Protocol.

---

## Pattern Drift Detection

Pattern drift is when the same logical concern is implemented differently across files.
It is the most common cross-file quality issue in growing codebases.

### Category 1 — Auth & Authorization Pattern Drift

```typescript
// ❌ DRIFT DETECTED — 3 different auth patterns across 3 routes

// routes/users.ts — uses middleware
router.get('/users', requireAuth, async (req, res) => { ... });

// routes/orders.ts — uses manual check
router.get('/orders', async (req, res) => {
  if (!req.user) return res.status(401).json({ error: 'Unauthorized' });
  ...
});

// routes/products.ts — no auth at all (intentional or missed?)
router.get('/products', async (req, res) => { ... });

// ✅ Consistent — always use middleware
router.get('/users',    requireAuth, handleGetUsers);
router.get('/orders',   requireAuth, handleGetOrders);
router.get('/products', requireAuth, handleGetProducts);  // or explicitly mark as public
```

**DAVID flags:** Any route that has auth implemented differently from the majority pattern.
The route without auth is flagged as SEC finding if it handles non-public data.

### Category 2 — Error Handling Pattern Drift

```typescript
// ❌ DRIFT — 4 different error patterns across controllers

// UserController: returns string
return res.status(400).json("Invalid input");

// OrderController: returns object with 'error' key
return res.status(400).json({ error: "Invalid input" });

// ProductController: returns object with 'message' key
return res.status(400).json({ message: "Invalid input" });

// PaymentController: throws and relies on global handler
throw new ValidationError("Invalid input");

// ✅ Consistent — always use the same error envelope
return res.status(400).json({ error: "Invalid input", code: "VALIDATION_ERROR" });
```

### Category 3 — Data Fetching Pattern Drift

```typescript
// ❌ DRIFT — mixing direct db calls and repository pattern

// UserService: direct Prisma
const user = await prisma.user.findUnique({ where: { id } });

// OrderService: via repository
const order = await orderRepository.findById(id);

// ProductService: direct Prisma
const product = await prisma.product.findFirst({ where: { id } });
```

**DAVID reports:** Which pattern is used more (the "majority pattern") and flags
the minority uses as drift. Does not force one pattern — reports the inconsistency
and asks which to standardize on.

### Category 4 — Async / Promise Handling Drift

```typescript
// ❌ DRIFT — mixing async/await and .then()/.catch() in same codebase

// userService.ts
async function getUser(id) {
  return await db.user.findUnique({ where: { id } });
}

// orderService.ts
function getOrder(id) {
  return db.order.findUnique({ where: { id } })
    .then(order => order)
    .catch(err => { throw err; });
}
```

---

## Naming Consistency Audit

### Variable / Parameter Naming Drift

```typescript
// ❌ DRIFT — same concept, different names across files

// auth.ts
function validateUser(userId: string) { ... }
function getUserData(user_id: string) { ... }  // snake_case here

// orders.ts  
function getOrdersForUser(uid: string) { ... }  // uid here
function createOrder(userIdentifier: string) { ... }  // different again
```

**DAVID's consistency report format:**
```
🔍 NAMING DRIFT: User identifier param
  auth.ts        → userId (3 uses)
  auth.ts        → user_id (1 use)   ← drift
  orders.ts      → uid (2 uses)      ← drift
  orders.ts      → userIdentifier (1 use)  ← drift

  Majority pattern: userId (camelCase)
  Recommendation: Standardize to `userId` across all files.
  Files to update: auth.ts line 24, orders.ts lines 11, 47
```

### Function Naming Convention Drift

```typescript
// ❌ DRIFT — mixed conventions for same operation type

// UserRepository
getUserById(id)      // get prefix
fetchOrderById(id)   // fetch prefix — inconsistent

// ProductRepository
findProductById(id)  // find prefix — third convention
readCategory(id)     // read prefix — fourth convention
```

**Pattern:** DAVID identifies the dominant prefix (`get`, `fetch`, `find`) and flags
deviations. In a team codebase, consistent function naming is load-bearing for
discoverability.

### File / Module Naming Drift

```
❌ DRIFT detected in file naming:
  src/services/UserService.ts      → PascalCase
  src/services/order-service.ts    → kebab-case
  src/services/productservice.ts   → lowercase, no separator
  src/services/auth_service.ts     → snake_case

✅ Pick one convention and apply consistently.
  Most common in this repo: PascalCase → rename deviating files.
```

---

## Type & Interface Drift

### Shared Entity Shape Drift

```typescript
// ❌ DRIFT — User type defined differently in 3 places

// types/user.ts
interface User {
  id: string;
  email: string;
  name: string;
}

// api/responses.ts — same concept, different shape
type UserResponse = {
  userId: string;    // id vs userId
  emailAddress: string;  // email vs emailAddress
  fullName: string;  // name vs fullName
}

// db/models.ts — yet another
type UserRecord = {
  user_id: number;  // string vs number!
  user_email: string;
}
```

**DAVID flags:** Shape drift on the same domain entity is the #1 cause of runtime bugs
in TypeScript projects. Reports all definitions and their locations, highlights
field name mismatches and type mismatches (especially string vs number).

### DTO / API Contract Drift

```typescript
// ❌ Request DTOs inconsistent between routes

// POST /users — validates with Zod
const body = CreateUserSchema.parse(req.body);

// POST /orders — manual validation
if (!req.body.userId) return res.status(400)...
if (!req.body.amount) return res.status(400)...

// POST /products — no validation at all
const { name, price } = req.body;
```

---

## Error Handling Consistency

### Async Error Boundary Drift

```typescript
// ❌ DRIFT — some async functions have try/catch, others don't

// userService.ts
async function createUser(data) {
  try {
    return await db.user.create({ data });
  } catch (err) {
    throw new DatabaseError(err);
  }
}

// orderService.ts — no error handling
async function createOrder(data) {
  return await db.order.create({ data });  // unhandled rejection if DB fails
}

// productService.ts — swallows error silently
async function createProduct(data) {
  try {
    return await db.product.create({ data });
  } catch (err) {
    console.error(err);  // ← silent failure: returns undefined to caller
    // no rethrow, no return value
  }
}
```

**DAVID output:** Error coverage map across all submitted files:

```
🛡️ ERROR HANDLING COVERAGE MAP (Scanner AJ + R)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
File                  │ Async fns │ Handled │ Silent │ Missing
──────────────────────│───────────│─────────│────────│────────
userService.ts        │     4     │    4    │    0   │    0   ✅
orderService.ts       │     6     │    2    │    1   │    3   ❌
productService.ts     │     3     │    1    │    2   │    0   ⚠️
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Overall coverage: 7/13 (54%) — Target: >90%
```

---

## API Response Envelope Consistency

### Envelope Drift (most common in Express/FastAPI/NestJS)

```typescript
// ❌ DRIFT — 4 different response shapes across endpoints

GET /users     → { users: User[] }
GET /users/:id → User  (no wrapper)
POST /users    → { data: User, message: "Created" }
DELETE /users/:id → "deleted"  (string!)
```

**Standard envelope patterns** — DAVID identifies which pattern is dominant:

```typescript
// Pattern A — Data wrapper (most common)
{ data: T, meta?: { total, page } }

// Pattern B — Named resource
{ users: User[] }  /  { user: User }

// Pattern C — JSend
{ status: "success", data: T }  /  { status: "error", message: string }
```

**DAVID report:**
```
📋 API ENVELOPE CONSISTENCY (Scanner AJ + AA)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Dominant pattern : { data: T } wrapper (used in 6/10 endpoints)
Drifting endpoints:
  GET /users/:id  → returns User directly (no wrapper)
  DELETE /users/:id → returns string "deleted"
  POST /login     → returns { token, user } (custom shape)

Recommendation: Standardize all to { data: T } envelope.
```

---

## Import & Dependency Consistency

### Utility Library Drift

```typescript
// ❌ DRIFT — 3 date libraries used across codebase

// userService.ts
import moment from 'moment';
const now = moment().toISOString();

// orderService.ts
import { format } from 'date-fns';
const now = format(new Date(), 'yyyy-MM-dd');

// reportService.ts
const now = new Date().toISOString();  // native
```

**DAVID flags:** Multiple libraries solving the same problem. Reports which is used most
frequently and which should be removed. Checks `package.json` if available to
confirm whether unused libraries are still in deps.

### Relative vs Absolute Import Drift

```typescript
// ❌ DRIFT — mixing relative and absolute imports

// Some files use relative
import { UserService } from '../services/user';
import { OrderRepository } from '../../repositories/order';

// Others use absolute (tsconfig path alias)
import { UserService } from '@/services/user';
import { OrderRepository } from '@/repositories/order';
```

---

## Test Coverage Consistency

### Module Coverage Drift

```
📊 TEST COVERAGE CONSISTENCY (Scanner AJ + TQ)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Module              │ Has tests? │ Approx. coverage
────────────────────│────────────│──────────────────
userService.ts      │ ✅ Yes     │ ~80% (15 test cases)
orderService.ts     │ ✅ Yes     │ ~40% (4 test cases — weak)
productService.ts   │ ❌ No      │ 0%
paymentService.ts   │ ❌ No      │ 0%  ← HIGH RISK (payment logic!)
authMiddleware.ts   │ ✅ Yes     │ ~90% (comprehensive)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Priority: Add tests for paymentService.ts first (P0 risk, zero coverage)
```

---

## Cross-File Impact Propagation

When DAVID fixes a bug or changes a function signature in File A, Scanner AJ
identifies all callers in other files that may need updating.

### Impact propagation output

```
⚡ CROSS-FILE IMPACT — getUserProfile() signature change
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CHANGED: userService.ts
  getUserProfile(userId: string)
  →  getUserProfile(userId: string, includeDeleted?: boolean)

CALLERS THAT NEED REVIEW:
  ✅ pages/profile.tsx line 24 — passes only userId (safe, new param is optional)
  ✅ api/admin/users.ts line 67 — passes only userId (safe)
  ⚠️ tests/userService.spec.ts line 45 — mock may need updating
  ❌ scripts/migrate-users.ts line 12 — passes hardcoded second arg (may break)

ACTION REQUIRED: Review scripts/migrate-users.ts line 12 before shipping.
```

### Propagation depth rules

- **Depth 1** (direct callers) — always checked
- **Depth 2** (callers of callers) — checked only if Depth 1 has a public API change
- **Depth 3+** — flagged as "requires full impact analysis; provide more files"

---

## AJ Report Output Format

Full cross-file consistency report structure:

```
🔗 CROSS-FILE CONSISTENCY REPORT (Scanner AJ)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files analyzed: [N] files
Session files : [list of filenames]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATTERN DRIFT
  [finding card per drift pattern detected]

NAMING INCONSISTENCY
  [finding card per naming drift]

TYPE / INTERFACE DRIFT
  [finding card per shape mismatch]

API ENVELOPE CONSISTENCY
  [finding card if envelope patterns diverge]

ERROR HANDLING MAP
  [table: file × async functions × covered]

IMPORT CONSISTENCY
  [finding card per library duplication]

CROSS-FILE IMPACT (from current session fixes)
  [impact table for each changed function]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AJ CONSISTENCY SCORE: [N]/100
  Pattern drift   : [N] instances
  Naming drift    : [N] instances
  Type drift      : [N] instances
  Envelope drift  : [consistent / N drifting]
  Error coverage  : [N]% across files
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### AJ Consistency Score weights

| Category | Weight |
|----------|--------|
| Pattern drift (auth, error handling, async) | 35% |
| Naming consistency (variables, functions, files) | 20% |
| Type / interface shape consistency | 25% |
| API envelope consistency | 10% |
| Import / dependency consistency | 10% |

Score bands: **90–100** Excellent · **75–89** Good · **60–74** Needs work · **<60** Critical drift

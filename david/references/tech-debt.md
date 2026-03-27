# DAVID — Tech Debt Reference (Scanners Q, R, S, Z, AD)

Patterns for dead code, error coverage, cyclomatic complexity, feature flags, and tech debt heatmap.

---

## Dead Code Detection (Scanner Q)

### Zombie Export Detection

DAVID cross-references exports against imports across all provided files:

```typescript
// In utils/format.ts:
export function formatCurrency(amount: number): string { ... }  // Used in 3 files ✅
export function formatSSN(ssn: string): string { ... }           // Used in 0 files 💀
export function legacyFormat(x: any): string { ... }             // Used in 0 files 💀

// DAVID output:
// 💀 DEAD CODE [DEAD-001]: formatSSN — exported but never imported (0 usages found)
// 💀 DEAD CODE [DEAD-002]: legacyFormat — exported but never imported (0 usages found)
```

### Dead Condition Detection

```typescript
// ❌ Condition always true/false
const IS_PROD = true;
if (!IS_PROD) {
  doDevOnlyThing();  // 💀 Never executes
}

// ❌ Impossible type check
function process(id: string) {
  if (typeof id === 'number') {  // 💀 TypeScript says this is always false
    handleNumber(id);
  }
}

// ❌ Default case that can't be reached (with exhaustive union)
type Status = 'active' | 'inactive';
switch (status) {
  case 'active': return 'green';
  case 'inactive': return 'red';
  default: return 'gray';  // 💀 Unreachable if Status is exhaustive
}
```

### CSS Dead Classes

```css
/* ❌ Class defined but never used in templates */
.card-header-v1 { ... }    /* v2 replaced this — never referenced */
.btn-legacy { ... }         /* Migration to new system complete */
.modal-old-style { ... }    /* Old modal removed 6 months ago */
```

---

## Error Coverage Map (Scanner R)

### Coverage Computation

DAVID counts and reports:

```
📊 ERROR HANDLING COVERAGE REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
File: src/services/payment.ts

Async functions total  : 8
  With try/catch       : 5 (62.5%)
  With .catch()        : 1 (12.5%)
  UNHANDLED            : 2 (25.0%) ← processRefund, scheduleRetry

Promise chains total   : 3
  With rejection handler: 2 (66.7%)
  FIRE-AND-FORGET      : 1 (33.3%) ← sendWebhook

API calls total        : 6
  With error UI        : 4 (66.7%)
  Silent on failure    : 2 (33.3%) ← updateProfile, logEvent

Overall error coverage : 65% → Target: >90%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Silence Pattern Library

```typescript
// ALL of these are flagged as "unhandled" by DAVID:

// 1. No handler at all
await sendEmail(user);

// 2. Empty catch
try { await sendEmail(user); } catch {}
try { await sendEmail(user); } catch(e) {}

// 3. Log-only (not handled, just noted)
try { await sendEmail(user); } catch(e) { console.error(e); }
// ✅ Needs: retry logic, fallback, or re-throw

// 4. Fire and forget promise
sendAnalyticsEvent(data);  // No await, no .catch()

// 5. .then() without .catch()
fetchUser(id).then(setUser);

// 6. Null-coalescing on async result without error handling
const user = await getUser(id).catch(() => null);  // Error silenced
// ✅ OK if null is handled downstream, flag for review

// 7. Result type (Go-style) where error is never checked
const [err, data] = await to(fetchUser(id));
console.log(data.name);  // err never inspected
```

---

## Cyclomatic Complexity (Scanner S)

### CC Computation Example

```javascript
// DAVID counts this function's CC:
function processOrder(order, user, config) {           // +1 base
  if (!order || !user) {                               // +1 if
    return null;
  }
  
  if (order.status !== 'pending') {                    // +1 if
    throw new Error('Order not pending');
  }
  
  const discount = user.isPremium                      // +1 ternary
    ? config.premiumDiscount
    : config.standardDiscount;
  
  for (const item of order.items) {                    // +1 for
    if (item.quantity <= 0) continue;                  // +1 if
    
    if (item.type === 'physical') {                    // +1 if
      validateShipping(item);
    } else if (item.type === 'digital') {              // +1 else if
      validateDownload(item);
    }
    
    if (!item.inStock && config.allowBackorder) {      // +1 if, +1 &&
      flagForBackorder(item);
    }
  }
  
  return calculateTotal(order, discount);
}
// Total CC = 10 — Moderate, acceptable but watch it
```

### Extraction Guide

When CC > 15, DAVID suggests specific extractions:

```javascript
// High CC function (CC = 22):
function validateAndProcessPayment(payment, order, user, config) {
  // Validation block (CC contribution: 6)
  if (!payment.cardNumber) throw new Error('...');
  if (!luhnCheck(payment.cardNumber)) throw new Error('...');
  if (!payment.cvv || payment.cvv.length < 3) throw new Error('...');
  if (payment.expiryYear < new Date().getFullYear()) throw new Error('...');
  if (payment.expiryYear === new Date().getFullYear() &&
      payment.expiryMonth < new Date().getMonth() + 1) throw new Error('...');
  
  // Processing block (CC contribution: 10)
  // ... complex processing ...
  
  // Notification block (CC contribution: 6)
  // ... notifications ...
}

// DAVID suggests extracting into:
function validateCard(payment) { ... }         // CC = 6
function processPaymentLogic(payment, order) { ... } // CC = 10
function notifyPaymentComplete(order, user) { ... }  // CC = 6

function validateAndProcessPayment(payment, order, user, config) {
  validateCard(payment);            // CC of this function now = 1
  const result = processPaymentLogic(payment, order);
  notifyPaymentComplete(order, user);
  return result;
}
```

---

## Feature Flag Inventory (Scanner Z)

### Flag Pattern Detection

```javascript
// DAVID detects ALL of these as feature flags:

// 1. Environment variable flags
if (process.env.FEATURE_NEW_CHECKOUT === 'true') { ... }
if (process.env.NEXT_PUBLIC_ENABLE_DARK_MODE) { ... }

// 2. Hardcoded boolean flags
const ENABLE_V2_API = true;
const USE_LEGACY_PARSER = false;

// 3. Config object flags
if (config.features.socialLogin) { ... }
if (appConfig.experimental.newUI) { ... }

// 4. Constants that look like flags
const SHOW_BETA_BADGE = true;
const IS_MAINTENANCE_MODE = false;
```

### Flag Health Classification

| Flag State | Signal | Action |
|-----------|--------|--------|
| Always `true` hardcoded | Flag is dead — rollout complete | Remove flag, keep code |
| Always `false` hardcoded | Feature was abandoned or toggled off | Remove flag + dead code |
| Read from config/env | Active flag | Keep, document purpose |
| Same logic in multiple files | Inconsistent flag usage | Centralize |
| Flag checks with no else branch | May have orphaned code paths | Review |

---

## Tech Debt Heatmap Scoring

### Score Calculation

DAVID computes a debt score per file:

```
File Score = (TODO × 2) + (FIXME × 3) + (HACK × 4) + (XXX × 3)
           + (CC_violations × 5)      # Functions with CC > 15
           + (any_count × 2)           # TypeScript `any` uses
           + (dead_exports × 3)        # Unused exports
           + (empty_catch × 4)         # Silent error swallowing
           + (hardcoded_flag × 3)      # Boolean feature flags
```

Score interpretation: 0 = clean, 1–10 = minor, 11–20 = needs attention, >20 = urgent

### TODO/FIXME Pattern Library

```javascript
// DAVID inventories ALL of these:
// TODO: refactor this when new API is ready
// TODO(alice): clean up after migration — ticket PROJ-123
// FIXME: this crashes when list is empty
// FIXME(bob): race condition here — reproduce with 2 concurrent requests
// HACK: workaround for Safari bug — remove when Safari 17 is baseline
// XXX: this is O(n²) and will blow up at scale
// DEPRECATED: use newMethod() instead
// NOTE: this is intentionally duplicated — do not DRY
```

DAVID extracts:
- Who owns it (if name in parens)
- Ticket reference (if present)
- Age (if file has git blame available)
- Urgency indicator (FIXME vs TODO vs HACK)

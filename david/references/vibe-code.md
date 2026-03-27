# DAVID — Vibe Code Detector Reference

Full pattern library for Scanner AK (🎯 VIBECODE). Load this when the user mentions AI-generated code, Copilot/Cursor/ChatGPT output, "just paste and fix", or when code exhibits LLM artifact signatures without explicit mention.

---

## Table of Contents
1. [What Is Vibe Code?](#what-is-vibe-code)
2. [Activation Heuristics](#activation-heuristics)
3. [Pattern 1 — TODO Debris](#pattern-1--todo-debris)
4. [Pattern 2 — Hallucinated APIs](#pattern-2--hallucinated-apis)
5. [Pattern 3 — Console.log Archaeology](#pattern-3--consolelog-archaeology)
6. [Pattern 4 — The `any` Epidemic (TypeScript)](#pattern-4--the-any-epidemic-typescript)
7. [Pattern 5 — Missing Edge Cases](#pattern-5--missing-edge-cases)
8. [Pattern 6 — Copy-Paste Boilerplate Soup](#pattern-6--copy-paste-boilerplate-soup)
9. [Pattern 7 — Unused Imports & Dead Variables](#pattern-7--unused-imports--dead-variables)
10. [Pattern 8 — Over-Generic Naming](#pattern-8--over-generic-naming)
11. [Pattern 9 — Optimistic-Only UI (No Error/Loading State)](#pattern-9--optimistic-only-ui)
12. [Pattern 10 — Magic Numbers & Strings](#pattern-10--magic-numbers--strings)
13. [Pattern 11 — Shallow Mocking in Tests](#pattern-11--shallow-mocking-in-tests)
14. [Pattern 12 — README/Comment Mismatch](#pattern-12--readmecomment-mismatch)
15. [Vibe Code Confidence Score](#vibe-code-confidence-score)
16. [DAVID's Tone in Vibe Code Mode](#davids-tone-in-vibe-code-mode)

---

## What Is Vibe Code?

Vibe code is code generated quickly by AI tools (ChatGPT, GitHub Copilot, Cursor, Claude, etc.) that:
- **Compiles and runs** — it looks correct at first glance
- **Lacks critical review** — the human hit "accept" without deeply reading
- **Has blind spots** — AI tools optimize for "plausible completion", not correctness
- **Accumulates entropy** — each AI generation adds a little more debt

Vibe code is not bad by definition. The problem is **unreviewed vibe code** in production.

DAVID's job in this mode: find the blind spots the AI left behind, explain why they matter, and fix them — while teaching the user to spot them next time.

---

## Activation Heuristics

DAVID auto-activates Scanner AK when ANY of these are true:

**Explicit triggers (user says it):**
- "AI wrote this", "ChatGPT made this", "Copilot generated"
- "vibe coding", "vibe coded", "vibed this out"
- "just paste and fix", "generated this, check it"
- "cursor did this", "claude wrote this", "ai slop"

**Implicit triggers (code patterns):**
- ≥3 TODO/FIXME comments in < 100 lines
- ≥3 `console.log` or `print(` debug statements
- `any` used ≥5 times in a TypeScript file
- Same error-handling block copy-pasted ≥3 times
- Function names like `handleData`, `processResult`, `doStuff`, `handleThing`
- Import of a package that doesn't exist on npm/pypi
- Function body that references a method never defined anywhere

**Confidence threshold:** If ≥3 implicit triggers fire, DAVID reports `AI Artifact Confidence: HIGH` and activates Scanner AK automatically.

---

## Pattern 1 — TODO Debris

AI tools write TODO comments as placeholders they never follow up on. These become permanent technical debt.

**What DAVID flags:**
```javascript
// TODO: implement error handling         ← Left by AI, never done
// TODO: add validation here              ← Left by AI, never done
// TODO: handle edge cases                ← Left by AI, never done
// TODO: optimize this later              ← "Later" = never
// FIXME: this might break for large inputs  ← AI admitted the bug, didn't fix it
// NOTE: this is a simplified version     ← Vague, no action
```

**Triage categories:**

| TODO type | DAVID action |
|-----------|-------------|
| `TODO: implement X` — X is critical path | 🔴 P0 — implement it now or block feature |
| `TODO: add validation` | 🟠 P1 — generate validation immediately |
| `TODO: optimize` / `TODO: refactor` | 🟡 P2 — add to tech debt heatmap |
| `TODO: add tests` | 🟠 P1 — generate tests now |
| `FIXME: might break` | 🟠 P1 — confirm if it breaks, fix it |

**DAVID output:**
```
🎯 VIBE CODE [VC-TODO-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pattern   : TODO Debris
File:Line : src/api/payment.ts:47
Content   : "// TODO: implement error handling"
Severity  : 🟠 P1 — this is a payment endpoint with no error handling
Action    : [DAVID generates the missing error handling inline]
```

---

## Pattern 2 — Hallucinated APIs

AI tools confidently reference methods, packages, and APIs that don't exist. These compile fine but crash at runtime.

**Common hallucination categories:**

### Non-existent package methods
```javascript
// ❌ Hallucinated React Query API
import { useOptimisticMutation } from 'react-query';
// Real: useMutation({ onMutate }) — no such hook as useOptimisticMutation

// ❌ Hallucinated Prisma method
await db.user.findAll({ where: { active: true } });
// Real: db.user.findMany() — Prisma uses findMany, not findAll

// ❌ Hallucinated Express method
app.delete('/users/:id', handler);
// This exists — but check for:
app.get('/users/:id').delete(handler);  // Method chaining that doesn't work

// ❌ Hallucinated Axios config
axios.get(url, { retry: 3 });  // Axios has no built-in retry option
```

### Invented package names
```javascript
import sanitize from 'html-sanitizer';        // Doesn't exist → should be 'dompurify'
import { validate } from 'express-validate';  // Doesn't exist → should be 'express-validator'
import cryptoUtils from 'node-crypto-utils';  // Not a real package
```

### Non-existent Python packages/methods
```python
import pandas as pd
df.filter_by_value(column='age', min=18)  # No such method — use df[df['age'] >= 18]

from fastapi import JSONResponse, HTMLResponse, FileResponse
from fastapi import StreamingResponse  # Real
from fastapi import BinaryResponse     # ❌ Hallucinated — doesn't exist
```

**DAVID verification protocol:**
1. Check if the import matches a known, real package in DAVID's knowledge
2. Check if the method exists in the package's public API
3. If uncertain (60–80% confidence): flag with `🤖 DAVID ASSUMPTION: Unverified API — confirm this method exists`
4. If clearly hallucinated (>80% confidence): flag as P0, provide correct alternative

**Finding format:**
```
🎯 VIBE CODE [VC-API-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pattern    : Hallucinated API
File:Line  : src/hooks/useData.ts:3
Code       : import { useOptimisticMutation } from 'react-query'
Confidence : 95% — this hook does not exist in react-query
Risk       : Runtime crash — "useOptimisticMutation is not a function"
Fix        : Use useMutation with onMutate for optimistic updates:
             [DAVID provides correct implementation]
```

---

## Pattern 3 — Console.log Archaeology

AI tools use `console.log` liberally during generation. These almost never get cleaned up.

**What DAVID flags:**
```javascript
console.log("here");                      // ← Location debug, meaningless
console.log("data:", data);               // ← Data inspection left in
console.log("testing 123");              // ← Test print
console.log("it works!");                // ← Confirmation debug
console.log(JSON.stringify(user, null, 2)); // ← Deep inspection, left in
console.log("response:", response);      // ← API response debug
print(f"DEBUG: {variable}")             // Python equivalent
fmt.Println("DEBUG:", variable)         // Go equivalent
```

**Categorization:**

| Type | Severity | Action |
|------|----------|--------|
| Debug location (`"here"`, `"step 1"`) | 🟡 P2 | Remove entirely |
| Data dump (`console.log("data:", x)`) | 🟡 P2 | Remove or convert to `logger.debug` |
| PII in log (`console.log("user:", user)`) | 🟠 P1 | Remove immediately — potential data leak |
| Error swallowed then logged (`catch(e){ console.log(e) }`) | 🟠 P1 | Convert to proper error handler |

**DAVID never silently removes** console.logs — always shows what it removed and why.

---

## Pattern 4 — The `any` Epidemic (TypeScript)

AI tools collapse to `any` when they're uncertain about types — which is often. This defeats the entire purpose of TypeScript.

**Detection patterns:**
```typescript
// ❌ Tier 1: Direct any
const data: any = await fetchUsers();
function process(input: any): any { ... }
const config: any = { ... };

// ❌ Tier 2: as any cast (type erasure)
const result = (response as any).data;
const user = (thing as any).user.name;  // Will crash if thing.user is null

// ❌ Tier 3: Suppression comments
// @ts-ignore                    // Suppressing a real problem
// @ts-expect-error              // Without explaining why

// ❌ Tier 4: Indirect any
const res = await fetch(url);
const json = await res.json();   // json is typed as any — downstream infects everything
const items = json.items;        // Now items is any too

// ❌ Tier 5: Object/Function types that are secretly any
function transform(data: object): object { ... }  // object is barely better than any
const handler: Function = () => { ... };           // Function loses all param types
```

**DAVID's fix protocol — always infer or provide the real type:**
```typescript
// For API responses: generate interface from usage
interface UsersResponse {
  items: User[];
  total: number;
  page: number;
}

// For fetch().json(): cast at the boundary, not downstream
const json = await res.json() as UsersResponse;

// For unknown shapes: use unknown + type guard instead of any
function process(input: unknown): ProcessedResult {
  if (!isValidInput(input)) throw new TypeError('Invalid input shape');
  return transform(input);
}
```

**Epidemic score:**
```
🔷 TYPE EPIDEMIC SCORE
━━━━━━━━━━━━━━━━━━━━━━━
Direct `any`         : [N] usages
`as any` casts       : [N] usages
@ts-ignore           : [N] suppressions
Untyped fetch().json : [N] call sites
─────────────────────
Total any surface    : [N] — [Low/Medium/High/Critical]
Estimated type safety: [XX]%
```

---

## Pattern 5 — Missing Edge Cases

AI tools write the happy path brilliantly. They consistently miss the edges. This is the most dangerous vibe code pattern.

**The classic blind spots:**

```javascript
// ❌ Division without zero check
function getAverage(nums) {
  return nums.reduce((a, b) => a + b, 0) / nums.length;
  // Crash: nums = [] → 0 / 0 = NaN, returned silently
}

// ❌ Array access without bounds check
function getFirst(arr) {
  return arr[0].id;  // Crash: arr = [] → arr[0] = undefined → undefined.id throws
}

// ❌ Object access without null check
function getUserName(userId) {
  const user = users.find(u => u.id === userId);
  return user.name;  // Crash: no user found → user = undefined → undefined.name throws
}

// ❌ String operations without type check
function slugify(title) {
  return title.toLowerCase().replace(/\s+/g, '-');
  // Crash: title = null/undefined → null.toLowerCase() throws
}

// ❌ Async without rejection handling
async function loadDashboard() {
  const data = await fetchDashboardData();
  setData(data);
  // If fetchDashboardData rejects → unhandled promise rejection
  // UI hangs in loading state forever
}

// ❌ parseInt/parseFloat without NaN check
function calculateTotal(price, qty) {
  return parseInt(price) * parseInt(qty);
  // If price = "abc" → NaN * qty = NaN → total shown as NaN in UI
}
```

**DAVID's edge case checklist — for every function:**
```
□ Empty/null/undefined input → does it crash or return sensible default?
□ Zero value → off-by-zero? NaN? Division by zero?
□ Boundary value → at the limit (max, min, exactly 1)?
□ Wrong type → string where number expected? null where object expected?
□ Async rejection → what happens if the promise fails?
□ Empty array/list → does it handle zero items?
□ Long string/large number → does it clip, overflow, or handle gracefully?
```

---

## Pattern 6 — Copy-Paste Boilerplate Soup

AI tools generate self-contained, complete blocks per request. Across multiple requests, these blocks duplicate heavily.

**What DAVID finds:**

```javascript
// ❌ Same try/catch pattern copy-pasted across N files
// In api/users.ts:
try {
  const result = await db.query('SELECT ...');
  return res.json({ success: true, data: result });
} catch (err) {
  console.error(err);
  return res.status(500).json({ success: false, error: 'Internal error' });
}

// In api/orders.ts: IDENTICAL (copy-paste)
try {
  const result = await db.query('SELECT ...');
  return res.json({ success: true, data: result });
} catch (err) {
  console.error(err);
  return res.status(500).json({ success: false, error: 'Internal error' });
}
// → Extract to: asyncHandler(fn) middleware
```

```javascript
// ❌ Same validation logic duplicated across routes
// 5 different API routes all manually checking:
if (!req.body.email || !req.body.email.includes('@')) {
  return res.status(400).json({ error: 'Invalid email' });
}
// → Extract to: validateEmail(email) or zod schema
```

```javascript
// ❌ Same auth check copy-pasted into every route handler
// → Should be an auth middleware applied once
```

**DAVID's DRY proposal format:**
```
📋 DUPLICATE DETECTED [VC-DRY-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pattern   : Error handler boilerplate
Occurrences: 6 files (api/users.ts:45, api/orders.ts:67, ...)
Lines     : ~12 lines × 6 = 72 duplicated lines
Fix       : Extract to shared asyncHandler(fn) utility
Savings   : 60 lines removed, single point of maintenance
[DAVID provides the extracted utility + updated call sites]
```

---

## Pattern 7 — Unused Imports & Dead Variables

AI tools import everything they think they might need. Half get abandoned.

```javascript
// ❌ Hallucinated then unused
import { useState, useEffect, useCallback, useMemo, useRef, useContext } from 'react';
// Only useState and useEffect used

import { Button, Input, Modal, Select, Checkbox, DatePicker, Tooltip } from 'antd';
// Only Button and Input used

// ❌ Assigned but never read
const config = loadConfig();       // config never referenced below
const logger = createLogger();     // logger created but console.log used instead
const { data, error, loading } = useQuery(QUERY);  // error and loading never used
```

**Python equivalents:**
```python
import os, sys, json, re, datetime, collections  # Only os used
from typing import List, Dict, Optional, Union, Tuple  # Only List used
```

**DAVID's rule:** Never silently remove unused imports. Always show the list and confirm — one of them might be a side-effect import that's intentional (e.g., `import './polyfills'`).

---

## Pattern 8 — Over-Generic Naming

AI tools name things generically when they lack domain context. This makes code unreadable for humans.

**Detection patterns:**
```javascript
// ❌ No domain meaning
const data = await fetchData();
const result = processResult(data);
const output = handleOutput(result);
function doStuff(thing) { ... }
function handleThing(item) { ... }
const temp = calculateValue();
const x = getInfo();

// ✅ Domain-meaningful
const invoices = await fetchUserInvoices(userId);
const filteredInvoices = applyDateRangeFilter(invoices, startDate, endDate);
const invoiceSummary = summarizeByMonth(filteredInvoices);
```

**Variables to flag:**
- `data`, `result`, `output`, `info`, `thing`, `item`, `obj`, `temp`, `val`, `x`, `y`
- `handleX` where X is also generic
- `processX`, `doX`, `getX` with no domain noun

**DAVID's approach:** Suggests better names based on context — reads the surrounding code to infer domain intent.

---

## Pattern 9 — Optimistic-Only UI

AI tools write the success path UI. Loading, error, and empty states are afterthoughts.

```jsx
// ❌ Classic vibe code UI — only happy path
function UserProfile({ userId }) {
  const { data } = useQuery(['user', userId], fetchUser);

  return (
    <div>
      <h1>{data.name}</h1>       {/* Crash if data is null/undefined */}
      <p>{data.email}</p>
      <img src={data.avatar} />
    </div>
  );
}

// ✅ All states handled
function UserProfile({ userId }) {
  const { data, isLoading, isError } = useQuery(['user', userId], fetchUser);

  if (isLoading) return <UserProfileSkeleton />;
  if (isError)   return <ErrorState message="Couldn't load profile" retry />;
  if (!data)     return <EmptyState message="User not found" />;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
      <img src={data.avatar} alt={`${data.name}'s avatar`} />
    </div>
  );
}
```

**The 4-state matrix DAVID checks every data-driven UI against:**
```
□ Loading state    → skeleton / spinner / shimmer
□ Error state      → error message + retry option
□ Empty state      → empty-state illustration + CTA
□ Success state    → the actual content
```

---

## Pattern 10 — Magic Numbers & Strings

AI tools inline constants without naming them.

```javascript
// ❌ Magic numbers / strings from AI generation
if (user.age > 18) { ... }              // Why 18? Legal age? Voting? Drinking?
setTimeout(callback, 3000);             // Why 3000? Is it 3 seconds? Why 3?
if (items.length > 100) paginate();     // Why 100? Arbitrary?
const rate = amount * 0.029 + 0.30;    // Stripe's rate — but completely opaque
if (status === 'PNDG') { ... }         // Abbreviation — what does PNDG mean?

// ✅ Named constants with intent
const LEGAL_MINIMUM_AGE = 18;
const TOAST_AUTO_DISMISS_MS = 3000;
const PAGINATION_THRESHOLD = 100;
const STRIPE_PERCENTAGE_FEE = 0.029;
const STRIPE_FLAT_FEE_CENTS = 0.30;
const ORDER_STATUS = { PENDING: 'PNDG', COMPLETED: 'CMPL', REFUNDED: 'RFND' };
```

---

## Pattern 11 — Shallow Mocking in Tests

When AI generates tests, it often mocks everything — resulting in tests that test nothing real.

```javascript
// ❌ Test that only tests the mock, not the code
it('creates a user', async () => {
  const mockCreate = jest.fn().mockResolvedValue({ id: 1, name: 'Alice' });
  UserService.prototype.create = mockCreate;

  const result = await UserService.create({ name: 'Alice' });

  expect(mockCreate).toHaveBeenCalledWith({ name: 'Alice' });
  // This test only proves that mocks work, not that the service does
});

// ✅ Test actual behavior with controlled dependencies
it('creates a user and hashes the password', async () => {
  const fakeDb = new InMemoryUserRepository();
  const service = new UserService(fakeDb);

  const user = await service.create({ name: 'Alice', password: 'plaintext' });

  expect(user.id).toBeDefined();
  expect(user.password).not.toBe('plaintext');  // Confirms hashing happened
  expect(fakeDb.users).toHaveLength(1);         // Confirms persistence
});
```

---

## Pattern 12 — README/Comment Mismatch

AI regenerates READMEs and comments based on what it thinks the code does — often wrong after edits.

**What DAVID checks:**
- Function JSDoc that describes different behavior than the actual implementation
- README installation steps referencing packages not in package.json
- `@param` docs with wrong types or names vs actual signature
- Comments referencing deleted/renamed variables or functions

```javascript
// ❌ Comment lies about what the function does
/**
 * Fetches user data from the database.
 * @param {string} userId - The user's UUID
 * @returns {Promise<User>} The user object
 */
async function getUserProfile(id) {   // param renamed but doc still says userId
  const user = await cache.get(id);   // Fetches from CACHE, not database!
  if (!user) {
    return null;                       // Returns null, not throws — doc says returns User
  }
  return user;
}
```

---

## Vibe Code Confidence Score

DAVID computes a Vibe Code Confidence Score at the start of AK scan:

```
🎯 VIBE CODE CONFIDENCE ASSESSMENT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Signal                          │ Found │ Weight
────────────────────────────────│───────│────────
TODO/FIXME debris               │  YES  │ +20%
Hallucinated API call           │  YES  │ +25%
console.log debris (3+)         │  YES  │ +15%
`any` epidemic (5+)             │   NO  │  +0%
Copy-paste duplicates (3+)      │  YES  │ +15%
Over-generic naming (5+)        │  YES  │ +10%
Missing error/loading state     │  YES  │ +10%
Unused imports (5+)             │   NO  │  +0%
────────────────────────────────│───────│────────
AI Artifact Confidence          │       │  95%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Assessment: HIGH — This code shows strong AI-generation signatures.
DAVID will run full vibe code audit + standard scanners.
```

---

## DAVID's Tone in Vibe Code Mode

When in Vibe Code mode, DAVID shifts to **educational + constructive** — not critical.

**Principles:**
1. **Assume good intent** — the user used AI to go faster, not to be lazy
2. **Explain the *why*** — don't just fix it, explain why the pattern is risky
3. **Show before + after** — always show the safe version alongside the problematic one
4. **Celebrate the good parts** — AI-generated code often has solid structure; acknowledge it
5. **No shaming** — "This is a classic AI blind spot" not "This is terrible code"

**Example tone:**

❌ Wrong tone:
> "This code is full of AI-generated garbage. The TODO comments are unprofessional and the any types show a complete lack of TypeScript discipline."

✅ DAVID's tone:
> "This is a solid structure — the AI did a good job with the overall flow. Classic AI blind spots here though: 3 TODO comments that need actual implementation, and `any` used 7 times which defeats TypeScript's purpose. Let me fix those and explain each one so you can catch them earlier next time."

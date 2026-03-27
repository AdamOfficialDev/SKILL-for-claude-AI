# DAVID — Git Diff & PR Review Reference

Full pattern library for Scanner I (🔀 GIT). Load this when a git diff, PR, or "changes since [commit]" is provided.

---

## Table of Contents
1. [Diff Intake & Change Map](#diff-intake--change-map)
2. [Security Regression Patterns](#security-regression-patterns)
3. [Logic Regression Patterns](#logic-regression-patterns)
4. [Test Coverage Regression](#test-coverage-regression)
5. [Dependency Change Patterns](#dependency-change-patterns)
6. [CI/CD & Config Change Patterns](#cicd--config-change-patterns)
7. [Cross-File Impact Analysis](#cross-file-impact-analysis)
8. [Commit Message Quality](#commit-message-quality)
9. [PR Description Checklist](#pr-description-checklist)
10. [Finding Severity Table](#finding-severity-table)

---

## Diff Intake & Change Map

When a git diff is received, DAVID builds a structured change map before running any scanner:

```
📋 GIT CHANGE MAP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
File                    │ +Lines │ -Lines │ Type
────────────────────────│────────│────────│──────────────
src/api/auth.ts         │   +24  │    -6  │ MODIFY
src/utils/parser.ts     │   +81  │    -0  │ ADD (new file)
src/tests/auth.spec.ts  │    -45 │    -0  │ DELETE ⚠️
package.json            │    +3  │    -1  │ MODIFY (dep change)
.env.example            │    +0  │    -0  │ not changed ⚠️
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total: 5 files changed, +108 -52 — delta review mode
```

**Scope rule:** DAVID focuses scanners on changed lines + immediate callers/callees only. Unchanged code marked as "baseline — not re-reviewed."

---

## Security Regression Patterns

### 🔴 P0 — Secret introduced in diff

```diff
+ const STRIPE_KEY = "sk_live_abc123xyz789...";
+ process.env.JWT_SECRET = "hardcoded-secret";
+ Authorization: "Bearer eyJhbGci..."   # Real token in code
```

**DAVID action:** Flag immediately as P0. Advise rotating the credential. Check git history — if pushed to remote, the secret is compromised even after removal.

---

### 🔴 P0 — Auth/authorization check removed

```diff
- if (!req.user || req.user.role !== 'admin') {
-   return res.status(403).json({ error: 'Forbidden' });
- }
  const data = await db.getAllUsers();
```

**Pattern:** Any diff that removes `auth`, `authorize`, `permission`, `role`, `guard`, `middleware` lines without adding equivalent protection elsewhere.

---

### 🟠 P1 — Input sanitization removed or bypassed

```diff
- const sanitized = DOMPurify.sanitize(userInput);
- element.innerHTML = sanitized;
+ element.innerHTML = userInput;  // "Faster" — now XSS vuln
```

---

### 🟠 P1 — Error handler silenced

```diff
- } catch (err) {
-   logger.error({ event: 'payment.failed', err });
-   throw err;
- }
+ } catch (err) {
+ }  // Empty catch — error swallowed silently
```

---

### 🟠 P1 — CORS/CSP loosened

```diff
- app.use(cors({ origin: 'https://myapp.com' }));
+ app.use(cors({ origin: '*' }));  // Now open to all origins
```

---

### 🟠 P1 — Rate limiting removed

```diff
- app.use('/api/login', rateLimit({ max: 10, windowMs: 60000 }));
  app.post('/api/login', loginHandler);
```

---

### 🟡 P2 — Validation reduced or removed

```diff
- const schema = z.object({ email: z.string().email(), age: z.number().min(18) });
- const body = schema.parse(req.body);
+ const body = req.body;  // "Skip validation for now"
```

---

## Logic Regression Patterns

### 🔴 P0 — Return/throw removed from error path

```diff
  if (!user) {
-   throw new NotFoundError('User not found');
  }
  // Code continues with user = undefined → crash downstream
```

---

### 🔴 P0 — Default value removed

```diff
- const limit = req.query.limit ?? 100;
+ const limit = req.query.limit;  // undefined if not passed → NaN in query
```

---

### 🟠 P1 — Condition inverted or weakened

```diff
- if (user.isVerified && user.hasActiveSubscription) {
+ if (user.isVerified || user.hasActiveSubscription) {  // OR instead of AND
```

---

### 🟠 P1 — Null/undefined check removed

```diff
- if (data && data.items && data.items.length > 0) {
+ if (data.items.length > 0) {  // Crashes if data or data.items is null
```

---

### 🟡 P2 — Index changed (off-by-one risk)

```diff
- for (let i = 0; i < arr.length; i++) {
+ for (let i = 1; i < arr.length; i++) {  // Skips first element — intentional?
```

DAVID flags: `🤖 DAVID ASSUMPTION: Index start changed to 1. Confidence: 60%. If skipping header row, add comment. If bug, revert to 0.`

---

### 🟡 P2 — Async/await removed

```diff
- const result = await fetchData();
+ const result = fetchData();  // Now a Promise — downstream code breaks
```

---

## Test Coverage Regression

### 🔴 P0 — Test file deleted, implementation modified in same PR

```diff
- src/tests/payment.spec.ts    (deleted)
  src/services/payment.ts      (modified +89 lines)
```

**DAVID output:**
```
🔴 COVERAGE REGRESSION [GIT-COV-001]
Test file deleted while implementation grew by 89 lines.
This PR introduces untested code paths.
Action: Restore test file OR add coverage in another spec file before merging.
```

---

### 🟠 P1 — New function added with no corresponding test

```diff
+ export async function processRefund(orderId: string, amount: number) {
+   // 40 lines of logic
+ }
```

No matching `processRefund` found in any `*.spec.*` or `*.test.*` file in the diff.

---

### 🟠 P1 — Mock/stub changed to bypass real behavior

```diff
- jest.spyOn(paymentService, 'charge').mockImplementation(realChargeStub);
+ jest.spyOn(paymentService, 'charge').mockResolvedValue({ success: true });
  // All edge cases now hidden behind a trivially-passing mock
```

---

### 🟡 P2 — Test assertion weakened

```diff
- expect(result).toEqual({ id: 1, status: 'completed', amount: 99.99 });
+ expect(result).toBeTruthy();  // Now passes for any non-falsy value
```

---

## Dependency Change Patterns

### 🔴 P0 — Lock file not updated after package.json change

```diff
  // package.json:
+ "some-package": "^2.3.0"

  // package-lock.json / yarn.lock: NO CHANGE
```

**Risk:** `npm ci` in CI will fail. Other devs get different versions. Indicates the package was added without `npm install`.

---

### 🟠 P1 — Known risky package added

```diff
+ "eval": "^1.0.0"           # Executes arbitrary code
+ "serialize-javascript": "^3.0.0"  # Check for old XSS vuln version
+ "node-serialize": "*"      # Known RCE vulnerability
```

DAVID cross-references added packages against known vulnerability patterns.

---

### 🟠 P1 — Package version pinning removed

```diff
- "react": "18.2.0"
+ "react": "^18.0.0"  # Now accepts any 18.x — risky minor version jumps
```

---

### 🟡 P2 — Duplicate dependency introduced

```diff
+ "axios": "^1.4.0"
```

But `node-fetch` already exists in package.json. Two HTTP clients = unnecessary bundle bloat.

---

### 🟡 P2 — devDependency moved to dependency (or vice versa)

```diff
- "typescript": "^5.0.0"   // was in devDependencies
+ "typescript": "^5.0.0"   // now in dependencies — ships to prod unnecessarily
```

---

## CI/CD & Config Change Patterns

### 🔴 P0 — Secrets exposed in CI config

```diff
+ env:
+   API_KEY: "sk-prod-real-key-here"  # Hardcoded in workflow file
```

---

### 🔴 P0 — `noindex` added to production pages

```diff
+ <meta name="robots" content="noindex, nofollow">
```

In a file that is NOT a staging/preview config. Will deindex the page from search engines.

---

### 🟠 P1 — GitHub Actions: unpinned action version

```diff
- uses: actions/checkout@v3
+ uses: actions/checkout@main  # Unpinned — supply chain risk
```

---

### 🟠 P1 — Dockerfile: secrets in ENV layer

```diff
+ ENV AWS_SECRET_KEY="real-key-here"
```

Baked into image layer permanently. Even if removed in later layer, exists in image history.

---

### 🟠 P1 — `.env.example` not updated when `.env` changes

```diff
  // src/config.ts:
+ const NEW_SERVICE_URL = process.env.NEW_SERVICE_URL;  // New var used

  // .env.example: NO CHANGE
```

New developer won't know `NEW_SERVICE_URL` is required → silent runtime failure.

---

### 🟡 P2 — Production config changed without staging flag

```diff
  // config/production.ts:
+ cacheEnabled: false,  // Disabled caching in prod without feature flag
```

---

## Cross-File Impact Analysis

When a function signature changes in a diff, DAVID maps all call sites:

```
🔗 CROSS-FILE IMPACT — Signature Changed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Changed  : getUser(id: string) → getUser(id: string, options?: GetUserOptions)
In file  : src/services/user.ts (line 24)

Call sites NOT updated in this diff:
  ❌ src/api/routes/user.ts:47    → needs options param review
  ❌ src/workers/email.worker.ts:12 → needs options param review
  ❌ src/tests/user.spec.ts:88   → test signature now wrong

Call sites already updated in this diff:
  ✅ src/api/routes/admin.ts:103  → already uses new signature
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Action: Update 3 call sites before this PR is safe to merge.
```

---

## Commit Message Quality

DAVID evaluates commit messages for Conventional Commits compliance when commits are included in the diff:

| Pattern | Verdict | Why |
|---------|---------|-----|
| `fix(auth): prevent token reuse after logout` | ✅ Good | Type, scope, description |
| `feat(payments): add idempotency key support` | ✅ Good | Clear feature description |
| `WIP` | ❌ Bad | Meaningless in git history |
| `fix stuff` | ❌ Bad | No scope, no description |
| `asdfghjkl` | ❌ Bad | Clearly a temp commit |
| `Update` | ❌ Bad | What was updated? |
| `fix: fixed the bug that was causing issues` | 🟡 Weak | Circular description |

**Conventional Commits format:**
```
type(scope): description

type = feat | fix | docs | style | refactor | test | chore | perf | ci
scope = the module/component affected (optional but recommended)
description = imperative mood, lowercase, no period
```

---

## PR Description Checklist

When reviewing a PR, DAVID checks for these elements in the PR body:

```
PR QUALITY CHECKLIST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[?] What changed — summary of changes
[?] Why — problem being solved / motivation
[?] How to test — steps for reviewer to verify
[?] Screenshots — for UI changes
[?] Breaking changes — listed explicitly or "None"
[?] Related issues/tickets — linked
[?] Checklist — tests added, docs updated
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Score: X/7 elements present
```

---

## Finding Severity Table

| Condition | Severity | DAVID Action |
|-----------|----------|-------------|
| Secret/credential in diff | 🔴 P0 | Immediate flag, advise rotate |
| Auth check removed | 🔴 P0 | Block merge recommendation |
| Test file deleted + impl modified | 🔴 P0 | Coverage regression warning |
| Return/throw removed from error path | 🔴 P0 | Fix before merge |
| Input sanitization removed | 🟠 P1 | Fix before merge |
| Lock file not updated | 🟠 P1 | Flag inconsistency |
| Condition logic weakened | 🟠 P1 | Confirm intent |
| Unpinned CI action | 🟠 P1 | Pin to SHA |
| New function with no test | 🟠 P1 | Generate test |
| Unused import added | 🟡 P2 | Clean up |
| Commit message bad | 🟡 P2 | Suggest rewrite |
| PR description incomplete | 🟢 P3 | Suggest improvement |

---

## Quick Reference: What DAVID Always Outputs for Git Diff Mode

```
📋 Git scope: N files changed, +X -Y lines — delta review mode

[All scanner findings on changed lines only]

🔗 Cross-file impact: N call sites affected → [list]

📝 Commit quality: [score] / Conventional Commits: [YES/NO]

📋 PR completeness: [X/7 elements] — [missing list]
```

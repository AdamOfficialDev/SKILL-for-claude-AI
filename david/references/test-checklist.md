# DAVID — Test Checklist by Framework

Use this reference when performing Phase 5: Verification & Testing. Select the relevant framework section and work through every applicable item before delivering fixed code.

---

## Table of Contents
1. [Universal Checks (All Languages)](#universal-checks)
2. [React / Next.js](#react--nextjs)
3. [Python / FastAPI / Django / Flask](#python-backend)
4. [Node.js / Express](#nodejs--express)
5. [Java / Spring Boot](#java--spring-boot)
6. [Mobile (Flutter / React Native)](#mobile)
7. [SQL / Database Changes](#sql--database-changes)
8. [CLI / Shell Scripts](#cli--shell-scripts)

---

## Universal Checks

These apply to EVERY fix regardless of language or framework.

### Integrity Checks
- [ ] **Line count sanity** — Output file has ≥ same number of lines as input (unless deletions were authorized)
- [ ] **Function inventory** — Every function/method that existed before still exists after
- [ ] **Export inventory** — Every exported symbol still exported with same name and signature
- [ ] **Import inventory** — No imports removed unless the symbol they provided is also unused after fix
- [ ] **Comment preservation** — All original comments still present
- [ ] **Indentation style** — Tabs vs spaces matches original; no reformatting of untouched code

### Logic Verification
- [ ] **Happy path trace** — Mentally execute with a typical valid input; confirm expected output
- [ ] **Null / None / undefined input** — Function handles gracefully (returns default or throws intentionally)
- [ ] **Empty input** — Empty string, empty array, zero value, empty object
- [ ] **Boundary values** — Max, min, exactly-at-limit values
- [ ] **Error path trace** — When something goes wrong, error is caught/returned correctly

### Regression
- [ ] **Unmodified functions** — At least spot-check 2-3 nearby unmodified functions still behave correctly
- [ ] **Callers** — Every known caller of the modified function still gets a compatible return value
- [ ] **Return type** — Return type/shape unchanged (unless fix required changing it; document this)
- [ ] **Side effects** — No new global state mutations introduced

---

## React / Next.js

### Component Checks
- [ ] Props interface unchanged (no removed or renamed props without authorization)
- [ ] Default prop values preserved
- [ ] All hook calls valid (no conditional hooks, hooks at top level only)
- [ ] `useEffect` dependency array correct — no missing or extra dependencies
- [ ] No state mutation (no `state.push()`, always spread)
- [ ] Keys present in all list renders (`.map(() => <el key={...}>)`)
- [ ] Event handlers properly bound / use arrow functions
- [ ] Loading and error states handled

### Next.js Specific
- [ ] `getServerSideProps` / `getStaticProps` return correct shape
- [ ] API routes return proper HTTP status codes
- [ ] `use client` / `use server` directives present where needed (App Router)
- [ ] No server-only imports in client components

### Performance
- [ ] No inline object/array literals as props to memoized components
- [ ] `useCallback` / `useMemo` not removed from performance-critical paths
- [ ] `React.memo` still applied where it was before

---

## Python Backend

### General Python
- [ ] All function signatures unchanged (positional args, keyword args, defaults)
- [ ] Type hints preserved
- [ ] Exception handling covers same exception types as before
- [ ] Generators / iterators still work for large data (not loaded to memory)
- [ ] Context managers (`with` statements) still close resources

### FastAPI
- [ ] Route decorators unchanged (`@app.get`, `@app.post`, path unchanged)
- [ ] Pydantic models have all original fields
- [ ] Response models match expected schema
- [ ] Dependencies (`Depends(...)`) preserved
- [ ] Status codes unchanged unless intentional

### Django
- [ ] Model fields unchanged (no removed fields, no changed `null=`, `blank=`)
- [ ] URL patterns match expected patterns
- [ ] View signatures compatible with URL dispatcher
- [ ] Template context variables preserved
- [ ] Migration required? (flag to user if model changed)

### Flask
- [ ] Route paths unchanged
- [ ] Blueprint registrations intact
- [ ] `before_request` / `after_request` hooks preserved
- [ ] Response objects return same content type

---

## Node.js / Express

### Express
- [ ] Route paths, HTTP methods unchanged
- [ ] Middleware order preserved
- [ ] `next()` called in middleware where it was before
- [ ] Error-handling middleware has 4 params `(err, req, res, next)`
- [ ] Response methods called once (`res.json()` / `res.send()` not called twice)
- [ ] Async route handlers wrapped in try/catch or `asyncHandler`

### General Node
- [ ] Event emitter listeners still attached
- [ ] Stream `pipe()` chains intact
- [ ] `process.env` accesses still work with same variable names
- [ ] No synchronous blocking calls introduced (`readFileSync` in hot path)

---

## Java / Spring Boot

### Spring
- [ ] `@Bean` / `@Component` / `@Service` annotations intact
- [ ] Constructor injection dependencies unchanged
- [ ] `@RequestMapping` / `@GetMapping` paths unchanged
- [ ] `@Transactional` boundaries preserved
- [ ] Exception handlers (`@ExceptionHandler`) still present

### General Java
- [ ] All `implements` / `extends` contracts satisfied
- [ ] `@Override` methods have same signature
- [ ] `final` fields initialized in constructor
- [ ] Resources closed in `finally` or try-with-resources
- [ ] Thread-safety annotations preserved (`@GuardedBy`, etc.)

---

## Mobile

### Flutter
- [ ] Widget tree structure preserved
- [ ] `State` class `dispose()` still cancels subscriptions
- [ ] `initState()` still initializes all required controllers
- [ ] `BuildContext` not used after `await` without `mounted` check
- [ ] Named routes unchanged

### React Native
- [ ] StyleSheet definitions preserved
- [ ] Platform-specific code (`Platform.OS`) still handles both platforms
- [ ] Native module calls unchanged
- [ ] `useEffect` cleanup functions return properly

---

## SQL / Database Changes

- [ ] No `DROP TABLE` / `DROP COLUMN` without explicit user authorization
- [ ] No destructive `UPDATE` without `WHERE` clause
- [ ] No data type change to column without confirming impact
- [ ] Indexes not removed
- [ ] Foreign key constraints preserved
- [ ] Transaction wrapped around multi-statement changes
- [ ] Rollback path exists if migration fails

---

## CLI / Shell Scripts

- [ ] Script still handles arguments in same order / with same flags
- [ ] Exit codes correct (0 for success, non-zero for failure)
- [ ] `set -e` / `set -u` guards preserved if present
- [ ] Pipe chains (`|`) intact and in correct order
- [ ] File paths still correct (no hardcoded paths changed)
- [ ] Permissions not changed (`chmod` calls still present)
- [ ] `trap` handlers for cleanup still registered

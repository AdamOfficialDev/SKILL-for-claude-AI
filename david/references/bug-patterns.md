# DAVID — Bug Pattern Library

A quick-reference compendium of common bugs by language/platform. DAVID consults this when a bug matches a known pattern to confirm the diagnosis faster.

---

## Table of Contents
1. [Python](#python)
2. [JavaScript / TypeScript](#javascript--typescript)
3. [Java / Kotlin](#java--kotlin)
4. [C / C++](#c--c)
5. [Go](#go)
6. [SQL](#sql)
7. [Async / Concurrency (Cross-Language)](#async--concurrency)
8. [API & HTTP](#api--http)

---

## Python

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| Mutable default argument | List/dict grows across calls | `def f(x=[])` shares the same object | Use `def f(x=None): if x is None: x = []` |
| Late binding closure | Loop variable captured wrong value | Lambda captures name, not value | `lambda i=i: ...` or use `functools.partial` |
| `is` vs `==` | False negative on equality check | `is` checks identity, not value | Use `==` for value comparison |
| Integer division | Unexpected truncation | Python 2 `/` vs Python 3 `/` | Use `//` for int division explicitly |
| Missing `copy()` | Mutation of "separate" list | Assignment copies reference | Use `list.copy()` or `deepcopy()` |
| UnboundLocalError | Variable used before assignment | Assigned later in same scope | Move assignment before use or declare `global`/`nonlocal` |
| `except Exception` too broad | Silent error swallowing | Catches everything including KeyboardInterrupt | Catch specific exceptions |
| Circular import | ImportError at startup | Module A imports B which imports A | Restructure modules or use lazy import |

---

## JavaScript / TypeScript

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| `==` vs `===` | Unexpected equality / inequality | Type coercion | Always use `===` |
| Callback hell / missing await | Promise never resolves or rejects silently | Missing `await` or `.catch()` | Add `await`; wrap in try/catch |
| `var` hoisting | Variable undefined at unexpected point | `var` is function-scoped | Use `let`/`const` |
| Stale closure | Event handler reads wrong value | Closure captures old reference | Use `useCallback`/`useRef` in React; restructure closure |
| Mutation of props | State change doesn't trigger re-render | Direct array/object mutation | Use spread: `[...arr, newItem]` |
| `this` binding lost | `TypeError: this.X is not a function` | Arrow function vs regular function context | Use arrow function or `.bind(this)` |
| `NaN` propagation | Numeric operations return `NaN` | Parsing failed silently | Check `isNaN()` after `parseInt`/`parseFloat` |
| Optional chaining missing | `TypeError: Cannot read property of undefined` | Deep property access without guard | Use `?.` optional chaining |
| Off-by-one in `.slice()` | Wrong subset of array | End index is exclusive | Remember: `arr.slice(0, 3)` gives indices 0,1,2 |
| Unhandled Promise rejection | Silent crash in async function | No `.catch()` or try/catch | Always handle rejections |
| TypeScript `any` escape hatch | Type errors at runtime | `as any` cast hides real type issues | Use proper types; avoid `any` |

---

## Java / Kotlin

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| NullPointerException | Crash on `.method()` call | Object reference is null | Null check; use `Optional`; Kotlin `?.` |
| `==` on strings | Always false comparison | Compares references, not values | Use `.equals()` |
| Integer overflow | Negative result from large sum | `int` max 2^31-1 | Use `long` or `BigInteger` |
| ConcurrentModificationException | Exception during iteration | Modifying collection while iterating | Use `Iterator.remove()` or copy first |
| Resource leak | Connection/file never closed | Missing `try-with-resources` | Use `try (Resource r = ...)` |
| `static` mutable state | Thread-safety bugs | Shared mutable static field | Use thread-local or synchronize |
| Autoboxing NPE | NPE on unboxing | `Integer` to `int` when null | Check for null before unboxing |

---

## C / C++

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| Buffer overflow | Crash, corruption, or exploit | Writing past array bounds | Bounds check; use `std::vector` |
| Use-after-free | Undefined behavior / crash | Accessing freed memory | Set pointer to `nullptr` after `free` |
| Dangling pointer | UB / crash after function returns | Returning pointer to local variable | Return by value or use heap allocation |
| Integer sign mismatch | Comparison always true/false | Comparing `signed` to `unsigned` | Cast explicitly; use consistent types |
| Missing `\0` terminator | String functions read garbage | Buffer filled without null terminator | Ensure last byte is `\0` |
| Memory leak | Growing RSS / OOM | `new` without matching `delete` | Use RAII; `std::unique_ptr` |
| Uninitialized variable | Unpredictable behavior | Variable declared but not set | Always initialize at declaration |

---

## Go

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| Nil pointer dereference | Panic at runtime | Calling method on nil interface/pointer | Nil check before use |
| Goroutine leak | Memory grows over time | Goroutine blocked forever | Use context cancellation; close channels |
| Closing closed channel | Panic | Closing already-closed channel | Use `sync.Once` or ownership pattern |
| Race on map | `fatal error: concurrent map read and map write` | Map accessed from multiple goroutines | Use `sync.RWMutex` or `sync.Map` |
| Error ignored | Bug silent until much later | `_, err := f()` without checking `err` | Always check errors |
| Loop variable capture | All goroutines use last value | Closure captures loop variable by reference | Pass `i` as argument: `go func(i int)` |

---

## SQL

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| N+1 query | Severe performance degradation | Query inside a loop | Use JOIN or batch fetch |
| Missing index | Slow query on large table | Full table scan | `CREATE INDEX` on filtered columns |
| SQL Injection | Data breach / unauthorized access | User input in raw query string | Use parameterized queries / ORM |
| NULL comparison | Rows silently excluded | `col = NULL` always false in SQL | Use `IS NULL` / `IS NOT NULL` |
| Implicit type cast | Wrong results or slow query | Comparing VARCHAR to INT | Explicit CAST; match column types |
| Missing transaction | Partial write on failure | Multi-step write without `BEGIN/COMMIT` | Wrap in transaction |
| SELECT * in production | Extra bandwidth, fragile schema | Fetching all columns unnecessarily | Select only needed columns |

---

## Async / Concurrency

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| Race condition | Inconsistent state | Two operations interleave unsafely | Use locks, atomics, or message-passing |
| Deadlock | App freezes forever | Two threads each wait for the other's lock | Consistent lock ordering; timeout |
| Starvation | One thread never runs | High-priority threads monopolize CPU | Fair scheduling; priority inversion fix |
| Missing synchronization | Data corruption | Shared state accessed without lock | Add mutex around shared state |
| Fire-and-forget | Errors never surfaced | Async task launched without waiting | Await or attach error handler |

---

## API & HTTP

| Pattern | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| No error handling on fetch | Silent failure | HTTP 4xx/5xx not checked | Check `response.ok`; throw on error |
| CORS error | Browser blocks request | Missing `Access-Control-Allow-Origin` | Configure CORS on server |
| Missing auth header | 401 Unauthorized | Token not attached to request | Add `Authorization: Bearer <token>` |
| No rate limiting | Client hammers API | No backoff or throttle | Implement exponential backoff |
| Timeout not set | Hang forever | No request timeout configured | Set `timeout` on HTTP client |
| Mutating on GET | State changes from GET request | GET endpoint has side effects | Move mutation to POST/PUT/PATCH |

# DAVID — Performance Profiling Reference

Full pattern library for Scanner C. Load when analyzing performance issues or bottlenecks.

---

## Table of Contents
1. [Complexity Analysis Guide](#complexity-analysis)
2. [N+1 Query Patterns](#n1-query-patterns)
3. [Memory Leak Patterns](#memory-leak-patterns)
4. [Async & Blocking Patterns](#async--blocking-patterns)
5. [Caching Opportunities](#caching-opportunities)
6. [Language-Specific Optimizations](#language-specific-optimizations)
7. [Database Query Optimization](#database-query-optimization)

---

## Complexity Analysis

### Big-O Quick Reference

| Complexity | Name | Example | Acceptable For |
|------------|------|---------|----------------|
| O(1) | Constant | Hash lookup, array index | Always ✅ |
| O(log n) | Logarithmic | Binary search, balanced BST | Always ✅ |
| O(n) | Linear | Single loop, linear scan | n < 10M ✅ |
| O(n log n) | Linearithmic | Sorting (merge/quick sort) | n < 1M ✅ |
| O(n²) | Quadratic | Nested loops | n < 1000 ⚠️ |
| O(n³) | Cubic | Triple nested loops | n < 100 ❌ |
| O(2ⁿ) | Exponential | Naive recursive fibonacci | n < 30 ❌ |
| O(n!) | Factorial | Naive TSP, permutations | n < 12 ❌ |

### How to Calculate

**Step 1:** Identify all loops — each nesting level multiplies complexity.
**Step 2:** Identify recursive calls — look for T(n) = T(n-1) + O(?) recurrences.
**Step 3:** Identify the dominant term — drop constants and lower-order terms.

```python
# O(n²) — nested loop over same data
for i in range(len(arr)):        # O(n)
    for j in range(len(arr)):    # O(n) × O(n) = O(n²)
        if arr[i] == arr[j]: ...

# O(n) — same task with set
seen = set()                     # O(1)
for item in arr:                 # O(n)
    if item in seen: ...         # O(1) hash lookup
    seen.add(item)
```

### Hot Path Detection

A "hot path" is code that executes frequently:
- Inside request handlers (called per HTTP request)
- Inside event loops / message consumers
- Inside render functions (React, game loops)
- Inside data processing pipelines

Flag O(n²) or worse **only** in hot paths. O(n²) in a one-time startup script is fine.

---

## N+1 Query Patterns

The most common and most damaging performance bug in web applications.

### Classic N+1

```javascript
// ❌ N+1 — 1 query for orders + N queries for users
const orders = await Order.findAll();           // Query 1
for (const order of orders) {
    order.user = await User.findById(order.userId);  // Query 2...N+1
}

// ✅ Single query with JOIN / eager loading
const orders = await Order.findAll({
    include: [{ model: User }]  // ORM JOIN
});

// ✅ Or batch fetch
const orders = await Order.findAll();
const userIds = orders.map(o => o.userId);
const users = await User.findAll({ where: { id: userIds } });  // 1 query
const userMap = Object.fromEntries(users.map(u => [u.id, u]));
orders.forEach(o => { o.user = userMap[o.userId]; });
```

### Hidden N+1 (async parallel)

```javascript
// ❌ Still N+1, just concurrent — still N round trips
const results = await Promise.all(
    ids.map(id => db.query('SELECT * FROM items WHERE id = ?', id))
);

// ✅ Batch query
const results = await db.query(
    'SELECT * FROM items WHERE id IN (?)', [ids]
);
```

### ORM Lazy Loading N+1

```python
# Django — ❌ N+1
posts = Post.objects.all()
for post in posts:
    print(post.author.name)  # Hits DB once per post

# Django — ✅ select_related (JOIN)
posts = Post.objects.select_related('author').all()

# Django — ✅ prefetch_related (separate optimized query)
posts = Post.objects.prefetch_related('tags').all()
```

---

## Memory Leak Patterns

### JavaScript / Node.js

```javascript
// ❌ Event listener never removed
element.addEventListener('click', handler);
// Fix: element.removeEventListener('click', handler) in cleanup

// ❌ Timer never cleared
setInterval(() => doThing(), 1000);
// Fix: const timer = setInterval(...); clearInterval(timer) in cleanup

// ❌ Unbounded cache
const cache = {};
function getValue(key) {
    if (!cache[key]) cache[key] = expensiveCompute(key);
    return cache[key];
}
// Fix: Use LRU cache with max size (lru-cache library)

// ❌ Closure holding large object
function createHandler(largeData) {
    return function() {
        return largeData.id;  // Holds entire largeData in memory
    };
}
// Fix: Extract only needed data
function createHandler(largeData) {
    const id = largeData.id;  // Only capture what's needed
    return function() { return id; };
}
```

### Python

```python
# ❌ Growing list in module scope
_cache = []
def process(item):
    _cache.append(item)  # Never cleared

# ❌ Circular references (prevents GC)
class Node:
    def __init__(self):
        self.parent = None
        self.children = []
# Fix: Use weakref.ref() for back-references

# ❌ Generator not consumed
def get_items():
    for i in range(1000000):
        yield i
items = get_items()  # Fine — lazy
list(items)          # ❌ Loads all into memory
# Fix: Process items one at a time; don't collect into list
```

### Java

```java
// ❌ Static collection grows forever
public class Cache {
    private static final List<Object> items = new ArrayList<>();
    public static void add(Object item) { items.add(item); }
    // No eviction, no size limit
}

// ❌ InputStream never closed
InputStream is = new FileInputStream("file.txt");
// No try-with-resources
// Fix:
try (InputStream is = new FileInputStream("file.txt")) {
    // use is
}
```

---

## Async & Blocking Patterns

### Blocking in Async Context

```javascript
// ❌ Synchronous file read blocks Node.js event loop
app.get('/data', (req, res) => {
    const data = fs.readFileSync('large-file.json');  // BLOCKS
    res.json(JSON.parse(data));
});

// ✅ Async version
app.get('/data', async (req, res) => {
    const data = await fs.promises.readFile('large-file.json');
    res.json(JSON.parse(data));
});
```

```python
# ❌ Blocking DB call in async FastAPI handler
@app.get("/users")
async def get_users():
    users = db.query("SELECT * FROM users")  # Synchronous! Blocks event loop
    return users

# ✅ Use async DB driver (asyncpg, motor, etc.)
@app.get("/users")
async def get_users():
    users = await async_db.fetch("SELECT * FROM users")
    return users
```

### Missing Parallelism

```javascript
// ❌ Sequential when parallel is safe
const user = await getUser(id);
const orders = await getOrders(id);
const profile = await getProfile(id);
// Time: T(user) + T(orders) + T(profile)

// ✅ Parallel — independent calls
const [user, orders, profile] = await Promise.all([
    getUser(id),
    getOrders(id),
    getProfile(id)
]);
// Time: max(T(user), T(orders), T(profile))
```

---

## Caching Opportunities

### When to Cache

| Scenario | Strategy |
|----------|----------|
| Same DB query called multiple times per request | In-memory (request-scoped) |
| Expensive computation with same inputs | Memoization |
| Frequently read, rarely written data | Redis / Memcached with TTL |
| Static file contents | HTTP cache headers + CDN |
| API responses from third-party | Redis with TTL |
| User session data | Redis |

### Memoization Pattern

```python
from functools import lru_cache

# ✅ Cache up to 128 results
@lru_cache(maxsize=128)
def expensive_compute(n: int) -> int:
    # ...

# For async:
import asyncio
from cachetools import TTLCache
cache = TTLCache(maxsize=1000, ttl=300)

async def get_data(key):
    if key in cache:
        return cache[key]
    result = await fetch_from_db(key)
    cache[key] = result
    return result
```

### Cache Invalidation Red Flags

- Cache set but never invalidated after write operations
- Cache with no TTL (stale forever)
- Cache key doesn't include all relevant inputs (returns wrong cached result)
- Cache not thread-safe under concurrent writes

---

## Language-Specific Optimizations

### Python

| Anti-Pattern | Optimized |
|-------------|-----------|
| `for i in range(len(lst))` | `for item in lst` |
| String concat in loop: `s += x` | `''.join(parts)` |
| `[x for x in lst if ...]` then `len()` | `sum(1 for x in lst if ...)` |
| `list(filter(fn, lst))` | `[x for x in lst if fn(x)]` |
| `x in list` for membership | `x in set` (O(1) vs O(n)) |
| `dict.keys()` iteration | `for key in dict` |
| Multiple DB queries in loop | `select_related` / batch |
| `pandas` row-by-row `iterrows()` | Vectorized operations |

### JavaScript

| Anti-Pattern | Optimized |
|-------------|-----------|
| `arr.forEach` + push into new array | `arr.map()` |
| `arr.filter().map()` (two passes) | `arr.reduce()` (one pass) |
| `.indexOf()` for existence check | `.includes()` |
| Object spread in tight loop | Pre-allocated object |
| Repeated DOM query: `document.getElementById` in loop | Cache reference outside loop |
| `JSON.parse(JSON.stringify(obj))` | `structuredClone(obj)` |
| Sorting on every render | Memoize sorted result |

### Java / Kotlin

| Anti-Pattern | Optimized |
|-------------|-----------|
| String concat in loop: `s += x` | `StringBuilder` |
| `ArrayList` when size known | `new ArrayList<>(expectedSize)` |
| Synchronized method for reads | `ReentrantReadWriteLock` |
| Blocking `Future.get()` | `CompletableFuture.thenApply()` |
| `Stream.collect(Collectors.toList())` re-sorted | Sort before collecting |

---

## Database Query Optimization

### Index Usage

```sql
-- ❌ Full table scan — no index on email
SELECT * FROM users WHERE email = 'user@example.com';

-- ✅ After: CREATE INDEX idx_users_email ON users(email);

-- ❌ Function on indexed column defeats index
SELECT * FROM users WHERE LOWER(email) = 'user@example.com';

-- ✅ Store normalized and index the stored value
SELECT * FROM users WHERE email = LOWER('user@example.com');
```

### Query Shape Red Flags

```sql
-- ❌ SELECT * — fetches all columns, including large blobs
SELECT * FROM documents WHERE user_id = 1;

-- ✅ Select only needed columns
SELECT id, title, created_at FROM documents WHERE user_id = 1;

-- ❌ No LIMIT — returns entire table
SELECT * FROM events WHERE type = 'click';

-- ✅ Always paginate
SELECT * FROM events WHERE type = 'click' LIMIT 50 OFFSET 0;

-- ❌ Correlated subquery (runs N times)
SELECT * FROM orders o
WHERE (SELECT COUNT(*) FROM items WHERE order_id = o.id) > 5;

-- ✅ JOIN with GROUP BY (runs once)
SELECT o.* FROM orders o
JOIN (SELECT order_id, COUNT(*) cnt FROM items GROUP BY order_id) i
ON o.id = i.order_id WHERE i.cnt > 5;
```

### Transaction Scope

```python
# ❌ Transaction too wide — holds locks too long
with db.transaction():
    orders = db.query("SELECT * FROM orders")  # Lock
    time.sleep(2)  # External API call inside transaction!
    db.execute("UPDATE orders SET status = 'processed'")

# ✅ Minimal transaction scope
orders = db.query("SELECT * FROM orders")
result = external_api_call(orders)  # Outside transaction
with db.transaction():
    db.execute("UPDATE orders SET status = 'processed'")
```

---

## Database Query Plan Hints (Scanner C Deepened — v6.0)

When DAVID identifies a likely slow query, it adds a specific `EXPLAIN ANALYZE` guidance block
alongside the finding so the user can verify before and after optimization.

### EXPLAIN ANALYZE Guidance Block

```sql
-- 💡 DAVID QUERY HINT: Run this to verify:
-- EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';
--
-- What to look for:
--   "Seq Scan" on a large table          → needs an index
--   "Nested Loop" with high rows estimate → consider join reorder or composite index
--   "Hash Join" with large batch size     → check work_mem setting
--   "Sort" without "Index Scan"           → add index on ORDER BY column
--
-- Expected result after fix:
--   "Index Scan using idx_orders_user_status on orders"
--   OR "Bitmap Index Scan" for multi-condition queries
```

### How to Read EXPLAIN Output

| Node Type | Meaning | Action |
|-----------|---------|--------|
| `Seq Scan` on table > 10K rows | Full table scan — no index used | Add index on WHERE columns |
| `Nested Loop` with rows > 1000 | O(n²) join | Add FK index; consider join order |
| `Hash Join` | In-memory hash build — OK for medium sets | Monitor `work_mem` |
| `Merge Join` | Sorted merge — good for large indexed sets | Usually fine |
| `Index Scan` | Single index lookup | Ideal |
| `Bitmap Index Scan` | Multi-condition index combo | Good |
| `Sort` without index | filesort — expensive at scale | Add index on ORDER BY |

### Attaching Query Hints to Findings

Every PERF finding that involves a DB query must include the hint block:

```
📊 PERFORMANCE FINDING [PERF-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Issue      : N+1 query — fetchOrdersByUser called inside a loop
File:Line  : src/services/orders.ts:47
Root Cause : Missing eager load — N SELECT calls for N users
Fix        : [batched query code]
Query Hint : EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = $1;
             Look for: Seq Scan → add index; expect: Index Scan after fix
```

### PostgreSQL-Specific Performance Patterns

```sql
-- Covering index (avoids heap fetch for read-heavy queries)
CREATE INDEX idx_orders_user_status_created
  ON orders(user_id, status)
  INCLUDE (created_at, total_amount);
-- "INCLUDE" columns are stored but not searchable — avoids heap fetch

-- Partial index (smaller, faster for filtered queries)
CREATE INDEX idx_active_orders ON orders(user_id)
  WHERE status = 'active';
-- Only indexes active orders — much smaller index

-- CONCURRENTLY (avoid table lock on large tables)
CREATE INDEX CONCURRENTLY idx_orders_created ON orders(created_at);
-- Takes longer but doesn't block reads/writes
```

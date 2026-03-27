# DAVID — Resilience & Fault Tolerance Reference (Scanner AF)

Full patterns for building systems that survive failures gracefully. Load for production backend code making external calls.

---

## Timeout Configuration Guide

### Timeout Values by Operation Type

| Operation | Recommended Timeout | Never Exceed |
|-----------|-------------------|-------------|
| Internal service call | 500ms–1s | 3s |
| External API (non-critical) | 3s | 10s |
| Payment gateway | 10s | 30s |
| Database query (simple) | 1s | 5s |
| Database query (complex report) | 30s | 120s |
| File upload (user-facing) | 30s | 60s |
| Background job | 5min | 30min |

### Implementation Patterns

```javascript
// ✅ Node.js fetch with timeout
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
    const response = await fetch(url, { signal: controller.signal });
    clearTimeout(timeoutId);
    return response.json();
} catch (error) {
    if (error.name === 'AbortError') {
        throw new TimeoutError(`Request to ${url} timed out after 5s`);
    }
    throw error;
}

// ✅ Axios with timeout
const client = axios.create({ timeout: 5000 });

// ✅ PostgreSQL query timeout (session-level)
await pool.query('SET LOCAL statement_timeout = 3000');  // 3s for this query
await pool.query('SELECT * FROM large_table WHERE...');

// ✅ Redis timeout
const redis = new Redis({ commandTimeout: 1000 });  // 1s per command

// ✅ Python requests
import requests
response = requests.get(url, timeout=(3.05, 10))
# (connect_timeout, read_timeout) — both should be set
```

---

## Exponential Backoff With Jitter

```typescript
// ✅ Full implementation with jitter, max delay, and abort conditions
interface RetryOptions {
    maxAttempts: number;
    baseDelayMs: number;
    maxDelayMs: number;
    shouldRetry?: (error: Error, attempt: number) => boolean;
}

async function withRetry<T>(
    fn: () => Promise<T>,
    options: RetryOptions = { maxAttempts: 3, baseDelayMs: 1000, maxDelayMs: 30000 }
): Promise<T> {
    let lastError: Error;

    for (let attempt = 0; attempt < options.maxAttempts; attempt++) {
        try {
            return await fn();
        } catch (error) {
            lastError = error as Error;

            // Don't retry client errors (4xx) — they won't succeed on retry
            if (error.response?.status >= 400 && error.response?.status < 500) {
                throw error;
            }

            // Don't retry if custom shouldRetry says no
            if (options.shouldRetry && !options.shouldRetry(lastError, attempt)) {
                throw error;
            }

            if (attempt < options.maxAttempts - 1) {
                // Exponential backoff: 1s, 2s, 4s, 8s...
                const exponential = options.baseDelayMs * Math.pow(2, attempt);
                // Jitter: add random 0–1s to prevent thundering herd
                const jitter = Math.random() * 1000;
                const delay = Math.min(exponential + jitter, options.maxDelayMs);

                console.warn(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms...`);
                await new Promise(r => setTimeout(r, delay));
            }
        }
    }

    throw lastError!;
}

// Usage
const user = await withRetry(
    () => userService.get(userId),
    { maxAttempts: 3, baseDelayMs: 500, maxDelayMs: 10000 }
);
```

```python
# ✅ Python with tenacity library
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import requests

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=10),
    retry=retry_if_exception_type(requests.Timeout),
    # Don't retry client errors:
    reraise=True,
)
def call_payment_api(payload: dict) -> dict:
    response = requests.post(
        'https://api.payment.com/charge',
        json=payload,
        timeout=5,
    )
    response.raise_for_status()
    return response.json()
```

---

## Circuit Breaker States

```
CLOSED (normal)
    │ failure_count < threshold
    │ Pass requests through normally
    │
    │ failure_count >= threshold
    ▼
OPEN (failing)
    │ Immediately reject all requests with fallback
    │ Wait recovery_time
    │
    │ After recovery_time
    ▼
HALF-OPEN (testing)
    │ Allow ONE request through
    │
    ├── SUCCESS → back to CLOSED
    └── FAILURE → back to OPEN (reset timer)
```

```python
# ✅ Production-ready circuit breaker
import time
from threading import Lock
from enum import Enum

class CircuitState(Enum):
    CLOSED = 'closed'
    OPEN = 'open'
    HALF_OPEN = 'half_open'

class CircuitBreaker:
    def __init__(self, name: str, failure_threshold=5, recovery_timeout=30):
        self.name = name
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
        self._lock = Lock()

    def call(self, func, *args, **kwargs):
        with self._lock:
            if self.state == CircuitState.OPEN:
                if time.time() - self.last_failure_time >= self.recovery_timeout:
                    self.state = CircuitState.HALF_OPEN
                else:
                    raise CircuitOpenError(f'Circuit {self.name} is OPEN')

        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        with self._lock:
            self.failure_count = 0
            self.state = CircuitState.CLOSED

    def _on_failure(self):
        with self._lock:
            self.failure_count += 1
            self.last_failure_time = time.time()
            if self.failure_count >= self.failure_threshold:
                self.state = CircuitState.OPEN

# Usage
recommendation_breaker = CircuitBreaker('recommendations', failure_threshold=5, recovery_timeout=30)

def get_recommendations(user_id: int) -> list:
    try:
        return recommendation_breaker.call(recommendation_api.get, user_id)
    except (CircuitOpenError, TimeoutError):
        return []  # Graceful degradation
```

---

## Graceful Degradation Patterns

### Priority Tiers

```
TIER 1 — CRITICAL (never degrade)
  Authentication, payment processing, core CRUD
  → These must work or the user session is invalid

TIER 2 — IMPORTANT (degrade with warning)
  Search, filtering, real-time notifications
  → Show cached/stale data with timestamp

TIER 3 — ENHANCEMENT (degrade silently)
  Recommendations, analytics, social features
  → Show empty state, log error, don't alert user

TIER 4 — OPTIONAL (fail silently)
  A/B test assignment, feature flags, personalization
  → Use default, never show error
```

```typescript
// ✅ Structured graceful degradation
async function buildProductPage(productId: string) {
    // TIER 1: Critical — fail hard if this fails
    const product = await db.getProduct(productId);
    if (!product) throw new NotFoundError(`Product ${productId} not found`);

    // TIER 2: Important — degrade with stale cache
    let inventory: InventoryStatus;
    try {
        inventory = await inventoryService.getStatus(productId);
    } catch {
        inventory = await cache.get(`inventory:${productId}`)
            ?? { status: 'unknown', lastChecked: null };
        // Show "(inventory may be outdated)" in UI
    }

    // TIER 3: Enhancement — degrade silently
    const [reviews, recommendations] = await Promise.allSettled([
        reviewsApi.getForProduct(productId),
        recoEngine.getSimilar(productId),
    ]);

    return {
        product,           // Always present
        inventory,         // Always present, may be stale
        reviews:         reviews.status         === 'fulfilled' ? reviews.value         : [],
        recommendations: recommendations.status === 'fulfilled' ? recommendations.value : [],
    };
}
```

---

## Idempotency Patterns

### Idempotency Key Implementation

```typescript
// ✅ Middleware to enforce idempotency on POST routes
async function idempotencyMiddleware(req: Request, res: Response, next: NextFunction) {
    if (req.method !== 'POST') return next();

    const key = req.headers['idempotency-key'] as string;
    if (!key) {
        return res.status(400).json({
            error: 'POST requests require Idempotency-Key header'
        });
    }

    if (!/^[\w-]{8,64}$/.test(key)) {
        return res.status(400).json({ error: 'Invalid Idempotency-Key format' });
    }

    // Check if we've seen this key
    const cacheKey = `idempotency:${req.path}:${key}`;
    const cached = await redis.get(cacheKey);
    if (cached) {
        const { statusCode, body } = JSON.parse(cached);
        return res.status(statusCode).json(body);
    }

    // Capture response for caching
    const originalJson = res.json.bind(res);
    res.json = (body) => {
        // Cache for 24 hours (RFC spec: minimum 24h)
        redis.setex(cacheKey, 86400, JSON.stringify({
            statusCode: res.statusCode,
            body
        }));
        return originalJson(body);
    };

    next();
}
```

### Database Idempotency

```sql
-- ✅ Upsert pattern for idempotent writes
INSERT INTO orders (idempotency_key, user_id, amount, status)
VALUES ($1, $2, $3, 'pending')
ON CONFLICT (idempotency_key)
DO UPDATE SET
    -- Don't change anything — just return existing row
    idempotency_key = EXCLUDED.idempotency_key
RETURNING *;

-- ✅ Conditional insert
INSERT INTO payments (transaction_id, amount, status)
SELECT $1, $2, 'completed'
WHERE NOT EXISTS (
    SELECT 1 FROM payments WHERE transaction_id = $1
);
```

---

## Health Check Patterns

```typescript
// ✅ Comprehensive health check — not just "I'm alive"
app.get('/health', async (req, res) => {
    const startTime = Date.now();

    const checks = await Promise.allSettled([
        checkDatabase(),
        checkRedis(),
        checkMessageQueue(),
        checkExternalDependency(),
    ]);

    const results = {
        status: 'ok',
        version: process.env.npm_package_version,
        uptime: process.uptime(),
        timestamp: new Date().toISOString(),
        responseTime: `${Date.now() - startTime}ms`,
        checks: {
            database:    formatCheck(checks[0]),
            redis:       formatCheck(checks[1]),
            queue:       formatCheck(checks[2]),
            external:    formatCheck(checks[3]),
        },
    };

    const hasFailure = Object.values(results.checks).some(c => c.status === 'fail');
    const hasDegraded = Object.values(results.checks).some(c => c.status === 'degraded');

    if (hasFailure) results.status = 'fail';
    else if (hasDegraded) results.status = 'degraded';

    res.status(hasFailure ? 503 : 200).json(results);
});

async function checkDatabase(): Promise<void> {
    const result = await db.query('SELECT 1');  // Simple connectivity check
    if (!result) throw new Error('Database returned empty result');
}

function formatCheck(settled: PromiseSettledResult<void>) {
    return settled.status === 'fulfilled'
        ? { status: 'ok' }
        : { status: 'fail', error: (settled.reason as Error).message };
}
```

---

## Bulkhead Pattern

Isolate failures so one bad service doesn't consume all resources:

```typescript
// ❌ Single connection pool for everything
// If recommendations service is slow, it uses all DB connections
// Other critical operations (auth, payments) can't get connections

// ✅ Separate pools per criticality tier
const criticalPool = new Pool({ max: 10 });  // Auth, payments
const standardPool = new Pool({ max: 20 });  // General operations
const analyticsPool = new Pool({ max: 5 });  // Analytics (can starve)

// ✅ Separate queues per priority
const criticalQueue = new Bull('critical', { limiter: { max: 1000, duration: 1000 } });
const backgroundQueue = new Bull('background', { limiter: { max: 100, duration: 1000 } });
```

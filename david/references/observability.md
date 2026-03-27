# DAVID — Observability Reference (Scanner U)

Full patterns for logging, metrics, tracing, and monitoring. Load for production backend code.

---

## Structured Logging

### Log Levels — When to Use Each

| Level | When to use | Example |
|-------|-------------|---------|
| `error` | Unexpected failure needing immediate attention | DB connection down, payment failed, unhandled exception |
| `warn` | Expected-but-bad state, no action required now | Auth failure, rate limit hit, deprecated API used |
| `info` | Business events worth knowing | `user.created`, `order.placed`, `payment.processed` |
| `debug` | Technical details for troubleshooting | `cache.miss`, `query.executed`, `http.request` |
| `trace` | Very verbose, disabled in production | Individual function calls, variable states |

### Structured Log Format

```javascript
// ✅ Winston structured logging
import winston from 'winston';
const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()  // Machine-parseable
  ),
  transports: [new winston.transports.Console()],
});

// ✅ Good log entries
logger.info({ event: 'user.login', userId: user.id, ip: req.ip, userAgent: req.headers['user-agent'] });
logger.warn({ event: 'auth.failed', reason: 'invalid_password', email: hashEmail(email) });
logger.error({ event: 'payment.failed', orderId, error: err.message, code: err.code });
logger.debug({ event: 'cache.miss', key, ttl: CACHE_TTL });

// ❌ Bad log entries — DAVID flags these:
console.log('User logged in: ' + userId);         // Unstructured
logger.info('Processing order ' + orderId);        // String concat, not structured
logger.error(err);                                  // Error object may not serialize well
logger.debug({ user });                             // May include PII
logger.info({ event: 'login', password });          // CRITICAL: PII in log
```

### Python Structured Logging

```python
import structlog

logger = structlog.get_logger()

# ✅ Structured
logger.info("user.login", user_id=user.id, ip=request.remote_addr)
logger.warning("auth.failed", reason="invalid_password")
logger.error("payment.failed", order_id=order_id, error=str(e))

# ❌ Unstructured
logging.info(f"User {user_id} logged in from {ip}")
print(f"Error: {error}")
```

---

## PII Detection Patterns

DAVID scans for these patterns and flags as 🔴 CRITICAL:

```javascript
// ❌ Direct PII in logs:
logger.info({ user: { email, password, ssn, creditCard, dateOfBirth } });
logger.debug('Request body:', req.body);          // body may have password
logger.error('Failed for:', JSON.stringify(user)); // serializes all fields
console.log(`Processing ${user.email}`);           // email in log

// ❌ Common indirect PII leaks:
logger.info({ headers: req.headers });             // May have Authorization token
logger.debug({ query: req.query });                // May have session/token params
logger.info({ formData });                          // May have password field

// ✅ Safe patterns:
logger.info({ userId: user.id, event: 'login' });  // ID only, not PII
logger.debug({ path: req.path, method: req.method }); // No body/headers
logger.error({ orderId, errorCode: err.code });    // Structured, no PII

// ✅ Scrubber utility
function scrubPII(obj) {
  const REDACTED = '[REDACTED]';
  const PII_FIELDS = ['password', 'token', 'secret', 'ssn', 'creditCard', 'cvv'];
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) =>
      PII_FIELDS.some(f => k.toLowerCase().includes(f)) ? [k, REDACTED] : [k, v]
    )
  );
}
logger.info(scrubPII(user));
```

---

## Request/Response Logging Middleware

```javascript
// ✅ Express middleware — log all requests
app.use((req, res, next) => {
  const start = Date.now();
  const requestId = crypto.randomUUID();
  
  // Attach request ID for tracing
  req.requestId = requestId;
  res.setHeader('x-request-id', requestId);
  
  logger.info({
    event: 'http.request',
    requestId,
    method: req.method,
    path: req.path,
    ip: req.ip,
  });
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const level = res.statusCode >= 500 ? 'error' : res.statusCode >= 400 ? 'warn' : 'info';
    
    logger[level]({
      event: 'http.response',
      requestId,
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration,
    });
    
    // Flag slow requests
    if (duration > 1000) {
      logger.warn({ event: 'http.slow', requestId, path: req.path, duration });
    }
  });
  
  next();
});
```

---

## Health Check Endpoint

```javascript
// ✅ Health check that actually checks dependencies
app.get('/health', async (req, res) => {
  const checks = await Promise.allSettled([
    checkDatabase(),
    checkRedis(),
    checkExternalAPI(),
  ]);
  
  const results = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
    checks: {
      database: checks[0].status === 'fulfilled' ? 'ok' : 'fail',
      redis:    checks[1].status === 'fulfilled' ? 'ok' : 'fail',
      api:      checks[2].status === 'fulfilled' ? 'ok' : 'fail',
    },
  };
  
  const allOk = Object.values(results.checks).every(v => v === 'ok');
  res.status(allOk ? 200 : 503).json(results);
});

// ❌ Fake health check — DAVID flags this
app.get('/health', (req, res) => res.json({ status: 'ok' }));
// Returns 200 even if DB is down!
```

---

## Distributed Tracing

```javascript
// ✅ Propagate trace context across services
app.use((req, res, next) => {
  // Extract from upstream service
  const traceId = req.headers['x-trace-id'] || crypto.randomUUID();
  const spanId = crypto.randomUUID();
  
  // Attach to request
  req.traceId = traceId;
  req.spanId = spanId;
  
  // Include in all outgoing requests
  req.axiosConfig = {
    headers: {
      'x-trace-id': traceId,
      'x-span-id': spanId,
    },
  };
  
  next();
});

// ✅ Include in all log entries
logger.info({
  event: 'order.created',
  traceId: req.traceId,
  spanId: req.spanId,
  orderId,
});
```

---

## Error Tracking Integration

```javascript
// ❌ Missing error tracking — DAVID flags for production code
app.use((err, req, res, next) => {
  console.error(err);  // Logged but not tracked/alerted
  res.status(500).json({ error: 'Internal server error' });
});

// ✅ Sentry integration
import * as Sentry from '@sentry/node';

Sentry.init({ dsn: process.env.SENTRY_DSN, environment: process.env.NODE_ENV });
app.use(Sentry.Handlers.requestHandler());
// ... routes ...
app.use(Sentry.Handlers.errorHandler());  // Must be after routes

app.use((err, req, res, next) => {
  logger.error({
    event: 'unhandled_error',
    error: err.message,
    stack: err.stack,
    requestId: req.requestId,
    traceId: req.traceId,
  });
  res.status(500).json({ error: 'Internal server error', requestId: req.requestId });
});
```

---

## Missing Observability Checklist

DAVID flags these as 🟠 P1 for production code:

- [ ] No structured logging library (only console.log/print)
- [ ] No request ID / correlation ID on requests
- [ ] No request/response logging middleware
- [ ] No error tracking service (Sentry/Datadog/etc.)
- [ ] `/health` endpoint missing or returns static 200
- [ ] No timing measurement on slow operations
- [ ] PII found in log statements
- [ ] Wrong log levels (errors logged as info, etc.)
- [ ] No distributed trace context propagation
- [ ] Logs with no event/action field (can't query by event type)

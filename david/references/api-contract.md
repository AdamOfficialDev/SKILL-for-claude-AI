# DAVID — API Contract Reference (Scanner AA)

Patterns for REST API consistency, response shapes, and contract validation.

---

## Table of Contents
1. [Response Envelope Consistency](#response-envelope-consistency)
2. [HTTP Method Semantic Violations](#http-method-semantic-violations)
3. [Status Code Correctness](#status-code-correctness)
4. [Pagination Response Shape](#pagination-response-shape)
5. [Validation Error Shape](#validation-error-shape)
6. [Rate Limiting Advisor (Scanner AB)](#rate-limiting-advisor-scanner-ab)

---


## Response Envelope Consistency

```javascript
// ❌ Inconsistent — different shape per endpoint
// GET /users → [{ id, name }]
// POST /users → { user: { id, name }, created: true }
// DELETE /users/:id → { success: true }
// GET /users/:id → { id, name } (no wrapper)
// Error responses vary: { error }, { message }, { errors: [] }

// ✅ Consistent envelope — same shape always
// Success (single):   { data: { id, name },           meta: null }
// Success (list):     { data: [{ id, name }],          meta: { total, page, limit } }
// Success (action):   { data: { affected: 1 },         meta: null }
// Error:              { error: { code, message, details: [] } }
```

### Express Response Helper

```javascript
// ✅ Centralized response shape
const respond = {
  ok: (res, data, meta = null) =>
    res.status(200).json({ data, meta }),
  
  created: (res, data) =>
    res.status(201).json({ data, meta: null }),
  
  noContent: (res) =>
    res.status(204).send(),
  
  badRequest: (res, message, details = []) =>
    res.status(400).json({ error: { code: 'BAD_REQUEST', message, details } }),
  
  unauthorized: (res) =>
    res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Authentication required' } }),
  
  forbidden: (res) =>
    res.status(403).json({ error: { code: 'FORBIDDEN', message: 'Insufficient permissions' } }),
  
  notFound: (res, resource = 'Resource') =>
    res.status(404).json({ error: { code: 'NOT_FOUND', message: `${resource} not found` } }),
  
  conflict: (res, message) =>
    res.status(409).json({ error: { code: 'CONFLICT', message } }),
  
  tooMany: (res) =>
    res.status(429).json({ error: { code: 'RATE_LIMITED', message: 'Too many requests' } }),
  
  serverError: (res, requestId) =>
    res.status(500).json({ error: { code: 'INTERNAL_ERROR', message: 'Something went wrong', requestId } }),
};
```

---

## HTTP Method Semantic Violations

```
CRUD Operations — Correct Mapping:
  CREATE:       POST   /api/resources
  READ (list):  GET    /api/resources
  READ (one):   GET    /api/resources/:id
  FULL UPDATE:  PUT    /api/resources/:id  (replaces entire resource)
  PARTIAL:      PATCH  /api/resources/:id  (updates specific fields)
  DELETE:       DELETE /api/resources/:id

❌ DAVID flags these semantic violations:
  GET    /api/deleteUser?id=1      → should be DELETE /api/users/1
  POST   /api/getUsers             → should be GET    /api/users
  GET    /api/updatePassword       → should be PATCH  /api/users/:id/password
  POST   /api/user/1               → should be GET    /api/users/1
  PUT    /api/users/1 (partial)    → should be PATCH  if only updating some fields
  DELETE /api/clearAll via body    → DELETE with body is non-standard
```

---

## Status Code Correctness

```javascript
// DAVID checks that status codes match the operation:

// ❌ 200 for everything (common mistake)
router.post('/users', async (req, res) => {
  const user = await createUser(req.body);
  res.status(200).json(user);  // Should be 201 Created
});

// ❌ 500 for client errors
if (!req.body.email) {
  res.status(500).json({ error: 'Email required' });  // Should be 400 Bad Request
}

// ❌ 200 with error in body
res.status(200).json({ success: false, error: 'Not found' });  // Should be 404

// ✅ Correct status codes:
// 200 OK         — successful GET, PATCH, PUT, DELETE with response body
// 201 Created    — successful POST that creates a resource
// 204 No Content — successful DELETE or action with no response body
// 400 Bad Request — validation error, malformed request
// 401 Unauthorized — not authenticated (missing/invalid token)
// 403 Forbidden  — authenticated but not authorized
// 404 Not Found  — resource doesn't exist
// 409 Conflict   — duplicate resource, version conflict
// 422 Unprocessable Entity — semantic validation failure
// 429 Too Many Requests — rate limit exceeded
// 500 Internal Server Error — unexpected server failure
```

---

## Pagination Response Shape

```javascript
// ✅ Consistent pagination metadata
{
  "data": [...],
  "meta": {
    "total": 1247,
    "page": 3,
    "limit": 20,
    "totalPages": 63,
    "hasNext": true,
    "hasPrev": true,
    "nextCursor": "eyJpZCI6MjB9"  // For cursor-based pagination
  }
}

// DAVID checks:
// - Total count included (needed for pagination UI)
// - Consistent field names across all list endpoints
// - Cursor-based for large datasets (offset pagination breaks > 10k rows)
```

---

## Validation Error Shape

```javascript
// ✅ Detailed validation errors — tells client exactly what to fix
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "value": "not-an-email"
      },
      {
        "field": "age",
        "message": "Must be at least 18",
        "value": 15
      }
    ]
  }
}

// ❌ Vague validation error — client can't fix without guessing
{ "error": "Invalid input" }
{ "message": "Validation failed" }
```

---

## Rate Limiting Advisor (Scanner AB)

### Missing Debounce on User Inputs

```javascript
// ❌ Fires API on every keystroke
<input onChange={(e) => fetchSearchResults(e.target.value)} />

// ✅ Debounced — fires 300ms after user stops typing
const debouncedSearch = useMemo(
  () => debounce(fetchSearchResults, 300),
  []
);
<input onChange={(e) => debouncedSearch(e.target.value)} />
```

### Double-Submit Prevention

```javascript
// ❌ No submit guard — user can click 5× before response arrives
<button onClick={handleSubmit}>Pay Now</button>

// ✅ Disabled during submission + loading state
<button
  onClick={handleSubmit}
  disabled={isSubmitting}
  aria-busy={isSubmitting}
>
  {isSubmitting ? 'Processing...' : 'Pay Now'}
</button>
```

**Critical paths requiring double-submit protection:**
- Payment / checkout buttons
- Form submit buttons
- Delete / destructive action buttons
- Any button triggering a state-changing API call

### Infinite Scroll / Pagination Throttle

```javascript
// ❌ Fires new request before previous completes
window.addEventListener('scroll', () => {
  if (nearBottom()) fetchNextPage();
});

// ✅ Guard with ref
const isFetchingRef = useRef(false);
window.addEventListener('scroll', async () => {
  if (nearBottom() && !isFetchingRef.current) {
    isFetchingRef.current = true;
    await fetchNextPage();
    isFetchingRef.current = false;
  }
});
```

### Server-Side Rate Limiting Gaps

```javascript
// ❌ Auth endpoints without rate limiting — brute force risk
app.post('/login', async (req, res) => { ... });
app.post('/forgot-password', async (req, res) => { ... });

// ✅ Rate limited
import rateLimit from 'express-rate-limit';
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 10,                    // 10 attempts per window
  standardHeaders: true,
  legacyHeaders: false,
});
app.post('/login', authLimiter, async (req, res) => { ... });
```

### Finding Format

```
⏱️ RATE LIMIT FINDING [AB-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Type       : [Missing debounce / Double-submit / Scroll throttle / Server rate limit]
Severity   : [🔴/🟠/🟡]
File:Line  : [file:N]
Trigger    : [what user action causes the issue]
Risk       : [duplicate API calls / charges / brute force]
Fix        : [exact implementation]
```

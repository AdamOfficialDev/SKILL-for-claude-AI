# DAVID — Documentation Generation Reference (Scanner N)

Scanner N auto-generates documentation on every touched function after a fix.
Load this file when Phase 7 (Documentation Generation) activates, or when the user
explicitly requests docs, JSDoc, docstrings, README updates, or OpenAPI snippets.

## Table of Contents
1. [When Scanner N Activates](#when-scanner-n-activates)
2. [JSDoc / TSDoc — JavaScript & TypeScript](#jsdoc--tsdoc)
3. [Python Docstrings](#python-docstrings)
4. [Go Doc Comments](#go-doc-comments)
5. [Java / Kotlin KDoc](#java--kotlin-kdoc)
6. [Inline Comment Standards](#inline-comment-standards)
7. [README Diff Protocol](#readme-diff-protocol)
8. [OpenAPI Snippet Generator](#openapi-snippet-generator)
9. [The @davidfix Tag](#the-davidfix-tag)
10. [What DAVID Never Documents](#what-david-never-documents)

---

## When Scanner N Activates

Scanner N runs automatically as Phase 7 after every fix delivery. It also activates on
explicit user requests.

| Trigger | Action |
|---------|--------|
| Post-fix (any language) | Add/update docstring on every modified function |
| `"generate docs"` / `"add docs"` / `"document this"` | Full documentation pass on all functions in file |
| `"buat dokumentasi"` / `"tambahkan docs"` / `"jelaskan kodenya"` | Same, in Indonesian context |
| Public API route modified | Generate/update OpenAPI snippet |
| Public function signature changed | README diff if function is exported/public |
| `@davidfix` tag present with no doc update | Flag as incomplete — doc must accompany every fix |

**Scope rule:** DAVID only documents what it touched. Never add docs to functions
DAVID did not modify — that changes unrelated lines and violates Law #4 (Minimal Blast Radius).

---

## JSDoc / TSDoc — JavaScript & TypeScript

### Standard JSDoc Template

```typescript
/**
 * [One-line summary — what this function does, not how.]
 *
 * [Optional: 1–2 sentence explanation of non-obvious behavior,
 *  side effects, or important constraints.]
 *
 * @param paramName - Description of the parameter and its valid range/format.
 * @param optionalParam - Description. Defaults to [value] if not provided.
 * @returns Description of return value. Specify what is returned on each
 *          branch if there are multiple return paths.
 * @throws {ErrorType} When [condition that causes the throw].
 *
 * @example
 * const result = myFunction('input', { flag: true });
 * // result => { id: 1, status: 'ok' }
 *
 * @davidfix [FINDING_ID] - [one-line description of what was fixed]
 */
```

### Real Examples by Pattern

#### Async function with error path
```typescript
/**
 * Fetches a user by ID and returns the public profile.
 *
 * Returns null if the user does not exist. Throws if the database
 * is unreachable or the query times out (>5s).
 *
 * @param userId - UUID v4 string. Must be a valid existing user ID.
 * @returns Public user profile, or null if not found.
 * @throws {DatabaseError} On connection failure or query timeout.
 *
 * @davidfix BUG-001 - Added null check; previously threw on missing user
 */
async function getUserProfile(userId: string): Promise<UserProfile | null> {
```

#### React component
```typescript
/**
 * Displays the user's order history with pagination.
 *
 * Fetches orders on mount. Shows skeleton while loading.
 * Empty state shown when orders array is empty after load.
 *
 * @param userId - ID of the user whose orders to display.
 * @param pageSize - Number of orders per page. Defaults to 20.
 *
 * @davidfix PERF-003 - Memoized order list to prevent re-render on parent state change
 */
export function OrderHistory({ userId, pageSize = 20 }: OrderHistoryProps) {
```

#### Utility / pure function
```typescript
/**
 * Formats a number as Indonesian Rupiah currency string.
 *
 * @param amount - Amount in IDR (whole number, no decimals).
 * @returns Formatted string, e.g. 15000 → "Rp 15.000"
 *
 * @example
 * formatRupiah(15000)  // "Rp 15.000"
 * formatRupiah(0)      // "Rp 0"
 * formatRupiah(-5000)  // "-Rp 5.000"
 */
function formatRupiah(amount: number): string {
```

### TSDoc Additions (TypeScript-specific)

Use TSDoc tags for library/SDK code where tooling will parse the docs:

```typescript
/**
 * @public   — Part of public API; stable across minor versions
 * @beta     — Unstable; may change without notice
 * @internal — Not for external use; may be removed at any time
 * @deprecated Use {@link newFunction} instead. Will be removed in v3.0.
 * @see {@link RelatedClass} for related functionality
 */
```

### What to NEVER put in JSDoc
```typescript
// ❌ Restates the code — zero information value
/** Increments the counter by 1 */
function incrementCounter() { counter++ }

// ❌ States the obvious type (TypeScript already has types)
/** @param id - string — The ID string of the user (string type) */

// ❌ Documents implementation details that will change
/** Uses a for loop to iterate through the array */

// ✅ Documents WHY and WHAT, not HOW
/** Returns the first active admin, prioritising the most recently active. */
```

---

## Python Docstrings

DAVID uses **Google style** as the default. If the codebase already uses NumPy or
Sphinx style, match the existing convention — never mix styles in one file.

### Google Style (Default)

```python
def process_payment(amount: float, currency: str, user_id: str) -> PaymentResult:
    """Process a payment and return the result.

    Calls the payment gateway synchronously. For amounts above 10,000,000 IDR,
    requires a two-step verification which adds ~2s to response time.

    Args:
        amount: Payment amount in the specified currency. Must be positive.
        currency: ISO 4217 currency code (e.g. "IDR", "USD"). Case-insensitive.
        user_id: UUID of the user making the payment. Must be an active user.

    Returns:
        PaymentResult with status, transaction_id, and timestamp.
        Returns PaymentResult(status='failed') on gateway timeout — never raises.

    Raises:
        ValueError: If amount is zero or negative.
        UserNotFoundError: If user_id does not correspond to an active user.

    Example:
        >>> result = process_payment(150000, "IDR", "uuid-123")
        >>> result.status
        'success'

    Note:
        @davidfix SEC-002 - Added amount validation; previously allowed negative amounts
    """
```

### Class Docstring

```python
class OrderRepository:
    """Handles all database operations for the Order domain.

    Wraps the SQLAlchemy session with domain-specific query methods.
    All methods are transactional — call commit() explicitly after writes.

    Attributes:
        db: Active SQLAlchemy session. Injected via dependency injection.
        _cache: In-memory LRU cache for recent order lookups (max 128 entries).
    """
```

### Short function — one-liner is fine

```python
def is_valid_email(email: str) -> bool:
    """Return True if email matches RFC 5322 format, False otherwise."""
```

### When to use `#` inline comments vs docstrings

| Use docstring | Use `#` inline comment |
|--------------|------------------------|
| Every function, method, class | Non-obvious single line inside a function |
| Module-level explanation | Temporary workaround with ticket reference |
| Public API | Algorithm step that needs explanation |
| Complex return value | Magic number or threshold value |

---

## Go Doc Comments

Go uses `//` comments directly above the declaration. No `/** */` syntax.

```go
// ProcessPayment processes a payment request and returns the result.
//
// It calls the payment gateway synchronously with a 5-second timeout.
// Returns an error if the gateway is unavailable or the amount is invalid.
// For amounts above 10,000,000 IDR, a two-step verification is triggered.
//
// Example:
//
//	result, err := ProcessPayment(ctx, PaymentRequest{
//	    Amount:   150000,
//	    Currency: "IDR",
//	    UserID:   "uuid-123",
//	})
//
// davidfix BUG-004: Added context cancellation support
func ProcessPayment(ctx context.Context, req PaymentRequest) (*PaymentResult, error) {
```

**Go doc rules:**
- First sentence is the summary — used in `go doc` output. Must start with the function name.
- Leave blank line before extended description.
- Example block uses `//` with a tab indent inside.
- Exported names only — unexported functions get `//` inline comments, not full doc blocks.

---

## Java / Kotlin KDoc

### Java Javadoc
```java
/**
 * Processes a payment and returns the transaction result.
 *
 * <p>This method calls the payment gateway synchronously.
 * For amounts above 10,000,000 IDR, two-step verification is triggered,
 * adding approximately 2 seconds to the response time.</p>
 *
 * @param amount   Payment amount. Must be a positive value.
 * @param currency ISO 4217 currency code (case-insensitive).
 * @param userId   UUID of the authenticated user.
 * @return {@link PaymentResult} with status, transactionId, and timestamp.
 * @throws IllegalArgumentException if amount is zero or negative.
 * @throws UserNotFoundException if userId does not match an active user.
 *
 * <!-- @davidfix SEC-002: Added amount validation -->
 */
public PaymentResult processPayment(BigDecimal amount, String currency, UUID userId) {
```

### Kotlin KDoc
```kotlin
/**
 * Processes a payment and returns the transaction result.
 *
 * Calls the payment gateway synchronously. For amounts above 10,000,000 IDR,
 * two-step verification is triggered (~2s additional latency).
 *
 * @param amount Payment amount. Must be positive.
 * @param currency ISO 4217 currency code (case-insensitive).
 * @param userId UUID of the authenticated user.
 * @return [PaymentResult] with status, transactionId, and timestamp.
 * @throws IllegalArgumentException if amount is zero or negative.
 *
 * @sample com.example.PaymentExamples.processPaymentExample
 */
suspend fun processPayment(amount: BigDecimal, currency: String, userId: UUID): PaymentResult {
```

---

## Inline Comment Standards

Applies to all languages. These rules complement docstrings — they govern comments
*inside* function bodies.

### The WHY Rule — most important rule

```python
# ❌ States the obvious — never write this
# Loop through the orders
for order in orders:

# ✅ Explains a non-obvious decision
# select_related avoids N+1: each order fetches user in same query
for order in orders.select_related('user'):
```

### When inline comments are required

| Situation | Comment required |
|-----------|-----------------|
| Magic number or threshold | Yes — always explain origin |
| Non-obvious algorithm step | Yes |
| Workaround for external bug | Yes — include ticket/issue reference |
| Performance-critical shortcut | Yes |
| `@davidfix` annotation | Yes — one-liner on what was fixed |
| Obvious code | No |

### @davidfix annotation format
```python
# @davidfix BUG-003: was `user.id` — now `user.uuid` to match API contract
user_id = user.uuid
```
```typescript
// @davidfix SEC-001: sanitize input before DB write — was directly using req.body
const sanitized = sanitizeInput(req.body.username);
```

---

## README Diff Protocol

Scanner N updates the README when DAVID modifies a public API.

### When README update is required

| Change | README action |
|--------|--------------|
| Public function signature changed | Update usage example in README |
| New endpoint added | Add to API reference section |
| Breaking change in exported interface | Add to "Breaking Changes" or "Migration" section |
| Environment variable added | Update `.env.example` docs |
| New CLI flag or command | Update usage/CLI section |

### README update output format

```markdown
📄 README UPDATE REQUIRED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Function : getUserProfile(userId)
Change   : Added optional `includeDeleted` param (defaults to false)
Section  : ## API Reference > Users

BEFORE:
  getUserProfile(userId: string): Promise<User>

AFTER:
  getUserProfile(userId: string, includeDeleted?: boolean): Promise<User>

Update the README section "## API Reference > Users" with the new signature.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## OpenAPI Snippet Generator

Activates when DAVID modifies an API route. Generates/updates the OpenAPI 3.0 snippet.

### Output format

```yaml
# DAVID — OpenAPI snippet for POST /users/{id}/deactivate
# Add or update in your openapi.yaml / swagger.json

paths:
  /users/{id}/deactivate:
    post:
      summary: Deactivate a user account
      description: |
        Soft-deletes the user by setting deleted_at timestamp.
        Does not remove any associated data (orders, comments).
        Requires admin scope.
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [reason]
              properties:
                reason:
                  type: string
                  description: Reason for deactivation (for audit log)
                  example: "Violated terms of service"
      responses:
        '200':
          description: User deactivated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found
        '403':
          description: Insufficient permissions (requires admin scope)
      security:
        - bearerAuth: []
      tags:
        - Users
```

### When NOT to generate OpenAPI

- Internal utility functions
- Private methods or unexported functions
- Routes that are already documented and were not modified by DAVID
- Test helper functions

---

## The @davidfix Tag

Every function modified by DAVID gets a `@davidfix` annotation. This is non-negotiable
(Inviolable Law #9: Document What You Change).

### Format per language

```typescript
// TypeScript / JavaScript
/** @davidfix [ID]: [one-line description of what was fixed] */

// Python
# @davidfix [ID]: [description]

// Go
// davidfix [ID]: [description]

// Java
/* @davidfix [ID]: [description] */
```

### What goes in the description

```
✅ "@davidfix BUG-002: added null check for missing user before accessing .email"
✅ "@davidfix SEC-001: replaced string concatenation with parameterized query"
✅ "@davidfix PERF-003: memoized result to prevent N+1 on repeated calls"

❌ "@davidfix: fixed bug"   — too vague
❌ "@davidfix BUG-001"      — no description
```

---

## What DAVID Never Documents

```
❌ Functions DAVID did not modify (scope violation — Law #4)
❌ Private implementation details that will change frequently
❌ Restating what the code already clearly says
❌ TODO comments without an owner and ticket reference
❌ Commenting out code instead of deleting it
❌ Adding docs to test helper functions unless explicitly requested
```

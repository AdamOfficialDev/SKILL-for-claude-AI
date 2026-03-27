# DAVID — Test Quality Audit Reference (Scanner AD2)

Deep patterns for auditing existing tests. This is NOT about generating new tests — it's about finding tests that are broken, useless, or dangerous.

---

## The Danger of Bad Tests

A test suite with 90% coverage but bad assertions is worse than 0% coverage. It:
1. Creates false confidence — "all tests pass" but bugs ship
2. Breaks on every refactor — teams disable tests to "fix" them
3. Slows development — flaky tests waste hours of CI time
4. Hides real problems — noisy test output masks real failures

---

## Anti-Assertion Taxonomy

### Category 1: Always-Passing Assertions

```javascript
// DAVID flags ALL of these patterns:

// Truthiness (passes for any non-falsy value including bugs)
expect(result).toBeTruthy();
expect(result).toBeDefined();
expect(!!result).toBe(true);

// Existence without shape (passes for empty object)
expect(response.data).toBeDefined();
expect(user).not.toBeNull();

// Type without value (too loose)
expect(typeof result).toBe('object');
expect(Array.isArray(items)).toBe(true);

// Snapshot traps (auto-updated without review)
expect(component).toMatchSnapshot();  // Flag if snapshots are > 50 lines or never reviewed
```

**The test DAVID wants instead:**
```javascript
// Assert the actual expected value — specific and meaningful
expect(result).toEqual({
    id: 1,
    name: 'Alice',
    role: 'admin',
    active: true
});
expect(items).toHaveLength(3);
expect(items[0].id).toBe(42);
expect(response.status).toBe(201);
expect(response.body.user.email).toBe('alice@example.com');
```

### Category 2: Testing Implementation Details

```javascript
// ❌ DAVID flags: testing internal method calls
test('calls the discount calculator', () => {
    const spy = jest.spyOn(orderService, '_calculateDiscount');
    orderService.processOrder(order);
    expect(spy).toHaveBeenCalledWith('gold', 100);
});
// Problem: renaming _calculateDiscount breaks this test even if behavior is correct

// ❌ Testing internal state
test('sets loading state', () => {
    component.handleSubmit();
    expect(component.state.isLoading).toBe(true);  // Implementation detail
});

// ❌ Testing mock calls instead of behavior
test('saves user', async () => {
    await userService.create(userData);
    expect(mockDb.save).toHaveBeenCalledTimes(1);  // Tests the mock, not the outcome
});

// ✅ Test behavior / outcome
test('returns discounted price for gold tier users', () => {
    const price = orderService.processOrder({ ...order, userTier: 'gold' });
    expect(price).toBe(80);  // 20% off
});

test('creates user and returns created record', async () => {
    const user = await userService.create(userData);
    expect(user.id).toBeDefined();
    expect(user.email).toBe(userData.email);
    // If you want DB verification: query it
    const found = await db.findUser(user.id);
    expect(found.email).toBe(userData.email);
});
```

---

## Flaky Test Patterns

### Timing Dependencies

```javascript
// ❌ Hard-coded delays
await new Promise(resolve => setTimeout(resolve, 500));
await page.waitForTimeout(1000);

// ❌ Race conditions
const result = fetchData();  // Promise not awaited
expect(cache.size).toBe(1);  // May run before fetch completes

// ✅ Wait for condition
await waitFor(() => expect(element).toBeVisible());
await screen.findByText('Success');  // Waits automatically
await expect(locator).toBeVisible();  // Playwright style
```

### Environment Dependencies

```javascript
// ❌ Tests that fail in different timezones
const today = new Date().toLocaleDateString();
expect(result).toBe('1/15/2024');  // Fails in non-US locales

// ❌ Tests that fail based on system locale
expect(1234.56.toLocaleString()).toBe('1,234.56');  // Different in German: '1.234,56'

// ❌ Tests that fail on different OS (path separators)
expect(filePath).toBe('src/components/Button.js');  // \ vs / on Windows

// ✅ Locale-neutral
jest.useFakeTimers().setSystemTime(new Date('2024-01-15T12:00:00Z'));
expect(formatDate(new Date(), 'en-US')).toBe('1/15/2024');

// ✅ OS-neutral paths
expect(filePath).toBe(path.join('src', 'components', 'Button.js'));
```

### Network Dependencies

```javascript
// ❌ Real network calls in unit/integration tests
test('sends email', async () => {
    await emailService.sendWelcome(user);
    // Hits real SendGrid API — fails without internet, costs money, non-deterministic
});

// ✅ Mock the network boundary
jest.mock('../services/email', () => ({
    sendWelcome: jest.fn().mockResolvedValue({ messageId: 'test-id' }),
}));

test('sends welcome email on registration', async () => {
    await userService.register(userData);
    expect(emailService.sendWelcome).toHaveBeenCalledWith(
        expect.objectContaining({ email: userData.email })
    );
});
```

---

## Test Isolation Failures

### Shared Mutable State

```javascript
// ❌ Global state mutated between tests
const db = new InMemoryDb();  // Module-level — shared across all tests

test('creates item', () => {
    db.items.push({ id: 1 });
});

test('counts items', () => {
    expect(db.items.length).toBe(0);  // FAILS if first test ran — order-dependent
});

// ✅ Fresh state per test
let db;
beforeEach(() => {
    db = new InMemoryDb();  // Fresh instance for each test
});
// OR
afterEach(() => {
    db.clear();
});
```

### Missing Mock Cleanup

```javascript
// ❌ Mocks leak between tests
test('first test', () => {
    global.fetch = jest.fn().mockResolvedValue({ ok: true });
    // fetch is now mocked for ALL subsequent tests in this file
});

test('second test', () => {
    // Accidentally uses mocked fetch from previous test
});

// ✅ Restore after each test
afterEach(() => {
    jest.restoreAllMocks();
    // OR configure globally: jest.config: { restoreMocks: true }
});
```

---

## Coverage Quality Analysis

DAVID distinguishes meaningful vs meaningless coverage:

```
Coverage Report Analysis:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

src/services/payment.ts    95% coverage
  ✅ Happy path tested with real assertions
  ✅ Error paths tested
  ⚠️  Lines 47-52: covered but assertion is toBeTruthy() — weak
  ❌ Timeout scenario: covered by mock but assertion missing

src/utils/validator.ts     100% coverage
  ❌ 100% coverage but ONLY tests that return true
  ❌ No test for invalid inputs — branches covered by truthy checks only
  ❌ Edge cases (empty string, null, special chars) NOT tested

src/components/Button.tsx  80% coverage
  ✅ Click handler tested with outcome assertion
  ❌ Loading state not tested
  ❌ Disabled state not tested
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  Coverage % is misleading — quality matters more than quantity
```

---

## Python Test Quality Patterns

```python
# ❌ Weak assertions in pytest
def test_get_user():
    user = get_user(1)
    assert user  # Passes for any truthy value — empty dict {} would pass!
    assert user is not None  # Same problem

# ✅ Assert specific structure
def test_get_user():
    user = get_user(1)
    assert user == {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'}
    # OR with dataclass/pydantic:
    assert user.id == 1
    assert user.email == 'alice@example.com'
    assert user.is_active is True

# ❌ Missing parametrize for multiple cases
def test_validate_email_valid():
    assert validate_email('user@example.com')
def test_validate_email_valid2():
    assert validate_email('user+tag@example.co.uk')

# ✅ Use parametrize
@pytest.mark.parametrize('email,expected', [
    ('user@example.com', True),
    ('user+tag@example.co.uk', True),
    ('not-an-email', False),
    ('', False),
    (None, False),
    ('a@b', False),
])
def test_validate_email(email, expected):
    assert validate_email(email) == expected
```

---

## Go Test Quality Patterns

```go
// ❌ Test doesn't check the actual value
func TestGetUser(t *testing.T) {
    user, err := GetUser(1)
    if err != nil {
        t.Fatal(err)
    }
    // No assertion about user! Test passes even if user is empty struct
}

// ✅ Assert what matters
func TestGetUser(t *testing.T) {
    user, err := GetUser(1)
    if err != nil {
        t.Fatalf("GetUser(1) returned error: %v", err)
    }
    if user.ID != 1 {
        t.Errorf("user.ID = %d, want 1", user.ID)
    }
    if user.Email != "alice@example.com" {
        t.Errorf("user.Email = %q, want %q", user.Email, "alice@example.com")
    }
}

// ✅ Table-driven for multiple cases
func TestValidateEmail(t *testing.T) {
    tests := []struct{
        input string
        want  bool
    }{
        {"user@example.com", true},
        {"not-an-email", false},
        {"", false},
    }
    for _, tt := range tests {
        t.Run(tt.input, func(t *testing.T) {
            got := ValidateEmail(tt.input)
            if got != tt.want {
                t.Errorf("ValidateEmail(%q) = %v, want %v", tt.input, got, tt.want)
            }
        })
    }
}
```

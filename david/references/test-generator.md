# DAVID — Test Generator Reference

Templates and patterns for auto-generating unit tests after fixing bugs. Load when in TESTGEN mode.

---

## Table of Contents
1. [Test Strategy by Bug Type](#test-strategy-by-bug-type)
2. [Jest (JavaScript / TypeScript)](#jest)
3. [Pytest (Python)](#pytest)
4. [JUnit 5 (Java / Kotlin)](#junit-5)
5. [Go test](#go-test)
6. [Vitest (Vite / Vue / React)](#vitest)
7. [RSpec (Ruby)](#rspec)
8. [Test Naming Conventions](#test-naming-conventions)

---

## Test Strategy by Bug Type

For each bug type, generate these test categories:

| Bug Type | Required Tests |
|----------|---------------|
| Null/undefined dereference | Null input, undefined input, missing property |
| Index out of bounds | Empty array, single element, exactly-at-boundary |
| Logic error (wrong condition) | True branch, false branch, boundary value |
| Async / missing await | Resolved promise, rejected promise, timeout |
| Security (injection) | Malicious payload, special chars, empty string |
| Security (auth bypass) | Unauthenticated, wrong user, correct user |
| Performance (N+1) | Single item, many items (assert query count) |
| Type mismatch | Wrong type, correct type, coerced type |

### DAVID Test Naming Formula

```
test('[functionName] [scenario] [expected outcome]')

Examples:
test('getUser returns null when user does not exist')
test('processPayment throws InsufficientFundsError when balance is zero')
test('validateEmail accepts valid RFC 5322 address')
test('validateEmail rejects SQL injection payload')
```

---

## Jest

### File Setup

```javascript
// __tests__/[module].test.js  OR  [module].test.js
import { functionToTest } from '../path/to/module';

// If testing Express routes:
import request from 'supertest';
import app from '../app';

// Mock dependencies
jest.mock('../services/database', () => ({
  findUser: jest.fn(),
  saveUser: jest.fn(),
}));
import { findUser } from '../services/database';
```

### Standard Test Templates

```javascript
describe('[ClassName or module name]', () => {

  // ── Setup & Teardown ─────────────────────────────────────
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // cleanup if needed
  });

  // ── 🐛 Regression Test (one per fixed bug) ───────────────
  describe('[functionName]', () => {

    // BUG-001: Empty array caused IndexError
    test('returns null for empty input array', () => {
      expect(processItems([])).toBeNull();
    });

    // BUG-002: Missing await caused race condition
    test('resolves correctly under concurrent calls', async () => {
      const [r1, r2] = await Promise.all([
        processAsync('a'),
        processAsync('b'),
      ]);
      expect(r1).toBe('A');
      expect(r2).toBe('B');
    });

    // ── Edge Cases ──────────────────────────────────────────
    test('handles null input without throwing', () => {
      expect(() => processItems(null)).not.toThrow();
    });

    test('handles undefined input without throwing', () => {
      expect(() => processItems(undefined)).not.toThrow();
    });

    test('handles single-element array', () => {
      expect(processItems(['x'])).toEqual(['X']);
    });

    test('handles maximum allowed input size', () => {
      const large = Array(10000).fill('a');
      expect(() => processItems(large)).not.toThrow();
    });

    // ── Happy Path ──────────────────────────────────────────
    test('processes valid input and returns expected output', () => {
      const input = ['a', 'b', 'c'];
      const expected = ['A', 'B', 'C'];
      expect(processItems(input)).toEqual(expected);
    });

    // ── Error Cases ─────────────────────────────────────────
    test('throws [SpecificError] when [condition]', () => {
      expect(() => processItems('invalid-type')).toThrow(TypeError);
      expect(() => processItems('invalid-type')).toThrow('Expected array');
    });
  });

  // ── 🔐 Security Tests ────────────────────────────────────
  describe('security', () => {
    test('rejects SQL injection in input fields', async () => {
      const malicious = "'; DROP TABLE users; --";
      const result = await findUser({ username: malicious });
      expect(result).toBeNull();
      // Ensure DB was called with parameterized query, not raw string
      expect(findUser).toHaveBeenCalledWith({ username: malicious });
    });

    test('sanitizes XSS payload in display fields', () => {
      const xss = '<script>alert("xss")</script>';
      const result = sanitizeInput(xss);
      expect(result).not.toContain('<script>');
    });
  });

  // ── 🌐 HTTP Route Tests (Supertest) ─────────────────────
  describe('POST /api/users', () => {
    test('returns 201 and user object on valid input', async () => {
      const res = await request(app)
        .post('/api/users')
        .send({ name: 'Alice', email: 'alice@example.com' })
        .expect(201);

      expect(res.body).toMatchObject({ name: 'Alice' });
      expect(res.body.id).toBeDefined();
    });

    test('returns 400 on missing required fields', async () => {
      const res = await request(app)
        .post('/api/users')
        .send({ name: 'Alice' })  // Missing email
        .expect(400);

      expect(res.body.error).toBeDefined();
    });

    test('returns 401 when not authenticated', async () => {
      await request(app)
        .post('/api/users')
        .send({ name: 'Alice', email: 'alice@example.com' })
        .expect(401);
    });
  });
});
```

---

## Pytest

### File Setup

```python
# tests/test_[module].py
import pytest
from unittest.mock import MagicMock, patch, AsyncMock
from myapp.module import function_to_test

# Fixtures
@pytest.fixture
def mock_db():
    db = MagicMock()
    db.query.return_value = [{"id": 1, "name": "Alice"}]
    return db

@pytest.fixture
def sample_user():
    return {"id": 1, "name": "Alice", "email": "alice@example.com"}
```

### Standard Test Templates

```python
class TestFunctionName:

    # ── 🐛 Regression Tests ──────────────────────────────────
    def test_handles_empty_list_without_index_error(self):
        """BUG-001: Empty list caused IndexError."""
        result = process_items([])
        assert result is None

    def test_handles_none_input(self):
        result = process_items(None)
        assert result is None

    # ── Edge Cases ───────────────────────────────────────────
    def test_single_element_list(self):
        assert process_items(["a"]) == ["A"]

    def test_boundary_value_at_max(self):
        large = ["x"] * 10_000
        result = process_items(large)
        assert len(result) == 10_000

    @pytest.mark.parametrize("invalid_input,expected_error", [
        (123, TypeError),
        ("string", TypeError),
        ({}, TypeError),
    ])
    def test_raises_type_error_on_invalid_input(self, invalid_input, expected_error):
        with pytest.raises(expected_error):
            process_items(invalid_input)

    # ── Happy Path ───────────────────────────────────────────
    def test_processes_valid_input_correctly(self):
        assert process_items(["a", "b", "c"]) == ["A", "B", "C"]

    # ── Async Tests ──────────────────────────────────────────
    @pytest.mark.asyncio
    async def test_async_function_resolves_correctly(self):
        result = await fetch_data("key")
        assert result is not None

    @pytest.mark.asyncio
    async def test_async_function_handles_exception(self):
        with pytest.raises(ServiceError):
            await fetch_data("bad-key")


class TestSecurity:

    def test_rejects_sql_injection(self, mock_db):
        """SEC-001: SQL injection in username parameter."""
        payload = "'; DROP TABLE users; --"
        with patch('myapp.module.db', mock_db):
            result = find_user(payload)
        # DB should have been called with parameterized query
        mock_db.execute.assert_called_once()
        call_args = mock_db.execute.call_args
        # The payload should not be in the raw SQL string
        assert payload not in str(call_args[0][0])

    def test_sanitizes_html_in_user_content(self):
        """SEC-002: XSS via unsanitized HTML."""
        xss = '<script>alert("xss")</script>'
        result = sanitize_input(xss)
        assert '<script>' not in result


class TestPerformance:

    def test_batch_query_not_n_plus_one(self, mock_db):
        """PERF-001: Should use single batch query, not N queries."""
        orders = [{"id": i, "user_id": i} for i in range(100)]
        process_orders(orders, db=mock_db)
        # Should be called at most twice (orders + users batch), not 101 times
        assert mock_db.query.call_count <= 2
```

### FastAPI Test Template

```python
from fastapi.testclient import TestClient
from myapp.main import app

client = TestClient(app)

class TestUserEndpoint:

    def test_get_user_returns_200_for_valid_id(self):
        response = client.get("/users/1")
        assert response.status_code == 200
        assert response.json()["id"] == 1

    def test_get_user_returns_404_for_unknown_id(self):
        response = client.get("/users/99999")
        assert response.status_code == 404

    def test_create_user_returns_422_on_invalid_email(self):
        response = client.post("/users", json={"name": "Alice", "email": "not-an-email"})
        assert response.status_code == 422

    def test_endpoint_requires_authentication(self):
        response = client.get("/users/me")
        assert response.status_code == 401
```

---

## JUnit 5

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@DisplayName("UserService")
class UserServiceTest {

    private UserRepository mockRepo;
    private UserService service;

    @BeforeEach
    void setUp() {
        mockRepo = mock(UserRepository.class);
        service = new UserService(mockRepo);
    }

    // ── 🐛 Regression ────────────────────────────────────────
    @Test
    @DisplayName("returns empty Optional when user not found (BUG-001)")
    void returnsEmptyWhenUserNotFound() {
        when(mockRepo.findById(99L)).thenReturn(Optional.empty());
        Optional<User> result = service.getUser(99L);
        assertTrue(result.isEmpty());
    }

    // ── Parametrized Edge Cases ──────────────────────────────
    @ParameterizedTest
    @NullAndEmptySource
    @DisplayName("rejects null or empty email")
    void rejectsNullOrEmptyEmail(String email) {
        assertThrows(IllegalArgumentException.class,
            () -> service.createUser("Alice", email));
    }

    @ParameterizedTest
    @ValueSource(strings = {"not-email", "@nodomain", "no@", "spaces in@email.com"})
    @DisplayName("rejects malformed email addresses")
    void rejectsMalformedEmail(String email) {
        assertThrows(ValidationException.class,
            () -> service.createUser("Alice", email));
    }

    // ── Happy Path ───────────────────────────────────────────
    @Test
    @DisplayName("creates user and returns persisted entity")
    void createsUserSuccessfully() {
        User saved = new User(1L, "Alice", "alice@example.com");
        when(mockRepo.save(any(User.class))).thenReturn(saved);

        User result = service.createUser("Alice", "alice@example.com");

        assertAll(
            () -> assertEquals("Alice", result.getName()),
            () -> assertEquals("alice@example.com", result.getEmail()),
            () -> assertNotNull(result.getId())
        );
        verify(mockRepo, times(1)).save(any(User.class));
    }

    // ── Security ─────────────────────────────────────────────
    @Test
    @DisplayName("uses parameterized query for SQL injection safety")
    void usesSafeParameterizedQuery() {
        String malicious = "'; DROP TABLE users; --";
        service.findByUsername(malicious);
        // Verify the repo was called (not a raw string query execution)
        verify(mockRepo).findByUsername(malicious);
    }
}
```

---

## Go test

```go
package mypackage_test

import (
    "testing"
    "errors"
    "github.com/yourorg/yourrepo/mypackage"
)

// ── 🐛 Regression ──────────────────────────────────────────
func TestProcessItems_EmptySlice(t *testing.T) {
    // BUG-001: panic on empty slice
    result, err := mypackage.ProcessItems([]string{})
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if result != nil {
        t.Errorf("expected nil for empty input, got %v", result)
    }
}

// ── Table-Driven Tests (Go idiom) ──────────────────────────
func TestProcessItems(t *testing.T) {
    tests := []struct {
        name    string
        input   []string
        want    []string
        wantErr bool
    }{
        {
            name:  "valid input",
            input: []string{"a", "b", "c"},
            want:  []string{"A", "B", "C"},
        },
        {
            name:    "nil input",
            input:   nil,
            want:    nil,
            wantErr: false,
        },
        {
            name:    "empty slice",
            input:   []string{},
            want:    nil,
            wantErr: false,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := mypackage.ProcessItems(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("ProcessItems() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("ProcessItems() = %v, want %v", got, tt.want)
            }
        })
    }
}

// ── Benchmark ───────────────────────────────────────────────
func BenchmarkProcessItems(b *testing.B) {
    input := make([]string, 1000)
    for i := range input {
        input[i] = "item"
    }
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        mypackage.ProcessItems(input)
    }
}
```

---

## Vitest

Same API as Jest. Key differences:

```javascript
import { describe, test, expect, vi, beforeEach } from 'vitest';
import { mount } from '@vue/test-utils';  // or @testing-library/react

// Mocking
vi.mock('../services/api');

// Spy
const spy = vi.spyOn(object, 'method');

// Timer mocks
vi.useFakeTimers();
vi.advanceTimersByTime(1000);
vi.useRealTimers();
```

---

## RSpec

```ruby
require 'rails_helper'

RSpec.describe UserService do
  let(:service) { described_class.new }
  let(:user) { create(:user) }

  # 🐛 Regression
  describe '#process_items' do
    context 'with empty array (BUG-001)' do
      it 'returns nil without raising' do
        expect { service.process_items([]) }.not_to raise_error
        expect(service.process_items([])).to be_nil
      end
    end

    context 'with valid input' do
      it 'returns processed results' do
        expect(service.process_items(['a', 'b'])).to eq(['A', 'B'])
      end
    end

    context 'with nil' do
      it 'returns nil gracefully' do
        expect(service.process_items(nil)).to be_nil
      end
    end
  end

  # 🔐 Security
  describe '#find_user' do
    it 'uses parameterized query (no SQL injection)' do
      expect(User).to receive(:where).with(username: "'; DROP TABLE users; --")
      service.find_user("'; DROP TABLE users; --")
    end
  end
end
```

---

## Test Naming Conventions

### Pattern
```
[unit under test] [condition/scenario] [expected result]
```

### Examples by Language

**Jest/Vitest:**
```javascript
test('processItems returns null for empty array')
test('processItems throws TypeError when given a string')
test('findUser returns null when user does not exist')
```

**Pytest:**
```python
def test_process_items_returns_none_for_empty_list():
def test_process_items_raises_type_error_for_string_input():
def test_find_user_returns_none_when_not_found():
```

**JUnit:**
```java
@DisplayName("processItems returns null for empty list")
@DisplayName("processItems throws TypeError for string input")
```

**Go:**
```go
func TestProcessItems_EmptySlice_ReturnsNil(t *testing.T) {}
func TestProcessItems_StringInput_ReturnsError(t *testing.T) {}
```

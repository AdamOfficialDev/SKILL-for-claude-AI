# REY — Testing Guide

> "Code without tests is legacy code the moment it's written." — Michael Feathers

---

## 🧪 TESTING PHILOSOPHY

```
TEST PYRAMID (REY's approach):

        /\
       /  \     E2E (Playwright)
      /----\    → Critical user journeys only
     /      \   → Login, checkout, core flow
    /--------\
   /          \  Integration Tests (Vitest)
  /            \ → API routes, DB queries
 /              \ → Server Actions, hooks
/----------------\
                   Unit Tests (Vitest)
                   → Utilities, validators, pure functions
                   → Most numerous, fastest
```

**REY Testing Rules:**
1. Test behavior, not implementation — test what users see, not how code works internally
2. Fast tests = tests that get run — slow tests get skipped
3. Test the unhappy path — happy path works by default; edge cases don't
4. One assertion per test (ideally) — easier to diagnose failures
5. Never mock what you don't own — mock third-party APIs, not your own modules

---

## ⚡ VITEST SETUP

### Installation
```bash
pnpm add -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/user-event @testing-library/jest-dom
```

### Config
```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config"
import react from "@vitejs/plugin-react"
import path from "path"

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./tests/setup.ts"],
    include: ["src/**/*.{test,spec}.{ts,tsx}", "tests/unit/**/*.{test,spec}.{ts,tsx}"],
    exclude: ["node_modules", ".next", "tests/e2e"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html", "lcov"],
      include: ["src/**/*.{ts,tsx}"],
      exclude: [
        "src/**/*.d.ts",
        "src/**/*.stories.{ts,tsx}",
        "src/**/index.ts",
        "src/types/**",
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 70,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
})
```

### Setup File
```typescript
// tests/setup.ts
import "@testing-library/jest-dom"
import { cleanup } from "@testing-library/react"
import { afterEach, vi } from "vitest"

// Cleanup after each test
afterEach(() => {
  cleanup()
})

// Mock Next.js router
vi.mock("next/navigation", () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    refresh: vi.fn(),
    back: vi.fn(),
    forward: vi.fn(),
    prefetch: vi.fn(),
  }),
  usePathname: () => "/",
  useSearchParams: () => new URLSearchParams(),
}))

// Mock Next.js headers
vi.mock("next/headers", () => ({
  headers: () => new Map(),
  cookies: () => ({
    get: vi.fn(),
    set: vi.fn(),
    delete: vi.fn(),
  }),
}))

// Mock environment variables
process.env.NEXT_PUBLIC_APP_URL = "http://localhost:3000"
process.env.DATABASE_URL = "postgresql://test:test@localhost:5432/testdb"

// Suppress console.error for expected errors in tests
const originalError = console.error
beforeEach(() => {
  console.error = (...args: unknown[]) => {
    if (typeof args[0] === "string" && args[0].includes("Warning:")) return
    originalError.call(console, ...args)
  }
})
afterEach(() => {
  console.error = originalError
})
```

### Package.json Scripts
```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:watch": "vitest watch",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

---

## 🧩 UNIT TESTS

### Testing Utilities
```typescript
// tests/unit/lib/utils.test.ts
import { describe, it, expect } from "vitest"
import { formatCurrency, formatRelativeTime, truncate } from "@/lib/utils"

describe("formatCurrency", () => {
  it("formats USD correctly", () => {
    expect(formatCurrency(1234.56)).toBe("$1,235")
  })

  it("formats with cents when present", () => {
    expect(formatCurrency(1234.56, "USD", "en-US")).toBe("$1,235")
  })

  it("handles zero", () => {
    expect(formatCurrency(0)).toBe("$0")
  })

  it("handles negative numbers", () => {
    expect(formatCurrency(-500)).toBe("-$500")
  })

  it("formats different currencies", () => {
    expect(formatCurrency(1000, "EUR", "de-DE")).toContain("1.000")
  })
})

describe("truncate", () => {
  it("returns original string if shorter than maxLength", () => {
    expect(truncate("hello", 10)).toBe("hello")
  })

  it("truncates and adds ellipsis", () => {
    expect(truncate("hello world", 8)).toBe("hello...")
  })

  it("handles exact length", () => {
    expect(truncate("hello", 5)).toBe("hello")
  })
})
```

### Testing Zod Validators
```typescript
// tests/unit/lib/validations/user.test.ts
import { describe, it, expect } from "vitest"
import { createUserSchema, updateProfileSchema } from "@/lib/validations/user"

describe("createUserSchema", () => {
  it("accepts valid user data", () => {
    const result = createUserSchema.safeParse({
      email: "test@example.com",
      name: "John Doe",
      role: "user",
    })
    expect(result.success).toBe(true)
  })

  it("rejects invalid email", () => {
    const result = createUserSchema.safeParse({
      email: "not-an-email",
      name: "John",
    })
    expect(result.success).toBe(false)
    expect(result.error?.errors[0].path).toContain("email")
  })

  it("rejects name too short", () => {
    const result = createUserSchema.safeParse({
      email: "test@example.com",
      name: "J",
    })
    expect(result.success).toBe(false)
  })

  it("applies defaults", () => {
    const result = createUserSchema.safeParse({
      email: "test@example.com",
      name: "John",
    })
    expect(result.success).toBe(true)
    if (result.success) {
      expect(result.data.role).toBe("user") // default applied
    }
  })
})
```

### Testing Custom Hooks
```typescript
// tests/unit/hooks/use-counter.test.ts
import { describe, it, expect } from "vitest"
import { renderHook, act } from "@testing-library/react"
import { useCounter } from "@/hooks/use-counter"

describe("useCounter", () => {
  it("initializes with default value", () => {
    const { result } = renderHook(() => useCounter())
    expect(result.current.count).toBe(0)
  })

  it("initializes with provided value", () => {
    const { result } = renderHook(() => useCounter(10))
    expect(result.current.count).toBe(10)
  })

  it("increments count", () => {
    const { result } = renderHook(() => useCounter())
    act(() => result.current.increment())
    expect(result.current.count).toBe(1)
  })

  it("decrements count", () => {
    const { result } = renderHook(() => useCounter(5))
    act(() => result.current.decrement())
    expect(result.current.count).toBe(4)
  })

  it("does not go below minimum", () => {
    const { result } = renderHook(() => useCounter(0, { min: 0 }))
    act(() => result.current.decrement())
    expect(result.current.count).toBe(0)
  })

  it("resets to initial value", () => {
    const { result } = renderHook(() => useCounter(5))
    act(() => result.current.increment())
    act(() => result.current.reset())
    expect(result.current.count).toBe(5)
  })
})
```

---

## 🔗 INTEGRATION TESTS

### Testing API Routes
```typescript
// tests/integration/api/users.test.ts
import { describe, it, expect, beforeEach, afterEach, vi } from "vitest"
import { GET, POST } from "@/app/api/users/route"
import { db } from "@/lib/db"
import { users } from "@/lib/db/schema"

// Mock auth
vi.mock("@clerk/nextjs/server", () => ({
  auth: () => Promise.resolve({ userId: "test-clerk-id" }),
}))

describe("GET /api/users", () => {
  it("returns list of users", async () => {
    const req = new Request("http://localhost/api/users")
    const res = await GET(req)

    expect(res.status).toBe(200)
    const data = await res.json()
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it("returns 401 without auth", async () => {
    vi.mocked(auth).mockResolvedValueOnce({ userId: null } as any)
    const req = new Request("http://localhost/api/users")
    const res = await GET(req)
    expect(res.status).toBe(401)
  })
})

describe("POST /api/users", () => {
  it("creates a user with valid data", async () => {
    const req = new Request("http://localhost/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        name: "Test User",
        email: "test@example.com",
      }),
    })
    const res = await POST(req)

    expect(res.status).toBe(201)
    const data = await res.json()
    expect(data.data.email).toBe("test@example.com")
  })

  it("returns 422 with invalid data", async () => {
    const req = new Request("http://localhost/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name: "T", email: "not-email" }),
    })
    const res = await POST(req)
    expect(res.status).toBe(422)
  })
})
```

### Testing Server Actions
```typescript
// tests/integration/actions/update-profile.test.ts
import { describe, it, expect, vi } from "vitest"
import { updateProfile } from "@/actions/users"

vi.mock("@clerk/nextjs/server", () => ({
  auth: () => Promise.resolve({ userId: "clerk_test_123" }),
}))

vi.mock("next/cache", () => ({
  revalidatePath: vi.fn(),
}))

describe("updateProfile action", () => {
  it("updates user profile with valid data", async () => {
    const formData = new FormData()
    formData.set("name", "Updated Name")
    formData.set("bio", "New bio text")

    const result = await updateProfile({ success: false }, formData)
    expect(result.success).toBe(true)
  })

  it("returns error with invalid name", async () => {
    const formData = new FormData()
    formData.set("name", "X") // Too short

    const result = await updateProfile({ success: false }, formData)
    expect(result.success).toBe(false)
    expect(result.error).toBeDefined()
  })
})
```

### Testing DB Queries
```typescript
// tests/integration/db/users.test.ts
import { describe, it, expect, beforeEach } from "vitest"
import { db } from "@/lib/db"
import { users } from "@/lib/db/schema"
import { getUserById, createUser } from "@/lib/db/queries/users"
import { eq } from "drizzle-orm"

// Use a test transaction that rolls back
describe("User DB Queries", () => {
  let testUserId: string

  beforeEach(async () => {
    // Create test user
    const [user] = await db.insert(users).values({
      clerkId: `test_clerk_${Date.now()}`,
      email: `test_${Date.now()}@example.com`,
      name: "Test User",
    }).returning()
    testUserId = user.id
  })

  afterEach(async () => {
    // Cleanup
    await db.delete(users).where(eq(users.id, testUserId))
  })

  it("finds user by ID", async () => {
    const user = await getUserById(testUserId)
    expect(user).not.toBeNull()
    expect(user?.id).toBe(testUserId)
  })

  it("returns null for non-existent user", async () => {
    const user = await getUserById("non-existent-id")
    expect(user).toBeNull()
  })
})
```

---

## 🎭 COMPONENT TESTS

### Testing with React Testing Library
```typescript
// tests/unit/components/Button.test.tsx
import { describe, it, expect, vi } from "vitest"
import { render, screen } from "@testing-library/react"
import userEvent from "@testing-library/user-event"
import { Button } from "@/components/ui/button"

describe("Button component", () => {
  it("renders with correct text", () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole("button", { name: "Click me" })).toBeInTheDocument()
  })

  it("calls onClick when clicked", async () => {
    const user = userEvent.setup()
    const onClick = vi.fn()

    render(<Button onClick={onClick}>Click me</Button>)
    await user.click(screen.getByRole("button"))

    expect(onClick).toHaveBeenCalledTimes(1)
  })

  it("is disabled when disabled prop passed", () => {
    render(<Button disabled>Click me</Button>)
    expect(screen.getByRole("button")).toBeDisabled()
  })

  it("shows loading state", () => {
    render(<Button loading>Submit</Button>)
    // Loading indicator replaces text or shows alongside
    expect(screen.getByRole("button")).toHaveAttribute("aria-busy", "true")
  })

  it("does not call onClick when disabled", async () => {
    const user = userEvent.setup()
    const onClick = vi.fn()

    render(<Button disabled onClick={onClick}>Click me</Button>)
    await user.click(screen.getByRole("button"))

    expect(onClick).not.toHaveBeenCalled()
  })
})
```

### Testing Forms
```typescript
// tests/unit/components/LoginForm.test.tsx
import { describe, it, expect, vi } from "vitest"
import { render, screen, waitFor } from "@testing-library/react"
import userEvent from "@testing-library/user-event"
import { LoginForm } from "@/components/features/auth/login-form"

describe("LoginForm", () => {
  const user = userEvent.setup()

  it("renders email and password fields", () => {
    render(<LoginForm onSubmit={vi.fn()} />)
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument()
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument()
  })

  it("shows validation errors on empty submit", async () => {
    render(<LoginForm onSubmit={vi.fn()} />)
    await user.click(screen.getByRole("button", { name: /sign in/i }))

    await waitFor(() => {
      expect(screen.getByText(/email is required/i)).toBeInTheDocument()
    })
  })

  it("shows error for invalid email", async () => {
    render(<LoginForm onSubmit={vi.fn()} />)
    await user.type(screen.getByLabelText(/email/i), "not-an-email")
    await user.click(screen.getByRole("button", { name: /sign in/i }))

    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument()
    })
  })

  it("calls onSubmit with correct values", async () => {
    const onSubmit = vi.fn()
    render(<LoginForm onSubmit={onSubmit} />)

    await user.type(screen.getByLabelText(/email/i), "user@example.com")
    await user.type(screen.getByLabelText(/password/i), "Password123!")
    await user.click(screen.getByRole("button", { name: /sign in/i }))

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: "user@example.com",
        password: "Password123!",
      })
    })
  })
})
```

---

## 🌐 MSW — MOCK SERVICE WORKER

### Setup
```bash
pnpm add -D msw
npx msw init public/ --save
```

```typescript
// tests/mocks/handlers.ts
import { http, HttpResponse } from "msw"

export const handlers = [
  // Mock user API
  http.get("/api/users", () => {
    return HttpResponse.json({
      success: true,
      data: [
        { id: "1", name: "Alice", email: "alice@example.com" },
        { id: "2", name: "Bob", email: "bob@example.com" },
      ],
      meta: { total: 2 },
    })
  }),

  http.get("/api/users/:id", ({ params }) => {
    if (params.id === "not-found") {
      return HttpResponse.json(
        { success: false, error: { code: "NOT_FOUND", message: "User not found" } },
        { status: 404 }
      )
    }
    return HttpResponse.json({
      success: true,
      data: { id: params.id, name: "Test User", email: "test@example.com" },
    })
  }),

  http.post("/api/users", async ({ request }) => {
    const body = await request.json() as { name: string; email: string }
    return HttpResponse.json(
      {
        success: true,
        data: { id: "new-id", ...body, createdAt: new Date().toISOString() },
      },
      { status: 201 }
    )
  }),

  // Mock external APIs
  http.post("https://api.stripe.com/v1/payment_intents", () => {
    return HttpResponse.json({
      id: "pi_test_123",
      client_secret: "pi_test_123_secret",
      status: "requires_payment_method",
    })
  }),
]

// Error scenarios (use in specific tests)
export const errorHandlers = [
  http.get("/api/users", () => {
    return HttpResponse.json(
      { success: false, error: { code: "INTERNAL_ERROR", message: "Server error" } },
      { status: 500 }
    )
  }),
]
```

```typescript
// tests/mocks/server.ts
import { setupServer } from "msw/node"
import { handlers } from "./handlers"

export const server = setupServer(...handlers)
```

```typescript
// tests/setup.ts — add MSW setup
import { server } from "./mocks/server"
import { beforeAll, afterAll, afterEach } from "vitest"

beforeAll(() => server.listen({ onUnhandledRequest: "warn" }))
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

### Using MSW in Tests
```typescript
// tests/unit/hooks/use-users.test.tsx
import { describe, it, expect } from "vitest"
import { renderHook, waitFor } from "@testing-library/react"
import { server } from "../../mocks/server"
import { errorHandlers } from "../../mocks/handlers"
import { useUsers } from "@/hooks/use-users"
import { createWrapper } from "../../helpers/query-wrapper"

describe("useUsers", () => {
  it("fetches and returns users", async () => {
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper(),
    })

    await waitFor(() => expect(result.current.isSuccess).toBe(true))
    expect(result.current.data?.length).toBe(2)
    expect(result.current.data?.[0].name).toBe("Alice")
  })

  it("handles server errors", async () => {
    server.use(...errorHandlers)

    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper(),
    })

    await waitFor(() => expect(result.current.isError).toBe(true))
  })
})
```

```typescript
// tests/helpers/query-wrapper.tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query"
import { ReactNode } from "react"

export function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false, // Don't retry in tests
        gcTime: 0,
      },
    },
  })

  return function Wrapper({ children }: { children: ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    )
  }
}
```

---

## 🎭 E2E TESTS — PLAYWRIGHT

### Setup
```bash
pnpm add -D @playwright/test
npx playwright install --with-deps

# Or install only specific browsers
npx playwright install chromium
```

### Playwright Config
```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test"

export default defineConfig({
  testDir: "./tests/e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: process.env.CI ? "github" : "html",

  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
    video: "on-first-retry",
  },

  projects: [
    // Setup: authenticate and save state
    { name: "setup", testMatch: /.*\.setup\.ts/ },

    {
      name: "chromium",
      use: {
        ...devices["Desktop Chrome"],
        storageState: "tests/e2e/.auth/user.json",
      },
      dependencies: ["setup"],
    },
    {
      name: "mobile",
      use: {
        ...devices["iPhone 14"],
        storageState: "tests/e2e/.auth/user.json",
      },
      dependencies: ["setup"],
    },
  ],

  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
})
```

### Auth Setup (Reuse across tests)
```typescript
// tests/e2e/auth.setup.ts
import { test as setup, expect } from "@playwright/test"

const authFile = "tests/e2e/.auth/user.json"

setup("authenticate", async ({ page }) => {
  await page.goto("/sign-in")

  await page.getByLabel("Email").fill(process.env.TEST_USER_EMAIL!)
  await page.getByLabel("Password").fill(process.env.TEST_USER_PASSWORD!)
  await page.getByRole("button", { name: "Sign In" }).click()

  await expect(page).toHaveURL("/dashboard")

  // Save authentication state
  await page.context().storageState({ path: authFile })
})
```

### Critical Path Tests
```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from "@playwright/test"

test.describe("Authentication", () => {
  test.use({ storageState: { cookies: [], origins: [] } }) // No auth state

  test("user can sign up", async ({ page }) => {
    const email = `test_${Date.now()}@example.com`

    await page.goto("/sign-up")
    await page.getByLabel("Email").fill(email)
    await page.getByLabel("Password").fill("SecurePass123!")
    await page.getByLabel("Confirm Password").fill("SecurePass123!")
    await page.getByRole("button", { name: "Create Account" }).click()

    // Should redirect to onboarding or dashboard
    await expect(page).toHaveURL(/\/(onboarding|dashboard)/)
  })

  test("user can sign in", async ({ page }) => {
    await page.goto("/sign-in")
    await page.getByLabel("Email").fill(process.env.TEST_USER_EMAIL!)
    await page.getByLabel("Password").fill(process.env.TEST_USER_PASSWORD!)
    await page.getByRole("button", { name: "Sign In" }).click()

    await expect(page).toHaveURL("/dashboard")
    await expect(page.getByText("Welcome")).toBeVisible()
  })

  test("shows error for wrong credentials", async ({ page }) => {
    await page.goto("/sign-in")
    await page.getByLabel("Email").fill("wrong@example.com")
    await page.getByLabel("Password").fill("wrongpassword")
    await page.getByRole("button", { name: "Sign In" }).click()

    await expect(page.getByText(/invalid credentials/i)).toBeVisible()
    await expect(page).toHaveURL("/sign-in") // Stays on sign-in
  })

  test("redirects to sign-in when accessing protected route", async ({ page }) => {
    await page.goto("/dashboard")
    await expect(page).toHaveURL(/\/sign-in/)
  })
})
```

```typescript
// tests/e2e/dashboard.spec.ts
import { test, expect } from "@playwright/test"

test.describe("Dashboard", () => {
  test("displays user data after login", async ({ page }) => {
    await page.goto("/dashboard")

    await expect(page.getByRole("heading", { name: /dashboard/i })).toBeVisible()
    await expect(page.getByTestId("user-avatar")).toBeVisible()
  })

  test("navigation works correctly", async ({ page }) => {
    await page.goto("/dashboard")

    await page.getByRole("link", { name: "Settings" }).click()
    await expect(page).toHaveURL("/settings")

    await page.goBack()
    await expect(page).toHaveURL("/dashboard")
  })
})
```

### Page Object Pattern (For Complex Tests)
```typescript
// tests/e2e/page-objects/dashboard.page.ts
import { Page, Locator } from "@playwright/test"

export class DashboardPage {
  readonly page: Page
  readonly heading: Locator
  readonly statsCards: Locator
  readonly newProjectButton: Locator
  readonly searchInput: Locator

  constructor(page: Page) {
    this.page = page
    this.heading = page.getByRole("heading", { name: /dashboard/i })
    this.statsCards = page.getByTestId("stats-card")
    this.newProjectButton = page.getByRole("button", { name: "New Project" })
    this.searchInput = page.getByRole("searchbox")
  }

  async goto() {
    await this.page.goto("/dashboard")
    await this.heading.waitFor()
  }

  async createProject(name: string) {
    await this.newProjectButton.click()
    await this.page.getByLabel("Project Name").fill(name)
    await this.page.getByRole("button", { name: "Create" }).click()
  }

  async search(query: string) {
    await this.searchInput.fill(query)
    await this.page.keyboard.press("Enter")
  }
}

// Usage in test
test("creates a new project", async ({ page }) => {
  const dashboard = new DashboardPage(page)
  await dashboard.goto()
  await dashboard.createProject("My New Project")
  await expect(page.getByText("My New Project")).toBeVisible()
})
```

---

## 📊 TEST COVERAGE TARGETS

```
By Layer:
├── Utils / Pure functions:    95%+ (easy to test, no excuses)
├── Validators / Schemas:      90%+ (all validation paths)
├── Custom Hooks:              85%+ (all state transitions)
├── API Route Handlers:        80%+ (happy + error paths)
├── Server Actions:            80%+ (validation + execution)
├── React Components:          70%+ (user interactions)
└── E2E Critical Paths:        100% of critical user journeys

Critical Paths that MUST have E2E coverage:
□ Sign up flow
□ Sign in / Sign out flow
□ Core feature: [main action users do]
□ Payment flow (if applicable)
□ Settings update
```

---

## 🔧 TESTING UTILITIES

### Custom Render with Providers
```typescript
// tests/helpers/render.tsx
import { ReactNode } from "react"
import { render, RenderOptions } from "@testing-library/react"
import { QueryClient, QueryClientProvider } from "@tanstack/react-query"
import { ThemeProvider } from "next-themes"

function AllProviders({ children }: { children: ReactNode }) {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  })

  return (
    <QueryClientProvider client={queryClient}>
      <ThemeProvider attribute="class" defaultTheme="light">
        {children}
      </ThemeProvider>
    </QueryClientProvider>
  )
}

function customRender(ui: React.ReactElement, options?: Omit<RenderOptions, "wrapper">) {
  return render(ui, { wrapper: AllProviders, ...options })
}

export * from "@testing-library/react"
export { customRender as render }
```

### Database Test Helper
```typescript
// tests/helpers/db.ts
import { db } from "@/lib/db"
import { users } from "@/lib/db/schema"

export async function createTestUser(overrides = {}) {
  const [user] = await db.insert(users).values({
    clerkId: `test_clerk_${Math.random().toString(36).slice(2)}`,
    email: `test_${Math.random().toString(36).slice(2)}@example.com`,
    name: "Test User",
    ...overrides,
  }).returning()
  return user
}

export async function cleanupTestUsers(clerkIdPattern = "test_clerk_") {
  await db.delete(users).where(
    // Delete test users by pattern
    sql`${users.clerkId} LIKE ${clerkIdPattern + "%"}`
  )
}
```

# REY — Code Intelligence Reference

> Kode yang baik dibaca seperti prosa. Kode yang hebat dibaca seperti tidak perlu dibaca.

---

## 🧬 ADVANCED TYPESCRIPT PATTERNS

### Type-Safe Builder Pattern
```typescript
// Fluent API dengan type safety — setiap step enforce required fields
type Prettify<T> = { [K in keyof T]: T[K] } & {}

class QueryBuilder<T extends object = object> {
  private query: Partial<T> = {}
  
  select<K extends keyof T>(fields: K[]): QueryBuilder<Pick<T, K>> {
    return this as unknown as QueryBuilder<Pick<T, K>>
  }
  
  where(condition: Partial<T>): this {
    Object.assign(this.query, condition)
    return this
  }
  
  build(): Prettify<T> {
    return this.query as T
  }
}
```

### Branded Types (Prevent Primitive Obsession)
```typescript
// Prevent mixing IDs accidentally
type Brand<T, B extends string> = T & { readonly _brand: B }

type UserId = Brand<string, 'UserId'>
type OrderId = Brand<string, 'OrderId'>
type ProductId = Brand<string, 'ProductId'>

const createUserId = (id: string): UserId => id as UserId
const createOrderId = (id: string): OrderId => id as OrderId

// This now causes a compile error:
function getOrder(orderId: OrderId): Promise<Order> { ... }
const userId = createUserId('user_123')
getOrder(userId) // ❌ TypeScript Error: Type 'UserId' is not assignable to 'OrderId'
getOrder(createOrderId('order_456')) // ✅
```

### Discriminated Unions (Type-Safe State)
```typescript
// Model async state exhaustively
type AsyncState<T, E = Error> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: E }

function useAsync<T>(): AsyncState<T> {
  // ...
}

// Usage — TypeScript forces you to handle ALL cases
const state = useAsync<User>()
switch (state.status) {
  case 'idle':    return <Placeholder />
  case 'loading': return <Skeleton />
  case 'success': return <UserCard user={state.data} /> // data is User here
  case 'error':   return <ErrorMessage error={state.error} />
  // TypeScript will error if you miss a case
}
```

### Template Literal Types
```typescript
// Type-safe event system
type EventName = 'user' | 'order' | 'payment'
type EventAction = 'created' | 'updated' | 'deleted'
type EventKey = `${EventName}.${EventAction}` // "user.created" | "user.updated" | ... etc

type EventPayloads = {
  'user.created': { userId: string; email: string }
  'user.updated': { userId: string; changes: Partial<User> }
  'order.created': { orderId: string; customerId: string }
  // TypeScript enforces completeness
}

class TypedEventBus {
  on<K extends EventKey>(
    event: K,
    handler: (payload: EventPayloads[K]) => void
  ): void { ... }
  
  emit<K extends EventKey>(event: K, payload: EventPayloads[K]): void { ... }
}

const bus = new TypedEventBus()
bus.on('user.created', ({ userId, email }) => { // fully typed!
  console.log(userId, email)
})
```

### Conditional Types & Inference
```typescript
// Extract type from Promise
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T

// Make specific keys required
type RequireKeys<T, K extends keyof T> = T & Required<Pick<T, K>>

// Deep partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K]
}

// Readonly deep
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K]
}

// Extract route params from string
type ExtractRouteParams<T extends string> =
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<`/${Rest}`>
    : T extends `${infer _Start}:${infer Param}`
    ? Param
    : never

type Params = ExtractRouteParams<'/users/:userId/orders/:orderId'>
// type Params = "userId" | "orderId"
```

### Zod + TypeScript Integration (The Right Way)
```typescript
import { z } from 'zod'

// Schema-first — derive types FROM schema (single source of truth)
const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  role: z.enum(['user', 'admin']).default('user'),
  age: z.number().int().min(13).max(120).optional(),
})

// Derive input/output types
type CreateUserInput = z.input<typeof createUserSchema>   // Before transform
type CreateUserData = z.output<typeof createUserSchema>   // After transform/default

// Safe parse for API handlers
function validateInput<T>(schema: z.ZodSchema<T>, data: unknown): 
  | { success: true; data: T }
  | { success: false; errors: z.ZodError['errors'] } {
  const result = schema.safeParse(data)
  if (result.success) return { success: true, data: result.data }
  return { success: false, errors: result.error.errors }
}
```

---

## 🎨 DESIGN PATTERNS IN TYPESCRIPT

### Strategy Pattern (Replace if/else chains)
```typescript
// Instead of: if (paymentMethod === 'stripe') ... else if (paymentMethod === 'paypal')...
interface PaymentStrategy {
  charge(amount: number, currency: string): Promise<PaymentResult>
  refund(transactionId: string): Promise<void>
}

class StripePaymentStrategy implements PaymentStrategy {
  async charge(amount: number, currency: string): Promise<PaymentResult> {
    // Stripe-specific implementation
  }
  async refund(transactionId: string): Promise<void> { ... }
}

class PayPalPaymentStrategy implements PaymentStrategy {
  async charge(amount: number, currency: string): Promise<PaymentResult> {
    // PayPal-specific implementation
  }
  async refund(transactionId: string): Promise<void> { ... }
}

class PaymentProcessor {
  constructor(private readonly strategy: PaymentStrategy) {}
  
  async processPayment(amount: number, currency: string): Promise<PaymentResult> {
    return this.strategy.charge(amount, currency)
  }
}

// Adding new payment method = new class, no existing code changes (Open/Closed)
const processor = new PaymentProcessor(new StripePaymentStrategy())
```

### Observer Pattern (Event-driven)
```typescript
type EventHandler<T> = (event: T) => void | Promise<void>

class EventEmitter<Events extends Record<string, unknown>> {
  private handlers: Partial<{
    [K in keyof Events]: EventHandler<Events[K]>[]
  }> = {}
  
  on<K extends keyof Events>(event: K, handler: EventHandler<Events[K]>): () => void {
    if (!this.handlers[event]) this.handlers[event] = []
    this.handlers[event]!.push(handler)
    
    // Return unsubscribe function
    return () => {
      this.handlers[event] = this.handlers[event]!.filter(h => h !== handler)
    }
  }
  
  async emit<K extends keyof Events>(event: K, data: Events[K]): Promise<void> {
    const handlers = this.handlers[event] ?? []
    await Promise.all(handlers.map(handler => handler(data)))
  }
}
```

### Decorator Pattern (Cross-cutting concerns)
```typescript
// Logging decorator
function withLogging<T extends (...args: unknown[]) => Promise<unknown>>(
  fn: T,
  name: string
): T {
  return (async (...args) => {
    const start = Date.now()
    console.log(`[${name}] Starting...`, { args })
    try {
      const result = await fn(...args)
      console.log(`[${name}] Completed in ${Date.now() - start}ms`)
      return result
    } catch (error) {
      console.error(`[${name}] Failed after ${Date.now() - start}ms`, error)
      throw error
    }
  }) as T
}

// Retry decorator
function withRetry<T extends (...args: unknown[]) => Promise<unknown>>(
  fn: T,
  options: { maxRetries: number; delay: number }
): T {
  return (async (...args) => {
    let lastError: Error
    for (let i = 0; i <= options.maxRetries; i++) {
      try {
        return await fn(...args)
      } catch (error) {
        lastError = error as Error
        if (i < options.maxRetries) {
          await new Promise(r => setTimeout(r, options.delay * Math.pow(2, i))) // Exponential backoff
        }
      }
    }
    throw lastError!
  }) as T
}

// Usage
const processPaymentSafe = withRetry(
  withLogging(processPayment, 'processPayment'),
  { maxRetries: 3, delay: 1000 }
)
```

### Result Type (Error Handling without Exceptions)
```typescript
// Rust-inspired Result type — explicit error handling
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E }

const ok = <T>(value: T): Result<T, never> => ({ ok: true, value })
const err = <E>(error: E): Result<never, E> => ({ ok: false, error })

// Function that can fail — error is in the type signature
async function findUser(id: string): Promise<Result<User, 'NOT_FOUND' | 'DB_ERROR'>> {
  try {
    const user = await db.query.users.findFirst({ where: eq(users.id, id) })
    if (!user) return err('NOT_FOUND')
    return ok(user)
  } catch {
    return err('DB_ERROR')
  }
}

// Caller must handle both cases — no surprise exceptions
const result = await findUser('123')
if (!result.ok) {
  switch (result.error) {
    case 'NOT_FOUND': return Response.json({ error: 'User not found' }, { status: 404 })
    case 'DB_ERROR':  return Response.json({ error: 'Server error' }, { status: 500 })
  }
}
// Here result.value is User — TypeScript knows!
return Response.json(result.value)
```

---

## 🔧 REACT PATTERNS

### Compound Components
```tsx
// More flexible than prop-drilling
const Select = {
  Root: ({ children, onChange }: SelectRootProps) => (
    <SelectContext.Provider value={{ onChange }}>
      <div className="relative">{children}</div>
    </SelectContext.Provider>
  ),
  Trigger: ({ children }: { children: React.ReactNode }) => {
    const { isOpen, toggle } = useSelectContext()
    return (
      <button onClick={toggle} aria-expanded={isOpen}>
        {children}
      </button>
    )
  },
  Options: ({ children }: { children: React.ReactNode }) => {
    const { isOpen } = useSelectContext()
    return isOpen ? (
      <ul role="listbox" className="absolute top-full left-0 w-full">{children}</ul>
    ) : null
  },
  Option: ({ value, children }: OptionProps) => {
    const { onChange } = useSelectContext()
    return (
      <li role="option" onClick={() => onChange(value)}>{children}</li>
    )
  }
}

// Usage — flexible, composable
<Select.Root onChange={setCategory}>
  <Select.Trigger>Select category</Select.Trigger>
  <Select.Options>
    {categories.map(cat => (
      <Select.Option key={cat.id} value={cat.id}>{cat.name}</Select.Option>
    ))}
  </Select.Options>
</Select.Root>
```

### Custom Hook Patterns
```tsx
// useFetch with full state machine
function useFetch<T>(url: string) {
  const [state, setState] = useState<AsyncState<T>>({ status: 'idle' })
  
  const fetch = useCallback(async () => {
    setState({ status: 'loading' })
    try {
      const res = await window.fetch(url)
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      const data: T = await res.json()
      setState({ status: 'success', data })
    } catch (error) {
      setState({ status: 'error', error: error as Error })
    }
  }, [url])
  
  useEffect(() => { fetch() }, [fetch])
  
  return { ...state, refetch: fetch }
}

// useLocalStorage with SSR safety
function useLocalStorage<T>(key: string, initialValue: T) {
  const [stored, setStored] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch {
      return initialValue
    }
  })
  
  const setValue = useCallback((value: T | ((prev: T) => T)) => {
    setStored(prev => {
      const next = value instanceof Function ? value(prev) : value
      try {
        localStorage.setItem(key, JSON.stringify(next))
      } catch (error) {
        console.warn(`useLocalStorage: Could not save key "${key}"`, error)
      }
      return next
    })
  }, [key])
  
  return [stored, setValue] as const
}
```

### Server Action Patterns (Next.js App Router)
```typescript
// actions/users.ts
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'

const updateProfileSchema = z.object({
  name: z.string().min(2).max(100),
  bio: z.string().max(500).optional(),
})

// Server Action with type-safe form state
export async function updateProfile(
  prevState: { success: boolean; error?: string },
  formData: FormData
): Promise<{ success: boolean; error?: string }> {
  // Auth check
  const { userId } = await auth()
  if (!userId) return { success: false, error: 'Unauthorized' }
  
  // Validate
  const parsed = updateProfileSchema.safeParse({
    name: formData.get('name'),
    bio: formData.get('bio'),
  })
  if (!parsed.success) {
    return { success: false, error: parsed.error.errors[0].message }
  }
  
  // Update
  try {
    await db.update(users)
      .set(parsed.data)
      .where(eq(users.clerkId, userId))
    
    revalidatePath('/profile')
    return { success: true }
  } catch {
    return { success: false, error: 'Failed to update profile' }
  }
}
```

---

## 🏗️ REFACTORING PATTERNS

### Extract Method (Most Common)
```typescript
// ❌ Before — long function doing multiple things
async function processOrder(orderId: string): Promise<void> {
  const order = await db.select().from(orders).where(eq(orders.id, orderId)).limit(1)
  if (!order[0]) throw new Error('Order not found')
  
  let total = 0
  for (const item of order[0].items) {
    const product = await db.select().from(products).where(eq(products.id, item.productId)).limit(1)
    if (!product[0]) throw new Error('Product not found')
    total += product[0].price * item.quantity
  }
  
  // ... more logic
}

// ✅ After — each function does ONE thing
async function getOrderOrThrow(orderId: string): Promise<Order> {
  const result = await db.select().from(orders).where(eq(orders.id, orderId)).limit(1)
  if (!result[0]) throw new NotFoundError('Order', orderId)
  return result[0]
}

async function calculateOrderTotal(items: OrderItem[]): Promise<number> {
  const productIds = items.map(i => i.productId)
  const products = await db.select().from(productsTable).where(inArray(productsTable.id, productIds))
  const productMap = new Map(products.map(p => [p.id, p]))
  
  return items.reduce((total, item) => {
    const product = productMap.get(item.productId)
    if (!product) throw new NotFoundError('Product', item.productId)
    return total + product.price * item.quantity
  }, 0)
}

async function processOrder(orderId: string): Promise<void> {
  const order = await getOrderOrThrow(orderId)
  const total = await calculateOrderTotal(order.items)
  // ... rest of logic, readable and testable
}
```

### Replace Conditional with Polymorphism
```typescript
// ❌ Before — switch statement that keeps growing
function calculateShipping(order: Order, method: string): number {
  switch (method) {
    case 'standard': return order.weight * 2.5
    case 'express': return order.weight * 5.0 + 10
    case 'overnight': return order.weight * 8.0 + 25
    default: throw new Error('Unknown shipping method')
  }
}

// ✅ After — closed for modification, open for extension
interface ShippingStrategy {
  calculate(order: Order): number
  estimatedDays: number
  label: string
}

const shippingStrategies: Record<string, ShippingStrategy> = {
  standard: {
    calculate: (order) => order.weight * 2.5,
    estimatedDays: 5,
    label: 'Standard Shipping'
  },
  express: {
    calculate: (order) => order.weight * 5.0 + 10,
    estimatedDays: 2,
    label: 'Express Shipping'
  },
  overnight: {
    calculate: (order) => order.weight * 8.0 + 25,
    estimatedDays: 1,
    label: 'Overnight Shipping'
  }
}

function getShippingStrategy(method: string): ShippingStrategy {
  const strategy = shippingStrategies[method]
  if (!strategy) throw new Error(`Unknown shipping method: ${method}`)
  return strategy
}
```

---

## 📏 CODE QUALITY HEURISTICS

### The SOLID Rules (applied to TS/React)

```
S — Single Responsibility
    One function/class/component does ONE thing
    If you need "and" to describe it, split it

O — Open/Closed
    Open for extension, closed for modification
    Add new behavior via new code, not editing existing

L — Liskov Substitution
    Subtypes must be substitutable for their base types
    In TS: implementations must honor their interfaces

I — Interface Segregation
    Many specific interfaces > one general interface
    Don't force classes to implement methods they don't need

D — Dependency Inversion
    Depend on abstractions, not concretions
    Inject dependencies, don't hard-code them
```

### REY Code Smell Detector

```
🚨 God Object/Component — knows too much, does too much
🚨 Primitive Obsession — string/number where domain type should be
🚨 Long Parameter List — more than 3-4 params = object/config
🚨 Feature Envy — method uses other class's data more than its own
🚨 Shotgun Surgery — one change requires edits in many places
🚨 Data Clumps — same data grouped together everywhere (extract class)
🚨 Switch Statements — usually means polymorphism opportunity
🚨 Parallel Inheritance Hierarchies — adding class A always needs class B
🚨 Lazy Class — doesn't do enough to justify existence
🚨 Speculative Generality — "we might need this later" = YAGNI
```

### The Boy Scout Rule
> "Always leave the code cleaner than you found it."

Every PR should include at least one small cleanup — rename confusing variable, extract method, add type, improve comment. Not a refactor marathon. Just increment.

---

## ⚡ PERFORMANCE PATTERNS IN CODE

### Memoization
```typescript
// Simple memoize
function memoize<TArgs extends unknown[], TReturn>(
  fn: (...args: TArgs) => TReturn
): (...args: TArgs) => TReturn {
  const cache = new Map<string, TReturn>()
  
  return (...args: TArgs): TReturn => {
    const key = JSON.stringify(args)
    if (cache.has(key)) return cache.get(key)!
    
    const result = fn(...args)
    cache.set(key, result)
    return result
  }
}

// LRU Cache for bounded memory
class LRUCache<K, V> {
  private cache = new Map<K, V>()
  
  constructor(private readonly maxSize: number) {}
  
  get(key: K): V | undefined {
    if (!this.cache.has(key)) return undefined
    
    // Move to end (most recently used)
    const value = this.cache.get(key)!
    this.cache.delete(key)
    this.cache.set(key, value)
    return value
  }
  
  set(key: K, value: V): void {
    if (this.cache.has(key)) this.cache.delete(key)
    else if (this.cache.size >= this.maxSize) {
      // Delete least recently used (first item)
      this.cache.delete(this.cache.keys().next().value)
    }
    this.cache.set(key, value)
  }
}
```

### Lazy Loading & Code Splitting
```tsx
// Route-based (Next.js does this automatically)

// Component-based (manual)
const HeavyDataTable = dynamic(() => 
  import('./DataTable').then(mod => ({ default: mod.DataTable })),
  { 
    loading: () => <TableSkeleton rows={10} />,
    ssr: false  // Client-only components
  }
)

// On interaction
function DashboardPage() {
  const [showChart, setShowChart] = useState(false)
  const Chart = showChart 
    ? lazy(() => import('./Chart')) 
    : null
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>Load Chart</button>
      {Chart && <Suspense fallback={<ChartSkeleton />}><Chart /></Suspense>}
    </div>
  )
}
```

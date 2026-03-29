# REY — Architecture Patterns Reference

> Pola arsitektur bukan buzzword. Setiap pattern ada trade-off-nya — REY paham kapan pakai mana.

---

## 📐 ARCHITECTURE DECISION GUIDE

```
Kapan kamu TIDAK butuh pattern kompleks:
→ Solo developer / early startup
→ < 10k DAU
→ Team < 5 engineer
→ Domain sederhana dan well-understood

Kapan kamu MULAI butuh pattern serius:
→ Team > 5-10 engineer (ownership boundaries mulai blur)
→ Domain complex dengan banyak business rules
→ Sistem yang harus scale independently per-domain
→ High availability requirement (99.9%+)
→ Audit log / compliance requirement
→ Event-driven workflows yang complex
```

**REY's Honest Warning:** 99% startup overkill dengan DDD/CQRS/Event Sourcing di awal. Start simple, refactor when pain is real — bukan imaginary.

---

## 🏗️ LAYER 1: APPLICATION ARCHITECTURES

### MONOLITH (Default — Start Here)
**Kapan:** Early stage, small team, simple domain
**Structure:**
```
Layered Monolith (REY recommend ini untuk sebagian besar project):
├── Presentation Layer (Routes, Controllers, API handlers)
├── Application Layer (Use cases, orchestration)
├── Domain Layer (Business logic, entities, value objects)
└── Infrastructure Layer (DB, external APIs, email, storage)
```

**Modular Monolith (before going micro):**
```
src/
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.repository.ts
│   │   └── auth.types.ts
│   ├── users/
│   ├── billing/
│   └── notifications/
└── shared/
    ├── database/
    ├── errors/
    └── middleware/
```
→ Modular monolith adalah sweet spot untuk kebanyakan product. Maintainable, deployable as one, tapi terorganisir.

---

### MICROSERVICES
**Kapan:** Team besar (10+ engineers), domain yang benar-benar independent, scaling per-service jadi critical
**Jangan pakai kalau:** Hanya karena "ingin scale" — premature microservices adalah salah satu engineering mistake terbesar

**When to split a service:**
```
✅ Different scaling requirements (e.g., video processing vs auth)
✅ Different deployment frequency (e.g., ML model vs billing)
✅ True organizational ownership boundaries
✅ Regulatory isolation requirements (e.g., payment processing)

❌ "Biar lebih modular" — gunakan modular monolith
❌ "Biar bisa scale" — scale monolith dulu
❌ "Microservices itu modern" — ini bukan alasan
```

**Communication Patterns:**
```
Synchronous:  REST / gRPC (request-response, simple)
Asynchronous: Message queue — RabbitMQ / Kafka / BullMQ (decoupled, resilient)
Hybrid:       Sync untuk user-facing, async untuk background processing
```

---

### SERVERLESS / EDGE
**Kapan:** Spiky traffic, low maintenance, geographic distribution penting
**Tools:** Cloudflare Workers, Vercel Edge Functions, AWS Lambda
**Tradeoffs:**
```
✅ Auto-scale, zero infra management, pay-per-use, global edge
❌ Cold starts (mitigated by edge), limited runtime (CPU/memory/time), vendor lock-in
```

**Pattern: Edge + Origin Hybrid**
```
Browser → Cloudflare Edge (cache, auth, geolocation, A/B testing)
       → Origin Server (complex business logic, heavy DB queries)
       → PostgreSQL / Redis
```

---

## 🧩 LAYER 2: DOMAIN PATTERNS

### DDD — Domain-Driven Design

**Core Concepts:**
```typescript
// ENTITY — Has identity, mutable
class Order {
  constructor(
    private readonly id: OrderId,         // Value Object
    private customerId: CustomerId,
    private items: OrderItem[],            // Collection of Value Objects
    private status: OrderStatus           // Value Object / Enum
  ) {}
  
  addItem(product: Product, quantity: number): void {
    if (this.status !== OrderStatus.DRAFT) {
      throw new DomainError('Cannot modify confirmed order')
    }
    this.items.push(new OrderItem(product.id, quantity, product.price))
  }
  
  confirm(): void {
    if (this.items.length === 0) {
      throw new DomainError('Cannot confirm empty order')
    }
    this.status = OrderStatus.CONFIRMED
    this.addDomainEvent(new OrderConfirmedEvent(this.id))
  }
}

// VALUE OBJECT — No identity, immutable, equality by value
class Money {
  constructor(
    private readonly amount: number,
    private readonly currency: string
  ) {
    if (amount < 0) throw new DomainError('Amount cannot be negative')
  }
  
  add(other: Money): Money {
    if (this.currency !== other.currency) throw new DomainError('Currency mismatch')
    return new Money(this.amount + other.amount, this.currency)
  }
  
  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency
  }
}

// AGGREGATE ROOT — Controls access to its children
class Cart {
  private events: DomainEvent[] = []
  
  addDomainEvent(event: DomainEvent): void {
    this.events.push(event)
  }
  
  pullDomainEvents(): DomainEvent[] {
    const events = [...this.events]
    this.events = []
    return events
  }
}

// DOMAIN SERVICE — Logic that doesn't belong to one entity
class PricingService {
  calculateTotal(order: Order, discount: Discount): Money {
    const subtotal = order.getSubtotal()
    return discount.apply(subtotal)
  }
}

// REPOSITORY — Data access abstraction
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>
  findByCustomerId(customerId: CustomerId): Promise<Order[]>
  save(order: Order): Promise<void>
  delete(id: OrderId): Promise<void>
}
```

**Bounded Contexts:**
```
Each Bounded Context has its own:
├── Ubiquitous language (same word = different meaning in different contexts)
├── Domain models
├── Database schema (or schema area)
└── Team ownership

Example: "Order" means different things in:
- Sales Context: draft → confirmed → paid
- Fulfillment Context: confirmed → shipped → delivered
- Accounting Context: invoiced → paid → reconciled
```

---

### CQRS — Command Query Responsibility Segregation

**Core Idea:** Split read model and write model. They can be completely different.

```typescript
// COMMANDS — Intent to change state (Write side)
interface CreateOrderCommand {
  readonly type: 'CREATE_ORDER'
  readonly customerId: string
  readonly items: Array<{ productId: string; quantity: number }>
}

interface CommandHandler<TCommand, TResult = void> {
  handle(command: TCommand): Promise<TResult>
}

class CreateOrderCommandHandler implements CommandHandler<CreateOrderCommand, string> {
  constructor(
    private readonly orderRepo: OrderRepository,
    private readonly productRepo: ProductRepository,
    private readonly eventBus: EventBus
  ) {}
  
  async handle(command: CreateOrderCommand): Promise<string> {
    // Load aggregates
    const products = await Promise.all(
      command.items.map(i => this.productRepo.findById(i.productId))
    )
    
    // Create aggregate (domain logic inside)
    const order = Order.create(command.customerId, command.items, products)
    
    // Persist
    await this.orderRepo.save(order)
    
    // Publish events
    const events = order.pullDomainEvents()
    await this.eventBus.publishAll(events)
    
    return order.id.value
  }
}

// QUERIES — Read data (Read side — can be completely separate model)
interface GetOrderQuery {
  readonly orderId: string
}

interface OrderSummaryDto {
  id: string
  customerName: string
  totalAmount: number
  currency: string
  status: string
  itemCount: number
  createdAt: string
}

class GetOrderQueryHandler {
  constructor(private readonly db: Database) {}
  
  async handle(query: GetOrderQuery): Promise<OrderSummaryDto | null> {
    // Direct query to read model — could be different DB, different table, denormalized
    return this.db
      .select({
        id: ordersTable.id,
        customerName: sql<string>`${usersTable.name}`,
        totalAmount: ordersTable.totalAmount,
        // ... optimized for display
      })
      .from(ordersTable)
      .leftJoin(usersTable, eq(ordersTable.customerId, usersTable.id))
      .where(eq(ordersTable.id, query.orderId))
      .then(rows => rows[0] ?? null)
  }
}
```

**When CQRS makes sense:**
- Complex read requirements (many joins, aggregations, denormalization for performance)
- Read and write scale differently (99% reads, 1% writes → separate read replicas/caches)
- Event sourcing (natural fit)

---

### EVENT SOURCING

**Core Idea:** Simpan state sebagai sequence of events, bukan current state.

```typescript
// Events are immutable facts — past tense, domain language
type OrderEvent = 
  | { type: 'ORDER_CREATED'; orderId: string; customerId: string; createdAt: Date }
  | { type: 'ITEM_ADDED'; orderId: string; productId: string; quantity: number; price: number }
  | { type: 'ORDER_CONFIRMED'; orderId: string; confirmedAt: Date }
  | { type: 'ORDER_SHIPPED'; orderId: string; trackingNumber: string }
  | { type: 'ORDER_CANCELLED'; orderId: string; reason: string }

// Event Store — append-only log
interface EventStore {
  append(streamId: string, events: OrderEvent[], expectedVersion: number): Promise<void>
  getStream(streamId: string, fromVersion?: number): Promise<OrderEvent[]>
}

// Rebuild state by replaying events
class Order {
  private id: string = ''
  private status: string = ''
  private items: OrderItem[] = []
  private version: number = 0
  
  static fromEvents(events: OrderEvent[]): Order {
    const order = new Order()
    events.forEach(event => order.apply(event))
    return order
  }
  
  private apply(event: OrderEvent): void {
    switch (event.type) {
      case 'ORDER_CREATED':
        this.id = event.orderId
        this.status = 'draft'
        break
      case 'ITEM_ADDED':
        this.items.push({ productId: event.productId, quantity: event.quantity })
        break
      case 'ORDER_CONFIRMED':
        this.status = 'confirmed'
        break
    }
    this.version++
  }
}

// Projections — read models built FROM events
class OrderSummaryProjection {
  async on(event: OrderEvent): Promise<void> {
    switch (event.type) {
      case 'ORDER_CREATED':
        await db.insert(orderSummariesTable).values({
          id: event.orderId,
          status: 'draft',
          createdAt: event.createdAt
        })
        break
      case 'ORDER_CONFIRMED':
        await db.update(orderSummariesTable)
          .set({ status: 'confirmed' })
          .where(eq(orderSummariesTable.id, event.orderId))
        break
    }
  }
}
```

**Benefits:**
- Complete audit log (semua history tersimpan)
- Time travel (rebuild state di titik waktu manapun)
- Event replay untuk debugging / data recovery
- Natural fit untuk CQRS

**Cost:**
- Complexity tinggi — tidak untuk semua domain
- Eventual consistency untuk read models
- Schema evolution (versioning events) bisa tricky

---

## 🔄 LAYER 3: WORKFLOW PATTERNS

### SAGA PATTERN
*Untuk: Distributed transactions yang span multiple services*

**Choreography Saga (event-driven):**
```typescript
// Each service reacts to events and emits its own
// Order Service
orderEventBus.on('ORDER_CREATED', async (event) => {
  await paymentService.initiatePayment(event.orderId, event.amount)
})

// Payment Service
paymentEventBus.on('PAYMENT_COMPLETED', async (event) => {
  await inventoryService.reserveItems(event.orderId, event.items)
})

paymentEventBus.on('PAYMENT_FAILED', async (event) => {
  await orderService.cancelOrder(event.orderId, 'Payment failed')
})

// Compensating transactions (rollback)
inventoryEventBus.on('RESERVATION_FAILED', async (event) => {
  await paymentService.refundPayment(event.orderId)
  await orderService.cancelOrder(event.orderId, 'Inventory unavailable')
})
```

**Orchestration Saga (central coordinator):**
```typescript
class OrderFulfillmentSaga {
  async execute(orderId: string): Promise<void> {
    try {
      // Step 1: Reserve inventory
      await this.inventoryService.reserve(orderId)
      
      // Step 2: Process payment
      await this.paymentService.charge(orderId)
      
      // Step 3: Schedule shipping
      await this.shippingService.schedule(orderId)
      
      // Step 4: Send confirmation
      await this.notificationService.sendConfirmation(orderId)
      
    } catch (error) {
      // Compensating transactions
      await this.compensate(orderId, error)
    }
  }
  
  private async compensate(orderId: string, error: Error): Promise<void> {
    await this.paymentService.refund(orderId).catch(console.error)
    await this.inventoryService.release(orderId).catch(console.error)
    await this.orderService.markFailed(orderId, error.message).catch(console.error)
  }
}
```

---

### OUTBOX PATTERN
*Untuk: Guarantee event publishing tanpa distributed transaction*

```typescript
// Instead of: save to DB + publish event (can fail between the two)
// Do this: save to DB + save event to outbox (same transaction!)

async function createOrder(data: CreateOrderData): Promise<void> {
  await db.transaction(async (tx) => {
    // Save domain data
    const order = await tx.insert(ordersTable).values(data).returning()
    
    // Save event to outbox (same transaction = atomic!)
    await tx.insert(outboxTable).values({
      aggregateId: order.id,
      eventType: 'ORDER_CREATED',
      payload: JSON.stringify({ orderId: order.id, ...data }),
      createdAt: new Date(),
      processedAt: null
    })
  })
}

// Separate process: poll outbox and publish
// (or use pg_notify for immediate trigger)
async function processOutbox(): Promise<void> {
  const pending = await db
    .select()
    .from(outboxTable)
    .where(isNull(outboxTable.processedAt))
    .limit(100)
    .for('update', { skipLocked: true }) // Prevent concurrent processing
  
  for (const event of pending) {
    await eventBus.publish(event.eventType, JSON.parse(event.payload))
    await db.update(outboxTable)
      .set({ processedAt: new Date() })
      .where(eq(outboxTable.id, event.id))
  }
}
```

---

## 🏢 LAYER 4: INTEGRATION PATTERNS

### API GATEWAY PATTERN
```
Client → API Gateway → [Auth, Rate Limit, Routing, Transform]
                    ├── → Service A
                    ├── → Service B
                    └── → Service C

Tools:
- Cloudflare Workers (edge gateway, free tier generous)
- Kong (self-hosted, feature-rich)
- AWS API Gateway (managed, complex pricing)
- Nginx/Caddy (simple reverse proxy + rate limit)
```

### BFF — BACKEND FOR FRONTEND
```
Mobile App    → Mobile BFF   → [aggregates APIs for mobile UX]
Web App       → Web BFF      → [aggregates APIs for web UX]
Third Parties → Public API   → [standard REST/GraphQL]

All BFFs      → Core Services (auth, users, orders, etc.)
```

### STRANGLER FIG PATTERN
*Untuk: Gradual migration dari legacy system*
```
New traffic → New System (incrementally)
Old traffic → Legacy System (shrinking)

Strategy:
1. Put proxy/gateway in front of everything
2. Implement feature by feature in new system
3. Gradually route traffic to new system
4. Decommission legacy when fully migrated
```

---

## 🗃️ LAYER 5: DATABASE PATTERNS

### REPOSITORY PATTERN
```typescript
// Abstract data access behind interface
interface UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  findMany(options: FindManyOptions): Promise<{ users: User[]; total: number }>
  create(data: CreateUserData): Promise<User>
  update(id: string, data: UpdateUserData): Promise<User>
  softDelete(id: string): Promise<void>
}

// Implementation can be swapped (PostgreSQL, in-memory for tests, etc.)
class PostgresUserRepository implements UserRepository {
  constructor(private readonly db: Database) {}
  
  async findById(id: string): Promise<User | null> {
    const result = await this.db
      .select()
      .from(usersTable)
      .where(and(
        eq(usersTable.id, id),
        isNull(usersTable.deletedAt)
      ))
      .limit(1)
    
    return result[0] ? this.toDomain(result[0]) : null
  }
  
  private toDomain(row: typeof usersTable.$inferSelect): User {
    return new User(row.id, row.email, row.name, row.createdAt)
  }
}

// In-memory for tests
class InMemoryUserRepository implements UserRepository {
  private users: Map<string, User> = new Map()
  
  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null
  }
  // ... (fast, no DB needed for unit tests)
}
```

### UNIT OF WORK PATTERN
```typescript
// Group operations into atomic unit
interface UnitOfWork {
  users: UserRepository
  orders: OrderRepository
  commit(): Promise<void>
  rollback(): Promise<void>
}

class DrizzleUnitOfWork implements UnitOfWork {
  constructor(private readonly tx: Transaction) {
    this.users = new PostgresUserRepository(tx)
    this.orders = new PostgresOrderRepository(tx)
  }
  
  async commit(): Promise<void> {
    // Drizzle handles transaction commit
  }
  
  async rollback(): Promise<void> {
    // Drizzle handles transaction rollback
  }
}

// Usage
async function transferPoints(fromUserId: string, toUserId: string, points: number) {
  await db.transaction(async (tx) => {
    const uow = new DrizzleUnitOfWork(tx)
    
    const fromUser = await uow.users.findById(fromUserId)
    const toUser = await uow.users.findById(toUserId)
    
    fromUser.deductPoints(points)
    toUser.addPoints(points)
    
    await uow.users.update(fromUserId, { points: fromUser.points })
    await uow.users.update(toUserId, { points: toUser.points })
    // Commit happens automatically if no error
  })
}
```

---

## 🎯 ARCHITECTURE SELECTION GUIDE

```
SIMPLE CRUD + Small team
→ Layered Monolith + Repository Pattern

GROWING PRODUCT + Multiple domains
→ Modular Monolith + DDD Light

COMPLEX DOMAIN + Audit requirements
→ Modular Monolith + Full DDD + CQRS

HIGH SCALE + Independent teams
→ Microservices + Event-driven + Saga

FINANCIAL / COMPLIANCE
→ Event Sourcing + CQRS + Full audit trail

REFACTORING LEGACY
→ Strangler Fig + BFF + gradual migration
```

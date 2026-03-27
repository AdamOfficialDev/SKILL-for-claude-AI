# DAVID — Type Safety Reference (Scanner P)

Full TypeScript type safety pattern library. Load for .ts / .tsx files.

---

## The `any` Epidemic

### Detection Patterns

```typescript
// ALL of these are flagged by DAVID:

// 1. Direct any annotation
const data: any = response.json();
let result: any;
function process(input: any): any {}

// 2. Implicit any (no annotation + inference fails)
function transform(data) { return data.value; }  // param implicitly any

// 3. Type assertion to any (escape hatch)
const safe = (untrusted as any).dangerousMethod();

// 4. Suppression comments without explanation
// @ts-ignore
// @ts-expect-error

// 5. Object / {} as wide types (almost as bad as any)
function handle(event: object) {}   // Can't access any properties
function process(data: {}) {}       // Same problem
```

### Safe Alternatives

```typescript
// Instead of: const data: any = await fetch(...).then(r => r.json())
// ✅ Use zod for runtime + compile-time safety:
import { z } from 'zod';
const UserSchema = z.object({ id: z.number(), name: z.string() });
type User = z.infer<typeof UserSchema>;
const data = UserSchema.parse(await response.json());

// Instead of: function process(input: any)
// ✅ Use generics:
function process<T extends Record<string, unknown>>(input: T): T { return input; }

// Instead of: // @ts-ignore
// ✅ Use type guards:
function isUser(value: unknown): value is User {
  return typeof value === 'object' && value !== null && 'id' in value;
}
```

---

## Non-Null Assertion Overuse

```typescript
// ❌ Overuse — hiding null bugs
const name = user!.profile!.name!;
const el = document.querySelector('#app')!;
const item = list.find(x => x.id === id)!;

// ✅ Explicit handling
const name = user?.profile?.name ?? 'Anonymous';
const el = document.querySelector('#app');
if (!el) throw new Error('#app element not found');
const item = list.find(x => x.id === id);
if (!item) throw new Error(`Item ${id} not found`);
```

DAVID flag threshold: >3 non-null assertions in a single file = review needed.

---

## Missing Return Types on Exported Functions

```typescript
// ❌ Return type inferred — brittle, any change silently changes the type
export function getUser(id: number) {
  return db.users.find(u => u.id === id);  // Returns User | undefined — but nobody knows
}

// ✅ Explicit return type — contract enforced
export function getUser(id: number): User | undefined {
  return db.users.find(u => u.id === id);
}

// ✅ Async functions need explicit return type too
export async function createOrder(data: CreateOrderDto): Promise<Order> {
  return db.orders.create(data);
}
```

---

## Unsafe JSON.parse and API Responses

```typescript
// ❌ Parsed JSON used directly — completely untyped
const config = JSON.parse(fs.readFileSync('config.json', 'utf8'));
config.database.host;  // any — runtime error if missing

// ❌ API response cast without validation
const user = response.data as User;  // Trusts API to match type — dangerous

// ✅ Validate with zod
const ConfigSchema = z.object({
  database: z.object({
    host: z.string(),
    port: z.number(),
  })
});
const config = ConfigSchema.parse(JSON.parse(fs.readFileSync('config.json', 'utf8')));

// ✅ Or use a type guard
function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' && data !== null &&
    typeof (data as User).id === 'number' &&
    typeof (data as User).name === 'string'
  );
}
if (!isUser(response.data)) throw new Error('Invalid user data from API');
```

---

## Generic Opportunities Missed

```typescript
// ❌ Duplicated functions that differ only by type
function getStringItem(list: string[], id: number): string | undefined {
  return list[id];
}
function getNumberItem(list: number[], id: number): number | undefined {
  return list[id];
}

// ✅ Generic function
function getItem<T>(list: T[], id: number): T | undefined {
  return list[id];
}

// ❌ API helper without generic
async function fetchData(url: string): Promise<any> {
  const res = await fetch(url);
  return res.json();
}

// ✅ Generic API helper
async function fetchData<T>(url: string, schema: z.ZodSchema<T>): Promise<T> {
  const res = await fetch(url);
  return schema.parse(await res.json());
}
```

---

## Discriminated Union Opportunities

```typescript
// ❌ Boolean flags to simulate state — leads to impossible states
interface LoadState {
  isLoading: boolean;
  data: User | null;
  error: Error | null;
  // Problem: isLoading=true + data=User + error=Error is representable but impossible
}

// ✅ Discriminated union — only valid states representable
type LoadState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User }
  | { status: 'error'; error: Error };

// TypeScript now enforces correct access:
if (state.status === 'success') {
  console.log(state.data.name);  // ✅ TypeScript knows data exists here
}
```

---

## Type Guard Patterns

```typescript
// ✅ User-defined type guard
function isError(value: unknown): value is Error {
  return value instanceof Error;
}

// ✅ Narrowing in catch blocks
try {
  await riskyOperation();
} catch (error) {
  if (error instanceof NetworkError) {
    handleNetworkError(error);  // error narrowed to NetworkError
  } else if (isError(error)) {
    logger.error(error.message);  // error narrowed to Error
  } else {
    logger.error('Unknown error', error);
  }
}

// ❌ Common mistake — error is 'unknown' in TS 4+, not Error
} catch (error) {
  console.log(error.message);  // TypeScript error — error is unknown
```

---

## Strict TypeScript Config Checklist

DAVID checks `tsconfig.json` for these settings:

```json
{
  "compilerOptions": {
    "strict": true,              // Enables all strict checks — MUST HAVE
    "noUncheckedIndexedAccess": true,  // arr[0] returns T | undefined, not T
    "exactOptionalPropertyTypes": true, // { x?: string } ≠ { x: string | undefined }
    "noImplicitReturns": true,   // All code paths must return
    "noFallthroughCasesInSwitch": true, // No accidental switch fallthrough
    "forceConsistentCasingInFileNames": true  // Mac/Linux case sensitivity
  }
}
```

If `"strict": false` or `strict` missing → flag as 🟠 HIGH.
If `"any"` appears in compiler options as `suppressImplicitAnyErrors: true` → flag as 🔴 CRITICAL.

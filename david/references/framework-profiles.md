# DAVID — Framework Profile Reference

Full framework profile definitions for Phase 1.5 (Framework Profile Activation).
Load this file when DAVID needs to apply framework-specific scanner adjustments.

## Table of Contents
1. [How Profiles Work](#how-profiles-work)
2. [Next.js App Router](#nextjs-app-router)
3. [Next.js Pages Router](#nextjs-pages-router)
4. [React + Vite (SPA)](#react-vite-spa)
5. [FastAPI (Python)](#fastapi-python)
6. [Express / NestJS (Node.js)](#express-nestjs-nodejs)
7. [Flutter / Dart](#flutter-dart)
8. [Django (Python)](#django-python)
9. [Generic / Unknown Stack](#generic-unknown-stack)
10. [Profile Output in Fingerprint](#profile-output-in-fingerprint)

---

## How Profiles Work

After Code Fingerprint (Phase 1.1), DAVID loads the matching profile. Each profile defines:
- Which scanners are **prioritized** (run first, report prominently)
- **Special rules** unique to that framework
- Common **vibe code traps** seen in that stack
- Which scanners to **skip or adjust**

Detection method: import statements, file structure, config files (`next.config.js`, `vite.config.ts`,
`pyproject.toml`, etc.), and dependency names in `package.json` / `pubspec.yaml` / `requirements.txt`.

---

## Next.js App Router

**Activated when:** `next.config.*` exists, OR folder `app/` contains `page.tsx`/`layout.tsx` pattern.

```
NEXT.JS APP ROUTER — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : SEO (T), Bundle (V), TypeScript (P), PERF (C), A11Y (F)

⚠️  SPECIAL RULES:
  - Server Component vs Client Component boundary — 'use client' placement audit
  - fetch() caching strategy — default cache, revalidate, no-store audit
  - Metadata API — generateMetadata() vs static metadata object
  - Server Actions — 'use server' placement, input validation, CSRF protection
  - Image: next/image mandatory; width/height or fill prop must be present
  - Font: next/font mandatory for font loading — not <link> from Google Fonts
  - Route Handler vs API Routes — don't mix paradigms
  - Dynamic imports with { ssr: false } for client-only libraries
  - Streaming & Suspense boundaries — loading.tsx pattern

⚠️  COMMON VIBE CODE TRAPS:
  - 'use client' in root layout → entire app becomes client-rendered
  - async Server Component calling client-only hook (useState)
  - Data fetching in Client Component when Server Component would work
  - Missing error.tsx or not-found.tsx in route segments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### App Router Specific Patterns

```typescript
// ❌ Incorrect: 'use client' at root layout kills SSR for entire tree
// app/layout.tsx
'use client';  // ← This forces entire app to client-render
export default function RootLayout({ children }) { ... }

// ✅ Correct: only add 'use client' to components that need it
// app/components/Counter.tsx
'use client';
export default function Counter() { ... }
// app/layout.tsx has NO 'use client' directive

// ❌ Calling useState in a Server Component (will crash)
// app/page.tsx (Server Component by default)
import { useState } from 'react';  // ← Can't use in Server Component
export default function Page() {
  const [count, setCount] = useState(0);  // ← Runtime error
}

// ✅ Correct: fetch in Server Component, pass data to Client Component
// app/page.tsx
async function Page() {
  const data = await fetch('/api/data').then(r => r.json());
  return <ClientChart data={data} />;  // Client Component receives data as prop
}

// ❌ Missing route segment error handling
// app/dashboard/page.tsx — no error.tsx sibling
// When this throws, Next.js shows a generic error

// ✅ Always add error.tsx and loading.tsx to important route segments
// app/dashboard/error.tsx
'use client';
export default function Error({ error, reset }) {
  return <div><h2>Something went wrong</h2><button onClick={reset}>Try again</button></div>;
}

// ❌ fetch() with no cache config — defaults to full cache (often wrong for dynamic data)
const data = await fetch('https://api.example.com/users');

// ✅ Explicit cache strategy
const data = await fetch('https://api.example.com/users', {
  next: { revalidate: 60 }  // Revalidate every 60 seconds (ISR)
});
// OR
const data = await fetch('https://api.example.com/users', {
  cache: 'no-store'  // Always fresh (SSR)
});
```

---

## Next.js Pages Router

**Activated when:** `pages/` folder exists, `_app.tsx`/`_document.tsx` detected.

```
NEXT.JS PAGES ROUTER — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : SEO (T), Bundle (V), TypeScript (P)

⚠️  SPECIAL RULES:
  - getStaticProps vs getServerSideProps — flag SSR when SSG is sufficient (PERF)
  - getStaticPaths — fallback: blocking vs true vs false trade-offs
  - _app.tsx bloat — global imports that should be per-page
  - API routes: /pages/api/ — auth middleware pattern check
  - next/head vs Metadata API — don't mix paradigms between Pages and App Router
  - Image: next/image, never plain <img>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Pages Router Specific Patterns

```typescript
// ❌ getServerSideProps when data doesn't change per-request
export async function getServerSideProps() {
  const posts = await fetchAllBlogPosts();  // Same data for everyone
  return { props: { posts } };
  // SSR = full server render on every request — expensive
}

// ✅ getStaticProps with revalidation (ISR) for near-static data
export async function getStaticProps() {
  const posts = await fetchAllBlogPosts();
  return { props: { posts }, revalidate: 3600 };  // Rebuild at most once per hour
}

// ❌ _app.tsx importing heavy libraries globally
// pages/_app.tsx
import 'react-big-calendar/lib/css/react-big-calendar.css';  // 200KB loaded on every page
import HeavyAnalytics from 'heavy-analytics';

// ✅ Dynamic import where actually needed
// pages/calendar.tsx
const BigCalendar = dynamic(() => import('react-big-calendar'), { ssr: false });
```

---

## React + Vite (SPA)

**Activated when:** `vite.config.*` exists; no SSR framework detected.

```
REACT + VITE SPA — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : Bundle (V), State (W), TypeScript (P), UX (O)

⚠️  SPECIAL RULES:
  - Bundle splitting — React.lazy() + Suspense for route-level splitting
  - Vite aliases — path alias in vite.config must sync with tsconfig paths
  - Environment variables — VITE_ prefix mandatory for client-exposed vars
  - HMR-unfriendly patterns — module-level side effects that break hot reload
  - react-router-dom v6 patterns — createBrowserRouter vs BrowserRouter
  - SEO scanner (T) — SKIP or flag as "SPA: SEO limited without SSR/prerendering"

⚠️  VITE TRAP: process.env is not available in Vite — use import.meta.env
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Vite Specific Patterns

```typescript
// ❌ process.env in Vite (Node.js style — won't work in browser)
const API_URL = process.env.REACT_APP_API_URL;

// ✅ Vite environment variables
const API_URL = import.meta.env.VITE_API_URL;
// In .env: VITE_API_URL=https://api.example.com
// Only VITE_ prefixed vars are exposed to the client bundle

// ❌ No code splitting — entire app in one bundle
import Dashboard from './pages/Dashboard';
import Analytics from './pages/Analytics';
import Reports from './pages/Reports';

// ✅ Route-level lazy loading
const Dashboard  = lazy(() => import('./pages/Dashboard'));
const Analytics  = lazy(() => import('./pages/Analytics'));
const Reports    = lazy(() => import('./pages/Reports'));
// Each route chunk loads only when navigated to
```

---

## FastAPI (Python)

**Activated when:** `fastapi` in imports, `uvicorn` in deps, or `@app.get`/`@app.post` patterns.

```
FASTAPI — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : Security (B), GDPR (AE), Resilience (AF), DB Schema (SQ)

⚠️  SPECIAL RULES:
  - Pydantic models — all request/response must be typed with BaseModel
  - Dependency Injection — FastAPI Depends() for auth, DB session
  - async def vs def — sync route in async server = thread pool blocking
  - Background Tasks vs Celery — flag when each is appropriate
  - HTTPException vs custom exception handlers
  - CORS middleware — origins list; never wildcard in production
  - Alembic migrations — every model change must have a migration
  - SQLAlchemy async session — session per-request via Depends()
  - Pydantic v1 vs v2 — breaking changes if mixed

⚠️  COMMON VIBE CODE TRAPS:
  - Global db session (not per-request) → thread safety issue
  - Missing response_model → API leaks fields it shouldn't
  - sync def with blocking I/O in async app → bottleneck
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### FastAPI Specific Patterns

```python
# ❌ Global DB session — not thread-safe
db = SessionLocal()  # Module-level — shared across all requests

# ✅ Per-request session via Depends()
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/{user_id}")
async def get_user(user_id: int, db: Session = Depends(get_db)):
    return db.query(User).filter(User.id == user_id).first()

# ❌ Missing response_model — leaks password hash, internal IDs, etc.
@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return db.query(User).first()  # Returns ALL fields including hashed_password

# ✅ response_model controls what's exposed
class UserPublic(BaseModel):
    id: int
    email: str
    name: str

@app.get("/users/{user_id}", response_model=UserPublic)
async def get_user(user_id: int):
    return db.query(User).first()  # Only UserPublic fields sent to client

# ❌ Blocking I/O in async handler — blocks the event loop
@app.get("/data")
async def get_data():
    return requests.get("https://api.example.com/data").json()  # Blocking!

# ✅ Use async HTTP client
import httpx
@app.get("/data")
async def get_data():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
    return response.json()
```

---

## Express / NestJS (Node.js)

**Activated when:** `express` or `@nestjs/core` in deps.

```
EXPRESS / NESTJS — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : Security (B), Resilience (AF), TypeScript (P), API Contract (AA)

⚠️  SPECIAL RULES (Express):
  - Middleware order — auth before route handlers; error handler MUST be last
  - express-async-errors or manual try/catch — async errors not auto-caught
  - helmet, cors, rate-limit — three of these are mandatory in production
  - Body size limit — express.json({ limit: '10mb' }) — flag if unlimited
  - SQL via string concatenation → SQLi — must use parameterized query

⚠️  SPECIAL RULES (NestJS):
  - Guards vs Middleware vs Interceptors — flag if used in wrong context
  - Circular dependency between modules — NestJS DI doesn't handle this gracefully
  - forwardRef() — must have a comment explaining why it's needed
  - DTO validation — class-validator mandatory; no manual validation
  - Exception filters — global filter must exist for unhandled errors
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Express Specific Patterns

```javascript
// ❌ Async error not caught — Express won't handle rejected promises
app.get('/data', async (req, res) => {
  const data = await fetchData();  // If this throws, Express hangs or crashes
  res.json(data);
});

// ✅ Option A: Manual try/catch
app.get('/data', async (req, res, next) => {
  try {
    const data = await fetchData();
    res.json(data);
  } catch (err) {
    next(err);  // Pass to error middleware
  }
});

// ✅ Option B: express-async-errors (auto-wraps all async routes)
require('express-async-errors');
app.get('/data', async (req, res) => {
  const data = await fetchData();  // Errors auto-forwarded to error middleware
  res.json(data);
});

// ❌ Error handler not last — won't catch errors from routes defined after it
app.use((err, req, res, next) => { res.status(500).json({ error: err.message }); });
app.get('/route-after-handler', ...);  // This route's errors won't be caught

// ✅ Error handler always last
app.get('/all-routes', ...);
app.use((err, req, res, next) => { res.status(500).json({ error: err.message }); });
```

---

## Flutter / Dart

**Activated when:** `pubspec.yaml` exists, `.dart` files present, `flutter` in dependencies.

```
FLUTTER — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : UX/Mobile (O6), Performance (C), State (W), A11Y (F)

⚠️  SPECIAL RULES:
  - setState() in StatefulWidget — flag if StatelessWidget would work
  - BuildContext across async gaps — context.mounted check mandatory
  - Widget rebuild scope — flag fat build() that rebuilds too much
  - const constructor — mandatory on all widgets with no mutable state
  - State management: Provider vs Riverpod vs Bloc — flag mixing paradigms
  - Platform-specific code — .platform == check vs dart:io Platform
  - Image caching — CachedNetworkImage, not Image.network directly
  - Memory leaks — AnimationController, StreamSubscription must be disposed()
  - Dart null safety — sound null safety mandatory; late keyword needs justification

⚠️  SCANNERS THAT SKIP:
  - SEO (T) — not relevant for mobile app
  - Bundle (V) — replaced with APK/IPA size check
  - PWA (X) — not relevant
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Flutter Specific Patterns

```dart
// ❌ BuildContext used after async gap without mounted check
void onButtonPressed() async {
  final result = await someAsyncOperation();
  Navigator.of(context).push(...);  // context may be invalid if widget disposed
}

// ✅ Check mounted before using context after await
void onButtonPressed() async {
  final result = await someAsyncOperation();
  if (!mounted) return;  // Widget may have been disposed during await
  Navigator.of(context).push(...);

// ❌ AnimationController not disposed — memory leak
class _MyWidgetState extends State<MyWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, duration: const Duration(seconds: 1));
  }
  // No dispose() → memory leak
}

// ✅ Always dispose controllers
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

// ❌ Missing const — widget rebuilds unnecessarily
return Padding(
  padding: EdgeInsets.all(16.0),  // New EdgeInsets object on every build
  child: Text('Hello'),
);

// ✅ const constructors — widget cached, no rebuild
return const Padding(
  padding: EdgeInsets.all(16.0),
  child: Text('Hello'),
);
```

---

## Django (Python)

**Activated when:** `django` in deps, `settings.py`, `urls.py`, `models.py` pattern detected.

```
DJANGO — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : Security (B), DB Schema (SQ), GDPR (AE), Migration (J)

⚠️  SPECIAL RULES:
  - DEBUG=True detection — must be False in production settings
  - SECRET_KEY — must not be hardcoded; must come from env
  - ALLOWED_HOSTS — ['*'] in production is a critical security issue
  - ORM N+1 — select_related() and prefetch_related() audit
  - Migration squashing — too many migration files = squash needed
  - Custom User model — must be set up from project start, not mid-project
  - Django REST Framework — serializer validation, permission_classes mandatory
  - CSRF — csrf_exempt only with explicit documented justification
  - Django admin — don't expose at production URL without strong auth
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Django Specific Patterns

```python
# ❌ Hardcoded SECRET_KEY
SECRET_KEY = 'django-insecure-abc123...'

# ✅ From environment
import os
SECRET_KEY = os.environ['DJANGO_SECRET_KEY']

# ❌ DEBUG=True in production
DEBUG = True
ALLOWED_HOSTS = ['*']  # + wildcard = full attack surface exposed

# ✅ Environment-aware settings
DEBUG = os.environ.get('DJANGO_DEBUG', 'False') == 'True'
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# ❌ N+1: separate query per related object
for order in Order.objects.filter(status='pending'):
    print(order.user.email)  # N extra queries — one per order

# ✅ select_related eliminates N+1 for ForeignKey
for order in Order.objects.filter(status='pending').select_related('user'):
    print(order.user.email)  # Single JOIN query

# ❌ Custom user model added mid-project — requires painful migration
# (All auth tables need rebuilding)

# ✅ Always set custom user model at project start
# myapp/models.py
class User(AbstractUser):
    pass

# settings.py
AUTH_USER_MODEL = 'myapp.User'
```

---

## Generic / Unknown Stack

**Activated when:** Framework not detected, or stack not in profile list.

```
GENERIC / UNKNOWN STACK — Scanner Adjustments
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRIORITIZED  : Security (B), Bug (A), Code Review (D), Error Coverage (R)

⚠️  BEHAVIOR:
  - All scanners run with default weights — no framework-specific boosts
  - DAVID explicitly flags in the fingerprint: "Framework not identified"
  - Coverage may be suboptimal compared to a matched profile
  - Framework-specific traps (e.g., Next.js server/client boundary) are NOT checked
  - DAVID proceeds with documented assumption: generic best practices apply
  - At session end: prompt user to share dependency/config file for better accuracy

⚠️  SCANNER ADJUSTMENTS FOR UNKNOWN STACK:
  - SEO (T): SKIP unless HTML meta tags detected (cannot infer SSR capability)
  - Bundle (V): Run only if webpack.config.* or vite.config.* is present
  - PWA (X): SKIP unless service-worker.* or manifest.json detected
  - State (W): Run only if Redux/Zustand/Context import patterns detected
  - CICD (K): Run only if Dockerfile or *.yml workflow files provided

⚠️  COMMON UNKNOWN STACK SCENARIOS:
  - Legacy codebase without package.json → apply generic JS/TS rules
  - Polyglot repo (Python + JS + Go) → run language-specific scanners per file
  - Internal framework / proprietary stack → note explicitly, apply nearest analog
  - Config-only submission (YAML/JSON only) → run CICD (K) + ENVCONF (Y) only
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Generic Profile Fingerprint Output

```
📦 DAVID v6.0 CODE FINGERPRINT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
...
Framework Profile : Generic / Unknown ← LOADED
Profile Rules     : Default weights, no framework-specific rules applied
Coverage Note     : ⚠️ For more accurate results, share: package.json /
                    pubspec.yaml / pyproject.toml / requirements.txt
Scanners boosted  : None
Scanners adjusted : SEO (T) — skipped; Bundle (V) — conditional;
                    PWA (X) — skipped; State (W) — conditional
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Unknown Framework — What DAVID Outputs to User

When framework cannot be identified, DAVID includes this block before the main findings:

```
🤖 DAVID ASSUMPTION: Framework not identified from available files.
   Reason    : No package.json / pubspec.yaml / pyproject.toml detected
   Confidence: 40% (generic rules may miss framework-specific issues)
   If wrong  : Share your dependency file and I'll re-run with the correct profile
   Impact    : Framework-specific scanner adjustments not applied (SEO, Bundle, PWA, State)
```

---

## Profile Output in Fingerprint

After framework detection, DAVID appends to the Code Fingerprint block:

```
📦 DAVID v6.0 CODE FINGERPRINT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
...
Framework Profile : Next.js App Router ← LOADED
Profile Rules     : Server/Client boundary, Metadata API, next/image, fetch caching
Scanners boosted  : T (SEO), V (Bundle), P (TypeScript), F (A11Y)
Scanners adjusted : [list any that are skipped or modified]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

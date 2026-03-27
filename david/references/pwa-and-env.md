# DAVID — PWA & Environment Config Reference

## PWA / Offline Reference (Scanner X)

---

### Service Worker Cache Strategies

| Strategy | Use For | Behavior |
|----------|---------|----------|
| Cache First | Static assets (JS, CSS, fonts, images) | Serve from cache, update in background |
| Network First | API calls, dynamic data | Try network, fall back to cache on failure |
| Stale While Revalidate | Pages, semi-dynamic content | Serve cache instantly, update cache from network |
| Network Only | POST/PUT/DELETE requests | Never cache write operations |
| Cache Only | Precached critical assets | Always serve from cache |

```javascript
// ✅ Workbox strategy implementation
import { registerRoute } from 'workbox-routing';
import { CacheFirst, NetworkFirst, StaleWhileRevalidate } from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';

// Static assets — Cache First
registerRoute(
  ({ request }) => request.destination === 'script'
    || request.destination === 'style'
    || request.destination === 'font',
  new CacheFirst({
    cacheName: 'static-assets',
    plugins: [
      new ExpirationPlugin({ maxEntries: 60, maxAgeSeconds: 30 * 24 * 60 * 60 }), // 30 days
    ],
  })
);

// API calls — Network First with 3s timeout
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    networkTimeoutSeconds: 3,
    plugins: [
      new ExpirationPlugin({ maxEntries: 50, maxAgeSeconds: 5 * 60 }), // 5 mins
    ],
  })
);

// Pages — Stale While Revalidate
registerRoute(
  ({ request }) => request.mode === 'navigate',
  new StaleWhileRevalidate({ cacheName: 'pages' })
);

// ❌ DAVID flags: caching POST/PUT/DELETE requests
// ❌ DAVID flags: no cache size limit (ExpirationPlugin missing)
// ❌ DAVID flags: no offline fallback page
```

### Offline Fallback Page

```javascript
// ✅ Register offline fallback
import { setCatchHandler } from 'workbox-routing';
import { matchPrecache } from 'workbox-precaching';

// Precache the offline page at build time
precacheAndRoute([{ url: '/offline.html', revision: '1' }]);

// Serve offline page when navigation fails
setCatchHandler(async ({ event }) => {
  if (event.request.destination === 'document') {
    return matchPrecache('/offline.html');
  }
  return Response.error();
});
```

### manifest.json Checklist

```json
{
  "name": "Full App Name (max 45 chars)",
  "short_name": "ShortName",
  "description": "Brief description for app stores",
  "start_url": "/?source=pwa",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#ffffff",
  "theme_color": "#3b82f6",
  "lang": "en",
  "icons": [
    { "src": "/icons/icon-72x72.png",   "sizes": "72x72",   "type": "image/png" },
    { "src": "/icons/icon-96x96.png",   "sizes": "96x96",   "type": "image/png" },
    { "src": "/icons/icon-128x128.png", "sizes": "128x128", "type": "image/png" },
    { "src": "/icons/icon-144x144.png", "sizes": "144x144", "type": "image/png" },
    { "src": "/icons/icon-152x152.png", "sizes": "152x152", "type": "image/png" },
    { "src": "/icons/icon-192x192.png", "sizes": "192x192", "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-384x384.png", "sizes": "384x384", "type": "image/png" },
    { "src": "/icons/icon-512x512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/maskable-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ],
  "screenshots": [
    { "src": "/screenshots/desktop.png", "sizes": "1280x720", "form_factor": "wide" },
    { "src": "/screenshots/mobile.png",  "sizes": "390x844",  "form_factor": "narrow" }
  ]
}
```

DAVID flags missing: 192px icon, 512px icon, maskable icon, start_url, display.

---

## Environment Config Reference (Scanner Y)

### Zod Env Validation Pattern

```typescript
// ✅ Validate all env vars at startup — fail fast with clear errors
import { z } from 'zod';

const envSchema = z.object({
  // Required strings
  DATABASE_URL:    z.string().url('DATABASE_URL must be a valid URL'),
  JWT_SECRET:      z.string().min(32, 'JWT_SECRET must be at least 32 characters'),
  
  // Required with options
  NODE_ENV:        z.enum(['development', 'test', 'production']),
  LOG_LEVEL:       z.enum(['error', 'warn', 'info', 'debug']).default('info'),
  
  // Numbers (coerce from string)
  PORT:            z.coerce.number().min(1).max(65535).default(3000),
  DB_POOL_SIZE:    z.coerce.number().min(1).max(100).default(10),
  
  // Optional
  REDIS_URL:       z.string().url().optional(),
  SENTRY_DSN:      z.string().url().optional(),
  
  // Booleans
  ENABLE_METRICS:  z.string().transform(v => v === 'true').default('false'),
});

// Parse at app startup — throws immediately if invalid
export const env = envSchema.parse(process.env);

// Usage: env.DATABASE_URL (fully typed, never undefined)
```

### .env.example Sync Check

DAVID compares these and flags drift:

```bash
# .env.example (should document ALL vars)
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
JWT_SECRET=your-secret-here-minimum-32-chars
PORT=3000
NODE_ENV=development
REDIS_URL=redis://localhost:6379  # Optional - for caching

# .env (actual, gitignored)
DATABASE_URL=postgresql://prod-user:prod-pass@prod-host:5432/proddb
JWT_SECRET=actual-production-secret-key-here
PORT=8080
NODE_ENV=production
STRIPE_SECRET_KEY=sk_live_xxx  # ❌ Not in .env.example — new dev won't know!
```

DAVID flags:
- Keys in `.env` not in `.env.example` → documentation gap
- Keys in `.env.example` not read anywhere in code → stale docs
- Secrets in `.env.example` (should have placeholder, not real value)

### Next.js Client Bundle Exposure

```javascript
// ❌ CRITICAL: Server secret accessible in browser
// In any component or shared code:
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
// If this file is imported by a client component, the key ships to the browser!

// ✅ Server-only enforcement
// In server-only files (pages/api/*, server components):
import 'server-only';  // Next.js: throws if imported in client code
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

// ✅ Public config uses NEXT_PUBLIC_ prefix (explicitly public)
const publishableKey = process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY;
// This is fine — publishable keys are meant to be public
```

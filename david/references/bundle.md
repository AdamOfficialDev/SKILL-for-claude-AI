# DAVID — Bundle Size Reference (Scanner V)

Patterns for detecting and fixing bundle bloat. Load for Webpack/Vite/Next.js projects.

---

## Heavy Package Detection

### npm Package Weight Reference

| Package | Gzipped Size | DAVID Action |
|---------|-------------|-------------|
| `moment` | 72KB | 🔴 Replace with `dayjs` (3KB) or `date-fns` (tree-shakeable) |
| `lodash` (full) | 25KB | 🟠 Use `lodash-es` + named imports, or native JS |
| `lodash-es` (full import) | 25KB | 🟠 Use named imports only |
| `jquery` | 30KB | 🟠 Use native DOM APIs |
| `antd` (full) | 350KB+ | 🔴 Use deep imports: `antd/lib/button` |
| `@mui/material` (full) | 300KB+ | 🔴 Use deep imports |
| `react-icons` (full package) | varies | 🟠 Import from specific icon set |
| `axios` | 5KB | ✅ OK — reasonable size |
| `date-fns` (named imports) | 2–10KB | ✅ OK — tree-shakeable |
| `zod` | 8KB | ✅ OK |
| `uuid` | 0.5KB | ✅ OK |

### Problematic Import Patterns

```javascript
// ❌ Imports entire lodash library
import _ from 'lodash';
import * as _ from 'lodash';
const { debounce } = require('lodash');  // Still pulls full lib in CJS

// ✅ Named import (requires lodash-es for tree-shaking)
import { debounce } from 'lodash-es';
// Or direct path import (works with CJS lodash)
import debounce from 'lodash/debounce';

// ❌ Full icon library
import { FiHome, FiUser } from 'react-icons/fi';  // Pulls all Feather icons

// ✅ Direct import (same API, smaller bundle)
import FiHome from 'react-icons/fi/FiHome';
import FiUser from 'react-icons/fi/FiUser';

// ❌ Full date library
import moment from 'moment';

// ✅ dayjs (same API, 97% smaller)
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
dayjs.extend(relativeTime);
```

---

## Code Splitting Patterns

### React Lazy Loading

```javascript
// ❌ All pages in main bundle — slow initial load
import { Dashboard } from './pages/Dashboard';
import { Analytics } from './pages/Analytics';
import { Settings } from './pages/Settings';

// ✅ Code-split — each page loads only when navigated to
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Settings  = lazy(() => import('./pages/Settings'));

// ✅ With Suspense + meaningful loading state
<Suspense fallback={<PageSkeleton />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/analytics" element={<Analytics />} />
  </Routes>
</Suspense>
```

### Next.js Dynamic Imports

```javascript
// ❌ Heavy chart library in main bundle
import { ComplexChart } from './ComplexChart';

// ✅ Dynamic import — loads only when component mounts
import dynamic from 'next/dynamic';
const ComplexChart = dynamic(() => import('./ComplexChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false,  // Client-only libraries (e.g. chart.js)
});

// ✅ Conditional heavy features
const RichTextEditor = dynamic(() => import('./RichTextEditor'), {
  ssr: false,
  loading: () => <EditorSkeleton />,
});
// Only render (and load) when user needs it:
{isEditing && <RichTextEditor />}
```

---

## Next.js-Specific Bundle Issues

```javascript
// ❌ Using getServerSideProps when getStaticProps would work
// getServerSideProps: builds HTML on EVERY request (slow, expensive)
// getStaticProps: builds HTML ONCE at build time (fast, cached)

export async function getServerSideProps() {  // ❌ For static content
  const posts = await getBlogPosts();  // Same data every time
  return { props: { posts } };
}

// ✅ Static content should use getStaticProps
export async function getStaticProps() {
  const posts = await getBlogPosts();
  return { props: { posts }, revalidate: 3600 };  // ISR: regenerate every hour
}

// ❌ Importing server-only modules in client components
// In a React component file:
import { db } from '../lib/db';  // Database client in browser bundle!

// ✅ Use server-only package to catch this at build time
import 'server-only';  // Throws if imported in client component
import { db } from '../lib/db';
```

---

## Bundle Analysis Commands

When DAVID suggests investigating bundle size, these commands help:

```bash
# Next.js — visual bundle analyzer
npm install --save-dev @next/bundle-analyzer
# Add to next.config.js:
# const withBundleAnalyzer = require('@next/bundle-analyzer')({ enabled: true });
# module.exports = withBundleAnalyzer({});
ANALYZE=true npm run build

# Vite — rollup visualizer
npm install --save-dev rollup-plugin-visualizer
# Add to vite.config.ts plugins: visualizer({ open: true })
npm run build

# Generic webpack
npm install --save-dev webpack-bundle-analyzer
```

---

## Image Optimization

```jsx
// ❌ Unoptimized images — LCP killer
<img src="/hero.jpg" alt="Hero" />

// ✅ Next.js Image — auto-resizes, WebP, lazy-loads
import Image from 'next/image';
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority  // Only for above-the-fold images (LCP)
  quality={85}  // Balance quality vs size
/>

// ✅ Responsive srcset for plain HTML
<img
  src="/hero-800.jpg"
  srcset="/hero-400.jpg 400w, /hero-800.jpg 800w, /hero-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 900px) 800px, 1200px"
  alt="Hero"
  loading="lazy"  // Omit for above-fold images
  decoding="async"
>
```

# DAVID — UX Enhancement Reference: Core (E1–E6)

Core enhancement sub-systems for Scanner EN.
Load for: anti-hallucination rules, scoring system, loading states, empty states, microinteractions, form UX, navigation, visual hierarchy.

## Table of Contents
1. [Anti-Hallucination Rules](#-anti-hallucination-rules--wajib-dipatuhi-sebelum-semua-sub-enhancer) ← READ FIRST
   - Rule 0 — FILE GATE (Hard Gate) ← **NEW**
   - Rule 1 — Prove Before You Propose
   - Rule 2 — Search Before Suggesting
   - Rule 3 — Never Assume From Component Name
   - Rule 4 — "Not Found" Is Explicit Output
   - Rule 5 — Partial Read = Partial Scope
   - Rule 6 — Pre-Scan Feature Inventory ← **NEW**
   - Rule 7 — "Already Exists" Section Wajib ← **NEW**
2. [Enhancement Scoring System](#enhancement-scoring-system)
3. [E1 — Loading State Upgrader](#e1--loading-state-upgrader)
4. [E2 — Empty State Upgrader](#e2--empty-state-upgrader)
5. [E3 — Microinteraction Injector](#e3--microinteraction-injector)
6. [E4 — Form UX Upgrader](#e4--form-ux-upgrader)
7. [E5 — Navigation Upgrader](#e5--navigation-upgrader)
8. [E6 — Visual Hierarchy Booster](#e6--visual-hierarchy-booster)

---

# DAVID — UX Enhancement Reference

Full pattern library for Scanner EN (🚀 ENHANCE). Load when user asks to enhance, upgrade, or improve UI/UX — not just fix what's broken, but elevate what's already working.

**The core difference:**
- AUDIT mode → "This loading state is missing — here's the fix" (fix the broken)
- ENHANCE mode → "This loading state exists, but here's how to make it feel premium" (elevate the good)

---

## ⚠️ ANTI-HALLUCINATION RULES — WAJIB DIPATUHI SEBELUM SEMUA SUB-ENHANCER

> Pelanggaran rules ini = output yang tidak valid. Jangan skip sekali pun.

---

### Rule 0 — FILE GATE (HARD GATE — RUNS FIRST, BEFORE ANYTHING ELSE)

**Scanner EN TIDAK BOLEH menghasilkan satu pun rekomendasi sebelum Rule 0 selesai dieksekusi.**

Sebelum melakukan scan apapun, DAVID **wajib** melakukan langkah berikut secara berurutan:

#### Step 0A — Verifikasi File Diterima
Jika user meminta enhance UI/UX tetapi **tidak melampirkan kode**, DAVID **wajib** menolak dan meminta file dulu:

```
🚫 EN GATE BLOCKED — Tidak ada kode yang diterima.

Scanner EN tidak dapat berjalan tanpa membaca implementasi aktual.
Asumsi dari nama komponen atau deskripsi = hallucination.

Lampirkan file atau paste kode yang ingin di-enhance.
EN scan akan dimulai segera setelah kode diterima.
```

DILARANG melanjutkan ke saran apapun sebelum kode tersedia.

#### Step 0B — File Receipt Confirmation Banner
Setelah kode diterima, DAVID **wajib** menampilkan banner konfirmasi sebelum scan dimulai:

```
📂 EN FILE GATE — CONFIRMED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files received   : [daftar nama file / komponen]
Lines read       : [baris N–M dari total X per file]
Read coverage    : [FULL / PARTIAL — jika partial, deklarasikan batas]
Scope lock       : EN scan HANYA valid untuk baris yang sudah dibaca
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Memulai Pre-Scan Feature Inventory...
```

#### Step 0C — Jalankan Pre-Scan Feature Inventory (lihat Rule 6)
Setelah banner dikonfirmasi, langsung jalankan Rule 6 (Pre-Scan Feature Inventory) sebelum menghasilkan rekomendasi apapun.

---

### Rule 1 — PROVE BEFORE YOU PROPOSE
Setiap rekomendasi HARUS disertai field `Evidence` berisi nomor baris aktual dari kode yang disubmit. Evidence membuktikan:
- (a) Fitur belum ada → tuliskan keyword apa yang dicari dan tidak ditemukan
- (b) Fitur ada tapi bisa di-upgrade → tuliskan di baris berapa ditemukannya dan level saat ini

```
✅ VALID Evidence:
   Evidence: Line 27: `if (loading) return <Spinner />` — CONFIRMED spinner only.
             Search: skeleton, Skeleton, animate-pulse, shimmer → NOT FOUND di seluruh file.

❌ INVALID (tidak ada evidence):
   Current Level: Basic (spinner only)   ← David mengasumsikan, tidak membaca
```

### Rule 2 — SEARCH BEFORE SUGGESTING
Sebelum suggest fitur X, DAVID **wajib** search keyword berikut di seluruh file:

| Akan suggest | Keyword yang harus dicari dulu |
|---|---|
| Skeleton / shimmer | `skeleton`, `Skeleton`, `animate-pulse`, `shimmer`, `LoadingSkeleton` |
| Empty state | `EmptyState`, `empty-state`, `"no data"`, `"no results"`, `trigger=` |
| Hover/microinteraction | `transition`, `animate`, `scale(`, `:hover`, `:active`, `transform` |
| Floating label | `floating`, `field-label`, `placeholder-shown`, `labelFloat` |
| Password strength | `strength`, `calculateStrength`, `passwordStrength` |
| Bottom sheet | `BottomSheet`, `bottom-sheet`, `translate-y-full`, `safe-area` |
| Dark mode polish | `dark:`, `.dark {`, `darkMode`, `prefers-color-scheme` |
| Optimistic UI | `optimistic`, `setOptimistic`, `rollback`, `prefetch` |
| Skip links | `SkipLinks`, `skip-link`, `#main-content`, `sr-only` |
| AnimatedNumber | `AnimatedNumber`, `animateValue`, `requestAnimationFrame`, `toLocaleString` |

Kalau keyword ditemukan → jangan suggest dari nol. Suggest upgrade ke level berikutnya dan sebutkan level saat ini.

### Rule 3 — NEVER ASSUME FROM COMPONENT NAME
Nama `<UserCard>`, `<Dashboard>`, `<Button>` bukan bukti tidak ada hover state, loading state, atau animasi. DAVID harus membaca **implementasi** komponen, bukan menebak dari namanya.

### Rule 4 — "NOT FOUND" IS EXPLICIT OUTPUT
Kalau setelah scan tidak menemukan implementasi suatu fitur, output **eksplisit**:
```
Search dilakukan: skeleton, Skeleton, animate-pulse, shimmer — NOT FOUND di seluruh file (1–247 baris)
```
Bukan diam lalu langsung propose.

### Rule 5 — PARTIAL READ = PARTIAL SCOPE
Kalau file terlalu panjang dan tidak semua baris terbaca, DAVID **wajib** deklarasikan:
```
⚠️ File dibaca: baris 1–300 dari 580 total.
   EN scan hanya valid untuk baris yang sudah dibaca.
   Untuk scan lengkap: minta user submit ulang atau split file.
```
DILARANG output finding untuk bagian yang belum dibaca.

---

### Rule 6 — PRE-SCAN FEATURE INVENTORY (WAJIB SEBELUM REKOMENDASI)

Setelah FILE GATE (Rule 0) dikonfirmasi, DAVID **wajib** menjalankan inventory scan menyeluruh terhadap seluruh kode yang diterima. Tujuan: mengetahui fitur apa yang **sudah ada**, sehingga tidak ada saran duplikat atau saran untuk fitur yang sudah diimplementasi.

#### Cara kerja:
DAVID scan seluruh file yang diterima dan mengisi tabel berikut **berdasarkan bukti nyata dari kode**, bukan asumsi:

```
🔍 PRE-SCAN FEATURE INVENTORY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files scanned : [nama file]   Lines scanned : [range]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

LOADING STATES
  ☑ Spinner          : [FOUND line N] / [NOT FOUND]
  ☑ Skeleton loader  : [FOUND line N — animate-pulse] / [NOT FOUND]
  ☑ Shimmer          : [FOUND line N — shimmer keyframe] / [NOT FOUND]
  ☑ Staggered reveal : [FOUND line N] / [NOT FOUND]
  ☑ Progress bar     : [FOUND line N] / [NOT FOUND]

EMPTY STATES
  ☑ Empty text       : [FOUND line N — "No data found"] / [NOT FOUND]
  ☑ Empty icon       : [FOUND line N] / [NOT FOUND]
  ☑ Empty CTA button : [FOUND line N] / [NOT FOUND]
  ☑ Illustrated empty: [FOUND line N] / [NOT FOUND]

MICROINTERACTIONS
  ☑ Hover transition : [FOUND line N — transition-colors] / [NOT FOUND]
  ☑ Press/active     : [FOUND line N — :active scale()] / [NOT FOUND]
  ☑ Success animation: [FOUND line N] / [NOT FOUND]
  ☑ Ripple effect    : [FOUND line N] / [NOT FOUND]

FORM UX
  ☑ Inline validation: [FOUND line N] / [NOT FOUND]
  ☑ Floating label   : [FOUND line N] / [NOT FOUND]
  ☑ Password strength: [FOUND line N — calculateStrength] / [NOT FOUND]
  ☑ Character counter: [FOUND line N] / [NOT FOUND]
  ☑ Error message    : [FOUND line N] / [NOT FOUND]

NAVIGATION
  ☑ Active state     : [FOUND line N — aria-current] / [NOT FOUND]
  ☑ Page transition  : [FOUND line N — fade animation] / [NOT FOUND]
  ☑ Breadcrumb       : [FOUND line N] / [NOT FOUND]
  ☑ Animated collapse: [FOUND line N] / [NOT FOUND]

VISUAL HIERARCHY
  ☑ Typography scale : [FOUND — h1/h2/h3 defined] / [FLAT — all same size]
  ☑ Card elevation   : [FOUND line N — shadow-sm/md/lg] / [NOT FOUND]
  ☑ Badge/status     : [FOUND line N] / [NOT FOUND]
  ☑ Color contrast   : [FOUND — dark: variants present] / [NOT FOUND]

FEEDBACK & COMMUNICATION
  ☑ Toast/notif      : [FOUND line N] / [NOT FOUND]
  ☑ Optimistic UI    : [FOUND line N — setOptimistic] / [NOT FOUND]
  ☑ Error boundary   : [FOUND line N] / [NOT FOUND]

MOBILE / RESPONSIVE
  ☑ Responsive layout: [FOUND — md: lg: breakpoints] / [NOT FOUND]
  ☑ Bottom sheet     : [FOUND line N] / [NOT FOUND]
  ☑ Touch target     : [FOUND — min-h-[44px] / py-3] / [UNVERIFIED]
  ☑ Safe area inset  : [FOUND line N — safe-area] / [NOT FOUND]

DARK MODE
  ☑ Dark variants    : [FOUND — dark: classes present] / [NOT FOUND]
  ☑ Elevation system : [FOUND — dark:bg-gray-800/900] / [NOT FOUND]

PERFORMANCE PERCEPTION
  ☑ Debounce/throttle: [FOUND line N] / [NOT FOUND]
  ☑ Lazy load        : [FOUND line N — Suspense/lazy] / [NOT FOUND]
  ☑ Prefetch         : [FOUND line N] / [NOT FOUND]

ACCESSIBILITY
  ☑ aria-label       : [FOUND — N instances] / [NOT FOUND]
  ☑ Skip links       : [FOUND line N — sr-only] / [NOT FOUND]
  ☑ Focus ring       : [FOUND — focus-visible:ring] / [NOT FOUND]
  ☑ Keyboard nav     : [FOUND — onKeyDown] / [NOT FOUND]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ ALREADY EXISTS  : [N items — akan di-skip atau di-upgrade saja]
🚀 MISSING / UPGRADEABLE : [N items — kandidat enhancement]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Aturan setelah inventory:**
- Item **FOUND** = jangan suggest dari nol. Jika masih bisa di-upgrade, tandai sebagai `[UPGRADE]` bukan `[MISSING]`
- Item **NOT FOUND** = boleh suggest implementasi baru, wajib tandai sebagai `[NEW]`
- Item yang sudah di level Premium = tandai `[SKIP — ALREADY OPTIMAL]` dan jangan masuk ke rekomendasi

---

### Rule 7 — "ALREADY EXISTS" SECTION WAJIB ADA DI OUTPUT

Setiap output Enhancement Session **wajib** menyertakan section `✅ ALREADY EXISTS` sebelum section rekomendasi. Format:

```
✅ ALREADY EXISTS — TIDAK AKAN DI-PROPOSE ULANG
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fitur berikut sudah diimplementasi di kode kamu.
DAVID skip dari rekomendasi baru — hanya akan di-upgrade jika belum optimal.

  ✓ Spinner loading      — line 27 (basic, upgrade tersedia → skeleton)
  ✓ Hover transition     — line 84 (transition-colors, 150ms)
  ✓ Error message form   — line 156 (inline, sudah functional)
  ✓ Dark mode variants   — line 203+ (dark: class hadir di semua komponen)
  ✓ Responsive layout    — line 12 (md: lg: breakpoints terdefinisi)
  ✓ aria-label           — line 67, 89, 134 (3 instance ditemukan)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[N] fitur di-skip · [N] bisa di-upgrade · [N] belum ada → kandidat baru
```

**CRITICAL:** Jika section ini kosong (tidak ada fitur yang ditemukan sama sekali), DAVID **wajib** menulis:
```
✅ ALREADY EXISTS — TIDAK ADA FITUR YANG DITEMUKAN
Kode yang diterima belum memiliki implementasi UI enhancement yang terdeteksi.
Semua rekomendasi di bawah bersifat [NEW].
```

Bukan diam — selalu deklarasikan hasil scan secara eksplisit.

---

---

## Table of Contents
1. [Enhancement Scoring System](#enhancement-scoring-system)
2. [E1 — Loading State Upgrader](#e1--loading-state-upgrader)
3. [E2 — Empty State Upgrader](#e2--empty-state-upgrader)
4. [E3 — Microinteraction Injector](#e3--microinteraction-injector)
5. [E4 — Form UX Upgrader](#e4--form-ux-upgrader)
6. [E5 — Navigation Upgrader](#e5--navigation-upgrader)
7. [E6 — Visual Hierarchy Booster](#e6--visual-hierarchy-booster)
8. [E7 — Feedback & Communication Upgrader](#e7--feedback--communication-upgrader)
9. [E8 — Mobile Experience Enhancer](#e8--mobile-experience-enhancer)
10. [E9 — Dark Mode Polish](#e9--dark-mode-polish)
11. [E10 — Performance Perception Enhancer](#e10--performance-perception-enhancer)
12. [E11 — Accessibility Delight Layer](#e11--accessibility-delight-layer)
13. [E12 — Data Display Enhancer](#e12--data-display-enhancer)
14. [Enhancement Delivery Format](#enhancement-delivery-format)

---

## Enhancement Scoring System

Before proposing enhancements, DAVID evaluates the current UX maturity.

> ⚠️ This section runs AFTER Rule 0 (File Gate), Rule 6 (Pre-Scan Inventory), and Rule 7 (Already Exists section). The banner below is the consolidated output — it includes inventory results, what already exists, and what will be proposed.

```
📂 EN FILE GATE — CONFIRMED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files received   : [daftar nama file]
Lines read       : [baris N–M dari total X]
Read coverage    : [FULL / PARTIAL]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ ALREADY EXISTS — TIDAK AKAN DI-PROPOSE ULANG
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓ [Fitur ditemukan]     — line [N] ([status: optimal / upgrade tersedia])
  ✓ [Fitur ditemukan]     — line [N] ([status])
  ... (semua hasil FOUND dari Pre-Scan Inventory)
[N] fitur di-skip · [N] bisa di-upgrade · [N] belum ada
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🚀 UX ENHANCEMENT SCAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Component       : [component name]
Current Level   : [Basic / Functional / Good / Polished / Premium]

Enhancement Opportunities (hanya item MISSING atau UPGRADE):
  E1 Loading States   : [label — UPGRADE: spinner → skeleton] / [SKIP — already skeleton]
  E2 Empty States     : [label — NEW: belum ada] / [SKIP — already illustrated]
  E3 Microinteractions: [label — NEW: belum ada press feedback] / [SKIP — optimal]
  E4 Form UX          : [label — UPGRADE: inline validation ada, floating label belum]
  E5 Navigation       : [label — NEW: active state belum ada]
  E6 Visual Hierarchy : [label — UPGRADE: flat → scale defined]
  E7 Feedback         : [label — NEW / UPGRADE / SKIP]
  E8 Mobile           : [label — NEW / UPGRADE / SKIP]
  E9 Dark Mode        : [label — NEW / UPGRADE / SKIP]
  E10 Perf Perception : [label — NEW / UPGRADE / SKIP]

UX Maturity Score (before) : [N]/100
Estimated Score (after)    : [N]/100
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Maturity levels:**
| Level | Score | Description |
|-------|-------|-------------|
| Basic | 0–30 | Functional but raw — no states, no feedback |
| Functional | 31–50 | Works correctly — minimal polish |
| Good | 51–70 | All states present — lacks delight |
| Polished | 71–85 | Smooth experience — minor gaps |
| Premium | 86–100 | Delightful — every detail considered |

---

## E1 — Loading State Upgrader

### Level 1 → Level 2: Replace spinner with skeleton

```jsx
// 📊 Current (Basic — Level 1)
function UserList() {
  if (loading) return <div className="flex justify-center py-8"><Spinner /></div>;
  return <ul>{users.map(u => <UserCard key={u.id} user={u} />)}</ul>;
}

// 🚀 DAVID ENHANCE → Level 2: Skeleton that mirrors real content
function UserListSkeleton() {
  return (
    <ul className="divide-y">
      {[...Array(5)].map((_, i) => (
        <li key={i} className="flex items-center gap-3 py-4 animate-pulse">
          <div className="w-10 h-10 rounded-full bg-gray-200 dark:bg-gray-700 flex-shrink-0" />
          <div className="flex-1">
            <div className="h-4 bg-gray-200 dark:bg-gray-700 rounded w-36 mb-2" />
            <div className="h-3 bg-gray-200 dark:bg-gray-700 rounded w-52" />
          </div>
          <div className="h-8 bg-gray-200 dark:bg-gray-700 rounded w-20" />
        </li>
      ))}
    </ul>
  );
}
// Result: Layout stable during load, no visual jump, 40% faster perceived
```

### Level 2 → Level 3: Add shimmer animation

```css
/* 🚀 DAVID ENHANCE → Level 3: Shimmer instead of pulse */
@keyframes shimmer {
  0%   { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

.skeleton-shimmer {
  background: linear-gradient(
    90deg,
    var(--skeleton-base) 25%,
    var(--skeleton-highlight) 50%,
    var(--skeleton-base) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
}

:root {
  --skeleton-base:      #f0f0f0;
  --skeleton-highlight: #e0e0e0;
}
.dark {
  --skeleton-base:      #1f2937;
  --skeleton-highlight: #374151;
}
/* Result: Feels like content is "flowing in" — more premium than static pulse */
```

### Level 3 → Level 4: Staggered reveal

```jsx
// 🚀 DAVID ENHANCE → Level 4: Content reveals with stagger
function UserList({ users, isLoading }) {
  if (isLoading) return <UserListSkeleton />;

  return (
    <ul>
      {users.map((user, i) => (
        <li
          key={user.id}
          style={{
            animationDelay: `${Math.min(i * 40, 300)}ms`, // Cap at 300ms total stagger
            animationFillMode: 'both',
          }}
          className="animate-[fade-slide-up_200ms_ease-out]"
        >
          <UserCard user={user} />
        </li>
      ))}
    </ul>
  );
}
// Result: Content "flows in" gracefully instead of all appearing at once
```

---

## E2 — Empty State Upgrader

### Level 1 → Level 5: Progressive enhancement

```jsx
// 📊 Current (Basic — Level 1)
function EmptyProjects() {
  return <p className="text-gray-500">No projects found.</p>;
}

// 🚀 Level 2: Add icon
function EmptyProjects() {
  return (
    <div className="text-center py-8 text-gray-400">
      <Folder size={40} className="mx-auto mb-2" />
      <p>No projects found</p>
    </div>
  );
}

// 🚀 Level 3: Add CTA
function EmptyProjects({ onCreateProject }) {
  return (
    <div className="text-center py-12">
      <Folder size={48} className="mx-auto mb-3 text-gray-300" />
      <h3 className="font-medium text-gray-900 dark:text-gray-100">No projects yet</h3>
      <p className="mt-1 text-sm text-gray-500">Create your first project to get started.</p>
      <button onClick={onCreateProject} className="mt-4 btn-primary">
        Create project
      </button>
    </div>
  );
}

// 🚀 Level 4: Context-aware empty states (different per trigger)
function EmptyProjects({ trigger, onCreateProject, onClearFilter }) {
  if (trigger === 'search') return (
    <div className="text-center py-12">
      <SearchX size={48} className="mx-auto mb-3 text-gray-300" />
      <h3 className="font-medium">No projects match your search</h3>
      <p className="mt-1 text-sm text-gray-500">Try different keywords or clear your search</p>
      <button onClick={onClearFilter} className="mt-4 btn-secondary">Clear search</button>
    </div>
  );

  if (trigger === 'filter') return (
    <div className="text-center py-12">
      <FilterX size={48} className="mx-auto mb-3 text-gray-300" />
      <h3 className="font-medium">No projects match your filters</h3>
      <button onClick={onClearFilter} className="mt-4 btn-secondary">Clear all filters</button>
    </div>
  );

  // First-time empty
  return (
    <div className="text-center py-16">
      <div className="mx-auto w-24 h-24 rounded-full bg-blue-50 dark:bg-blue-950 flex items-center justify-center mb-4">
        <Folder size={40} className="text-blue-500" />
      </div>
      <h3 className="text-lg font-semibold">No projects yet</h3>
      <p className="mt-2 text-sm text-gray-500 max-w-xs mx-auto">
        Projects help you organize your work. Create one to get started.
      </p>
      <button onClick={onCreateProject} className="mt-6 btn-primary">
        Create your first project
      </button>
    </div>
  );
}

// 🚀 Level 5: Animated empty state with micro-delight
function EmptyProjects({ onCreateProject }) {
  return (
    <div className="text-center py-16">
      {/* Subtle floating animation on the icon */}
      <div className="mx-auto w-24 h-24 rounded-full bg-gradient-to-br from-blue-50 to-indigo-50
        dark:from-blue-950 dark:to-indigo-950 flex items-center justify-center mb-4
        animate-[float_3s_ease-in-out_infinite]">
        <Folder size={40} className="text-blue-500" />
      </div>
      <h3 className="text-xl font-semibold tracking-tight">Your workspace is empty</h3>
      <p className="mt-2 text-gray-500 max-w-sm mx-auto">
        Projects organize everything in one place. It only takes 30 seconds to create one.
      </p>
      <div className="mt-8 flex flex-col items-center gap-3">
        <button onClick={onCreateProject} className="btn-primary btn-lg">
          Create a project
        </button>
        <p className="text-xs text-gray-400">No credit card required</p>
      </div>
    </div>
  );
}

// Required CSS:
// @keyframes float { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-8px)} }
```

---

## E3 — Microinteraction Injector

### Button Enhancement Suite

```css
/* 🚀 DAVID ENHANCE — Full button microinteraction system */

/* Base: Press feedback */
.btn {
  transition: transform 80ms ease-out, box-shadow 80ms ease-out, background-color 150ms ease;
  transform-origin: center;
}
.btn:hover  { transform: translateY(-1px); box-shadow: 0 4px 12px rgba(0,0,0,0.15); }
.btn:active { transform: scale(0.97) translateY(0); box-shadow: none; }

/* Primary: Shimmer on hover */
.btn-primary {
  position: relative;
  overflow: hidden;
}
.btn-primary::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(105deg, transparent 40%, rgba(255,255,255,0.2) 50%, transparent 60%);
  transform: translateX(-100%);
  transition: transform 400ms ease;
}
.btn-primary:hover::after { transform: translateX(100%); }

/* Destructive: Shake on prevent */
@keyframes shake {
  0%,100% { transform: translateX(0); }
  20%,60%  { transform: translateX(-4px); }
  40%,80%  { transform: translateX(4px); }
}
.btn-destructive-blocked { animation: shake 400ms ease-out; }

/* Icon button: Rotation on action */
.btn-icon-refresh:active svg { animation: spin-once 600ms ease-in-out; }
@keyframes spin-once { to { transform: rotate(360deg); } }
```

### Form Field Focus Enhancement

```css
/* 🚀 DAVID ENHANCE — Premium focus states */

/* Floating label transition */
.field-wrapper {
  position: relative;
}
.field-label {
  position: absolute;
  top: 50%;
  left: 12px;
  transform: translateY(-50%);
  transition: all 150ms ease;
  color: var(--color-text-secondary);
  font-size: 14px;
  pointer-events: none;
}
.field-input:focus ~ .field-label,
.field-input:not(:placeholder-shown) ~ .field-label {
  top: 0;
  transform: translateY(-50%);
  font-size: 11px;
  color: var(--color-primary);
  background: var(--color-bg-primary);
  padding: 0 4px;
}

/* Subtle glow on focus */
.field-input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.15);
}

/* Error shake */
.field-input-error {
  animation: shake 300ms ease-out;
  border-color: var(--color-danger);
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.12);
}
```

### Success State Animation

```jsx
// 🚀 DAVID ENHANCE — Animated success feedback
function SuccessCheckmark({ size = 48 }) {
  return (
    <svg width={size} height={size} viewBox="0 0 52 52">
      {/* Circle draws in */}
      <circle
        cx="26" cy="26" r="25"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
        className="text-green-500"
        strokeDasharray="157"
        strokeDashoffset="157"
        style={{ animation: 'circle-draw 400ms ease-out 0ms forwards' }}
      />
      {/* Checkmark draws in after circle */}
      <path
        fill="none"
        stroke="currentColor"
        strokeWidth="3"
        strokeLinecap="round"
        strokeLinejoin="round"
        d="M14 27 L22 35 L38 19"
        className="text-green-500"
        strokeDasharray="32"
        strokeDashoffset="32"
        style={{ animation: 'check-draw 300ms ease-out 300ms forwards' }}
      />
    </svg>
  );
}

// CSS:
// @keyframes circle-draw { to { stroke-dashoffset: 0; } }
// @keyframes check-draw  { to { stroke-dashoffset: 0; } }
```

### Number Counter Animation

```jsx
// 🚀 DAVID ENHANCE — Animated stat numbers
function AnimatedNumber({ value, duration = 1000, format = n => n }) {
  const [displayed, setDisplayed] = useState(0);
  const ref = useRef(null);

  useEffect(() => {
    const start = performance.now();
    const startVal = 0;

    function update(now) {
      const elapsed = now - start;
      const progress = Math.min(elapsed / duration, 1);
      // Ease out cubic
      const eased = 1 - Math.pow(1 - progress, 3);
      setDisplayed(Math.round(startVal + (value - startVal) * eased));
      if (progress < 1) requestAnimationFrame(update);
    }

    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) requestAnimationFrame(update);
    });
    observer.observe(ref.current);
    return () => observer.disconnect();
  }, [value, duration]);

  return <span ref={ref}>{format(displayed)}</span>;
}

// Usage:
// <AnimatedNumber value={12847} format={n => n.toLocaleString()} />
// Output: 0 → 12,847 animated on scroll into view
```

---

## E4 — Form UX Upgrader

### Password Field — Full Enhancement

```jsx
// 📊 Current (Basic)
<input type="password" placeholder="Password" onChange={handleChange} />

// 🚀 DAVID ENHANCE — Premium password field
function EnhancedPasswordField({ label, value, onChange, showStrength = true }) {
  const [visible, setVisible] = useState(false);
  const strength = useMemo(() => calculateStrength(value), [value]);

  return (
    <div className="field-wrapper">
      <label className="field-label" htmlFor="password">{label}</label>
      <div className="relative">
        <input
          id="password"
          type={visible ? 'text' : 'password'}
          value={value}
          onChange={onChange}
          autoComplete="new-password"
          className="field-input w-full pr-10"
        />
        <button
          type="button"
          onClick={() => setVisible(v => !v)}
          aria-label={visible ? 'Hide password' : 'Show password'}
          className="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600"
        >
          {visible ? <EyeOff size={16} /> : <Eye size={16} />}
        </button>
      </div>

      {/* Strength meter */}
      {showStrength && value && (
        <div className="mt-2">
          <div className="flex gap-1">
            {[1, 2, 3, 4].map(level => (
              <div
                key={level}
                className={`h-1 flex-1 rounded-full transition-colors duration-300
                  ${strength >= level
                    ? level <= 1 ? 'bg-red-500'
                    : level <= 2 ? 'bg-orange-500'
                    : level <= 3 ? 'bg-yellow-500'
                    : 'bg-green-500'
                    : 'bg-gray-200 dark:bg-gray-700'
                  }`}
              />
            ))}
          </div>
          <p className="mt-1 text-xs text-gray-500" aria-live="polite">
            {strength <= 1 ? 'Weak — add numbers and symbols'
            : strength <= 2 ? 'Fair — make it longer'
            : strength <= 3 ? 'Good — add a special character'
            : '✓ Strong password'}
          </p>
        </div>
      )}
    </div>
  );
}
```

### Smart Input Enhancements

```jsx
// 🚀 Phone input with auto-formatting
function PhoneInput({ value, onChange }) {
  function handleChange(e) {
    const digits = e.target.value.replace(/\D/g, '');
    // Format: 0812-3456-7890
    let formatted = digits;
    if (digits.length > 4)  formatted = `${digits.slice(0,4)}-${digits.slice(4)}`;
    if (digits.length > 8)  formatted = `${digits.slice(0,4)}-${digits.slice(4,8)}-${digits.slice(8,12)}`;
    onChange(formatted);
  }

  return (
    <input
      type="tel"
      inputMode="numeric"
      value={value}
      onChange={handleChange}
      placeholder="0812-3456-7890"
      maxLength={14}
    />
  );
}

// 🚀 Character counter for limited text areas
function LimitedTextarea({ value, onChange, maxLength = 280, label }) {
  const remaining = maxLength - value.length;
  const isWarning = remaining <= 20;
  const isOver    = remaining < 0;

  return (
    <div>
      <label>{label}</label>
      <textarea
        value={value}
        onChange={e => onChange(e.target.value)}
        maxLength={maxLength + 50} // Allow typing past to show red count
        className={`w-full ${isOver ? 'border-red-500' : ''}`}
        rows={4}
      />
      <div className="flex justify-end mt-1">
        <span className={`text-xs tabular-nums
          ${isOver ? 'text-red-600 font-medium'
          : isWarning ? 'text-orange-500'
          : 'text-gray-400'}`}
        >
          {remaining}
        </span>
      </div>
    </div>
  );
}

// 🚀 Smart date input with relative hint
function SmartDateInput({ value, onChange, label }) {
  const relativeHint = value ? formatRelativeTime(new Date(value)) : null;

  return (
    <div>
      <label>{label}</label>
      <input type="date" value={value} onChange={e => onChange(e.target.value)} />
      {relativeHint && (
        <p className="text-xs text-gray-500 mt-1">{relativeHint}</p>
        // e.g. "3 days from now" or "Yesterday"
      )}
    </div>
  );
}
```

---

## E5 — Navigation Upgrader

### Active State Enhancement

```jsx
// 📊 Current (Basic — no active state feedback)
<nav>
  <a href="/dashboard">Dashboard</a>
  <a href="/projects">Projects</a>
  <a href="/settings">Settings</a>
</nav>

// 🚀 DAVID ENHANCE — Rich active states with transition
function NavItem({ href, label, icon: Icon }) {
  const pathname = usePathname();
  const isActive = pathname === href || pathname.startsWith(href + '/');

  return (
    <a
      href={href}
      aria-current={isActive ? 'page' : undefined}
      className={`flex items-center gap-2.5 px-3 py-2 rounded-lg text-sm font-medium
        transition-all duration-150
        ${isActive
          ? 'bg-blue-50 dark:bg-blue-950 text-blue-700 dark:text-blue-300'
          : 'text-gray-600 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-800 hover:text-gray-900 dark:hover:text-gray-100'
        }`}
    >
      <Icon
        size={18}
        className={isActive ? 'text-blue-600' : 'text-gray-400'}
        strokeWidth={isActive ? 2.5 : 2}  // Bolder when active
      />
      {label}
      {/* Active indicator dot */}
      {isActive && (
        <span className="ml-auto w-1.5 h-1.5 rounded-full bg-blue-600" aria-hidden />
      )}
    </a>
  );
}
```

### Sidebar with Animated Collapse

```jsx
// 🚀 DAVID ENHANCE — Animated sidebar collapse
function Sidebar({ isCollapsed, onToggle }) {
  return (
    <aside
      className={`flex flex-col border-r transition-all duration-200 ease-out overflow-hidden
        ${isCollapsed ? 'w-16' : 'w-64'}`}
    >
      <button
        onClick={onToggle}
        aria-label={isCollapsed ? 'Expand sidebar' : 'Collapse sidebar'}
        className="p-3 hover:bg-gray-100 dark:hover:bg-gray-800 transition-colors"
      >
        <ChevronLeft
          size={16}
          className={`transition-transform duration-200 ${isCollapsed ? 'rotate-180' : ''}`}
        />
      </button>

      <nav>
        {navItems.map(item => (
          <NavItem
            key={item.href}
            {...item}
            showLabel={!isCollapsed}  // Hide label when collapsed — icon only
          />
        ))}
      </nav>
    </aside>
  );
}
```

### Page Transition Enhancement

```jsx
// 🚀 DAVID ENHANCE — Smooth page transitions (Next.js App Router)
// layout.tsx — wrap page content
function PageWrapper({ children }) {
  return (
    <div
      className="animate-[fade-slide-up_200ms_ease-out]"
      style={{ animationFillMode: 'both' }}
    >
      {children}
    </div>
  );
}

// CSS required:
// @keyframes fade-slide-up {
//   from { opacity: 0; transform: translateY(6px); }
//   to   { opacity: 1; transform: translateY(0); }
// }
```

---

## E6 — Visual Hierarchy Booster

### Typography Hierarchy System

```jsx
// 📊 Current — no clear hierarchy
<div>
  <p>Dashboard</p>
  <p>Welcome back, Alex</p>
  <p>Here's what's happening</p>
  <p>12 projects · 47 tasks</p>
</div>

// 🚀 DAVID ENHANCE — Clear visual hierarchy
<div>
  <span className="text-xs font-medium text-gray-400 uppercase tracking-widest">
    Dashboard
  </span>
  <h1 className="text-3xl font-bold text-gray-900 dark:text-white mt-1 tracking-tight">
    Welcome back, Alex
  </h1>
  <p className="mt-2 text-gray-500">
    Here's what's happening with your projects today.
  </p>
  <div className="flex items-center gap-3 mt-3">
    <span className="text-sm font-medium text-gray-700 dark:text-gray-300">12 projects</span>
    <span className="text-gray-300 dark:text-gray-600">·</span>
    <span className="text-sm font-medium text-gray-700 dark:text-gray-300">47 tasks</span>
  </div>
</div>
```

### Card Elevation System

```jsx
// 🚀 DAVID ENHANCE — Elevation-based card hierarchy
const CARD_VARIANTS = {
  // Flat — for list items, table rows
  flat: "bg-white dark:bg-gray-900 border border-gray-200 dark:border-gray-800",

  // Default — standard content card
  default: "bg-white dark:bg-gray-900 shadow-sm border border-gray-200 dark:border-gray-800",

  // Raised — featured or interactive card
  raised: "bg-white dark:bg-gray-900 shadow-md hover:shadow-lg transition-shadow duration-200",

  // Floating — modal, dropdown, overlay
  floating: "bg-white dark:bg-gray-900 shadow-xl",

  // Highlighted — primary CTA card
  highlighted: "bg-gradient-to-br from-blue-600 to-blue-700 text-white shadow-lg shadow-blue-500/25",
};

function Card({ variant = 'default', className, children }) {
  return (
    <div className={`rounded-xl p-6 ${CARD_VARIANTS[variant]} ${className}`}>
      {children}
    </div>
  );
}
```

### Badge & Status System

```jsx
// 🚀 DAVID ENHANCE — Semantic, consistent badge system
const BADGE_VARIANTS = {
  default:  "bg-gray-100 text-gray-700 dark:bg-gray-800 dark:text-gray-300",
  primary:  "bg-blue-100 text-blue-700 dark:bg-blue-900 dark:text-blue-300",
  success:  "bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300",
  warning:  "bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-300",
  danger:   "bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300",
  info:     "bg-purple-100 text-purple-700 dark:bg-purple-900 dark:text-purple-300",
};

function Badge({ variant = 'default', dot = false, children }) {
  return (
    <span className={`inline-flex items-center gap-1.5 px-2 py-0.5 rounded-full text-xs font-medium
      ${BADGE_VARIANTS[variant]}`}>
      {dot && <span className={`w-1.5 h-1.5 rounded-full bg-current`} aria-hidden />}
      {children}
    </span>
  );
}
// Usage: <Badge variant="success" dot>Active</Badge>
```

---


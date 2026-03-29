# DAVID v6.0 — Scanner Map Reference

Complete operating mode triggers, scanner engine definitions, deduplication rules,
per-scanner trigger detail, and supported technology matrix.

Load this file during Phase 1 to determine which scanners to activate, route to the
correct reference file, and resolve cross-scanner conflicts.

## Table of Contents

1. [Operating Modes](#1-operating-modes)
2. [Always-On Scanners](#2-always-on-scanners)
3. [Scanner Engine — Full Table](#3-scanner-engine--full-table)
4. [Scanner Deduplication Map](#4-scanner-deduplication-map)
5. [Scanner Trigger Detail](#5-scanner-trigger-detail)
   - [Group 1 — Core Debug & Security](#group-1--core-debug--security) (A · B · AC · AE · AF)
   - [Group 2 — Code Quality & Architecture](#group-2--code-quality--architecture) (D · E · P · Q · R · S · UA · RF · AD)
   - [Group 3 — Performance & Bundle](#group-3--performance--bundle) (C · V · W)
   - [Group 4 — Frontend & UX](#group-4--frontend--ux) (O · EN · F · T)
   - [Group 5 — Testing & Documentation](#group-5--testing--documentation) (M · N · TQ)
   - [Group 6 — Infrastructure & Config](#group-6--infrastructure--config) (G · H · I · J · K · X · Y)
   - [Group 7 — API, State & Data](#group-7--api-state--data) (AA · AB · Z · SQ · U)
   - [Group 8 — DX & Delivery Layer](#group-8--dx--delivery-layer) (AH · AI · AJ · AK)
   - [Group 9 — Always-On & Meta](#group-9--always-on--meta) (B · HS · L)
6. [Cross-Scanner Interaction Map](#6-cross-scanner-interaction-map)
7. [Supported Technology Matrix](#7-supported-technology-matrix)

---

## 1. Operating Modes

DAVID auto-detects which modes to activate. Multiple modes always combine simultaneously.

| Mode | Trigger Keywords / Conditions | Scanner(s) |
|------|-------------------------------|------------|
| DEBUG | error / crash / bug / stack trace / "tidak jalan" | A |
| SECURITY | every code submission (always on) | B |
| PERF | "slow" / "lambat" / "optimize" / N+1 / memory / timeout | C |
| REVIEW | "review" / "PR" / "feedback" / "code quality" | D + E |
| ARCH | "architecture" / "structure" / "SOLID" / multi-file | E |
| UIUX | JSX/TSX/Vue/Svelte/HTML + CSS / "ux" / "form" / "mobile" / "tampilan" | O |
| ENHANCE | "enhance" / "improve" / "polish" / "upgrade" / "tingkatkan" | EN |
| A11Y | HTML / JSX / Vue / Svelte components present | F |
| I18N | "localize" / "locale" / multi-region / hardcoded strings | G |
| DEPS | package.json / requirements.txt / go.mod / Gemfile | H |
| GIT | git diff / "changes since" / "what changed" | I |
| MIGRATE | migration files / DDL / ALTER TABLE | J |
| CICD | Dockerfile / .github/workflows / .gitlab-ci.yml | K |
| EXPLAIN | "explain this" / "walk me through" / first look at new codebase | L |
| TESTGEN | after any fix delivery / "generate tests" / "buat test" | M |
| DOCGEN | after any fix delivery / "generate docs" / "document this" | N |
| TYPES | TypeScript files (.ts / .tsx) / "type error" / "TS error" | P |
| DEADCODE | "cleanup" / "unused" / "remove dead" | Q |
| ERRCOV | "reliability" / async-heavy code / "unhandled" | R |
| COMPLEXITY | any function > 20 lines / "refactor this function" | S |
| SEO | Next.js / Nuxt / Remix / Astro / any SSR app | T |
| OBSERV | "logging" / "monitoring" / "production" / backend code | U |
| BUNDLE | Webpack / Vite / Next.js frontend / "bundle size" | V |
| STATE | Redux / Zustand / Context / Pinia / Jotai present | W |
| PWA | service-worker.js / manifest.json | X |
| ENVCONF | .env / config files / startup entry point | Y |
| FLAGS | feature flags / if (FLAG_X) / isFeatureEnabled() | Z |
| APICONTRACT | API routes / OpenAPI / fetch() / axios calls | AA |
| RATELIMIT | search inputs / form submits / API calls without debounce | AB |
| INPUTSANIT | file uploads / user input handling / FormData | AC |
| TECHDEBT | TODO / FIXME / deprecated / @deprecated / "cleanup" | AD |
| GDPR | user data handling / PII / consent / "privacy" / "GDPR" | AE |
| RESILIENCE | external HTTP calls / DB queries / "timeout" / "circuit breaker" | AF |
| REFACTOR | "refactor" / CC score > 15 / complex restructuring | RF |
| UPGRADE | old patterns / legacy APIs / deprecated usage detected | UA |
| HEALTHSCORE | end of every session (always on) | HS |
| PRDESC | post-fix session with diff / "buatkan PR desc" / "write PR" | AH |
| CHANGELOG | "changelog" / "release notes" / commit list provided | AI |
| CROSSFILE | 2+ files submitted / "check consistency" / multi-file session | AJ |
| VIBECODE | AI-generated code / "vibe coding" / "from Copilot" / high TODO density | AK |
| TESTQUAL | *.test.* / *.spec.* files / "audit tests" / "flaky" | TQ |
| SCHEMA | SQL schema / ORM models (Prisma / SQLAlchemy / TypeORM / Drizzle) | SQ |
| FULL SCAN | code submitted with no specific complaint | ALL |

---

## 2. Always-On Scanners

These scanners / steps activate on **every** code interaction — no trigger required:

| Scanner / Step | When | Cannot be disabled by |
|---------------|------|----------------------|
| **B — SECURITY** | Every code submission | Any user instruction. Always runs. |
| **HS — HEALTHSCORE** | End of every session | Only `david: no health score` command disables it |
| **L — EXPLAIN** (partial) | FULL SCAN with no specific complaint | Auto-includes a codebase orientation step before scanning |
| **Framework Profile** (Phase 1.5) | Every code submission | Auto-detected; falls back to Generic profile if unknown | 

> Framework Profile is not a scanner — it is a Phase 1.5 configuration step that adjusts scanner behavior for the detected stack.
> Full profile rules: `references/framework-profiles.md`

> All other scanners are conditional — they activate only when their trigger conditions match.

---

## 3. Scanner Engine — Full Table

All active scanners run simultaneously. Findings are batched by priority (see SKILL.md Phase 2 Fix Priority Order).

| Scanner | Always On | What it does | Load when | Reference |
|---------|:---------:|-------------|-----------|-----------|
| A — BUG | | 5-pass AST-aware: syntax/type → logic → async → resources → integration | any code + error / bug | `references/bug-patterns.md` |
| B — SECURITY | ✅ | OWASP Top 10, secrets (15+ families), ReDoS, supply chain | always | `references/security-audit.md` |
| C — PERF | | Big-O, N+1, memory leaks, sync blocking, DB query plan hints | "slow" / DB code / any loop | `references/performance.md` |
| D — REVIEW | | Code smells, naming violations, missing error handling, doc gaps | "review" / any code | `references/code-review.md` |
| E — ARCH | | SOLID, coupling & cohesion, layer violations, anti-patterns | "architecture" / multi-file | `references/architecture.md` |
| F — A11Y | | WCAG 2.2 Level A + AA + new 2.2 criteria | HTML / JSX / Vue / Svelte | `references/accessibility.md` |
| G — I18N | | Hardcoded strings, string concat, locale ops, RTL, fixed containers | "localize" / multi-region | `references/i18n.md` |
| H — DEPS | | CVE patterns, license conflicts, abandoned packages, wildcard pins | package.json / requirements | `references/dependencies.md` |
| I — GIT | | Diff parsing, change map, cross-file impact, regression detection | git diff submitted | `references/git-diff.md` |
| J — MIGRATE | | Destructive ops, missing rollback, non-idempotent, table locks | migration / DDL files | `references/cicd.md` |
| K — CICD | | Docker (non-root, caching, secrets), GitHub Actions (SHA pinning, permissions) | Dockerfile / workflow yml | `references/cicd.md` |
| L — EXPLAIN | | Codebase map: purpose, entry points, core flow, data flow, external deps | "explain this" / new codebase | *(inline — no reference file)* |
| M — TESTGEN | | Post-fix test generation. Complete runnable files only. Never pseudocode. | post-fix / "generate tests" | `references/test-generator.md` |
| N — DOCGEN | | JSDoc/TSDoc, Python docstrings, Go doc, KDoc, README stubs, OpenAPI snippets | post-fix / "generate docs" | `references/docgen.md` |
| O — UIUX | | 14 sub-scanners: visual, heuristics, states, forms, cognitive load, mobile, animation, dark mode, design system, microcopy, navigation, responsive, delight | frontend code | `references/uiux/index.md` |
| EN — ENHANCE | | 12 sub-enhancers elevating what already works. UX Maturity Score 0–100. | "enhance" / "polish" | `references/enhance/index.md` |
| P — TYPES | | `any` epidemic, missing annotations, unsafe assertions, non-null overuse | TypeScript files | `references/type-safety.md` |
| Q — DEADCODE | | Unused exports, unreachable code, dead dependencies, dead CSS | "cleanup" / "unused" | `references/tech-debt.md` |
| R — ERRCOV | | Error handling coverage map: % async functions handled, silent failures | "reliability" / async code | `references/tech-debt.md` |
| S — COMPLEXITY | | CC per function: 1–5 ok · 6–10 note · 11–15 extract · 16–20 refactor · >20 mandatory split | any function >20 lines | `references/tech-debt.md` |
| T — SEO | | Meta tags, OG/Twitter, structured data, noindex bugs, canonical errors | Next.js / Nuxt / SSR | `references/seo.md` |
| U — OBSERV | | Structured logging, log levels, PII in logs, missing tracing, health check endpoint | production backend | `references/observability.md` |
| V — BUNDLE | | Barrel file patterns, missing code splitting, heavy dependency alternatives | Webpack / Vite / Next.js | `references/bundle.md` |
| W — STATE | | Context re-render, Redux anti-patterns, Zustand pitfalls, server state in Redux | Redux / Zustand / Context | `references/state-management.md` |
| X — PWA | | manifest.json completeness, cache strategy audit, offline support | service-worker / manifest | `references/pwa-and-env.md` |
| Y — ENVCONF | | Startup validation, client bundle exposure, .env.example drift | .env / config files | `references/pwa-and-env.md` |
| Z — FLAGS | | Stale flag detection, hardcoded booleans, inconsistent names, flag inventory | feature flags / if (FLAG_X) | `references/tech-debt.md` |
| AA — APICONTRACT | | Response envelope consistency, HTTP method semantics, status codes | API routes / OpenAPI / fetch | `references/api-contract.md` |
| AB — RATELIMIT | | Missing debounce, double-submit prevention, infinite scroll throttle | search inputs / form submits | `references/api-contract.md` |
| AC — INPUTSANIT | | Path traversal, file upload magic bytes, CSV injection. Dedup with B: AC owns card. | file uploads / user input | `references/security-audit.md` |
| AD — TECHDEBT | | TODO/FIXME inventory, debt heatmap scoring per file | any code | `references/tech-debt.md` |
| AE — GDPR | | PII flow mapping, Right to Erasure (Art.17), Minimization, Consent, Retention | user data handling | `references/privacy-gdpr.md` |
| AF — RESILIENCE | | Timeout, exponential backoff, circuit breaker, graceful degradation, idempotency | external calls | `references/resilience.md` |
| RF — REFACTOR | | Safe step-by-step extraction plan. Activates when CC > 15. | "refactor" / CC > 15 | `references/architecture.md` |
| UA — UPGRADE | | Deprecated API detection, modern equivalents, effort + breaking-change assessment | old patterns / legacy code | `references/code-review.md` |
| HS — HEALTHSCORE | ✅ | Health Score banner, before/after delta, category weights. Always on. | always (end of session) | `references/session-protocols.md` |
| AH — PRDESC | | Structured PR description: What/Why/How/Testing/Breaking/Checklist/Score | post-fix with diff | `references/changelog.md` |
| AI — CHANGELOG | | Versioned changelog: conventional commits parsing, semver bump logic, grouping | "changelog" / commits list | `references/changelog.md` |
| AJ — CROSSFILE | | Pattern drift, naming consistency, type drift, error handling map, API envelope | 2+ files submitted | `references/crossfile.md` |
| AK — VIBECODE | | AI artifact patterns: TODO debris, hallucinated APIs, console.log, `any` epidemic | AI-generated code | `references/vibe-code.md` |
| TQ — TESTQUAL | | Audit existing tests: weak assertions, impl testing, flaky patterns, isolation | test files (*.test.* / *.spec.*) | `references/test-quality.md` |
| SQ — SCHEMA | | Missing indexes, constraints, audit fields, soft delete inconsistency | SQL schema / ORM models | `references/db-schema.md` |

---

## 4. Scanner Deduplication Map

When two scanners would flag the same issue, this map determines who **owns** the finding card and who adds only a cross-reference note. Never double-report the same line.

| Conflict | Card Owner | Other Scanner Action |
|----------|-----------|---------------------|
| B (Security) + AC (Input Sanitization) flag same input validation issue | **AC owns** | B adds cross-reference note |
| C (Perf) + AF (Resilience) flag same external call | **AF owns** if it's a timeout/retry issue; **C owns** if it's Big-O | Both note the other's angle |
| D (Review) + E (Arch) flag same structural issue | **E owns** if SOLID/layer violation; **D owns** if naming/smell | No double card |
| Q (DeadCode) + AD (TechDebt) flag same unused item | **Q owns** the finding card | AD inventories it in the debt heatmap |
| R (ErrCov) + A (Bug) flag same unhandled async | **A owns** if it's causing a concrete bug; **R owns** if it's a coverage gap | No double card |
| S (Complexity) + RF (Refactor) flag same high-CC function | **S owns** the finding card; **RF owns** the migration plan | Linked by finding ID |
| P (Types) + A (Bug) flag same type error causing crash | **A owns** (bug takes priority) | P adds type annotation fix note |
| T (SEO) + F (A11Y) flag same `<img>` missing alt | **F owns** (accessibility is the primary issue) | T notes SEO implication |
| AA (ApiContract) + AB (RateLimit) flag same form submit | **AA owns** if envelope/status code; **AB owns** if missing debounce | No double card |
| AE (GDPR) + B (Security) flag same PII leak | **AE owns** the finding card | B adds OWASP cross-reference |
| UA (Upgrade) + D (Review) flag same deprecated API | **UA owns** the finding card | D notes code smell aspect |

---

## 5. Scanner Trigger Detail

Per-scanner activation rules, what to check, output expectations, and cross-scanner interactions.
Use this section to decide whether a specific scanner should activate and what to look for.

---

### Group 1 — Core Debug & Security

#### A — BUG (5-Pass Debug)

**Triggers:** Any code + error message OR stack trace OR "crash" / "bug" / "tidak jalan" / "error ini" / "kenapa ini salah"

**5 Passes (sequential, all code):**
1. **Syntax & Type** — undefined vars, wrong types, missing return, typo in identifiers, wrong import path
2. **Logic** — off-by-one, wrong operator (= vs ==), inverted conditions, wrong loop bounds, negation error
3. **Async** — unhandled Promise, missing await, race condition, callback after cleanup, concurrent mutation
4. **Resources** — memory leak, unclosed file/socket, missing dispose/cleanup, event listener not removed
5. **Integration** — mismatched API contract, wrong HTTP status handling, bad serialization, env var not read

**Output:** BUG finding card per issue (template: `references/session-protocols.md` § 8 Base Finding Card)

**Cross-scanner:** B (Security) always runs in parallel; R (ErrCov) activates if async-heavy; P (Types) activates if TypeScript

---

#### B — SECURITY (Always On)

**Triggers:** Every code submission, unconditionally. Cannot be disabled.

**15+ Secret Families to detect:**
AWS keys, GCP service account keys, Azure tokens, GitHub PATs, Stripe/PayPal API keys, JWT secrets hardcoded, DB connection strings, SSH private keys, API keys (generic pattern), Slack bot tokens, Twilio auth tokens, Sendgrid/Mailgun keys, generic password assignments, .env values in client bundle, OAuth client secrets

**OWASP Top 10 checks per finding:**
A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection, A04 Insecure Design,
A05 Security Misconfiguration, A06 Vulnerable Components, A07 Auth Failures,
A08 Integrity Failures, A09 Logging Failures, A10 SSRF

**Additional checks:** ReDoS (regex complexity), prototype pollution, path traversal, SSRF patterns, header injection

**Output:** SEC finding card (template: `references/session-protocols.md` § 8 SEC Finding Card) with OWASP category, CVSS estimate, CVE ref if known

**Cross-scanner:** Deduplicates with AC (input sanitization) — AC owns the card, B adds cross-reference. Pairs with AE (GDPR) for PII-related findings — AE owns the card.

---

#### AC — INPUTSANIT (Input Sanitization)

**Triggers:** File uploads present / FormData / user-controlled file paths / CSV parsing / XML parsing / HTML rendering of user input

**Checks:**
- Path traversal: user-controlled filename used in `fs.readFile` / `open()` / `path.join()` without sanitization
- File upload magic bytes: MIME type checked via extension only, not file header bytes
- CSV injection: user data written to CSV without escaping formula characters (`=`, `+`, `-`, `@`)
- HTML injection: user data rendered via `innerHTML` / `dangerouslySetInnerHTML` without sanitization
- XML/JSON injection: user data interpolated directly into XML/JSON strings
- Command injection: user input used in `exec()` / `spawn()` / `subprocess.run()` without escaping

**Output:** SEC finding card — AC owns the card when B+AC conflict (dedup rule §4)

**Cross-scanner:** B (Security) confirms; dedup rule — AC owns, B cross-references

---

#### AE — GDPR (Privacy & GDPR Compliance)

**Triggers:** User data handling / PII fields (email, name, phone, address, IP) / consent logic / "privacy" / "GDPR" / "data retention" / user deletion flows

**GDPR Article checks:**
- **Art. 17 — Right to Erasure:** User deletion endpoint exists? Deletes all PII? Cascade deletes related records? Soft delete doesn't retain PII indefinitely?
- **Art. 25 — Privacy by Design:** Minimum data collected? PII not logged? PII not in error messages? PII not in URLs?
- **Art. 7 — Consent:** Consent recorded with timestamp? Consent withdrawal supported? Consent not bundled with T&C?
- **Art. 5 — Data Minimization:** Only necessary fields collected? Retention period enforced? Unused PII fields present?
- **Art. 32 — Security of Processing:** PII encrypted at rest? PII encrypted in transit? Access to PII logged?

**PII detection patterns:** email, name, phone, dob, address, ip_address, user_agent, device_id, ssn, passport, credit_card fields in schema/API responses

**Output:** AE finding card (use Base Finding Card template) with GDPR article reference

**Cross-scanner:** B (Security) confirms technical aspect; AE owns the finding card. Pairs with SQ (Schema) for PII fields in database schema.

---

#### AF — RESILIENCE (Fault Tolerance)

**Triggers:** External HTTP calls (`fetch`, `axios`, `httpx`, `requests`) / database queries / message queue interactions / "timeout" / "circuit breaker" / "retry" / "idempotent"

**Checks:**
- **Timeouts:** Every external call has an explicit timeout set? Default timeouts are catastrophically long?
- **Retry logic:** Transient failures retried? Exponential backoff with jitter? Max retry count set?
- **Circuit breaker:** Repeated failures to same service trip a circuit breaker? Half-open state implemented?
- **Graceful degradation:** Service unavailable returns fallback/cached response rather than 500?
- **Idempotency:** Mutation endpoints (POST/PUT) are idempotent? Payment/order endpoints have idempotency keys?
- **Bulkhead:** One slow dependency can starve the entire thread pool?
- **Dead letter queue:** Failed async jobs have a DLQ configured?

**Output:** AF finding card (use Base Finding Card template) with pattern category

**Cross-scanner:** C (Perf) for timeout/latency overlap — AF owns if resilience pattern; C owns if Big-O. Pairs with AA (ApiContract) for external API calls.

---

### Group 2 — Code Quality & Architecture

#### D — REVIEW (Code Review)

**Triggers:** "review" / "PR" / "feedback" / "code quality" / any code submission (always checks quality baseline)

**Checks:**
- **Naming:** Vague names (`data`, `temp`, `stuff`, `x`), misleading names (boolean not prefixed `is`/`has`/`can`), inconsistent casing
- **Function design:** Functions > 30 lines doing multiple things, side effects without documentation, mutation of inputs without documentation
- **Error handling:** Errors swallowed silently (`catch (e) {}`), errors logged without re-throw when caller needs to handle, generic error messages
- **Documentation gaps:** Public functions/classes with no JSDoc/docstring, exported APIs undocumented
- **Code smells:** Magic numbers, hardcoded strings, copy-paste blocks (>5 lines identical), deeply nested conditionals (>3 levels)
- **Consistency:** Mixed async styles (callbacks + Promises + async/await in same file), inconsistent error return patterns

**Output:** REVIEW finding card (use Base Finding Card template) with smell category

**Cross-scanner:** E (Arch) for structural issues — E owns if SOLID/layer violation, D owns if smell/naming. UA (Upgrade) for deprecated patterns.

---

#### E — ARCH (Architecture)

**Triggers:** "architecture" / "structure" / "SOLID" / multi-file submissions / code with obvious layer violations

**Checks:**
- **Single Responsibility:** Class/module does more than one thing? Controller contains business logic? Service contains DB queries?
- **Open/Closed:** Adding behavior requires modifying existing class (not extending)?
- **Liskov Substitution:** Subclass can't be substituted for parent without breaking behavior?
- **Interface Segregation:** Fat interface forcing implementors to implement unused methods?
- **Dependency Inversion:** High-level modules depend on low-level modules directly (no abstraction)?
- **Coupling:** Circular dependencies? Feature A imports from Feature B imports from Feature A?
- **Layer violations:** UI code importing from DB layer directly? Controller calling repository directly?
- **Anti-patterns:** God class, anemic domain model, service locator, singleton abuse

**Output:** ARCH finding card (template: `references/session-protocols.md` § 8 ARCH Finding Card) — always CONFIRM BEFORE APPLY

**Cross-scanner:** RF (Refactor) for high-CC functions needing extraction; RF owns the migration plan. D (Review) for naming within architectural decisions.

---

#### P — TYPES (TypeScript Safety)

**Triggers:** Any `.ts` / `.tsx` file / "type error" / "TS error" / TypeScript-related complaint

**Checks:**
- **`any` epidemic:** `any` used as escape hatch; count density — flag if >5% of type annotations are `any`
- **Missing annotations:** Function parameters unannotated, return types missing on public functions
- **Unsafe assertions:** `as SomeType` without validation, `!` non-null assertions without guard, `as unknown as T` double-cast
- **Structural issues:** `interface` vs `type` used inconsistently, union types not exhaustive in switch/if chains
- **Import types:** Value imports used where `import type` is appropriate (tree-shaking risk)
- **Strict mode gaps:** `strictNullChecks` violations, implicit `any` from untyped libraries

**Output:** TYPES finding card (use Base Finding Card template) with specific TypeScript rule violated

**Cross-scanner:** A (Bug) for type errors causing concrete crashes — A owns the card. P documents the type fix.

---

#### Q — DEADCODE (Dead Code)

**Triggers:** "cleanup" / "unused" / "remove dead" / any code (passive check for unreachable blocks)

**Checks:**
- Unused exported functions/classes/constants (exported but never imported anywhere in seen files)
- Unreachable code after `return` / `throw` / `break`
- Dead CSS classes (class defined in CSS but not present in any template)
- Dead dependencies: package in `dependencies` but not imported anywhere in seen files
- Commented-out code blocks > 5 lines (technical debt, not preservation)
- `if (false)` / `if (0)` / always-false condition blocks

**Output:** DEBT finding card with `Type: dead code` (template: `references/session-protocols.md` § 8 DEBT Finding Card)

**Cross-scanner:** AD (TechDebt) inventories in heatmap — Q owns the finding card, AD adds to heatmap.

---

#### R — ERRCOV (Error Coverage)

**Triggers:** "reliability" / async-heavy code / "unhandled" / Promise chains / async/await functions

**Checks:**
- **Coverage map:** What % of async functions have try/catch or `.catch()`?
- **Silent failures:** `catch (e) {}` with no logging, no re-throw, no return of error state
- **Partial handling:** Error caught in one branch but not another of the same function
- **Promise floating:** `void someAsyncFn()` called without await or `.catch()` — unhandled rejection risk
- **Error propagation:** Errors transformed to `null` return without notifying the caller
- **Retry without limit:** Recursive retry without max count — infinite loop on persistent failure

**Output:** DEBT finding card with `Type: error coverage gap`

**Cross-scanner:** A (Bug) if unhandled error is causing a concrete crash — A owns the card.

---

#### S — COMPLEXITY (Cyclomatic Complexity)

**Triggers:** Any function > 20 lines / "refactor this function" / CC audit requested

**CC scoring and action:**
| CC Score | Rating | Action |
|----------|--------|--------|
| 1–5 | OK | No action |
| 6–10 | Note | Flag in report, suggest if user asks |
| 11–15 | Extract | Recommend extraction in REVIEW finding |
| 16–20 | Refactor | Flag with RF (Refactor) plan |
| >20 | Mandatory split | Block: do not add new features until split |

**Output:** DEBT finding card with `Type: CC score [N]`; triggers RF (Refactor) if CC > 15

**Cross-scanner:** RF (Refactor) — S owns the finding card, RF owns the step-by-step extraction plan.

---

#### UA — UPGRADE (Upgrade Advisor)

**Triggers:** Old patterns detected / deprecated API usage / legacy library usage / `@deprecated` JSDoc present

**Checks:**
- Deprecated React patterns: class components, `componentWillMount`, `findDOMNode`
- Old Next.js patterns: Pages Router `getServerSideProps` where App Router server components are better
- Deprecated Node.js: `Buffer()` constructor, `require('url').parse`, legacy `crypto` methods
- Old fetch patterns: XMLHttpRequest when fetch/axios available, callback-based where async/await fits
- Outdated package patterns: Moment.js (use date-fns/dayjs), Request.js (deprecated), node-sass (use sass)
- Assessment output: effort estimate (Low/Med/High) + breaking change risk (None/Low/High)

**Output:** DEBT finding card with `Type: deprecated pattern` and upgrade path

**Cross-scanner:** D (Review) for naming — UA owns the deprecated pattern card.

---

#### RF — REFACTOR (Refactoring Planner)

**Triggers:** "refactor" / S (Complexity) detects CC > 15 / user requests extraction plan

**Output format:**
```
REFACTOR PLAN — [function/module name]
Current CC: [N]  Target CC: [≤10 per extracted function]
Step 1: [specific extraction — name + what to move]
Step 2: [next extraction]
Step N: [final state + test strategy]
Estimated effort: [Low / Med / High]
Breaking change risk: [None / Low — update N callers]
```

**Rule:** RF always **proposes only** — never auto-applies. Always CONFIRM BEFORE APPLY.

**Cross-scanner:** S (Complexity) triggers RF — S owns the CC finding card, RF owns the migration plan.

---

#### AD — TECHDEBT (Tech Debt Heatmap)

**Triggers:** TODO / FIXME / HACK / XXX comments / `@deprecated` / "cleanup" / any code

**Heatmap output per file:**
- Count of TODO/FIXME/HACK comments
- CC scores of top 3 most complex functions
- Dead code items from Q
- Stale flags from Z
- Deprecated patterns from UA
- Aggregate debt score (0–100, lower is worse)

**Output:** DEBT finding card per item + aggregate debt heatmap at Phase 8

**Cross-scanner:** Aggregates from Q (DeadCode), S (Complexity), Z (Flags), UA (Upgrade) — AD owns the heatmap. Individual finding cards owned by their respective scanners.

---

### Group 3 — Performance & Bundle

#### C — PERF (Performance)

**Triggers:** "slow" / "lambat" / "optimize" / "N+1" / "memory" / "timeout" OR any DB query code OR any loop iterating over data

**Checks:**
- **Big-O analysis:** O(n²) loops, O(n³) nested operations, exponential recursion
- **N+1 queries:** ORM `.findMany()` inside a loop without include/join, lazy loading in loops
- **Synchronous blocking:** `fs.readFileSync` in request handler, `JSON.parse` of large payload on main thread
- **Memory allocation:** Object creation in hot path, closures capturing large arrays, missing WeakMap for caches
- **Missing pagination:** Query returns all rows without LIMIT, array processed without streaming
- **React re-renders:** Object/array literals as props (new reference each render), missing useMemo/useCallback for expensive computations, context updates triggering full tree re-render
- **Large bundle imports:** `import _ from 'lodash'` vs `import debounce from 'lodash/debounce'`

**Output:** PERF finding card (template: `references/session-protocols.md` § 8 PERF Finding Card) with Big-O annotation and estimated impact

**Cross-scanner:** V (Bundle) for frontend import analysis. AF (Resilience) for external call timeouts — AF owns if resilience pattern, C owns if algorithmic.

---

#### V — BUNDLE (Bundle Optimization)

**Triggers:** Webpack / Vite / Next.js frontend code / `import` statements in frontend files / "bundle size"

**Checks:**
- **Barrel files:** `index.ts` that re-exports everything — prevents tree-shaking
- **Missing code splitting:** Large components not lazy-loaded, routes not code-split
- **Heavy dependencies:** moment.js (use date-fns), lodash full import (use named), uuid (use crypto.randomUUID), jquery
- **Duplicate dependencies:** Same library imported under two package names (e.g. `react-router` + `react-router-dom` at incompatible versions)
- **Missing dynamic imports:** Large third-party libraries loaded synchronously
- **CSS-in-JS bloat:** Runtime CSS-in-JS when static CSS or Tailwind would suffice

**Output:** DEBT finding card with `Type: bundle bloat` and size impact estimate

**Cross-scanner:** C (Perf) for runtime performance aspect.

---

#### W — STATE (State Management)

**Triggers:** Redux / Zustand / React Context / Pinia / Jotai present in imports

**Checks:**
- **Context re-render:** Context value is a new object literal on every render — all consumers re-render
- **Redux anti-patterns:** Mutations in reducer, async logic in reducer (not thunk/saga), selector not memoized
- **Zustand pitfalls:** Store subscribed to entire state when only one field needed, missing shallow comparison
- **Server state in client store:** API response data managed in Redux/Zustand instead of React Query/SWR
- **Derived state:** Value computed on every render that should be memoized in selector
- **Store coupling:** Two slices directly importing each other

**Output:** REVIEW finding card with state management anti-pattern description

**Cross-scanner:** C (Perf) for re-render performance impact.

---

### Group 4 — Frontend & UX

#### O — UIUX (14 Sub-Scanners)

**Triggers:** Any JSX/TSX/Vue/Svelte/HTML + CSS code, OR "form" / "modal" / "button" / "UI" / "UX" / "tampilan"

**Sub-scanner map:**

| Sub-Scanner | Name | Trigger |
|------------|------|---------|
| O1 | Visual Hierarchy | Any component |
| O2 | Framework Heuristics | Framework-specific patterns |
| O3 | Spacing & Color | CSS / Tailwind classes |
| O4 | Form UX | `<form>` / `<input>` / `<select>` present |
| O5 | Cognitive Load | Complex UI with multiple actions |
| O6 | Mobile & Touch | Mobile viewport / touch targets |
| O7 | Animation | CSS transitions / Framer Motion |
| O8 | Dark Mode | `dark:` Tailwind / CSS custom properties |
| O9 | Empty States | Lists / tables that could be empty |
| O10 | Design System | Component library detected |
| O11 | Skeleton Loading | Loading states in data-fetching components |
| O12 | Navigation | Nav / sidebar / breadcrumb present |
| O13 | Microcopy | Button labels / error messages / empty states |
| O14 | Responsive | Grid / flex layouts |

**Load order:**
1. `references/uiux/index.md` — always load first (router)
2. `references/uiux/foundation.md` — O1, O3 (always for frontend)
3. `references/uiux/interaction.md` — O4, O6, O7, O8, O9, O11, O13 (if forms/mobile/animation detected)
4. `references/uiux/heuristics.md` — O2, O5, O10, O12, O14 (if deep review or design system)
5. `references/uiux/patterns.md` — O extended (specialized components: tables, wizards, file upload, etc.)

**Output:** UX finding card (template: `references/session-protocols.md` § 8 UX Finding Card)

**Cross-scanner:** F (A11Y) always runs in parallel. EN (Enhance) runs after O if "enhance"/"polish" keyword detected.

---

#### EN — ENHANCE (12 Sub-Enhancers)

**Triggers:** "enhance" / "improve" / "polish" / "upgrade" / "tingkatkan" / "percanggih" + frontend code

**Pre-scan mandatory:** Load `references/enhance/core.md` first. Run EN pre-scan to assess current UX maturity before recommending ANYTHING.

**Anti-hallucination rule:** NEVER recommend components, libraries, or patterns that don't already exist in the codebase. Scope strictly to what is actually present.

**Sub-enhancer map:**

| Sub-Enhancer | Name | Scope |
|-------------|------|-------|
| E1 | Loading States → Skeleton | Replace spinners with content-shaped skeletons |
| E2 | Empty States → Illustrated | Add helpful illustrated empty states |
| E3 | Microinteractions | Button feedback, hover states, focus rings |
| E4 | Form UX Upgrade | Inline validation, autofocus, progress |
| E5 | Navigation Upgrade | Active states, breadcrumbs, scroll restoration |
| E6 | Visual Hierarchy | Spacing, weight, size contrast improvements |
| E7 | Feedback Patterns | Toast messages, success states, error recovery |
| E8 | Mobile Enhancements | Touch target sizing, swipe gestures, bottom sheets |
| E9 | Dark Mode Polish | Contrast ratios, elevation shadows in dark |
| E10 | Performance Perception | Optimistic UI, progressive loading, perceived speed |
| E11 | Accessibility Delight | Focus management, announcements, keyboard nav |
| E12 | Data Display Upgrade | Table pagination, chart accessibility, empty charts |

**Load:** `references/enhance/core.md` (E1–E6, always) + `references/enhance/extended.md` (E7–E12)

**Output:** Enhancement proposal with before/after code, UX Maturity Score 0–100 before/after, per-category breakdown. Every proposal card includes mandatory `Perf Cost` field (NONE / LOW / MEDIUM / HIGH) + `Safe to ship` status — format defined in `references/enhance/extended.md` § Enhancement Delivery Format.

---

#### F — A11Y (Accessibility)

**Triggers:** HTML / JSX / Vue / Svelte components present

**WCAG 2.2 checks:**

*Level A (Critical — always fix):*
- Images missing `alt` text (or `alt=""` for decorative — must be explicit)
- Form inputs missing associated `<label>` or `aria-label`
- Interactive elements not keyboard-accessible (no `tabindex`, no keyboard handler)
- Color used as the only means of conveying information
- `<div>`/`<span>` used as button without `role="button"` and keyboard handler

*Level AA (Important — fix in scope):*
- Color contrast ratio < 4.5:1 for normal text, < 3:1 for large text
- Focus indicator not visible (outline: none without replacement)
- Form errors not programmatically associated with the field (`aria-describedby`)
- Page title missing or non-descriptive
- Language attribute missing on `<html>`

*WCAG 2.2 new criteria:*
- 2.5.3 Focus Appearance: focus indicator meets minimum size and contrast
- 2.4.11 Focus Not Obscured: focused element not fully hidden by sticky content
- 3.2.6 Consistent Help: help mechanisms in same relative order across pages

**Output:** REVIEW finding card with WCAG criterion reference (e.g. "WCAG 1.1.1 Level A")

**Cross-scanner:** O (UIUX) runs in parallel — F owns accessibility findings, O owns UX findings.

---

#### T — SEO (Search Engine Optimization)

**Triggers:** Next.js / Nuxt / Remix / Astro / any SSR/SSG app

**Checks:**

*Critical (P1 — fix immediately):*
- Missing `<title>` tag or non-descriptive title
- Missing `<meta name="description">` or description > 160 chars
- `<meta name="robots" content="noindex">` on pages that should be indexed
- Multiple `<h1>` on same page
- Duplicate `<title>` across pages (static site)

*Important (P2):*
- Missing Open Graph tags (`og:title`, `og:description`, `og:image`)
- Missing Twitter Card tags
- Missing canonical URL (`<link rel="canonical">`)
- Images missing `alt` text (also A11Y — F owns if A11Y in scope)
- Missing structured data for content types (articles, products, FAQs)

*Next.js specific:*
- `generateMetadata()` not implemented in App Router pages
- `next/image` not used for images (missing lazy loading + size optimization)
- `loading="lazy"` on above-the-fold images (hurts LCP)

**Output:** REVIEW finding card with SEO impact category

---

### Group 5 — Testing & Documentation

#### M — TESTGEN (Test Generation)

**Triggers:** After any fix delivery / "generate tests" / "buat test" / "tambah test" / "write unit tests"

**Rules:**
- Never output pseudocode, placeholder tests, or `// TODO: implement`
- Always output complete runnable test files
- Always include: runner command + import path + setup/teardown if needed

**Test types to generate per finding:**

| Finding Type | Tests to Generate |
|-------------|-------------------|
| BUG fix | Regression test that would have caught the bug + boundary/null tests |
| SEC fix | Injection/injection attempt test + auth bypass attempt |
| PERF fix | Benchmark test showing improvement |
| UX fix | Interaction test (e.g. form submission, keyboard nav) |
| Type fix | Type-level test (`expectTypeOf` or compile-time assertion) |
| A11Y fix | Accessibility assertion (aria-label, role, focus) |

**Output:** Complete test file per fixed module + runner command

**Cross-scanner:** N (DocGen) runs in parallel after fixes.

---

#### N — DOCGEN (Documentation Generation)

**Triggers:** After any fix delivery / "generate docs" / "document this" / "add JSDoc"

**Rules:**
- Only document what DAVID touched — never generate docs for untouched functions
- Every modified function gets an updated docstring with `@davidfix` tag
- Tag format: `@davidfix [FINDING-ID] [one-line description of fix]`

**Language-specific format:**
- **TypeScript/JavaScript:** JSDoc (`/** */`) with `@param`, `@returns`, `@throws`, `@example`
- **Python:** Google-style or NumPy-style docstring matching existing file convention
- **Go:** Package-level and function-level doc comments (`// FunctionName ...`)
- **Java/Kotlin:** KDoc (`/** */`) with `@param`, `@return`, `@throws`
- **Rust:** `///` doc comments with `# Examples` section

**Additional outputs:**
- README diff: if a public API changed, generate the diff for the README
- OpenAPI snippet: if an API route was modified, generate the updated OpenAPI path object

**Output:** Updated docstring per touched function + optional README diff + optional OpenAPI snippet

> Full templates for all languages: `references/docgen.md`

---

#### TQ — TESTQUAL (Test Quality Audit)

**Triggers:** Any `*.test.*` or `*.spec.*` file submitted / "audit tests" / "test quality" / "flaky test"

**6 Anti-patterns to detect:**

| Anti-pattern | Description | Severity |
|-------------|-------------|----------|
| Weak assertions | `toBeTruthy()` / `toBeDefined()` instead of `toEqual(expected)` | P2 |
| Implementation testing | Testing internal state/methods, not observable behavior | P2 |
| Flaky timing | `setTimeout` / `sleep()` in tests instead of fake timers or `waitFor` | P1 |
| Missing isolation | Shared mutable state between tests, no `beforeEach` reset | P1 |
| Incomplete edge cases | No null input tests, no error path tests, happy path only | P2 |
| Snapshot overuse | Giant snapshots that change on every minor UI tweak | P3 |

**Output:** DEBT finding card with `Type: test quality` + per-file test quality score (0–100) + refactored test example

---

### Group 6 — Infrastructure & Config

#### G — I18N (Internationalization)

**Triggers:** "localize" / "locale" / multi-region mention / hardcoded user-visible strings detected

**Checks:**
- Hardcoded English strings in JSX/templates (not using i18n function)
- String concatenation with translated parts (`"Hello " + name` vs `t('greeting', { name })`)
- Date/number formatting without locale: `new Date().toLocaleDateString()` without locale arg
- Fixed pixel containers that break with RTL or long translated text
- Missing RTL support: no `dir="rtl"` handling, no `margin-inline` vs `margin-left`
- Plural forms: `"1 item" / "N items"` hardcoded instead of plural rule

**Output:** REVIEW finding card with i18n category

---

#### H — DEPS (Dependency Audit)

**Triggers:** `package.json` / `requirements.txt` / `go.mod` / `Gemfile` / `pyproject.toml` submitted

**Checks:**
- **CVE patterns:** Known vulnerable version ranges for common packages (express < 4.18, lodash < 4.17.21, etc.)
- **License conflicts:** GPL dependencies in commercial projects, license incompatibilities
- **Abandoned packages:** Last publish > 2 years ago + no maintained fork
- **Wildcard pins:** `"*"` or `"^0.x"` pins that allow breaking changes
- **Duplicate resolution:** Same package at 3+ different versions in lockfile
- **Dev/prod confusion:** Testing tools in `dependencies` instead of `devDependencies`

**Output:** REVIEW finding card with dependency health category

---

#### I — GIT (Git Diff Review)

**Triggers:** Git diff submitted / "changes since" / "what changed" / diff format detected in input

**Actions:**
1. Parse diff → build change map (files changed, lines added/removed per file)
2. Focus scanners on changed lines + immediate callers/callees of changed functions
3. Mark unchanged code as "baseline — not re-reviewed" in the fingerprint
4. Detect regression risk: did the change touch a function called from N other places?

**Output:** `📋 Git scope: N files changed, +X -Y lines — delta review mode`

> Full diff patterns and change map rules: `references/git-diff.md`

---

#### J — MIGRATE (Migration Safety)

**Triggers:** SQL migration files / DDL statements / `ALTER TABLE` / `CREATE TABLE` / Prisma migrations

**Checks:**
- **Destructive operations:** `DROP TABLE`, `DROP COLUMN`, `TRUNCATE` without rollback plan
- **Missing rollback:** Migration has `up()` but no `down()` function
- **Non-idempotent:** Migration fails if run twice (missing `IF NOT EXISTS`, `IF EXISTS`)
- **Table locks:** `ALTER TABLE` on large tables without `ALGORITHM=INPLACE` or online DDL
- **Data loss risk:** Changing column type without data migration step
- **Missing index:** FK column added without index (full table scan on joins)

**Output:** SEC finding card (severity: data loss risk) or REVIEW finding card

---

#### K — CICD (CI/CD & Docker)

**Triggers:** `Dockerfile` / `.github/workflows/*.yml` / `.gitlab-ci.yml` / `docker-compose.yml`

**Docker checks:**
- Running as root (no `USER` instruction)
- Missing `.dockerignore` (copies node_modules into image)
- Secrets in `ENV` or `ARG` (visible in image layers)
- Base image without version pin (`FROM node:latest`)
- Missing health check (`HEALTHCHECK` instruction)
- Multi-stage build not used for compiled languages

**GitHub Actions checks:**
- Actions pinned to branch (`@main`) not commit SHA (`@abc123`)
- Excessive permissions (`permissions: write-all`)
- Secrets echoed to logs (`echo ${{ secrets.TOKEN }}`)
- Missing `timeout-minutes` (job can run indefinitely)
- Untrusted input in `run:` command (injection via PR title/comment)

**Output:** SEC or REVIEW finding card with CI/CD category

---

#### X — PWA (Progressive Web App)

**Triggers:** `service-worker.js` / `sw.js` / `manifest.json` / `workbox` imports

**Checks:**
- `manifest.json`: missing `name`, `short_name`, `icons` (192px + 512px), `start_url`, `display`, `theme_color`
- Cache strategy: is the cache strategy appropriate for the content type? (Network-first for API, Cache-first for static)
- Service worker update flow: stale worker blocks app update? `skipWaiting()` and `clients.claim()` present?
- Offline fallback: offline page registered? Network failures handled gracefully?

**Output:** REVIEW finding card with PWA category

---

#### Y — ENVCONF (Environment Configuration)

**Triggers:** `.env` / `.env.example` / `config.ts` / startup entry point / `process.env` access

**Checks:**
- **Startup validation:** All required env vars validated at startup (fail-fast) vs discovered at runtime
- **Client bundle exposure:** `NEXT_PUBLIC_SECRET_KEY` pattern — secrets accidentally exposed to client bundle
- **.env.example drift:** Variables in `.env.example` not present in actual config validation, or vice versa
- **Type coercion:** `process.env.PORT` used as number without parseInt
- **Default values:** Missing defaults for optional env vars (app crashes if var absent)
- **CI/CD alignment:** Env vars required in production not in CI environment config

**Output:** REVIEW finding card with env config category

---

### Group 7 — API, State & Data

#### AA — APICONTRACT (API Contract Validation)

**Triggers:** API route handlers / OpenAPI / `fetch()` calls / `axios` calls / REST endpoints

**Checks:**
- **Response envelope consistency:** Some endpoints return `{ data, error }`, others return raw objects
- **HTTP method semantics:** POST used for queries, GET used for mutations, DELETE returning 200 with body
- **Status codes:** 200 returned for errors, 500 for validation failures (should be 400), missing 404 for not-found
- **Pagination:** Large collection endpoint missing `limit`/`offset` or cursor
- **Versioning:** No API version in path or header — breaking changes affect all clients
- **Error format:** Error responses have no consistent shape across endpoints

**Output:** REVIEW finding card with API contract category

---

#### AB — RATELIMIT (Rate Limiting & Debounce)

**Triggers:** Search inputs / form submit handlers / `fetch()` in event handlers / infinite scroll / polling

**Checks:**
- Search input fires API call on every keystroke (missing `debounce`)
- Form submit button not disabled after first click (double-submit)
- Infinite scroll fires multiple simultaneous requests (missing in-flight guard)
- Polling interval too aggressive (< 5s for non-critical data)
- File upload no progress indicator and can be submitted multiple times

**Output:** REVIEW finding card with rate limit category

---

#### Z — FLAGS (Feature Flags)

**Triggers:** Feature flags / `if (FLAG_X)` / `isFeatureEnabled()` / `FEATURE_*` constants

**Checks:**
- **Stale flags:** Flag referenced in code but not in the flag configuration system
- **Hardcoded booleans:** `const FLAG_NEW_UI = true` (should be in flag system)
- **Inconsistent naming:** Mix of `FEATURE_X`, `flag_x`, `isFeatureX` in same codebase
- **Flag inventory:** Count of flags per file, total flags in codebase
- **Cleanup candidate:** Flag that is always `true` in production (dead branch of `if (flag)`)

**Output:** DEBT finding card with `Type: stale flag` or `Type: flag inconsistency`

---

#### SQ — SCHEMA (Database Schema Quality)

**Triggers:** SQL schema files / Prisma schema / SQLAlchemy models / TypeORM entities / Drizzle schemas / Django models

**Checks:**
- **Missing primary key:** Table has no PK defined
- **Missing FK constraints:** Relationship implied by naming (`user_id`) but no FK defined
- **Unindexed FK columns:** FK column has no index — full table scan on every join
- **Unindexed query columns:** Columns used in WHERE/ORDER BY without index (inferred from API query patterns if visible)
- **Missing NOT NULL:** Columns that should never be null but aren't constrained
- **Missing audit fields:** Table missing `created_at` / `updated_at`
- **Soft delete inconsistency:** Some tables have `deleted_at`, others use hard delete
- **Missing unique constraints:** Business rule "email must be unique" enforced only in app code, not DB

**Output:** SQ finding card (use Base Finding Card template) with SQL fix or ORM equivalent

---

#### U — OBSERV (Observability)

**Triggers:** "logging" / "monitoring" / "production" / backend service code

**Checks:**
- **Structured logging:** Using `console.log` string interpolation instead of structured log object (`logger.info({ userId, action })`)
- **Log levels:** `console.error` for info-level events, `console.log` in production paths
- **PII in logs:** Email, phone, password, token, credit card logged in any level
- **Missing correlation ID:** Requests have no trace ID / correlation ID in logs
- **Missing health check:** No `/health` or `/readyz` endpoint
- **Missing metrics:** No latency, error rate, or throughput instrumentation for critical paths
- **Silent 500s:** Unhandled exceptions caught globally but not logged

**Output:** REVIEW finding card with observability category

---

### Group 8 — DX & Delivery Layer

#### AH — PRDESC (PR Description Generator)

**Triggers:** Post-fix session with git diff / "buatkan PR desc" / "generate PR description" / "buat deskripsi PR" / "write PR" / "PR ini apa aja yang berubah"

**Input needed:** Git diff OR changed file list + task context (ticket number or one-sentence summary)

**Output structure:**
```
## What
[What changed — concrete, technical]

## Why
[Root cause / ticket / business reason]

## How
[Approach taken — key decisions]

## Testing
[How to verify — manual steps + automated tests added]

## Breaking Changes
[None / or: [list of breaking changes]]

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated
- [ ] No console.log left
- [ ] No unauthorized deletions
- [ ] Health score improved

Quality Score: [N]/7
```

**Score thresholds:** 7/7 — ready to merge · 5–6 — acceptable · <5 — flag missing sections

**Cross-scanner:** Pairs with AI (Changelog) when user wants both PR desc and version bump.

> Full PR description templates: `references/changelog.md`

---

#### AI — CHANGELOG (Changelog Generator)

**Triggers:** "changelog" / "release notes" / "buat changelog" / "release v" / commit list submitted / "what changed since v"

**Input formats accepted:** Git log `--oneline`, DAVID session fix list, manual bullet list, mixed

**Grouping order:**
1. ⚠️ Breaking Changes
2. ✨ Features
3. 🐛 Bug Fixes
4. 🔐 Security
5. ⚡ Performance
6. 📦 Dependencies
7. 🔧 Refactor / Internal

**Semver bump logic:**
- `BREAKING CHANGE` in any commit → MAJOR bump
- `feat:` commits → MINOR bump
- `fix:` / `perf:` / `refactor:` only → PATCH bump

**Cross-scanner:** Pairs with AH (PRDesc) when user wants both.

> Full changelog templates + semver rules: `references/changelog.md`

---

#### AJ — CROSSFILE (Cross-File Consistency)

**Triggers:** 2+ files submitted / "cek konsistensi" / "check consistency" / "are these consistent" / multi-file session

**Scope:** Only files DAVID has seen in the session. Never speculates about unseen files.

**6 Drift categories:**

| Category | What to check |
|----------|--------------|
| Pattern drift | Auth pattern, error handling pattern, async pattern — consistent across files? |
| Naming consistency | Variable/function/file naming conventions — consistent? |
| Type/interface shape drift | Same entity has different shape in different files? |
| API response envelope | All endpoints return the same `{ data, error, meta }` envelope shape? |
| Import/dependency duplication | Same utility imported differently across files (path alias vs relative)? |
| Test coverage consistency | Some modules have tests, others don't — flag the gap |

**Output:** AJ Consistency Report with per-category findings + overall consistency score 0–100

**Cross-scanner:** Always runs alongside I (GIT) in multi-file diff sessions. Pairs with R (ErrCov) for error handling map consistency.

---

#### AK — VIBECODE (AI-Generated Code Audit)

**Triggers:** "from Copilot" / "from ChatGPT" / "AI generated" / "vibe coding" / "Cursor" / "ini dari AI" / OR high density of TODO/`console.log`/`any` in code

**6 Epidemic patterns:**

| # | Pattern | Description | Fix |
|---|---------|-------------|-----|
| 1 | TODO debris | Code full of TODO placeholders that should be real implementations | Implement or remove |
| 2 | Hallucinated API | Calling `.method()` that doesn't exist on that type | Replace with correct API |
| 3 | console.log epidemic | Debugging artifacts left in production paths | Remove or convert to logger |
| 4 | `any` epidemic | TypeScript `any` used as escape hatch everywhere | Type properly |
| 5 | Copy-paste boilerplate | Identical blocks repeated with minor variations | Extract to shared function |
| 6 | Missing edge cases | Happy-path-only code with no null/undefined/error handling | Add guards |

**Output:** Vibe code audit report with epidemic score per pattern (0–10) and total epidemic score

---

### Group 9 — Always-On & Meta

#### B — SECURITY (see also Group 1)

Always on. Fires unconditionally on every code submission. See [Group 1](#group-1--core-debug--security) for full detail.

---

#### HS — HEALTHSCORE (Always On)

**Triggers:** Always. End of every session. Cannot be disabled unless `david: no health score` is active.

**Category weights:**
| Category | Weight |
|----------|--------|
| Bug-Free | 20% |
| Security | 20% |
| Privacy/GDPR | 10% |
| Performance | 12% |
| UI/UX Quality | 10% |
| Test Quality | 10% |
| Resilience | 8% |
| Schema Quality | 5% |
| Code Quality | 3% |
| Type Safety | 2% |

**Score bands:** 90–100 Excellent · 75–89 Good · 60–74 Needs Work · <60 Critical

**Output:** Full Health Score banner (template: `references/session-protocols.md` § 12)

---

#### L — EXPLAIN (Self-Explain Mode)

**Triggers:** "explain this" / "walk me through" / "how does this work" / "new codebase" / first look at unfamiliar project

**Action sequence:**
1. Read all submitted files
2. Build Codebase Map:
   - **Purpose:** What does this codebase do in one sentence?
   - **Entry point:** Where does execution start?
   - **Core flow:** Main execution path in plain language, in order
   - **Key modules:** What does each file/folder do?
   - **Data flow:** How does data move through the system?
   - **External dependencies:** What external services/APIs does it call?
3. Deliver plain-language walkthrough in execution order
4. Still run all relevant scanners after explaining — L does not replace the audit

**Output:** Codebase Map block followed by normal audit output

> No reference file — L is purely analytical using submitted code.

---

## 6. Cross-Scanner Interaction Map

Summary of all cross-scanner relationships. Use this to route findings to the correct scanner and avoid duplicate cards.

| When you see... | Primary scanner | Secondary scanner | Rule |
|----------------|-----------------|-------------------|------|
| Input validation failure | AC | B | AC owns card; B cross-references |
| External call timeout | AF | C | AF owns if resilience pattern; C owns if Big-O |
| Structural code issue | E | D | E owns if SOLID/layer; D owns if smell/naming |
| Unused code item | Q | AD | Q owns card; AD adds to heatmap |
| Unhandled async error causing crash | A | R | A owns card (bug takes priority) |
| High CC function needing split | S | RF | S owns card; RF owns migration plan |
| Type error causing crash | A | P | A owns card; P documents type fix |
| `<img>` missing alt (accessibility + SEO) | F | T | F owns card; T notes SEO implication |
| Form submit without debounce | AB | AA | AB owns if missing debounce; AA owns if envelope |
| PII leaked in API response | AE | B | AE owns card; B adds OWASP ref |
| Deprecated API in use | UA | D | UA owns card; D notes code smell aspect |
| A11Y + UX issue in same component | F | O | F owns accessibility card; O owns UX card |
| PR description + changelog both wanted | AH | AI | Both run; AH for PR, AI for changelog |
| Tech debt item (TODO/deprecated/dead) | Q/S/Z/UA | AD | Each owns their card; AD aggregates into heatmap |

---

## 7. Supported Technology Matrix

### Languages
Python · JavaScript · TypeScript · Java · Kotlin · C · C++ · C# · Go · Rust · PHP · Ruby · Swift · Dart · SQL · Bash · HTML · CSS · YAML · Terraform

### Frontend
React · Next.js (App Router + Pages Router) · Vue 3 · Nuxt · Angular · Svelte · Astro · Tailwind CSS · Redux · Zustand · React Query (TanStack) · Jotai · Pinia · SWR

### Backend
FastAPI · Django · Flask · Express · NestJS · Spring Boot · Laravel · Rails · Gin · Axum · .NET / ASP.NET · GraphQL · tRPC · Hono

### Mobile
Flutter · React Native · SwiftUI · Jetpack Compose

### Infrastructure
Docker · Kubernetes · GitHub Actions · GitLab CI · AWS (Lambda, ECS, RDS, S3, IAM) · GCP · Azure · Terraform · Nginx · Caddy

### Databases & ORMs
PostgreSQL · MySQL · SQLite · MongoDB · Redis · DynamoDB · Supabase · Prisma · SQLAlchemy · TypeORM · Drizzle · Django ORM · ActiveRecord

### Observability
Winston · Pino · structlog · Sentry · Datadog · OpenTelemetry · Prometheus · Grafana · Loki

### Testing
Jest · Vitest · Pytest · Go test · JUnit · Playwright · Cypress · Testing Library · Supertest

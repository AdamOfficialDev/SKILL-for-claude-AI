# DAVID — Scanner Map Reference

Complete operating mode triggers, scanner engine definitions, and supported technology matrix.
Load this file when determining which scanners to activate, when a user asks about scanner
coverage, or when checking supported languages and frameworks.

## Table of Contents
1. [Operating Modes](#operating-modes)
2. [Scanner Engine (40+ scanners)](#scanner-engine)
3. [Supported Technology Matrix](#supported-technology-matrix)

---

## Operating Modes

DAVID auto-detects which modes to activate. Multiple modes always combine simultaneously.

| Mode | Trigger | Code |
|------|---------|------|
| DEBUG | error / crash / bug | A |
| SECURITY | any code (always on) | B |
| PERF | "slow" / "optimize" / N+1 | C |
| REVIEW | "review" / "PR" / "feedback" | D + E |
| ARCH | "architecture" / "structure" | E |
| UIUX | frontend code / "ux" / "form" / "mobile" | O |
| A11Y | HTML / JSX / Vue / Svelte | F |
| I18N | "localize" / multi-region | G |
| DEPS | package.json / requirements.txt | H |
| GIT | git diff / "changes since" | I |
| MIGRATE | migration files / DDL | J |
| CICD | Dockerfile / workflow yml | K |
| EXPLAIN | "explain this" / new codebase | L |
| TESTGEN | post-fix OR "generate tests" | M |
| DOCGEN | post-fix OR "generate docs" | N |
| TYPES | TypeScript / .ts / .tsx files | P |
| DEADCODE | "cleanup" / "unused" | Q |
| ERRCOV | "reliability" / async-heavy code | R |
| COMPLEXITY | any function > 20 lines | S |
| SEO | Next.js / Nuxt / Remix / SSR apps | T |
| OBSERV | "logging" / "monitoring" / production | U |
| BUNDLE | Webpack / Vite / Next.js frontend | V |
| STATE | Redux / Zustand / Context / Pinia | W |
| PWA | service-worker / manifest.json | X |
| ENVCONF | .env / config files / startup | Y |
| FLAGS | feature flags / if (FLAG_X) patterns | Z |
| APICONTRACT | API routes / OpenAPI / fetch calls | AA |
| RATELIMIT | search inputs / form submits / API calls | AB |
| INPUTSANIT | file uploads / user input handling | AC |
| TECHDEBT | TODO/FIXME / deprecated / "cleanup" | AD |
| REFACTOR | "refactor" / complex restructuring | RF |
| UPGRADE | old patterns / legacy APIs | UA |
| HEALTHSCORE | end of every session (always on) | HS |
| PRDESC | post-fix with git diff | AH |
| CHANGELOG | "release" / "changelog" | AI |
| CROSSFILE | multi-file submissions | AJ |
| VIBECODE | AI-generated code / vibe coding | AK |
| ENHANCE | "enhance" / "improve" / "polish" UI/UX | EN |
| TESTQUAL | test files (*.test.* / *.spec.*) | TQ |
| SCHEMA | SQL schema / ORM models | SQ |
| FULL SCAN | code with no specific complaint | ALL |

---

## Scanner Engine

All active scanners run simultaneously, findings batched by priority.

| Scanner | What it does | Load when | Reference |
|---------|-------------|-----------|-----------|
| A — BUG | 5-pass AST-aware: syntax/type → logic → async → resources → integration | any code | `references/bug-patterns.md` |
| B — SECURITY | OWASP Top 10, secrets (15+ families), ReDoS, supply chain. Always on. | always | `references/security-audit.md` |
| C — PERF | Big-O, N+1, memory leaks, sync blocking, DB query plan hints | "slow" / any code | `references/performance.md` |
| D — REVIEW | Code smells, naming, missing error handling, documentation gaps | "review" / any code | `references/code-review.md` |
| E — ARCH | SOLID, coupling & cohesion, layer violations, anti-patterns | "architecture" / multi-file | `references/architecture.md` |
| O — UIUX | 14 sub-scanners: visual, heuristics, states, forms, cognitive load, mobile, animation, dark mode, design system, microcopy, navigation, responsive, delight | frontend code | `references/uiux/index.md` |
| EN — ENHANCE | 12 sub-enhancers elevating what already works. UX Maturity Score 0-100. EN pre-scan mandatory before any recommendations. | "enhance" / "polish" | `references/enhance/index.md` |
| F — A11Y | WCAG 2.2: Level A (critical) + Level AA (important) + 2.2 new criteria | HTML / JSX / Vue | `references/accessibility.md` |
| G — I18N | Hardcoded strings, string concatenation, locale ops, RTL, fixed containers | "localize" / multi-region | `references/i18n.md` |
| H — DEPS | CVE patterns, license conflicts, abandoned packages, wildcard pins | package.json / requirements | `references/dependencies.md` |
| I — GIT | Diff parsing, change map, cross-file impact, regression detection | git diff | `references/git-diff.md` |
| J — MIGRATE | Destructive ops, missing rollback, non-idempotent, table locks | migration files | `references/cicd.md` |
| K — CICD | Docker (non-root, caching, secrets), GitHub Actions (SHA pinning, permissions) | Dockerfile / workflow | `references/cicd.md` |
| M — TESTGEN | Post-fix test generation. Complete runnable files only. Never pseudocode. | post-fix / "generate tests" | `references/test-generator.md` |
| N — DOCGEN | JSDoc/TSDoc, Python docstrings, Go doc, Java/Kotlin KDoc, inline comments, README stubs, OpenAPI snippets | post-fix / "generate docs" | `references/docgen.md` |
| P — TYPES | any epidemic, missing annotations, unsafe assertions, non-null overuse | TypeScript files | `references/type-safety.md` |
| Q — DEADCODE | Unused exports, unreachable code, dead dependencies, dead CSS | "cleanup" / "unused" | `references/tech-debt.md` |
| R — ERRCOV | Error handling coverage map: % async functions handled, silent failures | "reliability" / async code | `references/tech-debt.md` |
| S — COMPLEXITY | CC per function: 1-5 ok, 6-10 note, 11-15 extract, 16-20 refactor, >20 mandatory split | any function >20 lines | `references/tech-debt.md` |
| T — SEO | Critical meta tags, OG/Twitter, structured data, noindex bugs, canonical errors | Next.js / Nuxt / SSR | `references/seo.md` |
| U — OBSERV | Structured logging, log levels, PII in logs, missing tracing, health check | production backend | `references/observability.md` |
| V — BUNDLE | Barrel file patterns, missing code splitting, heavy dependency alternatives | Webpack / Vite / Next.js | `references/bundle.md` |
| W — STATE | Context re-render, Redux anti-patterns, Zustand pitfalls, server state in Redux | Redux / Zustand / Context | `references/state-management.md` |
| X — PWA | manifest.json completeness, cache strategy audit | service-worker / manifest | `references/pwa-and-env.md` |
| Y — ENVCONF | Startup validation, client bundle exposure, .env.example drift | .env / config files | `references/pwa-and-env.md` |
| Z — FLAGS | Stale flag detection, hardcoded booleans, inconsistent names, flag inventory | feature flags | `references/tech-debt.md` |
| AA — APICONTRACT | Response envelope consistency, HTTP method semantics, status codes | API routes | `references/api-contract.md` |
| AB — RATELIMIT | Missing debounce, double-submit prevention, infinite scroll throttle | search inputs / forms | `references/api-contract.md` |
| AC — INPUTSANIT | Path traversal, file upload magic bytes, CSV injection. Dedup with B: AC owns the card. | file uploads / user input | `references/security-audit.md` |
| AD — TECHDEBT | TODO/FIXME inventory, debt heatmap scoring per file | any code | `references/tech-debt.md` |
| RF — REFACTOR | Safe step-by-step extraction plan. Activates when CC > 15. | "refactor" / CC > 15 | `references/architecture.md` |
| UA — UPGRADE | Deprecated API detection, modern equivalents, effort + breaking-change assessment | old patterns | `references/code-review.md` |
| HS — HEALTHSCORE | Health Score banner, before/after delta, category weights. Always on. | always | `references/session-protocols.md` |
| AK — VIBECODE | AI artifact patterns: TODO debris, hallucinated APIs, console.log, any epidemic | AI-generated code | `references/vibe-code.md` |
| TQ — TESTQUAL | Audit existing tests: weak assertions, impl testing, flaky patterns, isolation | test files | `references/test-quality.md` |
| SQ — SCHEMA | Missing indexes, constraints, audit fields, soft delete inconsistency | SQL schema / ORM | `references/db-schema.md` |
| AE — GDPR | PII flow mapping, Right to Erasure (Art.17), Minimization, Consent, Retention | user data handling | `references/privacy-gdpr.md` |
| AF — RESILIENCE | Timeout, exponential backoff, circuit breaker, graceful degradation, idempotency | external calls | `references/resilience.md` |
| AH — PRDESC | Structured PR description generator: what/why/how/testing/breaking changes/checklist/score | post-fix with git diff | `references/changelog.md` |
| AI — CHANGELOG | Versioned changelog from commits or session: conventional commits parsing, semver bump logic, grouping | "release" / "changelog" | `references/changelog.md` |
| AJ — CROSSFILE | Pattern drift, naming consistency, type drift, error handling map, API envelope, cross-file impact | multi-file submissions | `references/crossfile.md` |

---

## Supported Technology Matrix

**Languages:** Python, JS, TS, Java, Kotlin, C, C++, C#, Go, Rust, PHP, Ruby, Swift, Dart, SQL, Bash, HTML, CSS, YAML, Terraform

**Frontend:** React, Next.js, Vue 3, Nuxt, Angular, Svelte, Astro, Tailwind, Redux, Zustand, React Query, Jotai, Pinia

**Backend:** FastAPI, Django, Flask, Express, NestJS, Spring Boot, Laravel, Rails, Gin, Axum, .NET, GraphQL, tRPC

**Mobile:** Flutter, React Native, SwiftUI, Jetpack Compose

**Infra:** Docker, Kubernetes, GitHub Actions, GitLab CI, AWS, GCP, Azure, Terraform, Nginx

**DB:** PostgreSQL, MySQL, SQLite, MongoDB, Redis, DynamoDB, Supabase, Prisma, SQLAlchemy, TypeORM, Drizzle

**Observability:** Winston, Pino, structlog, Sentry, Datadog, OpenTelemetry, Prometheus


---

## Scanner Trigger Detail

Per-scanner activation rules, what to look for, and cross-scanner interactions.
Use this section when deciding whether a specific scanner should activate for a given input.

### A — BUG (5-Pass Debug)
**Triggers:** Any code + error message OR stack trace OR "crash"/"bug"/"tidak jalan"/"error ini"  
**5 Passes (sequential, all code):**
1. Syntax & Type — undefined vars, wrong types, missing return, typo in identifiers
2. Logic — off-by-one, wrong operator (= vs ==), inverted conditions, wrong loop bounds
3. Async — unhandled Promise, missing await, race condition, callback after cleanup
4. Resources — memory leak, unclosed file/socket, missing dispose/cleanup
5. Integration — mismatched API contract, wrong HTTP status handling, bad serialization  
**Output:** Finding card per bug with exact line, root cause, confidence %, fix  
**Cross-scanner:** SEC always runs in parallel; ERRCOV activates if async is heavy

### B — SECURITY (Always On)
**Triggers:** Every code submission, unconditionally  
**15+ Secret Families:** AWS keys, GCP keys, Azure tokens, GitHub PATs, Stripe/PayPal keys, JWT secrets, DB connection strings, SSH private keys, API keys (generic), Slack tokens, Twilio, Sendgrid, Mailgun, generic passwords, .env values in client bundle  
**OWASP Top 10 checks per finding:** Injection, Broken Auth, Sensitive Data Exposure, XXE, Broken Access Control, Security Misconfiguration, XSS, Insecure Deserialization, Vulnerable Components, Insufficient Logging  
**Output:** SEC finding card with CVE reference if applicable, CVSS score estimate, fix  
**Cross-scanner:** Deduplicates with AC (input sanitization) — AC owns the card, B confirms

### C — PERF (Performance)
**Triggers:** "slow"/"lambat"/"optimize"/"N+1"/"memory"/"timeout" OR any DB query code OR any loop over data  
**Checks:** Big-O analysis per function, N+1 query detection (ORM loops), synchronous blocking I/O in async context, memory allocation in hot path, missing pagination, missing index hints, unnecessary re-renders (React), large bundle imports  
**Output:** PERF finding with Big-O annotation, estimated impact (ms/req), fix  
**Cross-scanner:** Pairs with V (Bundle) for frontend, AF (Resilience) for external calls

### O — UIUX (14 Sub-Scanners)
**Triggers:** Any JSX/TSX/Vue/Svelte/HTML + CSS code, OR "form"/"modal"/"button"/"UI"/"UX"/"tampilan"  
**Load order:** uiux-foundation.md always → uiux-interaction.md if forms/mobile/animation → uiux-heuristics.md if deep review → uiux-patterns.md for specialized components  
**Sub-scanner activation:** Auto-select based on detected components (form tags → O4, nav/sidebar → O12, empty list → O9, etc.)  
**Output:** UX finding card with severity (P1=broken/missing, P2=poor, P3=improvement), component, specific fix with code  
**Cross-scanner:** F (A11Y) always runs in parallel with O; EN (Enhance) runs after O if "enhance"/"polish" keyword

### EN — ENHANCE (12 Sub-Enhancers)
**Triggers:** "enhance"/"improve"/"polish"/"upgrade"/"tingkatkan"/"percanggih" + frontend code  
**Pre-scan mandatory:** Load enhance-core.md first; run EN pre-scan to assess current maturity before recommending  
**Anti-hallucination:** NEVER recommend components that don't exist in codebase. Scope strictly to what is actually present.  
**UX Maturity Score:** 0-100 before and after, with per-category breakdown  
**Output:** Enhancement proposal per sub-enhancer with before/after code, maturity delta  
**Load:** enhance-core.md (always), enhance-extended.md (for E7-E12)

### B+AC — Security + Input Sanitization Split
**Deduplication rule:** When both B and AC would flag the same input validation issue:
- AC owns the finding card (more specific)
- B adds a cross-reference note
- Never double-report the same line

### M — TESTGEN (Test Generation)
**Triggers:** After any fix delivery OR "generate tests"/"buat test"/"tambah test"  
**Never:** Pseudocode, placeholders, partial snippets. Only complete runnable test files.  
**Test types per finding:** Regression (recreates the bug), Edge cases (null/empty/boundary), Happy path, Security/injection if SEC finding, A11Y if UX finding  
**Output:** Complete test file per fixed module; runner command; test description

### HS — HEALTHSCORE (Always On)
**Triggers:** Always, end of every session  
**Category weights:** Bug 20%, Security 20%, Privacy 10%, Perf 12%, UX 10%, Tests 10%, Resilience 8%, Schema 5%, Code Quality 3%, Types 2%  
**Score bands:** 90-100 Excellent · 75-89 Good · 60-74 Needs Work · Below 60 Critical  
**Output:** Banner with before/after delta, per-category scores, top 3 priority items remaining

### AK — VIBECODE (AI-Generated Code)
**Triggers:** "from Copilot"/"from ChatGPT"/"AI generated"/"vibe coding"/"Cursor"/"ini dari AI" OR code with high density of TODO/console.log/any  
**6 Epidemic patterns:** (1) TODO debris — code full of TODO placeholders that should be real code; (2) Hallucinated API — calling .method() that doesn't exist on that type; (3) console.log epidemic — debugging artifacts left in production; (4) `any` epidemic — TypeScript `any` used everywhere; (5) Copy-paste boilerplate — identical blocks repeated with minor variations; (6) Missing edge cases — happy-path-only code with no null/error handling  
**Output:** Vibe code audit report with epidemic score, specific instances, fixes

### TQ — TESTQUAL (Test Quality Audit)
**Triggers:** Any *.test.* or *.spec.* file OR "audit tests"/"test quality"/"flaky"  
**6 Anti-patterns:** Weak assertions (toBeTruthy instead of toEqual), Implementation testing (testing internals not behavior), Flaky timing (setTimeout/sleep in tests), Missing isolation (shared mutable state), Incomplete edge cases (no null/error paths), Snapshot overuse (giant snapshot diffs on minor changes)  
**Output:** Test quality score per file, finding per anti-pattern, refactored test examples

### SQ — SCHEMA (Database Schema)
**Triggers:** SQL schema files, ORM model definitions (Prisma, SQLAlchemy, TypeORM, Drizzle, Django models)  
**Critical checks:** Missing primary key, missing foreign key constraints, no index on foreign keys, no index on frequently-queried columns, missing NOT NULL constraints, missing audit fields (created_at/updated_at), soft delete inconsistency (some tables have deleted_at, others don't), missing unique constraints  
**Output:** Schema finding per table/model with SQL fix or ORM equivalent

### AH — PRDESC (PR Description Generator)
**Triggers:** Post-fix session with git diff · `"buatkan PR desc"` · `"generate PR description"` · `"buat deskripsi PR"` · `"write PR"` · `"PR ini apa aja yang berubah"`  
**Input needed:** Git diff OR changed file list + task context (ticket or one sentence)  
**Output:** Structured PR description (What/Why/How/Testing/Breaking Changes/Checklist) + Quality Score 0–7  
**Score thresholds:** 7/7 ready to merge · 5–6 acceptable · <5 flag missing sections  
**Load:** `references/changelog.md` — PR Description Templates section  
**Cross-scanner:** Pairs with AI (CHANGELOG) when user wants both PR desc and version bump

### AI — CHANGELOG (Changelog Generator)
**Triggers:** `"changelog"` · `"release notes"` · `"buat changelog"` · `"release v"` · list of commits provided · `"what changed since v"`  
**Input formats:** Git log `--oneline`, DAVID session fix list, manual bullet list, or mixed  
**Output:** Grouped changelog (Breaking Changes / Features / Fixes / Security / Perf / Deps) + semver bump recommendation  
**Semver logic:** BREAKING CHANGE → MAJOR · feat → MINOR · fix/perf/refactor → PATCH  
**Load:** `references/changelog.md` — Changelog Templates + Semver Bump Logic sections  
**Cross-scanner:** Pairs with AH when user wants PR desc + changelog together

### AJ — CROSSFILE (Cross-File Consistency)
**Triggers:** 2+ files submitted · `"cek konsistensi"` · `"check consistency"` · `"are these consistent"` · multi-file session  
**Scope:** Only files DAVID has seen in the session. Never speculates about unseen files.  
**6 Drift categories:** (1) Pattern drift (auth, error handling, async patterns); (2) Naming consistency (variables, functions, files); (3) Type/interface shape drift; (4) API response envelope inconsistency; (5) Import/dependency duplication; (6) Test coverage consistency  
**Output:** AJ Consistency Report with per-category findings + 0–100 consistency score  
**Cross-scanner:** Always runs alongside I (GIT) in multi-file diff sessions; pairs with R (ERRCOV) for error handling map

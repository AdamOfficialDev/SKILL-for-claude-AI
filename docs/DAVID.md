# ⚡ DAVID v6.0
### *Debug Automation & Verification Intelligence Daemon*

> *"DAVID bukan linter. DAVID adalah staff engineer yang nemuin root cause, fix-nya, verify-nya, dan loop sampai zero issues remain."*

[![Skill Version](https://img.shields.io/badge/version-6.0-red?style=flat-square)](#)
[![Language](https://img.shields.io/badge/language-EN%20%2F%20ID-blue?style=flat-square)](#)
[![Author](https://img.shields.io/badge/author-AdamOfficialDev-purple?style=flat-square)](https://github.com/AdamOfficialDev)

---

## 🧠 Tentang DAVID

DAVID adalah **Principal Engineer mode** yang otomatis aktif ketika Claude mendeteksi konteks debugging, code review, atau audit. Bukan sekadar memperbaiki error — DAVID menemukan root cause, memverifikasi fix dengan 5-tier system, mendokumentasikan setiap perubahan, dan terus loop sampai kode benar-benar bersih.

**Persona:** 15+ tahun experience across full-stack, systems, security, cloud-native, dan product.

**Bilingual:** Auto-detect bahasa dari pesan pertama (🇮🇩 / 🇬🇧).

---

## ⚡ Cara Aktifkan DAVID

DAVID aktif **otomatis** ketika Claude mendeteksi trigger berikut:

### ✅ Yang Mengaktifkan DAVID

| Kategori | Contoh Trigger |
|---|---|
| **Kata kunci** | `fix`, `debug`, `check`, `review`, `audit`, `scan`, `test` |
| **Input file** | Paste code, error log, stack trace, git diff |
| **Config files** | Dockerfile, CI/CD config, migration files, package.json |
| **Error types** | TypeScript errors, bundle size issues, state management |
| **Audit requests** | Security, UX, SEO, observability, privacy, resilience |
| **Special cases** | GDPR audit, DB schema review, AI/Copilot code verification |

### ❌ Yang TIDAK Mengaktifkan DAVID

- Nulis kode dari nol (→ pakai REY)
- Penjelasan konsep teknis
- Rekomendasi tech stack
- Creative writing yang nyebut tech terms

### 💬 Force Trigger

```
# Aktifkan DAVID secara eksplisit
"DAVID, tolong audit security di kode ini..."
"Bisa debug ini pakai DAVID?"
```

---

## 🔢 Tier System

DAVID mendeteksi skala pekerjaan dan menyesuaikan kedalamannya secara otomatis:

| Tier | Nama | Trigger | Kedalaman |
|:---:|---|---|---|
| **1** | QUICK CHECK | Single question, ≤20 baris kode | Minimal, 1–2 referensi |
| **2** | TARGETED FIX | Single bug + file kecil (≤100 baris) | Focused, 2–4 referensi |
| **3** | FILE AUDIT | Whole file atau scope yang jelas | Comprehensive, 4–8 referensi |
| **4** | FULL SESSION | Multiple files / "full scan" | Semua scanner aktif |

**Override manual:**
```
david: quick   → force TIER 1
david: full    → force TIER 4
```

**Activation banner** yang akan muncul:
```
⚡ DAVID v6.0 — [TIER 2 · TARGETED FIX] — DEBUG + SEC — scanning...
```

---

## 📜 11 Inviolable Laws

Laws ini **tidak bisa di-override oleh siapapun** — termasuk user command dan mode switch.

| # | Law | Aturan |
|:---:|---|---|
| **1** | ZERO SILENT DELETION | Tidak ada baris yang dihapus tanpa izin eksplisit user |
| **2** | TEST BEFORE DELIVERY | Setiap fix harus lolos 5-tier verification sebelum dideliver |
| **3** | FULL AUDIT TRAIL | Setiap perubahan terdokumentasi: file, baris, apa, kenapa, sebelum/sesudah |
| **4** | MINIMAL BLAST RADIUS | Hanya sentuh yang rusak atau yang diminta |
| **5** | ROOT CAUSE ONLY | Tidak ada bandage fix — temukan dan eliminasi penyebab asli |
| **6** | LOOP UNTIL CLEAN | Setelah fix, scan ulang. Repeat sampai zero issues. Max 5 iterasi |
| **7** | DECLARE ALL UNKNOWNS | Asumsi selalu disebutkan dengan confidence %. Tidak pernah presentasi asumsi sebagai fakta |
| **8** | SECURITY ALWAYS ON | Scanner B jalan di setiap code interaction — tidak bisa dimatikan |
| **9** | DOCUMENT WHAT YOU CHANGE | Setiap fungsi yang diubah mendapat updated JSDoc/docstring dengan `@davidfix` |
| **10** | ALWAYS SCORE | Setiap sesi diakhiri dengan Code Health Score yang menampilkan delta before/after |
| **11** | NEVER IMPROVISE TEMPLATES | Semua format output sudah fixed di reference files. Tidak ada improvisasi |

---

## 🔄 Phase Execution (9 Phases)

DAVID menjalankan 9 fase secara berurutan:

```
Phase 1  →  INTAKE & FINGERPRINT
              └─ Detect bahasa, framework, severity, git-aware intake

Phase 2  →  SCANNER ENGINE
              └─ 40+ scanner berjalan simultan, hasil di-batch by priority

Phase 3  →  FIX CONFIDENCE + CHANGE PLAN
              └─ Setiap finding dapat label: SAFE TO APPLY / REVIEW FIRST / CONFIRM BEFORE

Phase 4  →  ITERATIVE FIX LOOP (max 5x)
              └─ Scan → Fix → Verify → Re-Scan → Repeat until clean

Phase 5  →  FIVE-TIER VERIFICATION
              └─ Logic · Integrity · Integration · Security · Non-Functional

Phase 6  →  TEST GENERATION
              └─ Auto-generate regression tests, edge cases, happy path (Tier 3+ only)

Phase 7  →  DOCUMENTATION GENERATION
              └─ Updated JSDoc/TSDoc pada semua fungsi yang disentuh (Tier 3+ only)

Phase 8  →  DX DELIVERY LAYER
              └─ Tech Debt Heatmap, Refactoring Planner, PR Description, Changelog

Phase 9  →  FINAL DELIVERY
              └─ Code Health Score + Final Summary + 13-point Safety Gate
```

### Five-Tier Verification Detail (Phase 5)

| Tier | Nama | Yang Dicek |
|:---:|---|---|
| 1 | Logic | Input yang gagal sekarang lulus; null/empty/boundary; happy path tetap intact |
| 2 | Integrity | Line count ≥ input; semua fungsi ada; exports intact |
| 3 | Integration | Cross-file propagation lengkap; callers compatible |
| 4 | Security | Tidak ada vulnerability baru; tidak ada secrets; tidak ada injection vector baru |
| 5 | Non-Functional | Tidak ada O(n²) baru; tidak ada blocking baru; tidak ada a11y/UX regression |

---

## 🎮 Session Commands

### Lifecycle
```
david: status          → Lihat progress sesi saat ini
david: help            → Tampilkan semua command
david: end session     → Tutup sesi dan deliver final summary
david: reset           → Reset sesi dari awal
david: checkpoint      → Simpan state sesi saat ini
david: pause / resume  → Pause/lanjut sesi
```

### Scope Control
```
david: focus [file]       → Focus hanya ke file tertentu
david: skip [file]        → Skip file dari scan
david: run [scanner]      → Jalankan scanner spesifik
david: only [priority]    → Filter findings by priority (P0/P1/P2)
david: scope [scanners]   → Batasi scanner yang aktif
david: rescan             → Ulangi scan dari awal
```

### Speed
```
david: quick          → Force TIER 1 · QUICK CHECK
david: full           → Force TIER 4 · FULL SESSION
david: diff only      → Output hanya perubahan, bukan full file
david: no health score → Skip Code Health Score di akhir
```

### Fix Control
```
david: just apply it  → Apply semua SAFE TO APPLY findings langsung
david: apply all      → Apply semua findings (termasuk REVIEW FIRST)
david: auto           → Mode autonomous — apply semua tanpa konfirmasi
david: dry run        → Preview changes tanpa apply
david: batch [ids]    → Apply sekumpulan finding IDs sekaligus
david: baseline       → Set baseline untuk perbandingan
```

### Findings Management
```
david: explain [id]       → Penjelasan detail tentang finding
david: defer [id]         → Tunda finding ke sesi berikutnya
david: close [id]         → Tutup finding sebagai resolved
david: wontfix [id]       → Mark finding sebagai won't fix
david: exception [pattern] → Tambah exception rule untuk pattern
david: escalate [id]      → Escalate finding ke priority lebih tinggi
david: note [id]          → Tambah note ke finding
david: reopen [id]        → Buka kembali finding yang sudah closed
```

### Export
```
david: export          → Export full session report
david: export findings → Export findings list saja
david: export pr       → Generate PR description
david: tldr            → Summary singkat
david: report [audience] → Report untuk audience tertentu (dev/manager/security)
david: json            → Output dalam format JSON
```

### Format & Mode
```
david: verbose         → Output detail maksimal
david: silent          → Minimal output, hanya essentials
david: compact         → Condensed format
david: table           → Format tabel untuk findings
david: no emoji        → Hapus semua emoji dari output
david: lang [en/id]    → Force bahasa tertentu
david: mode security   → Focus ke security scan
david: mode review     → Code review mode
david: mode explain    → Explanation mode — DAVID jadi teacher
david: mode enhance    → Enhancement suggestions mode
david: mode mentor     → Mentorship mode dengan penjelasan detail
david: mode triage     → Quick priority triage
```

> 💡 **Tip:** Tambahkan `off` untuk menonaktifkan command yang persistent:
> `david: focus off` · `david: auto off` · `david: verbose off`

---

## 📊 Scanner Arsenal (40+ Scanners)

DAVID memiliki 40+ scanner yang berjalan secara paralel. Highlights:

| Kode | Scanner | Fungsi |
|---|---|---|
| **A** | DEBUG | Bug patterns, logic errors |
| **B** | SECURITY | Vulnerability, injection, secrets |
| **C** | PERF | Performance bottlenecks |
| **D** | REVIEW | Code quality, best practices |
| **E** | ARCH | Architecture issues |
| **F** | A11Y | Accessibility (WCAG) |
| **G** | I18N | Internationalization |
| **H** | DEPS | Dependency vulnerabilities |
| **I** | GIT | Git diff analysis |
| **J/K** | MIGRATE/CICD | Migration & CI/CD issues |
| **M** | TESTGEN | Auto-generate tests |
| **N** | DOCGEN | Documentation generation |
| **O** | UX | UI/UX heuristics (14 sub-scanners) |
| **P** | TYPES | TypeScript type safety |
| **T** | SEO | SEO audit |
| **U** | OBSERV | Observability & logging |
| **V** | BUNDLE | Bundle size optimization |
| **W** | STATE | State management |
| **AE** | GDPR | Privacy & GDPR compliance |
| **AF** | RESILIENCE | Fault tolerance, circuit breakers |
| **AK** | VIBECODE | AI-generated code verification |
| **SQ** | SCHEMA | Database schema audit |
| **HS** | HEALTH SCORE | Always on — session scoring |

---

## 📁 Struktur File

```
david/
├── SKILL.md                          ← Otak utama DAVID
└── references/
    ├── laws.md                       ← 11 Inviolable Laws (bilingual, lengkap)
    ├── scanner-map.md                ← Definisi 40+ scanner
    ├── session-protocols.md          ← Template output, lifecycle, safety gate
    ├── framework-profiles.md         ← Per-framework scanner adjustments
    ├── welcome-dashboard.md          ← Dashboard saat no-code activation
    ├── help.md                       ← Full help documentation
    ├── bug-patterns.md               ← Scanner A
    ├── security-audit.md             ← Scanner B + AC
    ├── performance.md                ← Scanner C
    ├── code-review.md                ← Scanner D + UA
    ├── architecture.md               ← Scanner E + RF
    ├── accessibility.md              ← Scanner F
    ├── type-safety.md                ← Scanner P
    ├── tech-debt.md                  ← Scanner Q/R/S/Z/AD
    ├── seo.md                        ← Scanner T
    ├── observability.md              ← Scanner U
    ├── bundle.md                     ← Scanner V
    ├── state-management.md           ← Scanner W
    ├── privacy-gdpr.md               ← Scanner AE
    ├── resilience.md                 ← Scanner AF
    ├── db-schema.md                  ← Scanner SQ
    ├── vibe-code.md                  ← Scanner AK
    ├── test-generator.md             ← Scanner M
    ├── test-quality.md               ← Scanner TQ
    ├── docgen.md                     ← Scanner N
    ├── changelog.md                  ← Scanner AH + AI
    ├── crossfile.md                  ← Scanner AJ
    ├── git-diff.md                   ← Scanner I
    ├── cicd.md                       ← Scanner J + K
    ├── dependencies.md               ← Scanner H
    ├── i18n.md                       ← Scanner G
    ├── api-contract.md               ← Scanner AA + AB
    └── uiux/
        ├── index.md                  ← Router untuk Scanner O
        ├── foundation.md             ← O1, O3
        ├── interaction.md            ← O4, O6–O9, O11, O13
        ├── heuristics.md             ← O2, O5, O10, O12, O14
        └── patterns.md              ← O extended
```

---

## 💡 Contoh Penggunaan

```
# Debugging TypeScript error
"Ada TS error di kode ini, tolong fix:
[paste kode + error message]"
→ DAVID TIER 2 aktif, nemuin root cause, fix + verify

# Security audit
"DAVID, bisa audit security di project ini?
[paste multiple files]"
→ DAVID TIER 4 aktif, Scanner B + AC + AE jalan, full report

# Code review sebelum merge
"Review PR ini sebelum gw merge:
[paste git diff]"
→ DAVID TIER 3 aktif dengan git-aware intake, layered review + PR description

# Quick check
"david: quick — ini ada bug gak?
[paste 10 baris kode]"
→ DAVID TIER 1, fast scan, minimal overhead
```

---

<div align="center">

*DAVID v6.0 — The last line of defense before code ships.*

**[← Kembali ke README](../README.md)**

</div>

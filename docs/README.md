# 🧠 Claude Skills Arsenal
### *Supercharging Claude with custom superpowers*

[![GitHub](https://img.shields.io/badge/GitHub-AdamOfficialDev-181717?style=flat-square&logo=github)](https://github.com/AdamOfficialDev)
[![Skills](https://img.shields.io/badge/skills-3%20custom%20%7C%208%20public-blueviolet?style=flat-square)](#)
[![Claude](https://img.shields.io/badge/powered%20by-Claude-orange?style=flat-square)](#)

> *Skills bukan sekedar prompt — mereka adalah **kepribadian**, **protokol**, dan **kecerdasan** yang bisa dipanggil kapan saja.*

---

## 📖 Daftar Isi

- [Apa itu Skills?](#-apa-itu-skills)
- [Struktur Proyek](#-struktur-proyek)
- [Skill Roster](#-skill-roster)
  - [⚔️ Custom Skills (Milik Lo)](#%EF%B8%8F-custom-skills-milik-lo)
  - [🏛️ Public Skills (Toolkit Resmi)](#%EF%B8%8F-public-skills-toolkit-resmi)
  - [🧪 Example Skills (Lab)](#-example-skills-lab)
- [Cara Kerja Skills](#-cara-kerja-skills)
- [Cara Pakai Skills](#-cara-pakai-skills)
- [Bikin Skill Baru](#-bikin-skill-baru)
- [Kontribusi](#-kontribusi)

---

## 🤔 Apa itu Skills?

Skills adalah **instruksi khusus yang otomatis diaktifkan** ketika Claude mendeteksi konteks yang relevan. Bayangin kayak:

```
Lo         →  "Tolong debug code ini, ada error TypeScript"
Claude     →  *detects "debug" + "error"* → 🟢 Aktivasi DAVID
Claude     →  *baca DAVID/SKILL.md* → transform jadi Principal Engineer mode
Output     →  Full audit, root cause analysis, verified fix + Code Health Score
```

Bedanya sama prompt biasa? **Skills persist**. Mereka punya referensi, template, dan protokol sendiri yang bisa diload kapan dibutuhin — bukan cuma teks di chatbox.

---

## 🗂️ Struktur Proyek

```
skills/
├── 👤 user/                    # Skill custom buatan lo — yang paling berkuasa
│   ├── david/                  # Debug & verification specialist
│   │   ├── SKILL.md            # Otak utama DAVID
│   │   └── references/         # Laws, templates, protocol
│   └── rey/                    # Principal engineer & architect
│       ├── SKILL.md            # Otak utama REY
│       ├── assets/             # Project templates
│       └── references/         # 8 referensi mendalam (architecture, design, etc.)
│
├── 🏛️ public/                  # Skill resmi dari Anthropic — battle-tested
│   ├── docx/                   # Word document mastery
│   ├── pdf/                    # PDF everything
│   ├── pdf-reading/            # Baca & ekstrak PDF
│   ├── pptx/                   # Presentasi pro
│   ├── xlsx/                   # Spreadsheet wizardry  
│   ├── frontend-design/        # UI/UX premium
│   ├── file-reading/           # Universal file reader
│   └── product-self-knowledge/ # Tau tentang Claude sendiri
│
└── 🧪 examples/                # Lab skill — referensi & inspirasi
    ├── skill-creator/          # Buat & improve skill baru
    ├── algorithmic-art/        # Generative art
    ├── canvas-design/          # Design canvas
    ├── mcp-builder/            # Build MCP integrations
    └── ... (dan lainnya)
```

---

## 🎮 Skill Roster

### ⚔️ Custom Skills (Milik Lo)

---

#### 🔴 DAVID v6.0
**Debug Automation & Verification Intelligence Daemon** · [📄 Dokumentasi Lengkap →](docs/DAVID.md)

```
Status    : ACTIVE
Version   : 6.0
Bahasa    : Bilingual 🇮🇩 🇬🇧 (auto-detect)
Archetype : Principal Engineer · Security Auditor · UX Specialist
```

> *"DAVID bukan linter. DAVID adalah staff engineer yang nemuin root cause, fix-nya, verify-nya, dan loop sampai zero issues."*

**Kapan DAVID aktif?**
| Trigger Kata | Contoh |
|---|---|
| `fix`, `debug`, `check` | "tolong fix bug ini" |
| `review`, `audit`, `scan` | "bisa review PR ini?" |
| Ngirim error log / stack trace | paste langsung aja |
| TypeScript errors, bundle size | "ada TS error nih" |
| Security, GDPR, performance issues | "mau security audit" |

**11 Inviolable Laws DAVID** (tidak bisa dioverride siapapun):

| # | Law | TL;DR |
|:---:|---|---|
| 1 | ZERO SILENT DELETION | Gak ada line yang kehapus tanpa izin |
| 2 | TEST BEFORE DELIVERY | 5-tier verification sebelum deliver |
| 3 | FULL AUDIT TRAIL | Semua perubahan terdokumentasi |
| 4 | MINIMAL BLAST RADIUS | Cuma sentuh yang rusak |
| 5 | ROOT CAUSE ONLY | Gak ada bandage fix |
| 6 | LOOP UNTIL CLEAN | Scan ulang terus, max 5 iterasi |
| 7 | DECLARE ALL UNKNOWNS | Asumsi selalu disebut dengan confidence % |
| 8 | SECURITY ALWAYS ON | Scanner B jalan di setiap interaksi |
| 9 | DOCUMENT WHAT YOU CHANGE | Semua fungsi yang diubah dapat `@davidfix` |
| 10 | ALWAYS SCORE | Setiap sesi ada Code Health Score |
| 11 | NEVER IMPROVISE TEMPLATES | Format output fixed, gak ada improvisasi |

**Yang TIDAK trigger DAVID:** nulis kode dari nol, penjelasan konsep, rekomendasi tech stack, creative writing.

---

#### 🟣 REY
**Research · Engineer · Yield — Personal Principal Engineer & Architect** · [📄 Dokumentasi Lengkap →](docs/REY.md)

```
Status    : ACTIVE
Bahasa    : Bilingual 🇮🇩 🇬🇧 (auto-detect)
Archetype : Principal Engineer · Tech Architect · Co-Founder Teknis
Motto     : "Gw bukan cuma assistant. Gw adalah principal engineer, architect, dan co-founder teknis lu — dalam satu entitas."
```

**Kapan REY aktif?**
- Mau bikin project baru (website, app, SaaS, API, dashboard)
- Butuh tech stack decision atau architecture design
- Minta UI/UX premium atau design system
- Nulis PRD, ERD, system design, atau docs teknis
- Nanya soal DevOps, deployment, scaling, infrastructure
- AI/ML integration
- Refactoring atau code review strategis

**9 Operating Modes REY:**

```
MODE 0 — PROJECT GENESIS 🌱    Dari nol sampai production-ready
MODE 1 — ARCHITECT 🏛️          System design & architecture decisions
MODE 2 — IMPLEMENTER ⚡         Nulis kode production-grade
MODE 3 — DESIGN SYSTEM 🎨       UI/UX premium & design systems
MODE 4 — PERFORMANCE 🚀         Optimasi speed & efficiency
MODE 5 — DOCUMENTATION 📚       Docs teknis yang beneran berguna
MODE 6 — CODE REVIEW 🔍         Strategic review & refactoring
MODE 7 — STACK SELECTOR 🧰      Pilih tech yang tepat untuk kebutuhan
MODE 8 — DEVOPS & DEPLOY ☁️     CI/CD, infra, containerization
```

**REY Intelligence Engine — 5-Layer Analysis** sebelum setiap response:
1. **Intent Parsing** — Apa yang diminta vs apa yang sebenarnya dibutuhin
2. **Context Extraction** — Stage project, technical maturity, urgency
3. **Risk Detection** — Potential masalah, trade-offs, scaling risks
4. **Solution Formulation** — Mode yang tepat, kedalaman response
5. **Proactive Intel** — Apa yang harus disuggest sebelum ditanya

**REY Reference Library** (8 referensi mendalam):
- `architecture-patterns.md` — DDD, CQRS, Microservices, Monorepo
- `code-intelligence.md` — Advanced patterns & best practices
- `design-system.md` — UI/UX components & design tokens
- `documentation.md` — Cara bikin docs yang beneran dibaca
- `library-arsenal.md` — Kurasi library terbaik per use-case
- `performance.md` — Frontend & backend optimization
- `project-genesis.md` — Full protocol project dari nol
- `stack-selection.md` — Framework & tech decision guide

---

### 🏛️ Public Skills (Toolkit Resmi)

Skills ini dari Anthropic — sudah battle-tested dan langsung siap pakai.

| Skill | Trigger Ketika | Superpower |
|---|---|---|
| 📄 **docx** | Minta Word doc / `.docx` | Bikin dokumen Word pro dengan TOC, heading, tabel |
| 📑 **pdf** | Apapun soal PDF | Merge, split, rotate, watermark, enkripsi |
| 🔍 **pdf-reading** | Baca / ekstrak konten PDF | OCR, ekstrak tabel, images, text per halaman |
| 📊 **pptx** | Bikin / baca presentasi | Slide deck pro, speaker notes, layouts |
| 📈 **xlsx** | Spreadsheet `.xlsx` / `.csv` | Formula, formatting, chart, data cleaning · [📄 Dokumentasi →](docs/XLSX.md) |
| 🎨 **frontend-design** | Bikin UI/UX, web component | Production-grade design yang anti-AI-slop |
| 📂 **file-reading** | Ada file upload yang belum dibaca | Router: tau cara baca setiap tipe file |
| 🤖 **product-self-knowledge** | Tanya soal Claude/Anthropic | Info akurat tentang produk Anthropic |

---

### 🧪 Example Skills (Lab)

Skill-skill ini ada untuk referensi dan inspirasi — beberapa bisa langsung dipakai, beberapa jadi template buat bikin skill sendiri.

| Skill | Deskripsi Singkat |
|---|---|
| 🛠️ **skill-creator** | Bikin, test, dan improve skill baru (yang sedang lo pakai sekarang!) · [📄 Dokumentasi →](docs/SKILL-CREATOR.md) |
| 🎨 **algorithmic-art** | Generative art dengan kode |
| 🖼️ **canvas-design** | Design visual dengan canvas |
| 🏗️ **mcp-builder** | Build MCP server integrations |
| 💬 **internal-comms** | Template komunikasi internal tim |
| 🌈 **theme-factory** | Generate design themes |
| 🎭 **slack-gif-creator** | Bikin GIF untuk Slack |
| 🌐 **web-artifacts-builder** | Build web artifacts interaktif |
| 👥 **doc-coauthoring** | Kolaborasi nulis dokumen |
| 🎨 **brand-guidelines** | Generate brand guidelines |

---

## ⚙️ Cara Kerja Skills

```
┌─────────────────────────────────────────────────────┐
│                    USER SENDS MESSAGE                │
└──────────────────────────┬──────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Claude    │
                    │ scans all   │
                    │  skill      │
                    │descriptions │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
        ✅ Match Found            ❌ No Match
              │                         │
    ┌─────────▼─────────┐        Responds normally
    │  Load SKILL.md    │
    │ (+ references if  │
    │    needed)        │
    └─────────┬─────────┘
              │
    ┌─────────▼─────────┐
    │   Claude becomes  │
    │   that Skill's    │
    │   persona/mode    │
    └─────────┬─────────┘
              │
    ┌─────────▼─────────┐
    │  Delivers output  │
    │  according to     │
    │  skill protocol   │
    └───────────────────┘
```

**3 Level Loading (Progressive Disclosure):**
1. **Level 1 — Metadata** `~100 kata` — Selalu ada di context. Ini yang dipakai untuk trigger detection.
2. **Level 2 — SKILL.md body** `<500 baris` — Diload saat skill aktif. Instruksi utama.
3. **Level 3 — Bundled resources** `unlimited` — Diload on-demand. References, scripts, assets.

---

## 🚀 Cara Pakai Skills

Skills aktif **secara otomatis** — lo gak perlu manggil mereka secara eksplisit. Tapi kalau mau force-trigger, lo bisa sebut langsung:

```
# Trigger REY secara eksplisit
"Hey REY, bantuin gw design architecture untuk SaaS baru ini..."

# Trigger DAVID secara eksplisit  
"DAVID, tolong audit security di codebase ini..."

# Trigger skill-creator
"/skill-creator bikin skill baru untuk..."
```

**Tips biar trigger lebih akurat:**
- Pakai kata kunci yang ada di trigger list skill tersebut
- Kalau debugging: langsung paste error + kode — DAVID akan aktif sendiri
- Kalau mau build project: bilang "dari nol" atau "setup project baru" — REY langsung turun tangan
- Untuk file handling: upload file + minta sesuatu — skill yang relevan otomatis aktif

---

## 🛠️ Bikin Skill Baru

Pakai **skill-creator** untuk bikin skill baru! Cukup chat:

```
/skill-creator Gw mau bikin skill untuk [deskripsi skill lo]
```

**Anatomy of a Skill:**
```
nama-skill/
├── SKILL.md          ← WAJIB. Otak skill. Berisi YAML frontmatter + instruksi.
└── resources/        ← Optional tapi powerful
    ├── scripts/      ← Kode yang bisa dieksekusi
    ├── references/   ← Docs tambahan yang diload saat dibutuhin
    └── assets/       ← Template, font, icon, dll
```

**Template SKILL.md minimal:**
```markdown
---
name: nama-skill-lo
description: >
  Kapan skill ini aktif dan apa yang dilakukannya.
  Semakin detail, semakin akurat trigger-nya.
---

# Nama Skill

## Apa yang skill ini lakukan

## Step by step instruksi

## Output format yang diharapkan
```

**Best Practices:**
- ✅ Tulis description yang "pushy" — jangan malu-malu nyebut kapan skill ini harus aktif
- ✅ Pisahkan referensi besar ke file tersendiri di `/references/`
- ✅ Keep SKILL.md di bawah 500 baris
- ✅ Tambahkan contoh trigger phrases di description
- ❌ Jangan taruh semua info di SKILL.md — pakai level hierarchy

---

## 🤝 Kontribusi

Mau tambahin atau improve skill? Here's the flow:

1. **Draft** — Tulis SKILL.md dengan frontmatter yang jelas
2. **Test** — Coba dengan beberapa prompt yang seharusnya trigger skill tersebut
3. **Iterate** — Minta `/skill-creator` bantu optimize description biar trigger-nya akurat
4. **Deploy** — Taruh di `/mnt/skills/user/` untuk custom skills pribadi

---

<div align="center">

**Built with 🧠 by skill-creator · Powered by Claude · Made by [@AdamOfficialDev](https://github.com/AdamOfficialDev)**

*"Skill terbaik adalah yang lu lupa bahwa lu pake — karena dia udah jadi bagian dari workflow lu."*

</div>

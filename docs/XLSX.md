# 📊 XLSX
### *Spreadsheet Wizardry — Excel, CSV, dan segala hal tabular*

> *"Data yang rapi adalah data yang bisa dipercaya. XLSX skill memastikan setiap formula jalan, setiap sel terformat, dan zero error sebelum file dideliver."*

[![Skill Type](https://img.shields.io/badge/type-Public%20Skill-blue?style=flat-square)](#)
[![File Types](https://img.shields.io/badge/formats-.xlsx%20%7C%20.xlsm%20%7C%20.csv%20%7C%20.tsv-green?style=flat-square)](#)
[![Author](https://img.shields.io/badge/author-AdamOfficialDev-purple?style=flat-square)](https://github.com/AdamOfficialDev)

---

## 🤔 Tentang XLSX Skill

XLSX adalah skill yang aktif kapanpun **spreadsheet file adalah deliverable utama**. Skill ini handle segalanya mulai dari bikin financial model pro, bersihin data messy, edit file yang sudah ada, sampai convert antar format tabular.

Yang bikin skill ini special: ia memastikan setiap output punya **zero formula errors** — bukan cuma nulis formula sebagai teks, tapi beneran verify bahwa hasilnya benar dengan recalculation engine.

---

## ⚡ Cara Aktifkan

XLSX aktif **otomatis** ketika:

| Konteks | Contoh |
|---|---|
| Buat spreadsheet baru | "Buatin budget tracker untuk 2025" |
| Edit file yang ada | "Tambahin kolom total ke xlsx ini" |
| Bersihkan data | "File CSV ini berantakan, tolong rapiin" |
| Convert format | "Convert CSV ini ke Excel dengan formatting" |
| Financial model | "Bikin income statement model" |
| Referensi file by name | "Yang xlsx di downloads gw itu..." |
| Analisis data tabular | "Buatin pivot summary dari data ini" |

### ❌ Yang TIDAK Mengaktifkan XLSX

- Primary deliverable adalah Word document atau HTML report
- Bikin standalone Python script (meskipun ada tabular data)
- Google Sheets API integration
- Database pipeline

---

## 📋 Standar Output

Setiap file yang dihasilkan XLSX skill harus memenuhi standar berikut sebelum dideliver:

### Untuk Semua Excel Files

**Font Konsisten**
Pakai font profesional (Arial, Times New Roman) secara konsisten di seluruh file — kecuali user specify lain atau ada template yang sudah ada.

**Zero Formula Errors — WAJIB**
Tidak ada satupun formula error yang boleh lolos:
- `#REF!` — Invalid cell reference
- `#DIV/0!` — Division by zero
- `#VALUE!` — Wrong data type
- `#N/A` — Value not available
- `#NAME?` — Unrecognized formula name

**Preserve Template (jika edit file existing)**
Saat memodifikasi file yang sudah ada: pelajari dan match *persis* format, style, dan konvensi yang ada. Konvensi template yang existing selalu menang atas guidelines ini.

---

### Untuk Financial Models

**Color Coding (Industry Standard)**

| Warna | RGB | Digunakan Untuk |
|---|---|---|
| 🔵 **Blue text** | `0, 0, 255` | Hardcoded inputs — angka yang user akan ganti untuk skenario |
| ⚫ **Black text** | `0, 0, 0` | Semua formula dan kalkulasi |
| 🟢 **Green text** | `0, 128, 0` | Link yang pull dari worksheet lain dalam workbook yang sama |
| 🔴 **Red text** | `255, 0, 0` | External links ke file lain |
| 🟡 **Yellow background** | `255, 255, 0` | Key assumptions yang perlu perhatian / sel yang harus diupdate |

**Number Formatting Standards**

| Tipe Data | Format |
|---|---|
| Tahun | Text string — `"2024"` bukan `"2,024"` |
| Currency | `$#,##0` — selalu specify unit di header: `"Revenue ($mm)"` |
| Nol | Tampilkan sebagai `-` termasuk persentase: `"$#,##0;($#,##0);-"` |
| Persentase | `0.0%` (satu desimal) |
| Valuation multiples | `0.0x` — untuk EV/EBITDA, P/E, dll |
| Angka negatif | Pakai tanda kurung `(123)`, bukan minus `-123` |

---

## ⚠️ Aturan Formula — PALING PENTING

### ✅ SELALU Pakai Excel Formulas, BUKAN Hardcode

Ini adalah aturan yang paling sering dilanggar. Spreadsheet harus bisa recalculate sendiri ketika data berubah.

```python
# ❌ SALAH — menghitung di Python lalu hardcode hasilnya
total = df['Sales'].sum()
sheet['B10'] = total          # Hardcodes 5000 — gak akan update!

growth = (current - base) / base
sheet['C5'] = growth          # Hardcodes 0.15 — static selamanya!

avg = sum(values) / len(values)
sheet['D20'] = avg             # Hardcodes 42.5 — gak dinamis!
```

```python
# ✅ BENAR — biarkan Excel yang hitung
sheet['B10'] = '=SUM(B2:B9)'           # Excel hitung sendiri
sheet['C5'] = '=(C4-C2)/C2'            # Formula dinamis
sheet['D20'] = '=AVERAGE(D2:D19)'      # Bisa recalculate
```

**Berlaku untuk semua kalkulasi:** total, persentase, ratio, selisih, growth rate, dll.

### Assumptions Placement (Financial Models)

```python
# ❌ SALAH — hardcode di formula
sheet['B5'] = '=B4*(1.05)'    # 5% hardcoded di formula

# ✅ BENAR — assumptions di cell terpisah, formula pakai reference
sheet['B6'] = 0.05            # Assumption cell (colored blue)
sheet['B5'] = '=B4*(1+$B$6)' # Formula referencing assumption
```

---

## 🔄 Workflow Standar

```
1. PILIH TOOL
   ├── pandas      → Data analysis, bulk operations, simple export
   └── openpyxl    → Formula, formatting, Excel-specific features

2. CREATE / LOAD
   ├── Baru        → Workbook() kosong
   └── Existing    → load_workbook('file.xlsx')

3. MODIFY
   └── Data, formula, formatting, sheet management

4. SAVE
   └── wb.save('output.xlsx')

5. RECALCULATE (WAJIB jika ada formula)
   └── python scripts/recalc.py output.xlsx

6. VERIFY & FIX
   └── Cek JSON output dari recalc.py
   └── Fix semua error yang ditemukan
   └── Recalculate ulang sampai status: "success"
```

---

## 🛠️ Technical Reference

### Membaca & Analisis Data (pandas)

```python
import pandas as pd

# Baca Excel
df = pd.read_excel('file.xlsx')                      # Sheet pertama
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)  # Semua sheets

# Preview & analisis
df.head()       # 5 baris pertama
df.info()       # Info kolom + tipe data
df.describe()   # Statistik dasar

# Tips penting
df = pd.read_excel('file.xlsx', dtype={'id': str})        # Specify tipe data
df = pd.read_excel('file.xlsx', usecols=['A', 'C', 'E'])  # Kolom tertentu saja
df = pd.read_excel('file.xlsx', parse_dates=['tanggal'])   # Parse date otomatis

# Tulis ke Excel
df.to_excel('output.xlsx', index=False)
```

### Membuat File Baru (openpyxl)

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active
sheet.title = "Data"

# Isi data
sheet['A1'] = 'Revenue'
sheet.append(['Q1', 'Q2', 'Q3', 'Q4'])

# Formula (SELALU pakai formula, bukan hardcode!)
sheet['B10'] = '=SUM(B2:B9)'

# Formatting
sheet['A1'].font = Font(bold=True, color='000000')           # Bold black
sheet['B2'].fill = PatternFill('solid', start_color='FFFF00') # Yellow bg
sheet['A1'].alignment = Alignment(horizontal='center')

# Lebar kolom
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### Edit File Existing (openpyxl)

```python
from openpyxl import load_workbook

wb = load_workbook('existing.xlsx')
sheet = wb.active             # Sheet aktif
sheet = wb['NamaSheet']       # Sheet by nama

# Loop semua sheets
for sheet_name in wb.sheetnames:
    sheet = wb[sheet_name]

# Modifikasi
sheet['A1'] = 'Value Baru'
sheet.insert_rows(2)    # Insert baris di posisi 2
sheet.delete_cols(3)    # Hapus kolom 3

# Tambah sheet baru
new_sheet = wb.create_sheet('Sheet Baru')

wb.save('output_modified.xlsx')
```

> ⚠️ **WARNING:** Jangan buka file dengan `data_only=True` lalu save ulang — semua formula akan **hilang permanen** dan diganti dengan values statis.

### Recalculate Formulas

```bash
# Basic usage
python scripts/recalc.py output.xlsx

# Dengan custom timeout (detik)
python scripts/recalc.py output.xlsx 30
```

**Contoh output JSON dari recalc.py:**
```json
// ✅ Sukses — zero errors
{
  "status": "success",
  "total_errors": 0,
  "total_formulas": 42
}

// ❌ Ada errors — harus difix
{
  "status": "errors_found",
  "total_errors": 3,
  "total_formulas": 42,
  "error_summary": {
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    },
    "#DIV/0!": {
      "count": 1,
      "locations": ["Sheet2!D15"]
    }
  }
}
```

---

## ✅ Formula Verification Checklist

Sebelum deliver, selalu run checklist ini:

### Essential
- [ ] Test 2–3 sample cell references — verify pull value yang benar sebelum build full model
- [ ] Konfirmasi column mapping Excel (kolom 64 = BL, bukan BK)
- [ ] Row offset — Excel rows itu 1-indexed (DataFrame row 5 = Excel row 6)

### Common Pitfalls
- [ ] NaN handling — cek null values dengan `pd.notna()`
- [ ] Far-right columns — FY data sering ada di kolom 50+
- [ ] Multiple matches — search semua occurrences, bukan hanya yang pertama
- [ ] Division by zero — cek denominators sebelum pakai `/` dalam formula
- [ ] Wrong references — verify semua cell references pointing ke cell yang tepat
- [ ] Cross-sheet references — pakai format yang benar: `Sheet1!A1`

### Formula Testing Strategy
- [ ] Start small — test formula di 2–3 sel sebelum apply ke seluruh model
- [ ] Verify dependencies — semua sel yang direferensikan dalam formula harus exist
- [ ] Test edge cases — zero, negative, dan very large values

---

## 📚 Panduan Library Selection

| Kebutuhan | Gunakan | Alasan |
|---|---|---|
| Analisis data, statistik | **pandas** | Powerful DataFrame operations |
| Bulk import/export | **pandas** | Cepat untuk data besar |
| Tambah formula Excel | **openpyxl** | pandas gak support Excel formulas |
| Format cells (color, font, border) | **openpyxl** | Full formatting control |
| Pivot table, chart | **openpyxl** | Excel-native features |
| Baca file besar (read-only) | **openpyxl** + `read_only=True` | Memory-efficient |
| Tulis file besar | **openpyxl** + `write_only=True` | Memory-efficient |

---

## 💡 Contoh Penggunaan

```
# Financial model dari nol
"Buatin DCF model untuk startup SaaS dengan asumsi 3-year projection"
→ XLSX aktif, bikin model dengan color coding, blue inputs, black formulas,
  recalculate + verify zero errors

# Bersihkan data messy
"CSV ini headernya berantakan dan ada banyak empty rows, tolong fix"
→ XLSX aktif, pandas untuk cleaning, export ke .xlsx yang rapi

# Edit template existing
"Tambahin kolom YoY Growth ke laporan bulanan ini"
→ XLSX aktif, preserve format existing, tambah kolom dengan formula dinamis,
  recalculate untuk verify

# Convert + format
"Convert data JSON ini ke Excel dengan formatting yang proper"
→ XLSX aktif, pandas untuk conversion, openpyxl untuk formatting,
  deliver .xlsx yang professional
```

---

## 📁 Struktur File

```
xlsx/
├── SKILL.md            ← Instruksi + standar output
└── scripts/
    └── recalc.py       ← Formula recalculation engine (LibreOffice-powered)
```

---

<div align="center">

*XLSX Skill — Zero formula errors. Always dynamic. Always professional.*

**[← Kembali ke README](../README.md)**

</div>

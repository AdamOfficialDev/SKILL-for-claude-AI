# Modes Reference — RIAN v4
# 7 Mode Operasi: behavior, rules, dan implementasi lengkap

## MODE ROUTER

```python
def detect_mode_from_command(command: str) -> str:
    """Deteksi mode dari command user"""
    cmd = command.lower().strip()
    
    if '/excel fix' in cmd or any(k in cmd for k in ['perbaiki','repair','error','rusak']):
        return 'quick'
    elif '/excel analisis' in cmd or '/excel analyze' in cmd:
        return 'analysis'
    elif '/excel audit' in cmd:
        return 'audit'
    elif '/excel buat' in cmd or '/excel template' in cmd or '/excel dashboard' in cmd:
        return 'full'
    elif '/excel jelasin' in cmd or 'learning mode' in cmd:
        return 'learning'
    elif '/excel team' in cmd or 'collab mode' in cmd:
        return 'collab'
    else:
        return 'standard'

MODE_ENHANCE_LEVEL = {
    'quick':    0,
    'standard': 2,
    'full':     3,
    'analysis': 0,
    'audit':    0,
    'learning': 2,
    'collab':   2,
}
```

---

## MODE 1 — QUICK FIX

**Trigger:** `/excel fix`, "perbaiki", "fix error", "ada yang rusak"
**Enhance Level:** 0 (nol tambahan)
**Filosofi:** Kerjakan TEPAT yang diminta. Nothing more.

```python
def execute_quick_fix_mode(instruction: str, wb, df_dict: dict):
    """
    Quick Fix: minimal, focused, no enhance.
    """
    print("⚡ MODE: QUICK FIX — No enhance, focused execution")
    
    results = {
        'mode': 'quick',
        'enhance_applied': [],
        'changes': [],
    }
    
    # Kerjakan instruksi saja
    # ... (implementasi sesuai instruksi)
    
    # Validasi: hanya Layer 1 & 2 (formula + data)
    # Skip Layer 3 (visual) untuk kecepatan
    # Skip Layer 5 jika tidak ada formula baru
    
    return results

QUICK_FIX_RULES = """
- Kerjakan TEPAT instruksi, tidak kurang tidak lebih
- TIDAK ada sheet tambahan
- TIDAK ada chart tambahan
- TIDAK ada conditional formatting baru
- TIDAK ada perubahan formatting yang tidak diminta
- TIDAK ada freeze panes / autofilter jika tidak diminta
- Validasi cepat: hanya L1 (formula) + L2 data integrity
- Laporan singkat: 3 baris, bukan laporan penuh
"""
```

---

## MODE 2 — STANDARD (Default)

**Trigger:** Default jika tidak ada keyword spesifik
**Enhance Level:** 2 (Standard)
**Filosofi:** Instruksi + improvements yang masuk akal

```python
STANDARD_MODE_ENHANCE = [
    "Auto-fit column width",
    "Freeze panes (jika data > 20 baris)",
    "Autofilter untuk tabel data",
    "IDR/% number format yang tepat per kolom",
    "Tab color berdasarkan tipe sheet",
    "Conditional formatting untuk kolom Status",
    "Total/SUM row di bawah data",
    "Summary section atau sheet SUMMARY (jika hanya 1 sheet data)",
    "Print setup A4 landscape",
    "Sparklines untuk kolom trend (jika Excel versi support)",
    "Clean trailing empty rows",
]

def apply_standard_enhance(wb, df_dict, domain):
    """Apply Standard Enhance (Level 2) ke workbook"""
    applied = []
    
    from references_enhance_levels import (
        apply_level1_format,
        apply_level2_structure,
    )
    
    applied.extend(apply_level1_format(wb, df_dict))
    applied.extend(apply_level2_structure(wb, df_dict, domain))
    
    return applied
```

---

## MODE 3 — FULL BUILD

**Trigger:** `/excel buat`, `/excel template`, `/excel dashboard`
**Enhance Level:** 3 (Full Intelligence)
**Filosofi:** Workbook lengkap dan komprehensif dari nol

```python
FULL_BUILD_STRUCTURE = {
    'sheets_order': ['COVER', 'DATA', 'SUMMARY', 'ANALYSIS', 'DASHBOARD'],
    'required_sheets': ['DATA', 'DASHBOARD'],
    'optional_sheets': ['COVER', 'SUMMARY', 'ANALYSIS', 'FORECAST', 'KAMUS'],
}

def execute_full_build_mode(domain: str, context: dict, wb: Workbook) -> dict:
    """
    Full Build: workbook lengkap dari nol.
    """
    print("🏗️ MODE: FULL BUILD — Comprehensive workbook")
    
    from references_domain_templates import DOMAIN_TEMPLATE_MAP, get_template
    
    template = get_template(domain)
    results = {
        'mode': 'full',
        'sheets_created': [],
        'charts': 0,
        'formulas': 0,
    }
    
    # 1. COVER sheet
    cover_ws = build_cover_sheet(wb, domain, context)
    results['sheets_created'].append('COVER')
    
    # 2. DATA sheets (sesuai template)
    for sheet_cfg in template['sheets']:
        ws = build_data_sheet(wb, sheet_cfg, domain)
        results['sheets_created'].append(sheet_cfg['name'])
    
    # 3. SUMMARY sheet
    summary_ws = build_summary_sheet(wb, domain)
    results['sheets_created'].append('SUMMARY')
    
    # 4. ANALYSIS sheet (pivot + stat)
    analysis_ws = build_analysis_sheet(wb, domain)
    results['sheets_created'].append('ANALYSIS')
    
    # 5. DASHBOARD (Level 3 enhance)
    dashboard_ws = build_executive_dashboard(wb, {}, domain)
    results['sheets_created'].append('DASHBOARD')
    
    return results
```

---

## MODE 4 — ANALYSIS (Read-Only)

**Trigger:** `/excel analisis`
**Enhance Level:** 0 (tidak edit file asli)
**Filosofi:** Temukan insight dari data, output laporan terpisah

```python
def execute_analysis_mode(df_dict: dict, domain: str) -> str:
    """
    Analysis Mode: baca file, generate laporan insight terpisah.
    File asli TIDAK diubah sama sekali.
    Return: path ke file laporan baru.
    """
    print("🔍 MODE: ANALYSIS — Read-only, separate report")
    
    # Buat workbook laporan baru (terpisah dari file asli)
    report_wb = Workbook()
    report_wb.remove(report_wb.active)
    
    # Sheet 1: Executive Summary
    exec_ws = report_wb.create_sheet("EXECUTIVE_SUMMARY")
    exec_ws.sheet_properties.tabColor = "1F4E79"
    exec_ws.sheet_view.showGridLines = False
    build_executive_summary(exec_ws, df_dict, domain)
    
    # Sheet 2: Data Quality Report
    dq_ws = report_wb.create_sheet("DATA_QUALITY")
    dq_ws.sheet_properties.tabColor = "ED7D31"
    from references_auto_intelligence import calculate_dq_score
    dq_results = calculate_dq_score(df_dict)
    build_dq_report_sheet(dq_ws, dq_results)
    
    # Sheet 3: Statistik
    stat_ws = report_wb.create_sheet("STATISTIK")
    stat_ws.sheet_properties.tabColor = "70AD47"
    from references_python_patterns import generate_stat_summary
    for sheet_name, df in df_dict.items():
        stat_df = generate_stat_summary(df)
        df_to_sheet(stat_ws, stat_df)
    
    # Sheet 4: Anomali
    anomaly_ws = report_wb.create_sheet("ANOMALI")
    anomaly_ws.sheet_properties.tabColor = "FF0000"
    from references_auto_intelligence import detect_anomalies
    all_anomalies = []
    for sheet_name, df in df_dict.items():
        all_anomalies.extend(detect_anomalies(df, sheet_name))
    build_anomaly_sheet(anomaly_ws, all_anomalies)
    
    # Sheet 5: Temuan & Rekomendasi
    rec_ws = report_wb.create_sheet("TEMUAN_REKOMENDASI")
    rec_ws.sheet_properties.tabColor = "5B9BD5"
    from references_auto_intelligence import generate_suggestions, generate_auto_insights
    insights = generate_auto_insights(list(df_dict.values())[0], domain)
    suggestions = generate_suggestions(df_dict, domain)
    build_recommendation_sheet(rec_ws, insights, suggestions)
    
    # Save laporan
    output_name = f"RIAN_Analysis_Report_{pd.Timestamp.now().strftime('%Y%m%d_%H%M')}.xlsx"
    output_path = save_output(report_wb, output_name, recalculate=False)
    
    print(f"📊 Laporan analisis: {output_name}")
    return output_path
```

---

## MODE 5 — AUDIT

**Trigger:** `/excel audit`
**Enhance Level:** 0 (tidak edit file asli)
**Filosofi:** Deep forensic scan formula dan data

```python
def execute_audit_mode(wb, df_dict: dict, filepath: str) -> str:
    """
    Audit Mode: deep scan, laporan audit terpisah.
    """
    print("🔎 MODE: AUDIT — Deep formula & data forensics")
    
    audit_results = {
        'hardcoded_numbers': [],
        'circular_refs': [],
        'broken_refs': [],
        'inconsistent_formulas': [],
        'magic_numbers': [],
        'formula_patterns': [],
        'unused_named_ranges': [],
        'hidden_sheets': [],
        'very_complex_formulas': [],
    }
    
    # Scan 1: Hardcoded numbers dalam formula
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                if isinstance(cell.value, str) and cell.value.startswith('='):
                    import re
                    # Cari angka hardcoded dalam formula (bukan cell reference)
                    formula = cell.value
                    # Angka yang bukan referensi (misal: *12% bukan *A12)
                    hardcoded = re.findall(r'(?<![A-Za-z])(\d+\.?\d*)\b(?!\s*[:\$A-Z])', formula)
                    suspicious = [n for n in hardcoded if float(n) not in [0, 1, 2, 100]]
                    if suspicious:
                        audit_results['hardcoded_numbers'].append({
                            'location': f"{ws.title}!{cell.coordinate}",
                            'formula': formula[:80],
                            'values': suspicious,
                        })
    
    # Scan 2: Formula yang berbeda dari pola di kolom yang sama
    for ws in wb.worksheets:
        for col_idx in range(1, (ws.max_column or 0) + 1):
            col_formulas = {}
            for row_idx in range(1, (ws.max_row or 0) + 1):
                cell = ws.cell(row_idx, col_idx)
                if isinstance(cell.value, str) and cell.value.startswith('='):
                    # Normalisasi (ganti nomor baris dengan template)
                    import re
                    pattern = re.sub(r'\d+', 'N', cell.value)
                    col_formulas.setdefault(pattern, []).append(cell.coordinate)
            
            # Jika ada lebih dari satu pola di kolom yang sama → inconsistent
            if len(col_formulas) > 1:
                for pattern, locations in col_formulas.items():
                    if len(locations) < len(list(col_formulas.values())[0]):
                        audit_results['inconsistent_formulas'].append({
                            'sheet': ws.title,
                            'col': get_column_letter(col_idx),
                            'pattern': pattern[:60],
                            'locations': locations,
                        })
    
    # Scan 3: #REF! dan broken references
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                if isinstance(cell.value, str) and '#REF!' in cell.value:
                    audit_results['broken_refs'].append(f"{ws.title}!{cell.coordinate}")
    
    # Scan 4: Formula sangat kompleks (> 200 karakter)
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                if isinstance(cell.value, str) and cell.value.startswith('='):
                    if len(cell.value) > 200:
                        audit_results['very_complex_formulas'].append({
                            'location': f"{ws.title}!{cell.coordinate}",
                            'length': len(cell.value),
                            'formula': cell.value[:100] + '...',
                        })
    
    # Scan 5: Hidden sheets
    for ws in wb.worksheets:
        if ws.sheet_state != 'visible':
            audit_results['hidden_sheets'].append(ws.title)
    
    # Buat Audit Report Excel
    audit_wb = build_audit_report(audit_results, filepath)
    
    output_name = f"RIAN_Audit_{pd.Timestamp.now().strftime('%Y%m%d_%H%M')}.xlsx"
    output_path = save_output(audit_wb, output_name, recalculate=False)
    
    print(f"\n📋 AUDIT SUMMARY:")
    print(f"   Hardcoded numbers : {len(audit_results['hardcoded_numbers'])}")
    print(f"   Broken refs (#REF!): {len(audit_results['broken_refs'])}")
    print(f"   Inconsistent formula: {len(audit_results['inconsistent_formulas'])}")
    print(f"   Sangat kompleks   : {len(audit_results['very_complex_formulas'])}")
    print(f"   Hidden sheets     : {len(audit_results['hidden_sheets'])}")
    
    return output_path

def build_audit_report(audit_results: dict, source_file: str) -> Workbook:
    """Buat workbook laporan audit"""
    wb = Workbook()
    wb.remove(wb.active)
    
    # Sheet RINGKASAN
    ws = wb.create_sheet("RINGKASAN_AUDIT")
    ws.sheet_properties.tabColor = "E74C3C"
    ws.sheet_view.showGridLines = False
    
    # Title
    ws['A1'] = f"AUDIT REPORT — {Path(source_file).name}"
    ws['A1'].font = Font(size=16, bold=True, color="1F4E79", name="Calibri")
    ws['A2'] = f"Dibuat: {pd.Timestamp.now().strftime('%d %B %Y %H:%M')} | RIAN v4 Audit Engine"
    ws['A2'].font = Font(size=10, color="888888", italic=True)
    
    summary_data = [
        ['Temuan', 'Jumlah', 'Severity', 'Rekomendasi'],
        ['Hardcoded numbers dalam formula', len(audit_results['hardcoded_numbers']), 
         'MEDIUM', 'Pindahkan ke ASSUMPTIONS sheet sebagai named range'],
        ['Broken references (#REF!)', len(audit_results['broken_refs']),
         'HIGH', 'Perbaiki segera — formula tidak akan kalkulasi'],
        ['Formula tidak konsisten dalam kolom', len(audit_results['inconsistent_formulas']),
         'LOW', 'Review dan standardisasi formula'],
        ['Formula sangat kompleks', len(audit_results['very_complex_formulas']),
         'MEDIUM', 'Pecah dengan LET() atau helper column'],
        ['Hidden sheets', len(audit_results['hidden_sheets']),
         'INFO', 'Review apakah masih diperlukan'],
    ]
    
    severity_colors = {'HIGH': 'FF0000', 'MEDIUM': 'ED7D31', 'LOW': 'FFC000', 'INFO': '2E75B6'}
    
    for r_idx, row in enumerate(summary_data, 4):
        for c_idx, val in enumerate(row, 1):
            cell = ws.cell(r_idx, c_idx, val)
            if r_idx == 4:
                cell.font = Font(bold=True, color="FFFFFF", name="Calibri")
                cell.fill = PatternFill("solid", fgColor="1F4E79")
            else:
                cell.font = Font(name="Calibri", size=10)
                if c_idx == 3 and val in severity_colors:
                    cell.font = Font(bold=True, color=severity_colors[val], name="Calibri")
    
    # Sheets per kategori
    categories = [
        ('HARDCODED', audit_results['hardcoded_numbers'], 'E74C3C',
         ['Lokasi', 'Formula', 'Nilai Hardcoded']),
        ('BROKEN_REFS', audit_results['broken_refs'], 'FF0000',
         ['Lokasi']),
        ('INCONSISTENT', audit_results['inconsistent_formulas'], 'ED7D31',
         ['Sheet', 'Kolom', 'Pola Formula', 'Lokasi']),
        ('KOMPLEKS', audit_results['very_complex_formulas'], 'FFC000',
         ['Lokasi', 'Panjang', 'Formula (preview)']),
    ]
    
    for sheet_name, data, color, headers in categories:
        if not data: continue
        audit_ws = wb.create_sheet(sheet_name)
        audit_ws.sheet_properties.tabColor = color
        
        # Write headers + data
        for c_idx, h in enumerate(headers, 1):
            audit_ws.cell(1, c_idx, h).font = Font(bold=True, color="FFFFFF", name="Calibri")
            audit_ws.cell(1, c_idx).fill = PatternFill("solid", fgColor="1F4E79")
        
        for r_idx, item in enumerate(data, 2):
            if isinstance(item, dict):
                vals = list(item.values())
            elif isinstance(item, str):
                vals = [item]
            else:
                vals = [str(item)]
            
            for c_idx, val in enumerate(vals, 1):
                audit_ws.cell(r_idx, c_idx, str(val)).font = Font(name="Calibri", size=10)
        
        auto_fit_columns(audit_ws)
    
    return wb
```

---

## MODE 6 — LEARNING

**Trigger:** `/excel jelasin`
**Enhance Level:** 2 + komentar di semua formula + KAMUS_FORMULA sheet
**Filosofi:** Edukasi — setiap formula harus bisa dipahami pemula

```python
FORMULA_EXPLANATIONS_ID = {
    'VLOOKUP': 'Mencari nilai di kolom pertama tabel, lalu mengambil nilai dari kolom lain',
    'XLOOKUP': 'Versi modern VLOOKUP — mencari di range manapun, lebih fleksibel',
    'INDEX': 'Mengambil nilai dari baris dan kolom tertentu dalam range',
    'MATCH': 'Mencari posisi (nomor urut) suatu nilai dalam range',
    'SUMIF': 'Menjumlahkan hanya jika kondisi terpenuhi',
    'SUMIFS': 'Menjumlahkan dengan beberapa kondisi sekaligus',
    'COUNTIF': 'Menghitung jumlah sel yang memenuhi kondisi',
    'IFERROR': 'Menampilkan nilai alternatif jika formula menghasilkan error',
    'IF': 'Menampilkan nilai berbeda tergantung kondisi (ya/tidak)',
    'IFS': 'Menampilkan nilai berbeda untuk banyak kondisi sekaligus',
    'DATEDIF': 'Menghitung selisih antara dua tanggal (hari, bulan, tahun)',
    'TEXT': 'Mengubah angka atau tanggal menjadi teks dengan format tertentu',
    'SUMPRODUCT': 'Mengalikan array lalu menjumlahkan hasilnya — sangat powerful',
    'OFFSET': 'Menggeser referensi cell sejauh baris/kolom tertentu',
    'INDIRECT': 'Membuat referensi dari teks string',
    'NETWORKDAYS': 'Menghitung hari kerja antara dua tanggal (exclude Sabtu-Minggu)',
    'RANK': 'Memberikan peringkat suatu nilai dalam daftar',
    'PERCENTILE': 'Menghitung nilai pada persentil tertentu',
}

def add_formula_comments_learning_mode(ws):
    """
    Learning Mode: tambahkan Comment penjelasan di setiap cell formula.
    """
    import re
    
    for row in ws.iter_rows():
        for cell in row:
            if not isinstance(cell.value, str) or not cell.value.startswith('='):
                continue
            
            formula = cell.value
            # Temukan fungsi utama
            funcs = re.findall(r'([A-Z][A-Z0-9]+)\s*\(', formula.upper())
            
            explanations = []
            for func in funcs[:3]:  # Max 3 fungsi per komentar
                if func in FORMULA_EXPLANATIONS_ID:
                    explanations.append(f"• {func}: {FORMULA_EXPLANATIONS_ID[func]}")
            
            if explanations:
                comment_text = f"RIAN — Penjelasan Formula:\n" + "\n".join(explanations)
                comment_text += f"\n\nFormula lengkap:\n{formula}"
                cell.comment = Comment(comment_text, "RIAN Tutor")
                cell.comment.width = 300
                cell.comment.height = 120

def build_kamus_formula_sheet(wb, df_dict: dict) -> None:
    """
    Buat sheet KAMUS_FORMULA — daftar semua formula yang dipakai + penjelasan.
    Ditambahkan di akhir workbook dalam Learning Mode.
    """
    ws = wb.create_sheet("KAMUS_FORMULA")
    ws.sheet_properties.tabColor = "3498DB"
    ws.sheet_view.showGridLines = False
    
    # Collect semua formula unik yang dipakai
    used_formulas = set()
    for sheet in wb.worksheets:
        if sheet.title == "KAMUS_FORMULA": continue
        for row in sheet.iter_rows():
            for cell in row:
                if isinstance(cell.value, str) and cell.value.startswith('='):
                    import re
                    funcs = re.findall(r'([A-Z][A-Z0-9]+)\s*\(', cell.value.upper())
                    used_formulas.update(funcs)
    
    # Title
    ws.merge_cells('A1:D1')
    ws['A1'] = "KAMUS FORMULA — Penjelasan Semua Formula yang Digunakan"
    ws['A1'].font = Font(size=14, bold=True, color="1F4E79", name="Calibri")
    ws['A1'].alignment = Alignment(horizontal='center', vertical='center')
    ws.row_dimensions[1].height = 35
    
    # Headers
    headers = ['Formula', 'Penjelasan', 'Contoh Penggunaan', 'Tips']
    for c, h in enumerate(headers, 1):
        cell = ws.cell(2, c, h)
        cell.font = Font(bold=True, color="FFFFFF", name="Calibri")
        cell.fill = PatternFill("solid", fgColor="1F4E79")
        cell.alignment = Alignment(horizontal='center', vertical='center')
    ws.row_dimensions[2].height = 22
    
    # Data
    FORMULA_TIPS = {
        'VLOOKUP': 'Selalu gunakan FALSE/0 untuk exact match. Pastikan kolom lookup diurutkan.',
        'XLOOKUP': 'Argumen ke-4 adalah nilai jika tidak ditemukan. Lebih aman dari VLOOKUP.',
        'SUMIF':   'Pastikan range criteria dan sum_range berukuran sama.',
        'IFERROR': 'Gunakan IFERROR untuk semua formula LOOKUP agar tidak tampil #N/A.',
        'DATEDIF': 'Gunakan "Y" untuk tahun, "M" untuk bulan, "D" untuk hari.',
    }
    
    FORMULA_EXAMPLES = {
        'VLOOKUP': '=VLOOKUP(A2,$D$2:$F$100,2,0)',
        'XLOOKUP': '=XLOOKUP(A2,$D$2:$D$100,$E$2:$E$100,"-")',
        'SUMIF':   '=SUMIF(A:A,"Jakarta",B:B)',
        'SUMIFS':  '=SUMIFS(C:C,A:A,"Jakarta",B:B,"2024")',
        'IFERROR': '=IFERROR(VLOOKUP(A2,D:F,2,0),"-")',
        'IF':      '=IF(A2>100,"Di atas target","Di bawah target")',
    }
    
    row_idx = 3
    for func in sorted(used_formulas):
        if func not in FORMULA_EXPLANATIONS_ID and func not in ['SUM','AVERAGE','MAX','MIN']:
            continue
        
        explanation = FORMULA_EXPLANATIONS_ID.get(func, f'Fungsi Excel: {func}')
        example = FORMULA_EXAMPLES.get(func, f'={func}()')
        tip = FORMULA_TIPS.get(func, 'Selalu wrap dengan IFERROR untuk keamanan.')
        
        ws.cell(row_idx, 1, func).font = Font(bold=True, name="Calibri", color="1F4E79")
        ws.cell(row_idx, 2, explanation).font = Font(name="Calibri", size=10)
        ws.cell(row_idx, 3, example).font = Font(name="Calibri", size=10, color="0070C0")
        ws.cell(row_idx, 4, tip).font = Font(name="Calibri", size=9, color="666666", italic=True)
        ws.cell(row_idx, 2).alignment = Alignment(wrap_text=True)
        ws.cell(row_idx, 4).alignment = Alignment(wrap_text=True)
        ws.row_dimensions[row_idx].height = 30
        
        # Zebra
        if row_idx % 2 == 0:
            for c in range(1, 5):
                ws.cell(row_idx, c).fill = PatternFill("solid", fgColor="F2F7FF")
        
        row_idx += 1
    
    # Column widths
    ws.column_dimensions['A'].width = 14
    ws.column_dimensions['B'].width = 45
    ws.column_dimensions['C'].width = 35
    ws.column_dimensions['D'].width = 45
```

---

## MODE 7 — COLLABORATION

**Trigger:** `/excel team`
**Enhance Level:** 2 + protection + change log + petunjuk
**Filosofi:** File siap dibagi ke tim dengan dokumentasi lengkap

```python
def execute_collab_mode(wb, df_dict: dict, domain: str) -> dict:
    """
    Collaboration Mode: protection + documentation + change log.
    """
    print("👥 MODE: COLLABORATION — Team-ready file")
    
    results = {'sheets_added': [], 'cells_locked': 0, 'cells_unlocked': 0}
    
    # 1. Tambah Sheet PETUNJUK_PENGGUNAAN
    petunjuk_ws = build_petunjuk_sheet(wb, domain)
    results['sheets_added'].append('PETUNJUK')
    
    # 2. Tambah/Update Sheet CHANGE_LOG
    changelog_ws = build_change_log_sheet(wb)
    results['sheets_added'].append('CHANGE_LOG')
    
    # 3. Lock formula cells, unlock input cells
    for ws in wb.worksheets:
        if ws.title in ('PETUNJUK_PENGGUNAAN', 'CHANGE_LOG'): continue
        locked, unlocked = protect_worksheet_smart(ws)
        results['cells_locked'] += locked
        results['cells_unlocked'] += unlocked
    
    # 4. Protect all sheets
    for ws in wb.worksheets:
        if ws.title == 'PETUNJUK_PENGGUNAAN': continue
        ws.protection.sheet = True
        ws.protection.enable()
        # Allow: select locked/unlocked, use autofilter, sort
        ws.protection.autoFilter = False
        ws.protection.sort = False
    
    return results

def build_petunjuk_sheet(wb: Workbook, domain: str) -> any:
    """Buat sheet PETUNJUK_PENGGUNAAN"""
    ws = wb.create_sheet("PETUNJUK_PENGGUNAAN")
    ws.sheet_properties.tabColor = "27AE60"
    ws.sheet_view.showGridLines = False
    
    ws.merge_cells('A1:F1')
    ws['A1'] = "PETUNJUK PENGGUNAAN FILE"
    ws['A1'].font = Font(size=16, bold=True, color="1F4E79", name="Calibri")
    ws['A1'].alignment = Alignment(horizontal='center', vertical='center')
    ws.row_dimensions[1].height = 40
    
    sections = [
        ("CARA MENGGUNAKAN", [
            "File ini terdiri dari beberapa sheet. Navigasi menggunakan tab di bawah.",
            f"Sheet DATA adalah tempat input data. Sheet lainnya dikalkulasi otomatis.",
            "Sel berwarna biru muda = INPUT (bisa diedit). Sel lain = FORMULA (terkunci).",
            "Untuk filter data, gunakan dropdown di baris header (baris pertama).",
        ]),
        ("ATURAN INPUT", [
            "Angka: gunakan angka saja tanpa Rp, titik, atau koma (kecuali desimal).",
            "Tanggal: ketik dalam format DD/MM/YYYY (contoh: 15/03/2025).",
            "Status: pilih dari dropdown yang tersedia, jangan ketik manual.",
            "Jangan hapus baris header (baris 1) atau formula di baris TOTAL.",
        ]),
        ("JIKA ADA ERROR", [
            "Jika muncul #REF! → jangan hapus baris/kolom di tengah data.",
            "Jika muncul #N/A → data yang dicari tidak ada di tabel referensi.",
            "Jika muncul #DIV/0! → ada pembagian dengan nol, cek data input.",
            "Hubungi pengelola file atau tim IT untuk bantuan lebih lanjut.",
        ]),
        ("KONTAK & VERSI", [
            f"Dibuat oleh: RIAN v4 | {pd.Timestamp.now().strftime('%d %B %Y')}",
            "Domain: " + domain.upper(),
            "Versi Excel target: " + get_session('excel_version', '2019'),
            "Untuk request perubahan, catat di sheet CHANGE_LOG.",
        ]),
    ]
    
    row = 3
    for section_title, items in sections:
        ws.merge_cells(start_row=row, end_row=row, start_column=1, end_column=6)
        ws.cell(row, 1, section_title).font = Font(bold=True, color="FFFFFF", name="Calibri")
        ws.cell(row, 1).fill = PatternFill("solid", fgColor="1F4E79")
        ws.cell(row, 1).alignment = Alignment(indent=1)
        ws.row_dimensions[row].height = 22
        row += 1
        
        for item in items:
            ws.merge_cells(start_row=row, end_row=row, start_column=1, end_column=6)
            cell = ws.cell(row, 1, f"  • {item}")
            cell.font = Font(name="Calibri", size=10)
            cell.fill = PatternFill("solid", fgColor="F2F7FF" if row % 2 == 0 else "FFFFFF")
            ws.row_dimensions[row].height = 18
            row += 1
        
        row += 1  # Spacer
    
    ws.column_dimensions['A'].width = 80
    return ws

def build_change_log_sheet(wb: Workbook) -> any:
    """Buat/update sheet CHANGE_LOG"""
    # Cek apakah sudah ada
    if 'CHANGE_LOG' in wb.sheetnames:
        ws = wb['CHANGE_LOG']
        # Cari baris berikutnya
        next_row = ws.max_row + 1
    else:
        ws = wb.create_sheet("CHANGE_LOG")
        ws.sheet_properties.tabColor = "C0392B"
        
        headers = ['No', 'Tanggal', 'Oleh', 'Tipe Perubahan', 'Sheet', 'Deskripsi', 'Status']
        for c, h in enumerate(headers, 1):
            ws.cell(1, c, h).font = Font(bold=True, color="FFFFFF", name="Calibri")
            ws.cell(1, c).fill = PatternFill("solid", fgColor="1F4E79")
            ws.cell(1, c).alignment = Alignment(horizontal='center', vertical='center')
        ws.row_dimensions[1].height = 22
        next_row = 2
    
    # Add entry untuk perubahan RIAN saat ini
    ws.cell(next_row, 1, next_row - 1)
    ws.cell(next_row, 2, pd.Timestamp.now().strftime('%d/%m/%Y %H:%M'))
    ws.cell(next_row, 3, "RIAN v4")
    ws.cell(next_row, 4, "Auto-update")
    ws.cell(next_row, 5, "All Sheets")
    ws.cell(next_row, 6, f"RIAN v4 — {get_session('mode','standard').title()} Mode applied")
    ws.cell(next_row, 7, "✅ Done")
    
    for c in range(1, 8):
        ws.cell(next_row, c).font = Font(name="Calibri", size=10)
        ws.cell(next_row, c).fill = PatternFill("solid", fgColor="F2F7FF")
    
    # Format
    ws.column_dimensions['A'].width = 5
    ws.column_dimensions['B'].width = 18
    ws.column_dimensions['C'].width = 15
    ws.column_dimensions['D'].width = 18
    ws.column_dimensions['E'].width = 20
    ws.column_dimensions['F'].width = 45
    ws.column_dimensions['G'].width = 12
    
    ws.freeze_panes = 'A2'
    return ws

def protect_worksheet_smart(ws) -> tuple:
    """Lock formula cells, unlock input cells (biru/kosong)"""
    locked = 0
    unlocked = 0
    
    from openpyxl.styles.protection import Protection
    
    # Unlock semua dulu
    for row in ws.iter_rows():
        for cell in row:
            cell.protection = Protection(locked=False, hidden=False)
    
    # Lock cell yang ada formula
    for row in ws.iter_rows():
        for cell in row:
            if isinstance(cell.value, str) and cell.value.startswith('='):
                cell.protection = Protection(locked=True)
                locked += 1
            elif cell.value is not None:
                unlocked += 1
    
    return locked, unlocked
```

---

## ═══════════════════════════════════════════════════
## MODE 8 — MIGRATION MODE (v4 NEW)
## ═══════════════════════════════════════════════════

**Trigger**: `/excel migrasi [dari] [ke]`
**Contoh**:
- `/excel migrasi 2016 365` — upgrade semua formula ke versi modern
- `/excel migrasi excel gsheets` — konversi file untuk Google Sheets
- `/excel migrasi 365 2019` — downgrade file untuk user versi lama

**Enhance Level**: Off (L0) — hanya lakukan konversi, tidak ada tambahan

### Workflow Migration Mode

```
[S0] Session init (mode=migration)
   → Deteksi source version & target version
   → Set migration_plan

[S1] Baca file + profiling
   → Scan SEMUA formula di workbook
   → Buat inventory formula: {sheet, cell, formula, funcs_used}

[MIGRATION STEP] Proses tiap formula:
   → Jika target > source: upgrade ke formula modern (jika ada)
   → Jika target < source: downgrade ke fallback kompatibel
   → Jika target = 'gsheets': konversi ke GSheets equivalent

[S6] Validasi
   → Layer 5: verifikasi SEMUA formula kompatibel dengan TARGET version
   → Tidak ada formula yang "selip"

[S8] Deliver
   → File yang sudah dikonversi
   → Sheet MIGRATION_REPORT: daftar semua perubahan
   → Catatan khusus untuk GSheets jika relevan
```

```python
def run_migration_mode(wb, df_dict: dict, source_version: str,
                        target_version: str) -> tuple:
    """
    Main runner Migration Mode.
    Return: (wb_migrated, migration_report: list)
    """
    print(f"\n{'═'*55}")
    print(f"  🔄 RIAN MIGRATION MODE")
    print(f"  {source_version.upper()} → {target_version.upper()}")
    print(f"{'═'*55}")

    migration_report = []
    changed_count    = 0
    skipped_count    = 0

    # ── Scan semua formula ────────────────────────────────────────
    for ws in wb.worksheets:
        if ws.title in ['MIGRATION_REPORT', 'VBA_GUIDE', 'CATATAN_GSHEETS']:
            continue

        for row in ws.iter_rows():
            for cell in row:
                if not isinstance(cell.value, str) or not cell.value.startswith('='):
                    continue

                original  = cell.value
                converted = migrate_formula(original, source_version, target_version)

                if converted != original:
                    cell.value = converted
                    changed_count += 1
                    migration_report.append({
                        'sheet':    ws.title,
                        'cell':     cell.coordinate,
                        'original': original[:80],
                        'migrated': converted[:80],
                        'type':     'upgraded' if _is_newer(target_version, source_version) else 'downgraded',
                    })
                    print(f"  ✏️  [{ws.title}!{cell.coordinate}] {_extract_funcs(original)} → {_extract_funcs(converted)}")
                else:
                    skipped_count += 1

    # ── GSheets special handling ──────────────────────────────────
    gsheets_notes = []
    if target_version == 'gsheets':
        gsheets_notes = gsheets_convert_notes(wb)
        # Tambahkan sheet panduan GSheets
        _add_gsheets_notes_sheet(wb, gsheets_notes)

    # ── Buat MIGRATION_REPORT sheet ───────────────────────────────
    _create_migration_report_sheet(wb, migration_report, source_version,
                                    target_version, gsheets_notes)

    print(f"\n✅ Migration selesai:")
    print(f"   Dikonversi : {changed_count} formula")
    print(f"   Tidak berubah: {skipped_count} formula")

    return wb, migration_report


def migrate_formula(formula: str, source: str, target: str) -> str:
    """Konversi satu formula dari source ke target version."""
    result = formula

    source_year = int(source) if source.isdigit() else (9999 if source == '365' else 0)
    target_year = int(target) if target.isdigit() else (9999 if target == '365' else 0)
    is_to_gsheets = (target == 'gsheets')

    if target_year > source_year or is_to_gsheets:
        # UPGRADE: pakai formula modern jika target lebih baru
        result = upgrade_formula(result, source, target)
    else:
        # DOWNGRADE: fallback ke formula kompatibel
        import re
        incompatible = check_formula_string(result)
        for func in incompatible:
            ctx = {}
            result = downgrade_formula(result, func, ctx)

    return result


def upgrade_formula(formula: str, source: str, target: str) -> str:
    """Upgrade formula ke versi lebih modern."""
    import re
    result = formula
    target_year = int(target) if target.isdigit() else 9999

    # IFERROR(INDEX(MATCH())) → XLOOKUP (jika target >= 2021)
    if target_year >= 2021:
        pattern = r'IFERROR\(INDEX\(([^,]+),\s*MATCH\(([^,]+),\s*([^,]+),\s*0\)\),\s*"([^"]*)"\)'
        replacement = r'XLOOKUP(\2,\3,\1,"\4")'
        result = re.sub(pattern, replacement, result, flags=re.IGNORECASE)

    # Nested IF → IFS (jika target >= 2016)
    # (simplified — hanya 2-level nested IF)
    if target_year >= 2016:
        pattern2 = r'IF\(([^,]+),([^,]+),IF\(([^,]+),([^,]+),([^)]+)\)\)'
        replacement2 = r'IFS(\1,\2,\3,\4,TRUE,\5)'
        result = re.sub(pattern2, replacement2, result, flags=re.IGNORECASE)

    # CONCATENATE → CONCAT (target >= 2016)
    if target_year >= 2016:
        result = re.sub(r'\bCONCATENATE\b', 'CONCAT', result, flags=re.IGNORECASE)

    # GSheets: TEXTSPLIT equivalent (pakai SPLIT)
    if target == 'gsheets':
        result = re.sub(r'\bTEXTSPLIT\b', 'SPLIT', result, flags=re.IGNORECASE)
        result = re.sub(r'\bTEXTJOIN\b', 'JOIN', result, flags=re.IGNORECASE)

    return result


def _is_newer(target: str, source: str) -> bool:
    t = int(target) if target.isdigit() else (9999 if target in ['365','gsheets'] else 0)
    s = int(source) if source.isdigit() else (9999 if source in ['365','gsheets'] else 0)
    return t > s

def _extract_funcs(formula: str) -> str:
    import re
    funcs = re.findall(r'([A-Z][A-Z0-9]+)\s*\(', formula.upper())
    return ', '.join(funcs[:3]) or formula[:30]

def _create_migration_report_sheet(wb, report: list, source: str,
                                    target: str, gsheets_notes: list) -> None:
    from openpyxl.styles import Font, PatternFill, Alignment
    # Hapus jika ada
    if 'MIGRATION_REPORT' in [s.title for s in wb.worksheets]:
        del wb['MIGRATION_REPORT']

    ws = wb.create_sheet('MIGRATION_REPORT')
    ws.sheet_properties.tabColor = "FF6B35"

    ws['A1'] = f'MIGRATION REPORT — {source.upper()} → {target.upper()}'
    ws['A1'].font = Font(bold=True, size=13, color='FFFFFF')
    ws['A1'].fill = PatternFill('solid', fgColor='FF6B35')
    ws['A2'] = f'Dibuat: {pd.Timestamp.now().strftime("%d/%m/%Y %H:%M")} | RIAN v4 Migration Engine'
    ws['A2'].font = Font(italic=True, size=10, color='666666')

    headers = ['Sheet', 'Cell', 'Formula Asli', 'Formula Baru', 'Tipe']
    for c, h in enumerate(headers, 1):
        cell = ws.cell(4, c, h)
        cell.font = Font(bold=True, color='FFFFFF')
        cell.fill = PatternFill('solid', fgColor='1F4E79')
        cell.alignment = Alignment(horizontal='center')

    for r, item in enumerate(report, 5):
        ws.cell(r, 1, item['sheet'])
        ws.cell(r, 2, item['cell'])
        ws.cell(r, 3, item['original'])
        ws.cell(r, 4, item['migrated'])
        type_cell = ws.cell(r, 5, item['type'])
        color = '70AD47' if item['type'] == 'upgraded' else 'FF6B35'
        type_cell.fill = PatternFill('solid', fgColor=color)
        type_cell.font = Font(color='FFFFFF', bold=True)

    if gsheets_notes:
        note_row = len(report) + 7
        ws.cell(note_row, 1, 'CATATAN GOOGLE SHEETS:').font = Font(bold=True)
        for i, note in enumerate(gsheets_notes):
            ws.cell(note_row + i + 1, 1, note)

    ws.column_dimensions['A'].width = 16
    ws.column_dimensions['B'].width = 8
    ws.column_dimensions['C'].width = 50
    ws.column_dimensions['D'].width = 50
    ws.column_dimensions['E'].width = 14
```

### Update Tabel Mode di SKILL.md
Mode ke-8 ini sudah ditambahkan ke shortcut commands:
`/excel migrasi [dari] [ke]` — Migration Mode

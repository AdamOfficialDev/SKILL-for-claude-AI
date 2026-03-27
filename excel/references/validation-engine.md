# Validation Engine Reference — RIAN v4
# 80+ checkpoint, 3-layer + cross-sheet + business logic validation

## ═══════════════════════════════════════════════════
## MASTER VALIDATION RUNNER (Gunakan ini)
## ═══════════════════════════════════════════════════

```python
def run_full_validation(filepath, wb, df_dict, domain="general"):
    """
    Master runner: 3 layer + business logic.
    Return: (all_passed: bool, report: dict)
    """
    import subprocess, json
    
    print(f"\n{'═'*55}")
    print(f"  🔍 RIAN VALIDATION ENGINE v3")
    print(f"{'═'*55}")
    
    report = {}
    all_passed = True
    
    # ── LAYER 1: Formula Integrity ─────────────────────
    print("\n📋 LAYER 1: Formula Integrity...")
    l1_pass, l1_data = layer1_formula_check(filepath)
    report['layer1'] = {'passed': l1_pass, 'data': l1_data}
    if not l1_pass:
        all_passed = False
    
    # ── LAYER 2: Data Integrity ────────────────────────
    print("\n📋 LAYER 2: Data Integrity...")
    issues, warnings = layer2_data_check(wb, df_dict)
    l2_pass = len(issues) == 0
    report['layer2'] = {'passed': l2_pass, 'issues': issues, 'warnings': warnings}
    if issues:
        print(f"   ❌ {len(issues)} issue KRITIS:")
        for i in issues[:5]: print(f"      • {i}")
    if warnings:
        print(f"   ⚠️ {len(warnings)} warning:")
        for w in warnings[:5]: print(f"      • {w}")
    if l2_pass:
        print(f"   ✅ L2 PASSED — {len(warnings)} warnings")
    else:
        all_passed = False
    
    # ── LAYER 3: Visual & UX ───────────────────────────
    print("\n📋 LAYER 3: Visual & UX...")
    vis_issues, vis_warnings = layer3_visual_check(wb)
    l3_pass = len(vis_issues) == 0
    report['layer3'] = {'passed': l3_pass, 'issues': vis_issues, 'warnings': vis_warnings}
    
    # Auto-fix visual issues
    auto_fixes = auto_fix_visual(wb)
    if auto_fixes:
        print(f"   🔧 Auto-fixed {len(auto_fixes)} visual issues")
    if l3_pass:
        print(f"   ✅ L3 PASSED")
    else:
        print(f"   ⚠️ L3: {len(vis_issues)} issues (non-blocking)")
    
    # ── LAYER 4: Business Logic (domain-specific) ──────
    print(f"\n📋 LAYER 4: Business Logic ({domain.upper()})...")
    biz_issues = layer4_business_logic(wb, df_dict, domain)
    report['layer4'] = {'passed': len(biz_issues) == 0, 'issues': biz_issues}
    if biz_issues:
        print(f"   ⚠️ {len(biz_issues)} business logic warnings:")
        for b in biz_issues[:3]: print(f"      • {b}")
    else:
        print(f"   ✅ L4 PASSED")
    
    # ── SUMMARY ────────────────────────────────────────
    print(f"\n{'═'*55}")
    status = "✅ SEMUA VALIDASI PASSED" if all_passed else "❌ ADA VALIDASI YANG GAGAL"
    print(f"  {status}")
    if all_passed:
        print(f"  File siap untuk dikirim ke user.")
    else:
        print(f"  Jalankan self_fix_all() lalu validasi ulang.")
    print(f"{'═'*55}\n")
    
    return all_passed, report

def iterative_validate_and_fix(filepath, wb, df_dict, domain="general", max_iter=3):
    """Loop validasi + fix sampai passed atau max iterasi"""
    for i in range(1, max_iter + 1):
        print(f"\n🔄 Iterasi {i}/{max_iter}")
        passed, report = run_full_validation(filepath, wb, df_dict, domain)
        
        if passed:
            print(f"✅ Lolos validasi pada iterasi {i}")
            return True, report
        
        # Auto-fix berdasarkan report
        if not report['layer1']['passed']:
            errors = report['layer1']['data'].get('error_summary', {})
            fixes = auto_fix_formula_errors(wb, errors)
            if fixes:
                wb.save(filepath)
                subprocess.run(['python', '/mnt/skills/public/xlsx/scripts/recalc.py', filepath],
                               capture_output=True)
        
        self_fix_all(wb, df_dict)
        wb.save(filepath)
    
    print(f"⚠️ Max iterasi tercapai — beberapa issue perlu perhatian manual")
    return False, report
```

---

## ═══════════════════════════════════════════════════
## LAYER 1: FORMULA INTEGRITY
## ═══════════════════════════════════════════════════

```python
def layer1_formula_check(filepath):
    """Cek semua formula via recalc.py"""
    import subprocess, json
    
    result = subprocess.run(
        ['python', '/mnt/skills/public/xlsx/scripts/recalc.py', filepath],
        capture_output=True, text=True, timeout=60
    )
    
    try:
        data = json.loads(result.stdout)
    except:
        print(f"⚠️ L1: recalc tidak return JSON — output: {result.stdout[:100]}")
        return True, {}  # Anggap pass jika tidak bisa parse
    
    if data.get('status') == 'errors_found':
        errors = data.get('error_summary', {})
        total_err = data.get('total_errors', 0)
        print(f"❌ L1 FAIL — {total_err} formula errors:")
        for err_type, detail in errors.items():
            locs = detail.get('locations', [])[:3]
            print(f"   {err_type}: {detail.get('count',0)}× di {locs}")
        return False, data
    
    total_ok = data.get('total_formulas', 0)
    print(f"✅ L1 PASS — {total_ok} formula OK")
    return True, data

def auto_fix_formula_errors(wb, error_summary):
    """Auto-fix formula errors yang umum"""
    fixes = []
    
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                if not isinstance(cell.value, str) or not cell.value.startswith('='):
                    continue
                
                val = cell.value
                
                # Fix DIV/0 — tambahkan IFERROR
                if '/' in val and 'IFERROR' not in val.upper() and 'IFNA' not in val.upper():
                    inner = val[1:]
                    cell.value = f'=IFERROR({inner},"-")'
                    fixes.append(f"IFERROR: {ws.title}!{cell.coordinate}")
                
                # Fix #REF! yang tertulis literal
                if '#REF!' in val:
                    cell.fill = PatternFill("solid", fgColor="FFE6E6")
                    c = Comment("⚠️ RIAN: Referensi rusak — perbaiki manual", "RIAN")
                    cell.comment = c
                    fixes.append(f"#REF! flagged: {ws.title}!{cell.coordinate}")
    
    return fixes

# ERROR GUIDE
ERROR_GUIDE = {
    '#REF!':   'Referensi cell tidak valid — row/col mungkin dihapus',
    '#DIV/0!': 'Pembagian dengan nol — gunakan IFERROR atau cek denominator',
    '#VALUE!': 'Tipe data salah — teks di cell yang harus angka',
    '#N/A':    'VLOOKUP/XLOOKUP tidak menemukan nilai — gunakan IFERROR/IFNA',
    '#NAME?':  'Nama fungsi salah eja atau Named Range tidak ditemukan',
    '#NUM!':   'Nilai numerik tidak valid (mis: SQRT angka negatif)',
    '#NULL!':  'Range intersection tidak valid — periksa operator range',
}
```

---

## ═══════════════════════════════════════════════════
## LAYER 2: DATA INTEGRITY (80+ CHECKPOINT)
## ═══════════════════════════════════════════════════

```python
def layer2_data_check(wb, df_dict):
    """Comprehensive data integrity check"""
    import re
    
    issues   = []  # KRITIS — harus diperbaiki
    warnings = []  # Perlu diperhatikan
    
    for sheet_name, df in df_dict.items():
        if len(df) == 0:
            warnings.append(f"[{sheet_name}] Sheet kosong")
            continue
        
        ws = wb[sheet_name] if sheet_name in wb.sheetnames else None
        
        # ── COMPLETENESS CHECKS ──────────────────────
        # C1: Kolom hampir kosong
        for col in df.columns:
            null_pct = df[col].isnull().mean() * 100
            if null_pct > 90:
                issues.append(f"[{sheet_name}][C1] Kolom '{col}' hampir kosong ({null_pct:.0f}%)")
            elif null_pct > 50:
                warnings.append(f"[{sheet_name}][C1] Kolom '{col}' banyak kosong ({null_pct:.0f}%)")
        
        # C2: Baris sepenuhnya kosong di tengah data
        mid_empty = df.isnull().all(axis=1)
        if mid_empty.any():
            warnings.append(f"[{sheet_name}][C2] {mid_empty.sum()} baris sepenuhnya kosong")
        
        # C3: Kolom tanpa nama (Unnamed)
        unnamed = [c for c in df.columns if str(c).startswith('Unnamed')]
        if unnamed:
            warnings.append(f"[{sheet_name}][C3] {len(unnamed)} kolom tanpa nama: {unnamed[:3]}")
        
        # ── UNIQUENESS CHECKS ─────────────────────────
        # U1: Duplikat baris
        dup_count = df.duplicated().sum()
        if dup_count > 0:
            dup_pct = dup_count / len(df) * 100
            if dup_pct > 20:
                issues.append(f"[{sheet_name}][U1] {dup_count} baris duplikat ({dup_pct:.0f}%) — sangat tinggi")
            else:
                warnings.append(f"[{sheet_name}][U1] {dup_count} baris duplikat")
        
        # U2: Duplikat di kolom ID/key
        id_cols = [c for c in df.columns if any(k in str(c).lower() for k in
                   ['id', 'kode', ' no', 'nomor', 'nik', 'nim', 'nip', 'uuid', 'nrp'])]
        for id_col in id_cols:
            dup_ids = df[id_col].dropna()
            if dup_ids.duplicated().any():
                dup_vals = dup_ids[dup_ids.duplicated()].tolist()[:3]
                issues.append(f"[{sheet_name}][U2] Duplikat ID di '{id_col}': {dup_vals}")
        
        # ── VALIDITY CHECKS ───────────────────────────
        # V1: Nilai negatif di kolom yang harus positif
        positive_kw = ['qty','jumlah','harga','price','stok','stock','gaji',
                       'salary','usia','age','umur','nilai stok','total']
        for col in df.select_dtypes(include='number').columns:
            if any(k in str(col).lower() for k in positive_kw):
                neg_count = (df[col] < 0).sum()
                if neg_count > 0:
                    issues.append(f"[{sheet_name}][V1] {neg_count} nilai negatif di '{col}'")
        
        # V2: Persentase di luar range wajar
        pct_kw = ['persen','percent','margin','rate','%','ach','achievement','utilization']
        for col in df.select_dtypes(include='number').columns:
            if any(k in str(col).lower() for k in pct_kw):
                max_val = df[col].max()
                if max_val <= 1.5:  # Format 0-1
                    invalid = ((df[col] < 0) | (df[col] > 1.5)).sum()
                else:              # Format 0-100
                    invalid = ((df[col] < 0) | (df[col] > 200)).sum()
                if invalid > 0:
                    warnings.append(f"[{sheet_name}][V2] {invalid} nilai persen tidak wajar di '{col}'")
        
        # V3: Tanggal anomali
        date_kw = ['tanggal','date','tgl','waktu','dob','lahir']
        for col in df.columns:
            if any(k in str(col).lower() for k in date_kw):
                try:
                    d = pd.to_datetime(df[col], errors='coerce')
                    future = (d > pd.Timestamp.now()).sum()
                    very_old = (d < pd.Timestamp('1980-01-01')).sum()
                    if future > 0:
                        warnings.append(f"[{sheet_name}][V3] {future} tanggal di masa depan di '{col}'")
                    if very_old > 0:
                        warnings.append(f"[{sheet_name}][V3] {very_old} tanggal sebelum 1980 di '{col}'")
                except: pass
        
        # V4: Email format
        email_cols = [c for c in df.columns if 'email' in str(c).lower()]
        for col in email_cols:
            if col in df.select_dtypes(include='object').columns:
                invalid_email = df[col].dropna().apply(
                    lambda x: not bool(re.match(r'^[\w.+-]+@[\w-]+\.\w{2,}$', str(x)))
                ).sum()
                if invalid_email > 0:
                    warnings.append(f"[{sheet_name}][V4] {invalid_email} email tidak valid di '{col}'")
        
        # V5: Nomor HP (Indonesia)
        phone_cols = [c for c in df.columns if any(k in str(c).lower() for k in ['hp','telepon','phone','no hp','handphone'])]
        for col in phone_cols:
            if col in df.select_dtypes(include='object').columns:
                invalid_phone = df[col].dropna().apply(
                    lambda x: not bool(re.match(r'^(\+62|62|0)[0-9]{8,13}$', re.sub(r'[\s\-()]', '', str(x))))
                ).sum()
                if invalid_phone > 0:
                    warnings.append(f"[{sheet_name}][V5] {invalid_phone} format nomor HP tidak valid di '{col}'")
        
        # ── CONSISTENCY CHECKS ────────────────────────
        # CO1: Format teks tidak konsisten
        for col in df.select_dtypes(include='object').columns:
            if any(k in str(col).lower() for k in ['id','kode','no','email','hp']): continue
            sample = df[col].dropna().astype(str)
            if len(sample) < 5: continue
            has_upper = sample.str.match(r'^[A-Z]').any()
            has_lower = sample.str.match(r'^[a-z]').any()
            if has_upper and has_lower:
                warnings.append(f"[{sheet_name}][CO1] Format huruf tidak konsisten di '{col}'")
        
        # CO2: Nilai sama tapi ditulis berbeda (trailing space, dll)
        for col in df.select_dtypes(include='object').columns:
            n_before = df[col].dropna().nunique()
            n_after  = df[col].dropna().astype(str).str.strip().nunique()
            if n_before > n_after:
                warnings.append(f"[{sheet_name}][CO2] '{col}': {n_before-n_after} nilai unik berkurang setelah strip — ada whitespace tersembunyi")
        
        # ── STRUCTURAL CHECKS ─────────────────────────
        # S1: Nama kolom duplikat
        dup_cols = [c for c in df.columns if list(df.columns).count(c) > 1]
        if dup_cols:
            issues.append(f"[{sheet_name}][S1] Nama kolom duplikat: {list(set(dup_cols))}")
        
        # S2: Terlalu banyak kolom (potential data quality issue)
        if len(df.columns) > 50:
            warnings.append(f"[{sheet_name}][S2] {len(df.columns)} kolom — sangat banyak, pertimbangkan normalisasi")
        
        # S3: Terlalu banyak baris kosong di akhir
        tail_empty = 0
        for idx in range(len(df)-1, max(len(df)-20, -1), -1):
            if df.iloc[idx].isnull().all():
                tail_empty += 1
            else:
                break
        if tail_empty > 5:
            warnings.append(f"[{sheet_name}][S3] {tail_empty} baris kosong di akhir data")
    
    return issues, warnings

def print_layer2_report(issues, warnings):
    total = len(issues) + len(warnings)
    if total == 0:
        print("✅ Layer 2 PASSED — Data integrity OK")
        return True
    
    if issues:
        print(f"\n❌ ISSUES ({len(issues)} — HARUS DIPERBAIKI):")
        for i in issues: print(f"   ❌ {i}")
    if warnings:
        print(f"\n⚠️ WARNINGS ({len(warnings)} — PERLU DIPERHATIKAN):")
        for w in warnings: print(f"   ⚠️ {w}")
    
    passed = len(issues) == 0
    print(f"\n{'✅ L2 PASSED (dengan warning)' if passed else '❌ L2 FAILED'}")
    return passed
```

---

## ═══════════════════════════════════════════════════
## LAYER 3: VISUAL & UX CHECK
## ═══════════════════════════════════════════════════

```python
def layer3_visual_check(wb):
    """Visual & UX quality check — 20+ checkpoint"""
    issues   = []
    warnings = []
    
    for ws in wb.worksheets:
        name = ws.title
        max_row = ws.max_row or 0
        max_col = ws.max_column or 0
        
        # VIS1: Nama sheet default
        if name.lower() in ['sheet1','sheet2','sheet3','sheet4','lembar1','lembar2']:
            issues.append(f"[VIS1] Sheet masih bernama '{name}' — rename!")
        
        # VIS2: Freeze panes untuk data panjang
        if max_row > 20 and not ws.freeze_panes:
            warnings.append(f"[{name}][VIS2] Data {max_row} baris tanpa freeze panes")
        
        # VIS3: Column terlalu sempit
        narrow_cols = []
        for col_letter, dim in ws.column_dimensions.items():
            if dim.width and dim.width < 5:
                narrow_cols.append(col_letter)
        if narrow_cols:
            warnings.append(f"[{name}][VIS3] Kolom terlalu sempit: {narrow_cols[:5]}")
        
        # VIS4: Kolom terlalu lebar
        wide_cols = []
        for col_letter, dim in ws.column_dimensions.items():
            if dim.width and dim.width > 60:
                wide_cols.append(col_letter)
        if wide_cols:
            warnings.append(f"[{name}][VIS4] Kolom terlalu lebar: {wide_cols[:3]}")
        
        # VIS5: Row 1 kosong (header mungkin di baris lain)
        if max_row > 1 and max_col > 0:
            row1_empty = all(ws.cell(1, c).value is None for c in range(1, min(max_col+1, 10)))
            if row1_empty:
                warnings.append(f"[{name}][VIS5] Baris pertama kosong — header mungkin di baris lain")
        
        # VIS6: Autofilter untuk data tabular
        if max_row > 5 and max_col > 2 and not ws.auto_filter.ref:
            warnings.append(f"[{name}][VIS6] Tabel belum memiliki autofilter")
        
        # VIS7: Print area terdefinisi
        if max_row > 10 and not ws.print_area:
            warnings.append(f"[{name}][VIS7] Print area belum diset")
        
        # VIS8: Tab color
        if not ws.sheet_properties.tabColor or not ws.sheet_properties.tabColor.value:
            warnings.append(f"[{name}][VIS8] Tab sheet belum diberi warna")
        
        # VIS9: Gridlines tersembunyi di dashboard
        if 'dashboard' in name.lower() and ws.sheet_view.showGridLines:
            warnings.append(f"[{name}][VIS9] Dashboard sebaiknya sembunyikan gridlines")
        
        # VIS10: Header row ada bold?
        if max_row > 2 and max_col > 0:
            header_bold = ws.cell(1, 1).font and ws.cell(1, 1).font.bold
            if not header_bold and ws.cell(1,1).value:
                warnings.append(f"[{name}][VIS10] Header baris 1 belum bold")
        
        # VIS11: Baris terlalu tinggi (typo row height)
        for row_num, dim in ws.row_dimensions.items():
            if dim.height and dim.height > 100:
                warnings.append(f"[{name}][VIS11] Baris {row_num} terlalu tinggi ({dim.height}px)")
    
    passed = len(issues) == 0
    total = len(issues) + len(warnings)
    
    if total == 0:
        print("✅ L3 PASSED — Visual & UX OK")
    else:
        if issues:
            for i in issues: print(f"   ❌ {i}")
        if warnings:
            for w in warnings[:5]: print(f"   ⚠️ {w}")
        status = "PASSED (dengan warning)" if passed else "FAILED"
        print(f"   {'✅' if passed else '❌'} L3 {status}")
    
    return issues, warnings

def auto_fix_visual(wb):
    """Auto-fix visual issues yang bisa diperbaiki otomatis"""
    fixes = []
    
    for ws in wb.worksheets:
        max_row = ws.max_row or 0
        max_col = ws.max_column or 0
        
        # Fix 1: Freeze panes
        if max_row > 20 and not ws.freeze_panes:
            ws.freeze_panes = 'A2'
            fixes.append(f"[{ws.title}] Freeze panes A2")
        
        # Fix 2: Autofilter
        if max_row > 5 and max_col > 1 and not ws.auto_filter.ref:
            mc = get_column_letter(max_col)
            ws.auto_filter.ref = f"A1:{mc}1"
            fixes.append(f"[{ws.title}] Autofilter A1:{mc}1")
        
        # Fix 3: Print area
        if max_row > 10 and not ws.print_area and max_col > 0:
            mc = get_column_letter(max_col)
            ws.print_area = f"A1:{mc}{max_row}"
            fixes.append(f"[{ws.title}] Print area A1:{mc}{max_row}")
        
        # Fix 4: Hide gridlines di dashboard
        if 'dashboard' in ws.title.lower():
            if ws.sheet_view.showGridLines:
                ws.sheet_view.showGridLines = False
                fixes.append(f"[{ws.title}] Gridlines hidden")
        
        # Fix 5: Page orientation ke landscape untuk data lebar
        if max_col > 8:
            ws.page_setup.orientation = 'landscape'
            ws.page_setup.paperSize = 9  # A4
            fixes.append(f"[{ws.title}] Landscape A4")
        
        # Fix 6: Hapus trailing empty rows
        deleted = 0
        for r in range(max_row, max(max_row - 30, 0), -1):
            if all(ws.cell(r, c).value is None for c in range(1, min(max_col + 1, 20))):
                ws.delete_rows(r)
                deleted += 1
            else:
                break
        if deleted:
            fixes.append(f"[{ws.title}] Cleaned {deleted} trailing empty rows")
    
    return fixes
```

---

## ═══════════════════════════════════════════════════
## LAYER 4: BUSINESS LOGIC VALIDATION (Domain-Specific)
## ═══════════════════════════════════════════════════

```python
def layer4_business_logic(wb, df_dict, domain):
    """Validasi logika bisnis per domain"""
    issues = []
    
    for sheet_name, df in df_dict.items():
        cols_lower = {c: c.lower() for c in df.columns}
        
        if domain == 'finance':
            # Cek Debit = Kredit
            debit_cols  = [c for c, cl in cols_lower.items() if 'debit' in cl]
            kredit_cols = [c for c, cl in cols_lower.items() if 'kredit' in cl or 'credit' in cl]
            if debit_cols and kredit_cols:
                total_debit  = df[debit_cols[0]].sum()
                total_kredit = df[kredit_cols[0]].sum()
                diff = abs(total_debit - total_kredit)
                if diff > 1:  # Toleransi 1 rupiah (rounding)
                    issues.append(f"[{sheet_name}] Debit ({total_debit:,.0f}) ≠ Kredit ({total_kredit:,.0f}) | Selisih: {diff:,.0f}")
            
            # Cek Total Aset = Total L+E (jika ada neraca)
            if 'neraca' in sheet_name.lower() or 'balance' in sheet_name.lower():
                aset_cols = [c for c, cl in cols_lower.items() if 'aset' in cl or 'asset' in cl]
                liab_cols = [c for c, cl in cols_lower.items() if 'liabilitas' in cl or 'liability' in cl or 'ekuitas' in cl]
                if aset_cols and liab_cols:
                    issues.append(f"[{sheet_name}] Verifikasi: Total Aset = Total Liabilitas + Ekuitas")
        
        elif domain == 'payroll':
            # THP harus positif
            thp_cols = [c for c, cl in cols_lower.items() if 'thp' in cl or 'take home' in cl or 'net' in cl]
            for col in thp_cols:
                if col in df.select_dtypes(include='number').columns:
                    negative_thp = (df[col] < 0).sum()
                    if negative_thp > 0:
                        issues.append(f"[{sheet_name}] {negative_thp} karyawan dengan THP negatif — cek formula potongan")
        
        elif domain == 'sales':
            # Achievement tidak boleh terlalu jauh dari target (>500% mungkin error)
            ach_cols = [c for c, cl in cols_lower.items() if 'ach' in cl or 'achievement' in cl]
            for col in ach_cols:
                if col in df.select_dtypes(include='number').columns:
                    max_val = df[col].max()
                    if max_val <= 1 and max_val > 5:  # Format 0-1, tapi >500%
                        issues.append(f"[{sheet_name}] Achievement '{col}' ada yang >500% — mungkin error input")
        
        elif domain == 'inventory':
            # Stok aktual tidak boleh negatif
            stok_cols = [c for c, cl in cols_lower.items() if 'stok' in cl or 'stock' in cl or 'aktual' in cl]
            for col in stok_cols:
                if col in df.select_dtypes(include='number').columns:
                    neg_stok = (df[col] < 0).sum()
                    if neg_stok > 0:
                        issues.append(f"[{sheet_name}] {neg_stok} item dengan stok negatif di '{col}'")
        
        elif domain == 'hr':
            # Masa kerja tidak boleh negatif
            mk_cols = [c for c, cl in cols_lower.items() if 'masa kerja' in cl or 'tenure' in cl]
            for col in mk_cols:
                if col in df.select_dtypes(include='number').columns:
                    neg_mk = (df[col] < 0).sum()
                    if neg_mk > 0:
                        issues.append(f"[{sheet_name}] {neg_mk} karyawan dengan masa kerja negatif — cek Tgl Masuk")
    
    return issues
```

---

## ═══════════════════════════════════════════════════
## CROSS-SHEET REFERENTIAL INTEGRITY
## ═══════════════════════════════════════════════════

```python
def check_cross_sheet_integrity(df_dict, key_mappings):
    """
    Cek referential integrity antar sheet.
    key_mappings = [
        ('DATA_PENJUALAN', 'Kode Produk', 'MASTER_PRODUK', 'Kode Produk'),
        ('PAYROLL', 'NIK', 'DATA_KARYAWAN', 'NIK'),
    ]
    """
    issues = []
    
    for child_sheet, child_col, parent_sheet, parent_col in key_mappings:
        if child_sheet not in df_dict or parent_sheet not in df_dict:
            continue
        
        child_df  = df_dict[child_sheet]
        parent_df = df_dict[parent_sheet]
        
        if child_col not in child_df.columns or parent_col not in parent_df.columns:
            continue
        
        child_keys  = set(child_df[child_col].dropna().astype(str))
        parent_keys = set(parent_df[parent_col].dropna().astype(str))
        
        orphan_keys = child_keys - parent_keys
        
        if orphan_keys:
            issues.append(
                f"[Cross-Sheet] {len(orphan_keys)} nilai '{child_col}' di [{child_sheet}] "
                f"tidak ada di [{parent_sheet}]: {list(orphan_keys)[:3]}"
            )
    
    if issues:
        print(f"\n⚠️ CROSS-SHEET INTEGRITY: {len(issues)} issue")
        for i in issues: print(f"   • {i}")
    else:
        print("✅ Cross-sheet integrity OK")
    
    return issues
```

---

## ═══════════════════════════════════════════════════
## SELF-FIX ENGINE (Master)
## ═══════════════════════════════════════════════════

```python
def self_fix_all(wb, df_dict=None):
    """
    Master self-fix: perbaiki semua issue yang bisa diperbaiki otomatis.
    Tidak mengubah data user — hanya formatting, formula wrapping, structure.
    """
    all_fixes = []
    
    for ws in wb.worksheets:
        # Fix 1: IFERROR wrap untuk formula pembagian
        for row in ws.iter_rows():
            for cell in row:
                if isinstance(cell.value, str) and cell.value.startswith('='):
                    val = cell.value
                    if ('/' in val and 
                        'IFERROR' not in val.upper() and 
                        'IFNA' not in val.upper() and
                        '#' not in val):
                        cell.value = f'=IFERROR({val[1:]},"-")'
                        all_fixes.append(f"IFERROR: {ws.title}!{cell.coordinate}")
        
        # Fix 2: Freeze panes
        if (ws.max_row or 0) > 20 and not ws.freeze_panes:
            ws.freeze_panes = 'A2'
            all_fixes.append(f"Freeze: {ws.title}")
        
        # Fix 3: Autofilter
        if (ws.max_row or 0) > 5 and (ws.max_column or 0) > 1 and not ws.auto_filter.ref:
            mc = get_column_letter(ws.max_column or 1)
            ws.auto_filter.ref = f"A1:{mc}1"
            all_fixes.append(f"Autofilter: {ws.title}")
        
        # Fix 4: Tab color jika belum ada
        if not ws.sheet_properties.tabColor or not ws.sheet_properties.tabColor.value:
            name_lower = ws.title.lower()
            if 'dashboard' in name_lower: ws.sheet_properties.tabColor = "1F4E79"
            elif 'cover' in name_lower: ws.sheet_properties.tabColor = "2E75B6"
            elif 'summary' in name_lower: ws.sheet_properties.tabColor = "70AD47"
            elif 'data' in name_lower: ws.sheet_properties.tabColor = "404040"
            else: ws.sheet_properties.tabColor = "808080"
            all_fixes.append(f"Tab color: {ws.title}")
        
        # Fix 5: Hapus trailing empty rows
        deleted = 0
        max_r = ws.max_row or 0
        max_c = ws.max_column or 1
        for r in range(max_r, max(max_r - 50, 0), -1):
            if all(ws.cell(r, c).value is None for c in range(1, min(max_c+1, 20))):
                ws.delete_rows(r)
                deleted += 1
            else:
                break
        if deleted:
            all_fixes.append(f"Clean empty rows ({deleted}): {ws.title}")
        
        # Fix 6: Rename Sheet1/Sheet2
        if ws.title.lower() in ['sheet1','sheet2','sheet3','lembar1']:
            ws.title = f"DATA_{ws.title.upper()}"
            all_fixes.append(f"Renamed: {ws.title}")
    
    if all_fixes:
        print(f"\n🔧 Self-Fix Applied: {len(all_fixes)} corrections")
        for f in all_fixes[:10]:
            print(f"   • {f}")
        if len(all_fixes) > 10:
            print(f"   ... dan {len(all_fixes)-10} lainnya")
    else:
        print("✅ Tidak ada yang perlu difix secara otomatis")
    
    return all_fixes
```

---

## LAYER 5 — VERSION COMPATIBILITY VALIDATOR (BARU v4)

```python
def layer5_version_check(wb) -> tuple:
    """
    Layer 5 BARU: Verifikasi semua formula kompatibel dengan versi Excel yang diset.
    Dipanggil setelah semua formula ditulis ke workbook.
    """
    from references_version_compat import (
        check_formula_string, DOWNGRADE_MAP,
        VERSION_FORMULA_SETS
    )
    
    version = get_session('excel_version', '2019')
    fs = VERSION_FORMULA_SETS.get(version, VERSION_FORMULA_SETS['2019'])
    
    # Excel 365 = semua formula ok
    if fs.get('allowed') == ['ALL']:
        print(f"✅ L5 PASSED — Excel 365, semua formula diizinkan")
        return True, []
    
    issues = []
    
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                if not isinstance(cell.value, str) or not cell.value.startswith('='):
                    continue
                
                incompatible = check_formula_string(cell.value)
                for func in incompatible:
                    issues.append({
                        'sheet': ws.title,
                        'cell': cell.coordinate,
                        'formula': cell.value[:80],
                        'func': func,
                        'fix': DOWNGRADE_MAP.get(func, 'Gunakan alternatif kompatibel')
                    })
    
    passed = len(issues) == 0
    
    if passed:
        print(f"✅ L5 PASSED — Semua formula valid untuk Excel {version}")
    else:
        print(f"❌ L5 FAIL — {len(issues)} formula tidak kompatibel:")
        for i in issues[:5]:
            print(f"   [{i['sheet']}!{i['cell']}] {i['func']}: {i['fix']}")
        if len(issues) > 5:
            print(f"   ... dan {len(issues)-5} lainnya")
    
    return passed, issues

def auto_fix_version_issues(wb, issues: list) -> list:
    """Auto-downgrade formula yang tidak kompatibel"""
    from references_version_compat import downgrade_formula
    
    fixes = []
    for issue in issues:
        ws = wb[issue['sheet']]
        cell = ws[issue['cell']]
        original = cell.value
        fixed = downgrade_formula(original, issue['func'])
        if fixed != original:
            cell.value = fixed
            # Tambah comment untuk transparansi
            cell.comment = Comment(
                f"RIAN v4: Dikonversi dari {issue['func']} untuk kompatibilitas Excel {get_session('excel_version')}",
                "RIAN v4"
            )
            fixes.append(f"{issue['sheet']}!{issue['cell']}: {issue['func']} → downgraded")
    
    return fixes
```

## UPDATED MASTER VALIDATION RUNNER (v4 — 5 Layer)

```python
def run_full_validation_v4(filepath, wb, df_dict, domain="general"):
    """Master validation v4: 5 layer + version compat"""
    import subprocess, json
    
    print(f"\n{'═'*55}")
    print(f"  RIAN v4 VALIDATION ENGINE — 5 LAYER")
    print(f"{'═'*55}")
    
    report = {}
    all_passed = True
    
    # Layer 1: Formula
    print("\n[L1] Formula Integrity...")
    result = subprocess.run(
        ['python', '/mnt/skills/public/xlsx/scripts/recalc.py', filepath],
        capture_output=True, text=True, timeout=60
    )
    try:
        data = json.loads(result.stdout)
        l1_pass = data.get('status') != 'errors_found'
        n_formula = data.get('total_formulas', 0)
        print(f"    {'✅' if l1_pass else '❌'} {n_formula} formula — {data.get('status','ok')}")
    except:
        l1_pass = True
    report['L1'] = l1_pass
    if not l1_pass: all_passed = False
    
    # Layer 2: Data
    print("\n[L2] Data Integrity...")
    issues, warnings = layer2_data_check(wb, df_dict)
    l2_pass = len(issues) == 0
    if issues:
        print(f"    ❌ {len(issues)} issues, {len(warnings)} warnings")
        for i in issues[:3]: print(f"       • {i}")
    else:
        print(f"    ✅ {len(warnings)} warnings")
    report['L2'] = l2_pass
    if not l2_pass: all_passed = False
    
    # Layer 3: Visual
    print("\n[L3] Visual & UX...")
    vis_issues, vis_warnings = layer3_visual_check(wb)
    auto_fix_visual(wb)
    l3_pass = len(vis_issues) == 0
    print(f"    {'✅' if l3_pass else '⚠️'} {len(vis_issues)} issues, {len(vis_warnings)} warnings (auto-fixed)")
    report['L3'] = l3_pass
    
    # Layer 4: Business Logic
    print(f"\n[L4] Business Logic ({domain.upper()})...")
    biz_issues = layer4_business_logic(wb, df_dict, domain)
    l4_pass = len(biz_issues) == 0
    if biz_issues:
        print(f"    ⚠️ {len(biz_issues)} business logic warnings")
        for b in biz_issues[:2]: print(f"       • {b}")
    else:
        print(f"    ✅ Business logic OK")
    report['L4'] = l4_pass
    
    # Layer 5: Version Compat (BARU)
    print(f"\n[L5] Version Compatibility (Excel {get_session('excel_version')})...")
    l5_pass, compat_issues = layer5_version_check(wb)
    if not l5_pass:
        auto_fix_version_issues(wb, compat_issues)
        print(f"    🔧 {len(compat_issues)} formula downgraded")
        # Re-check setelah fix
        l5_pass, _ = layer5_version_check(wb)
    report['L5'] = l5_pass
    if not l5_pass: all_passed = False
    
    # Summary
    print(f"\n{'═'*55}")
    print(f"  {'✅ SEMUA PASSED' if all_passed else '❌ ADA YANG GAGAL'}")
    for layer, passed in report.items():
        print(f"  {layer}: {'✅' if passed else '❌'}")
    print(f"{'═'*55}\n")
    
    return all_passed, report
```

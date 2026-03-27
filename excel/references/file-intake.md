# File Intake Reference — RIAN v4
# Protocol baca semua format file + error handling + streaming

## ═══════════════════════════════════════════════════
## 1. DETEKSI & LISTING FILE UPLOAD
## ═══════════════════════════════════════════════════

```python
import os, shutil, chardet
from pathlib import Path
import pandas as pd
from openpyxl import load_workbook

UPLOAD_DIR = Path("/mnt/user-data/uploads/")
WORK_DIR   = Path("/home/claude/")
OUTPUT_DIR = Path("/mnt/user-data/outputs/")
OUTPUT_DIR.mkdir(exist_ok=True)

def list_uploads():
    """List semua file upload yang tersedia"""
    files = []
    for f in sorted(UPLOAD_DIR.iterdir()):
        if f.is_file():
            size_kb = f.stat().st_size / 1024
            files.append({
                'name': f.name,
                'ext': f.suffix.lower(),
                'size_kb': size_kb,
                'path': str(f)
            })
            print(f"  📄 {f.name:40} {size_kb:8.1f} KB")
    
    print(f"\n  Total: {len(files)} file(s)\n")
    return files

def prep_file(filename):
    """Copy file upload ke working directory untuk diproses"""
    src = UPLOAD_DIR / filename
    dst = WORK_DIR / f"work_{filename}"
    shutil.copy2(str(src), str(dst))
    print(f"✅ Copied: {filename} → work_{filename}")
    return str(dst)
```

---

## ═══════════════════════════════════════════════════
## 2. UNIVERSAL FILE READER
## ═══════════════════════════════════════════════════

```python
def read_any_excel(filepath, max_rows_preview=5):
    """
    Universal reader untuk semua format spreadsheet.
    Return: (df_dict, wb_or_None, file_info)
    """
    ext = Path(filepath).suffix.lower()
    df_dict = {}
    wb = None
    file_info = {'ext': ext, 'path': filepath}
    
    # ── .XLSX / .XLSM ────────────────────────────────
    if ext in ('.xlsx', '.xlsm'):
        try:
            # Baca dengan openpyxl dulu (untuk formula info)
            wb = load_workbook(filepath, data_only=True)
            file_info['has_formulas'] = _check_has_formulas(filepath)
            file_info['sheets'] = wb.sheetnames
        except Exception as e:
            print(f"⚠️ openpyxl error: {e}")
            wb = None
        
        # Baca data dengan pandas
        try:
            df_dict = pd.read_excel(filepath, sheet_name=None)
        except Exception as e:
            print(f"❌ pandas read error: {e}")
            raise
    
    # ── .XLS (Legacy) ────────────────────────────────
    elif ext == '.xls':
        try:
            df_dict = pd.read_excel(filepath, sheet_name=None, engine='xlrd')
            file_info['legacy'] = True
            file_info['sheets'] = list(df_dict.keys())
        except Exception as e:
            print(f"❌ xlrd error: {e}")
            raise
    
    # ── .ODS ─────────────────────────────────────────
    elif ext == '.ods':
        try:
            df_dict = pd.read_excel(filepath, sheet_name=None, engine='odf')
            file_info['sheets'] = list(df_dict.keys())
        except Exception as e:
            print(f"❌ odf error: {e}")
            raise
    
    # ── .XLSB (Binary) ───────────────────────────────
    elif ext == '.xlsb':
        try:
            df_dict = pd.read_excel(filepath, engine='pyxlsb', sheet_name=None)
            file_info['sheets'] = list(df_dict.keys())
        except ImportError:
            print("❌ pyxlsb tidak terinstall — install dengan: pip install pyxlsb --break-system-packages")
            raise
    
    # ── .CSV ─────────────────────────────────────────
    elif ext == '.csv':
        df, enc, sep = _read_csv_smart(filepath)
        df_dict = {'Sheet1': df}
        file_info['encoding'] = enc
        file_info['delimiter'] = sep
        file_info['sheets'] = ['Sheet1']
    
    # ── .TSV ─────────────────────────────────────────
    elif ext == '.tsv':
        with open(filepath, 'rb') as f:
            enc = chardet.detect(f.read(50000))['encoding'] or 'utf-8'
        df = pd.read_csv(filepath, sep='\t', encoding=enc, on_bad_lines='skip')
        df_dict = {'Sheet1': df}
        file_info['encoding'] = enc
        file_info['sheets'] = ['Sheet1']
    
    # ── .TXT (Tabular) ───────────────────────────────
    elif ext == '.txt':
        df = _read_txt_as_table(filepath)
        df_dict = {'Sheet1': df}
        file_info['sheets'] = ['Sheet1']
    
    else:
        raise ValueError(f"Format tidak didukung: {ext}")
    
    # Clean column names
    for sheet_name in list(df_dict.keys()):
        df = df_dict[sheet_name]
        df.columns = [str(c).strip() for c in df.columns]
        df_dict[sheet_name] = df
    
    # Print profil
    _print_file_profile(df_dict, file_info, max_rows_preview)
    
    return df_dict, wb, file_info

def _read_csv_smart(filepath):
    """Baca CSV dengan auto-detect encoding dan delimiter"""
    # Detect encoding
    with open(filepath, 'rb') as f:
        raw = f.read(100000)
        enc_result = chardet.detect(raw)
        enc = enc_result['encoding'] or 'utf-8'
        confidence = enc_result.get('confidence', 0)
    
    print(f"  CSV encoding: {enc} (confidence: {confidence:.0%})")
    
    # Try common delimiters
    for sep in [',', ';', '\t', '|']:
        try:
            df = pd.read_csv(filepath, sep=sep, encoding=enc, 
                             nrows=5, on_bad_lines='skip')
            if len(df.columns) > 1:
                # Full read
                df = pd.read_csv(filepath, sep=sep, encoding=enc, 
                                 on_bad_lines='skip')
                print(f"  Delimiter: '{sep}' | {len(df.columns)} kolom")
                return df, enc, sep
        except: continue
    
    # Fallback: auto-detect
    df = pd.read_csv(filepath, sep=None, engine='python', 
                     encoding=enc, on_bad_lines='skip')
    return df, enc, 'auto'

def _read_txt_as_table(filepath):
    """Baca .txt sebagai tabular data"""
    for sep in ['\t', ',', ';', '|', ' ']:
        try:
            df = pd.read_csv(filepath, sep=sep, nrows=3)
            if len(df.columns) > 1:
                return pd.read_csv(filepath, sep=sep, on_bad_lines='skip')
        except: continue
    return pd.read_csv(filepath, sep=None, engine='python')

def _check_has_formulas(filepath):
    """Cek apakah file memiliki formula"""
    try:
        wb_formula = load_workbook(filepath, data_only=False)
        for ws in wb_formula.worksheets:
            for row in ws.iter_rows():
                for cell in row:
                    if isinstance(cell.value, str) and cell.value.startswith('='):
                        return True
        return False
    except:
        return False

def _print_file_profile(df_dict, file_info, max_rows_preview=3):
    """Print profil file yang informatif"""
    print(f"\n{'═'*60}")
    print(f"  📊 FILE PROFILE — {Path(file_info['path']).name}")
    print(f"{'═'*60}")
    print(f"  Format : {file_info['ext'].upper()}")
    print(f"  Sheets : {len(df_dict)}")
    if file_info.get('has_formulas'):
        print(f"  Formula: ✅ Ada")
    if file_info.get('encoding'):
        print(f"  Encoding: {file_info['encoding']}")
    
    for sheet_name, df in df_dict.items():
        print(f"\n  {'─'*55}")
        print(f"  📋 Sheet: [{sheet_name}]")
        print(f"  Dimensi: {len(df):,} baris × {len(df.columns)} kolom")
        
        null_total = df.isnull().sum().sum()
        null_pct   = df.isnull().mean().mean() * 100
        dups       = df.duplicated().sum()
        
        print(f"  Kualitas: {100-null_pct:.0f}% lengkap | "
              f"Missing: {null_total:,} | Duplikat: {dups:,}")
        
        # Tipe kolom
        num_cols  = df.select_dtypes(include='number').columns.tolist()
        text_cols = df.select_dtypes(include='object').columns.tolist()
        date_cols = [c for c in df.columns if any(k in str(c).lower() 
                     for k in ['tanggal','date','tgl'])]
        
        print(f"  Numerik: {num_cols[:4]}{'...' if len(num_cols)>4 else ''}")
        print(f"  Teks   : {text_cols[:4]}{'...' if len(text_cols)>4 else ''}")
        if date_cols:
            print(f"  Tanggal: {date_cols}")
        
        if len(df) > 0:
            print(f"\n  Preview ({min(max_rows_preview,len(df))} baris):")
            print(df.head(max_rows_preview).to_string(max_cols=6, max_colwidth=20))
    
    print(f"\n{'═'*60}\n")
```

---

## ═══════════════════════════════════════════════════
## 3. HANDLING FILE BERMASALAH
## ═══════════════════════════════════════════════════

```python
def safe_read_with_fallback(filepath):
    """
    Baca file dengan multiple fallback strategies.
    Menangani: password, corrupt, legacy, encoding errors.
    """
    ext = Path(filepath).suffix.lower()
    
    strategies = []
    
    if ext == '.xlsx':
        strategies = [
            ('openpyxl_standard', lambda: pd.read_excel(filepath, sheet_name=None)),
            ('openpyxl_readonly', lambda: _read_xlsx_readonly(filepath)),
            ('xlrd_fallback', lambda: pd.read_excel(filepath, engine='xlrd', sheet_name=None)),
        ]
    elif ext == '.xls':
        strategies = [
            ('xlrd', lambda: pd.read_excel(filepath, engine='xlrd', sheet_name=None)),
        ]
    elif ext == '.csv':
        strategies = [
            ('csv_smart', lambda: {k: v for k, v in [('Sheet1', _read_csv_smart(filepath)[0])]}),
            ('csv_utf8', lambda: {'Sheet1': pd.read_csv(filepath, encoding='utf-8', on_bad_lines='skip')}),
            ('csv_latin1', lambda: {'Sheet1': pd.read_csv(filepath, encoding='latin-1', on_bad_lines='skip')}),
            ('csv_cp1252', lambda: {'Sheet1': pd.read_csv(filepath, encoding='cp1252', on_bad_lines='skip')}),
        ]
    else:
        return read_any_excel(filepath)
    
    last_error = None
    for strategy_name, strategy_fn in strategies:
        try:
            print(f"  Mencoba: {strategy_name}...")
            df_dict = strategy_fn()
            print(f"  ✅ Berhasil dengan: {strategy_name}")
            return df_dict
        except Exception as e:
            last_error = e
            print(f"  ❌ Gagal {strategy_name}: {e}")
            # Cek apakah password-protected
            if any(kw in str(e).lower() for kw in ['password','encrypted','protected']):
                print("  ⚠️ File terproteksi password — tidak bisa dibuka tanpa password")
                return {}
    
    raise ValueError(f"Semua strategi gagal. Error terakhir: {last_error}")

def _read_xlsx_readonly(filepath):
    """Baca XLSX dalam mode read-only untuk file besar"""
    wb = load_workbook(filepath, read_only=True, data_only=True)
    df_dict = {}
    
    for sheet_name in wb.sheetnames:
        ws = wb[sheet_name]
        data = []
        headers = None
        
        for r_idx, row in enumerate(ws.iter_rows(values_only=True)):
            if r_idx == 0:
                headers = [str(c) if c is not None else f"Col_{i}" 
                           for i, c in enumerate(row)]
            else:
                data.append(row)
            
            if r_idx > 100000:  # Safety limit
                print(f"⚠️ [{sheet_name}] Dipotong di 100k baris")
                break
        
        if headers and data:
            df_dict[sheet_name] = pd.DataFrame(data, columns=headers)
    
    wb.close()
    return df_dict
```

---

## ═══════════════════════════════════════════════════
## 4. LARGE FILE HANDLING (> 10MB)
## ═══════════════════════════════════════════════════

```python
def handle_large_file(filepath, chunk_size=50000):
    """
    Strategi untuk file besar:
    - Streaming read dengan openpyxl read_only
    - Chunked pandas read untuk CSV
    - Summary preview tanpa load semua
    """
    size_mb = Path(filepath).stat().st_size / (1024 * 1024)
    print(f"📦 File besar: {size_mb:.1f} MB")
    
    ext = Path(filepath).suffix.lower()
    
    if ext == '.csv':
        # Chunked read untuk CSV besar
        print(f"  Mode: Chunked CSV (chunk={chunk_size:,} baris)")
        
        # First pass: count rows dan preview
        try:
            preview = pd.read_csv(filepath, nrows=5, sep=None, engine='python')
            total_rows = sum(1 for _ in open(filepath, encoding='utf-8', errors='ignore')) - 1
            print(f"  Total baris (estimasi): {total_rows:,}")
            print(f"  Preview:\n{preview.to_string()}")
        except Exception as e:
            print(f"  Preview error: {e}")
        
        # Process in chunks
        chunks = []
        for chunk in pd.read_csv(filepath, chunksize=chunk_size, 
                                  sep=None, engine='python', on_bad_lines='skip'):
            chunks.append(chunk)
            print(f"  Loaded chunk: {len(chunk):,} baris")
        
        df = pd.concat(chunks, ignore_index=True)
        print(f"  ✅ Total loaded: {len(df):,} baris")
        return {'Sheet1': df}
    
    elif ext in ('.xlsx', '.xlsm'):
        # Streaming openpyxl
        print(f"  Mode: Streaming XLSX (read_only)")
        wb = load_workbook(filepath, read_only=True, data_only=True)
        df_dict = {}
        
        for sheet_name in wb.sheetnames:
            ws = wb[sheet_name]
            data = []
            headers = None
            row_count = 0
            
            for row in ws.iter_rows(values_only=True):
                if headers is None:
                    headers = [str(c) if c is not None else f"Col_{i}" 
                               for i, c in enumerate(row)]
                else:
                    data.append(row)
                    row_count += 1
                
                if row_count % 10000 == 0:
                    print(f"  [{sheet_name}] Loading... {row_count:,} baris")
            
            if headers:
                df_dict[sheet_name] = pd.DataFrame(data, columns=headers)
                print(f"  ✅ [{sheet_name}]: {row_count:,} baris")
        
        wb.close()
        return df_dict
    
    else:
        return read_any_excel(filepath)
```

---

## ═══════════════════════════════════════════════════
## 5. MULTI-FILE DETECTION & ROUTING
## ═══════════════════════════════════════════════════

```python
def detect_multi_file_operation(files):
    """
    Deteksi operasi yang harus dilakukan ketika ada multiple file.
    Return: operation type + file groupings
    """
    if len(files) == 0:
        return 'none', []
    elif len(files) == 1:
        return 'single', files
    elif len(files) == 2:
        # Cek apakah format sama — mungkin compare
        if files[0]['ext'] == files[1]['ext']:
            return 'compare_or_merge', files
        else:
            return 'merge_different_formats', files
    else:
        # Multiple files — biasanya merge/consolidate
        return 'consolidate', files

def auto_route_multiple_files():
    """
    Auto-detect semua file upload dan tentukan operasi terbaik.
    Panggil ini di awal jika ada banyak file.
    """
    all_files = list_uploads()
    
    if not all_files:
        print("❌ Tidak ada file upload ditemukan")
        return None, []
    
    excel_files = [f for f in all_files if f['ext'] in ('.xlsx','.xlsm','.xls','.ods','.xlsb')]
    csv_files   = [f for f in all_files if f['ext'] in ('.csv','.tsv','.txt')]
    other_files = [f for f in all_files if f not in excel_files and f not in csv_files]
    
    print(f"File terdeteksi:")
    print(f"  Excel : {len(excel_files)} file")
    print(f"  CSV   : {len(csv_files)} file")
    print(f"  Lain  : {len(other_files)} file")
    
    all_spreadsheet = excel_files + csv_files
    op, files = detect_multi_file_operation(all_spreadsheet)
    
    print(f"\n🎯 Operasi terdeteksi: {op.upper()}")
    return op, files
```

---

## ═══════════════════════════════════════════════════
## 6. SAVE & OUTPUT PATTERNS
## ═══════════════════════════════════════════════════

```python
def save_output(wb, filename, recalculate=True):
    """
    Simpan workbook ke output directory.
    Jalankan recalc sebelum save jika diminta.
    """
    import subprocess, json
    
    output_path = OUTPUT_DIR / filename
    
    # Save
    wb.save(str(output_path))
    print(f"💾 Saved: {output_path}")
    
    # Recalculate
    if recalculate:
        result = subprocess.run(
            ['python', '/mnt/skills/public/xlsx/scripts/recalc.py', str(output_path)],
            capture_output=True, text=True, timeout=60
        )
        try:
            data = json.loads(result.stdout)
            status = data.get('status', 'unknown')
            n_formula = data.get('total_formulas', 0)
            
            if status == 'errors_found':
                print(f"⚠️ Recalc: {data.get('total_errors',0)} errors ditemukan")
            else:
                print(f"✅ Recalc: {n_formula} formula OK")
        except:
            pass  # Recalc output tidak selalu JSON parseable
    
    return str(output_path)

def save_multiple_sheets_csv(df_dict, base_name="output"):
    """Export setiap sheet sebagai CSV terpisah"""
    saved = []
    for sheet_name, df in df_dict.items():
        safe_name = re.sub(r'[^A-Za-z0-9_-]', '_', sheet_name)
        path = OUTPUT_DIR / f"{base_name}_{safe_name}.csv"
        df.to_csv(str(path), index=False, encoding='utf-8-sig')  # utf-8-sig = Excel-compatible
        saved.append(str(path))
        print(f"✅ CSV: {path.name}")
    return saved
```

---

## ═══════════════════════════════════════════════════
## 7. QUICK CHECKLIST — FILE INTAKE
## ═══════════════════════════════════════════════════

```
Ketika ada file upload, jalankan URUT:

□ 1. list_uploads()              → Lihat semua file
□ 2. prep_file(filename)         → Copy ke /home/claude/
□ 3. read_any_excel(work_path)   → Baca + profiling otomatis
□ 4. detect_domain(df_dict)      → Domain bisnis
□ 5. calculate_dq_score(df_dict) → Data quality score
□ 6. detect_anomalies(df, name)  → Temukan anomali
□ 7. smart_detect_types(df)      → Type mapping untuk format
□ 8. generate_suggestions(...)   → Saran improvement

Setelah itu:
□ 9.  Kerjakan instruksi user
□ 10. SMART ENHANCE (tambahan proaktif)
□ 11. run_full_validation(...)
□ 12. iterative_validate_and_fix(...) jika ada error
□ 13. save_output(wb, filename)
□ 14. present_files(output_path)
□ 15. Laporan RIAN format
```

---

## ═══════════════════════════════════════════════════
## 8. LARGE FILE HANDLING (v4 NEW)
## ═══════════════════════════════════════════════════

> Strategy untuk file besar yang bisa crash openpyxl atau memakan terlalu banyak RAM.

```python
import os

LARGE_FILE_THRESHOLDS = {
    'warn_mb':      5,     # MB — tampilkan warning
    'confirm_mb':   20,    # MB — minta konfirmasi user
    'max_rows_full':50_000, # rows — load penuh
    'sample_rows':  10_000, # rows — sampel untuk analysis
    'chunk_rows':   5_000,  # rows — proses per chunk
}

def check_file_size(filepath: str) -> dict:
    """
    Cek ukuran file dan tentukan strategy loading.
    Return: {size_mb, row_estimate, strategy, warning}
    """
    size_bytes = os.path.getsize(filepath)
    size_mb = size_bytes / (1024 * 1024)
    
    result = {
        'size_mb': round(size_mb, 2),
        'filepath': filepath,
        'strategy': 'full',
        'warning': None,
        'row_estimate': None,
    }
    
    if size_mb > LARGE_FILE_THRESHOLDS['confirm_mb']:
        result['strategy'] = 'sample'
        result['warning'] = (
            f"⚠️ FILE BESAR: {size_mb:.1f} MB\n"
            f"Untuk analisis, RIAN akan load sample {LARGE_FILE_THRESHOLDS['sample_rows']:,} baris pertama.\n"
            f"Ketik 'load semua' jika ingin proses seluruh data (mungkin lambat)."
        )
    elif size_mb > LARGE_FILE_THRESHOLDS['warn_mb']:
        result['strategy'] = 'full_with_warning'
        result['warning'] = f"ℹ️ File cukup besar ({size_mb:.1f} MB) — proses mungkin membutuhkan beberapa detik."
    
    print(f"\n📁 File: {os.path.basename(filepath)}")
    print(f"   Ukuran: {size_mb:.1f} MB | Strategy: {result['strategy']}")
    if result['warning']:
        print(f"   {result['warning']}")
    
    return result


def read_excel_large(filepath: str, sample_only: bool = False) -> dict:
    """
    Baca file Excel besar dengan openpyxl read_only mode.
    Jauh lebih hemat RAM untuk file >10MB.
    """
    from openpyxl import load_workbook
    import pandas as pd
    
    size_info = check_file_size(filepath)
    use_sample = sample_only or size_info['strategy'] == 'sample'
    
    df_dict = {}
    
    # read_only=True untuk file besar — bisa 5-10x lebih cepat
    wb_ro = load_workbook(filepath, read_only=True, data_only=True)
    
    for sheet_name in wb_ro.sheetnames:
        ws = wb_ro[sheet_name]
        
        rows_data = []
        header = None
        row_count = 0
        max_rows = LARGE_FILE_THRESHOLDS['sample_rows'] if use_sample else None
        
        for row in ws.iter_rows(values_only=True):
            if header is None:
                header = [str(c) if c is not None else f"Col_{i}" 
                          for i, c in enumerate(row)]
                continue
            
            rows_data.append(row)
            row_count += 1
            
            if max_rows and row_count >= max_rows:
                print(f"   [{sheet_name}] Loaded {row_count:,} baris (sample mode)")
                break
        
        if rows_data:
            df = pd.DataFrame(rows_data, columns=header)
            df_dict[sheet_name] = df
            
            total_indicator = f"(dari estimasi lebih banyak)" if use_sample else ""
            print(f"   [{sheet_name}] {len(df):,} baris × {len(df.columns)} kolom {total_indicator}")
    
    wb_ro.close()
    return df_dict


def process_in_chunks(filepath: str, sheet_name: str, 
                       process_func, chunk_size: int = 5000) -> list:
    """
    Proses file besar baris per baris tanpa load ke memory sekaligus.
    process_func dipanggil dengan setiap chunk DataFrame.
    """
    from openpyxl import load_workbook
    import pandas as pd
    
    print(f"\n🔄 Chunked processing: {os.path.basename(filepath)} — sheet '{sheet_name}'")
    
    wb_ro = load_workbook(filepath, read_only=True, data_only=True)
    ws = wb_ro[sheet_name]
    
    header = None
    chunk_rows = []
    chunk_num = 0
    results = []
    total_rows = 0
    
    for row in ws.iter_rows(values_only=True):
        if header is None:
            header = list(row)
            continue
        
        chunk_rows.append(row)
        
        if len(chunk_rows) >= chunk_size:
            chunk_num += 1
            total_rows += len(chunk_rows)
            df_chunk = pd.DataFrame(chunk_rows, columns=header)
            result = process_func(df_chunk, chunk_num)
            results.append(result)
            chunk_rows = []
            print(f"   Chunk {chunk_num}: {total_rows:,} baris diproses...")
            
    # Sisa rows
    if chunk_rows:
        chunk_num += 1
        total_rows += len(chunk_rows)
        df_chunk = pd.DataFrame(chunk_rows, columns=header)
        result = process_func(df_chunk, chunk_num)
        results.append(result)
    
    wb_ro.close()
    print(f"✅ Selesai: {total_rows:,} baris dalam {chunk_num} chunk")
    return results


# ── Update Quick Checklist untuk file besar ────────────────────────────
# TAMBAHKAN sebelum step 3 (read_any_excel):
#   2b. check_file_size(work_path)
#   → Jika size_mb > 20: gunakan read_excel_large(work_path, sample_only=True)
#   → Jika size_mb > 5:  gunakan read_excel_large(work_path, sample_only=False)
#   → Jika size_mb <= 5: gunakan read_any_excel(work_path) seperti biasa
```

### Updated Quick Checklist (dengan Large File Check)
```
Ketika ada file upload, jalankan URUT:

□ 1.  list_uploads()                     → Lihat semua file
□ 2.  prep_file(filename)                → Copy ke /home/claude/
□ 2b. check_file_size(work_path)         → ⭐ NEW: cek ukuran & tentukan strategy
□ 3.  read_any_excel / read_excel_large  → Baca sesuai strategy (full/sample/chunked)
□ 4.  detect_domain(df_dict)             → Domain bisnis
□ 5.  calculate_dq_score(df_dict)        → Data quality score
□ 6.  detect_anomalies(df, name)         → Temukan anomali
□ 7.  smart_detect_types(df)             → Type mapping untuk format
□ 8.  generate_suggestions(...)          → Saran improvement (15 domain)

Setelah itu:
□ 9.  Kerjakan instruksi user
□ 10. SMART ENHANCE (tambahan proaktif)
□ 11. run_full_validation(...)
□ 12. iterative_validate_and_fix(...) jika ada error
□ 13. save_output(wb, filename)
□ 14. present_files(output_path)
□ 15. Laporan RIAN format
```

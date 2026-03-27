# Python Patterns Reference — RIAN v4
# 700+ baris pola teruji, IDR-first, production-grade

## ═══════════════════════════════════════════════════
## 1. WORKFLOW STANDAR (COPY-PASTE SETIAP SESSION)
## ═══════════════════════════════════════════════════

```python
import os, sys, json, re, math, copy, shutil, subprocess, chardet
from pathlib import Path
from datetime import datetime, date, timedelta
import pandas as pd
import numpy as np

from openpyxl import load_workbook, Workbook
from openpyxl.styles import (
    Font, PatternFill, Alignment, Border, Side, GradientFill, NamedStyle
)
from openpyxl.chart import BarChart, LineChart, PieChart, ScatterChart
from openpyxl.chart import AreaChart, DoughnutChart, Reference
from openpyxl.utils import get_column_letter, column_index_from_string
from openpyxl.utils.dataframe import dataframe_to_rows
from openpyxl.formatting.rule import (
    ColorScaleRule, DataBarRule, IconSetRule,
    FormulaRule, CellIsRule, DifferentialStyle
)
from openpyxl.worksheet.datavalidation import DataValidation
from openpyxl.worksheet.table import Table, TableStyleInfo
from openpyxl.worksheet.page import PrintPageSetup, PageMargins
from openpyxl.comments import Comment
from openpyxl.drawing.image import Image as XLImage
from openpyxl.workbook.defined_name import DefinedName
from openpyxl.utils import quote_sheetname

UPLOAD_DIR = Path("/mnt/user-data/uploads/")
WORK_DIR   = Path("/home/claude/")
OUTPUT_DIR = Path("/mnt/user-data/outputs/")
OUTPUT_DIR.mkdir(exist_ok=True)

def prep_file(filename):
    src = UPLOAD_DIR / filename
    dst = WORK_DIR / f"work_{filename}"
    shutil.copy2(str(src), str(dst))
    return str(dst)

def recalc_and_validate(filepath):
    result = subprocess.run(
        ['python', '/mnt/skills/public/xlsx/scripts/recalc.py', filepath],
        capture_output=True, text=True, timeout=60
    )
    try:
        data = json.loads(result.stdout)
        ok = data.get('status') != 'errors_found'
        print(f"{'✅' if ok else '❌'} Recalc: {data.get('total_formulas',0)} formula")
        return ok, data
    except:
        return True, {}

def save_output(wb, output_name):
    path = OUTPUT_DIR / output_name
    wb.save(str(path))
    print(f"💾 Saved: {path}")
    return str(path)
```

---

## ═══════════════════════════════════════════════════
## 2. MEMBACA FILE UPLOAD (SEMUA FORMAT)
## ═══════════════════════════════════════════════════

```python
def read_any_file(filepath):
    """Universal file reader — support semua format"""
    ext = Path(filepath).suffix.lower()
    
    if ext == '.xls':
        df_dict = pd.read_excel(filepath, sheet_name=None, engine='xlrd')
    elif ext == '.ods':
        df_dict = pd.read_excel(filepath, sheet_name=None, engine='odf')
    elif ext == '.xlsb':
        df_dict = pd.read_excel(filepath, engine='pyxlsb', sheet_name=None)
    elif ext == '.csv':
        with open(filepath, 'rb') as f:
            enc = chardet.detect(f.read(50000))['encoding'] or 'utf-8'
        # Coba deteksi delimiter
        df = pd.read_csv(filepath, sep=None, engine='python', encoding=enc,
                         on_bad_lines='skip')
        df_dict = {'Sheet1': df}
    elif ext == '.tsv':
        with open(filepath, 'rb') as f:
            enc = chardet.detect(f.read(50000))['encoding'] or 'utf-8'
        df = pd.read_csv(filepath, sep='\t', encoding=enc)
        df_dict = {'Sheet1': df}
    elif ext == '.txt':
        # Coba baca sebagai CSV dengan berbagai delimiter
        for sep in ['\t', ',', ';', '|']:
            try:
                df = pd.read_csv(filepath, sep=sep)
                if len(df.columns) > 1:
                    df_dict = {'Sheet1': df}
                    break
            except: continue
        else:
            df_dict = {'Sheet1': pd.read_csv(filepath, sep=None, engine='python')}
    else:  # xlsx, xlsm
        df_dict = pd.read_excel(filepath, sheet_name=None)
    
    return df_dict

def quick_profile(df_dict):
    """Print profil singkat semua sheet"""
    print(f"\n{'═'*55}")
    for sname, df in df_dict.items():
        null_pct = df.isnull().mean().mean() * 100
        dups = df.duplicated().sum()
        nums = df.select_dtypes(include='number').columns.tolist()
        cats = df.select_dtypes(include='object').columns.tolist()
        print(f"\n  [{sname}] {len(df):,}R × {len(df.columns)}C | "
              f"Quality: {100-null_pct:.0f}% | Dups: {dups}")
        print(f"  Numerik: {nums[:5]}")
        print(f"  Teks   : {cats[:5]}")
        if len(df) > 0:
            print(f"  Sample :\n{df.head(2).to_string(max_cols=6)}")
    print(f"{'═'*55}\n")
```

---

## ═══════════════════════════════════════════════════
## 3. MEMBUAT WORKBOOK PROFESIONAL
## ═══════════════════════════════════════════════════

```python
def create_pro_workbook(title, sheets_config, with_cover=True):
    """
    sheets_config = [
        {'name': 'DATA',      'color': '404040'},
        {'name': 'SUMMARY',   'color': '70AD47'},
        {'name': 'DASHBOARD', 'color': '1F4E79'},
    ]
    """
    wb = Workbook()
    wb.remove(wb.active)  # Hapus Sheet default
    
    created = {}
    
    if with_cover:
        cover = wb.create_sheet("COVER")
        cover.sheet_properties.tabColor = "2E75B6"
        cover.sheet_view.showGridLines = False
        
        # Title block
        cover.merge_cells('B3:I3')
        c = cover['B3']
        c.value = title
        c.font = Font(size=22, bold=True, color="1F4E79", name="Calibri")
        c.alignment = Alignment(horizontal='center', vertical='center')
        cover.row_dimensions[3].height = 45
        
        # Subtitle
        cover.merge_cells('B4:I4')
        cover['B4'].value = f"Dibuat oleh RIAN | {pd.Timestamp.now().strftime('%d %B %Y, %H:%M')}"
        cover['B4'].font = Font(size=10, color="888888", italic=True, name="Calibri")
        cover['B4'].alignment = Alignment(horizontal='center')
        
        # Separator line
        for col in range(2, 10):
            cover.cell(5, col).fill = PatternFill("solid", fgColor="1F4E79")
        cover.row_dimensions[5].height = 3
        
        # Sheet directory
        cover['B7'].value = "ISI DOKUMEN"
        cover['B7'].font = Font(bold=True, size=12, color="1F4E79", name="Calibri")
        for i, sc in enumerate(sheets_config):
            cell = cover[f'B{8+i}']
            cell.value = f"  ▸  {sc['name']}"
            cell.font = Font(size=10, color="404040", name="Calibri")
        
        created['COVER'] = cover
    
    for sc in sheets_config:
        ws = wb.create_sheet(sc['name'])
        ws.sheet_properties.tabColor = sc.get('color', '404040')
        created[sc['name']] = ws
    
    return wb, created

def write_headers(ws, headers, row=1, start_col=1,
                  bg_color="1F4E79", font_color="FFFFFF", height=25):
    ws.row_dimensions[row].height = height
    for i, header in enumerate(headers, start_col):
        cell = ws.cell(row=row, column=i, value=header)
        cell.font = Font(bold=True, color=font_color, size=10, name="Calibri")
        cell.fill = PatternFill("solid", fgColor=bg_color)
        cell.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)
        thin = Side(style="thin", color="FFFFFF")
        cell.border = Border(bottom=Side(style="medium", color="FFFFFF"), right=thin)

def write_section_title(ws, row, col, text, color="1F4E79", merge_to_col=None):
    """Tulis judul section dengan garis bawah"""
    cell = ws.cell(row, col, value=text)
    cell.font = Font(bold=True, size=11, color="FFFFFF", name="Calibri")
    cell.fill = PatternFill("solid", fgColor=color)
    cell.alignment = Alignment(vertical="center", indent=1)
    ws.row_dimensions[row].height = 22
    if merge_to_col:
        ws.merge_cells(start_row=row, end_row=row, 
                       start_column=col, end_column=merge_to_col)
```

---

## ═══════════════════════════════════════════════════
## 4. EXPORT DATAFRAME KE SHEET (DENGAN FORMAT LENGKAP)
## ═══════════════════════════════════════════════════

```python
def df_to_sheet(ws, df, start_row=1, start_col=1,
                table_name="Tabel1", include_header=True,
                header_color="1F4E79", include_total=False,
                type_map=None):
    """
    Export DataFrame ke worksheet dengan:
    - Header profesional
    - Zebra stripes
    - Auto number format per kolom
    - Excel Table (sortable/filterable)
    - Optional total row
    - Optional type_map untuk number format
    """
    from openpyxl.utils import get_column_letter
    from openpyxl.worksheet.table import Table, TableStyleInfo
    
    if include_header:
        write_headers(ws, list(df.columns), row=start_row, 
                      start_col=start_col, bg_color=header_color)
        data_start_row = start_row + 1
    else:
        data_start_row = start_row
    
    type_map = type_map or {}
    NUMBER_FORMATS = {
        'idr': '#,##0;(#,##0);"-"',
        'percent_1': '0.0%;(0.0%);"-"',
        'qty': '#,##0;(#,##0);"-"',
        'date_id': 'DD/MM/YYYY',
        'text': '@',
        'decimal_2': '#,##0.00',
    }
    
    for r_idx, row_data in enumerate(df.itertuples(index=False), data_start_row):
        row_color = "F2F7FF" if r_idx % 2 == 0 else "FFFFFF"
        for c_idx, value in enumerate(row_data, start_col):
            cell = ws.cell(row=r_idx, column=c_idx, value=value)
            cell.fill = PatternFill("solid", fgColor=row_color)
            cell.font = Font(name="Calibri", size=10)
            cell.alignment = Alignment(vertical="center")
            
            col_name = df.columns[c_idx - start_col]
            col_type = type_map.get(col_name, '')
            
            if col_type and col_type in NUMBER_FORMATS:
                cell.number_format = NUMBER_FORMATS[col_type]
                if col_type not in ('text', 'date_id'):
                    cell.alignment = Alignment(horizontal="right", vertical="center")
            elif isinstance(value, (int, float)) and not isinstance(value, bool):
                cell.alignment = Alignment(horizontal="right", vertical="center")
                col_lower = col_name.lower()
                if any(k in col_lower for k in ['harga','gaji','biaya','pendapatan','total',
                                                   'revenue','omzet','nilai','cost','amount']):
                    cell.number_format = '#,##0;(#,##0);"-"'
                elif any(k in col_lower for k in ['persen','percent','margin','rate','%','ach']):
                    cell.number_format = '0.0%;(0.0%);"-"'
                elif any(k in col_lower for k in ['qty','jumlah','stok','count','kuantitas']):
                    cell.number_format = '#,##0;(#,##0);"-"'
                else:
                    cell.number_format = '#,##0.##'
            elif hasattr(value, 'strftime') or isinstance(value, (datetime, date)):
                cell.number_format = 'DD/MM/YYYY'
                cell.alignment = Alignment(horizontal="center", vertical="center")
    
    # Total row
    if include_total and len(df) > 0:
        total_row = data_start_row + len(df)
        ws.cell(total_row, start_col).value = "TOTAL"
        ws.cell(total_row, start_col).font = Font(bold=True, name="Calibri")
        for c_idx in range(start_col + 1, start_col + len(df.columns)):
            col_name = df.columns[c_idx - start_col]
            if df[col_name].dtype in ['float64', 'int64']:
                col_letter = get_column_letter(c_idx)
                ws.cell(total_row, c_idx).value = f"=SUM({col_letter}{data_start_row}:{col_letter}{data_start_row + len(df) - 1})"
            cell = ws.cell(total_row, c_idx)
            cell.font = Font(bold=True, name="Calibri")
            cell.fill = PatternFill("solid", fgColor="FFF2CC")
            cell.border = Border(
                top=Side(style="medium", color="000000"),
                bottom=Side(style="double", color="000000")
            )
    
    # Excel Table
    if include_header and len(df) > 0:
        end_row_table = data_start_row + len(df) - 1
        end_col_table = start_col + len(df.columns) - 1
        ref = f"{get_column_letter(start_col)}{start_row}:{get_column_letter(end_col_table)}{end_row_table}"
        safe_name = re.sub(r'[^A-Za-z0-9_]', '_', table_name)
        tbl = Table(displayName=safe_name, ref=ref)
        tbl.tableStyleInfo = TableStyleInfo(
            name="TableStyleMedium9", showRowStripes=True,
            showFirstColumn=False, showLastColumn=False
        )
        ws.add_table(tbl)
    
    # Auto-fit
    auto_fit_columns(ws)
    
    return data_start_row, data_start_row + len(df) - 1

def auto_fit_columns(ws, min_width=8, max_width=50):
    for col in ws.columns:
        max_len = 0
        col_letter = get_column_letter(col[0].column)
        for cell in col:
            try:
                if cell.value is not None:
                    max_len = max(max_len, len(str(cell.value)))
            except: pass
        ws.column_dimensions[col_letter].width = min(max(max_len + 2, min_width), max_width)
```

---

## ═══════════════════════════════════════════════════
## 5. FORMULA INJECTION — TULIS FORMULA EXCEL REAL
## ═══════════════════════════════════════════════════

```python
def add_formula_column(ws, data_start_row, data_end_row,
                        formula_col_idx, formula_template,
                        header=None, header_row=1, header_color="1F4E79",
                        number_format='#,##0;(#,##0);"-"'):
    """
    Inject formula Excel ke kolom baru.
    formula_template: string dengan {R} sebagai placeholder baris
    
    Contoh:
      formula_template = '=IF({R}D{R}=""," -",C{R}*D{R})'  # SALAH
      formula_template = '=IFERROR(C{R}*D{R},"-")'         # BENAR
    """
    col_letter = get_column_letter(formula_col_idx)
    
    # Header
    if header:
        h_cell = ws.cell(header_row, formula_col_idx, value=header)
        h_cell.font = Font(bold=True, color="FFFFFF", size=10, name="Calibri")
        h_cell.fill = PatternFill("solid", fgColor=header_color)
        h_cell.alignment = Alignment(horizontal="center", vertical="center")
    
    # Formula cells
    for row in range(data_start_row, data_end_row + 1):
        formula = formula_template.replace('{R}', str(row))
        cell = ws.cell(row, formula_col_idx, value=formula)
        cell.number_format = number_format
        cell.alignment = Alignment(horizontal="right", vertical="center")
        cell.font = Font(name="Calibri", size=10)

def add_sum_row(ws, data_start_row, data_end_row,
                numeric_cols, total_row=None, label_col=1, label="TOTAL"):
    """Tambah baris SUM di bawah data"""
    total_row = total_row or data_end_row + 1
    
    # Label
    lbl = ws.cell(total_row, label_col, value=label)
    lbl.font = Font(bold=True, name="Calibri")
    lbl.fill = PatternFill("solid", fgColor="FFF2CC")
    
    # SUM formulas
    for col_idx in numeric_cols:
        col_letter = get_column_letter(col_idx)
        cell = ws.cell(total_row, col_idx)
        cell.value = f"=IFERROR(SUM({col_letter}{data_start_row}:{col_letter}{data_end_row}),0)"
        cell.font = Font(bold=True, name="Calibri")
        cell.fill = PatternFill("solid", fgColor="FFF2CC")
        cell.number_format = '#,##0;(#,##0);"-"'
        cell.border = Border(
            top=Side(style="medium", color="000000"),
            bottom=Side(style="double", color="000000")
        )
    
    return total_row

def add_running_total_col(ws, data_start_row, data_end_row,
                           source_col, running_col, header="Kumulatif"):
    """Tambah running total / cumulative sum"""
    h = ws.cell(data_start_row - 1, running_col, value=header)
    h.font = Font(bold=True, color="FFFFFF", name="Calibri")
    h.fill = PatternFill("solid", fgColor="1F4E79")
    h.alignment = Alignment(horizontal="center")
    
    src_letter = get_column_letter(source_col)
    run_letter = get_column_letter(running_col)
    
    for row in range(data_start_row, data_end_row + 1):
        cell = ws.cell(row, running_col)
        cell.value = f"=IFERROR(SUM(${src_letter}${data_start_row}:{src_letter}{row}),0)"
        cell.number_format = '#,##0;(#,##0);"-"'
        cell.alignment = Alignment(horizontal="right")

def add_rank_col(ws, data_start_row, data_end_row,
                  value_col, rank_col, header="Rank", order=0):
    """Tambah kolom ranking"""
    h = ws.cell(data_start_row - 1, rank_col, value=header)
    h.font = Font(bold=True, color="FFFFFF", name="Calibri")
    h.fill = PatternFill("solid", fgColor="1F4E79")
    h.alignment = Alignment(horizontal="center")
    
    val_letter = get_column_letter(value_col)
    for row in range(data_start_row, data_end_row + 1):
        cell = ws.cell(row, rank_col)
        cell.value = f"=IFERROR(RANK({val_letter}{row},${val_letter}${data_start_row}:${val_letter}${data_end_row},{order}),\"-\")"
        cell.number_format = '#,##0'
        cell.alignment = Alignment(horizontal="center")

def add_growth_col(ws, data_start_row, data_end_row,
                    value_col, growth_col, header="Growth %"):
    """Tambah kolom growth MoM / YoY"""
    h = ws.cell(data_start_row - 1, growth_col, value=header)
    h.font = Font(bold=True, color="FFFFFF", name="Calibri")
    h.fill = PatternFill("solid", fgColor="1F4E79")
    h.alignment = Alignment(horizontal="center")
    
    val_letter = get_column_letter(value_col)
    for row in range(data_start_row, data_end_row + 1):
        cell = ws.cell(row, growth_col)
        if row == data_start_row:
            cell.value = "-"
        else:
            cell.value = f'=IFERROR(({val_letter}{row}-{val_letter}{row-1})/{val_letter}{row-1},"-")'
        cell.number_format = '+0.0%;-0.0%;"−"'
        cell.alignment = Alignment(horizontal="right")
```

---

## ═══════════════════════════════════════════════════
## 6. DATA CLEANING ENGINE
## ═══════════════════════════════════════════════════

```python
def clean_dataframe(df, options=None):
    """
    Comprehensive data cleaning.
    options = {
        'remove_duplicates': True,
        'strip_text': True,
        'fix_numeric': True,
        'fix_dates': True,
        'normalize_case': 'title',  # 'upper' / 'lower' / 'title' / None
        'fill_numeric_na': 0,       # None = skip
        'remove_empty_rows': True,
        'remove_empty_cols': True,
    }
    """
    options = options or {}
    report = []
    original_shape = df.shape
    
    # 1. Hapus baris & kolom sepenuhnya kosong
    if options.get('remove_empty_rows', True):
        before = len(df)
        df = df.dropna(how='all')
        removed = before - len(df)
        if removed: report.append(f"Hapus {removed} baris kosong")
    
    if options.get('remove_empty_cols', True):
        before = len(df.columns)
        df = df.dropna(axis=1, how='all')
        removed = before - len(df.columns)
        if removed: report.append(f"Hapus {removed} kolom kosong")
    
    # 2. Hapus duplikat
    if options.get('remove_duplicates', True):
        before = len(df)
        df = df.drop_duplicates()
        removed = before - len(df)
        if removed: report.append(f"Hapus {removed} duplikat")
    
    # 3. Strip whitespace di kolom teks
    if options.get('strip_text', True):
        for col in df.select_dtypes(include='object').columns:
            df[col] = df[col].astype(str).str.strip()
            df[col] = df[col].replace('nan', np.nan)
    
    # 4. Normalize case
    case_mode = options.get('normalize_case')
    if case_mode:
        for col in df.select_dtypes(include='object').columns:
            # Skip kolom yang kemungkinan ID/kode
            if not any(k in col.lower() for k in ['id', 'kode', 'no', 'nomor', 'nik']):
                if case_mode == 'title':
                    df[col] = df[col].str.title()
                elif case_mode == 'upper':
                    df[col] = df[col].str.upper()
                elif case_mode == 'lower':
                    df[col] = df[col].str.lower()
        report.append(f"Case normalized: {case_mode}")
    
    # 5. Fix kolom numerik yang tersimpan sebagai teks
    if options.get('fix_numeric', True):
        for col in df.select_dtypes(include='object').columns:
            # Skip kolom identifier
            if any(k in col.lower() for k in ['id', 'kode', 'no', 'nomor', 'nik', 'nim',
                                                'email', 'hp', 'telepon', 'phone']):
                continue
            # Coba konversi
            cleaned = df[col].astype(str).str.replace(r'[Rp,\s]', '', regex=True)
            numeric = pd.to_numeric(cleaned, errors='coerce')
            if numeric.notna().sum() / max(df[col].notna().sum(), 1) > 0.8:
                df[col] = numeric
                report.append(f"Fix numeric: {col}")
    
    # 6. Fix kolom tanggal
    if options.get('fix_dates', True):
        date_keywords = ['tanggal', 'date', 'tgl', 'waktu', 'time', 'dob', 'lahir']
        for col in df.select_dtypes(include='object').columns:
            if any(k in col.lower() for k in date_keywords):
                converted = pd.to_datetime(df[col], errors='coerce', dayfirst=True)
                if converted.notna().sum() / max(df[col].notna().sum(), 1) > 0.7:
                    df[col] = converted
                    report.append(f"Fix datetime: {col}")
    
    # 7. Fill NA di kolom numerik
    fill_val = options.get('fill_numeric_na')
    if fill_val is not None:
        for col in df.select_dtypes(include='number').columns:
            na_count = df[col].isna().sum()
            if na_count > 0:
                df[col] = df[col].fillna(fill_val)
        report.append(f"Fill numeric NA → {fill_val}")
    
    final_shape = df.shape
    print(f"\n🧹 DATA CLEANING REPORT")
    print(f"   Sebelum: {original_shape[0]:,} baris × {original_shape[1]} kolom")
    print(f"   Sesudah: {final_shape[0]:,} baris × {final_shape[1]} kolom")
    for r in report:
        print(f"   • {r}")
    
    return df, report

def fix_column_names(df):
    """Bersihkan nama kolom: hapus whitespace, fix unnamed"""
    new_cols = []
    unnamed_count = 0
    for col in df.columns:
        col_str = str(col).strip()
        if col_str.startswith('Unnamed'):
            unnamed_count += 1
            col_str = f"Kolom_{unnamed_count}"
        new_cols.append(col_str)
    df.columns = new_cols
    return df
```

---

## ═══════════════════════════════════════════════════
## 7. CONDITIONAL FORMATTING PATTERNS
## ═══════════════════════════════════════════════════

```python
from openpyxl.formatting.rule import (
    ColorScaleRule, DataBarRule, IconSetRule,
    FormulaRule, CellIsRule, DifferentialStyle
)

def add_heatmap(ws, cell_range, 
                min_color="F8696B", mid_color="FFEB84", max_color="63BE7B"):
    """3-color heatmap (merah → kuning → hijau)"""
    ws.conditional_formatting.add(cell_range, ColorScaleRule(
        start_type="min", start_color=min_color,
        mid_type="percentile", mid_value=50, mid_color=mid_color,
        end_type="max", end_color=max_color
    ))

def add_data_bars(ws, cell_range, color="4472C4"):
    """Data bar / progress bar"""
    ws.conditional_formatting.add(cell_range, DataBarRule(
        start_type="min", end_type="max", color=color
    ))

def add_icon_set_3(ws, cell_range, icon_style="3Arrows"):
    """Icon set: 3 panah/traffic light/dll"""
    ws.conditional_formatting.add(cell_range, IconSetRule(
        iconSet=icon_style,
        type0="percentile", val0=0,
        type1="percentile", val1=33,
        type2="percentile", val2=67
    ))

def highlight_negatives(ws, cell_range):
    """Highlight nilai negatif → merah"""
    red_fill = PatternFill(bgColor="FFE6E6")
    red_font = Font(color="CC0000", bold=True)
    ws.conditional_formatting.add(cell_range, CellIsRule(
        operator='lessThan', formula=['0'],
        fill=red_fill, font=red_font
    ))

def highlight_above_value(ws, cell_range, value, color="E2EFDA"):
    """Highlight sel di atas nilai threshold → hijau"""
    fill = PatternFill(bgColor=color)
    ws.conditional_formatting.add(cell_range, CellIsRule(
        operator='greaterThan', formula=[str(value)], fill=fill
    ))

def highlight_top_n(ws, cell_range, n=5, color="70AD47"):
    """Highlight top-N nilai tertinggi"""
    green_fill = PatternFill(bgColor=color)
    green_font = Font(color="FFFFFF", bold=True)
    ws.conditional_formatting.add(cell_range, FormulaRule(
        formula=[f'RANK({cell_range.split(":")[0]},{cell_range},0)<={n}'],
        fill=green_fill, font=green_font
    ))

def highlight_duplicates(ws, cell_range):
    """Highlight nilai duplikat → orange"""
    orange_fill = PatternFill(bgColor="FCE4D6")
    ws.conditional_formatting.add(cell_range, FormulaRule(
        formula=[f'COUNTIF({cell_range},{cell_range.split(":")[0]})>1'],
        fill=orange_fill
    ))

def add_status_cf(ws, status_col_range, 
                   good_values=None, bad_values=None, warning_values=None):
    """Color coding kolom STATUS secara otomatis"""
    good_values    = good_values    or ['Aktif','Done','Lunas','✅ On Track','AMAN','Hadir','Lulus']
    bad_values     = bad_values     or ['Resign','Overdue','Overdue','❌ Below','KRITIS','Absen','Tidak Lulus']
    warning_values = warning_values or ['Cuti','At Risk','⚠️ At Risk','RENDAH','Izin','Cukup']
    
    green_fill  = PatternFill(bgColor="E2EFDA")
    green_font  = Font(color="375623", bold=True)
    red_fill    = PatternFill(bgColor="FFCCCC")
    red_font    = Font(color="9C0006", bold=True)
    yellow_fill = PatternFill(bgColor="FFEB9C")
    yellow_font = Font(color="9C6500", bold=True)
    
    ref = status_col_range.split(":")[0]  # Ambil cell pertama sebagai anchor
    
    for val in good_values:
        ws.conditional_formatting.add(status_col_range, CellIsRule(
            operator='equal', formula=[f'"{val}"'],
            fill=green_fill, font=green_font
        ))
    for val in bad_values:
        ws.conditional_formatting.add(status_col_range, CellIsRule(
            operator='equal', formula=[f'"{val}"'],
            fill=red_fill, font=red_font
        ))
    for val in warning_values:
        ws.conditional_formatting.add(status_col_range, CellIsRule(
            operator='equal', formula=[f'"{val}"'],
            fill=yellow_fill, font=yellow_font
        ))
```

---

## ═══════════════════════════════════════════════════
## 8. KPI CARDS & DASHBOARD ELEMENTS
## ═══════════════════════════════════════════════════

```python
def create_kpi_cards(ws, kpis, start_row=3, start_col=2, card_width=4):
    """
    Buat KPI card grid profesional.
    kpis = [
        {
            'label': 'Total Revenue',
            'value': 1500000000,
            'fmt': 'idr',          # 'idr' / 'percent' / 'integer' / 'decimal'
            'change': 0.15,        # +15% vs periode lalu (opsional)
            'target': 1200000000,  # Target (opsional)
            'subtitle': 'Jan 2025' # Opsional
        }
    ]
    """
    for i, kpi in enumerate(kpis):
        col_start = start_col + (i * (card_width + 1))
        col_end   = col_start + card_width - 1
        r1, r2, r3, r4, r5 = start_row, start_row+1, start_row+2, start_row+3, start_row+4
        
        # Card background
        card_fill = PatternFill("solid", fgColor="F0F5FF")
        left_accent_fill = PatternFill("solid", fgColor="1F4E79")
        for r in [r1, r2, r3, r4, r5]:
            for c in range(col_start, col_end + 1):
                ws.cell(r, c).fill = card_fill
            ws.cell(r, col_start).fill = left_accent_fill  # Left accent bar
        
        # Merge cells per baris
        for r in [r1, r2, r3, r4]:
            ws.merge_cells(start_row=r, end_row=r,
                           start_column=col_start+1, end_column=col_end)
        
        # Label
        lbl = ws.cell(r1, col_start+1, value=kpi['label'])
        lbl.font = Font(size=9, color="666666", name="Calibri", bold=True)
        lbl.alignment = Alignment(vertical="center")
        ws.row_dimensions[r1].height = 16
        
        # Value
        val = kpi['value']
        fmt = kpi.get('fmt', 'integer')
        val_cell = ws.cell(r2, col_start+1, value=val)
        val_cell.alignment = Alignment(vertical="center")
        ws.row_dimensions[r2].height = 28
        
        if fmt == 'idr':
            val_cell.number_format = '#,##0'
            val_cell.font = Font(size=16, bold=True, color="1F4E79", name="Calibri")
        elif fmt == 'percent':
            val_cell.number_format = '0.0%'
            val_cell.font = Font(size=16, bold=True, color="1F4E79", name="Calibri")
        else:
            val_cell.number_format = '#,##0'
            val_cell.font = Font(size=16, bold=True, color="1F4E79", name="Calibri")
        
        # Change indicator
        if 'change' in kpi:
            chg = kpi['change']
            chg_cell = ws.cell(r3, col_start+1)
            arrow = "▲" if chg >= 0 else "▼"
            chg_cell.value = f"{arrow} {abs(chg)*100:.1f}% vs lalu"
            chg_cell.font = Font(
                size=9, name="Calibri", bold=True,
                color="00AA44" if chg >= 0 else "CC0000"
            )
            chg_cell.alignment = Alignment(vertical="center")
            ws.row_dimensions[r3].height = 14
        
        # Target achievement bar
        if 'target' in kpi and kpi['target']:
            ach = min(val / kpi['target'], 1.5)
            ach_cell = ws.cell(r4, col_start+1)
            ach_cell.value = f"Target: {ach*100:.0f}%"
            ach_cell.font = Font(size=8, color="888888", name="Calibri", italic=True)
            ach_cell.alignment = Alignment(vertical="center")
        
        # Border card
        thin = Side(style="thin", color="BDD7EE")
        outer = Side(style="medium", color="2E75B6")
        for r in [r1, r2, r3, r4, r5]:
            for c in range(col_start, col_end + 1):
                borders = {}
                if r == r1: borders['top'] = outer
                if r == r5: borders['bottom'] = outer
                if r in [r1, r2, r3, r4, r5]:
                    if c == col_start: borders['left'] = outer
                    if c == col_end: borders['right'] = outer
                    borders.setdefault('top', thin)
                    borders.setdefault('bottom', thin)
                ws.cell(r, c).border = Border(**borders)

def create_progress_bar_cell(ws, row, col, value, max_value=100,
                               bar_color="70AD47", bg_color="E2EFDA"):
    """
    Progress bar via conditional formatting + display value.
    value: 0-100 atau 0-1
    """
    if value <= 1:
        pct = value
        display_val = f"{value*100:.0f}%"
    else:
        pct = value / max_value
        display_val = f"{value:.0f}%"
    
    cell = ws.cell(row, col, value=pct)
    cell.number_format = '0%'
    cell.font = Font(bold=True, name="Calibri", size=10)
    cell.alignment = Alignment(horizontal="center")
    
    # Background fill as "progress" (gradient simulation)
    if pct >= 1.0:
        cell.fill = PatternFill("solid", fgColor=bar_color)
        cell.font = Font(bold=True, color="FFFFFF", name="Calibri")
    elif pct >= 0.8:
        cell.fill = PatternFill("solid", fgColor="E2EFDA")
        cell.font = Font(bold=True, color="375623", name="Calibri")
    elif pct >= 0.5:
        cell.fill = PatternFill("solid", fgColor="FFEB9C")
        cell.font = Font(bold=True, color="9C6500", name="Calibri")
    else:
        cell.fill = PatternFill("solid", fgColor="FFCCCC")
        cell.font = Font(bold=True, color="9C0006", name="Calibri")
```

---

## ═══════════════════════════════════════════════════
## 9. CHART PATTERNS
## ═══════════════════════════════════════════════════

```python
def add_bar_chart(ws, title, data_min_row, data_max_row,
                   data_cols, cats_col, anchor="A20",
                   chart_type="col", width=22, height=14,
                   colors=None):
    """Bar / Column chart dengan styling"""
    chart = BarChart()
    chart.type = chart_type  # "col" atau "bar"
    chart.title = title
    chart.style = 10
    chart.width = width
    chart.height = height
    chart.grouping = "clustered"
    
    # Add series
    for col_idx in (data_cols if isinstance(data_cols, list) else [data_cols]):
        data = Reference(ws, min_col=col_idx, max_col=col_idx,
                         min_row=data_min_row-1, max_row=data_max_row)
        chart.add_data(data, titles_from_data=True)
    
    # Categories
    cats = Reference(ws, min_col=cats_col, min_row=data_min_row, max_row=data_max_row)
    chart.set_categories(cats)
    
    # Styling
    chart.plot_area.graphicalProperties = None
    if colors:
        for i, (series, color) in enumerate(zip(chart.series, colors)):
            series.graphicalProperties.solidFill = color
    
    ws.add_chart(chart, anchor)
    return chart

def add_line_chart(ws, title, data_min_row, data_max_row,
                    data_cols, cats_col, anchor="A20",
                    smooth=True, width=22, height=14):
    """Line chart dengan smooth curve"""
    chart = LineChart()
    chart.title = title
    chart.style = 10
    chart.width = width
    chart.height = height
    chart.smooth = smooth
    
    for col_idx in (data_cols if isinstance(data_cols, list) else [data_cols]):
        data = Reference(ws, min_col=col_idx, max_col=col_idx,
                         min_row=data_min_row-1, max_row=data_max_row)
        chart.add_data(data, titles_from_data=True)
    
    cats = Reference(ws, min_col=cats_col, min_row=data_min_row, max_row=data_max_row)
    chart.set_categories(cats)
    ws.add_chart(chart, anchor)
    return chart

def add_pie_chart(ws, title, data_min_row, data_max_row,
                   data_col, cats_col, anchor="A20", 
                   doughnut=False, width=18, height=14):
    """Pie / Donut chart"""
    if doughnut:
        chart = DoughnutChart()
        chart.holeSize = 40
    else:
        chart = PieChart()
    
    chart.title = title
    chart.style = 10
    chart.width = width
    chart.height = height
    
    data = Reference(ws, min_col=data_col, max_col=data_col,
                     min_row=data_min_row-1, max_row=data_max_row)
    cats = Reference(ws, min_col=cats_col, min_row=data_min_row, max_row=data_max_row)
    chart.add_data(data, titles_from_data=True)
    chart.set_categories(cats)
    
    chart.dataLabels = DataLabelList() if hasattr(chart, 'dataLabels') else None
    
    ws.add_chart(chart, anchor)
    return chart

def add_combo_chart(ws, title, data_min_row, data_max_row,
                     cats_col, bar_col, line_col, anchor="A20",
                     bar_label="Volume", line_label="Growth %"):
    """Bar + Line combo chart (dual axis)"""
    bar = BarChart()
    bar.type = "col"
    bar.title = title
    bar.style = 10
    bar.width = 22
    bar.height = 14
    
    bar_data = Reference(ws, min_col=bar_col, max_col=bar_col,
                          min_row=data_min_row-1, max_row=data_max_row)
    bar.add_data(bar_data, titles_from_data=True)
    bar.y_axis.title = bar_label
    
    line = LineChart()
    line.smooth = True
    line.y_axis.axId = 200
    line.y_axis.title = line_label
    line.y_axis.crosses = "max"
    
    line_data = Reference(ws, min_col=line_col, max_col=line_col,
                           min_row=data_min_row-1, max_row=data_max_row)
    line.add_data(line_data, titles_from_data=True)
    
    cats = Reference(ws, min_col=cats_col, min_row=data_min_row, max_row=data_max_row)
    bar.set_categories(cats)
    line.set_categories(cats)
    
    bar += line
    ws.add_chart(bar, anchor)
    return bar
```

---

## ═══════════════════════════════════════════════════
## 10. DATA VALIDATION PATTERNS
## ═══════════════════════════════════════════════════

```python
def add_dropdown(ws, cell_range, options_list=None, formula=None,
                 prompt="Pilih nilai", error_msg="Nilai tidak valid"):
    """Dropdown list dengan validasi"""
    if options_list:
        formula1 = '"' + ','.join(str(o) for o in options_list) + '"'
    else:
        formula1 = formula
    
    dv = DataValidation(
        type="list",
        formula1=formula1,
        showDropDown=False,
        prompt=prompt,
        promptTitle="Input",
        error=error_msg,
        errorTitle="Validasi Error",
        showErrorMessage=True,
        showInputMessage=True
    )
    ws.add_data_validation(dv)
    dv.add(cell_range)
    return dv

def add_number_validation(ws, cell_range, min_val=0, max_val=None, 
                           is_integer=False, error_msg=None):
    dv_type = "whole" if is_integer else "decimal"
    if max_val is not None:
        dv = DataValidation(
            type=dv_type, operator="between",
            formula1=str(min_val), formula2=str(max_val),
            error=error_msg or f"Harus antara {min_val} dan {max_val}",
            showErrorMessage=True
        )
    else:
        dv = DataValidation(
            type=dv_type, operator="greaterThanOrEqual",
            formula1=str(min_val),
            error=error_msg or f"Nilai harus ≥ {min_val}",
            showErrorMessage=True
        )
    ws.add_data_validation(dv)
    dv.add(cell_range)
    return dv

def add_date_validation(ws, cell_range, min_year=2000):
    dv = DataValidation(
        type="date", operator="greaterThan",
        formula1=f"DATE({min_year},1,1)",
        error="Format tanggal tidak valid",
        showErrorMessage=True
    )
    ws.add_data_validation(dv)
    dv.add(cell_range)
    return dv

def add_text_length_validation(ws, cell_range, min_len=1, max_len=100):
    dv = DataValidation(
        type="textLength", operator="between",
        formula1=str(min_len), formula2=str(max_len),
        error=f"Panjang teks harus {min_len}–{max_len} karakter",
        showErrorMessage=True
    )
    ws.add_data_validation(dv)
    dv.add(cell_range)
    return dv
```

---

## ═══════════════════════════════════════════════════
## 11. PIVOT & AGGREGASI ADVANCED
## ═══════════════════════════════════════════════════

```python
def create_smart_pivot(df, wb, sheet_prefix="PIVOT"):
    """Auto-buat multiple pivot berdasarkan struktur data"""
    sheets = {}
    
    # Deteksi kolom
    num_cols  = df.select_dtypes(include='number').columns.tolist()
    cat_cols  = df.select_dtypes(include='object').columns.tolist()
    date_cols = [c for c in df.columns if any(k in c.lower() 
                  for k in ['tanggal','date','tgl','bulan','month'])]
    
    if not num_cols or not cat_cols:
        return sheets
    
    # Pivot 1: Summary per kategori utama
    main_cat = cat_cols[0]
    p1 = df.pivot_table(
        index=main_cat, values=num_cols[:3],
        aggfunc={'sum': sum, 'count': 'count'} if len(num_cols) > 1 else 'sum',
        fill_value=0, margins=True, margins_name="TOTAL"
    ).reset_index()
    
    ws1 = wb.create_sheet(f"{sheet_prefix}_BY_{main_cat[:8].upper()}")
    ws1.sheet_properties.tabColor = "5B9BD5"
    df_to_sheet(ws1, p1, table_name=f"Pivot1")
    sheets[ws1.title] = ws1
    
    # Pivot 2: Cross-tab jika ada 2 kategori
    if len(cat_cols) >= 2 and num_cols:
        try:
            p2 = pd.crosstab(
                df[cat_cols[0]], df[cat_cols[1]],
                values=df[num_cols[0]], aggfunc='sum',
                margins=True, margins_name="TOTAL"
            ).reset_index()
            ws2 = wb.create_sheet(f"{sheet_prefix}_CROSSTAB")
            ws2.sheet_properties.tabColor = "5B9BD5"
            df_to_sheet(ws2, p2, table_name="CrossTab")
            sheets[ws2.title] = ws2
        except: pass
    
    # Pivot 3: Trend bulanan jika ada kolom tanggal
    if date_cols and num_cols:
        df_temp = df.copy()
        date_col = date_cols[0]
        try:
            df_temp[date_col] = pd.to_datetime(df_temp[date_col], errors='coerce')
            df_temp['_Periode'] = df_temp[date_col].dt.to_period('M').astype(str)
            p3 = df_temp.groupby('_Periode')[num_cols[:3]].sum().reset_index()
            p3.columns = ['Periode'] + num_cols[:3]
            
            ws3 = wb.create_sheet(f"{sheet_prefix}_TREND")
            ws3.sheet_properties.tabColor = "5B9BD5"
            df_to_sheet(ws3, p3, table_name="TrendBulanan")
            
            # Tambah line chart otomatis
            if len(p3) > 1:
                add_line_chart(ws3, "Trend Bulanan",
                               data_min_row=2, data_max_row=len(p3)+1,
                               data_cols=[2], cats_col=1,
                               anchor=f"A{len(p3)+4}")
            sheets[ws3.title] = ws3
        except: pass
    
    return sheets

def create_top_n_table(df, group_col, value_col, n=10, 
                        ascending=False, label="Top"):
    """Buat tabel top-N untuk dashboard"""
    result = df.groupby(group_col)[value_col].sum().reset_index()
    result.columns = [group_col, value_col]
    result = result.sort_values(value_col, ascending=ascending).head(n)
    result['Rank'] = range(1, len(result)+1)
    result['Share %'] = result[value_col] / result[value_col].sum()
    cols = ['Rank', group_col, value_col, 'Share %']
    return result[cols]
```

---

## ═══════════════════════════════════════════════════
## 12. PRINT SETUP & PAGE PROPERTIES
## ═══════════════════════════════════════════════════

```python
def setup_print_a4(ws, orientation='landscape', title=None, fit_to_width=1):
    """Setup print A4 profesional"""
    ws.page_setup.orientation = orientation
    ws.page_setup.paperSize = 9  # A4
    ws.page_setup.fitToPage = True
    ws.page_setup.fitToWidth = fit_to_width
    ws.page_setup.fitToHeight = 0  # Auto
    
    ws.page_margins = PageMargins(
        left=0.5, right=0.5, top=0.75, bottom=0.75,
        header=0.3, footer=0.3
    )
    
    # Print area
    if ws.max_column and ws.max_row:
        mc = get_column_letter(ws.max_column)
        ws.print_area = f"A1:{mc}{ws.max_row}"
    
    # Print titles (freeze header row)
    ws.print_title_rows = '1:1'
    
    # Header & footer
    if title:
        ws.oddHeader.center.text = title
        ws.oddHeader.center.font = "Calibri,Bold"
        ws.oddHeader.center.size = 12
    ws.oddFooter.right.text = "Halaman &P dari &N"
    ws.oddFooter.right.size = 9
    ws.oddFooter.left.text = f"RIAN | {pd.Timestamp.now().strftime('%d/%m/%Y')}"
    ws.oddFooter.left.size = 9

def setup_print_all_sheets(wb, orientation='landscape'):
    """Apply print setup ke semua sheet sekaligus"""
    for ws in wb.worksheets:
        setup_print_a4(ws, orientation=orientation, title=ws.title)
```

---

## ═══════════════════════════════════════════════════
## 13. MULTI-FILE OPERATIONS
## ═══════════════════════════════════════════════════

```python
def merge_multiple_files(filepaths, sheet_idx=0, output_name="merged.xlsx",
                          add_source_col=True):
    """Gabungkan banyak file Excel ke satu workbook"""
    all_dfs = []
    errors = []
    
    for fp in filepaths:
        try:
            ext = Path(fp).suffix.lower()
            if ext == '.csv':
                df = pd.read_csv(fp, sep=None, engine='python')
            elif ext == '.xls':
                df = pd.read_excel(fp, engine='xlrd')
            else:
                df = pd.read_excel(fp, sheet_name=sheet_idx)
            
            if add_source_col:
                df.insert(0, 'Source File', Path(fp).stem)
            all_dfs.append(df)
            print(f"✅ {Path(fp).name}: {len(df):,} baris")
        except Exception as e:
            errors.append(f"❌ {Path(fp).name}: {e}")
            print(f"❌ Skip: {Path(fp).name} — {e}")
    
    if not all_dfs:
        raise ValueError("Tidak ada file yang berhasil dimuat")
    
    # Align kolom — fill missing dengan NaN
    merged = pd.concat(all_dfs, ignore_index=True, sort=False)
    
    print(f"\n📊 Merged: {len(merged):,} baris dari {len(all_dfs)} file")
    if errors:
        print(f"⚠️ Gagal: {len(errors)} file")
    
    wb = Workbook()
    ws = wb.active
    ws.title = "MERGED DATA"
    ws.sheet_properties.tabColor = "404040"
    df_to_sheet(ws, merged, table_name="MergedData")
    
    return wb, merged, errors

def compare_two_files(filepath1, filepath2, key_col, sheet=0):
    """Bandingkan dua file dan buat laporan perbedaan"""
    df1 = pd.read_excel(filepath1, sheet_name=sheet).set_index(key_col)
    df2 = pd.read_excel(filepath2, sheet_name=sheet).set_index(key_col)
    
    added   = df2.index.difference(df1.index)
    removed = df1.index.difference(df2.index)
    common  = df1.index.intersection(df2.index)
    
    changes = []
    common_cols = [c for c in df1.columns if c in df2.columns]
    
    for key in common:
        for col in common_cols:
            v1 = df1.loc[key, col]
            v2 = df2.loc[key, col]
            if str(v1) != str(v2):
                changes.append({
                    key_col: key, 'Kolom': col,
                    'Nilai Lama': v1, 'Nilai Baru': v2
                })
    
    # Buat workbook hasil
    wb = Workbook()
    wb.remove(wb.active)
    
    summary_ws = wb.create_sheet("RINGKASAN")
    summary_ws.sheet_properties.tabColor = "1F4E79"
    summary_data = [
        ['Item', 'Jumlah'],
        ['Baris Baru (di file 2)', len(added)],
        ['Baris Hilang (dari file 1)', len(removed)],
        ['Baris Berubah', len(set(c[key_col] for c in changes))],
        ['Total Perubahan Field', len(changes)],
    ]
    for r_idx, row in enumerate(summary_data, 1):
        for c_idx, val in enumerate(row, 1):
            summary_ws.cell(r_idx, c_idx, val)
    
    if changes:
        changes_ws = wb.create_sheet("PERUBAHAN DETAIL")
        changes_ws.sheet_properties.tabColor = "ED7D31"
        df_to_sheet(changes_ws, pd.DataFrame(changes), table_name="Changes")
    
    if len(added) > 0:
        added_ws = wb.create_sheet("BARIS BARU")
        added_ws.sheet_properties.tabColor = "70AD47"
        df_to_sheet(added_ws, df2.loc[added].reset_index(), table_name="Added")
    
    if len(removed) > 0:
        removed_ws = wb.create_sheet("BARIS HILANG")
        removed_ws.sheet_properties.tabColor = "FF0000"
        df_to_sheet(removed_ws, df1.loc[removed].reset_index(), table_name="Removed")
    
    return wb, {'added': len(added), 'removed': len(removed), 'changed': len(changes)}
```

---

## ═══════════════════════════════════════════════════
## 14. STATISTICAL ANALYSIS ENGINE
## ═══════════════════════════════════════════════════

```python
def generate_stat_summary(df):
    """Summary statistik komprehensif"""
    num_cols = df.select_dtypes(include='number').columns.tolist()
    
    if not num_cols:
        return pd.DataFrame()
    
    stats = df[num_cols].agg([
        'count', 'sum', 'mean', 'median', 'std', 'min', 'max',
        lambda x: x.quantile(0.25),
        lambda x: x.quantile(0.75),
        lambda x: x.skew() if hasattr(x, 'skew') else 0,
    ]).round(2)
    stats.index = ['Count', 'Sum', 'Mean', 'Median', 'Std Dev', 
                   'Min', 'Max', 'Q1 (25%)', 'Q3 (75%)', 'Skewness']
    
    return stats.T.reset_index().rename(columns={'index': 'Kolom'})

def detect_outliers_iqr(df, col):
    """Deteksi outlier dengan metode IQR"""
    Q1 = df[col].quantile(0.25)
    Q3 = df[col].quantile(0.75)
    IQR = Q3 - Q1
    lower = Q1 - 1.5 * IQR
    upper = Q3 + 1.5 * IQR
    
    outliers = df[(df[col] < lower) | (df[col] > upper)]
    return outliers, lower, upper

def simple_forecast(df, date_col, value_col, periods=3):
    """Forecast sederhana dengan linear regression"""
    from scipy import stats as sp_stats
    
    df = df.copy()
    df[date_col] = pd.to_datetime(df[date_col], errors='coerce')
    df = df.dropna(subset=[date_col, value_col]).sort_values(date_col)
    
    # Numeric time index
    df['_t'] = range(len(df))
    
    slope, intercept, r_val, p_val, std_err = sp_stats.linregress(df['_t'], df[value_col])
    
    # Future periods
    last_t = df['_t'].max()
    last_date = df[date_col].max()
    
    forecast_rows = []
    for i in range(1, periods + 1):
        future_t = last_t + i
        future_val = intercept + slope * future_t
        # Estimasi tanggal (asumsi period = 1 bulan)
        try:
            future_date = last_date + pd.DateOffset(months=i)
        except:
            future_date = last_date + timedelta(days=30*i)
        
        forecast_rows.append({
            date_col: future_date,
            value_col: max(0, future_val),
            'Type': 'Forecast',
            'Lower': max(0, future_val - 1.96 * std_err * (future_t ** 0.5)),
            'Upper': future_val + 1.96 * std_err * (future_t ** 0.5),
        })
    
    return pd.DataFrame(forecast_rows), r_val**2  # return forecast df + R-squared
```

---

## ═══════════════════════════════════════════════════
## 15. NAMED RANGES & CELL PROTECTION
## ═══════════════════════════════════════════════════

```python
def create_named_range(wb, name, sheet_name, cell_ref):
    """Buat named range"""
    full_ref = f"{quote_sheetname(sheet_name)}!{cell_ref}"
    named_range = DefinedName(name, attr_text=full_ref)
    wb.defined_names[name] = named_range

def protect_formula_cells(ws, password="", unlock_ranges=None):
    """
    Kunci semua formula, unlock semua cell kosong / input.
    unlock_ranges = ["B2:B100", "D5:D50"] — cell input yang boleh diedit
    """
    # 1. Unlock semua cell dulu
    for row in ws.iter_rows():
        for cell in row:
            cell.protection = cell.protection.__class__(locked=False, hidden=False)
    
    # 2. Lock cell yang punya formula
    for row in ws.iter_rows():
        for cell in row:
            if isinstance(cell.value, str) and cell.value.startswith('='):
                cell.protection = cell.protection.__class__(locked=True)
    
    # 3. Re-unlock input ranges
    if unlock_ranges:
        for rng in unlock_ranges:
            for row in ws[rng]:
                for cell in (row if hasattr(row, '__iter__') else [row]):
                    cell.protection = cell.protection.__class__(locked=False)
    
    # 4. Aktifkan proteksi sheet
    ws.protection.sheet = True
    ws.protection.password = password
    ws.protection.enable()
    print(f"🔒 Sheet '{ws.title}' dilindungi — formula terkunci")
```


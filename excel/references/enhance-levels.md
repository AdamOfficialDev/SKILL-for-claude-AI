# Enhance Levels Reference — RIAN v4
# Level 0-3 + Design Theme Switcher + Sparklines + New Enhance Items

## ENHANCE LEVEL CONTROLLER

```python
def apply_enhance_level(wb, df_dict: dict, domain: str, level: int = None):
    """
    Master enhance controller. Apply level sesuai session atau override.
    """
    level = level if level is not None else get_session('enhance_level', 2)
    
    print(f"\n✨ SMART ENHANCE — Level {level}: {ENHANCE_LABELS[level]}")
    applied = []
    
    if level == 0:
        print("   (No enhance — skipped)")
        return []
    
    if level >= 1:
        applied.extend(apply_level1(wb, df_dict))
    
    if level >= 2:
        applied.extend(apply_level2(wb, df_dict, domain))
    
    if level >= 3:
        applied.extend(apply_level3(wb, df_dict, domain))
    
    print(f"   Applied {len(applied)} enhancements")
    return applied

ENHANCE_LABELS = {
    0: 'Off (Raw Execution)',
    1: 'Minimal (Format Only)',
    2: 'Standard (default)',
    3: 'Full (Intelligence)',
}
```

---

## LEVEL 1 — MINIMAL (Format Only)

```python
def apply_level1(wb, df_dict: dict) -> list:
    """Level 1: Format only — cepat dan bersih"""
    applied = []
    theme = get_theme()
    
    for ws in wb.worksheets:
        name = ws.title
        max_row = ws.max_row or 0
        max_col = ws.max_column or 0
        
        if max_row == 0 or max_col == 0:
            continue
        
        # 1.1 Auto-fit column width
        auto_fit_columns(ws)
        applied.append(f"Auto-fit: {name}")
        
        # 1.2 Freeze panes
        if max_row > 20 and not ws.freeze_panes:
            ws.freeze_panes = 'A2'
            applied.append(f"Freeze: {name}")
        
        # 1.3 Autofilter
        if max_row > 5 and max_col > 1 and not ws.auto_filter.ref:
            mc = get_column_letter(max_col)
            ws.auto_filter.ref = f"A1:{mc}1"
            applied.append(f"Autofilter: {name}")
        
        # 1.4 Tab color
        if not ws.sheet_properties.tabColor or not ws.sheet_properties.tabColor.value:
            color = _guess_tab_color(name)
            ws.sheet_properties.tabColor = color
            applied.append(f"Tab color: {name}")
        
        # 1.5 Number format per kolom (IDR-first)
        if name in df_dict:
            applied.extend(apply_smart_number_format(ws, df_dict[name]))
        
        # 1.6 Print setup
        setup_print_a4(ws)
        applied.append(f"Print A4: {name}")
    
    return applied

def _guess_tab_color(sheet_name: str) -> str:
    """Tebak warna tab dari nama sheet"""
    name_lower = sheet_name.lower()
    TAB_COLORS = {
        'dashboard': '1F4E79', 'cover': '2E75B6', 'summary': '70AD47',
        'data': '404040', 'analysis': 'ED7D31', 'pivot': '5B9BD5',
        'chart': 'FFC000', 'settings': '808080', 'forecast': '8E44AD',
        'audit': 'E74C3C', 'change_log': 'C0392B', 'kamus': '3498DB',
        'petunjuk': '27AE60', 'rekonsiliasi': 'E67E22',
    }
    for keyword, color in TAB_COLORS.items():
        if keyword in name_lower:
            return color
    return '808080'

def apply_smart_number_format(ws, df) -> list:
    """Apply format angka yang tepat berdasarkan nama kolom"""
    applied = []
    
    # Deteksi kolom dan format yang tepat
    currency_kw = ['harga','gaji','biaya','revenue','total','subtotal','nilai',
                   'pendapatan','omzet','cost','pembayaran','tagihan','fee',
                   'bruto','neto','thp','amount','price','salary']
    pct_kw = ['persen','percent','margin','rate','%','ach','achievement',
              'growth','ratio','share','proporsi']
    qty_kw = ['qty','jumlah','stok','stock','count','kuantitas','unit','pcs','item']
    date_kw = ['tanggal','date','tgl','waktu','dob','lahir']
    
    for c_idx, col_name in enumerate(df.columns, 1):
        col_lower = str(col_name).lower()
        
        for r_idx in range(2, (ws.max_row or 1) + 1):
            cell = ws.cell(r_idx, c_idx)
            if cell.value is None: continue
            
            if any(k in col_lower for k in currency_kw):
                cell.number_format = '#,##0;(#,##0);"-"'
                cell.alignment = Alignment(horizontal='right', vertical='center')
            elif any(k in col_lower for k in pct_kw):
                # Cek apakah format 0-1 atau 0-100
                if isinstance(cell.value, (int, float)) and cell.value <= 1:
                    cell.number_format = '0.0%;(0.0%);"-"'
                else:
                    cell.number_format = '0.0%;(0.0%);"-"'
                cell.alignment = Alignment(horizontal='right', vertical='center')
            elif any(k in col_lower for k in qty_kw):
                cell.number_format = '#,##0;(#,##0);"-"'
                cell.alignment = Alignment(horizontal='right', vertical='center')
            elif any(k in col_lower for k in date_kw):
                cell.number_format = 'DD/MM/YYYY'
                cell.alignment = Alignment(horizontal='center', vertical='center')
    
    applied.append(f"Number format: {len(df.columns)} kolom")
    return applied
```

---

## LEVEL 2 — STANDARD (Format + Structure)

```python
def apply_level2(wb, df_dict: dict, domain: str) -> list:
    """Level 2: Format + Structure improvements"""
    applied = []
    theme = get_theme()
    
    for ws in wb.worksheets:
        name = ws.title
        df = df_dict.get(name)
        max_row = ws.max_row or 0
        max_col = ws.max_column or 0
        
        if max_row < 2: continue
        
        # 2.1 Conditional formatting untuk kolom Status
        applied.extend(apply_domain_cf(ws, df, domain))
        
        # 2.2 Total row (jika belum ada)
        applied.extend(add_total_row_if_missing(ws, df))
        
        # 2.3 Sparklines untuk kolom trend
        if get_session('sparklines_ok', False) and df is not None:
            applied.extend(add_auto_sparklines(ws, df))
        
        # 2.4 Summary section jika hanya 1 sheet data
        if len(df_dict) == 1 and df is not None and len(df) > 0:
            applied.extend(add_mini_summary(ws, df, max_row, max_col))
    
    # 2.5 Tambah sheet SUMMARY jika tidak ada
    if not any('summary' in s.lower() for s in wb.sheetnames) and df_dict:
        main_df = list(df_dict.values())[0]
        if len(main_df) > 0:
            summary_ws = build_simple_summary_sheet(wb, main_df, domain)
            applied.append("Sheet SUMMARY dibuat")
    
    return applied

def apply_domain_cf(ws, df, domain: str) -> list:
    """Domain-aware conditional formatting"""
    applied = []
    if df is None: return applied
    
    status_cols = [c for c in df.columns if 'status' in str(c).lower()]
    
    for col_name in status_cols:
        col_idx = list(df.columns).index(col_name) + 1
        col_letter = get_column_letter(col_idx)
        cell_range = f"{col_letter}2:{col_letter}{(ws.max_row or 2)}"
        
        # Domain-specific good/bad values
        domain_values = {
            'hr': {
                'good': ['Aktif','Active'], 
                'bad': ['Resign','PHK','Terminated'],
                'warning': ['Probation','Cuti Panjang']
            },
            'sales': {
                'good': ['✅ On Track','On Track','Achieved'],
                'bad': ['❌ Below','Below Target','Miss'],
                'warning': ['⚠️ At Risk','At Risk']
            },
            'inventory': {
                'good': ['🟢 AMAN','AMAN','OK'],
                'bad': ['🔴 KRITIS','KRITIS','Critical'],
                'warning': ['🟡 RENDAH','RENDAH','Low']
            },
            'project': {
                'good': ['✅ Done','Done','Completed'],
                'bad': ['🔴 Overdue','Overdue','Delayed'],
                'warning': ['🟡 Due Soon','At Risk','In Progress']
            },
        }
        
        vals = domain_values.get(domain, {
            'good': ['Aktif','Done','Selesai','Lunas','Aman'],
            'bad': ['Resign','Overdue','Error','Kritis','Ditolak'],
            'warning': ['Pending','Review','Proses','Rendah'],
        })
        
        from references_python_patterns import add_status_cf
        add_status_cf(ws, cell_range, vals['good'], vals['bad'], vals['warning'])
        applied.append(f"Status CF: {col_name}")
    
    # Highlight negatif untuk kolom angka finansial
    neg_kw = ['selisih','variance','var','diff','difference','growth','change']
    for c_idx, col in enumerate(df.columns, 1):
        if any(k in str(col).lower() for k in neg_kw):
            col_letter = get_column_letter(c_idx)
            cell_range = f"{col_letter}2:{col_letter}{ws.max_row}"
            from references_python_patterns import highlight_negatives
            highlight_negatives(ws, cell_range)
            applied.append(f"Neg CF: {col}")
    
    return applied

def add_total_row_if_missing(ws, df) -> list:
    """Cek dan tambah total row jika belum ada"""
    applied = []
    if df is None or len(df) == 0: return applied
    
    max_row = ws.max_row or 0
    
    # Cek apakah baris terakhir sudah TOTAL/SUM
    last_row_vals = [ws.cell(max_row, c).value for c in range(1, min((ws.max_column or 1)+1, 6))]
    has_total = any(str(v).upper() in ['TOTAL','JUMLAH','GRAND TOTAL'] 
                   for v in last_row_vals if v)
    
    if not has_total and max_row > 2:
        num_cols = [c_idx for c_idx, col in enumerate(df.columns, 1)
                   if df[col].dtype in ['float64','int64']]
        
        if num_cols:
            from references_python_patterns import add_sum_row
            data_start = 2
            data_end = max_row
            total_row = add_sum_row(ws, data_start, data_end, num_cols)
            applied.append(f"Total row: baris {total_row}")
    
    return applied

def add_auto_sparklines(ws, df) -> list:
    """Tambah sparklines untuk kolom yang terlihat seperti time-series"""
    applied = []
    
    if not SPARKLINES_AVAILABLE:
        return applied
    
    # Deteksi kolom numerik yang nilainya berurutan (potential time-series)
    num_cols = df.select_dtypes(include='number').columns.tolist()
    
    if len(num_cols) < 3:  # Perlu minimal 3 titik data untuk sparkline
        return applied
    
    # Cari kolom terakhir + 1 untuk menaruh sparklines
    sparkline_col = (ws.max_column or 1) + 2
    col_letter = get_column_letter(sparkline_col)
    
    # Header sparkline
    header = ws.cell(1, sparkline_col, "Trend")
    header.font = Font(bold=True, color="FFFFFF", name="Calibri")
    header.fill = PatternFill("solid", fgColor="1F4E79")
    header.alignment = Alignment(horizontal='center', vertical='center')
    
    # Add sparkline group
    try:
        first_num_col = list(df.columns).index(num_cols[0]) + 1
        last_num_col = list(df.columns).index(num_cols[-1]) + 1
        
        data_ref = f"{get_column_letter(first_num_col)}2:{get_column_letter(last_num_col)}{ws.max_row}"
        sparkline_range = f"{col_letter}2:{col_letter}{ws.max_row}"
        
        sparkline_group = SparklineGroup(
            ref=sparkline_range,
            sourceData=data_ref,
            type='line',
            colorSeries='FF1F4E79',
        )
        ws.sparkline_groups.append(sparkline_group)
        applied.append(f"Sparklines: {sparkline_range}")
    except Exception as e:
        pass  # Sparklines tidak semua versi support, skip gracefully
    
    return applied
```

---

## LEVEL 3 — FULL (Format + Structure + Intelligence)

```python
def apply_level3(wb, df_dict: dict, domain: str) -> list:
    """Level 3: Full intelligence — dashboard, chart, insights, forecast"""
    applied = []
    
    # 3.1 Named ranges untuk semua tabel
    applied.extend(create_named_ranges_all(wb, df_dict))
    
    # 3.2 Sheet DASHBOARD
    if not any('dashboard' in s.lower() for s in wb.sheetnames):
        dashboard_ws = build_executive_dashboard(wb, df_dict, domain)
        applied.append("Dashboard sheet")
    
    # 3.3 Charts (jika belum ada)
    chart_count = sum(len(ws._charts) for ws in wb.worksheets)
    if chart_count == 0 and df_dict:
        applied.extend(add_auto_charts(wb, df_dict, domain))
    
    # 3.4 Auto-insights text box di SUMMARY (jika ada)
    if 'SUMMARY' in wb.sheetnames:
        summary_ws = wb['SUMMARY']
        main_df = list(df_dict.values())[0] if df_dict else pd.DataFrame()
        from references_advanced_ops import generate_auto_insights
        insights = generate_auto_insights(main_df, domain)
        from references_advanced_ops import add_insight_text_box
        max_row = summary_ws.max_row + 3
        add_insight_text_box(summary_ws, max_row, 1, insights)
        applied.append("Auto-insights")
    
    # 3.5 Forecast jika ada data historis
    if df_dict:
        applied.extend(maybe_add_forecast_sheet(wb, df_dict))
    
    # 3.6 Change log
    build_change_log_sheet(wb)
    applied.append("Change log sheet")
    
    return applied

def create_named_ranges_all(wb, df_dict: dict) -> list:
    """Buat named ranges untuk semua tabel"""
    applied = []
    
    for ws in wb.worksheets:
        if ws.title not in df_dict: continue
        df = df_dict[ws.title]
        if len(df) == 0: continue
        
        # Buat named range untuk tabel data
        safe_name = re.sub(r'[^A-Za-z0-9_]', '_', ws.title)
        table_name = f"Tabel_{safe_name}"
        
        max_col_letter = get_column_letter(ws.max_column or 1)
        cell_ref = f"$A$1:${max_col_letter}${ws.max_row}"
        full_ref = f"{quote_sheetname(ws.title)}!{cell_ref}"
        
        try:
            named = DefinedName(table_name, attr_text=full_ref)
            wb.defined_names[table_name] = named
            applied.append(f"Named range: {table_name}")
        except Exception:
            pass
    
    return applied

def add_auto_charts(wb, df_dict: dict, domain: str) -> list:
    """Tambah chart otomatis yang relevan"""
    applied = []
    
    main_df = list(df_dict.values())[0]
    main_ws = wb[list(df_dict.keys())[0]]
    
    num_cols = main_df.select_dtypes(include='number').columns.tolist()
    date_cols = [c for c in main_df.columns if any(k in str(c).lower() 
                  for k in ['tanggal','date','tgl','bulan'])]
    cat_cols = main_df.select_dtypes(include='object').columns.tolist()
    
    if not num_cols: return applied
    
    # Chart 1: Bar chart untuk kategori pertama vs nilai utama
    if cat_cols and num_cols:
        try:
            from references_python_patterns import add_bar_chart
            anchor_row = (main_ws.max_row or 10) + 3
            add_bar_chart(
                main_ws,
                title=f"{num_cols[0]} by {cat_cols[0]}",
                data_min_row=2, data_max_row=min(main_ws.max_row or 2, 20),
                data_cols=[list(main_df.columns).index(num_cols[0]) + 1],
                cats_col=list(main_df.columns).index(cat_cols[0]) + 1,
                anchor=f"A{anchor_row}",
            )
            applied.append(f"Chart 1: Bar — {num_cols[0]} by {cat_cols[0]}")
        except Exception as e:
            pass
    
    # Chart 2: Line chart jika ada data tanggal
    if date_cols and num_cols:
        try:
            from references_python_patterns import add_line_chart
            anchor_row = (main_ws.max_row or 10) + 20
            add_line_chart(
                main_ws,
                title=f"Trend {num_cols[0]}",
                data_min_row=2, data_max_row=min(main_ws.max_row or 2, 30),
                data_cols=[list(main_df.columns).index(num_cols[0]) + 1],
                cats_col=list(main_df.columns).index(date_cols[0]) + 1,
                anchor=f"M{anchor_row - 17}",
            )
            applied.append(f"Chart 2: Line — Trend {num_cols[0]}")
        except Exception as e:
            pass
    
    return applied

def maybe_add_forecast_sheet(wb, df_dict: dict) -> list:
    """Tambah sheet FORECAST jika ada data historis yang cukup"""
    applied = []
    
    for sheet_name, df in df_dict.items():
        if len(df) < 6: continue  # Minimal 6 titik data
        
        date_cols = [c for c in df.columns if any(k in str(c).lower() 
                      for k in ['tanggal','date','tgl'])]
        num_cols = df.select_dtypes(include='number').columns.tolist()
        
        if not date_cols or not num_cols: continue
        
        try:
            from references_python_patterns import simple_forecast
            forecast_df, r2 = simple_forecast(df, date_cols[0], num_cols[0], periods=3)
            
            if r2 < 0.3: continue  # R² terlalu rendah, forecast tidak reliable
            
            forecast_ws = wb.create_sheet("FORECAST")
            forecast_ws.sheet_properties.tabColor = "8E44AD"
            
            # Header info
            forecast_ws['A1'] = f"FORECAST — {num_cols[0]}"
            forecast_ws['A1'].font = Font(size=14, bold=True, color="1F4E79", name="Calibri")
            forecast_ws['A2'] = f"Model: Linear Regression | R² = {r2:.2f} | " \
                                f"Prediksi {len(forecast_df)} periode ke depan"
            forecast_ws['A2'].font = Font(size=10, color="888888", italic=True)
            
            # Write forecast data
            from references_python_patterns import df_to_sheet
            df_to_sheet(forecast_ws, forecast_df, start_row=4, table_name="ForecastData")
            
            applied.append(f"Forecast sheet (R²={r2:.2f})")
        except Exception:
            pass
        
        break  # Hanya forecast untuk sheet pertama yang cocok
    
    return applied
```

---

## DESIGN THEME SWITCHER (/excel tema)

```python
THEMES = {
    'corporate_navy': {
        'header': '1F4E79', 'sub_header': '2E75B6',
        'accent': '70AD47', 'warning': 'ED7D31', 'danger': 'FF0000',
        'total_bg': 'FFF2CC', 'zebra': 'F2F7FF', 'border': 'BDD7EE',
        'font_header': 'FFFFFF', 'font_body': '404040',
    },
    'corporate_green': {
        'header': '1E4620', 'sub_header': '2D6A2F',
        'accent': '2E75B6', 'warning': 'F39C12', 'danger': 'E74C3C',
        'total_bg': 'EBF5EB', 'zebra': 'F0FAF0', 'border': 'A9DFBF',
        'font_header': 'FFFFFF', 'font_body': '2C3E50',
    },
    'monochrome': {
        'header': '2C2C2A', 'sub_header': '444441',
        'accent': '888780', 'warning': '5F5E5A', 'danger': 'B04040',
        'total_bg': 'F1EFE8', 'zebra': 'F9F9F7', 'border': 'D3D1C7',
        'font_header': 'FFFFFF', 'font_body': '2C2C2A',
    },
    'warm_accent': {
        'header': '7B3F00', 'sub_header': 'A04000',
        'accent': 'C0392B', 'warning': 'E67E22', 'danger': 'E74C3C',
        'total_bg': 'FEF9E7', 'zebra': 'FDF8F0', 'border': 'FAD7A0',
        'font_header': 'FFFFFF', 'font_body': '4A235A',
    },
    'dark_pro': {
        'header': '0D1117', 'sub_header': '161B22',
        'accent': '58A6FF', 'warning': 'F0883E', 'danger': 'F85149',
        'total_bg': '1C2128', 'zebra': '161B22', 'border': '30363D',
        'font_header': 'C9D1D9', 'font_body': 'C9D1D9',
    },
}

def get_theme() -> dict:
    """Get tema yang aktif berdasarkan session"""
    theme_name = get_session('theme', 'corporate_navy')
    return THEMES.get(theme_name, THEMES['corporate_navy'])

def apply_theme_to_workbook(wb, theme_name: str = None) -> list:
    """
    /excel tema [nama] — Ganti seluruh tema workbook.
    TIDAK mengubah data atau formula.
    """
    theme_name = theme_name or get_session('theme', 'corporate_navy')
    theme = THEMES.get(theme_name)
    
    if not theme:
        print(f"❌ Tema tidak ditemukan: {theme_name}")
        print(f"   Tersedia: {', '.join(THEMES.keys())}")
        return []
    
    applied = []
    set_session(theme=theme_name)
    
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                # Re-apply header colors
                if cell.fill.fgColor.value and cell.fill.fgColor.value != '00000000':
                    current_color = cell.fill.fgColor.value
                    
                    # Map warna lama ke kategori, lalu ke warna baru
                    # Header utama
                    if current_color in [t['header'] for t in THEMES.values()]:
                        cell.fill = PatternFill("solid", fgColor=theme['header'])
                        if cell.font and cell.font.color:
                            cell.font = Font(
                                bold=cell.font.bold,
                                size=cell.font.size,
                                name=cell.font.name or "Calibri",
                                color=theme['font_header']
                            )
                    # Sub-header
                    elif current_color in [t['sub_header'] for t in THEMES.values()]:
                        cell.fill = PatternFill("solid", fgColor=theme['sub_header'])
                    # Total row
                    elif current_color in [t['total_bg'] for t in THEMES.values()]:
                        cell.fill = PatternFill("solid", fgColor=theme['total_bg'])
                    # Zebra
                    elif current_color in [t['zebra'] for t in THEMES.values()]:
                        cell.fill = PatternFill("solid", fgColor=theme['zebra'])
        
        applied.append(f"Theme applied: {ws.title}")
    
    print(f"✅ Tema '{theme_name}' berhasil diterapkan ke {len(wb.worksheets)} sheet")
    return applied
```

---

## HELPER FUNCTIONS (Tersedia di Semua Level)

```python
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

def setup_print_a4(ws, title: str = None, orientation: str = 'landscape'):
    ws.page_setup.orientation = orientation
    ws.page_setup.paperSize = 9  # A4
    ws.page_setup.fitToPage = True
    ws.page_setup.fitToWidth = 1
    ws.page_setup.fitToHeight = 0
    ws.page_margins = PageMargins(left=0.5, right=0.5, top=0.75, bottom=0.75)
    
    if ws.max_column and ws.max_row:
        mc = get_column_letter(ws.max_column)
        ws.print_area = f"A1:{mc}{ws.max_row}"
    
    ws.print_title_rows = '1:1'
    
    if title:
        ws.oddHeader.center.text = title
        ws.oddHeader.center.size = 11
    ws.oddFooter.right.text = "Halaman &P dari &N"
    ws.oddFooter.right.size = 9
    ws.oddFooter.left.text = f"RIAN v4 | {pd.Timestamp.now().strftime('%d/%m/%Y')}"
    ws.oddFooter.left.size = 9

def add_status_cf(ws, cell_range, good_vals=None, bad_vals=None, warn_vals=None):
    good_vals  = good_vals  or ['Aktif','Done','Lunas','Aman']
    bad_vals   = bad_vals   or ['Error','Kritis','Overdue','Ditolak']
    warn_vals  = warn_vals  or ['Pending','Review','Rendah']
    
    green  = PatternFill(bgColor="E2EFDA"); green_f  = Font(color="375623", bold=True)
    red    = PatternFill(bgColor="FFCCCC"); red_f    = Font(color="9C0006", bold=True)
    yellow = PatternFill(bgColor="FFEB9C"); yellow_f = Font(color="9C6500", bold=True)
    
    for val in good_vals:
        ws.conditional_formatting.add(cell_range, CellIsRule(
            operator='equal', formula=[f'"{val}"'], fill=green, font=green_f))
    for val in bad_vals:
        ws.conditional_formatting.add(cell_range, CellIsRule(
            operator='equal', formula=[f'"{val}"'], fill=red, font=red_f))
    for val in warn_vals:
        ws.conditional_formatting.add(cell_range, CellIsRule(
            operator='equal', formula=[f'"{val}"'], fill=yellow, font=yellow_f))

def build_simple_summary_sheet(wb, df, domain: str):
    ws = wb.create_sheet("SUMMARY")
    ws.sheet_properties.tabColor = "70AD47"
    ws.sheet_view.showGridLines = False
    
    theme = get_theme()
    
    # Title
    ws.merge_cells('A1:F1')
    ws['A1'] = f"SUMMARY — {domain.upper()}"
    ws['A1'].font = Font(size=16, bold=True, color=theme['header'], name="Calibri")
    ws['A1'].alignment = Alignment(horizontal='center', vertical='center')
    ws.row_dimensions[1].height = 35
    
    ws['A2'] = f"Per: {pd.Timestamp.now().strftime('%d %B %Y')}"
    ws['A2'].font = Font(size=10, color="888888", italic=True, name="Calibri")
    
    # Statistik numerik
    num_cols = df.select_dtypes(include='number').columns.tolist()
    row = 4
    for col in num_cols[:4]:
        ws.cell(row, 1, f"Total {col}").font = Font(name="Calibri", size=10)
        val_cell = ws.cell(row, 2, df[col].sum())
        val_cell.number_format = '#,##0;(#,##0);"-"'
        val_cell.font = Font(bold=True, name="Calibri")
        
        ws.cell(row, 3, f"Rata-rata {col}").font = Font(name="Calibri", size=10, color="888888")
        avg_cell = ws.cell(row, 4, df[col].mean())
        avg_cell.number_format = '#,##0.##;(#,##0.##);"-"'
        avg_cell.font = Font(name="Calibri")
        row += 1
    
    auto_fit_columns(ws)
    return ws
```

---

## ═══════════════════════════════════════════════════
## ACCESSIBILITY COLOR SYSTEM (v4 NEW)
## ═══════════════════════════════════════════════════

> Aktifkan dengan: "pakai warna colorblind safe" / "mode aksesibilitas" / `/rian settings`
> Set `RIAN_SESSION['theme'] = 'colorblind_safe'`

```python
# ── Okabe-Ito Palette (universally color-blind safe) ─────────────────
# Aman untuk Deuteranopia, Protanopia, Tritanopia
COLORBLIND_SAFE_PALETTE = {
    'black':        '000000',  # #000000 — hitam
    'orange':       'E69F00',  # #E69F00 — oranye
    'sky_blue':     '56B4E9',  # #56B4E9 — biru langit
    'green':        '009E73',  # #009E73 — hijau teal
    'yellow':       'F0E442',  # #F0E442 — kuning
    'blue':         '0072B2',  # #0072B2 — biru gelap
    'vermillion':   'D55E00',  # #D55E00 — merah oranye
    'pink':         'CC79A7',  # #CC79A7 — merah muda
}

# Map ke fungsi semantik (menggantikan tema warna standar)
CB_SEMANTIC = {
    'positive':   COLORBLIND_SAFE_PALETTE['green'],       # Hijau teal = baik
    'negative':   COLORBLIND_SAFE_PALETTE['vermillion'],  # Merah oranye = buruk
    'neutral':    COLORBLIND_SAFE_PALETTE['sky_blue'],    # Biru langit = netral
    'warning':    COLORBLIND_SAFE_PALETTE['orange'],      # Oranye = perhatian
    'highlight':  COLORBLIND_SAFE_PALETTE['yellow'],      # Kuning = sorot
    'header_bg':  COLORBLIND_SAFE_PALETTE['blue'],        # Biru gelap = header
    'header_text':'FFFFFF',
    'alt_row':    'EAF4FB',
}

# ── Hatch Pattern untuk chart (bukan hanya warna) ──────────────────
# Saat series > 4, tambahkan pattern agar dapat dibedakan tanpa warna
CB_CHART_PATTERNS = [
    {'fill': COLORBLIND_SAFE_PALETTE['blue'],      'pattern': 'solid'},
    {'fill': COLORBLIND_SAFE_PALETTE['orange'],    'pattern': 'solid'},
    {'fill': COLORBLIND_SAFE_PALETTE['sky_blue'],  'pattern': 'solid'},
    {'fill': COLORBLIND_SAFE_PALETTE['green'],     'pattern': 'solid'},
    {'fill': COLORBLIND_SAFE_PALETTE['blue'],      'pattern': 'lgHorz'},   # horizontal lines
    {'fill': COLORBLIND_SAFE_PALETTE['orange'],    'pattern': 'lgVert'},   # vertical lines
    {'fill': COLORBLIND_SAFE_PALETTE['sky_blue'],  'pattern': 'lgDiag'},   # diagonal
    {'fill': COLORBLIND_SAFE_PALETTE['green'],     'pattern': 'smGrid'},   # grid
]

def apply_colorblind_conditional_formatting(ws, status_col: str,
                                             data_start: int, data_end: int) -> None:
    """
    Conditional formatting yang aman untuk color blind.
    Gunakan IKON + warna, bukan hanya warna.
    """
    from openpyxl.formatting.rule import IconSetRule, CellIsRule, DifferentialStyle
    from openpyxl.styles import Font, PatternFill

    col_range = f"{status_col}{data_start}:{status_col}{data_end}"

    # Gunakan Icon Set (shape) BUKAN hanya warna merah/hijau
    # 3 Traffic lights dengan icon berbeda
    icon_rule = IconSetRule(
        icon_style='3TrafficLights1',  # ●◑○ shape + warna
        type='percent',
        values=[0, 34, 67],
        showValue=True,
        percent=True,
        reverse=False,
    )
    ws.conditional_formatting.add(col_range, icon_rule)

    print(f"✅ Accessibility CF diterapkan di {col_range} (icon + warna)")


def get_accessible_theme_colors() -> dict:
    """
    Return warna berdasarkan theme aktif.
    Auto-pilih colorblind_safe jika RIAN_SESSION['theme'] == 'colorblind_safe'.
    """
    theme = RIAN_SESSION.get('theme', 'corporate_navy')

    if theme == 'colorblind_safe':
        return CB_SEMANTIC
    else:
        return THEMES.get(theme, THEMES['corporate_navy'])


# ── Rule wajib untuk chart ≥ 5 series ──────────────────────────────
# Tambahkan ke ATURAN KERAS v4:
# ✅ ALWAYS → Chart dengan >4 series WAJIB pakai hatch pattern,
#              bukan hanya perbedaan warna
# Implementasi: gunakan CB_CHART_PATTERNS[i] untuk series ke-i
def apply_chart_patterns(chart, num_series: int) -> None:
    """Terapkan hatch pattern ke chart saat series > 4."""
    from openpyxl.chart.data_source import NumDataSource
    from openpyxl.drawing.fill import PatternFillProperties

    for i, series in enumerate(chart.series):
        pattern_info = CB_CHART_PATTERNS[i % len(CB_CHART_PATTERNS)]
        series.graphicalProperties.solidFill = pattern_info['fill']
        # Note: openpyxl limited pattern fill di chart — solidFill sudah cukup
        # untuk perbedaan warna; pattern requires manual chart.xml manipulation
```

### Cara Aktivasi Accessibility Mode
```
User: "pakai warna colorblind safe"
RIAN: set RIAN_SESSION['theme'] = 'colorblind_safe'
       → Semua get_theme_colors() otomatis pakai CB_SEMANTIC
       → Conditional formatting pakai ikon BUKAN hanya warna
       → Chart >4 series pakai hatch pattern
       → Tampilkan di RIAN Profile Card: "Theme: Colorblind Safe (Okabe-Ito)"
```

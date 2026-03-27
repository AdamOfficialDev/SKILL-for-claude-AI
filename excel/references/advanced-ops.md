# Advanced Operations Reference — RIAN v4
# Formula Arsenal + Advanced Charts + Dashboard + Financial Patterns

## ═══════════════════════════════════════════════════
## 1. FORMULA ARSENAL — 300+ POLA EXCEL
## ═══════════════════════════════════════════════════

### LOOKUP & REFERENCE
```excel
# XLOOKUP (Excel 365/2021) — Lebih powerful dari VLOOKUP
=XLOOKUP(lookup_val, lookup_array, return_array, "Not Found", 0, 1)
=XLOOKUP(val, A:A, B:B)                    # Basic
=XLOOKUP(val, A:A, B:C)                    # Return multiple columns
=XLOOKUP(val, A:A, B:B, "-", 0, -1)       # Last match
=XLOOKUP("*"&val&"*", A:A, B:B, "-", 2)   # Wildcard match

# INDEX + MATCH — Universal fallback
=INDEX(return_range, MATCH(value, lookup_range, 0))
=INDEX(B:B, MATCH(val, A:A, 0))
=INDEX(B2:D100, MATCH(row_val, A2:A100,0), MATCH(col_val, B1:D1,0))  # 2-way

# Multi-criteria MATCH
=INDEX(C:C, MATCH(1, (A:A=x)*(B:B=y), 0))  # CTRL+SHIFT+ENTER array
=INDEX(C:C, MATCH(x&y, A:A&B:B, 0))         # Concatenate keys

# VLOOKUP (legacy — prefer XLOOKUP)
=VLOOKUP(value, table_array, col_index, 0)
=IFERROR(VLOOKUP(val, range, 3, 0), "-")   # Safe VLOOKUP

# Dynamic / Indirect Reference
=INDIRECT("'"&A1&"'!B2")           # Reference ke sheet bernama di A1
=INDIRECT(ADDRESS(ROW(), COL()-1)) # Reference relatif dinamis
=OFFSET(A1, 0, 0, COUNTA(A:A), 1) # Dynamic range

# Last non-empty value
=LOOKUP(2, 1/(A:A<>""), A:A)
=INDEX(A:A, XMATCH(1E+99, A:A, 1))   # Excel 365

# N-th match
=INDEX(B:B, SMALL(IF(A:A=val, ROW(A:A)-1), n))  # CSE array
```

### AGREGASI KONDISIONAL
```excel
# SUMIFS — Multiple criteria
=SUMIFS(sum_range, r1, c1, r2, c2)
=SUMIFS(D:D, A:A, "Januari", B:B, "Jakarta")
=SUMIFS(D:D, A:A, ">="&DATE(2024,1,1), A:A, "<="&DATE(2024,12,31))
=SUMIFS(D:D, B:B, "*"&keyword&"*")           # Wildcard

# SUMPRODUCT — Power aggregation
=SUMPRODUCT((A2:A100="X")*(B2:B100>0)*C2:C100)
=SUMPRODUCT((MONTH(D2:D100)=3)*(YEAR(D2:D100)=2025)*E2:E100)
=SUMPRODUCT(1/COUNTIF(A2:A100,A2:A100))      # Count unique values
=SUMPRODUCT((A2:A100<>"")*1)                  # Count non-empty

# COUNTIFS
=COUNTIFS(A:A, "Aktif", B:B, ">="&100)
=COUNTIFS(A:A, "<>"&"")                       # Count non-blank

# AVERAGEIFS
=AVERAGEIFS(C:C, A:A, "Jakarta", B:B, "Q1")

# MAXIFS / MINIFS (Excel 2019+)
=MAXIFS(value_range, criteria_range, criteria)
=MINIFS(value_range, criteria_range, criteria)
```

### LOGIKA & KONDISI
```excel
# IFS — Multiple conditions (lebih bersih dari nested IF)
=IFS(val>=90,"A", val>=80,"B", val>=70,"C", val>=60,"D", TRUE,"E")

# SWITCH — Pattern matching
=SWITCH(A1, "Jan",1, "Feb",2, "Mar",3, 0)

# Nested IF pattern
=IF(A1>=1000000, "Besar", IF(A1>=100000, "Sedang", "Kecil"))

# IFERROR / IFNA — Error handling
=IFERROR(formula, "-")           # Catch semua error
=IFNA(VLOOKUP(A1,B:C,2,0),"-") # Catch #N/A only

# AND / OR / NOT
=IF(AND(A1>0, B1>0, C1>0), "Semua positif", "Ada negatif")
=IF(OR(A1="Aktif", A1="Probation"), "Karyawan", "Bukan Karyawan")
```

### TEKS & STRING
```excel
# Manipulasi dasar
=TRIM(A1)                                 # Hapus spasi berlebih
=CLEAN(A1)                                # Hapus karakter non-printable
=UPPER(A1)  |  =LOWER(A1)  |  =PROPER(A1)
=LEN(A1)
=LEFT(A1, 5)  |  =RIGHT(A1, 4)  |  =MID(A1, 3, 4)
=SUBSTITUTE(A1, "lama", "baru")
=REPLACE(A1, 1, 3, "XXX")
=REPT("★", A1)                           # Repeat character

# Pencarian
=FIND("@", A1)             # Case-sensitive, error jika tidak ketemu
=SEARCH("@", A1)           # Case-insensitive, wildcard support
=ISNUMBER(SEARCH("kata", A1))  # Contains check

# Gabung / Join
=A1&" "&B1&" "&C1
=TEXTJOIN(", ", TRUE, A1:A10)          # Join dengan delimiter
=CONCAT(A1:A5)                          # Concatenate range

# Format angka ke teks
=TEXT(A1, "#,##0")                       # 1.500.000
=TEXT(A1, "DD/MM/YYYY")                  # 01/01/2025
=TEXT(A1, "DD MMMM YYYY")               # 01 Januari 2025
=TEXT(A1, "0.0%")                        # 25.5%
=TEXT(A1, "[>999999]#,,\"jt\";#,##0")  # Smart IDR

# Extract dari teks
=LEFT(A1, FIND(" ", A1)-1)              # Kata pertama
=MID(A1, FIND(" ",A1)+1, 999)           # Setelah kata pertama
=VALUE(SUBSTITUTE(SUBSTITUTE(A1,"Rp",""),",",""))  # Extract angka dari IDR
=TRIM(MID(A1,FIND("-",A1)+1,99))        # Setelah tanda minus

# Validasi format
=ISNUMBER(MATCH("*@*.*", A1, 0))        # Basic email check
```

### TANGGAL & WAKTU
```excel
=TODAY()  |  =NOW()
=DATE(2025, 1, 31)
=YEAR(A1)  |  =MONTH(A1)  |  =DAY(A1)
=WEEKDAY(A1, 2)          # 1=Senin, 7=Minggu
=WEEKNUM(A1)
=EOMONTH(A1, 0)          # Hari terakhir bulan ini
=EOMONTH(A1, 1)          # Hari terakhir bulan depan
=EDATE(A1, 3)            # +3 bulan

# Selisih tanggal
=DATEDIF(start, end, "D")   # Hari
=DATEDIF(start, end, "M")   # Bulan
=DATEDIF(start, end, "Y")   # Tahun
=DATEDIF(start, end, "YM")  # Sisa bulan (setelah tahun penuh)
=DATEDIF(start, end, "MD")  # Sisa hari (setelah bulan penuh)

# Hari kerja
=NETWORKDAYS(start, end)                   # Exclude Sabtu & Minggu
=NETWORKDAYS.INTL(start, end, 1)          # Custom weekend
=WORKDAY(A1, 10)                           # +10 hari kerja
=NETWORKDAYS(A1, TODAY()) - 1             # Hari kerja sudah lewat

# Format ke teks
=TEXT(A1, "DD/MM/YYYY")
=TEXT(A1, "MMM-YYYY")                     # Jan-2025
=TEXT(A1, "MMMM YYYY")                   # Januari 2025
="Q"&INT((MONTH(A1)-1)/3+1)&"-"&YEAR(A1) # Q1-2025

# Validasi
=AND(ISNUMBER(A1), A1>=DATE(2000,1,1))
```

### DYNAMIC ARRAYS (Excel 365)
```excel
=SORT(array)                              # Sort ascending
=SORT(array, 2, -1)                       # Sort by col 2, descending
=SORTBY(array, by_col, 1, by_col2, -1)   # Multi-sort
=FILTER(array, (A2:A100="X")*(B2:B100>0))# Multi-criteria filter
=UNIQUE(A2:A100)                           # Remove duplicates
=SEQUENCE(12, 1, 1)                        # 1 to 12
=RANDARRAY(5, 3, 1, 100, TRUE)            # 5×3 random integers
=TOCOL(range)  |  =TOROW(range)          # Reshape
=HSTACK(arr1, arr2)  |  =VSTACK(arr1,arr2) # Combine arrays
=TAKE(array, 5)                            # First 5 rows
=DROP(array, -5)                           # Remove last 5
=CHOOSEROWS(array, 1,3,5)                 # Select specific rows
=CHOOSECOLS(array, 1,3,5)                 # Select specific cols
```

### LAMBDA & LET (Excel 365)
```excel
# LET — Definisi variabel lokal
=LET(
  revenue,  SUM(B2:B13),
  cost,     SUM(C2:C13),
  profit,   revenue - cost,
  margin,   IF(revenue=0, 0, profit/revenue),
  TEXT(margin, "0.0%")
)

# LAMBDA — Custom function reusable
=LAMBDA(x, y, IFERROR(x/y, 0))          # Simple division safe
=BYROW(B2:D10, LAMBDA(r, SUM(r)))       # Sum per row
=BYCOL(B2:D10, LAMBDA(c, AVERAGE(c)))  # Average per column
=MAP(A2:A10, LAMBDA(x, x*1.11))         # Apply PPN 11%
=REDUCE(0, A2:A10, LAMBDA(acc, x, acc+IF(x>0,x,0)))  # Conditional sum
=MAKEARRAY(5, 3, LAMBDA(r, c, r*c))    # Generate multiplication table
```

### FINANSIAL
```excel
=NPV(rate, cashflow_range)
=XNPV(rate, cashflows, dates)
=IRR(cashflows)
=XIRR(cashflows, dates)
=PMT(rate/12, nper*12, -pv)             # Cicilan KPR bulanan
=IPMT(rate/12, period, nper*12, pv)     # Bunga di periode tertentu
=PPMT(rate/12, period, nper*12, pv)     # Pokok di periode tertentu
=FV(rate, nper, -pmt)                   # Future value
=PV(rate, nper, pmt)                    # Present value

# Penyusutan
=SLN(cost, salvage, life)               # Garis lurus
=DB(cost, salvage, life, period)        # Declining balance
=DDB(cost, salvage, life, period)       # Double declining
=VDB(cost, salvage, life, s, e)         # Variable declining
=SYD(cost, salvage, life, period)       # Sum-of-years digits
```

### STATISTIK
```excel
=STDEV(range)     |  =STDEVP(range)    # Sampel vs Populasi
=VAR(range)       |  =VARP(range)
=CORREL(arr1, arr2)
=COVARIANCE.S(arr1, arr2)
=PERCENTILE.INC(range, 0.75)           # Q3
=QUARTILE.INC(range, 1)                # Q1
=RANK.EQ(val, ref, 0)                  # Rank (tie = equal rank)
=LARGE(range, 1)  |  =SMALL(range, 1) # Max / Min (by rank)
=FREQUENCY(data_array, bins_array)     # Distribusi frekuensi
=FORECAST.LINEAR(x, known_y, known_x) # Linear prediction
=TREND(known_y, known_x, new_x)       # Array forecast
=LINEST(known_y, known_x)             # Regression stats
=NORM.DIST(x, mean, sd, TRUE)         # Normal CDF
=NORM.INV(probability, mean, sd)      # Inverse normal
=T.TEST(arr1, arr2, 2, 2)             # Independent t-test
=CHISQ.TEST(actual, expected)         # Chi-square test
```

---

## ═══════════════════════════════════════════════════
## 2. FORMULA PATTERNS PER DOMAIN
## ═══════════════════════════════════════════════════

### HR Formulas
```excel
# Masa kerja
=DATEDIF([@[Tgl Masuk]], TODAY(), "Y")         # Tahun
=DATEDIF([@[Tgl Masuk]], TODAY(), "M")         # Bulan total
=DATEDIF([@[Tgl Masuk]], TODAY(), "Y")&" thn "&DATEDIF([@[Tgl Masuk]],TODAY(),"YM")&" bln"

# Kategori masa kerja
=IFS([@[Masa Kerja]]<1,"< 1 Tahun",[@[Masa Kerja]]<3,"1-3 Tahun",[@[Masa Kerja]]<5,"3-5 Tahun",TRUE,"> 5 Tahun")

# Usia
=DATEDIF([@[Tgl Lahir]], TODAY(), "Y")

# Status aktif
=IF([@Status]="Aktif","✅ Aktif",IF([@Status]="Resign","❌ Resign","⚠️ "& [@Status]))

# Turnover rate
=(COUNTIF(Status_Range,"Resign")/AVERAGE(awal,akhir))*100

# Kategori gaji
=PERCENTRANK.INC(GajiRange, [@Gaji], 4)
```

### Sales Formulas
```excel
# Subtotal dengan diskon
=[@Qty]*[@[Harga Satuan]]*(1-[@[Diskon (%)]]/100)

# PPN 11%
=[@Subtotal]*11%

# Achievement %
=IFERROR([@Aktual]/[@Target], 0)

# Status Achievement
=IF([@Ach%]>=1,"✅ On Track",IF([@Ach%]>=0.8,"⚠️ At Risk","❌ Below Target"))

# YTD Cumulative (di pivot/summary)
=SUMIFS(Aktual, Bulan, "<="&MaksMonth)

# Growth YoY
=IFERROR(([@[2025]]-[@[2024]])/[@[2024]], 0)

# Kontribusi / Share
=IFERROR([@Revenue]/SUM(Revenue_Range), 0)

# Ranking sales person
=RANK([@Total], Total_Range, 0)

# Quarter
="Q"&INT((MONTH([@Tanggal])-1)/3+1)&"-"&YEAR([@Tanggal])
```

### Finance Formulas
```excel
# Running saldo
=IFERROR(SUM($C$2:C2)-SUM($D$2:D2), 0)   # Masuk - Keluar kumulatif

# Check debit=kredit
=IF(SUM(Debit_Range)=SUM(Kredit_Range),"✅ BALANCE","❌ SELISIH: "&TEXT(ABS(SUM(Debit)-SUM(Kredit)),"#,##0"))

# Gross profit margin
=IFERROR((Revenue-HPP)/Revenue, 0)

# EBITDA margin
=IFERROR(EBITDA/Revenue, 0)

# Current ratio
=IFERROR(Aset_Lancar/Liabilitas_Lancar, 0)

# Debt to equity ratio
=IFERROR(Total_Liabilitas/Total_Ekuitas, 0)

# Days Sales Outstanding (DSO)
=IFERROR(Piutang/(Revenue/365), 0)

# Days Inventory Outstanding (DIO)
=IFERROR(Persediaan/(HPP/365), 0)
```

### Inventory Formulas
```excel
# Status stok
=IF([@Aktual]<=[@Minimum],"🔴 KRITIS",IF([@Aktual]<=[@Minimum]*1.5,"🟡 RENDAH",IF([@Aktual]>=[@Maksimum],"🔵 OVER","🟢 AMAN")))

# Nilai stok
=[@[Stok Aktual]]*[@[Harga Beli]]

# Kebutuhan reorder
=MAX(0, [@[Reorder Point]]-[@[Stok Aktual]])

# Reorder point
=[@[Avg Daily Usage]]*[@[Lead Time]]

# Stock coverage (hari)
=IFERROR([@[Stok Aktual]]/[@[Avg Daily Usage]], 0)

# ABC Classification berdasarkan nilai stok
=IF(PERCENTRANK.INC(Nilai_Range,[@[Nilai Stok]])>=0.8,"A",IF(PERCENTRANK.INC(Nilai_Range,[@[Nilai Stok]])>=0.5,"B","C"))
```

### Project Formulas
```excel
# Sisa hari
=IF([@Status]="Done", 0, MAX(0, [@Deadline]-TODAY()))

# Status otomatis
=IF([@[% Progress]]=100,"✅ Done",IF([@Deadline]<TODAY(),"🔴 Overdue",IF([@Deadline]-TODAY()<=3,"🟡 Due Soon","🔵 On Track")))

# Duration rencana
=NETWORKDAYS([@[Tgl Mulai]], [@Deadline]) - 1

# % waktu terpakai
=IFERROR((TODAY()-[@[Tgl Mulai]])/NETWORKDAYS([@[Tgl Mulai]],[@Deadline]), 0)

# On time indicator
=IF([@[% Progress]]>=[@[% Waktu Terpakai]], "✅ Ahead", "⚠️ Behind")
```

---

## ═══════════════════════════════════════════════════
## 3. ADVANCED CHART PATTERNS
## ═══════════════════════════════════════════════════

### Waterfall Chart (via Bar Chart)
```python
def create_waterfall_chart(wb, items, values, title, sheet_name="WATERFALL"):
    """
    Waterfall chart via stacked bar trick.
    items  = ['Revenue', 'COGS', 'Gross Profit', 'OpEx', 'EBITDA', ...]
    values = [1000, -400, 600, -150, 450, ...]  # negatif = pengurangan
    """
    ws = wb.create_sheet(sheet_name)
    ws.sheet_properties.tabColor = "1F4E79"
    ws.sheet_view.showGridLines = False
    
    # Hitung invisible base (stacked bar trick)
    running = 0
    bases = []
    positives = []
    negatives = []
    
    for val in values:
        if val >= 0:
            bases.append(running)
            positives.append(val)
            negatives.append(0)
            running += val
        else:
            bases.append(running + val)
            positives.append(0)
            negatives.append(-val)
            running += val
    
    # Write data
    headers = ['Item', 'Base (hidden)', 'Kenaikan', 'Penurunan', 'Nilai Aktual']
    for c, h in enumerate(headers, 1):
        ws.cell(1, c, h).font = Font(bold=True)
    
    for r, (item, base, pos, neg, val) in enumerate(zip(items, bases, positives, negatives, values), 2):
        ws.cell(r, 1, item)
        ws.cell(r, 2, base).number_format = '#,##0'
        ws.cell(r, 3, pos).number_format = '#,##0'
        ws.cell(r, 4, neg).number_format = '#,##0'
        ws.cell(r, 5, val).number_format = '#,##0'
    
    # Chart
    chart = BarChart()
    chart.type = "col"
    chart.title = title
    chart.style = 10
    chart.grouping = "stacked"
    chart.width = 24
    chart.height = 14
    chart.overlap = 100
    
    max_row = len(items) + 1
    
    # Base series (invisible)
    base_data = Reference(ws, min_col=2, max_col=2, min_row=1, max_row=max_row)
    chart.add_data(base_data, titles_from_data=True)
    
    # Positive series
    pos_data = Reference(ws, min_col=3, max_col=3, min_row=1, max_row=max_row)
    chart.add_data(pos_data, titles_from_data=True)
    
    # Negative series
    neg_data = Reference(ws, min_col=4, max_col=4, min_row=1, max_row=max_row)
    chart.add_data(neg_data, titles_from_data=True)
    
    cats = Reference(ws, min_col=1, min_row=2, max_row=max_row)
    chart.set_categories(cats)
    
    # Make base series invisible
    if len(chart.series) > 0:
        chart.series[0].graphicalProperties.solidFill = "FFFFFF"  # White = invisible
        chart.series[0].graphicalProperties.line.solidFill = "FFFFFF"
    
    ws.add_chart(chart, "A" + str(max_row + 3))
    return ws

### Gantt Chart
def create_gantt_chart(wb, tasks, sheet_name="GANTT"):
    """
    tasks = [
        {'task': 'Design', 'start': date(2025,1,1), 'end': date(2025,1,15), 'pic': 'Budi', 'progress': 100, 'category': 'Phase 1'},
        {'task': 'Development', 'start': date(2025,1,10), 'end': date(2025,2,28), 'pic': 'Ani', 'progress': 60, 'category': 'Phase 2'},
    ]
    """
    ws = wb.create_sheet(sheet_name)
    ws.sheet_properties.tabColor = "1F4E79"
    ws.sheet_view.showGridLines = False
    
    if not tasks:
        return ws
    
    start_dates = [t['start'] for t in tasks if isinstance(t['start'], date)]
    end_dates   = [t['end']   for t in tasks if isinstance(t['end'], date)]
    
    if not start_dates:
        return ws
    
    project_start = min(start_dates)
    project_end   = max(end_dates)
    
    # Header baris 1: Bulan
    info_cols = 5  # Task, PIC, Start, End, Progress
    
    headers = ['Task', 'PIC', 'Mulai', 'Selesai', '% Done']
    for c, h in enumerate(headers, 1):
        cell = ws.cell(1, c, h)
        cell.font = Font(bold=True, color="FFFFFF", size=10, name="Calibri")
        cell.fill = PatternFill("solid", fgColor="1F4E79")
        cell.alignment = Alignment(horizontal="center", vertical="center")
    
    # Generate date columns (weekly)
    date_col = info_cols + 1
    current_date = project_start
    date_cols = {}
    
    while current_date <= project_end + timedelta(days=7):
        # Week header
        cell = ws.cell(1, date_col)
        cell.value = current_date.strftime("%d-%b")
        cell.font = Font(bold=True, color="FFFFFF", size=8, name="Calibri")
        cell.fill = PatternFill("solid", fgColor="2E75B6")
        cell.alignment = Alignment(horizontal="center")
        ws.column_dimensions[get_column_letter(date_col)].width = 6
        
        date_cols[current_date] = date_col
        date_col += 1
        current_date += timedelta(days=7)
    
    ws.row_dimensions[1].height = 25
    
    # Task rows
    category_colors = {
        'Phase 1': '70AD47', 'Phase 2': '4472C4',
        'Phase 3': 'ED7D31', 'Phase 4': 'FF0000',
    }
    
    for r_idx, task in enumerate(tasks, 2):
        # Task info
        ws.cell(r_idx, 1, task.get('task', '')).font = Font(size=9, name="Calibri")
        ws.cell(r_idx, 2, task.get('pic', '')).font = Font(size=9, name="Calibri")
        ws.cell(r_idx, 3, task.get('start', '')).number_format = 'DD/MM'
        ws.cell(r_idx, 4, task.get('end', '')).number_format = 'DD/MM'
        
        prog = task.get('progress', 0)
        prog_cell = ws.cell(r_idx, 5, prog/100 if prog > 1 else prog)
        prog_cell.number_format = '0%'
        prog_cell.alignment = Alignment(horizontal="center")
        
        ws.row_dimensions[r_idx].height = 18
        
        # Draw Gantt bars
        task_cat = task.get('category', 'Phase 1')
        bar_color = category_colors.get(task_cat, '4472C4')
        
        t_start = task.get('start')
        t_end   = task.get('end')
        
        if not isinstance(t_start, date) or not isinstance(t_end, date):
            continue
        
        for col_date, col_idx in date_cols.items():
            week_end = col_date + timedelta(days=6)
            if col_date <= t_end and week_end >= t_start:
                cell = ws.cell(r_idx, col_idx)
                cell.fill = PatternFill("solid", fgColor=bar_color)
        
        # Alternating row bg
        if r_idx % 2 == 0:
            for c in range(1, info_cols + 1):
                if not ws.cell(r_idx, c).fill.fgColor.rgb or ws.cell(r_idx, c).fill.fgColor.rgb == '00000000':
                    ws.cell(r_idx, c).fill = PatternFill("solid", fgColor="F2F7FF")
    
    # Set info column widths
    for w, col in zip([25, 12, 9, 9, 7], range(1, 6)):
        ws.column_dimensions[get_column_letter(col)].width = w
    
    ws.freeze_panes = 'F2'
    return ws
```

---

## ═══════════════════════════════════════════════════
## 4. DASHBOARD LAYOUT ENGINE
## ═══════════════════════════════════════════════════

```python
def build_executive_dashboard(wb, df_dict, domain="general", title=None):
    """
    Auto-build dashboard eksekutif yang komprehensif.
    Layout: Title bar → KPI Cards → Charts → Top-N Tables
    """
    ws = wb.create_sheet("DASHBOARD", 0)
    ws.sheet_properties.tabColor = "1F4E79"
    ws.sheet_view.showGridLines = False
    ws.sheet_view.showRowColHeaders = False
    
    # ── Title Bar ───────────────────────────────────
    ws.merge_cells('A1:Z1')
    ws.merge_cells('A2:Z2')
    
    title_text = title or f"EXECUTIVE DASHBOARD — {domain.upper()}"
    t = ws['A1']
    t.value = title_text
    t.font = Font(size=20, bold=True, color="FFFFFF", name="Calibri")
    t.fill = PatternFill("solid", fgColor="1F4E79")
    t.alignment = Alignment(horizontal="center", vertical="center")
    ws.row_dimensions[1].height = 45
    
    sub = ws['A2']
    sub.value = f"Per: {pd.Timestamp.now().strftime('%d %B %Y, %H:%M WIB')} | Dibuat oleh RIAN"
    sub.font = Font(size=10, color="FFFFFF", italic=True, name="Calibri")
    sub.fill = PatternFill("solid", fgColor="2E75B6")
    sub.alignment = Alignment(horizontal="center", vertical="center")
    ws.row_dimensions[2].height = 20
    
    # ── Auto-generate content berdasarkan data ───────
    main_df = list(df_dict.values())[0] if df_dict else pd.DataFrame()
    num_cols = main_df.select_dtypes(include='number').columns.tolist() if len(main_df) > 0 else []
    
    # KPI Cards row 4
    kpis = []
    for col in num_cols[:4]:
        total = main_df[col].sum()
        avg   = main_df[col].mean()
        kpis.append({
            'label': f"Total {col}",
            'value': total,
            'fmt': 'idr' if any(k in col.lower() for k in ['harga','gaji','biaya','revenue','total']) else 'integer',
        })
    
    if kpis:
        create_kpi_cards(ws, kpis, start_row=4, start_col=2, card_width=4)
    
    return ws

def add_insight_text_box(ws, row, col, insights, title="💡 INSIGHT & TEMUAN",
                          merge_cols=8, bg_color="F0F5FF"):
    """Tambah text box insight ke dashboard"""
    # Title
    ws.merge_cells(start_row=row, end_row=row,
                   start_column=col, end_column=col+merge_cols-1)
    t = ws.cell(row, col, value=title)
    t.font = Font(bold=True, size=11, color="1F4E79", name="Calibri")
    t.fill = PatternFill("solid", fgColor="1F4E79")
    t.font = Font(bold=True, size=11, color="FFFFFF", name="Calibri")
    t.alignment = Alignment(vertical="center", indent=1)
    ws.row_dimensions[row].height = 22
    
    # Insights
    for i, insight in enumerate(insights[:6], 1):
        cell = ws.cell(row + i, col, value=f"  {i}. {insight}")
        cell.font = Font(size=10, name="Calibri", color="333333")
        cell.fill = PatternFill("solid", fgColor=bg_color)
        cell.alignment = Alignment(vertical="center", indent=1)
        ws.merge_cells(start_row=row+i, end_row=row+i,
                       start_column=col, end_column=col+merge_cols-1)
        ws.row_dimensions[row + i].height = 18
    
    # Border
    thin = Side(style="thin", color="BDD7EE")
    for r in range(row, row + len(insights[:6]) + 2):
        for c in range(col, col + merge_cols):
            ws.cell(r, c).border = Border(left=thin, right=thin, top=thin, bottom=thin)
```

---

## ═══════════════════════════════════════════════════
## 5. AUTO-INSIGHTS ENGINE
## ═══════════════════════════════════════════════════

```python
def generate_auto_insights(df, domain="general"):
    """Generate insight otomatis dari data"""
    insights = []
    
    num_cols = df.select_dtypes(include='number').columns.tolist()
    cat_cols = df.select_dtypes(include='object').columns.tolist()
    date_cols = [c for c in df.columns if any(k in c.lower() 
                  for k in ['tanggal','date','tgl'])]
    
    # Insight umum
    total_rows = len(df)
    null_pct = df.isnull().mean().mean() * 100
    dups = df.duplicated().sum()
    
    insights.append(f"Dataset berisi {total_rows:,} baris dengan {len(df.columns)} kolom")
    if null_pct > 5:
        insights.append(f"⚠️ {null_pct:.0f}% data kosong — pertimbangkan untuk diisi atau dibersihkan")
    if dups > 0:
        insights.append(f"⚠️ Ditemukan {dups:,} baris duplikat")
    
    # Insight numerik
    for col in num_cols[:3]:
        s = df[col].dropna()
        if len(s) == 0: continue
        total = s.sum()
        mean  = s.mean()
        
        # Deteksi kolom revenue/nilai utama
        if any(k in col.lower() for k in ['revenue','total','penjualan','omzet','pendapatan']):
            insights.append(f"Total {col}: {total:,.0f} | Rata-rata per transaksi: {mean:,.0f}")
        
        # Outlier check
        Q1, Q3 = s.quantile(0.25), s.quantile(0.75)
        IQR = Q3 - Q1
        outlier_count = ((s < Q1 - 1.5*IQR) | (s > Q3 + 1.5*IQR)).sum()
        if outlier_count > 0:
            insights.append(f"⚠️ Kolom '{col}' memiliki {outlier_count} nilai outlier")
    
    # Insight kategoris
    for col in cat_cols[:2]:
        n_unique = df[col].nunique()
        top_val = df[col].value_counts().index[0] if len(df[col].dropna()) > 0 else "-"
        top_pct = df[col].value_counts().iloc[0] / len(df[col].dropna()) * 100 if len(df[col].dropna()) > 0 else 0
        if n_unique > 1:
            insights.append(f"Kolom '{col}': {n_unique} kategori unik | Dominan: '{top_val}' ({top_pct:.0f}%)")
    
    # Domain-specific insights
    if domain == 'sales':
        growth_cols = [c for c in df.columns if any(k in c.lower() for k in ['target','actual','aktual'])]
        if len(growth_cols) >= 2:
            insights.append("Gunakan sheet TARGET vs AKTUAL untuk analisis pencapaian per periode")
    elif domain == 'hr':
        if any('resign' in str(v).lower() for col in cat_cols for v in df[col].dropna()):
            resign_count = sum(df[cat_cols[0]].str.lower().eq('resign') if cat_cols else [])
            if resign_count > 0:
                insights.append(f"⚠️ {resign_count} karyawan berstatus Resign — perlu analisis turnover")
    elif domain == 'inventory':
        stok_cols = [c for c in num_cols if any(k in c.lower() for k in ['stok','stock','aktual'])]
        min_cols  = [c for c in num_cols if any(k in c.lower() for k in ['minimum','min'])]
        if stok_cols and min_cols:
            kritis = (df[stok_cols[0]] <= df[min_cols[0]]).sum()
            if kritis > 0:
                insights.append(f"🔴 {kritis} item stok di bawah minimum — segera reorder!")
    
    return insights[:8]  # Max 8 insights
```

---

## ═══════════════════════════════════════════════════
## 6. NAMED RANGES & STRUCTURED REFERENCES
## ═══════════════════════════════════════════════════

```python
from openpyxl.workbook.defined_name import DefinedName
from openpyxl.utils import quote_sheetname

def create_named_range(wb, name, sheet_name, cell_ref):
    """Buat named range untuk formula yang mudah dibaca"""
    safe_name = re.sub(r'[^A-Za-z0-9_]', '_', name)
    full_ref = f"{quote_sheetname(sheet_name)}!{cell_ref}"
    wb.defined_names[safe_name] = DefinedName(safe_name, attr_text=full_ref)

def setup_model_names(wb, assumptions_sheet, assumption_map):
    """
    Setup named ranges untuk financial model.
    assumption_map = {'Growth_Rate': 'C5', 'Tax_Rate': 'C6', ...}
    """
    for name, cell_ref in assumption_map.items():
        create_named_range(wb, name, assumptions_sheet, f"${cell_ref}")
    
    print(f"✅ {len(assumption_map)} named ranges dibuat")
```

---

## ═══════════════════════════════════════════════════
## 7. SCENARIO ANALYSIS TEMPLATE
## ═══════════════════════════════════════════════════

```python
def build_scenario_table(ws, scenarios, formula_cells, start_row=2, start_col=2):
    """
    Buat tabel scenario analysis.
    scenarios = {
        'Base Case':  {'growth': 0.10, 'margin': 0.25, 'discount': 0.12},
        'Bull Case':  {'growth': 0.20, 'margin': 0.30, 'discount': 0.10},
        'Bear Case':  {'growth': 0.02, 'margin': 0.18, 'discount': 0.15},
    }
    """
    # Headers
    headers = ['Skenario'] + list(list(scenarios.values())[0].keys())
    for c, h in enumerate(headers, start_col):
        cell = ws.cell(start_row, c, h)
        cell.font = Font(bold=True, color="FFFFFF", name="Calibri")
        cell.fill = PatternFill("solid", fgColor="1F4E79")
        cell.alignment = Alignment(horizontal="center", vertical="center")
    
    # Scenario rows
    colors = {'Base Case': 'DEEAF1', 'Bull Case': 'E2EFDA', 'Bear Case': 'FCE4D6'}
    for r_idx, (scen_name, params) in enumerate(scenarios.items(), start_row + 1):
        row_color = colors.get(scen_name, 'F2F7FF')
        
        name_cell = ws.cell(r_idx, start_col, scen_name)
        name_cell.font = Font(bold=True, name="Calibri")
        name_cell.fill = PatternFill("solid", fgColor=row_color)
        
        for c_idx, val in enumerate(params.values(), start_col + 1):
            cell = ws.cell(r_idx, c_idx, val)
            cell.fill = PatternFill("solid", fgColor=row_color)
            cell.alignment = Alignment(horizontal="center")
            if isinstance(val, float) and val <= 1:
                cell.number_format = '0.0%'
    
    ws.row_dimensions[start_row].height = 22
```


---

## ═══════════════════════════════════════════════════
## 8. WATERFALL CHART (v4 NEW)
## ═══════════════════════════════════════════════════

> Waterfall chart = visualisasi perubahan kumulatif (budget vs realisasi, laba-rugi).
> openpyxl tidak punya built-in waterfall → gunakan Stacked Bar trick.

```python
def create_waterfall_chart(ws, data: list, chart_title: str = "Waterfall Chart",
                            start_row: int = 2, start_col: int = 1) -> None:
    """
    Buat waterfall chart menggunakan stacked bar trick.
    
    data = [
        {'label': 'Pendapatan',    'value': 1000, 'type': 'total'},
        {'label': 'HPP',           'value': -300, 'type': 'negative'},
        {'label': 'Beban Operasi', 'value': -200, 'type': 'negative'},
        {'label': 'Laba Bersih',   'value': 500,  'type': 'total'},
    ]
    """
    from openpyxl.chart import BarChart, Reference, Series
    from openpyxl.chart.series import SeriesLabel
    
    # ── Hitung invisible base + visible bar ───────────────────────
    helper_data = []
    cumulative = 0
    
    for item in data:
        val = item['value']
        item_type = item.get('type', 'positive' if val >= 0 else 'negative')
        
        if item_type == 'total':
            invisible = 0
            visible = val
            cumulative = val
        elif val >= 0:
            invisible = cumulative
            visible = val
            cumulative += val
        else:
            invisible = cumulative + val  # cumulative sudah berkurang
            visible = -val               # bar ke atas (positif) untuk loss
            cumulative += val
        
        helper_data.append({
            'label': item['label'],
            'invisible': invisible,
            'visible': abs(visible),
            'is_negative': val < 0 and item_type != 'total',
            'is_total': item_type == 'total',
        })
    
    # ── Tulis helper data ke sheet ───────────────────────────────
    ws.cell(start_row - 1, start_col, 'Label')
    ws.cell(start_row - 1, start_col + 1, 'Invisible')
    ws.cell(start_row - 1, start_col + 2, 'Nilai')
    ws.cell(start_row - 1, start_col + 3, 'Type')
    
    for i, row in enumerate(helper_data):
        r = start_row + i
        ws.cell(r, start_col,     row['label'])
        ws.cell(r, start_col + 1, row['invisible'])
        ws.cell(r, start_col + 2, row['visible'])
        ws.cell(r, start_col + 3, 'total' if row['is_total'] else ('neg' if row['is_negative'] else 'pos'))
    
    # ── Buat chart ────────────────────────────────────────────────
    chart = BarChart()
    chart.type = "col"
    chart.grouping = "stacked"
    chart.title = chart_title
    chart.y_axis.title = "Nilai (Rp)"
    chart.style = 10
    chart.width = 20
    chart.height = 14
    
    data_rows = len(helper_data)
    
    # Series invisible (transparan)
    invisible_ref = Reference(ws,
        min_col=start_col + 1, max_col=start_col + 1,
        min_row=start_row - 1, max_row=start_row + data_rows - 1)
    invisible_series = Series(invisible_ref, title="Invisible")
    invisible_series.graphicalProperties.solidFill = "FFFFFF"  # White = transparan
    invisible_series.graphicalProperties.line.solidFill = "FFFFFF"
    chart.series.append(invisible_series)
    
    # Series visible (nilai sebenarnya)
    visible_ref = Reference(ws,
        min_col=start_col + 2, max_col=start_col + 2,
        min_row=start_row - 1, max_row=start_row + data_rows - 1)
    visible_series = Series(visible_ref, title=chart_title)
    visible_series.graphicalProperties.solidFill = "2E75B6"  # Blue untuk positif
    chart.series.append(visible_series)
    
    # Label (kategori)
    cats = Reference(ws,
        min_col=start_col, max_col=start_col,
        min_row=start_row, max_row=start_row + data_rows - 1)
    chart.set_categories(cats)
    
    # Tempatkan chart
    chart_anchor = ws.cell(start_row + data_rows + 2, start_col).coordinate
    ws.add_chart(chart, chart_anchor)
    
    print(f"✅ Waterfall chart '{chart_title}' dibuat di {chart_anchor}")
```

---

## ═══════════════════════════════════════════════════
## 9. GANTT CHART VIA CONDITIONAL FORMATTING (v4 NEW)
## ═══════════════════════════════════════════════════

```python
def create_gantt_chart(ws_gantt, tasks: list, 
                        project_start=None, project_end=None) -> None:
    """
    Buat Gantt Chart menggunakan conditional formatting.
    
    tasks = [
        {'task': 'Perencanaan',  'start': '2025-01-01', 'end': '2025-01-15', 'pic': 'Budi', 'progress': 100},
        {'task': 'Development',  'start': '2025-01-10', 'end': '2025-02-28', 'pic': 'Ani',  'progress': 60},
        {'task': 'Testing',      'start': '2025-02-15', 'end': '2025-03-15', 'pic': 'Cici', 'progress': 0},
        {'task': 'Go Live',      'start': '2025-03-16', 'end': '2025-03-31', 'pic': 'Budi', 'progress': 0},
    ]
    """
    import pandas as pd
    from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
    from openpyxl.formatting.rule import FormulaRule, DifferentialStyle
    from openpyxl.utils import get_column_letter
    from datetime import datetime, timedelta
    
    COLORS = {
        'header_bg':   '1F4E79',
        'header_text': 'FFFFFF',
        'bar_active':  '2E75B6',    # Bar sedang berjalan
        'bar_done':    '70AD47',    # Bar selesai (progress 100%)
        'bar_late':    'C00000',    # Bar terlambat (past end date, belum 100%)
        'weekend':     'F2F2F2',    # Kolom weekend
        'today':       'FFC000',    # Kolom hari ini
        'task_alt':    'EBF3FB',    # Baris ganjil
    }
    
    # Tentukan range tanggal
    all_dates = []
    for t in tasks:
        all_dates.extend([pd.to_datetime(t['start']), pd.to_datetime(t['end'])])
    
    if not all_dates:
        return
    
    if project_start:
        start_date = pd.to_datetime(project_start)
    else:
        start_date = min(all_dates) - timedelta(days=2)
    
    if project_end:
        end_date = pd.to_datetime(project_end)
    else:
        end_date = max(all_dates) + timedelta(days=5)
    
    date_range = pd.date_range(start=start_date, end=end_date, freq='D')
    
    # ── Layout kolom ─────────────────────────────────────────────
    COL_TASK    = 1
    COL_PIC     = 2
    COL_START   = 3
    COL_END     = 4
    COL_PROGRESS= 5
    COL_BARS    = 6  # Kolom pertama bar gantt
    
    HEADER_ROW  = 1
    MONTH_ROW   = 2
    DATE_ROW    = 3
    DATA_START  = 4
    
    # ── Header tetap ─────────────────────────────────────────────
    headers_fixed = ['Task / Aktivitas', 'PIC', 'Mulai', 'Selesai', 'Progress']
    for c, h in enumerate(headers_fixed, 1):
        cell = ws_gantt.cell(HEADER_ROW, c, h)
        cell.font = Font(bold=True, color=COLORS['header_text'], name='Calibri', size=10)
        cell.fill = PatternFill('solid', fgColor=COLORS['header_bg'])
        cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
    
    # ── Header tanggal (baris 2 = bulan, baris 3 = tanggal) ──────
    current_month = None
    month_start_col = COL_BARS
    
    for d_idx, date in enumerate(date_range):
        col = COL_BARS + d_idx
        
        # Baris bulan (merge per bulan nanti)
        if date.month != current_month:
            current_month = date.month
            month_cell = ws_gantt.cell(MONTH_ROW, col, date.strftime('%b %Y'))
            month_cell.font = Font(bold=True, color=COLORS['header_text'], name='Calibri', size=9)
            month_cell.fill = PatternFill('solid', fgColor='2E4057')
            month_cell.alignment = Alignment(horizontal='center')
        
        # Baris tanggal
        day_cell = ws_gantt.cell(DATE_ROW, col, date.day)
        day_cell.font = Font(bold=True, name='Calibri', size=8,
                             color=COLORS['header_text'])
        
        # Weekend = warna berbeda
        if date.weekday() >= 5:  # Sabtu/Minggu
            day_cell.fill = PatternFill('solid', fgColor='5A6475')
        else:
            day_cell.fill = PatternFill('solid', fgColor=COLORS['header_bg'])
        
        day_cell.alignment = Alignment(horizontal='center')
        ws_gantt.column_dimensions[get_column_letter(col)].width = 2.5
    
    # ── Data tasks ───────────────────────────────────────────────
    today = pd.Timestamp.now().normalize()
    
    for t_idx, task in enumerate(tasks):
        row = DATA_START + t_idx
        task_start = pd.to_datetime(task['start'])
        task_end   = pd.to_datetime(task['end'])
        progress   = task.get('progress', 0)
        
        # Kolom info
        ws_gantt.cell(row, COL_TASK, task['task']).font = Font(name='Calibri', size=10)
        ws_gantt.cell(row, COL_PIC, task.get('pic', '-')).font = Font(name='Calibri', size=9)
        
        start_cell = ws_gantt.cell(row, COL_START, task_start.date())
        start_cell.number_format = 'DD/MM/YY'
        
        end_cell = ws_gantt.cell(row, COL_END, task_end.date())
        end_cell.number_format = 'DD/MM/YY'
        
        prog_cell = ws_gantt.cell(row, COL_PROGRESS, progress / 100)
        prog_cell.number_format = '0%'
        
        # Warna baris alternating
        if t_idx % 2 == 0:
            for c in range(1, COL_BARS):
                ws_gantt.cell(row, c).fill = PatternFill('solid', fgColor=COLORS['task_alt'])
        
        # ── Isi bar Gantt ─────────────────────────────────────────
        for d_idx, date in enumerate(date_range):
            col = COL_BARS + d_idx
            cell = ws_gantt.cell(row, col, ' ')
            
            if task_start <= date <= task_end:
                # Tentukan warna bar
                if progress == 100:
                    bar_color = COLORS['bar_done']
                elif date < today and progress < 100:
                    bar_color = COLORS['bar_late']
                else:
                    bar_color = COLORS['bar_active']
                
                cell.fill = PatternFill('solid', fgColor=bar_color)
            
            elif date.weekday() >= 5:
                cell.fill = PatternFill('solid', fgColor=COLORS['weekend'])
            
            # Highlight hari ini
            if date.normalize() == today:
                cell.fill = PatternFill('solid', fgColor=COLORS['today'])
            
            # Border tipis
            thin = Side(style='thin', color='DDDDDD')
            cell.border = Border(right=thin)
    
    # ── Column widths tetap ──────────────────────────────────────
    ws_gantt.column_dimensions['A'].width = 28
    ws_gantt.column_dimensions['B'].width = 12
    ws_gantt.column_dimensions['C'].width = 10
    ws_gantt.column_dimensions['D'].width = 10
    ws_gantt.column_dimensions['E'].width = 9
    
    # ── Freeze panes ─────────────────────────────────────────────
    ws_gantt.freeze_panes = ws_gantt.cell(DATA_START, COL_BARS)
    
    # ── Legenda ──────────────────────────────────────────────────
    legend_row = DATA_START + len(tasks) + 2
    ws_gantt.cell(legend_row, 1, 'Legenda:').font = Font(bold=True, size=9)
    
    legends = [
        ('Sedang Berjalan', COLORS['bar_active']),
        ('Selesai (100%)',   COLORS['bar_done']),
        ('Terlambat',        COLORS['bar_late']),
        ('Hari Ini',         COLORS['today']),
    ]
    for i, (label, color) in enumerate(legends):
        col = 2 + (i * 2)
        ws_gantt.cell(legend_row, col, ' ').fill = PatternFill('solid', fgColor=color)
        ws_gantt.cell(legend_row, col + 1, label).font = Font(size=9)
    
    ws_gantt.sheet_properties.tabColor = "2E75B6"
    print(f"✅ Gantt Chart dibuat: {len(tasks)} task | {len(date_range)} hari")


# ── Combo Chart (Bar + Line, Secondary Axis) ─────────────────────────

def create_combo_chart(ws, chart_title: str, bar_data_range, line_data_range,
                        categories_range, start_cell: str = "A1",
                        bar_label: str = "Nilai", line_label: str = "Growth %") -> None:
    """
    Buat Combo Chart: Bar untuk nilai absolut + Line untuk % growth.
    Bar pakai primary axis (kiri), Line pakai secondary axis (kanan).
    """
    from openpyxl.chart import BarChart, LineChart, Reference, Series
    from openpyxl.chart.series import SeriesLabel
    
    # Bar chart untuk nilai
    bar = BarChart()
    bar.type = "col"
    bar.grouping = "clustered"
    bar.title = chart_title
    bar.y_axis.title = bar_label
    bar.y_axis.crosses = "autoZero"
    bar.style = 10
    bar.width = 20
    bar.height = 14
    
    bar_series = Series(bar_data_range, title=bar_label)
    bar_series.graphicalProperties.solidFill = "2E75B6"
    bar.series.append(bar_series)
    bar.set_categories(categories_range)
    
    # Line chart untuk %
    line = LineChart()
    line.y_axis.title = line_label
    line.y_axis.axId = 200
    line.y_axis.crosses = "max"
    line.y_axis.numFmt = '0.0%'
    
    line_series = Series(line_data_range, title=line_label)
    line_series.graphicalProperties.line.solidFill = "FF6B35"
    line_series.graphicalProperties.line.width = 25000  # 2pt
    line.series.append(line_series)
    
    # Gabung bar + line
    bar += line
    
    ws.add_chart(bar, start_cell)
    print(f"✅ Combo Chart '{chart_title}' dibuat di {start_cell}")
```

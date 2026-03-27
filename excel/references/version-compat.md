# Version Compatibility Engine — RIAN v4
# Formula Compatibility Matrix + Auto-Downgrade + Auto-Upgrade

## VERSION FORMULA SETS

```python
VERSION_FORMULA_SETS = {
    '2010': {
        'version_year': 2010,
        'allowed': [
            'VLOOKUP','HLOOKUP','INDEX','MATCH','OFFSET','INDIRECT',
            'SUMIF','SUMIFS','COUNTIF','COUNTIFS','AVERAGEIF','AVERAGEIFS',
            'IF','AND','OR','NOT','IFERROR',
            'LEFT','RIGHT','MID','LEN','TRIM','CLEAN','UPPER','LOWER','PROPER',
            'SUBSTITUTE','REPLACE','FIND','SEARCH','CONCATENATE','TEXT',
            'VALUE','DATE','YEAR','MONTH','DAY','TODAY','NOW','DATEDIF',
            'EDATE','EOMONTH','NETWORKDAYS','WORKDAY','WEEKDAY','WEEKNUM',
            'SUM','AVERAGE','MAX','MIN','COUNT','COUNTA','COUNTBLANK',
            'SUMPRODUCT','STDEV','VAR','CORREL','RANK','LARGE','SMALL',
            'NPV','IRR','PMT','IPMT','PPMT','FV','PV','RATE','NPER',
            'DB','SLN','DDB','SYD','CHOOSE','REPT','CHAR','CODE','LEN',
        ],
        'not_allowed': ['XLOOKUP','XMATCH','IFS','SWITCH','MAXIFS','MINIFS',
                        'TEXTJOIN','FILTER','SORT','SORTBY','UNIQUE','SEQUENCE',
                        'RANDARRAY','LAMBDA','LET','BYROW','BYCOL','MAP',
                        'REDUCE','MAKEARRAY','HSTACK','VSTACK','TOCOL','TOROW'],
        'sparklines': True,
        'tables': True,
        'dynamic_array': False,
        'power_query': False,
    },
    '2013': {
        'version_year': 2013,
        'allowed': [  # Same as 2010
            'VLOOKUP','HLOOKUP','INDEX','MATCH','OFFSET','INDIRECT',
            'SUMIF','SUMIFS','COUNTIF','COUNTIFS','AVERAGEIF','AVERAGEIFS',
            'IF','AND','OR','NOT','IFERROR','IFNA',
            'LEFT','RIGHT','MID','LEN','TRIM','CLEAN','UPPER','LOWER','PROPER',
            'SUBSTITUTE','REPLACE','FIND','SEARCH','CONCATENATE','TEXT',
            'VALUE','DATE','YEAR','MONTH','DAY','TODAY','NOW','DATEDIF',
            'EDATE','EOMONTH','NETWORKDAYS','WORKDAY','WEEKDAY','WEEKNUM',
            'SUM','AVERAGE','MAX','MIN','COUNT','COUNTA','COUNTBLANK',
            'SUMPRODUCT','STDEV','VAR','CORREL','RANK.EQ','RANK.AVG','LARGE','SMALL',
            'NPV','XNPV','IRR','XIRR','PMT','IPMT','PPMT','FV','PV','RATE','NPER',
        ],
        'not_allowed': ['XLOOKUP','XMATCH','IFS','SWITCH','MAXIFS','MINIFS',
                        'TEXTJOIN','FILTER','SORT','SORTBY','UNIQUE','SEQUENCE',
                        'LAMBDA','LET','BYROW','BYCOL'],
        'sparklines': True, 'tables': True, 'dynamic_array': False, 'power_query': False,
    },
    '2016': {
        'version_year': 2016,
        'allowed': [  # 2013 + IFS, SWITCH, MAXIFS, MINIFS, TEXTJOIN, CONCAT
            'VLOOKUP','HLOOKUP','INDEX','MATCH','OFFSET','INDIRECT',
            'SUMIF','SUMIFS','COUNTIF','COUNTIFS','AVERAGEIF','AVERAGEIFS',
            'MAXIFS','MINIFS',
            'IF','IFS','SWITCH','AND','OR','NOT','IFERROR','IFNA',
            'LEFT','RIGHT','MID','LEN','TRIM','CLEAN','UPPER','LOWER','PROPER',
            'SUBSTITUTE','REPLACE','FIND','SEARCH','CONCATENATE','CONCAT','TEXTJOIN','TEXT',
            'VALUE','DATE','YEAR','MONTH','DAY','TODAY','NOW','DATEDIF',
            'EDATE','EOMONTH','NETWORKDAYS','WORKDAY','WEEKDAY','WEEKNUM',
            'SUM','AVERAGE','MAX','MIN','COUNT','COUNTA','COUNTBLANK',
            'SUMPRODUCT','STDEV','VAR','CORREL','RANK.EQ','LARGE','SMALL',
            'NPV','XNPV','IRR','XIRR','PMT','IPMT','PPMT','FV','PV','RATE','NPER',
        ],
        'not_allowed': ['XLOOKUP','XMATCH','FILTER','SORT','SORTBY','UNIQUE',
                        'SEQUENCE','RANDARRAY','LAMBDA','LET','BYROW','BYCOL',
                        'HSTACK','VSTACK','TOCOL','TOROW'],
        'sparklines': True, 'tables': True, 'dynamic_array': False,
        'power_query': True,
    },
    '2019': {
        'version_year': 2019,
        'allowed': [  # Same as 2016 (2019 dan 2016 hampir sama untuk formula dasar)
            'VLOOKUP','HLOOKUP','INDEX','MATCH','OFFSET','INDIRECT',
            'SUMIF','SUMIFS','COUNTIF','COUNTIFS','AVERAGEIF','AVERAGEIFS',
            'MAXIFS','MINIFS',
            'IF','IFS','SWITCH','AND','OR','NOT','IFERROR','IFNA',
            'LEFT','RIGHT','MID','LEN','TRIM','CLEAN','UPPER','LOWER','PROPER',
            'SUBSTITUTE','REPLACE','FIND','SEARCH','CONCAT','TEXTJOIN','TEXT',
            'VALUE','DATE','YEAR','MONTH','DAY','TODAY','NOW','DATEDIF',
            'EDATE','EOMONTH','NETWORKDAYS','WORKDAY','WEEKDAY','WEEKNUM',
            'SUM','AVERAGE','MAX','MIN','COUNT','COUNTA','COUNTBLANK',
            'SUMPRODUCT','STDEV','VAR','CORREL','RANK.EQ','LARGE','SMALL','FREQUENCY',
            'NPV','XNPV','IRR','XIRR','PMT','IPMT','PPMT','FV','PV','RATE','NPER',
            'FORECAST.LINEAR','TREND','LINEST','NORM.DIST','NORM.INV',
        ],
        'not_allowed': ['XLOOKUP','XMATCH','FILTER','SORT','SORTBY','UNIQUE',
                        'SEQUENCE','RANDARRAY','LAMBDA','LET','BYROW','BYCOL',
                        'HSTACK','VSTACK','TOCOL','TOROW'],
        'sparklines': True, 'tables': True, 'dynamic_array': False,
        'power_query': True,
    },
    '2021': {
        'version_year': 2021,
        'allowed': [  # 2019 + XLOOKUP, XMATCH, dynamic arrays
            'VLOOKUP','HLOOKUP','INDEX','MATCH','XLOOKUP','XMATCH','OFFSET','INDIRECT',
            'SUMIF','SUMIFS','COUNTIF','COUNTIFS','AVERAGEIF','AVERAGEIFS','MAXIFS','MINIFS',
            'IF','IFS','SWITCH','AND','OR','NOT','IFERROR','IFNA',
            'LEFT','RIGHT','MID','LEN','TRIM','CLEAN','UPPER','LOWER','PROPER',
            'SUBSTITUTE','REPLACE','FIND','SEARCH','CONCAT','TEXTJOIN','TEXT',
            'VALUE','DATE','YEAR','MONTH','DAY','TODAY','NOW','DATEDIF',
            'EDATE','EOMONTH','NETWORKDAYS','WORKDAY','WEEKDAY','WEEKNUM',
            'FILTER','SORT','SORTBY','UNIQUE','SEQUENCE','RANDARRAY',
            'TOCOL','TOROW','HSTACK','VSTACK','TAKE','DROP','CHOOSECOLS','CHOOSEROWS',
            'SUM','AVERAGE','MAX','MIN','COUNT','COUNTA','SUMPRODUCT',
            'STDEV','VAR','CORREL','RANK.EQ','LARGE','SMALL','FREQUENCY',
            'NPV','XNPV','IRR','XIRR','PMT','IPMT','PPMT','FV','PV','RATE','NPER',
        ],
        'not_allowed': ['LAMBDA','LET','BYROW','BYCOL','MAP','REDUCE','MAKEARRAY',
                        'EXPAND'],
        'sparklines': True, 'tables': True, 'dynamic_array': True,
        'power_query': True,
    },
    '365': {
        'version_year': 9999,
        'allowed': ['ALL'],  # Semua formula tersedia
        'not_allowed': [],
        'sparklines': True, 'tables': True, 'dynamic_array': True,
        'power_query': True, 'lambda': True, 'let': True,
        'new_2022_2024': [
            # Array manipulation (2022)
            'TEXTSPLIT',    # =TEXTSPLIT(text, col_delim, row_delim)
            'VSTACK',       # =VSTACK(arr1, arr2) — stack vertical
            'HSTACK',       # =HSTACK(arr1, arr2) — stack horizontal
            'TOROW',        # =TOROW(array) — convert to single row
            'TOCOL',        # =TOCOL(array) — convert to single column
            'WRAPROWS',     # =WRAPROWS(array, wrap_count) — reshape ke rows
            'WRAPCOLS',     # =WRAPCOLS(array, wrap_count) — reshape ke cols
            'TAKE',         # =TAKE(array, rows, cols) — ambil N baris/kolom pertama
            'DROP',         # =DROP(array, rows, cols) — buang N baris/kolom
            'EXPAND',       # =EXPAND(array, rows, cols, pad) — expand array
            'CHOOSECOLS',   # =CHOOSECOLS(array, col1, col2) — pilih kolom tertentu
            'CHOOSEROWS',   # =CHOOSEROWS(array, row1, row2) — pilih baris tertentu
            # Higher-order functions (2022)
            'BYROW',        # =BYROW(array, LAMBDA(row, ...))
            'BYCOL',        # =BYCOL(array, LAMBDA(col, ...))
            'MAP',          # =MAP(arr, LAMBDA(x, ...))
            'REDUCE',       # =REDUCE(init, arr, LAMBDA(acc, x, ...))
            'MAKEARRAY',    # =MAKEARRAY(rows, cols, LAMBDA(r,c,...))
            'SCAN',         # =SCAN(init, arr, LAMBDA(acc, x, ...)) — running total
            'ISOMITTED',    # =ISOMITTED(arg) — cek parameter LAMBDA optional
            # Text (2022)
            'TEXTBEFORE',   # =TEXTBEFORE(text, delimiter, n)
            'TEXTAFTER',    # =TEXTAFTER(text, delimiter, n)
            # Lookup (2022)
            'XMATCH',       # Sudah ada di 2021, full feature di 365
            # New 2023-2024
            'GROUPBY',      # =GROUPBY(row_fields, values, function) — pivot-lite
            'PIVOTBY',      # =PIVOTBY(row, col, values, function) — full pivot formula
            'PERCENTOF',    # =PERCENTOF(data_subset, data_all)
            'REGEXP',       # Regex support (preview 2024)
        ],
    },
    'gsheets': {
        'version_year': 0,
        'allowed': [
            'VLOOKUP','INDEX','MATCH','XLOOKUP','IMPORTRANGE','QUERY',
            'FILTER','SORT','UNIQUE','ARRAYFORMULA',
            'SUMIF','SUMIFS','COUNTIF','COUNTIFS','AVERAGEIF','AVERAGEIFS',
            'IF','IFS','AND','OR','IFERROR','IFNA',
            'LEFT','RIGHT','MID','LEN','TRIM','CLEAN','UPPER','LOWER','PROPER',
            'SUBSTITUTE','REPLACE','REGEXMATCH','REGEXEXTRACT','JOIN','SPLIT',
            'TEXT','VALUE','DATE','TODAY','NOW','DATEDIF','EDATE','EOMONTH',
            'NETWORKDAYS','WORKDAY','SUM','AVERAGE','MAX','MIN','COUNT','COUNTA',
        ],
        'note': 'Google Sheets punya fungsi spesifik: QUERY, IMPORTRANGE, ARRAYFORMULA, REGEXMATCH',
        'not_allowed': ['LAMBDA','LET','BYROW','BYCOL','XLOOKUP_PARTIAL'],
        'sparklines': False, 'tables': False, 'dynamic_array': True,
        'power_query': False,
    },
}
```

---

## VERSION CHECK FUNCTION (STEP 2.5)

```python
def version_check():
    """
    WAJIB dijalankan di STEP 2.5 sebelum menulis formula apapun.
    Set RIAN_SESSION['formula_set'] berdasarkan versi.
    """
    version = get_session('excel_version', '2019')
    fs = VERSION_FORMULA_SETS.get(version, VERSION_FORMULA_SETS['2019'])
    
    RIAN_SESSION['formula_set'] = fs
    RIAN_SESSION['dynamic_array'] = fs.get('dynamic_array', False)
    RIAN_SESSION['sparklines_ok'] = fs.get('sparklines', False)
    RIAN_SESSION['power_query_ok'] = fs.get('power_query', False)
    RIAN_SESSION['lambda_ok'] = fs.get('lambda', False)
    
    # Print summary
    if fs['allowed'] == ['ALL']:
        print(f"✅ Version: Excel 365 — semua formula tersedia")
    else:
        print(f"✅ Version: Excel {version} — {len(fs['allowed'])} formula diizinkan")
        if fs.get('not_allowed'):
            print(f"   ⚠️ Tidak tersedia: {', '.join(fs['not_allowed'][:5])}...")
    
    return fs

def is_formula_allowed(formula_name: str) -> bool:
    """Cek apakah formula boleh dipakai di versi aktif"""
    fs = RIAN_SESSION.get('formula_set')
    if not fs:
        fs = version_check()
    if fs['allowed'] == ['ALL']:
        return True
    return formula_name.upper() in [f.upper() for f in fs['allowed']]

def check_formula_string(formula: str) -> list:
    """
    Cek formula string apakah ada formula yang tidak kompatibel.
    Return list of incompatible functions found.
    """
    import re
    fs = RIAN_SESSION.get('formula_set', {})
    not_allowed = fs.get('not_allowed', [])
    
    # Extract function names dari formula
    found_funcs = re.findall(r'([A-Z][A-Z0-9_\.]+)\s*\(', formula.upper())
    
    incompatible = [f for f in found_funcs if f in [n.upper() for n in not_allowed]]
    return incompatible

def safe_formula(formula: str, context: dict = None) -> str:
    """
    Cek dan auto-downgrade formula jika perlu.
    Return formula yang aman untuk versi aktif.
    """
    incompatible = check_formula_string(formula)
    
    if not incompatible:
        return formula
    
    # Auto-downgrade
    result = formula
    for func in incompatible:
        result = downgrade_formula(result, func, context)
    
    return result
```

---

## AUTO-DOWNGRADE ENGINE

```python
def downgrade_formula(formula: str, incompatible_func: str, context: dict = None) -> str:
    """
    Downgrade formula spesifik ke equivalent yang kompatibel.
    context = {'lookup_col': 'A', 'return_col': 'B', 'table': 'A:D', ...}
    """
    ctx = context or {}
    version_year = get_session('version_year', 2019)
    
    import re
    
    if incompatible_func == 'XLOOKUP':
        # XLOOKUP(lookup_val, lookup_arr, return_arr, if_na, match_mode, search_mode)
        # → IFERROR(INDEX(return_arr, MATCH(lookup_val, lookup_arr, 0)), if_na)
        # Pattern matching sederhana untuk common cases
        formula = re.sub(
            r'XLOOKUP\(([^,]+),\s*([^,]+),\s*([^,]+),\s*([^,]+)(?:,[^)]+)?\)',
            r'IFERROR(INDEX(\3,MATCH(\1,\2,0)),\4)',
            formula, flags=re.IGNORECASE
        )
        # Simple XLOOKUP tanpa fallback
        formula = re.sub(
            r'XLOOKUP\(([^,]+),\s*([^,]+),\s*([^)]+)\)',
            r'IFERROR(INDEX(\3,MATCH(\1,\2,0)),"-")',
            formula, flags=re.IGNORECASE
        )
    
    elif incompatible_func == 'IFS' and version_year < 2016:
        # IFS → nested IF
        # IFS(c1,v1,c2,v2,TRUE,vn) → IF(c1,v1,IF(c2,v2,vn))
        def ifs_to_nested_if(match):
            args_str = match.group(1)
            # Split by comma (naive — doesn't handle nested functions well)
            args = [a.strip() for a in args_str.split(',')]
            if len(args) < 2: return match.group(0)
            
            result = args[-1]  # Default value
            # Build nested IF from right to left
            for i in range(len(args)-2, 0, -2):
                if i > 0:
                    cond = args[i-1]
                    val  = args[i]
                    result = f'IF({cond},{val},{result})'
            return result
        
        formula = re.sub(r'IFS\((.+?)\)', ifs_to_nested_if, formula, flags=re.IGNORECASE)
    
    elif incompatible_func == 'MAXIFS' and version_year < 2016:
        # MAXIFS → array formula dengan MAX(IF())
        formula = re.sub(
            r'MAXIFS\(([^,]+),\s*([^,]+),\s*([^)]+)\)',
            r'MAX(IF(\2=\3,\1))',  # Perlu CSE (Ctrl+Shift+Enter)
            formula, flags=re.IGNORECASE
        )
    
    elif incompatible_func == 'MINIFS' and version_year < 2016:
        formula = re.sub(
            r'MINIFS\(([^,]+),\s*([^,]+),\s*([^)]+)\)',
            r'MIN(IF(\2=\3,\1))',
            formula, flags=re.IGNORECASE
        )
    
    elif incompatible_func == 'TEXTJOIN' and version_year < 2016:
        # TEXTJOIN tidak ada di <2016 — ganti dengan formula CONCATENATE atau note
        formula = f'/* TEXTJOIN tidak tersedia di Excel {get_session("excel_version")} — ' \
                  f'gunakan CONCATENATE atau formula manual */ ' + formula
    
    elif incompatible_func in ['FILTER', 'UNIQUE', 'SORT'] and not get_session('dynamic_array'):
        # Dynamic arrays tidak tersedia → pakai SUMPRODUCT/IF array
        formula = f'/* {incompatible_func} memerlukan Excel 2021/365 ' \
                  f'atau Dynamic Arrays — gunakan SUMPRODUCT sebagai alternatif */ '
    
    elif incompatible_func == 'LET':
        # LET tidak ada → langsung inline kalkulasi
        formula = re.sub(r'LET\s*\([^)]+\)', '[LET tidak tersedia — inline formula]', 
                         formula, flags=re.IGNORECASE)
    
    return formula

# Mapping formula → alternatif untuk referensi cepat
DOWNGRADE_MAP = {
    # Excel 365 → 2021
    'LAMBDA': 'Named Range workaround atau VBA function',
    'LET':    'Inline kalkulasi langsung',
    
    # Excel 2021 → 2019/2016
    'XLOOKUP':  'IFERROR(INDEX(return_range, MATCH(val, lookup_range, 0)), "-")',
    'XMATCH':   'MATCH(val, range, 0)',
    'FILTER':   'SUMPRODUCT/IF array formula (Ctrl+Shift+Enter)',
    'SORT':     'Sort manual atau pivot table',
    'UNIQUE':   'SUMPRODUCT(1/COUNTIF()) atau manual dedup',
    'SEQUENCE': 'ROW(INDIRECT("1:"&n)) - ROW() + 1',
    
    # Excel 2016 → 2013
    'IFS':      'Nested IF(cond1,val1,IF(cond2,val2,default))',
    'SWITCH':   'Nested IF atau CHOOSE(MATCH())',
    'MAXIFS':   'MAX(IF(criteria_range=criteria,value_range)) — CSE',
    'MINIFS':   'MIN(IF(criteria_range=criteria,value_range)) — CSE',
    'TEXTJOIN': 'CONCATENATE() atau &" "&',
    'CONCAT':   'CONCATENATE() atau &',
}
```

---

## FORMULA UPGRADE ENGINE (/excel upgrade)

```python
def upgrade_formulas_to_365(wb, report=True):
    """
    Upgrade semua formula lama ke equivalent modern Excel 365.
    Dipakai untuk /excel upgrade command.
    """
    upgrades = []
    
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                if not isinstance(cell.value, str) or not cell.value.startswith('='):
                    continue
                
                formula = cell.value
                upgraded = formula
                
                # VLOOKUP → XLOOKUP
                import re
                vlookup_match = re.search(
                    r'VLOOKUP\(([^,]+),\s*([^,]+),\s*(\d+),\s*(?:0|FALSE)\)',
                    formula, re.IGNORECASE
                )
                if vlookup_match:
                    val = vlookup_match.group(1)
                    table = vlookup_match.group(2)
                    col_idx = int(vlookup_match.group(3))
                    
                    # Coba rekonstruksi return array dari table + col idx
                    upgraded = re.sub(
                        r'VLOOKUP\(([^,]+),\s*([^,]+),\s*(\d+),\s*(?:0|FALSE)\)',
                        f'XLOOKUP({val},{table},... /* kolom ke-{col_idx} dari {table} */,"-")',
                        formula, flags=re.IGNORECASE
                    )
                    upgrades.append(f"VLOOKUP→XLOOKUP: {ws.title}!{cell.coordinate}")
                
                # Nested IF → IFS
                nested_if_count = formula.upper().count('IF(')
                if nested_if_count > 2:
                    upgrades.append(f"Nested IF ({nested_if_count}x) → pertimbangkan IFS: {ws.title}!{cell.coordinate}")
                
                # IFERROR(VLOOKUP) → XLOOKUP
                if 'IFERROR' in formula.upper() and 'VLOOKUP' in formula.upper():
                    upgrades.append(f"IFERROR(VLOOKUP) → XLOOKUP: {ws.title}!{cell.coordinate}")
                
                # CONCATENATE → CONCAT atau TEXTJOIN
                if 'CONCATENATE(' in formula.upper():
                    upgraded = re.sub(r'CONCATENATE\(', 'CONCAT(', 
                                      upgraded, flags=re.IGNORECASE)
                    upgrades.append(f"CONCATENATE→CONCAT: {ws.title}!{cell.coordinate}")
                
                if upgraded != formula:
                    cell.value = upgraded
    
    if report:
        print(f"\n🔄 FORMULA UPGRADE REPORT")
        print(f"   Total upgrades: {len(upgrades)}")
        for u in upgrades[:10]:
            print(f"   • {u}")
        if len(upgrades) > 10:
            print(f"   ... dan {len(upgrades)-10} lainnya")
    
    return upgrades

def upgrade_formula_single(formula: str, target_version: str = '365') -> str:
    """Upgrade satu formula ke versi target"""
    import re
    
    result = formula
    
    if target_version in ('2021', '365'):
        # IFERROR(INDEX(MATCH)) → XLOOKUP
        result = re.sub(
            r'IFERROR\(INDEX\(([^,]+),\s*MATCH\(([^,]+),\s*([^,]+),\s*0\)\),\s*([^)]+)\)',
            r'XLOOKUP(\2,\3,\1,\4)',
            result, flags=re.IGNORECASE
        )
        
        # CONCATENATE → CONCAT
        result = re.sub(r'CONCATENATE\(', 'CONCAT(', result, flags=re.IGNORECASE)
    
    return result
```

---

## LAYER 5 — VERSION COMPATIBILITY VALIDATOR

```python
def layer5_version_check(wb) -> tuple:
    """
    Scan semua formula di workbook dan verifikasi kompatibilitas.
    Return: (passed: bool, issues: list)
    """
    fs = RIAN_SESSION.get('formula_set', {})
    not_allowed = fs.get('not_allowed', [])
    version = get_session('excel_version', '2019')
    
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
                        'formula': cell.value[:60],
                        'incompatible_func': func,
                        'version': version,
                        'suggestion': DOWNGRADE_MAP.get(func, 'Gunakan alternatif yang kompatibel')
                    })
    
    passed = len(issues) == 0
    
    if passed:
        print(f"✅ L5 PASSED — Semua formula kompatibel dengan Excel {version}")
    else:
        print(f"❌ L5 FAIL — {len(issues)} formula tidak kompatibel dengan Excel {version}:")
        for i in issues[:5]:
            print(f"   [{i['sheet']}!{i['cell']}] {i['incompatible_func']} — {i['suggestion']}")
    
    return passed, issues
```

---

## GOOGLE SHEETS SUPPORT (v4 NEW)

> **Keputusan v4**: RIAN sekarang mendukung Google Sheets secara eksplisit.
> Saat user menyebut "Google Sheets" di Step 0, aktifkan GSHEETS mode.
> Output berupa file .xlsx yang dapat diimport ke Google Sheets + catatan konversi formula.

```python
GSHEETS_FORMULA_MAP = {
    # Excel → Google Sheets equivalent
    'XLOOKUP':    'XLOOKUP (tersedia di GSheets modern) atau VLOOKUP/INDEX-MATCH',
    'XLOOKUP_LEGACY': 'IFERROR(INDEX(return,MATCH(val,lookup,0)),"")',
    'IFS':        'IFS (tersedia di GSheets)',
    'MAXIFS':     'MAXIFS (tersedia di GSheets)',
    'MINIFS':     'MINIFS (tersedia di GSheets)',
    'FILTER':     'FILTER (tersedia — syntax sama)',
    'UNIQUE':     'UNIQUE (tersedia — syntax sama)',
    'SORT':       'SORT (tersedia — syntax sama)',
    'LAMBDA':     'LAMBDA (tersedia di GSheets 2022+)',
    'LET':        'LET (tersedia di GSheets 2022+)',
    'TEXTJOIN':   'JOIN() di GSheets atau TEXTJOIN (tersedia GSheets modern)',
    'PIVOT':      'Gunakan QUERY() di GSheets — lebih powerful dari PivotTable',
    'SPARKLINES': 'Tidak tersedia di GSheets — gunakan mini chart atau sparkline addon',
    'VSTACK':     'Tidak ada di GSheets — gunakan {} array literal atau QUERY UNION',
    'HSTACK':     'Tidak ada di GSheets — gunakan {} array horizontal literal',
}

GSHEETS_EXCLUSIVE = {
    # Formula yang ADA di GSheets tapi TIDAK ADA di Excel
    'QUERY':        '=QUERY(data, "SELECT A,B WHERE C>100 ORDER BY D DESC LIMIT 10")',
    'IMPORTRANGE':  '=IMPORTRANGE("spreadsheet_url","Sheet1!A1:D100")',
    'IMPORTDATA':   '=IMPORTDATA("https://url/data.csv")',
    'IMPORTHTML':   '=IMPORTHTML("url","table",1)',
    'GOOGLETRANSLATE': '=GOOGLETRANSLATE(A1,"id","en")',
    'ARRAYFORMULA': '=ARRAYFORMULA(A1:A100*B1:B100)  ← setara Ctrl+Shift+Enter Excel',
    'SPLIT':        '=SPLIT(A1,",")  ← setara TEXTSPLIT Excel',
    'REGEXMATCH':   '=REGEXMATCH(A1,"pattern")',
    'REGEXEXTRACT': '=REGEXEXTRACT(A1,"([0-9]+)")',
    'REGEXREPLACE': '=REGEXREPLACE(A1,"[^0-9]","")',
}

def gsheets_convert_notes(wb) -> list:
    """
    Scan workbook dan generate catatan konversi untuk GSheets import.
    Return list of conversion notes untuk ditampilkan ke user.
    """
    notes = []
    
    for ws in wb.worksheets:
        for row in ws.iter_rows():
            for cell in row:
                if not isinstance(cell.value, str) or not cell.value.startswith('='):
                    continue
                formula_upper = cell.value.upper()
                
                # Cek formula yang perlu perhatian khusus di GSheets
                if 'SPARKLINES' in formula_upper:
                    notes.append(f"⚠️ [{ws.title}!{cell.coordinate}] SPARKLINES tidak didukung GSheets")
                if 'VSTACK' in formula_upper or 'HSTACK' in formula_upper:
                    notes.append(f"⚠️ [{ws.title}!{cell.coordinate}] VSTACK/HSTACK belum tersedia GSheets — perlu workaround")
                if 'GETPIVOTDATA' in formula_upper:
                    notes.append(f"ℹ️ [{ws.title}!{cell.coordinate}] GETPIVOTDATA — gunakan QUERY() di GSheets")
    
    if not notes:
        notes.append("✅ File kompatibel dengan Google Sheets — tidak ada formula bermasalah")
    
    notes.append("💡 TIP GSheets: Tambahkan ARRAYFORMULA() untuk mengganti Ctrl+Shift+Enter array formulas")
    notes.append("💡 TIP GSheets: Gunakan QUERY() sebagai alternatif powerful untuk PivotTable sederhana")
    
    return notes

# Tambahkan ke VERSION_FORMULA_SETS
VERSION_FORMULA_SETS['gsheets'] = {
    'version_year': 9999,
    'allowed': ['ALL'],
    'not_allowed': ['SPARKLINES','GETPIVOTDATA'],  # Genuinely missing in GSheets
    'dynamic_array': True,
    'sparklines': False,  # GSheets tidak punya sparklines native
    'power_query': False,
    'gsheets_mode': True,
    'gsheets_exclusive': list(GSHEETS_EXCLUSIVE.keys()),
}
```

### GSheets Mode Notes untuk User
Saat GSheets mode aktif, RIAN:
1. Generate file `.xlsx` yang dioptimalkan untuk import Google Sheets
2. Hindari SPARKLINES (ganti dengan chart mini)
3. Gunakan formula yang kompatibel di kedua platform
4. Sertakan sheet `CATATAN_GSHEETS` berisi tips konversi
5. Tampilkan GSHEETS_EXCLUSIVE formulas yang tersedia jika relevan

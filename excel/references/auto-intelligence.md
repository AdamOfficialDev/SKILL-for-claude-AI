# Auto-Intelligence Reference — RIAN v4
# Domain Detection + DQ Score + Anomaly + Smart Type + Auto-Insights

## ═══════════════════════════════════════════════════
## 1. DOMAIN DETECTION ENGINE (Improved)
## ═══════════════════════════════════════════════════

```python
DOMAIN_SIGNATURES = {
    'hr': {
        'keywords': ['nama','name','nik','nip','nrp','karyawan','employee',
                     'gaji','salary','jabatan','position','departemen','department',
                     'absensi','attendance','cuti','leave','tunjangan','allowance',
                     'potongan','deduction','kontrak','pegawai','staff'],
        'weight': 1.0
    },
    'payroll': {
        'keywords': ['payroll','penggajian','slip gaji','upah','wage','lembur',
                     'overtime','pph21','bpjs','jkk','jht','jp','take home','thp',
                     'net salary','gross salary','biaya jabatan','ptkp','pkp'],
        'weight': 1.2  # Higher weight = more specific
    },
    'sales': {
        'keywords': ['penjualan','sales','revenue','omzet','pelanggan','customer',
                     'klien','client','produk','product','qty','quantity','harga',
                     'price','diskon','discount','invoice','faktur','order','pesanan',
                     'target','achievement','ach'],
        'weight': 1.0
    },
    'finance': {
        'keywords': ['debit','kredit','credit','saldo','balance','kas','cash',
                     'piutang','hutang','aset','asset','liabilitas','ekuitas',
                     'equity','laba','rugi','profit','loss','biaya','cost',
                     'expense','pendapatan','income','jurnal','gl','neraca',
                     'laporan keuangan','cashflow','ebitda','roe','roa'],
        'weight': 1.0
    },
    'inventory': {
        'keywords': ['stok','stock','persediaan','inventory','barang','item',
                     'gudang','warehouse','masuk','keluar','inflow','outflow',
                     'sku','barcode','kode barang','supplier','pemasok','reorder',
                     'minimum stock','kartu stok','bom','bill of material'],
        'weight': 1.0
    },
    'project': {
        'keywords': ['proyek','project','task','tugas','milestone','deadline',
                     'progress','status','pic','penanggung jawab','assignee',
                     'sprint','backlog','timeline','gantt','prioritas','priority',
                     'fase','phase','deliverable','wbs'],
        'weight': 1.0
    },
    'procurement': {
        'keywords': ['pengadaan','procurement','po','purchase order','rfq',
                     'tender','vendor','rekanan','hps','kontrak','spk','anggaran',
                     'budget','realisasi','pembayaran','e-procurement','tkdn'],
        'weight': 1.1
    },
    'production': {
        'keywords': ['produksi','production','output','kapasitas','capacity',
                     'mesin','machine','downtime','oee','reject','defect','scrap',
                     'kualitas','quality','shift','operator','lot','batch','wip'],
        'weight': 1.1
    },
    'academic': {
        'keywords': ['nilai','grade','score','siswa','student','mahasiswa',
                     'nim','nis','mata pelajaran','subject','semester','ipk',
                     'gpa','kelas','class','guru','dosen','teacher','rapor'],
        'weight': 1.0
    },
    'crm': {
        'keywords': ['lead','prospect','pipeline','deal','kontak','contact',
                     'follow up','stage','funnel','conversion','churn','retention',
                     'lifetime value','ltv','acquisition','opportunity','close rate'],
        'weight': 1.1
    },
    'marketing': {
        'keywords': ['campaign','iklan','ads','impressions','clicks','ctr','cpc',
                     'cpl','roas','roi','leads','conversion','channel','digital',
                     'social media','email marketing','brand','reach'],
        'weight': 1.2
    },
    'logistics': {
        'keywords': ['pengiriman','delivery','resi','tracking','ekspedisi',
                     'kurir','freight','cargo','sla','on time','otd','pod',
                     'last mile','3pl','warehouse','fulfillment'],
        'weight': 1.2
    },
    'retail': {
        'keywords': ['toko','store','kasir','pos','nota','receipt','transaksi',
                     'pelanggan','item terjual','stok toko','margin','diskon',
                     'promo','rak','display','kategori produk'],
        'weight': 1.1
    },
    'budget': {
        'keywords': ['anggaran','budget','realisasi','variance','rkap','rkat',
                     'forecast','rencana','estimasi','approval','revisi anggaran'],
        'weight': 1.1
    },
    'umkm': {
        'keywords': ['usaha','toko kecil','warung','jual beli','kas masuk',
                     'kas keluar','buku kas','untung','rugi','modal usaha'],
        'weight': 0.9
    }
}

def detect_domain(df_dict):
    """Deteksi domain dari kolom + nilai data, dengan weighted scoring"""
    all_text = []
    
    for df in df_dict.values():
        # Kolom names
        all_text.extend([str(c).lower() for c in df.columns])
        # Sample sheet name
        # Sample values (teks saja)
        for col in df.select_dtypes(include='object').columns:
            sample = df[col].dropna().head(30).astype(str).str.lower().tolist()
            all_text.extend(sample)
    
    combined = ' '.join(all_text)
    
    scores = {}
    for domain, config in DOMAIN_SIGNATURES.items():
        raw_score = sum(1 for kw in config['keywords'] if kw in combined)
        if raw_score > 0:
            scores[domain] = raw_score * config.get('weight', 1.0)
    
    if not scores:
        print("Domain: GENERAL (tidak terdeteksi spesifik)")
        return 'general'
    
    # Sort by score
    sorted_scores = sorted(scores.items(), key=lambda x: x[1], reverse=True)
    best_domain = sorted_scores[0][0]
    
    print(f"\n🧠 DOMAIN DETECTION:")
    print(f"   Domain: {best_domain.upper()} (skor: {sorted_scores[0][1]:.1f})")
    if len(sorted_scores) > 1:
        print(f"   Runner-up: {sorted_scores[1][0]} ({sorted_scores[1][1]:.1f})")
    
    return best_domain
```

---

## ═══════════════════════════════════════════════════
## 2. DATA QUALITY SCORING ENGINE
## ═══════════════════════════════════════════════════

```python
def calculate_dq_score(df_dict):
    """Hitung skor kualitas data 0-100 per sheet"""
    results = {}
    
    for sheet_name, df in df_dict.items():
        if len(df) == 0:
            results[sheet_name] = {'total_score': 0, 'grade': 'N/A', 'breakdown': {}, 'issues': []}
            continue
        
        scores = {}
        issues = []
        
        # 1. COMPLETENESS (0-30)
        null_pct = df.isnull().mean().mean() * 100
        scores['completeness'] = max(0, 30 * (1 - null_pct / 100))
        if null_pct > 20:
            null_cols = df.columns[df.isnull().mean() > 0.1].tolist()
            issues.append(f"Data kosong {null_pct:.0f}% — kolom bermasalah: {null_cols[:3]}")
        
        # 2. UNIQUENESS (0-25)
        dup_pct = df.duplicated().mean() * 100
        scores['uniqueness'] = max(0, 25 * (1 - dup_pct / 100))
        if dup_pct > 5:
            issues.append(f"{df.duplicated().sum()} baris duplikat ({dup_pct:.0f}%)")
        
        # 3. CONSISTENCY (0-20)
        consistency_penalty = 0
        for col in df.select_dtypes(include='object').columns:
            try:
                numeric_count = pd.to_numeric(df[col], errors='coerce').notna().sum()
                total_notnull = df[col].notna().sum()
                if 0 < numeric_count < total_notnull * 0.9 and numeric_count > total_notnull * 0.1:
                    consistency_penalty += 2
                    issues.append(f"Kolom '{col}' campur teks dan angka")
            except: pass
        scores['consistency'] = max(0, 20 - consistency_penalty)
        
        # 4. VALIDITY (0-15)
        validity_penalty = 0
        positive_cols = ['qty','jumlah','harga','price','stok','gaji','usia','umur','age']
        for col in df.select_dtypes(include='number').columns:
            if any(k in col.lower() for k in positive_cols):
                neg_count = (df[col] < 0).sum()
                if neg_count > 0:
                    validity_penalty += 3
                    issues.append(f"{neg_count} nilai negatif di '{col}'")
        scores['validity'] = max(0, 15 - validity_penalty)
        
        # 5. TIMELINESS (0-10)
        timeliness = 10
        for col in df.columns:
            if any(k in col.lower() for k in ['tanggal','date','tgl']):
                try:
                    dates = pd.to_datetime(df[col], errors='coerce')
                    if dates.notna().any():
                        days_old = (pd.Timestamp.now() - dates.max()).days
                        if days_old > 730: timeliness = 5
                        elif days_old > 365: timeliness = 8
                except: pass
        scores['timeliness'] = timeliness
        
        total = sum(scores.values())
        grade = 'A' if total >= 85 else 'B' if total >= 70 else 'C' if total >= 55 else 'D'
        
        results[sheet_name] = {
            'total_score': round(total, 1),
            'grade': grade,
            'breakdown': scores,
            'issues': issues
        }
    
    # Print report
    print(f"\n{'═'*50}")
    print(f"  DATA QUALITY REPORT")
    print(f"{'═'*50}")
    for sheet, r in results.items():
        emoji = {'A':'🟢','B':'🟡','C':'🟠','D':'🔴'}.get(r['grade'],'⚪')
        print(f"\n{emoji} [{sheet}] Grade: {r['grade']} | Score: {r['total_score']}/100")
        for dim, val in r['breakdown'].items():
            bar = '█' * int(val/5) + '░' * (6 - int(val/5))
            print(f"   {dim:13}: {val:4.0f} {bar}")
        for issue in r['issues'][:3]:
            print(f"   ⚠️ {issue}")
    
    return results
```

---

## ═══════════════════════════════════════════════════
## 3. ANOMALY DETECTION ENGINE
## ═══════════════════════════════════════════════════

```python
def detect_anomalies(df, sheet_name=""):
    """Deteksi outlier dan anomali statistik dengan multiple methods"""
    anomalies = []
    
    for col in df.select_dtypes(include='number').columns:
        series = df[col].dropna()
        if len(series) < 5:
            continue
        
        # Method 1: IQR (robust)
        Q1, Q3 = series.quantile(0.25), series.quantile(0.75)
        IQR = Q3 - Q1
        lower = Q1 - 1.5 * IQR
        upper = Q3 + 1.5 * IQR
        outliers_iqr = series[(series < lower) | (series > upper)]
        
        if len(outliers_iqr) > 0:
            anomalies.append({
                'sheet': sheet_name, 'column': col, 'method': 'IQR',
                'type': 'outlier', 'count': len(outliers_iqr),
                'examples': outliers_iqr.tolist()[:3],
                'normal_range': f"{lower:,.0f} — {upper:,.0f}"
            })
        
        # Method 2: Spike detection (baris berurutan)
        if len(series) > 3:
            pct_change = series.pct_change().abs()
            spikes = pct_change[pct_change > 3]  # >300% jump
            if len(spikes) > 0:
                anomalies.append({
                    'sheet': sheet_name, 'column': col, 'method': 'Spike',
                    'type': 'spike', 'count': len(spikes),
                    'description': f'Perubahan drastis (>300%) di {len(spikes)} titik'
                })
        
        # Method 3: Zero / negative anomaly
        if any(k in col.lower() for k in ['harga','gaji','price','cost','qty','stok','nilai']):
            zeros = (series == 0).sum()
            negatives = (series < 0).sum()
            if zeros > len(series) * 0.3:
                anomalies.append({
                    'sheet': sheet_name, 'column': col, 'method': 'Zero',
                    'type': 'suspicious_zero', 'count': zeros,
                    'description': f'{zeros} nilai nol — mungkin data hilang?'
                })
            if negatives > 0:
                anomalies.append({
                    'sheet': sheet_name, 'column': col, 'method': 'Negative',
                    'type': 'negative', 'count': negatives,
                    'description': f'{negatives} nilai negatif di kolom yang harusnya positif'
                })
    
    # Date anomalies
    for col in df.columns:
        if any(k in col.lower() for k in ['tanggal','date','tgl']):
            try:
                date_series = pd.to_datetime(df[col], errors='coerce')
                valid = date_series.dropna()
                if len(valid) == 0: continue
                
                future = (date_series > pd.Timestamp.now()).sum()
                very_old = (date_series < pd.Timestamp('1990-01-01')).sum()
                parse_fail = date_series.isna().sum() - df[col].isna().sum()
                
                if future > 0:
                    anomalies.append({'sheet': sheet_name, 'column': col,
                        'type': 'future_date', 'count': future,
                        'description': f'{future} tanggal di masa depan'})
                if very_old > 0:
                    anomalies.append({'sheet': sheet_name, 'column': col,
                        'type': 'old_date', 'count': very_old,
                        'description': f'{very_old} tanggal sebelum 1990 — mungkin input error'})
                if parse_fail > 0:
                    anomalies.append({'sheet': sheet_name, 'column': col,
                        'type': 'parse_fail', 'count': parse_fail,
                        'description': f'{parse_fail} nilai tidak bisa diparsing sebagai tanggal'})
            except: pass
    
    # Print summary
    if anomalies:
        print(f"\n⚠️ ANOMALY DETECTION — {sheet_name}: {len(anomalies)} temuan")
        for a in anomalies[:5]:
            print(f"   [{a['column']}] {a.get('description', a['type'])} (n={a['count']})")
    else:
        print(f"✅ No anomalies detected in [{sheet_name}]")
    
    return anomalies
```

---

## ═══════════════════════════════════════════════════
## 4. SMART COLUMN TYPE DETECTION
## ═══════════════════════════════════════════════════

```python
def smart_detect_types(df):
    """Deteksi tipe kolom lebih akurat untuk auto-formatting"""
    type_map = {}
    
    col_type_rules = {
        'identifier': ['id','kode','code','no ','nomor','nik','nim','nip','nrp','uuid'],
        'datetime':   ['tanggal','date','tgl','waktu','time','dob','lahir','masuk','keluar'],
        'currency':   ['harga','price','gaji','salary','biaya','cost','revenue','omzet',
                       'pembayaran','tagihan','nilai','jumlah uang','total','subtotal',
                       'bruto','neto','thp','fee','tarif'],
        'percentage': ['persen','percent','margin','rate','%','ach','achievement',
                       'growth','ratio','proporsi','share'],
        'quantity':   ['qty','quantity','jumlah','stok','stock','count','total item',
                       'unit','pcs','kg','liter','kuantitas'],
        'categorical':['status','kategori','category','tipe','type','jenis','class',
                       'grade','level','department','departemen','divisi','jabatan',
                       'gender','jk','pendidikan'],
        'text':       ['nama','name','deskripsi','description','keterangan','catatan',
                       'note','alamat','address','email','hp','telepon','phone'],
        'boolean':    ['aktif','active','enabled','is_','has_','flag_'],
    }
    
    for col in df.columns:
        col_lower = col.lower().strip()
        detected = None
        
        # Rule-based by column name
        for col_type, keywords in col_type_rules.items():
            if any(kw in col_lower for kw in keywords):
                detected = col_type
                break
        
        # If not matched by name, detect from values
        if not detected:
            s = df[col].dropna()
            if len(s) == 0:
                detected = 'empty'
            elif pd.api.types.is_numeric_dtype(s):
                if s.between(0, 1).all() and s.max() < 1:
                    detected = 'percentage'
                elif s.nunique() < 10 and s.max() < 100:
                    detected = 'categorical'
                else:
                    detected = 'numeric'
            elif pd.api.types.is_datetime64_any_dtype(s):
                detected = 'datetime'
            else:
                # Try date parse
                try:
                    pd.to_datetime(s.head(10), errors='raise')
                    detected = 'datetime'
                except:
                    unique_ratio = s.nunique() / len(s)
                    detected = 'categorical' if unique_ratio < 0.2 else 'text'
        
        type_map[col] = detected
    
    return type_map

def suggest_number_format(col_type):
    """Return format Excel yang tepat"""
    FORMAT_MAP = {
        'currency':    '#,##0;(#,##0);"-"',
        'percentage':  '0.0%;(0.0%);"-"',
        'quantity':    '#,##0;(#,##0);"-"',
        'numeric':     '#,##0.##;(#,##0.##);"-"',
        'datetime':    'DD/MM/YYYY',
        'identifier':  '@',
        'text':        '@',
        'categorical': '@',
        'boolean':     '"✅";"✅";"❌"',
        'growth':      '+0.0%;-0.0%;"0.0%"',
    }
    return FORMAT_MAP.get(col_type, 'General')
```

---

## ═══════════════════════════════════════════════════
## 5. AUTO-SUGGEST ENGINE v4 — 15 DOMAIN COVERAGE
## ═══════════════════════════════════════════════════

```python
def generate_suggestions(df_dict, domain, dq_results=None, anomalies=None):
    """
    Generate saran improvement yang konkret dan actionable.
    v4: Full 15-domain coverage (sebelumnya hanya 4 domain).
    """
    suggestions = []

    for sheet_name, df in df_dict.items():
        dq  = dq_results.get(sheet_name, {}) if dq_results else {}
        ec  = [c.lower() for c in df.columns]   # existing cols lowercase
        ecj = ' '.join(ec)                       # joined for 'in' checks

        # ── Saran DQ universal ─────────────────────────────
        if dq.get('total_score', 100) < 70:
            suggestions.append(
                f"[{sheet_name}] Data quality rendah "
                f"({dq.get('total_score',0):.0f}/100) — jalankan /excel bersihkan")

        num_cols = df.select_dtypes(include='number').columns.tolist()
        if num_cols:
            suggestions.append(
                f"[{sheet_name}] Apply number format IDR/% "
                f"pada {len(num_cols)} kolom numerik")

        if df.duplicated().any():
            suggestions.append(
                f"[{sheet_name}] Remove {df.duplicated().sum()} baris duplikat")

        # ── Domain-specific suggestions ────────────────────

        if domain == 'sales':
            if 'target' in ecj and 'achievement' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Achievement% = Aktual/Target")
            if not any(x in ecj for x in ['bulan','month']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Bulan dari Tanggal")
            if not any(x in ecj for x in ['kumulatif','ytd','year to date']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom YTD / Kumulatif")
            if not any(x in ecj for x in ['rank','peringkat']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Ranking salesperson/produk")
            if 'margin' not in ecj and any(x in ecj for x in ['harga jual','price','hpp']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Gross Margin % = (Jual-HPP)/Jual")

        elif domain == 'hr':
            if any(x in ecj for x in ['masuk','join','mulai']) and 'masa kerja' not in ecj and 'tenure' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Masa Kerja (tahun) dari Tgl Masuk")
            if any(x in ecj for x in ['lahir','dob']) and 'usia' not in ecj and 'age' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Usia dari Tgl Lahir")
            if 'kategori' not in ecj and 'masa kerja' in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Kategori Masa Kerja (<1/1-3/3-5/>5 thn)")
            if 'turnover' not in ecj and any(x in ecj for x in ['resign','keluar','status']):
                suggestions.append(f"[{sheet_name}] Hitung Turnover Rate per departemen/bulan")
            if any(x in ecj for x in ['gaji','salary']) and 'rasio gaji' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Rasio Gaji terhadap median departemen")

        elif domain == 'payroll':
            if 'pph21' not in ecj and 'pajak' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom PPh21 (gunakan /excel slip)")
            if 'bpjs' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom BPJS TK (2%) dan BPJS Kes (1%)")
            if 'take home' not in ecj and 'thp' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Take Home Pay = Bruto - Potongan")
            if 'slip' not in ecj:
                suggestions.append(f"[{sheet_name}] Generate slip gaji individual dengan /excel slip [NIK]")

        elif domain == 'inventory':
            if any(x in ecj for x in ['stok','stock','qty']):
                if 'status' not in ecj:
                    suggestions.append(f"[{sheet_name}] Tambahkan Status Stok (KRITIS/RENDAH/AMAN)")
                if 'reorder' not in ecj and 'minimum' not in ecj:
                    suggestions.append(f"[{sheet_name}] Tambahkan kolom Reorder Point & Minimum Stock")
                if any(x in ecj for x in ['harga','price','cost']) and 'nilai' not in ecj:
                    suggestions.append(f"[{sheet_name}] Tambahkan Nilai Stok = Qty × Harga")
            if 'perputaran' not in ecj and 'turnover' not in ecj:
                suggestions.append(f"[{sheet_name}] Hitung Inventory Turnover Ratio")
            if not any(x in ecj for x in ['masuk','keluar','inflow','outflow']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Masuk/Keluar untuk kartu stok")

        elif domain == 'finance':
            if 'debit' in ecj and 'kredit' in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan baris CHECK: Total Debit = Total Kredit")
            if 'saldo' not in ecj and 'balance' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Running Balance / Saldo Berjalan")
            if not any(x in ecj for x in ['rasio','ratio','roa','roe']):
                suggestions.append(f"[{sheet_name}] Tambahkan Financial Ratios (ROA, ROE, Current Ratio)")
            if 'cashflow' not in ecj and 'arus kas' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan sheet Arus Kas (Operasi/Investasi/Pendanaan)")

        elif domain == 'project':
            if 'progress' not in ecj and '%' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom % Progress per task/milestone")
            if not any(x in ecj for x in ['sisa','remaining','durasi']):
                suggestions.append(f"[{sheet_name}] Tambahkan Sisa Hari / Durasi tersisa ke deadline")
            if 'status' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Status (Belum Mulai/On Going/Selesai/Terlambat)")
            if not any(x in ecj for x in ['pic','penanggung','assignee']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom PIC / Penanggung Jawab")
            suggestions.append(f"[{sheet_name}] Buat Gantt Chart dengan /excel chart gantt")

        elif domain == 'procurement':
            if 'status' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Status PO (Draft/Approved/Delivered/Paid)")
            if not any(x in ecj for x in ['lead time','waktu','hari']):
                suggestions.append(f"[{sheet_name}] Tambahkan Lead Time per vendor")
            if 'realisasi' not in ecj and 'anggaran' in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Realisasi vs Anggaran + Sisa Budget")
            if 'vendor' in ecj or 'rekanan' in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Scorecard Vendor (tepat waktu, kualitas, harga)")
            suggestions.append(f"[{sheet_name}] Tambahkan kolom TKDN % jika pengadaan pemerintah")

        elif domain == 'production':
            if 'oee' not in ecj:
                suggestions.append(f"[{sheet_name}] Hitung OEE = Availability × Performance × Quality")
            if 'reject' in ecj or 'defect' in ecj:
                if 'defect rate' not in ecj and '%' not in ecj:
                    suggestions.append(f"[{sheet_name}] Tambahkan Defect Rate % = Reject/Total Output")
            if 'downtime' in ecj and 'penyebab' not in ecj and 'cause' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Penyebab Downtime untuk Pareto analysis")
            if not any(x in ecj for x in ['shift','gilir']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Shift untuk analisis per gilir kerja")
            suggestions.append(f"[{sheet_name}] Buat Pareto chart top 5 penyebab reject/downtime")

        elif domain == 'academic':
            if 'nilai' in ecj or 'score' in ecj or 'grade' in ecj:
                if not any(x in ecj for x in ['rata','mean','average']):
                    suggestions.append(f"[{sheet_name}] Tambahkan Rata-rata, Max, Min per siswa/mata pelajaran")
                if 'predikat' not in ecj and 'grade' not in ecj:
                    suggestions.append(f"[{sheet_name}] Tambahkan Predikat (A/B/C/D) dari nilai")
                if 'rank' not in ecj and 'peringkat' not in ecj:
                    suggestions.append(f"[{sheet_name}] Tambahkan Peringkat/Ranking di kelas")
            if 'kehadiran' not in ecj and 'hadir' not in ecj and 'absen' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Kehadiran (%) untuk korelasi nilai")
            suggestions.append(f"[{sheet_name}] Buat distribusi nilai (histogram) untuk laporan kelas")

        elif domain == 'crm':
            if 'stage' in ecj or 'funnel' in ecj:
                if 'konversi' not in ecj and 'conversion' not in ecj:
                    suggestions.append(f"[{sheet_name}] Hitung Conversion Rate per stage funnel")
            if not any(x in ecj for x in ['nilai','value','ltv']):
                suggestions.append(f"[{sheet_name}] Tambahkan Estimated Deal Value / LTV per prospect")
            if 'follow' not in ecj and 'next' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Next Action & Tanggal Follow Up")
            if not any(x in ecj for x in ['sumber','source','channel']):
                suggestions.append(f"[{sheet_name}] Tambahkan Lead Source untuk analisis channel terbaik")
            suggestions.append(f"[{sheet_name}] Buat Pipeline Dashboard dengan nilai per stage")

        elif domain == 'marketing':
            if 'ctr' not in ecj and 'click' in ecj and 'impression' in ecj:
                suggestions.append(f"[{sheet_name}] Hitung CTR = Clicks / Impressions × 100%")
            if 'cpc' not in ecj and 'cost' in ecj and 'click' in ecj:
                suggestions.append(f"[{sheet_name}] Hitung CPC = Total Cost / Total Clicks")
            if 'roas' not in ecj and any(x in ecj for x in ['revenue','omzet','pendapatan']) and 'spend' in ecj:
                suggestions.append(f"[{sheet_name}] Hitung ROAS = Revenue / Ad Spend")
            if 'channel' in ecj or 'platform' in ecj:
                suggestions.append(f"[{sheet_name}] Buat breakdown performance per channel/platform")
            suggestions.append(f"[{sheet_name}] Tambahkan trendline Week-over-Week / Month-over-Month")

        elif domain == 'logistics':
            if 'otd' not in ecj and not any(x in ecj for x in ['on time','tepat waktu']):
                suggestions.append(f"[{sheet_name}] Tambahkan OTD Rate = Pengiriman Tepat Waktu / Total")
            if 'sla' in ecj and 'status sla' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Status SLA (Met/Breached) vs target")
            if not any(x in ecj for x in ['biaya','ongkir','cost','freight']):
                suggestions.append(f"[{sheet_name}] Tambahkan Biaya Pengiriman per kg/rute untuk optimasi")
            suggestions.append(f"[{sheet_name}] Buat heatmap wilayah/zona berdasarkan volume & SLA")

        elif domain == 'retail':
            if not any(x in ecj for x in ['margin','laba']):
                suggestions.append(f"[{sheet_name}] Tambahkan Gross Margin % per produk/kategori")
            if 'sell through' not in ecj and 'terjual' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Sell-Through Rate = Terjual / Stok Awal")
            if not any(x in ecj for x in ['transaksi','transaction','nota']):
                suggestions.append(f"[{sheet_name}] Tambahkan Average Transaction Value (ATV)")
            suggestions.append(f"[{sheet_name}] Analisis produk fast-moving vs slow-moving (Pareto 80/20)")

        elif domain == 'budget':
            if 'realisasi' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Realisasi dan hitung Variance")
            if 'variance' not in ecj and 'selisih' not in ecj:
                suggestions.append(f"[{sheet_name}] Tambahkan Variance % = (Realisasi-Anggaran)/Anggaran")
            if not any(x in ecj for x in ['q1','q2','q3','q4','tw1','triwulan']):
                suggestions.append(f"[{sheet_name}] Pecah budget per Triwulan/Quarter untuk monitoring")
            suggestions.append(f"[{sheet_name}] Buat waterfall chart Anggaran vs Realisasi per unit")
            suggestions.append(f"[{sheet_name}] Tambahkan kolom Forecast YE (Year-End) dari run rate")

        elif domain == 'umkm':
            if not any(x in ecj for x in ['laba','profit','untung']):
                suggestions.append(f"[{sheet_name}] Tambahkan kolom Laba = Pemasukan - Pengeluaran")
            if not any(x in ecj for x in ['margin','%']):
                suggestions.append(f"[{sheet_name}] Tambahkan Profit Margin % per produk/hari")
            if not any(x in ecj for x in ['modal','bep','break even']):
                suggestions.append(f"[{sheet_name}] Hitung BEP (Break Even Point) dari data HPP dan harga jual")
            suggestions.append(f"[{sheet_name}] Buat laporan bulanan sederhana: Pemasukan, Pengeluaran, Laba")

        else:  # 'general' atau domain tidak dikenali
            suggestions.append(f"[{sheet_name}] Tambahkan sheet SUMMARY dengan KPI utama")
            suggestions.append(f"[{sheet_name}] Tambahkan row Total/Grand Total di bawah data numerik")
            suggestions.append(f"[{sheet_name}] Tambahkan conditional formatting pada kolom kritis")

    # ── Saran sheet structure ──────────────────────────
    existing_sheets = list(df_dict.keys())
    if len(existing_sheets) == 1:
        suggestions.append("Hanya 1 sheet — tambahkan SUMMARY dan DASHBOARD untuk insight lebih baik")
    if not any('dashboard' in s.lower() for s in existing_sheets):
        suggestions.append("Belum ada Dashboard — tambahkan dengan /excel dashboard")
    if not any('summary' in s.lower() or 'rekap' in s.lower() for s in existing_sheets):
        suggestions.append("Belum ada sheet ringkasan — tambahkan REKAP/SUMMARY")

    # Deduplicate & limit
    seen, final = set(), []
    for s in suggestions:
        if s not in seen:
            seen.add(s)
            final.append(s)
    return final[:15]  # v4: limit 15 (naik dari 10)
```

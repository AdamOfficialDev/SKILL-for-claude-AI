# Domain Templates Reference — RIAN v4
# 15 domain template siap pakai

## Cara Pakai
Pilih template berdasarkan domain yang terdeteksi. Setiap template mendefinisikan:
- Struktur sheet yang direkomendasikan
- Kolom standar + formula kunci
- KPI & metrics wajib
- SMART ENHANCE yang otomatis ditambahkan

---

## 1. HR (Human Resources)

### Sheet Structure
```
COVER → DATA_KARYAWAN → REKAP_ABSENSI → PAYROLL → DASHBOARD_HR
```

### Sheet: DATA_KARYAWAN
```
Kolom: NIK | Nama Lengkap | Departemen | Jabatan | Level |
       Tgl Masuk | Tgl Lahir | Status | Gender | Pendidikan |
       Email | No HP | Masa Kerja (thn) | Usia | Kategori Masa Kerja
Formula:
  Masa Kerja    = =DATEDIF([@[Tgl Masuk]],TODAY(),"Y")
  Masa Kerja Bln= =DATEDIF([@[Tgl Masuk]],TODAY(),"M")
  Usia          = =DATEDIF([@[Tgl Lahir]],TODAY(),"Y")
  Kategori MK   = =IFS([@[Masa Kerja]]<1,"< 1 Thn",[@[Masa Kerja]]<3,"1-3 Thn",[@[Masa Kerja]]<5,"3-5 Thn",TRUE,"> 5 Thn")
  Status Icon   = =IF([@Status]="Aktif","✅ Aktif",IF([@Status]="Resign","❌ Resign","⚠️ "&[@Status]))
CF: Status aktif → hijau, Resign → merah, Probation → kuning
```

### Sheet: PAYROLL / PENGGAJIAN
```
Kolom: NIK | Nama | Gaji Pokok | Tunj Jabatan | Tunj Transport |
       Tunj Makan | Lembur | Total Bruto | BPJS TK (2%) |
       BPJS Kes (1%) | PPh21 | Total Potongan | Take Home Pay
Formula:
  Total Bruto   = =[@[Gaji Pokok]]+[@[Tunj Jabatan]]+[@[Tunj Transport]]+[@[Tunj Makan]]+[@Lembur]
  BPJS TK       = =[@[Gaji Pokok]]*2%
  BPJS Kes      = =[@[Gaji Pokok]]*1%
  PPh21         = =hitung_pph21([@[Total Bruto]])   ← via Python pre-calc
  Total Potongan= =[@[BPJS TK]]+[@[BPJS Kes]]+[@PPh21]
  THP           = =[@[Total Bruto]]-[@[Total Potongan]]
Number format semua kolom Rp: #,##0;(#,##0);"-"
```

### KPI HR
- Headcount total | Karyawan baru bulan ini | Resign bulan ini | Turnover rate
- Rata-rata masa kerja | Rata-rata usia | Distribusi per departemen
- Total cost payroll | Biaya per karyawan

```python
# PPh21 Calculator
def hitung_pph21(bruto_setahun, status='TK/0'):
    ptkp = {'TK/0':54000000,'TK/1':58500000,'TK/2':63000000,'TK/3':67500000,
            'K/0':58500000,'K/1':63000000,'K/2':67500000,'K/3':72000000}
    biaya_jabatan = min(bruto_setahun * 0.05, 6000000)
    neto_setahun = bruto_setahun - biaya_jabatan
    pkp = max(0, neto_setahun - ptkp.get(status, 54000000))
    
    if pkp <= 60000000:        pph = pkp * 0.05
    elif pkp <= 250000000:     pph = 3000000 + (pkp-60000000) * 0.15
    elif pkp <= 500000000:     pph = 31500000 + (pkp-250000000) * 0.25
    elif pkp <= 5000000000:    pph = 94000000 + (pkp-500000000) * 0.30
    else:                      pph = 1444000000 + (pkp-5000000000) * 0.35
    return round(pph / 12)  # Per bulan
```

---

## 2. SALES (Penjualan)

### Sheet Structure
```
COVER → DATA_PENJUALAN → TARGET_ACTUAL → ANALISIS_PRODUK → ANALISIS_REGION → DASHBOARD_SALES
```

### Sheet: DATA_PENJUALAN
```
Kolom: No Transaksi | Tanggal | Bulan | Kuartal | Sales Person |
       Region | Kode Produk | Nama Produk | Kategori | Qty |
       Harga Satuan | Diskon (%) | Subtotal | PPN | Total | Margin %
Formula:
  Subtotal = =[@Qty]*[@[Harga Satuan]]*(1-[@[Diskon (%)]]/100)
  PPN      = =[@Subtotal]*11%
  Total    = =[@Subtotal]+[@PPN]
  Bulan    = =TEXT([@Tanggal],"MMM-YYYY")
  Kuartal  = ="Q"&INT((MONTH([@Tanggal])-1)/3+1)&"-"&YEAR([@Tanggal])
```

### Sheet: TARGET vs ACTUAL
```
Kolom: Sales Person | Jan T | Jan A | Jan Ach% | Feb T | ... | YTD T | YTD A | YTD Ach% | Status
Formula:
  Ach% = =IFERROR(Actual/Target,"-")
  Status= =IF(Ach%>=1,"✅ On Track",IF(Ach%>=0.8,"⚠️ At Risk","❌ Below"))
  YTD   = =SUM(Jan_Actual:Current_Month_Actual)
```

### KPI Sales
- Total Revenue MTD/QTD/YTD | vs Target Achievement % | YoY Growth
- Top 5 produk | Top 5 sales person | Top 5 region
- Average transaction value | Trend 12 bulan

---

## 3. FINANCE (Keuangan)

### Sheet Structure
```
COVER → ASSUMPTIONS → JURNAL → NERACA → LABA_RUGI → CASHFLOW → RASIO → DASHBOARD_FINANCE
```

### Sheet: LAPORAN LABA RUGI (P&L)
```
Revenue
  (-) HPP / COGS
= Laba Kotor (Gross Profit)  → margin: =Laba Kotor/Revenue
  (-) Beban Operasional
      Beban Gaji & Tunjangan
      Beban Marketing
      Beban Umum & Admin
      Beban Penyusutan & Amortisasi
= EBITDA                      → EBITDA margin: =EBITDA/Revenue
  (-) D&A
= EBIT / Laba Usaha
  (-) Beban Bunga / Keuangan
= EBT / Laba Sebelum Pajak
  (-) Pajak Penghasilan (22%)
= LABA BERSIH (Net Profit)    → Net margin: =Net Profit/Revenue
```

### Sheet: NERACA (Balance Sheet)
```
ASET
  Aset Lancar: Kas | Piutang Usaha | Persediaan | Aset Lancar Lain
  Aset Tidak Lancar: Aset Tetap | Akumulasi Penyusutan | Aset Lain
= TOTAL ASET

LIABILITAS & EKUITAS
  Liabilitas Lancar: Utang Dagang | Utang Pajak | Akrual
  Liabilitas JK Panjang: Utang Bank | Utang Obligasi
  Ekuitas: Modal | Laba Ditahan | Laba Tahun Berjalan
= TOTAL LIABILITAS & EKUITAS

Check: =IF(Total_Aset=Total_Liabilitas_Ekuitas,"✅ BALANCE","❌ SELISIH: "&TEXT(ABS(...),"#,##0"))
```

### Sheet: RASIO KEUANGAN
```
Likuiditas: Current Ratio | Quick Ratio | Cash Ratio
Solvabilitas: Debt-to-Equity | Debt-to-Asset | Interest Coverage
Profitabilitas: Gross Margin | EBITDA Margin | Net Margin | ROE | ROA | ROCE
Efisiensi: Asset Turnover | Inventory Turnover | Receivable Turnover | DSO | DIO
```

---

## 4. INVENTORY (Persediaan)

### Sheet Structure
```
COVER → MASTER_BARANG → KARTU_STOK → STATUS_STOK → REORDER_LIST → DASHBOARD_INVENTORY
```

### Sheet: STATUS STOK
```
Kolom: SKU | Nama | Kategori | Satuan | Stok Aktual | Stok Min |
       Stok Max | Nilai Stok | Status | Reorder Qty | Lead Time | Pemasok
Formula:
  Status    = =IF([@Aktual]<=[@Minimum],"🔴 KRITIS",IF([@Aktual]<=[@Minimum]*1.5,"🟡 RENDAH",IF([@Aktual]>=[@Maksimum],"🔵 OVER","🟢 AMAN")))
  Nilai Stok= =[@Aktual]*[@[Harga Beli]]
  Reorder   = =MAX(0,[@[Reorder Point]]-[@Aktual])
  Coverage  = =IFERROR([@Aktual]/[@[Avg Daily Usage]],"∞")
```

### KPI Inventory
- Total nilai stok | Item kritis (stok ≤ min) | Item over-stock
- Inventory turnover | Average days inventory | Nilai dead stock

---

## 5. PROJECT MANAGEMENT

### Sheet Structure
```
COVER → PROJECT_TRACKER → GANTT → MILESTONE → DASHBOARD_PROJECT
```

### Sheet: PROJECT TRACKER
```
Kolom: ID | Task | Sub-task | PIC | Departemen | Prioritas |
       Tgl Mulai | Deadline | % Progress | Status | Sisa Hari |
       Durasi Rencana | Durasi Aktual | Catatan
Formula:
  Status Auto = =IF([@[% Progress]]=100,"✅ Done",IF([@Deadline]<TODAY(),"🔴 Overdue",IF([@Deadline]-TODAY()<=3,"🟡 Due Soon","🔵 On Track")))
  Sisa Hari   = =IF([@Status]="✅ Done",0,MAX(0,[@Deadline]-TODAY()))
  Durasi      = =NETWORKDAYS([@[Tgl Mulai]],[@Deadline])-1
```

### KPI Project
- Total task | Done | In Progress | Overdue | On Track
- % Completion overall | Days to deadline (project end)
- By PIC, by Departemen, by Prioritas

---

## 6. PAYROLL DETAIL (Full Indonesia Compliance)

### Sheet: SLIP GAJI INDIVIDUAL
```
PENERIMAAN:
  Gaji Pokok
  Tunjangan Jabatan (X% × Gaji Pokok)
  Tunjangan Transport (Rp fix)
  Tunjangan Makan (Rp × Hari Kerja)
  Uang Lembur (Tarif × Jam)
  THR (1× Gaji Pokok, dibayar saat Lebaran)
  Bonus / Insentif
  TOTAL PENERIMAAN BRUTO

POTONGAN:
  JHT Karyawan (2% × Gaji Pokok)
  JP Karyawan (1% × Gaji Pokok)
  JKK — ditanggung perusahaan
  JKM — ditanggung perusahaan
  BPJS Kesehatan Karyawan (1% × Gaji)
  PPh21 (tarif progresif, lihat kalkulator)
  Cicilan Pinjaman (jika ada)
  TOTAL POTONGAN

TAKE HOME PAY = TOTAL BRUTO - TOTAL POTONGAN
```

---

## 7. LAPORAN KEUANGAN UMKM

### Sheet Structure
```
COVER → BUKU_KAS → REKAP_BULANAN → LABA_RUGI_SEDERHANA → GRAFIK
```

### Sheet: BUKU KAS HARIAN
```
Kolom: Tanggal | No Bukti | Keterangan | Kategori | Masuk | Keluar | Saldo
Formula:
  Saldo = =IF(ROW()=2,[@Masuk]-[@Keluar],INDIRECT("G"&(ROW()-1))+[@Masuk]-[@Keluar])
  Kategori CF: Dropdown: Penjualan|Pembelian|Operasional|Lainnya
```

### KPI UMKM
- Total pemasukan/pengeluaran bulan ini | Laba bersih | Margin bersih
- Arus kas | Perbandingan vs bulan lalu

---

## 8. ACADEMIC (Nilai & Akademik)

### Sheet: DAFTAR NILAI
```
Kolom: NIM/NIS | Nama | Kelas | UTS (30%) | UAS (40%) | Tugas (20%) | Kehadiran (10%) | Nilai Akhir | Grade | Bobot Mutu | Ket
Formula:
  Nilai Akhir = =[@UTS]*30%+[@UAS]*40%+[@Tugas]*20%+[@[Kehadiran%]]*10%
  Grade       = =IFS([@[Nilai Akhir]]>=85,"A",[@[Nilai Akhir]]>=75,"B",[@[Nilai Akhir]]>=65,"C",[@[Nilai Akhir]]>=55,"D",TRUE,"E")
  Bobot Mutu  = =IFS([@Grade]="A",4,[@Grade]="B",3,[@Grade]="C",2,[@Grade]="D",1,TRUE,0)
  Keterangan  = =IF([@Grade]="E","Tidak Lulus","Lulus")
  Ranking     = =RANK([@[Nilai Akhir]],Nilai_Range,0)
```

### KPI Academic
- Rata-rata nilai kelas | Nilai tertinggi/terendah
- Distribusi grade (A/B/C/D/E) | % lulus | Ranking

---

## 9. CRM / PIPELINE SALES

### Sheet: PIPELINE
```
Kolom: ID | Perusahaan | Kontak | Sumber | Produk | Est Value |
       Stage | Probability% | Exp Close | Weighted Value | Hari di Stage | Sales Owner | Last Activity | Next Action
Formula:
  Weighted Value = =[@[Est Value]]*[@Probability%]/100
  Hari di Stage  = =TODAY()-[@[Tgl Masuk]]
  Status Aging   = =IF([@[Hari di Stage]]>30,"⚠️ Stale",IF([@[Hari di Stage]]>14,"🟡 Review","🟢 Fresh"))
```

### KPI CRM
- Total pipeline value | Weighted pipeline | Win rate | Average deal size
- Conversion rate per stage | Average sales cycle | Pipeline by stage (funnel)

---

## 10. PROCUREMENT (Pengadaan)

### Sheet Structure
```
COVER → VENDOR_LIST → DATA_PO → MONITORING_PO → EVALUASI_VENDOR → DASHBOARD
```

### Sheet: DATA PURCHASE ORDER
```
Kolom: No PO | Tanggal PO | Nama Vendor | Item | Spesifikasi |
       Qty | Satuan | Harga Satuan | Total PO | Tgl Kirim Rencana |
       Tgl Terima Aktual | Status PO | Keterlambatan | Kesesuaian | Catatan
Formula:
  Keterlambatan = =MAX(0, [@[Tgl Terima Aktual]]-[@[Tgl Kirim Rencana]])
  Status        = =IF([@Status]="Terima","✅ Complete",IF([@[Tgl Kirim Rencana]]<TODAY(),"🔴 Overdue","🔵 In Process"))
  On-Time Rate  = =COUNTIF(Keterlambatan_Range,0)/COUNT(Keterlambatan_Range)
```

### Sheet: EVALUASI VENDOR
```
Kriteria penilaian (nilai 1-5):
  Kualitas Produk | Ketepatan Waktu | Harga Kompetitif |
  Responsiveness | Compliance Dokumen | Track Record
  
Skor Total    = =SUMPRODUCT(Nilai_Range, Bobot_Range)
Kategori      = =IF(Skor>=4,"⭐ Preferred",IF(Skor>=3,"✅ Qualified","⚠️ Review"))
```

---

## 11. PRODUCTION / OEE (Manufaktur)

### Sheet Structure
```
COVER → LOG_PRODUKSI → KUALITAS → OEE_CALCULATOR → DOWNTIME_LOG → DASHBOARD_OEE
```

### Sheet: OEE CALCULATOR
```
OEE = Availability × Performance × Quality

Availability  = (Planned Time - Downtime) / Planned Time
Performance   = (Actual Output × Ideal Cycle Time) / Operating Time
Quality Rate  = (Total Output - Defects) / Total Output
OEE           = Availability × Performance × Quality

Target OEE: ≥ 85% (World Class = 85%)
```

### KPI Production
- OEE harian/mingguan/bulanan | MTBF | MTTR
- Defect rate | Scrap rate | Utilization rate
- Output aktual vs target | Downtime per mesin

---

## 12. BUDGET PLANNING (Anggaran)

### Sheet Structure
```
COVER → ASSUMPTIONS → BUDGET_DETAIL → REALISASI → VARIANCE → FORECAST → DASHBOARD
```

### Sheet: BUDGET vs REALISASI
```
Kolom: Kode Akun | Nama Akun | Budget Tahunan | Jan B | Jan R | Jan Var | Jan Var% |
       Feb B | Feb R | ... | YTD Budget | YTD Realisasi | YTD Variance | YTD Var% | Full Year Forecast
Formula:
  Variance = =[@Realisasi]-[@Budget]
  Var%     = =IFERROR([@Variance]/ABS([@Budget]),0)
  Status   = =IF([@[Var%]]>0.05,"🔴 Over",IF([@[Var%]]<-0.1,"🟢 Under","🟡 On Budget"))
  Forecast = =[@[YTD Real]]+SUMIF(Remaining_Budget,Kode,Remaining)
```

---

## 13. MARKETING CAMPAIGN

### Sheet Structure
```
COVER → CAMPAIGN_LOG → CHANNEL_PERFORMANCE → LEAD_TRACKING → ROI_ANALYSIS → DASHBOARD
```

### Sheet: CAMPAIGN PERFORMANCE
```
Kolom: Campaign | Channel | Periode | Budget | Actual Spend |
       Impressions | Reach | Clicks | CTR | Leads | Conversions |
       Revenue Generated | CPC | CPL | CPA | ROAS | ROI%
Formula:
  CTR         = =IFERROR([@Clicks]/[@Impressions],0)
  CPL         = =IFERROR([@[Actual Spend]]/[@Leads],0)
  CPA         = =IFERROR([@[Actual Spend]]/[@Conversions],0)
  ROAS        = =IFERROR([@[Revenue Generated]]/[@[Actual Spend]],0)
  ROI%        = =IFERROR(([@[Revenue Generated]]-[@[Actual Spend]])/[@[Actual Spend]],0)
  Status      = =IF([@ROAS]>=3,"✅ Excellent",IF([@ROAS]>=1.5,"🟡 Good","🔴 Underperform"))
```

---

## 14. LOGISTICS & PENGIRIMAN

### Sheet Structure
```
COVER → DATA_PENGIRIMAN → TRACKING_STATUS → PERFORMA_KURIR → BIAYA_LOGISTIK → DASHBOARD
```

### Sheet: DATA PENGIRIMAN
```
Kolom: No Resi | Tanggal Kirim | Tujuan Kota | Zona | Berat (kg) |
       Jenis Layanan | Ekspedisi | Biaya Kirim | SLA (hari) |
       Tgl Estimasi Tiba | Tgl Aktual Tiba | Status | Keterlambatan |
       Rating Customer | Klaim
Formula:
  Keterlambatan = =MAX(0,[@[Tgl Aktual]]-[@[Tgl Estimasi]])
  Status        = =IF([@Status]="Terkirim",IF([@Keterlambatan]=0,"✅ On Time","⚠️ Terlambat"),IF([@[Tgl Estimasi]]<TODAY(),"🔴 Overdue","🔵 In Transit"))
  On-Time Rate  = =COUNTIF(Keterlambatan_Range,0)/COUNTA(Keterlambatan_Range)
```

---

## 15. RETAIL / TOKO

### Sheet Structure
```
COVER → DATA_TRANSAKSI → MASTER_PRODUK → STOK_HARIAN → LAPORAN_HARIAN → DASHBOARD_TOKO
```

### Sheet: DATA TRANSAKSI
```
Kolom: No Nota | Tanggal | Jam | Kasir | Produk | Qty | Harga Normal |
       Diskon | Harga Jual | Subtotal | Metode Bayar | Status
Formula:
  Harga Jual = =[@[Harga Normal]]*(1-[@[Diskon%]])
  Subtotal   = =[@Qty]*[@[Harga Jual]]
  Margin     = =IFERROR(([@[Harga Jual]]-[@HPP])/[@[Harga Jual]],0)
  Jam Block  = =CHOOSE(HOUR([@Jam])/6+1,"00-06","06-12","12-18","18-24")
```

### KPI Toko
- Omzet harian/mingguan/bulanan | Avg transaction value | Jumlah transaksi
- Produk terlaris | Jam tersibuk | Margin kotor | Stok kritis

---

## Mapping Domain → Template

```python
DOMAIN_TEMPLATE_MAP = {
    'hr':          ['DATA_KARYAWAN', 'REKAP_ABSENSI', 'PAYROLL', 'DASHBOARD_HR'],
    'payroll':     ['PAYROLL_DETAIL', 'SLIP_GAJI', 'REKAP_BULANAN'],
    'sales':       ['DATA_PENJUALAN', 'TARGET_ACTUAL', 'ANALISIS', 'DASHBOARD_SALES'],
    'finance':     ['ASSUMPTIONS', 'JURNAL', 'NERACA', 'LABA_RUGI', 'CASHFLOW', 'RASIO'],
    'inventory':   ['MASTER_BARANG', 'KARTU_STOK', 'STATUS_STOK', 'DASHBOARD_INV'],
    'project':     ['PROJECT_TRACKER', 'GANTT', 'MILESTONE', 'DASHBOARD_PROJECT'],
    'academic':    ['DAFTAR_NILAI', 'REKAP_KELAS', 'DISTRIBUSI_NILAI'],
    'crm':         ['DATA_LEADS', 'PIPELINE', 'FUNNEL', 'DASHBOARD_CRM'],
    'procurement': ['VENDOR_LIST', 'DATA_PO', 'EVALUASI_VENDOR', 'DASHBOARD_PROC'],
    'production':  ['LOG_PRODUKSI', 'KUALITAS', 'OEE', 'DOWNTIME', 'DASHBOARD_OEE'],
    'budget':      ['ASSUMPTIONS', 'BUDGET_DETAIL', 'REALISASI', 'VARIANCE', 'FORECAST'],
    'marketing':   ['CAMPAIGN_LOG', 'CHANNEL_PERF', 'ROI_ANALYSIS', 'DASHBOARD_MKT'],
    'logistics':   ['DATA_PENGIRIMAN', 'TRACKING', 'PERFORMA_KURIR', 'DASHBOARD_LOG'],
    'retail':      ['DATA_TRANSAKSI', 'MASTER_PRODUK', 'STOK_HARIAN', 'DASHBOARD_TOKO'],
    'umkm':        ['BUKU_KAS', 'REKAP_BULANAN', 'LABA_RUGI', 'GRAFIK'],
    'general':     ['DATA_UTAMA', 'SUMMARY', 'ANALISIS', 'DASHBOARD'],
}
```

---

## NEW COMMANDS IMPLEMENTATION

### /excel rekonsiliasi — Smart Reconciliation

```python
def execute_rekonsiliasi(file1_path: str, file2_path: str, key_col: str, sheet=0):
    """
    Bandingkan dua file secara komprehensif + buat laporan rekonsiliasi.
    Cocok untuk: SAP vs manual, data lama vs data baru, dll.
    """
    import pandas as pd
    
    df1 = pd.read_excel(file1_path, sheet_name=sheet)
    df2 = pd.read_excel(file2_path, sheet_name=sheet)
    
    name1 = Path(file1_path).stem
    name2 = Path(file2_path).stem
    
    # Set index
    df1_indexed = df1.set_index(key_col)
    df2_indexed = df2.set_index(key_col)
    
    # Analisis perbedaan
    only_in_1 = df1_indexed.index.difference(df2_indexed.index)
    only_in_2 = df2_indexed.index.difference(df1_indexed.index)
    in_both   = df1_indexed.index.intersection(df2_indexed.index)
    
    common_cols = [c for c in df1.columns if c in df2.columns and c != key_col]
    
    changes = []
    for key in in_both:
        for col in common_cols:
            v1 = df1_indexed.loc[key, col]
            v2 = df2_indexed.loc[key, col]
            if str(v1).strip() != str(v2).strip():
                diff = None
                try:
                    diff = float(v2) - float(v1)
                except: pass
                changes.append({
                    key_col: key, 'Kolom': col,
                    f'Nilai ({name1})': v1,
                    f'Nilai ({name2})': v2,
                    'Selisih': diff,
                    'Status': 'BERUBAH'
                })
    
    # Buat workbook rekonsiliasi
    recon_wb = Workbook()
    recon_wb.remove(recon_wb.active)
    
    # Sheet RINGKASAN
    rs = recon_wb.create_sheet("RINGKASAN")
    rs.sheet_properties.tabColor = "E67E22"
    rs['A1'] = f"LAPORAN REKONSILIASI: {name1} vs {name2}"
    rs['A1'].font = Font(size=14, bold=True, color="1F4E79", name="Calibri")
    
    summary_rows = [
        ['Metrik', 'Jumlah', 'Keterangan'],
        [f'Total baris ({name1})', len(df1), ''],
        [f'Total baris ({name2})', len(df2), ''],
        ['Hanya di file 1', len(only_in_1), 'Hilang dari file 2'],
        ['Hanya di file 2', len(only_in_2), 'Baru di file 2'],
        ['Sama persis', len(in_both) - len(set(c[key_col] for c in changes)), '✅ Match'],
        ['Berbeda nilai', len(set(c[key_col] for c in changes)), '⚠️ Perlu review'],
        ['Total perbedaan field', len(changes), ''],
    ]
    
    for r_idx, row in enumerate(summary_rows, 3):
        for c_idx, val in enumerate(row, 1):
            cell = rs.cell(r_idx, c_idx, val)
            if r_idx == 3:
                cell.font = Font(bold=True, color="FFFFFF", name="Calibri")
                cell.fill = PatternFill("solid", fgColor="1F4E79")
    
    # Sheet PERBEDAAN DETAIL
    if changes:
        diff_ws = recon_wb.create_sheet("PERBEDAAN_DETAIL")
        diff_ws.sheet_properties.tabColor = "E74C3C"
        from references_python_patterns import df_to_sheet
        df_to_sheet(diff_ws, pd.DataFrame(changes), table_name="Perbedaan")
        
        # Highlight selisih besar
        diff_df = pd.DataFrame(changes)
        if 'Selisih' in diff_df.columns:
            selisih_col = diff_df.columns.get_loc('Selisih') + 1
            cell_range = f"{get_column_letter(selisih_col)}2:{get_column_letter(selisih_col)}{len(changes)+1}"
            from references_python_patterns import highlight_negatives
            highlight_negatives(diff_ws, cell_range)
    
    # Sheet HANYA DI FILE 1
    if len(only_in_1) > 0:
        ws1 = recon_wb.create_sheet(f"HANYA_{name1[:10]}")
        ws1.sheet_properties.tabColor = "FF6B6B"
        from references_python_patterns import df_to_sheet
        df_to_sheet(ws1, df1_indexed.loc[only_in_1].reset_index(), table_name="HanyaFile1")
    
    # Sheet HANYA DI FILE 2
    if len(only_in_2) > 0:
        ws2 = recon_wb.create_sheet(f"HANYA_{name2[:10]}")
        ws2.sheet_properties.tabColor = "70AD47"
        df_to_sheet(ws2, df2_indexed.loc[only_in_2].reset_index(), table_name="HanyaFile2")
    
    output_name = f"RIAN_Rekonsiliasi_{pd.Timestamp.now().strftime('%Y%m%d')}.xlsx"
    save_output(recon_wb, output_name)
    
    print(f"\n📊 REKONSILIASI SELESAI:")
    print(f"   Match      : {len(in_both) - len(set(c[key_col] for c in changes)):,} baris")
    print(f"   Berbeda    : {len(set(c[key_col] for c in changes)):,} baris")
    print(f"   Hanya {name1}: {len(only_in_1):,} baris")
    print(f"   Hanya {name2}: {len(only_in_2):,} baris")
    
    return output_name

### /excel slip [NIK] — Slip Gaji Generator

def generate_slip_gaji(payroll_df, output_per_file=False):
    """
    Generate slip gaji untuk semua karyawan dari sheet payroll.
    Satu sheet per karyawan atau semua dalam 1 workbook.
    """
    nik_col  = next((c for c in payroll_df.columns if 'nik' in c.lower()), None)
    nama_col = next((c for c in payroll_df.columns if 'nama' in c.lower()), None)
    
    if not nik_col:
        raise ValueError("Kolom NIK tidak ditemukan di data payroll")
    
    slip_wb = Workbook()
    slip_wb.remove(slip_wb.active)
    
    PENERIMAAN_COLS = ['gaji pokok','tunjangan jabatan','tunjangan transport',
                        'tunjangan makan','uang lembur','bonus','thr','insentif']
    POTONGAN_COLS  = ['bpjs tk','bpjs kes','pph21','cicilan pinjaman','potongan lain']
    
    for _, row in payroll_df.iterrows():
        nik  = str(row.get(nik_col, 'UNKNOWN'))
        nama = str(row.get(nama_col, 'Karyawan')) if nama_col else 'Karyawan'
        
        ws = slip_wb.create_sheet(f"Slip_{nik}"[:31])
        ws.sheet_view.showGridLines = False
        
        # Header slip
        ws.merge_cells('A1:D1')
        ws['A1'] = "SLIP GAJI"
        ws['A1'].font = Font(size=16, bold=True, color="1F4E79", name="Calibri")
        ws['A1'].alignment = Alignment(horizontal='center', vertical='center')
        ws.row_dimensions[1].height = 35
        
        ws['A2'] = f"Periode: {pd.Timestamp.now().strftime('%B %Y')}"
        ws['B2'] = f"NIK: {nik}"
        ws['C2'] = f"Nama: {nama}"
        
        for cell in [ws['A2'], ws['B2'], ws['C2']]:
            cell.font = Font(size=10, name="Calibri")
        
        ws.row_dimensions[2].height = 18
        
        # Divider
        for c in range(1, 5):
            ws.cell(3, c).fill = PatternFill("solid", fgColor="1F4E79")
        ws.row_dimensions[3].height = 3
        
        # Section Penerimaan
        ws.cell(4, 1, "PENERIMAAN").font = Font(bold=True, name="Calibri", color="1F4E79")
        total_penerimaan = 0
        r = 5
        for col_name in payroll_df.columns:
            if any(k in col_name.lower() for k in PENERIMAAN_COLS):
                val = row.get(col_name, 0) or 0
                if val and val > 0:
                    ws.cell(r, 1, col_name.title()).font = Font(name="Calibri", size=10)
                    c = ws.cell(r, 3, val)
                    c.number_format = '#,##0;(#,##0);"-"'
                    c.font = Font(name="Calibri", size=10)
                    c.alignment = Alignment(horizontal='right')
                    total_penerimaan += float(val)
                    r += 1
        
        ws.cell(r, 1, "TOTAL PENERIMAAN").font = Font(bold=True, name="Calibri")
        tc = ws.cell(r, 3, total_penerimaan)
        tc.number_format = '#,##0'
        tc.font = Font(bold=True, name="Calibri")
        tc.fill = PatternFill("solid", fgColor="FFF2CC")
        r += 2
        
        # Section Potongan
        ws.cell(r, 1, "POTONGAN").font = Font(bold=True, name="Calibri", color="C0392B")
        total_potongan = 0
        r += 1
        for col_name in payroll_df.columns:
            if any(k in col_name.lower() for k in POTONGAN_COLS):
                val = row.get(col_name, 0) or 0
                if val and val > 0:
                    ws.cell(r, 1, col_name.title()).font = Font(name="Calibri", size=10)
                    c = ws.cell(r, 3, val)
                    c.number_format = '#,##0'
                    c.font = Font(name="Calibri", size=10)
                    c.alignment = Alignment(horizontal='right')
                    total_potongan += float(val)
                    r += 1
        
        ws.cell(r, 1, "TOTAL POTONGAN").font = Font(bold=True, name="Calibri", color="C0392B")
        tp = ws.cell(r, 3, total_potongan)
        tp.number_format = '#,##0'
        tp.font = Font(bold=True, color="C0392B", name="Calibri")
        tp.fill = PatternFill("solid", fgColor="FFCCCC")
        r += 2
        
        # TAKE HOME PAY
        thp = total_penerimaan - total_potongan
        ws.merge_cells(start_row=r, end_row=r, start_column=1, end_column=2)
        ws.cell(r, 1, "TAKE HOME PAY").font = Font(bold=True, size=12, color="FFFFFF", name="Calibri")
        ws.cell(r, 1).fill = PatternFill("solid", fgColor="1F4E79")
        ws.cell(r, 1).alignment = Alignment(horizontal='center', vertical='center')
        thp_c = ws.cell(r, 3, thp)
        thp_c.number_format = '#,##0'
        thp_c.font = Font(bold=True, size=12, color="FFFFFF", name="Calibri")
        thp_c.fill = PatternFill("solid", fgColor="1F4E79")
        thp_c.alignment = Alignment(horizontal='right', vertical='center')
        ws.row_dimensions[r].height = 28
        r += 2
        
        # Tanda tangan
        ws.cell(r, 1, "Karyawan,").font = Font(size=9, name="Calibri")
        ws.cell(r, 3, "Mengetahui,").font = Font(size=9, name="Calibri")
        ws.row_dimensions[r+4].height = 1
        ws.cell(r+5, 1, f"({nama})").font = Font(size=9, name="Calibri")
        ws.cell(r+5, 3, "(HRD / Payroll)").font = Font(size=9, name="Calibri")
        
        # Column widths
        ws.column_dimensions['A'].width = 30
        ws.column_dimensions['B'].width = 15
        ws.column_dimensions['C'].width = 18
        ws.column_dimensions['D'].width = 10
        
        # Print setup A5
        ws.page_setup.paperSize = 11  # A5
        ws.page_setup.orientation = 'portrait'
        ws.print_area = f"A1:D{r+6}"
    
    output_name = f"RIAN_SlipGaji_{pd.Timestamp.now().strftime('%Y%m')}.xlsx"
    save_output(slip_wb, output_name)
    print(f"✅ {len(payroll_df)} slip gaji dibuat: {output_name}")
    return output_name

### /excel sensitivity — Sensitivity Table Builder

def build_sensitivity_table(wb, base_val: float, 
                              var1_range: list, var2_range: list,
                              formula_template: str,
                              title: str = "Sensitivity Analysis"):
    """
    Buat 2-dimensi sensitivity table (What-If Analysis).
    var1_range = [0.05, 0.10, 0.15, 0.20, 0.25]  # Growth rates
    var2_range = [0.15, 0.20, 0.25, 0.30, 0.35]  # Margins
    formula_template = "base * (1 + v1) * v2"
    """
    ws = wb.create_sheet("SENSITIVITY")
    ws.sheet_properties.tabColor = "8E44AD"
    ws.sheet_view.showGridLines = False
    
    # Title
    ws.merge_cells(f'A1:{get_column_letter(len(var2_range)+1)}1')
    ws['A1'] = title
    ws['A1'].font = Font(size=14, bold=True, color="1F4E79", name="Calibri")
    ws['A1'].alignment = Alignment(horizontal='center', vertical='center')
    ws.row_dimensions[1].height = 35
    
    # Labels
    ws['A3'] = "Var 1 \\ Var 2"
    ws['A3'].font = Font(bold=True, name="Calibri")
    ws['A3'].fill = PatternFill("solid", fgColor="1F4E79")
    ws['A3'].font = Font(bold=True, color="FFFFFF", name="Calibri")
    
    for c_idx, v2 in enumerate(var2_range, 2):
        cell = ws.cell(3, c_idx, v2)
        cell.number_format = '0.0%'
        cell.font = Font(bold=True, color="FFFFFF", name="Calibri")
        cell.fill = PatternFill("solid", fgColor="2E75B6")
        cell.alignment = Alignment(horizontal='center')
    
    # Data
    for r_idx, v1 in enumerate(var1_range, 4):
        row_label = ws.cell(r_idx, 1, v1)
        row_label.number_format = '0.0%'
        row_label.font = Font(bold=True, color="FFFFFF", name="Calibri")
        row_label.fill = PatternFill("solid", fgColor="2E75B6")
        row_label.alignment = Alignment(horizontal='center')
        
        for c_idx, v2 in enumerate(var2_range, 2):
            try:
                result = eval(formula_template.replace('v1', str(v1))
                                              .replace('v2', str(v2))
                                              .replace('base', str(base_val)))
            except:
                result = 0
            
            cell = ws.cell(r_idx, c_idx, result)
            cell.number_format = '#,##0;(#,##0);"-"'
            cell.alignment = Alignment(horizontal='right')
            
            # Color scale
            if result >= base_val * 1.2:
                cell.fill = PatternFill("solid", fgColor="E2EFDA")
                cell.font = Font(color="375623", name="Calibri")
            elif result <= base_val * 0.8:
                cell.fill = PatternFill("solid", fgColor="FFCCCC")
                cell.font = Font(color="9C0006", name="Calibri")
            else:
                cell.fill = PatternFill("solid", fgColor="FFEB9C")
                cell.font = Font(name="Calibri")
    
    # Auto-fit
    auto_fit_columns(ws)
    ws.freeze_panes = 'B4'
    
    return ws
```

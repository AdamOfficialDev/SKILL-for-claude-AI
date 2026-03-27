# VBA Patterns Reference — RIAN v4
# Macro, UserForm, Event Handler, xlwings Python Integration

> Gunakan file ini saat: user minta "buat macro", "VBA", "xlsm", "automasi Excel",
> atau saat /excel buat menghasilkan file .xlsm dengan automasi.

---

## ═══════════════════════════════════════════════════
## 1. STRUKTUR DASAR MACRO VBA
## ═══════════════════════════════════════════════════

### Standard Module Template
```vba
Option Explicit  ' SELALU pakai ini — paksa deklarasi variabel

' ────────────────────────────────────────
' Modul: modRIAN_Main
' Deskripsi: Entry point macro RIAN-generated
' Dibuat: [tanggal] | RIAN v4
' ────────────────────────────────────────

Sub Auto_Open()
    ' Berjalan otomatis saat file dibuka
    Call InitializeWorkbook
    MsgBox "Selamat datang! Workbook siap digunakan.", vbInformation, "RIAN v4"
End Sub

Sub Auto_Close()
    ' Berjalan saat file ditutup
    Call SaveChangeLog
End Sub

Sub InitializeWorkbook()
    ' Setup awal workbook
    Application.ScreenUpdating = False
    Application.Calculation = xlCalculationAutomatic
    
    ' Navigasi ke sheet utama
    ThisWorkbook.Sheets("DATA").Activate
    Range("A1").Select
    
    Application.ScreenUpdating = True
End Sub
```

### Error Handling Standard
```vba
Sub SafeMacro()
    On Error GoTo ErrorHandler
    
    ' Matikan screen flicker
    Application.ScreenUpdating = False
    Application.EnableEvents = False
    Application.Calculation = xlCalculationManual
    
    ' ── KODE UTAMA DI SINI ──────────────
    
    
    ' ── SELESAI ─────────────────────────
    GoTo Cleanup
    
ErrorHandler:
    MsgBox "Error " & Err.Number & ": " & Err.Description, _
           vbCritical, "RIAN v4 — Error"
    
Cleanup:
    Application.ScreenUpdating = True
    Application.EnableEvents = True
    Application.Calculation = xlCalculationAutomatic
End Sub
```

---

## ═══════════════════════════════════════════════════
## 2. POLA LOOP & ITERASI
## ═══════════════════════════════════════════════════

```vba
' Loop baris terakhir (dinamis)
Sub LoopDynamicRows()
    Dim ws As Worksheet
    Dim lastRow As Long, i As Long
    
    Set ws = ThisWorkbook.Sheets("DATA")
    lastRow = ws.Cells(ws.Rows.Count, "A").End(xlUp).Row
    
    For i = 2 To lastRow  ' mulai dari baris 2 (skip header)
        Dim cellVal As Variant
        cellVal = ws.Cells(i, 1).Value
        
        If cellVal = "" Then GoTo NextIteration
        
        ' Proses baris...
        ws.Cells(i, 5).Value = ws.Cells(i, 3).Value * ws.Cells(i, 4).Value
        
NextIteration:
    Next i
End Sub

' Loop semua sheet
Sub LoopAllSheets()
    Dim ws As Worksheet
    For Each ws In ThisWorkbook.Worksheets
        If ws.Name <> "COVER" And ws.Name <> "DASHBOARD" Then
            Debug.Print "Processing: " & ws.Name
            ' Proses tiap sheet...
        End If
    Next ws
End Sub

' Loop dengan progress bar
Sub LoopWithProgress()
    Dim total As Long, i As Long
    total = 1000
    
    For i = 1 To total
        ' Update status bar setiap 50 baris
        If i Mod 50 = 0 Then
            Application.StatusBar = "Memproses baris " & i & " dari " & total & "..."
            DoEvents  ' Biar UI tidak freeze
        End If
        ' Proses...
    Next i
    
    Application.StatusBar = False  ' Reset status bar
End Sub
```

---

## ═══════════════════════════════════════════════════
## 3. MANIPULASI SHEET & RANGE
## ═══════════════════════════════════════════════════

```vba
' Copy sheet ke workbook baru
Sub ExportSheetToNewFile()
    Dim wsSource As Worksheet
    Dim newWb As Workbook
    Dim savePath As String
    
    Set wsSource = ThisWorkbook.Sheets("LAPORAN")
    savePath = ThisWorkbook.Path & "\Export_" & Format(Now, "YYYYMMDD_HHNN") & ".xlsx"
    
    wsSource.Copy
    Set newWb = ActiveWorkbook
    newWb.SaveAs savePath, xlOpenXMLWorkbook
    newWb.Close False
    
    MsgBox "File berhasil disimpan: " & savePath, vbInformation
End Sub

' Format range sebagai tabel
Sub FormatAsTable()
    Dim ws As Worksheet
    Dim tbl As ListObject
    Dim dataRange As Range
    
    Set ws = ActiveSheet
    Set dataRange = ws.Range("A1").CurrentRegion
    
    ' Hapus tabel lama jika ada
    On Error Resume Next
    ws.ListObjects("TabelData").Delete
    On Error GoTo 0
    
    ' Buat tabel baru
    Set tbl = ws.ListObjects.Add(xlSrcRange, dataRange, , xlYes)
    tbl.Name = "TabelData"
    tbl.TableStyle = "TableStyleMedium2"  ' Biru standar
End Sub

' Clear range dengan aman (tanpa hapus formula di luar range)
Sub SafeClearRange(ws As Worksheet, startRow As Long, endRow As Long, _
                   startCol As Long, endCol As Long)
    If endRow >= startRow And endCol >= startCol Then
        ws.Range(ws.Cells(startRow, startCol), ws.Cells(endRow, endCol)).ClearContents
    End If
End Sub

' Find last row / last col yang benar-benar terisi
Function GetLastRow(ws As Worksheet, Optional col As Long = 1) As Long
    GetLastRow = ws.Cells(ws.Rows.Count, col).End(xlUp).Row
End Function

Function GetLastCol(ws As Worksheet, Optional row As Long = 1) As Long
    GetLastCol = ws.Cells(row, ws.Columns.Count).End(xlToLeft).Column
End Function
```

---

## ═══════════════════════════════════════════════════
## 4. EVENT HANDLERS
## ═══════════════════════════════════════════════════

```vba
' Di ThisWorkbook module:
Private Sub Workbook_Open()
    ' Auto-run saat buka
    Application.Caption = "RIAN v4 Workbook — " & ThisWorkbook.Name
    Sheets("COVER").Activate
End Sub

Private Sub Workbook_BeforeSave(ByVal SaveAsUI As Boolean, Cancel As Boolean)
    ' Update timestamp sebelum save
    Dim wsLog As Worksheet
    On Error Resume Next
    Set wsLog = Sheets("CHANGE_LOG")
    On Error GoTo 0
    
    If Not wsLog Is Nothing Then
        Dim nextRow As Long
        nextRow = GetLastRow(wsLog) + 1
        wsLog.Cells(nextRow, 1).Value = Now
        wsLog.Cells(nextRow, 2).Value = Environ("USERNAME")
        wsLog.Cells(nextRow, 3).Value = "File disimpan"
    End If
End Sub

' Di Sheet module (klik kanan sheet tab → View Code):
Private Sub Worksheet_Change(ByVal Target As Range)
    ' Validasi input saat user ubah sel
    If Not Intersect(Target, Range("C:C")) Is Nothing Then
        Dim cell As Range
        For Each cell In Intersect(Target, Range("C:C"))
            If IsNumeric(cell.Value) Then
                If cell.Value < 0 Then
                    MsgBox "Nilai tidak boleh negatif di kolom C!", vbExclamation
                    Application.Undo
                End If
            End If
        Next cell
    End If
End Sub

Private Sub Worksheet_SelectionChange(ByVal Target As Range)
    ' Highlight baris yang dipilih
    Cells.Interior.ColorIndex = xlNone  ' Reset semua
    If Target.Row > 1 Then  ' Skip header
        Target.EntireRow.Interior.Color = RGB(235, 243, 255)  ' Light blue
    End If
End Sub
```

---

## ═══════════════════════════════════════════════════
## 5. USERFORM TEMPLATES
## ═══════════════════════════════════════════════════

```vba
' Form Input Data Sederhana
' Tambahkan UserForm bernama frmInputData ke project VBA

Private Sub btnSimpan_Click()
    ' Validasi input
    If txtNama.Value = "" Then
        MsgBox "Nama tidak boleh kosong!", vbExclamation
        txtNama.SetFocus
        Exit Sub
    End If
    
    If Not IsNumeric(txtJumlah.Value) Then
        MsgBox "Jumlah harus berupa angka!", vbExclamation
        txtJumlah.SetFocus
        Exit Sub
    End If
    
    ' Simpan ke sheet
    Dim ws As Worksheet
    Set ws = ThisWorkbook.Sheets("DATA")
    Dim nextRow As Long
    nextRow = GetLastRow(ws) + 1
    
    ws.Cells(nextRow, 1).Value = nextRow - 1          ' No
    ws.Cells(nextRow, 2).Value = txtNama.Value         ' Nama
    ws.Cells(nextRow, 3).Value = cboKategori.Value     ' Kategori
    ws.Cells(nextRow, 4).Value = CDbl(txtJumlah.Value) ' Jumlah
    ws.Cells(nextRow, 5).Value = Now                   ' Timestamp
    
    MsgBox "Data berhasil disimpan!", vbInformation
    Call ResetForm
End Sub

Private Sub ResetForm()
    txtNama.Value = ""
    txtJumlah.Value = ""
    cboKategori.ListIndex = 0
    txtNama.SetFocus
End Sub

Private Sub btnBatal_Click()
    Unload Me
End Sub

' Cara panggil UserForm dari macro:
Sub ShowInputForm()
    frmInputData.Show vbModeless  ' vbModeless = form tidak block Excel
End Sub
```

---

## ═══════════════════════════════════════════════════
## 6. PROTEKSI & KEAMANAN
## ═══════════════════════════════════════════════════

```vba
' Proteksi sheet dengan lock formula, unlock input
Sub ProtectWithUnlockInput(ws As Worksheet, Optional password As String = "")
    ws.Unprotect password
    
    ' Unlock semua dulu
    ws.Cells.Locked = False
    
    ' Lock hanya sel yang berisi formula
    Dim cell As Range
    For Each cell In ws.UsedRange
        If cell.HasFormula Then
            cell.Locked = True
        End If
    Next cell
    
    ' Lock sel header (baris 1)
    ws.Rows(1).Locked = True
    
    ' Protect dengan allow tertentu
    ws.Protect Password:=password, _
              DrawingObjects:=True, _
              Contents:=True, _
              AllowInsertingRows:=True, _
              AllowDeletingRows:=True, _
              AllowFiltering:=True, _
              AllowSorting:=True
    
    Debug.Print ws.Name & " dilindungi (formula locked, input cells unlocked)"
End Sub

' Protect semua sheet sekaligus
Sub ProtectAllSheets(Optional password As String = "RIAN2024")
    Dim ws As Worksheet
    For Each ws In ThisWorkbook.Worksheets
        If ws.Name <> "COVER" And ws.Name <> "PETUNJUK" Then
            Call ProtectWithUnlockInput(ws, password)
        End If
    Next ws
    MsgBox "Semua sheet telah diproteksi.", vbInformation
End Sub
```

---

## ═══════════════════════════════════════════════════
## 7. GENERATE VBA VIA PYTHON (xlwings)
## ═══════════════════════════════════════════════════

```python
# Cara inject VBA code ke file .xlsm menggunakan xlwings
# Pastikan xlwings terinstall: pip install xlwings --break-system-packages

import xlwings as xw

def inject_vba_module(filepath: str, module_name: str, vba_code: str) -> str:
    """
    Inject VBA module ke file Excel.
    PENTING: Hanya berjalan di Windows/Mac dengan Excel terinstall.
    Di Linux/container, gunakan pendekatan alternatif.
    """
    try:
        app = xw.App(visible=False)
        wb = app.books.open(filepath)
        
        # Tambah module baru
        vba_module = wb.api.VBProject.VBComponents.Add(1)  # 1 = vbext_ct_StdModule
        vba_module.Name = module_name
        vba_module.CodeModule.AddFromString(vba_code)
        
        wb.save()
        wb.close()
        app.quit()
        
        print(f"✅ VBA module '{module_name}' berhasil diinjeksi ke {filepath}")
        return filepath
        
    except Exception as e:
        print(f"❌ Gagal inject VBA: {e}")
        print("   Catatan: xlwings memerlukan Microsoft Excel terinstall di sistem.")
        return filepath


def embed_vba_in_xlsm(filepath: str, macros: dict) -> str:
    """
    Embed multiple VBA macros ke file .xlsm.
    macros = {'ModuleName': 'VBA code string', ...}
    """
    # Rename .xlsx ke .xlsm jika perlu
    if filepath.endswith('.xlsx'):
        import os
        new_path = filepath.replace('.xlsx', '.xlsm')
        os.rename(filepath, new_path)
        filepath = new_path
    
    for module_name, code in macros.items():
        inject_vba_module(filepath, module_name, code)
    
    return filepath


# ── Alternatif: Embed VBA sebagai komentar instruksi ─────────────────
def add_vba_instructions_sheet(wb, vba_snippets: dict):
    """
    Jika environment tidak support xlwings, tambahkan sheet VBA_GUIDE
    berisi kode VBA yang bisa di-copy user ke VBA Editor (Alt+F11).
    """
    from openpyxl.styles import Font, PatternFill, Alignment
    
    # Hapus sheet lama jika ada
    if 'VBA_GUIDE' in [s.title for s in wb.worksheets]:
        del wb['VBA_GUIDE']
    
    ws = wb.create_sheet('VBA_GUIDE')
    ws.sheet_properties.tabColor = "808080"
    
    # Header
    ws['A1'] = 'PANDUAN VBA — RIAN v4'
    ws['A1'].font = Font(bold=True, size=14)
    ws['A2'] = 'Cara pakai: Buka VBA Editor (Alt+F11) → Insert Module → Paste kode di bawah'
    ws['A2'].font = Font(italic=True, color="666666")
    
    row = 4
    for module_name, code in vba_snippets.items():
        ws.cell(row, 1, f'=== MODULE: {module_name} ===')
        ws.cell(row, 1).font = Font(bold=True, color="1F4E79")
        row += 1
        
        for line in code.split('\n'):
            ws.cell(row, 1, line)
            ws.cell(row, 1).font = Font(name='Courier New', size=10)
            row += 1
        row += 1
    
    ws.column_dimensions['A'].width = 120
    print(f"✅ Sheet VBA_GUIDE ditambahkan — {len(vba_snippets)} module")
    return wb
```

---

## ═══════════════════════════════════════════════════
## 8. SHORTCUT VBA PATTERNS UNTUK RIAN
## ═══════════════════════════════════════════════════

| Use Case | Pattern yang Digunakan |
|----------|------------------------|
| `/excel buat [domain]` dengan automasi | `Auto_Open` + `InitializeWorkbook` |
| `/excel team` | `ProtectAllSheets` + `Workbook_BeforeSave` log |
| `/excel slip [NIK]` | Loop karyawan + `ExportSheetToNewFile` per NIK |
| Input form interaktif | `UserForm` + `Worksheet_Change` validasi |
| `/excel merge` otomatis | Loop workbooks + `Copy` sheets |
| Auto-refresh pivot | `PivotTable.RefreshTable` di `Auto_Open` |
| Print per-area | `PrintArea` setting + `PrintOut` per sheet |

> **Catatan**: Saat environment tidak support xlwings (container/Linux),
> gunakan `add_vba_instructions_sheet()` untuk embed kode VBA sebagai panduan.
> User dapat copy-paste ke VBA Editor secara manual.

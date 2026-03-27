# Session Init Reference — RIAN v4
# RIAN Profile, Konfigurasi, Smart Defaults

## SESSION STATE MANAGER

```python
# State global yang persist sepanjang conversation
RIAN_SESSION = {
    'excel_version': '2019',      # Default safe middle ground
    'version_year':  2019,        # Numeric for comparison
    'enhance_level': 2,           # 0=off, 1=minimal, 2=standard, 3=full
    'mode':          'standard',  # quick/standard/full/analysis/audit/learning/collab
    'currency':      'IDR',       # IDR/USD/SGD/EUR
    'date_format':   'DD/MM/YYYY',
    'print_size':    'A4',
    'print_orient':  'landscape',
    'theme':         'corporate_navy',
    'language':      'id',
    'initialized':   False,
    'formula_set':   None,        # Diisi oleh version_check()
}

def set_session(**kwargs):
    """Update session state"""
    RIAN_SESSION.update(kwargs)
    RIAN_SESSION['initialized'] = True

def get_session(key, default=None):
    return RIAN_SESSION.get(key, default)
```

---

## AUTO-DETECT dari PESAN USER

```python
def auto_detect_session(user_message: str) -> dict:
    """
    Coba deteksi konfigurasi dari pesan user.
    Return dict berisi apa yang berhasil dideteksi.
    """
    detected = {}
    msg = user_message.lower()
    
    # Deteksi versi Excel
    version_patterns = {
        'excel 2010': ('2010', 2010),
        'excel 2013': ('2013', 2013),
        'excel 2016': ('2016', 2016),
        'excel 2019': ('2019', 2019),
        'excel 2021': ('2021', 2021),
        'excel 365': ('365', 365),
        'office 365': ('365', 365),
        'microsoft 365': ('365', 365),
        'google sheets': ('gsheets', 0),
        'libreoffice': ('libreoffice', 0),
        'wps': ('wps', 2016),
    }
    for pattern, (ver_str, ver_year) in version_patterns.items():
        if pattern in msg:
            detected['excel_version'] = ver_str
            detected['version_year'] = ver_year
            break
    
    # Deteksi mode dari keyword
    if any(k in msg for k in ['fix', 'perbaiki', 'error', 'rusak', 'salah']):
        detected['mode'] = 'quick'
        detected['enhance_level'] = 0
    elif any(k in msg for k in ['analisis', 'analyze', 'insight', 'temukan']):
        detected['mode'] = 'analysis'
        detected['enhance_level'] = 0
    elif any(k in msg for k in ['audit', 'cek formula', 'periksa formula']):
        detected['mode'] = 'audit'
        detected['enhance_level'] = 0
    elif any(k in msg for k in ['buat', 'create', 'template baru', 'dari awal']):
        detected['mode'] = 'full'
        detected['enhance_level'] = 3
    elif any(k in msg for k in ['jelasin', 'ajarkan', 'pemula', 'explain']):
        detected['mode'] = 'learning'
        detected['enhance_level'] = 2
    elif any(k in msg for k in ['team', 'tim', 'share', 'protect', 'kunci']):
        detected['mode'] = 'collab'
        detected['enhance_level'] = 2
    
    # Deteksi enhance preference
    if any(k in msg for k in ['jangan enhance', 'tanpa enhance', 'minimal saja']):
        detected['enhance_level'] = 0
    elif any(k in msg for k in ['enhance penuh', 'full enhance', 'maksimal']):
        detected['enhance_level'] = 3
    
    # Deteksi mata uang
    if 'usd' in msg or '$' in msg or 'dollar' in msg:
        detected['currency'] = 'USD'
    elif 'sgd' in msg or 'singapore' in msg:
        detected['currency'] = 'SGD'
    
    return detected

def run_session_init(user_message: str) -> bool:
    """
    Jalankan session init. Return True jika perlu tanya user.
    """
    if RIAN_SESSION['initialized']:
        # Override dari pesan baru
        detected = auto_detect_session(user_message)
        if detected:
            set_session(**detected)
        return False
    
    # First time init
    detected = auto_detect_session(user_message)
    
    if detected:
        # Bisa auto-detect → set dan lanjut tanpa tanya
        set_session(**detected)
        print_profile_card()
        return False
    else:
        # Tidak ada info → tanya minimum
        return True  # Caller harus tanya user
```

---

## RIAN PROFILE CARD

```python
def print_profile_card():
    """Print RIAN Profile Card ke terminal"""
    v = RIAN_SESSION
    
    # Formula set info
    formula_info = get_formula_set_summary(v['excel_version'])
    
    print("\n" + "╔" + "═"*50 + "╗")
    print("║  RIAN v4 — PROFILE AKTIF" + " "*25 + "║")
    print("╠" + "═"*50 + "╣")
    print(f"║  Excel Version : {v['excel_version']:30}  ║")
    print(f"║  Mode Operasi  : {v['mode'].title():30}  ║")
    print(f"║  Enhance Level : L{v['enhance_level']} — {ENHANCE_LABELS[v['enhance_level']]:26}  ║")
    print(f"║  Mata Uang     : {v['currency']:30}  ║")
    print(f"║  Format Tanggal: {v['date_format']:30}  ║")
    print(f"║  Print Setup   : {v['print_size']} {v['print_orient']:25}  ║")
    print(f"║  Design Theme  : {v['theme']:30}  ║")
    print("╠" + "═"*50 + "╣")
    print(f"║  Formula Safe  : {formula_info[:32]}  ║")
    print("╚" + "═"*50 + "╝")
    print("  Override: 'pakai IDR' / 'jangan enhance' / 'mode audit'\n")

ENHANCE_LABELS = {
    0: 'Off (Raw)',
    1: 'Minimal (Format)',
    2: 'Standard (default)',
    3: 'Full (Intelligence)',
}

def get_formula_set_summary(version: str) -> str:
    from references_version_compat import VERSION_FORMULA_SETS
    fs = VERSION_FORMULA_SETS.get(version, VERSION_FORMULA_SETS['2019'])
    return f"{len(fs['allowed'])} formula allowed"
```

---

## SMART DEFAULT MATRIX

```python
# Default yang RIAN pakai jika tidak ada info
SMART_DEFAULTS = {
    # Jika ada file upload dengan data bisnis Indonesia → IDR
    'domain_hr':        {'currency': 'IDR', 'date_format': 'DD/MM/YYYY'},
    'domain_finance':   {'currency': 'IDR', 'date_format': 'DD/MM/YYYY'},
    'domain_sales':     {'currency': 'IDR', 'date_format': 'DD/MM/YYYY'},
    'domain_general':   {'currency': 'IDR', 'date_format': 'DD/MM/YYYY'},
    
    # Quick commands → Quick Fix mode
    'cmd_fix':          {'mode': 'quick', 'enhance_level': 0},
    'cmd_analisis':     {'mode': 'analysis', 'enhance_level': 0},
    'cmd_audit':        {'mode': 'audit', 'enhance_level': 0},
    'cmd_buat':         {'mode': 'full', 'enhance_level': 3},
    'cmd_dashboard':    {'mode': 'full', 'enhance_level': 3},
    'cmd_jelasin':      {'mode': 'learning', 'enhance_level': 2},
    'cmd_team':         {'mode': 'collab', 'enhance_level': 2},
    
    # Safe formula default (Excel 2019 — most widely used)
    'excel_version':    '2019',
    'version_year':     2019,
}

def apply_domain_defaults(domain: str):
    """Apply domain-specific defaults ke session"""
    key = f'domain_{domain}'
    defaults = SMART_DEFAULTS.get(key, {})
    if defaults:
        set_session(**defaults)

def apply_command_defaults(command: str):
    """Apply command-specific defaults"""
    # Extract command (e.g. '/excel fix' → 'fix')
    parts = command.strip('/').lower().split()
    if len(parts) >= 2:
        cmd = parts[1]
        key = f'cmd_{cmd}'
        defaults = SMART_DEFAULTS.get(key, {})
        if defaults:
            set_session(**defaults)
```

---

## SETTINGS OVERRIDE COMMAND

```python
def handle_settings_command(user_message: str):
    """
    Handle /rian settings atau inline override seperti 'pakai IDR'.
    Dipanggil di awal setiap turn untuk cek ada override tidak.
    """
    msg = user_message.lower()
    changes = {}
    
    # Version override
    for ver in ['2010', '2013', '2016', '2019', '2021', '365']:
        if f'excel {ver}' in msg or f'versi {ver}' in msg:
            changes['excel_version'] = ver
            changes['version_year'] = int(ver) if ver != '365' else 9999
    
    # Enhance override
    if any(k in msg for k in ['jangan enhance', 'no enhance', 'enhance off']):
        changes['enhance_level'] = 0
    elif 'enhance minimal' in msg or 'enhance 1' in msg:
        changes['enhance_level'] = 1
    elif 'enhance standard' in msg or 'enhance 2' in msg:
        changes['enhance_level'] = 2
    elif any(k in msg for k in ['enhance penuh', 'enhance full', 'enhance 3']):
        changes['enhance_level'] = 3
    
    # Currency override
    if 'pakai idr' in msg or 'gunakan idr' in msg:
        changes['currency'] = 'IDR'
    elif 'pakai usd' in msg or 'gunakan usd' in msg:
        changes['currency'] = 'USD'
    
    # Theme override
    for theme in ['corporate_navy', 'corporate_green', 'monochrome', 'warm_accent', 'dark_pro']:
        if theme.replace('_', ' ') in msg:
            changes['theme'] = theme
    
    # Mode override
    mode_map = {
        'mode quick': 'quick', 'quick fix': 'quick',
        'mode audit': 'audit', 'mode analysis': 'analysis',
        'mode learning': 'learning', 'mode learning': 'learning',
        'mode team': 'collab', 'mode collab': 'collab',
    }
    for trigger, mode in mode_map.items():
        if trigger in msg:
            changes['mode'] = mode
    
    if changes:
        set_session(**changes)
        print(f"✅ Settings diupdate: {changes}")
        print_profile_card()
    
    return changes
```

---

## ═══════════════════════════════════════════════════
## CHECKPOINT & RESUME SYSTEM (v4 NEW)
## ═══════════════════════════════════════════════════

> Jika session crash di tengah proses, RIAN bisa resume dari step terakhir.
> User cukup ketik `/rian resume` di sesi baru.

```python
import json, os
from pathlib import Path

CHECKPOINT_FILE = Path("/home/claude/rian_checkpoint.json")

def save_checkpoint(step: int, context: dict) -> None:
    """Simpan state ke file checkpoint setelah tiap step selesai."""
    state = {
        'step':          step,
        'step_name':     STEP_NAMES.get(step, f'Step {step}'),
        'timestamp':     pd.Timestamp.now().isoformat(),
        'session':       RIAN_SESSION.copy(),
        'context':       context,
        'work_file':     context.get('work_file'),
        'output_file':   context.get('output_file'),
        'domain':        context.get('domain'),
        'instructions':  context.get('instructions'),
    }
    CHECKPOINT_FILE.write_text(json.dumps(state, indent=2, default=str))
    print(f"💾 Checkpoint disimpan: Step {step} — {STEP_NAMES.get(step, '')}")

def load_checkpoint() -> dict | None:
    """Load checkpoint jika ada."""
    if not CHECKPOINT_FILE.exists():
        return None
    try:
        state = json.loads(CHECKPOINT_FILE.read_text())
        age_minutes = (pd.Timestamp.now() - pd.Timestamp(state['timestamp'])).seconds / 60
        if age_minutes > 120:  # Checkpoint > 2 jam = expired
            print("⚠️ Checkpoint expired (>2 jam) — mulai ulang dari awal")
            CHECKPOINT_FILE.unlink()
            return None
        return state
    except Exception as e:
        print(f"⚠️ Gagal baca checkpoint: {e}")
        return None

def clear_checkpoint() -> None:
    """Hapus checkpoint setelah delivery sukses."""
    if CHECKPOINT_FILE.exists():
        CHECKPOINT_FILE.unlink()
        print("🗑️ Checkpoint dihapus (tugas selesai)")

STEP_NAMES = {
    0: 'SESSION INIT',
    1: 'FILE INTAKE',
    2: 'RECONNAISSANCE',
    3: 'SMART PLANNING',
    4: 'EXECUTION',
    5: 'SELF-ENHANCEMENT',
    6: 'VALIDATION',
    7: 'SELF-FIX',
    8: 'DELIVERY',
}

def resume_from_checkpoint() -> dict | None:
    """
    Dipanggil saat user ketik '/rian resume'.
    Return context jika ada checkpoint valid, None jika tidak ada.
    """
    state = load_checkpoint()
    if not state:
        print("ℹ️ Tidak ada checkpoint tersimpan — mulai sesi baru.")
        return None

    last_step     = state.get('step', 0)
    last_step_name = state.get('step_name', '')
    work_file     = state.get('work_file')
    domain        = state.get('domain', 'unknown')
    instructions  = state.get('instructions', 'tidak tercatat')

    # Restore session
    global RIAN_SESSION
    RIAN_SESSION.update(state.get('session', {}))

    print(f"""
╔══════════════════════════════════════════════════╗
║  RIAN v4 — RESUME CHECKPOINT                    ║
╠══════════════════════════════════════════════════╣
║  Terputus di  : Step {last_step} — {last_step_name:<24}║
║  File         : {str(work_file or '-')[:48]:<48}║
║  Domain       : {domain:<48}║
║  Instruksi    : {str(instructions)[:48]:<48}║
╚══════════════════════════════════════════════════╝
Melanjutkan dari Step {last_step + 1}...
""")
    return state

# ── Cara pakai di Iron Protocol ──────────────────────────────────────
# Setelah tiap step berhasil, tambahkan:
#   save_checkpoint(step_num, {'work_file': work_path, 'domain': domain, ...})
#
# Di Step 8 (Delivery) setelah present_files:
#   clear_checkpoint()
#
# Trigger dari user:
#   if '/rian resume' in user_message:
#       state = resume_from_checkpoint()
#       if state: lanjutkan dari state['step'] + 1
```

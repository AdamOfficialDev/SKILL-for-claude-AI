# DAVID — Security Audit Reference

Full pattern library for Scanner B. Load this when performing deep security analysis.

---

## Table of Contents
1. [OWASP Top 10 Detailed Patterns](#owasp-top-10)
2. [Secrets Detection Patterns](#secrets-detection)
3. [Language-Specific Security Bugs](#language-specific)
4. [API Security Checklist](#api-security)
5. [Dependency Vulnerability Patterns](#dependency-vulnerabilities)

---

## OWASP Top 10

### A01 — Broken Access Control

**Patterns to detect:**
- Routes/endpoints that don't verify the requesting user owns the requested resource
- Missing role checks before privileged operations
- Direct object reference: `GET /user/{id}` with no ownership check
- Path traversal: `../../../etc/passwd` in file path parameters
- CORS misconfiguration allowing untrusted origins
- JWT/session not invalidated on logout

**Code patterns (any language):**
```
# VULNERABLE: No ownership check
def get_document(doc_id):
    return db.query("SELECT * FROM docs WHERE id = ?", doc_id)

# SECURE: Ownership verified
def get_document(doc_id, current_user_id):
    return db.query("SELECT * FROM docs WHERE id = ? AND owner_id = ?", doc_id, current_user_id)
```

---

### A02 — Cryptographic Failures

**Patterns to detect:**
- MD5 or SHA1 used for password hashing (use bcrypt/argon2/scrypt)
- SHA256 without salt for passwords
- AES-ECB mode (use GCM or CBC with random IV)
- Hardcoded encryption keys
- HTTP instead of HTTPS for sensitive data
- Random numbers from `Math.random()` used for security tokens (use `crypto.randomBytes`)
- JWT with `alg: none` accepted

**Severity escalations:**
- Plaintext passwords in DB → P0 CRITICAL
- Weak hashing (MD5) for passwords → P0 CRITICAL
- Weak hashing for non-auth data → P1 HIGH

---

### A03 — Injection

**SQL Injection patterns:**
```python
# VULNERABLE — string concatenation
query = "SELECT * FROM users WHERE name = '" + username + "'"
query = f"SELECT * FROM users WHERE name = '{username}'"
query = "SELECT * FROM users WHERE name = '%s'" % username

# SECURE — parameterized
cursor.execute("SELECT * FROM users WHERE name = ?", (username,))
cursor.execute("SELECT * FROM users WHERE name = %s", (username,))
User.objects.filter(name=username)  # ORM is safe
```

**NoSQL Injection (MongoDB):**
```javascript
// VULNERABLE
db.users.find({ username: req.body.username })
// If username = { $gt: "" }, returns all users

// SECURE
const { username } = req.body;
if (typeof username !== 'string') throw new Error('Invalid input');
db.users.find({ username: username });
```

**Command Injection:**
```python
# VULNERABLE
os.system(f"ping {user_input}")
subprocess.call(f"ls {directory}", shell=True)

# SECURE
subprocess.run(["ping", user_input], shell=False)
subprocess.run(["ls", directory], shell=False)
```

**XSS patterns:**
```javascript
// VULNERABLE
element.innerHTML = userInput;
document.write(userInput);
eval(userInput);

// SECURE
element.textContent = userInput;
element.setAttribute('data-value', encodeURIComponent(userInput));
```

**LDAP Injection, XML/XPath Injection:** Scan for any user input concatenated into LDAP filter strings or XPath queries.

---

### A04 — Insecure Design

**Patterns:**
- No rate limiting on auth endpoints (brute force risk)
- No account lockout after N failed attempts
- Password reset via predictable token (timestamp-based, sequential)
- Missing input length limits (DoS via huge payload)
- Business logic: can a user skip steps? (order without payment, etc.)
- Mass assignment: `Object.assign(dbRecord, req.body)` — exposes all fields

---

### A05 — Security Misconfiguration

**Patterns:**
```javascript
// DANGEROUS — CORS allows everything
app.use(cors({ origin: '*' }));
res.header('Access-Control-Allow-Origin', '*');

// DANGEROUS — Verbose errors in production
app.use((err, req, res, next) => {
  res.json({ error: err.stack }); // Exposes internals
});

// DANGEROUS — Debug mode
DEBUG=True  # in Django settings
NODE_ENV=development  # in production

// DANGEROUS — Default credentials
const DB_PASS = 'root';
const ADMIN_PASS = 'admin123';
```

---

### A06 — Vulnerable Components

**Patterns to flag:**
- `npm install` with `^` or `~` pinning (minor updates may introduce CVE)
- Requirements with no version pinning (`requests` instead of `requests==2.31.0`)
- Dependencies with known CVE patterns (check against NIST NVD patterns)
- Use of `eval()`, `exec()`, `pickle.loads()` with untrusted data
- Deserializing untrusted data with `yaml.load()` (use `yaml.safe_load()`)

---

### A07 — Authentication Failures

**Patterns:**
```python
# WEAK — MD5 password hashing
import hashlib
hashed = hashlib.md5(password.encode()).hexdigest()

# WEAK — No iteration count
hashed = hashlib.sha256(password.encode()).hexdigest()

# SECURE
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
```

```javascript
// DANGEROUS — Session not invalidated
app.post('/logout', (req, res) => {
  res.clearCookie('session');  // Cookie deleted client-side but server still accepts old token
  // Missing: req.session.destroy()
});

// DANGEROUS — JWT never expires or expiry not checked
jwt.sign(payload, secret);  // No expiresIn
jwt.verify(token, secret, { ignoreExpiration: true });
```

---

### A08 — Software and Data Integrity Failures

**Patterns:**
```python
# DANGEROUS — Pickle with untrusted data
import pickle
data = pickle.loads(user_provided_bytes)  # Remote code execution possible

# DANGEROUS — eval with user data
eval(user_provided_expression)

# DANGEROUS — Unverified file download
urllib.request.urlretrieve(user_url, '/tmp/file')  # No checksum verify
```

---

### A09 — Security Logging and Monitoring Failures

**Patterns (missing = finding):**
- No logging of failed authentication attempts
- No logging of access control failures
- PII (email, phone, SSN) logged in plaintext
- Passwords or tokens appearing in log output
- No alert on repeated auth failures (brute force)

---

### A10 — Server-Side Request Forgery (SSRF)

**Patterns:**
```python
# VULNERABLE — Fetches arbitrary URL from user
url = request.args.get('url')
response = requests.get(url)  # Could fetch http://169.254.169.254/metadata

# SECURE — Allowlist
ALLOWED_DOMAINS = ['api.partner.com', 'cdn.example.com']
parsed = urlparse(url)
if parsed.hostname not in ALLOWED_DOMAINS:
    raise ValueError("URL not allowed")
```

---

## Secrets Detection

### Regex Patterns DAVID Applies (Mental Model)

| Secret Type | Pattern Signature |
|-------------|-------------------|
| AWS Access Key | `AKIA[0-9A-Z]{16}` |
| AWS Secret Key | 40-char alphanumeric near "aws_secret" |
| GitHub Token | `ghp_[a-zA-Z0-9]{36}` |
| Stripe Live Key | `sk_live_[a-zA-Z0-9]{24}` |
| Stripe Test Key | `sk_test_[a-zA-Z0-9]{24}` |
| Twilio | `SK[a-fA-F0-9]{32}` |
| Google API Key | `AIza[0-9A-Za-z\-_]{35}` |
| Slack Token | `xox[baprs]-[0-9a-zA-Z]{10,}` |
| Generic JWT | Three base64 segments separated by `.` |
| Private Key | `-----BEGIN (RSA|EC|DSA|PRIVATE) KEY-----` |
| DB Connection | Strings like `postgresql://user:PASSWORD@host` |

### Hardcoded Credential Indicators
- Variable names: `password`, `passwd`, `pwd`, `secret`, `api_key`, `token`, `auth` with string literal values
- Config files with actual values instead of env var references
- Example: `PASSWORD = "hunter2"` vs safe `PASSWORD = os.environ['DB_PASSWORD']`

---

## Language-Specific Security Bugs

### Python
- `pickle.loads()` on untrusted data → RCE
- `yaml.load()` without `Loader=yaml.SafeLoader` → RCE  
- `subprocess(shell=True)` with user input → command injection
- `__reduce__` method in class → pickle RCE vector
- Timing attacks in password comparison (use `hmac.compare_digest`)

### JavaScript / Node.js
- `eval()` / `Function()` with user input → RCE
- `child_process.exec()` with string concatenation → command injection
- `JSON.parse()` on untrusted huge payloads → ReDoS / DoS
- Prototype pollution: `obj[user_key] = user_value` where key = `__proto__`
- `req.query` used directly in MongoDB query without type check

### Java
- `Runtime.exec()` with string concatenation
- Java deserialization (`ObjectInputStream`) with untrusted data
- XML parsing without disabling external entities (XXE)
- Reflection with user-controlled class names

### PHP
- `eval()`, `assert()` with user input
- `include`/`require` with user-controlled path
- `$_GET`/`$_POST` directly in SQL queries
- `unserialize()` with untrusted data

---

## API Security Checklist

| Check | What to Look For |
|-------|-----------------|
| Authentication | Every endpoint has auth middleware (except intentional public routes) |
| Authorization | Auth ≠ authz — verify user can access *this* resource, not just any resource |
| Input validation | All request fields validated (type, length, format, range) |
| Rate limiting | Auth endpoints, resource-intensive endpoints have rate limits |
| HTTPS only | No plain HTTP endpoints for sensitive data |
| Sensitive data in URL | Passwords/tokens never in GET query params (end up in logs) |
| CORS | Not wildcard `*` unless intentional public API |
| Error messages | Don't reveal internal state, stack traces, or DB structure |
| Pagination | No endpoint returns unbounded result sets |
| File uploads | MIME type validated server-side, not just client-side |

---

## Dependency Vulnerabilities

### Known-Bad Patterns (flag for user to investigate)
- `node-serialize` — known RCE vulnerability in deserialization
- `lodash < 4.17.21` — prototype pollution
- `axios < 1.6.0` — SSRF in some configurations
- `jsonwebtoken < 9.0.0` — algorithm confusion attacks
- `log4j 2.x < 2.17.1` — Log4Shell (Java)
- `requests < 2.20.0` — header injection
- Python `pickle` module — always flag usage with untrusted data
- `PyYAML < 6.0` without SafeLoader

When DAVID spots these patterns, flag them as SEC findings even if not directly exploitable in current code.

---

## Supply Chain Security (Scanner B Deepened — v6.0)

### Dependency Confusion Attacks

```
Risk: internal package name 'mycompany-utils' published publicly on npm
Attack: attacker publishes 'mycompany-utils@99.0.0' on npm
Result: npm install fetches attacker's version (higher version wins)

DAVID checks:
  ✅ Private package scopes (@company/package) used for internal packages
  ✅ .npmrc has registry scope lock: @mycompany:registry=https://private.registry
  ✅ package.json private:true on internal packages not meant for public registry
```

**Flag patterns:**
```json
// ❌ Internal package without scope — vulnerable to dependency confusion
{ "name": "mycompany-utils", "private": false }

// ✅ Scoped + private — protected
{ "name": "@mycompany/utils", "private": true }
```

### SBOM (Software Bill of Materials) Awareness

When a project has no SBOM, DAVID notes:
```
⚠️ No SBOM found. For production software, consider generating one:
  npm:    npx @cyclonedx/cyclonedx-npm --output sbom.json
  python: pip install cyclonedx-bom && cyclonedx-bom -o sbom.json
  A SBOM enables automated CVE monitoring and supply chain transparency.
```

SBOM triggers: `package.json` present with >10 dependencies and no sbom.json/sbom.xml detected.

---

## Input Sanitization Deep (Scanner AC)

> Scanner B flags input sanitization issues at the OWASP A03 level (high-level finding).
> Scanner AC handles detailed sanitization patterns exclusively to prevent duplicate finding IDs.
> If both B and AC would flag the same line, only AC's finding card is generated.

### Path Traversal

```javascript
// ❌ User controls file path
const file = fs.readFile(`./uploads/${req.params.filename}`);
// Attack: ../../../etc/passwd

// ✅ Sanitized path
const safeName = path.basename(req.params.filename);
const safePath = path.join(__dirname, 'uploads', safeName);
if (!safePath.startsWith(path.join(__dirname, 'uploads'))) {
  throw new Error('Path traversal detected');
}
```

### File Upload Validation

```javascript
// ❌ Client-side MIME check only (trivially bypassed)
if (file.type.startsWith('image/')) { upload(file); }

// ❌ Extension check only (rename .exe to .jpg)
if (file.name.endsWith('.jpg')) { upload(file); }

// ✅ Server-side magic bytes validation
import fileType from 'file-type';
const type = await fileType.fromBuffer(buffer);
const ALLOWED = ['image/jpeg', 'image/png', 'image/webp'];
if (!type || !ALLOWED.includes(type.mime)) throw new Error('Invalid file type');

// ✅ Also check: max file size, max dimensions (for images), virus scan for critical uploads
```

### CSV / Formula Injection

```javascript
// ❌ User data exported to CSV without sanitization
// User enters: =CMD|' /C calc'!A0 as their name
// Opens calculator when victim opens CSV in Excel

// ✅ Sanitize cells that start with formula characters
function sanitizeCsvCell(value) {
  if (typeof value === 'string' && /^[=+\-@\t\r]/.test(value)) {
    return `'${value}`;  // Prefix with apostrophe — treated as text in Excel
  }
  return value;
}
```

### Finding Format

```
🧹 SANITIZATION FINDING [AC-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Type       : [Path traversal / File upload / CSV injection]
Severity   : [🔴/🟠/🟡]
File:Line  : [file:N]
Input Source: [req.params / req.body / user upload]
Issue      : [description]
Attack     : [example payload that exploits this]
Fix        : [exact code change]
```

# DAVID — CI/CD Advisor Reference

Full pattern library for Scanner K. Load when auditing Docker, GitHub Actions, or other CI/CD configs.

---

## Docker Best Practices

### Security

```dockerfile
# ❌ Running as root (default)
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]

# ✅ Non-root user
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
CMD ["node", "server.js"]
```

```dockerfile
# ❌ Secrets in ENV (baked into image layers — visible in docker history)
ENV DATABASE_URL=postgresql://user:password@db:5432/mydb
ENV API_KEY=sk_live_abc123

# ✅ Runtime secrets via environment (not baked into image)
# Pass at runtime: docker run -e DATABASE_URL=$DATABASE_URL ...
# Or use Docker secrets / Kubernetes secrets
```

### Layer Caching

```dockerfile
# ❌ Defeats layer caching — any code change reinstalls all deps
FROM node:18
WORKDIR /app
COPY . .                    # ALL files copied first
RUN npm install             # Reinstalls every time ANY file changes

# ✅ Dependencies layer cached separately from code
FROM node:18
WORKDIR /app
COPY package*.json ./       # Only copy package files first
RUN npm ci                  # This layer only invalidates when package.json changes
COPY . .                    # Copy code after — doesn't affect deps layer
```

### Multi-Stage Builds

```dockerfile
# ❌ Single stage — build tools shipped to production
FROM node:18
WORKDIR /app
COPY . .
RUN npm install && npm run build
CMD ["node", "dist/server.js"]

# ✅ Multi-stage — only production artifacts in final image
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/server.js"]
```

### Image Tag Pinning

```dockerfile
# ❌ latest tag — non-deterministic builds
FROM python:latest
FROM node:latest

# ✅ Pin to specific version
FROM python:3.12.3-alpine3.19
FROM node:20.12.2-alpine3.19
```

### Healthcheck

```dockerfile
# ❌ No healthcheck — Docker can't detect if app is actually serving
CMD ["node", "server.js"]

# ✅ Healthcheck defined
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1
```

---

## GitHub Actions Best Practices

### Action Version Pinning

```yaml
# ❌ Branch reference — supply chain attack risk
- uses: actions/checkout@main
- uses: actions/setup-node@v3    # Tag reference — can be moved

# ✅ Pin to commit SHA (immutable)
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1
- uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8  # v4.0.2
```

### Secret Handling

```yaml
# ❌ Secret echoed in run command (appears in logs)
- run: |
    echo "Deploying with key: ${{ secrets.API_KEY }}"
    curl -H "Authorization: ${{ secrets.API_KEY }}" https://api.example.com/deploy

# ✅ Pass as environment variable (masked in logs)
- run: curl -H "Authorization: $API_KEY" https://api.example.com/deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

```yaml
# ❌ Secret in artifact or cache
- uses: actions/cache@v3
  with:
    key: ${{ secrets.PRIVATE_KEY }}-${{ hashFiles('**/lock') }}

# ❌ Printing environment (exposes all secrets)
- run: env
- run: printenv
```

### Timeout Settings

```yaml
# ❌ No timeout — job can hang forever (costs money + blocks queue)
jobs:
  build:
    runs-on: ubuntu-latest

# ✅ Timeout set
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@...
        timeout-minutes: 5
```

### Dependency Caching

```yaml
# ❌ No caching — npm install on every run
- run: npm install

# ✅ Cache node_modules
- uses: actions/cache@...
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- run: npm ci
```

### Privilege Escalation

```yaml
# ❌ Unnecessary permissions
permissions:
  contents: write
  packages: write
  id-token: write
  pull-requests: write

# ✅ Minimal permissions
permissions:
  contents: read
  packages: write    # Only if needed for this job
```

---

## Migration Safety Patterns

(Reference for Scanner J)

### Dangerous Operations Checklist

```sql
-- ❌ ALWAYS flag — get user confirmation before running
DROP TABLE users;
DROP COLUMN email FROM users;
ALTER TABLE orders ALTER COLUMN amount TYPE varchar(50);  -- Was varchar(255) — truncation!
DELETE FROM sessions;  -- No WHERE clause — deletes all sessions
UPDATE users SET role = 'admin';  -- No WHERE — promotes everyone

-- ❌ Will lock table in production (PostgreSQL)
ALTER TABLE large_table ADD COLUMN processed boolean DEFAULT false;
-- On tables with millions of rows, this takes an exclusive lock for minutes

-- ✅ Safe alternative (PostgreSQL)
ALTER TABLE large_table ADD COLUMN processed boolean;          -- No default = no lock
UPDATE large_table SET processed = false WHERE processed IS NULL;
ALTER TABLE large_table ALTER COLUMN processed SET DEFAULT false;
ALTER TABLE large_table ALTER COLUMN processed SET NOT NULL;
```

### Rollback Requirements

Every migration MUST have a corresponding rollback:

```python
# Alembic
def upgrade():
    op.add_column('users', sa.Column('phone', sa.String(20), nullable=True))

def downgrade():
    op.drop_column('users', 'phone')

# ❌ Missing downgrade — no way to roll back
def upgrade():
    op.add_column('users', sa.Column('phone', sa.String(20)))

def downgrade():
    pass  # Missing rollback!
```

### Non-Idempotent Migrations

```sql
-- ❌ Will fail on second run
CREATE TABLE new_feature (id serial PRIMARY KEY);
INSERT INTO config (key, value) VALUES ('new_feature_enabled', 'true');

-- ✅ Idempotent
CREATE TABLE IF NOT EXISTS new_feature (id serial PRIMARY KEY);
INSERT INTO config (key, value) VALUES ('new_feature_enabled', 'true')
    ON CONFLICT (key) DO NOTHING;
```

### Adding NOT NULL to Existing Table

```sql
-- ❌ Will fail if any existing rows — can't add NOT NULL without default to populated table
ALTER TABLE orders ADD COLUMN currency varchar(3) NOT NULL;

-- ✅ Three-step safe migration
-- Step 1: Add nullable column
ALTER TABLE orders ADD COLUMN currency varchar(3);
-- Step 2: Backfill existing rows
UPDATE orders SET currency = 'USD' WHERE currency IS NULL;
-- Step 3: Add NOT NULL constraint (after all rows have data)
ALTER TABLE orders ALTER COLUMN currency SET NOT NULL;
```

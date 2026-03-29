# DAVID — Database Schema Quality Reference (Scanner AG2)

Full patterns for database schema design quality. Load for SQL schema files and ORM model definitions.

---

## Table of Contents
1. [Index Strategy Guide](#index-strategy-guide)
2. [Constraint Design Patterns](#constraint-design-patterns)
3. [Audit Fields Pattern](#audit-fields-pattern)
4. [Schema Anti-Patterns Catalog](#schema-anti-patterns-catalog)
5. [Query Pattern → Index Mapping](#query-pattern--index-mapping)
6. [PostgreSQL-Specific Best Practices](#postgresql-specific-best-practices)

---


## Index Strategy Guide

### When to Add an Index

```
ADD INDEX when column is used in:
  WHERE clause (especially equality: WHERE user_id = ?)
  JOIN conditions (ON orders.user_id = users.id)
  ORDER BY (if combined with filtered WHERE)
  GROUP BY on large result sets

SKIP INDEX when:
  Table < 10,000 rows (full scan is faster)
  Column has very low cardinality (boolean, status with 2 values)
  Column is written more than read (index overhead)
  Column is already first in composite index covering this query
```

### Index Coverage by ORM

```python
# Django
class Order(models.Model):
    user    = models.ForeignKey(User, on_delete=models.CASCADE)
    # ✅ Django auto-creates index on FK — verify with: manage.py sqlmigrate
    status  = models.CharField(max_length=20)
    # ❌ No auto-index on status

    class Meta:
        indexes = [
            models.Index(fields=['status']),              # Single column
            models.Index(fields=['user', 'status']),      # Composite
            models.Index(fields=['user', '-created_at']), # DESC for pagination
        ]
        constraints = [
            models.CheckConstraint(
                check=models.Q(status__in=['pending', 'paid', 'cancelled']),
                name='valid_order_status'
            )
        ]
```

```typescript
// Prisma — index must be explicit
model Order {
  id        Int      @id @default(autoincrement())
  userId    Int
  status    String
  createdAt DateTime @default(now())

  user      User     @relation(fields: [userId], references: [id])

  @@index([userId])                    // FK index — explicit in Prisma
  @@index([status])                    // Filtered queries
  @@index([userId, status])            // Composite for: WHERE user_id=? AND status=?
  @@index([userId, createdAt(sort: Desc)])  // Pagination: latest orders per user
}
```

```sql
-- Raw SQL — comprehensive index strategy
-- After creating the table, run:
CREATE INDEX CONCURRENTLY idx_orders_user_id
    ON orders(user_id);

CREATE INDEX CONCURRENTLY idx_orders_status
    ON orders(status)
    WHERE status IN ('pending', 'processing');  -- Partial index: only active statuses

CREATE INDEX CONCURRENTLY idx_orders_user_status_created
    ON orders(user_id, status, created_at DESC);  -- Covers: WHERE user_id=? AND status=? ORDER BY created_at DESC

-- For full-text search:
CREATE INDEX idx_products_search
    ON products USING gin(to_tsvector('english', name || ' ' || description));
```

---

## Constraint Design Patterns

### Complete Constraint Checklist

```sql
-- ✅ Every table should have these analyzed:
CREATE TABLE users (
    -- 1. Primary key
    id          BIGSERIAL PRIMARY KEY,

    -- 2. Unique constraints on natural keys
    email       VARCHAR(254) NOT NULL UNIQUE,    -- Email uniqueness
    username    VARCHAR(50)  NOT NULL UNIQUE,    -- Username uniqueness

    -- 3. NOT NULL on required fields
    name        VARCHAR(255) NOT NULL,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),

    -- 4. CHECK constraints for valid values
    status      VARCHAR(20)  NOT NULL
                CHECK (status IN ('active', 'inactive', 'suspended', 'deleted')),
    age         SMALLINT     CHECK (age IS NULL OR (age >= 0 AND age <= 150)),
    score       DECIMAL(5,2) CHECK (score >= 0 AND score <= 100),

    -- 5. Default values for non-nullable fields
    updated_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    is_verified BOOLEAN      NOT NULL DEFAULT FALSE,
    role        VARCHAR(20)  NOT NULL DEFAULT 'user'
                CHECK (role IN ('user', 'admin', 'moderator')),
);

-- ✅ Foreign keys with appropriate ON DELETE behavior
CREATE TABLE orders (
    id          BIGSERIAL PRIMARY KEY,
    user_id     BIGINT NOT NULL REFERENCES users(id)
                ON DELETE RESTRICT,     -- Prevent deleting users with orders
    coupon_id   BIGINT REFERENCES coupons(id)
                ON DELETE SET NULL,     -- OK to delete coupon, nullify reference
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### Status Column Anti-Patterns

```sql
-- ❌ VARCHAR status without constraint — anything goes
status VARCHAR(50)
-- Allows: 'Active', 'active', 'ACTIVE', 'aactive', 'pending ', null, ''

-- ❌ Integer status codes with no documentation
status INT  -- What does 1 mean? 2? -1?

-- ✅ Enum type (PostgreSQL)
CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled');
ALTER TABLE orders ADD COLUMN status order_status NOT NULL DEFAULT 'pending';

-- ✅ VARCHAR with CHECK constraint (more portable)
status VARCHAR(20) NOT NULL
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
    DEFAULT 'pending'
```

---

## Audit Fields Pattern

### Standard Audit Columns

```sql
-- ✅ Every production table should have these:
ALTER TABLE products ADD COLUMN created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW();
ALTER TABLE products ADD COLUMN updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW();
ALTER TABLE products ADD COLUMN created_by  BIGINT REFERENCES users(id) ON DELETE SET NULL;
ALTER TABLE products ADD COLUMN updated_by  BIGINT REFERENCES users(id) ON DELETE SET NULL;

-- Auto-update updated_at on every change (PostgreSQL)
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER products_updated_at
    BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### Soft Delete Pattern

```sql
-- ✅ Consistent soft delete across all tables
ALTER TABLE users    ADD COLUMN deleted_at TIMESTAMPTZ;
ALTER TABLE products ADD COLUMN deleted_at TIMESTAMPTZ;
ALTER TABLE orders   ADD COLUMN deleted_at TIMESTAMPTZ;

-- Partial indexes exclude soft-deleted records from common queries
CREATE INDEX idx_users_email_active    ON users(email)    WHERE deleted_at IS NULL;
CREATE INDEX idx_products_sku_active   ON products(sku)   WHERE deleted_at IS NULL;

-- Views for convenience
CREATE VIEW active_users AS SELECT * FROM users WHERE deleted_at IS NULL;
CREATE VIEW active_products AS SELECT * FROM products WHERE deleted_at IS NULL;
```

```python
# Django soft delete mixin
from django.db import models
from django.utils import timezone

class SoftDeleteManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(deleted_at__isnull=True)

class SoftDeleteModel(models.Model):
    deleted_at = models.DateTimeField(null=True, blank=True)
    objects = SoftDeleteManager()  # Default: active only
    all_objects = models.Manager()  # Includes deleted

    def delete(self, using=None, keep_parents=False):
        self.deleted_at = timezone.now()
        self.save(update_fields=['deleted_at'])

    def hard_delete(self):
        super().delete()

    class Meta:
        abstract = True
```

---

## Schema Anti-Patterns Catalog

### EAV (Entity-Attribute-Value) — The Schema Killer

```sql
-- ❌ EAV pattern — looks flexible, destroys performance and integrity
CREATE TABLE attributes (
    entity_id   BIGINT,
    entity_type VARCHAR(50),
    attribute   VARCHAR(100),
    value       TEXT  -- Everything is text — no types, no constraints
);
-- INSERT INTO attributes VALUES (1, 'product', 'price', '29.99');
-- INSERT INTO attributes VALUES (1, 'product', 'price', 'twenty dollars');  -- Also valid!
-- SELECT * FROM attributes WHERE entity_id=1 AND attribute='price'  -- Full scan

-- ✅ Proper table per entity type
CREATE TABLE products (
    id    BIGSERIAL PRIMARY KEY,
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    -- Real columns with real types and constraints
);
-- If you truly need dynamic attributes: use JSONB column (with indexes)
CREATE TABLE products (
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(255) NOT NULL,
    price      DECIMAL(10,2) NOT NULL,
    attributes JSONB DEFAULT '{}'::jsonb  -- Flexible but typed at root level
);
CREATE INDEX idx_products_attributes ON products USING gin(attributes);
-- Query: SELECT * FROM products WHERE attributes->>'color' = 'red'
```

### Storing Comma-Separated Lists

```sql
-- ❌ Anti-pattern: CSV in a column
CREATE TABLE posts (
    id      BIGSERIAL PRIMARY KEY,
    title   VARCHAR(255),
    tag_ids TEXT  -- '1,4,7,12' — can't query efficiently, no FK integrity
);

-- ✅ Junction table
CREATE TABLE post_tags (
    post_id BIGINT NOT NULL REFERENCES posts(id)  ON DELETE CASCADE,
    tag_id  BIGINT NOT NULL REFERENCES tags(id)   ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)  -- Composite PK prevents duplicates
);
CREATE INDEX idx_post_tags_tag_id ON post_tags(tag_id);  -- For: "find all posts with tag X"
```

### Missing Composite Unique Constraints

```sql
-- ❌ Missing uniqueness on junction table — allows duplicate relationships
CREATE TABLE user_roles (
    user_id BIGINT REFERENCES users(id),
    role_id BIGINT REFERENCES roles(id)
    -- No constraint — can insert same user/role pair multiple times
);

-- ✅ Composite unique or PK
CREATE TABLE user_roles (
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id BIGINT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id, role_id)  -- Prevents duplicates, provides index
);

-- ✅ Another example: one subscription per user per product
ALTER TABLE subscriptions
    ADD CONSTRAINT uq_user_product UNIQUE (user_id, product_id);
```

---

## Query Pattern → Index Mapping

DAVID matches common query patterns to required indexes:

| Query Pattern | Required Index |
|--------------|----------------|
| `WHERE user_id = ?` | `INDEX(user_id)` |
| `WHERE user_id = ? AND status = ?` | `INDEX(user_id, status)` |
| `WHERE user_id = ? ORDER BY created_at DESC` | `INDEX(user_id, created_at DESC)` |
| `WHERE status = ? AND created_at > ?` | `INDEX(status, created_at)` |
| `WHERE email ILIKE '%term%'` | `gin(lower(email) gin_trgm_ops)` — pg_trgm |
| `WHERE to_tsvector(content) @@ query` | `gin(to_tsvector(content))` |
| `WHERE data->>'key' = ?` (JSONB) | `gin(data)` or `((data->>'key'))` |
| `WHERE deleted_at IS NULL AND user_id = ?` | Partial: `INDEX(user_id) WHERE deleted_at IS NULL` |

---

## PostgreSQL-Specific Best Practices

```sql
-- ✅ Use BIGSERIAL not SERIAL for IDs (avoids 2B row limit)
id BIGSERIAL PRIMARY KEY  -- not: SERIAL

-- ✅ Use TIMESTAMPTZ not TIMESTAMP (stores timezone, avoids DST bugs)
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()  -- not: TIMESTAMP

-- ✅ Use TEXT not VARCHAR(n) unless you need the length check
-- TEXT and VARCHAR are same internally in PG, VARCHAR adds a check
name TEXT NOT NULL  -- or VARCHAR(255) if you want to enforce max length

-- ✅ CONCURRENTLY for indexes on live tables (doesn't lock)
CREATE INDEX CONCURRENTLY idx_orders_user_id ON orders(user_id);
-- Not: CREATE INDEX idx_orders_user_id ON orders(user_id);  -- Locks table!

-- ✅ Partial indexes for filtered queries
CREATE INDEX idx_users_unverified ON users(created_at)
    WHERE is_verified = FALSE;  -- Only indexes unverified users
```

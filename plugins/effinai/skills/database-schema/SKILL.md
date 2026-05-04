---
name: database-schema
description: "Database schema design from requirements with migration generation"
version: "1.0.0"
tags: [engineering, database, schema, migration]
createdBy: "built-in"
status: "active"
---

# Database Schema Design

## Activation
When the user asks to design a database, create tables, or generate migrations.

## Workflow

### Step 1: Gather Requirements
From the user, determine:
1. **Entities**: What data needs to be stored?
2. **Relationships**: How do entities relate? (1:1, 1:N, N:M)
3. **Database type**: PostgreSQL, MySQL, SQLite, MongoDB?
4. **ORM**: Prisma, Drizzle, TypeORM, Sequelize, SQLAlchemy, Knex?
5. **Scale expectations**: Approximate row counts, read/write ratio

### Step 2: Design the Schema

For each entity, define:
- Table name (plural, snake_case)
- Primary key (prefer UUID over serial for distributed systems)
- Columns with types, constraints, defaults
- Indexes for frequently queried columns
- Foreign keys with ON DELETE behavior
- Created/updated timestamps

**Standard columns for every table:**
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
updated_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
```

**Relationship patterns:**
- 1:1 — Foreign key with UNIQUE constraint on either table
- 1:N — Foreign key on the "many" side
- N:M — Junction table with composite primary key

### Step 3: Normalization Check
- 1NF: No repeating groups, atomic values
- 2NF: No partial dependencies (all non-key columns depend on full PK)
- 3NF: No transitive dependencies
- Consider strategic denormalization for read-heavy queries (document the trade-off)

### Step 4: Index Strategy
```
- Primary keys: automatic index
- Foreign keys: always index
- Columns in WHERE clauses: index
- Columns in ORDER BY: consider index
- Composite indexes: leftmost prefix rule
- Unique constraints: implicit index
- Full-text search: GIN/GiST index
```

### Step 5: Generate Output

**SQL Migration:**
```sql
CREATE TABLE [entity] (
  ...
);
CREATE INDEX idx_[entity]_[column] ON [entity]([column]);
```

**ORM Schema (Prisma example):**
```prisma
model Entity {
  id        String   @id @default(uuid())
  ...
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  @@map("entities")
}
```

**Generate both the UP and DOWN migration.**

### Step 6: Data Integrity Checks
- [ ] All foreign keys have appropriate ON DELETE (CASCADE, SET NULL, RESTRICT)
- [ ] Required fields are NOT NULL
- [ ] Enum values are constrained (CHECK or enum type)
- [ ] Unique constraints on natural keys (email, username, slug)
- [ ] Soft delete considered vs hard delete
- [ ] Audit trail for sensitive data changes

## Rules
- Always include created_at and updated_at
- Prefer UUIDs over auto-increment for distributed systems
- Always create indexes for foreign keys
- Generate reversible migrations (both up and down)
- Use snake_case for database names, camelCase for ORM models
- Consider row-level security for multi-tenant schemas
- Document any denormalization decisions

---
description: DB (SQLite) coding conventions for ab-java-react
globs: "back/src/main/resources/**/*.{sql,yml,properties}"
---
# DB Conventions — ab-java-react

## Summary

Embedded SQLite store owned by the `back` tier. Schema is JPA-managed for the archetype (`ddl-auto: update`) but follows portable, explicit conventions: `snake_case` identifiers, deliberate type affinity, ISO-8601 UTC timestamps stored as `TEXT`, and connection PRAGMAs tuned for reliability (WAL, foreign keys, busy timeout). SQLite is single-writer — design for that, not against it.

---

## Naming

### Folders
- **Pattern**: layer-based — DB file under `data/`, optional migrations under `db/migration/`
- **Casing**: kebab-case folders, lowercase file names
- **Examples**: `data/app.db`, `db/migration`

### Files
- **Database file**: `app.db` (plus WAL sidecars `app.db-wal`, `app.db-shm` — git-ignored)
- **Migration** (if adopted): `V{n}__{snake_summary}.sql` — `V1__create_health_check.sql`

### Language Elements
- **Tables**: `snake_case`, singular noun — `health_check`
- **Columns**: `snake_case` — `database_status`, `checked_at`
- **Primary key**: `id` (`INTEGER PRIMARY KEY` → aliases `rowid`)
- **Foreign key columns**: `{singular_table}_id` — `user_id`
- **Indexes**: `idx_{table}_{columns}` — `idx_health_check_checked_at`
- **Constraints**: `ck_{table}_{rule}`, `uq_{table}_{columns}`

---

## Artifact Roles

### Table

**Dominant pattern**: explicit column types via affinity, `NOT NULL` by default, no `VARCHAR(n)` length games (SQLite ignores them — use `TEXT`).

**Canonical example**:
```sql
CREATE TABLE health_check (
  id              INTEGER PRIMARY KEY AUTOINCREMENT,
  status          TEXT    NOT NULL CHECK (status IN ('UP','DOWN')),
  database_status TEXT    NOT NULL CHECK (database_status IN ('UP','DOWN')),
  checked_at      TEXT    NOT NULL  -- ISO-8601 UTC, e.g. 2026-05-29T12:00:00Z
);
```

**Anti-pattern**:
- Storing dates as localized strings or epoch-with-timezone ambiguity; use ISO-8601 UTC `TEXT` (or INTEGER unix seconds) consistently.
- Relying on implicit `rowid` when you need a stable id across `VACUUM` — declare `INTEGER PRIMARY KEY`.

### Index

**Dominant pattern**: index foreign keys and columns used in `WHERE`/`ORDER BY`; name them explicitly.

**Canonical example**:
```sql
CREATE INDEX idx_health_check_checked_at ON health_check (checked_at DESC);
```

**Anti-pattern**:
- Over-indexing a tiny/append-only table; add indexes only when a query needs them.

### Connection configuration (PRAGMAs)

**Dominant pattern**: set PRAGMAs per connection at startup; these are the reliability baseline for a server using SQLite.

**Canonical example**:
```sql
PRAGMA journal_mode = WAL;      -- concurrent readers during a write
PRAGMA foreign_keys = ON;       -- OFF by default in SQLite — must enable
PRAGMA busy_timeout = 5000;     -- wait for the single writer instead of failing
PRAGMA synchronous = NORMAL;    -- safe + fast under WAL
```
Configured in `application.yml` via the datasource URL/connection-init-sql, e.g.
`jdbc:sqlite:data/app.db?foreign_keys=on&journal_mode=WAL&busy_timeout=5000`.

**Anti-pattern**:
- Leaving `foreign_keys` OFF (SQLite's default) so FK constraints silently do nothing.
- Opening many writer connections — keep a small pool (effectively one writer) to avoid `SQLITE_BUSY`.

---

## Error Handling

SQLite serializes writes: a second concurrent writer gets `SQLITE_BUSY`. Rely on `busy_timeout` first; keep transactions short and commit promptly. Treat the DB file path as configuration; never hardcode absolute paths. Back up by copying the file only when the WAL is checkpointed.

---

## Known Deviations

- No deviations yet — greenfield baseline. Schema is JPA-generated; if drift or production needs arise, adopt Flyway migrations under `db/migration/` and switch `ddl-auto` to `validate`.

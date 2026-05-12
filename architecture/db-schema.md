# Database Schema — OpenSpectrum MVP

**Status**: Draft
**Last updated**: 2026-05-12
**Related ADR**: [ADR-001 — Local-first SQLite storage](./adr/001-local-first-sqlite.md)

---

## Design Principles

1. **Local-first**: Schema is designed for SQLite. No cloud-specific types.
2. **Child-centric**: All data belongs to a child profile, never to an account.
3. **Privacy by design**: No PII stored beyond what's entered by the parent.
4. **Migration-ready**: Schema can be migrated to PostgreSQL when self-hosting multi-user support is added.

---

## Entity Overview

```
Child ──< Event
Child ──< Medication
Child ──< ConsentLog
Event >── Category (enum)
```

---

## Tables

### `children`

Stores one record per child being tracked.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique identifier |
| `name` | TEXT | NOT NULL | Child's name (display only) |
| `date_of_birth` | DATE | nullable | Used for age calculations |
| `profile_notes` | TEXT | nullable | Free-form notes (diagnoses, therapist, etc.) |
| `created_at` | DATETIME | NOT NULL, default now | Record creation timestamp |
| `updated_at` | DATETIME | NOT NULL, default now | Last update timestamp |

---

### `events`

The core table. Every logged observation is an event.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique identifier |
| `child_id` | INTEGER | FK → children.id, NOT NULL | Which child this event belongs to |
| `timestamp` | DATETIME | NOT NULL | When the event occurred (user-provided, not insertion time) |
| `category` | TEXT | NOT NULL | One of: `food`, `medication`, `behavior`, `milestone`, `other` |
| `raw_text` | TEXT | NOT NULL | Free-text as entered by the parent |
| `parsed_data` | JSON | nullable | Structured data extracted from raw_text (future: AI parsing) |
| `input_method` | TEXT | default `text` | One of: `text`, `voice` |
| `created_at` | DATETIME | NOT NULL, default now | Insertion timestamp |

**Notes:**
- `timestamp` vs `created_at`: `timestamp` is when the event happened; `created_at` is when it was logged. They may differ (e.g., parent logs an event from memory hours later).
- `parsed_data` is reserved for future structured extraction (e.g., food items, dose amounts). MVP stores null here.
- `category` is stored as a string (not a foreign key to a categories table) for simplicity. Controlled by application-level validation.

---

### `medications`

Tracks recurring medications for a child, used for scheduling and correlation.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique identifier |
| `child_id` | INTEGER | FK → children.id, NOT NULL | Which child this medication belongs to |
| `name` | TEXT | NOT NULL | Medication or supplement name |
| `dose` | TEXT | nullable | Dose description (e.g., "5mg", "1 tablet") |
| `schedule` | TEXT | nullable | Free-text schedule (e.g., "Morning with food") |
| `active` | BOOLEAN | NOT NULL, default true | Whether this medication is currently active |
| `created_at` | DATETIME | NOT NULL, default now | Record creation timestamp |

**Notes:**
- Medication events are still logged in the `events` table. This table stores the *template* for a medication, not individual doses.
- Correlation queries will join `events` where `category = 'medication'` against this table.

---

### `consent_log`

Audit trail for any privacy-sensitive settings the parent has toggled.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique identifier |
| `child_id` | INTEGER | FK → children.id, NOT NULL | Which child this consent applies to |
| `setting` | TEXT | NOT NULL | Setting key (e.g., `anonymous_sharing`, `export_enabled`) |
| `value` | TEXT | NOT NULL | Setting value (e.g., `true`, `false`) |
| `updated_at` | DATETIME | NOT NULL, default now | When this consent was recorded |

**Notes:**
- Append-only: never delete or update rows. Add a new row when a setting changes.
- `child_id` is included so consent is tracked per child, not per user account.

---

## Category Values (MVP)

The `events.category` column must be one of:

| Value | Description |
|---|---|
| `food` | Meal, snack, or drink logged |
| `medication` | Medication or supplement taken |
| `behavior` | Behavioral observation (positive or challenging) |
| `milestone` | Achievement or developmental milestone |
| `other` | Anything that doesn't fit the above |

---

## Schema SQL (SQLite)

```sql
CREATE TABLE children (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    name        TEXT NOT NULL,
    date_of_birth DATE,
    profile_notes TEXT,
    created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE events (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    child_id    INTEGER NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    timestamp   DATETIME NOT NULL,
    category    TEXT NOT NULL CHECK(category IN ('food','medication','behavior','milestone','other')),
    raw_text    TEXT NOT NULL,
    parsed_data TEXT,  -- JSON
    input_method TEXT NOT NULL DEFAULT 'text' CHECK(input_method IN ('text','voice')),
    created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE medications (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    child_id    INTEGER NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    name        TEXT NOT NULL,
    dose        TEXT,
    schedule    TEXT,
    active      BOOLEAN NOT NULL DEFAULT 1,
    created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE consent_log (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    child_id    INTEGER NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    setting     TEXT NOT NULL,
    value       TEXT NOT NULL,
    updated_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Index for common query patterns
CREATE INDEX idx_events_child_timestamp ON events(child_id, timestamp);
CREATE INDEX idx_events_category ON events(category);
CREATE INDEX idx_consent_child_setting ON consent_log(child_id, setting);
```

---

## Future Considerations

- **Tags**: Add an `event_tags` junction table for arbitrary tagging (post-MVP)
- **Multi-user**: Add a `users` table and `user_children` junction table when caregiver sync is added
- **PostgreSQL migration**: All types used here have direct PostgreSQL equivalents. JSON → JSONB for better indexing.
- **Encryption**: For at-rest encryption, SQLCipher (a SQLite extension) can be adopted without schema changes.

---

## Open Questions

- [ ] Should `date_of_birth` be stored, or just the child's age range? (Privacy consideration)
- [ ] Does `medications` need a `notes` field for the MVP?

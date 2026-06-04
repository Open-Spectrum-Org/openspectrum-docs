# Database Schema — OpenSpectrum

**Status**: v2 — Comprehensive
**Last updated**: 2026-06-04
**Replaces**: v1 (2026-05-12, 4-table MVP schema)
**Related docs**:
- [ADR-001 — Local-first SQLite storage](./adr/001-local-first-sqlite.md)
- [Tech Stack](./tech-stack-chatgpt-2026-06-01.md)
- [User Roles & Permissions](./user-roles-table-2026-06-01.md)
- [10 Screens](../wireframes/screens/10-screens.md)

---

## Design Principles

1. **Offline-first**: Schema works in SQLite (local device) and PostgreSQL (cloud). No cloud-specific types.
2. **Family-centric**: Data belongs to families, not individual user accounts.
3. **Privacy by design**: Minimal PII. No exact dates of birth. No full legal names.
4. **Sync-ready**: UUID primary keys prevent ID collisions across devices. Every synced table carries sync metadata.
5. **Soft deletes**: Synced records are never physically deleted — `is_deleted` flag propagates deletions across devices.

---

## Entity Overview

```
User ──< FamilyMember >── Family
                           │
                    Family ──< Child
                    Family ──< FamilyInvite
                           │
                    Child ──< Observation ──< ObservationTag >── TagDefinition
                    Child ──< MedicationTemplate ──< MedicationEvent
                    Child ──< DailyReflection
                    Child ──< Reminder
                    Child ──< Report ──< ReportShare
                    Child ──< ConsentLog
                    Child ──< ChildTag >── TagDefinition
                           │
              Observation ──< VoiceLog ──< AIExtractedEvent
              Observation ──< Attachment
              Observation ──< ObservationAccess
```

---

## Table Summary (20 tables, 6 domains)

| Domain | Tables |
|---|---|
| Identity & Access | `users`, `families`, `family_members`, `family_invites` |
| Child Profiles & Tags | `children`, `tag_definitions`, `child_tags` |
| Observations & Logging | `observations`, `observation_tags`, `voice_logs`, `ai_extracted_events`, `observation_access` |
| Medications | `medication_templates`, `medication_events` |
| Reflections, Reminders, Reports | `daily_reflections`, `reminders`, `reports`, `report_shares` |
| System | `consent_log`, `sync_queue` |

---

## Sync Metadata Columns

Every table that syncs to the cloud includes these columns:

| Column | Type | Purpose |
|---|---|---|
| `is_deleted` | INTEGER (0/1) | Soft delete flag |
| `sync_status` | TEXT | `pending`, `synced`, or `conflict` |
| `last_synced_at` | TEXT (ISO 8601) | Last successful sync timestamp |
| `device_id` | TEXT | Device that created/last modified this record |
| `version` | INTEGER | Incremented on each update. Used for optimistic concurrency. |

Tables marked **[local-only]** do not sync and omit these columns.

---

## Domain 1: Identity & Access

### `users`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `email` | TEXT | UNIQUE, nullable | NULL for local-only users (no account yet) |
| `display_name` | TEXT | NOT NULL | |
| `user_type` | TEXT | NOT NULL | `parent`, `grandparent`, `caregiver`, `therapist`, `doctor`, `teacher`, `other` — label only, does not grant permissions |
| `avatar_url` | TEXT | nullable | |
| `auth_provider_id` | TEXT | nullable | Supabase Auth UID |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | `is_deleted`, `sync_status`, `last_synced_at`, `device_id`, `version` |

---

### `families`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `family_name` | TEXT | NOT NULL | Display name for the family unit |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

---

### `family_members`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `family_id` | TEXT | FK → families.id, NOT NULL | |
| `user_id` | TEXT | FK → users.id, NOT NULL | |
| `role` | TEXT | NOT NULL | `owner`, `editor`, `viewer` |
| `joined_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Constraints:** UNIQUE(`family_id`, `user_id`)

---

### `family_invites`

Co-parent invite tokens (Screen 10).

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `family_id` | TEXT | FK → families.id, NOT NULL | |
| `invited_by` | TEXT | FK → users.id, NOT NULL | |
| `invite_token` | TEXT | NOT NULL, UNIQUE | Secure random token |
| `role` | TEXT | NOT NULL | Role granted on acceptance |
| `expires_at` | TEXT | NOT NULL | Token expiry |
| `accepted_at` | TEXT | nullable | |
| `accepted_by` | TEXT | FK → users.id, nullable | |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

**Note:** Managed server-side when cloud sync is active.

---

## Domain 2: Child Profiles & Tags

### `children`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `family_id` | TEXT | FK → families.id, NOT NULL | Children belong to families |
| `display_name` | TEXT | NOT NULL | Nickname, not legal name |
| `birth_year_month` | TEXT | nullable | Format: `2018-04`. No exact DOB (privacy). |
| `avatar_url` | TEXT | nullable | Local file path or cloud URL |
| `profile_notes` | TEXT | nullable | Free-form: diagnoses, therapist info |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Changes from v1:** `name` → `display_name`, `date_of_birth` → `birth_year_month`, added `family_id`.

---

### `tag_definitions`

Catalog of predefined and custom tags. Covers the Quick Tap grid (Screen 2), incident behaviors/triggers (Screen 3), and custom behavioral labels (Screen 8).

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `category` | TEXT | NOT NULL | `behavior`, `food`, `medication`, `emotion`, `sleep`, `sensory`, `transitions`, `successes`, `trigger`, `other` |
| `name` | TEXT | NOT NULL | e.g., "Meltdown", "Ate Well", "Noise" |
| `is_system` | INTEGER | NOT NULL, DEFAULT 1 | 1 = predefined, 0 = custom |
| `child_id` | TEXT | FK → children.id, nullable | NULL for system tags; set for per-child custom tags |
| `family_id` | TEXT | FK → families.id, nullable | Set if custom tag is family-wide |
| `display_order` | INTEGER | DEFAULT 0 | For ordering in the Quick Tap grid |
| `color` | TEXT | nullable | Hex color for timeline display |
| `icon` | TEXT | nullable | Icon identifier |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Constraints:** UNIQUE(`category`, `name`, `child_id`)

**Predefined system tags (seeded on install):**

| Category | Tags |
|---|---|
| `behavior` | Meltdown, Calm, Aggressive, Focused, Stim, Anxious, Shutdown, Crying, Eloping, Self-harm |
| `food` | Ate Well, Refused Food, Dairy, Sugar |
| `medication` | Taken, Missed, Side Effect |
| `emotion` | Happy, Irritated, Tired, Overwhelmed |
| `sleep` | Slept Well, Poor Sleep, Nap |
| `sensory` | Sensory Overload, Sensory Seeking |
| `transitions` | Good Transition, Difficult Transition |
| `successes` | Success Moment |
| `trigger` | Noise, Hunger, Transition, School, Medication, Screen Time, Social Situation, Change in Routine, Sensory Overload, Unknown |

---

### `child_tags`

Controls which tags appear on Quick Capture for each child. Populated from `tag_definitions` when a child profile is created.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `child_id` | TEXT | FK → children.id, NOT NULL | |
| `tag_id` | TEXT | FK → tag_definitions.id, NOT NULL | |
| `is_enabled` | INTEGER | NOT NULL, DEFAULT 1 | Toggle on/off in Screen 8 |
| `display_order` | INTEGER | DEFAULT 0 | Child-specific ordering |

**Constraints:** UNIQUE(`child_id`, `tag_id`)

---

## Domain 3: Observations & Logging

### `observations`

The central table. Every logged event — quick tap, stress/incident, voice-derived, lock screen — becomes a row.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `child_id` | TEXT | FK → children.id, NOT NULL | |
| `created_by` | TEXT | FK → users.id, nullable | NULL for lock screen entries (no auth) |
| `occurred_at` | TEXT | NOT NULL | When the event happened (user-meaningful time) |
| `entry_type` | TEXT | NOT NULL | `quick_tap`, `incident`, `voice`, `lock_screen`, `manual` |
| `category` | TEXT | NOT NULL | `behavior`, `food`, `medication`, `emotion`, `sleep`, `sensory`, `transitions`, `successes`, `milestone`, `other` |
| `title` | TEXT | nullable | Short label: "Meltdown", "Medication Taken" |
| `notes` | TEXT | nullable | Free-text notes or raw text entry |
| `incident_data` | TEXT | nullable | JSON — structured incident fields (see below) |
| `visibility_level` | TEXT | NOT NULL, DEFAULT 'family' | `family`, `parents`, `clinical`, `private`, `custom` |
| `is_partial` | INTEGER | NOT NULL, DEFAULT 0 | 1 = incomplete entry (user interrupted) |
| `voice_log_id` | TEXT | FK → voice_logs.id, nullable | If derived from voice capture |
| `parent_observation_id` | TEXT | FK → observations.id, nullable | For AI-extracted sub-events linked to a parent voice log |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | When the record was inserted |
| `updated_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**`incident_data` JSON structure** (used when `entry_type = 'incident'`):

```json
{
  "behaviors": ["meltdown", "crying"],
  "severity": "moderate",
  "triggers": ["noise", "transition"],
  "duration_minutes": null
}
```

**Key distinctions:**
- `occurred_at` = when the event happened. `created_at` = when it was logged. They may differ.
- `is_partial` supports user story 4.2 (interruption-tolerant partial data display: "Incomplete entry — Tap to finish adding details").
- `parent_observation_id` links AI-extracted sub-events back to the parent voice log observation.

**Changes from v1:** Table renamed from `events`. `timestamp` → `occurred_at`. `raw_text` → `notes`. `input_method` → `entry_type` (expanded values). `parsed_data` → `incident_data`. Categories expanded from 5 to 10.

---

### `observation_tags`

Junction table linking observations to tags. A quick tap of "Meltdown" inserts one row. An incident with multiple behaviors selected inserts multiple rows.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `observation_id` | TEXT | FK → observations.id, NOT NULL | |
| `tag_id` | TEXT | FK → tag_definitions.id, NOT NULL | |

**Constraints:** PRIMARY KEY(`observation_id`, `tag_id`)

---

### `voice_logs`

Voice capture data linked 1:1 to an observation.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `observation_id` | TEXT | FK → observations.id, NOT NULL, UNIQUE | |
| `audio_file_path` | TEXT | nullable | Local path to audio recording |
| `audio_duration_secs` | INTEGER | nullable | |
| `raw_transcript` | TEXT | nullable | Live transcription output (NarrativeLogs) |
| `edited_transcript` | TEXT | nullable | User-edited version |
| `ai_parse_status` | TEXT | DEFAULT 'pending' | `pending`, `processing`, `completed`, `failed`, `skipped` |
| `ai_parsed_at` | TEXT | nullable | |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Note:** Audio files live on the filesystem, not in the database. Only the path is stored.

---

### `ai_extracted_events`

AI-parsed chips from voice logs. Maps to the interactive categorization chips on Screen 4.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `voice_log_id` | TEXT | FK → voice_logs.id, NOT NULL | Source voice log |
| `observation_id` | TEXT | FK → observations.id, nullable | The observation created when user confirms this chip |
| `extracted_category` | TEXT | NOT NULL | AI-determined category |
| `extracted_value` | TEXT | NOT NULL | e.g., "Loud Noise", "Ritalin taken" |
| `confidence` | REAL | nullable | AI confidence score 0.0–1.0 |
| `user_action` | TEXT | DEFAULT 'pending' | `pending`, `confirmed`, `edited`, `deleted` |
| `user_edited_value` | TEXT | nullable | If user edited the chip |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

**Pipeline:** Voice audio → `voice_logs.raw_transcript` → AI parser creates `ai_extracted_events` rows → user sees chips on Screen 4 → confirmed chips create `observations` rows → `observation_id` is set on the chip.

---

### `observation_access`

Custom visibility grants. Used only when `observations.visibility_level = 'custom'`.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `observation_id` | TEXT | FK → observations.id, NOT NULL | |
| `user_id` | TEXT | FK → users.id, NOT NULL | User granted access |

**Constraints:** UNIQUE(`observation_id`, `user_id`)

---

## Domain 4: Medications

### `medication_templates`

Recurring medication definitions — the "what" and "when".

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `child_id` | TEXT | FK → children.id, NOT NULL | |
| `name` | TEXT | NOT NULL | Medication or supplement name |
| `dose` | TEXT | nullable | e.g., "5mg", "1 tablet" |
| `schedule` | TEXT | nullable | Free-text: "Morning with food" |
| `schedule_times` | TEXT | nullable | JSON array: `["08:00", "20:00"]` for reminder scheduling |
| `notes` | TEXT | nullable | |
| `is_active` | INTEGER | NOT NULL, DEFAULT 1 | Currently prescribed |
| `started_at` | TEXT | nullable | When started |
| `ended_at` | TEXT | nullable | When discontinued |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Changes from v1:** Renamed from `medications`. Added `schedule_times`, `notes`, `started_at`, `ended_at`.

---

### `medication_events`

Individual dose logs. Linked to both the medication template and a timeline observation.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `medication_id` | TEXT | FK → medication_templates.id, NOT NULL | Which medication |
| `child_id` | TEXT | FK → children.id, NOT NULL | Denormalized for query performance |
| `observation_id` | TEXT | FK → observations.id, nullable | Links to the timeline observation |
| `status` | TEXT | NOT NULL | `taken`, `missed`, `skipped`, `side_effect` |
| `taken_at` | TEXT | NOT NULL | When the dose was taken/missed |
| `dose_override` | TEXT | nullable | If different from template dose |
| `notes` | TEXT | nullable | e.g., "Side effect: drowsiness" |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Note:** Dual storage serves two purposes: `medication_events` enables structured adherence queries (% taken this week, correlation with behavior), while the linked `observation` puts it on the timeline alongside everything else.

---

## Domain 5: Reflections, Reminders, Reports

### `daily_reflections`

End-of-day "How did today feel?" (Wireframe 3, Workflow 2).

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `child_id` | TEXT | FK → children.id, NOT NULL | |
| `created_by` | TEXT | FK → users.id, nullable | |
| `reflection_date` | TEXT | NOT NULL | Date: `2026-06-04` |
| `rating` | TEXT | NOT NULL | `better_than_usual`, `typical`, `difficult` |
| `notes` | TEXT | nullable | Optional free-text |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Constraints:** UNIQUE(`child_id`, `reflection_date`) — one reflection per child per day.

**Note:** Separate from observations because this is about the day as a whole, not a point-in-time event. Useful for correlation: "difficult days" vs meltdown count, sleep hours, etc.

---

### `reminders`

Follow-up reminders, scheduled check-ins, and medication reminders (Screens 3, 9).

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `child_id` | TEXT | FK → children.id, NOT NULL | |
| `created_by` | TEXT | FK → users.id, nullable | |
| `source_observation_id` | TEXT | FK → observations.id, nullable | If created by "Save and Remind" |
| `reminder_type` | TEXT | NOT NULL | `follow_up`, `scheduled`, `medication` |
| `title` | TEXT | NOT NULL | e.g., "Check back: How is the meltdown?" |
| `scheduled_at` | TEXT | NOT NULL | When to fire |
| `repeat_rule` | TEXT | nullable | JSON: `{"frequency": "daily", "times": ["15:30"]}` |
| `is_completed` | INTEGER | NOT NULL, DEFAULT 0 | |
| `completed_at` | TEXT | nullable | |
| `is_active` | INTEGER | NOT NULL, DEFAULT 1 | Can be cancelled |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

**Use cases:**
1. "Save and Remind" (Screen 3) → `follow_up` type, `source_observation_id` set, fires 30 min later
2. Screen 9 check-ins → `scheduled` type, e.g., "Ask how school transition went at 3:30 PM"
3. Medication schedule → `medication` type, derived from `medication_templates.schedule_times`

---

### `reports`

Generated report metadata (Screen 6: Pediatrician Visit Planner).

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `child_id` | TEXT | FK → children.id, NOT NULL | |
| `created_by` | TEXT | FK → users.id, nullable | |
| `title` | TEXT | NOT NULL | |
| `date_range_start` | TEXT | NOT NULL | |
| `date_range_end` | TEXT | NOT NULL | |
| `category_filters` | TEXT | nullable | JSON array: `["behavior", "medication", "sleep"]` |
| `report_format` | TEXT | NOT NULL, DEFAULT 'pdf' | `pdf`, `csv`, `json` |
| `file_path` | TEXT | nullable | Local path to generated file |
| `cloud_url` | TEXT | nullable | If uploaded for sharing |
| `generated_at` | TEXT | nullable | |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| _sync columns_ | | | |

---

### `report_shares`

Shareable links for reports (Screen 6: "Generate Shareable Link").

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `report_id` | TEXT | FK → reports.id, NOT NULL | |
| `share_token` | TEXT | NOT NULL, UNIQUE | Secure random token in the URL |
| `expires_at` | TEXT | NOT NULL | |
| `access_count` | INTEGER | NOT NULL, DEFAULT 0 | |
| `max_accesses` | INTEGER | nullable | Optional access limit |
| `is_revoked` | INTEGER | NOT NULL, DEFAULT 0 | |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |

---

## Domain 6: System

### `consent_log` **[local-only]**

Append-only audit trail for privacy settings. Never synced.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `child_id` | TEXT | FK → children.id, nullable | Per-child or global |
| `user_id` | TEXT | FK → users.id, nullable | Who made the change |
| `setting` | TEXT | NOT NULL | e.g., `cloud_sync`, `anonymous_sharing`, `export_enabled` |
| `value` | TEXT | NOT NULL | e.g., `true`, `false` |
| `context` | TEXT | nullable | Why this change was made |
| `recorded_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| `device_id` | TEXT | nullable | |

**Rules:** Strictly append-only. Never UPDATE or DELETE rows. Current effective value = most recent row for a given `(child_id, setting)`.

---

### `sync_queue` **[local-only]**

Tracks pending sync operations. Never synced to cloud.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID |
| `table_name` | TEXT | NOT NULL | Which table needs syncing |
| `record_id` | TEXT | NOT NULL | UUID of the record to sync |
| `operation` | TEXT | NOT NULL | `insert`, `update`, `delete` |
| `payload` | TEXT | nullable | JSON snapshot of the record |
| `retry_count` | INTEGER | NOT NULL, DEFAULT 0 | |
| `last_error` | TEXT | nullable | |
| `created_at` | TEXT | NOT NULL, DEFAULT CURRENT_TIMESTAMP | |
| `processed_at` | TEXT | nullable | NULL until processed |

**How it works:** When any synced table is modified locally, a row is inserted here. The sync service processes the queue when connectivity is available. After success, `processed_at` is set. Old processed rows are pruned periodically.

---

## Category Values

The `observations.category` and `tag_definitions.category` columns use these values:

| Value | Description | Quick Tap Color |
|---|---|---|
| `behavior` | Behavioral observation (positive or challenging) | Purple |
| `food` | Meal, snack, or drink | Teal |
| `medication` | Medication or supplement | Amber |
| `emotion` | Emotional state | Pink |
| `sleep` | Sleep quality and duration | — |
| `sensory` | Sensory load observations | — |
| `transitions` | Activity/environment transitions | — |
| `successes` | Positive achievements and milestones | Green |
| `milestone` | Developmental milestone | — |
| `trigger` | (tag_definitions only) Trigger tags for incidents | — |
| `other` | Anything that doesn't fit the above | — |

---

## Schema SQL (SQLite)

```sql
-- ============================================================
-- Domain 1: Identity & Access
-- ============================================================

CREATE TABLE users (
    id                TEXT PRIMARY KEY,
    email             TEXT UNIQUE,
    display_name      TEXT NOT NULL,
    user_type         TEXT NOT NULL DEFAULT 'parent'
                      CHECK(user_type IN ('parent','grandparent','caregiver','therapist','doctor','teacher','other')),
    avatar_url        TEXT,
    auth_provider_id  TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_users_email ON users(email) WHERE email IS NOT NULL;
CREATE INDEX idx_users_auth_provider ON users(auth_provider_id) WHERE auth_provider_id IS NOT NULL;

-- ---

CREATE TABLE families (
    id                TEXT PRIMARY KEY,
    family_name       TEXT NOT NULL,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1
);

-- ---

CREATE TABLE family_members (
    id                TEXT PRIMARY KEY,
    family_id         TEXT NOT NULL REFERENCES families(id) ON DELETE CASCADE,
    user_id           TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role              TEXT NOT NULL CHECK(role IN ('owner','editor','viewer')),
    joined_at         TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1,
    UNIQUE(family_id, user_id)
);

CREATE INDEX idx_fm_family ON family_members(family_id);
CREATE INDEX idx_fm_user ON family_members(user_id);

-- ---

CREATE TABLE family_invites (
    id                TEXT PRIMARY KEY,
    family_id         TEXT NOT NULL REFERENCES families(id) ON DELETE CASCADE,
    invited_by        TEXT NOT NULL REFERENCES users(id),
    invite_token      TEXT NOT NULL UNIQUE,
    role              TEXT NOT NULL CHECK(role IN ('owner','editor','viewer')),
    expires_at        TEXT NOT NULL,
    accepted_at       TEXT,
    accepted_by       TEXT REFERENCES users(id),
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invites_token ON family_invites(invite_token);

-- ============================================================
-- Domain 2: Child Profiles & Tags
-- ============================================================

CREATE TABLE children (
    id                TEXT PRIMARY KEY,
    family_id         TEXT NOT NULL REFERENCES families(id) ON DELETE CASCADE,
    display_name      TEXT NOT NULL,
    birth_year_month  TEXT,
    avatar_url        TEXT,
    profile_notes     TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_children_family ON children(family_id);

-- ---

CREATE TABLE tag_definitions (
    id                TEXT PRIMARY KEY,
    category          TEXT NOT NULL
                      CHECK(category IN ('behavior','food','medication','emotion','sleep','sensory','transitions','successes','trigger','other')),
    name              TEXT NOT NULL,
    is_system         INTEGER NOT NULL DEFAULT 1,
    child_id          TEXT REFERENCES children(id) ON DELETE CASCADE,
    family_id         TEXT REFERENCES families(id) ON DELETE CASCADE,
    display_order     INTEGER DEFAULT 0,
    color             TEXT,
    icon              TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1,
    UNIQUE(category, name, child_id)
);

CREATE INDEX idx_tags_category ON tag_definitions(category);
CREATE INDEX idx_tags_child ON tag_definitions(child_id) WHERE child_id IS NOT NULL;

-- ---

CREATE TABLE child_tags (
    id                TEXT PRIMARY KEY,
    child_id          TEXT NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    tag_id            TEXT NOT NULL REFERENCES tag_definitions(id) ON DELETE CASCADE,
    is_enabled        INTEGER NOT NULL DEFAULT 1,
    display_order     INTEGER DEFAULT 0,
    UNIQUE(child_id, tag_id)
);

CREATE INDEX idx_child_tags_child ON child_tags(child_id);

-- ============================================================
-- Domain 3: Observations & Logging
-- ============================================================

CREATE TABLE observations (
    id                    TEXT PRIMARY KEY,
    child_id              TEXT NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    created_by            TEXT REFERENCES users(id),
    occurred_at           TEXT NOT NULL,
    entry_type            TEXT NOT NULL
                          CHECK(entry_type IN ('quick_tap','incident','voice','lock_screen','manual')),
    category              TEXT NOT NULL
                          CHECK(category IN ('behavior','food','medication','emotion','sleep','sensory','transitions','successes','milestone','other')),
    title                 TEXT,
    notes                 TEXT,
    incident_data         TEXT,  -- JSON
    visibility_level      TEXT NOT NULL DEFAULT 'family'
                          CHECK(visibility_level IN ('family','parents','clinical','private','custom')),
    is_partial            INTEGER NOT NULL DEFAULT 0,
    voice_log_id          TEXT REFERENCES voice_logs(id),
    parent_observation_id TEXT REFERENCES observations(id),
    created_at            TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at            TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted            INTEGER NOT NULL DEFAULT 0,
    sync_status           TEXT NOT NULL DEFAULT 'pending'
                          CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at        TEXT,
    device_id             TEXT,
    version               INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_obs_child_occurred ON observations(child_id, occurred_at);
CREATE INDEX idx_obs_child_category ON observations(child_id, category);
CREATE INDEX idx_obs_entry_type ON observations(entry_type);
CREATE INDEX idx_obs_created_by ON observations(created_by) WHERE created_by IS NOT NULL;
CREATE INDEX idx_obs_voice_log ON observations(voice_log_id) WHERE voice_log_id IS NOT NULL;
CREATE INDEX idx_obs_parent ON observations(parent_observation_id) WHERE parent_observation_id IS NOT NULL;
CREATE INDEX idx_obs_visibility ON observations(visibility_level);

-- ---

CREATE TABLE observation_tags (
    observation_id    TEXT NOT NULL REFERENCES observations(id) ON DELETE CASCADE,
    tag_id            TEXT NOT NULL REFERENCES tag_definitions(id) ON DELETE CASCADE,
    PRIMARY KEY(observation_id, tag_id)
);

CREATE INDEX idx_obs_tags_tag ON observation_tags(tag_id);

-- ---

CREATE TABLE voice_logs (
    id                TEXT PRIMARY KEY,
    observation_id    TEXT NOT NULL UNIQUE REFERENCES observations(id) ON DELETE CASCADE,
    audio_file_path   TEXT,
    audio_duration_secs INTEGER,
    raw_transcript    TEXT,
    edited_transcript TEXT,
    ai_parse_status   TEXT NOT NULL DEFAULT 'pending'
                      CHECK(ai_parse_status IN ('pending','processing','completed','failed','skipped')),
    ai_parsed_at      TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_vl_observation ON voice_logs(observation_id);
CREATE INDEX idx_vl_parse_status ON voice_logs(ai_parse_status);

-- ---

CREATE TABLE ai_extracted_events (
    id                TEXT PRIMARY KEY,
    voice_log_id      TEXT NOT NULL REFERENCES voice_logs(id) ON DELETE CASCADE,
    observation_id    TEXT REFERENCES observations(id),
    extracted_category TEXT NOT NULL,
    extracted_value   TEXT NOT NULL,
    confidence        REAL,
    user_action       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(user_action IN ('pending','confirmed','edited','deleted')),
    user_edited_value TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_aie_voice_log ON ai_extracted_events(voice_log_id);

-- ---

CREATE TABLE observation_access (
    id                TEXT PRIMARY KEY,
    observation_id    TEXT NOT NULL REFERENCES observations(id) ON DELETE CASCADE,
    user_id           TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE(observation_id, user_id)
);

-- ============================================================
-- Domain 4: Medications
-- ============================================================

CREATE TABLE medication_templates (
    id                TEXT PRIMARY KEY,
    child_id          TEXT NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    name              TEXT NOT NULL,
    dose              TEXT,
    schedule          TEXT,
    schedule_times    TEXT,  -- JSON array: ["08:00", "20:00"]
    notes             TEXT,
    is_active         INTEGER NOT NULL DEFAULT 1,
    started_at        TEXT,
    ended_at          TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_medtpl_child ON medication_templates(child_id);
CREATE INDEX idx_medtpl_active ON medication_templates(child_id, is_active);

-- ---

CREATE TABLE medication_events (
    id                TEXT PRIMARY KEY,
    medication_id     TEXT NOT NULL REFERENCES medication_templates(id) ON DELETE CASCADE,
    child_id          TEXT NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    observation_id    TEXT REFERENCES observations(id),
    status            TEXT NOT NULL CHECK(status IN ('taken','missed','skipped','side_effect')),
    taken_at          TEXT NOT NULL,
    dose_override     TEXT,
    notes             TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_medevt_child_taken ON medication_events(child_id, taken_at);
CREATE INDEX idx_medevt_medication ON medication_events(medication_id);
CREATE INDEX idx_medevt_status ON medication_events(status);

-- ============================================================
-- Domain 5: Reflections, Reminders, Reports
-- ============================================================

CREATE TABLE daily_reflections (
    id                TEXT PRIMARY KEY,
    child_id          TEXT NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    created_by        TEXT REFERENCES users(id),
    reflection_date   TEXT NOT NULL,
    rating            TEXT NOT NULL CHECK(rating IN ('better_than_usual','typical','difficult')),
    notes             TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1,
    UNIQUE(child_id, reflection_date)
);

CREATE INDEX idx_dr_child_date ON daily_reflections(child_id, reflection_date);

-- ---

CREATE TABLE reminders (
    id                    TEXT PRIMARY KEY,
    child_id              TEXT NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    created_by            TEXT REFERENCES users(id),
    source_observation_id TEXT REFERENCES observations(id),
    reminder_type         TEXT NOT NULL CHECK(reminder_type IN ('follow_up','scheduled','medication')),
    title                 TEXT NOT NULL,
    scheduled_at          TEXT NOT NULL,
    repeat_rule           TEXT,  -- JSON
    is_completed          INTEGER NOT NULL DEFAULT 0,
    completed_at          TEXT,
    is_active             INTEGER NOT NULL DEFAULT 1,
    created_at            TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted            INTEGER NOT NULL DEFAULT 0,
    sync_status           TEXT NOT NULL DEFAULT 'pending'
                          CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at        TEXT,
    device_id             TEXT,
    version               INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_rem_child_scheduled ON reminders(child_id, scheduled_at);
CREATE INDEX idx_rem_active ON reminders(is_active, scheduled_at) WHERE is_active = 1;
CREATE INDEX idx_rem_source ON reminders(source_observation_id) WHERE source_observation_id IS NOT NULL;

-- ---

CREATE TABLE reports (
    id                TEXT PRIMARY KEY,
    child_id          TEXT NOT NULL REFERENCES children(id) ON DELETE CASCADE,
    created_by        TEXT REFERENCES users(id),
    title             TEXT NOT NULL,
    date_range_start  TEXT NOT NULL,
    date_range_end    TEXT NOT NULL,
    category_filters  TEXT,  -- JSON array
    report_format     TEXT NOT NULL DEFAULT 'pdf' CHECK(report_format IN ('pdf','csv','json')),
    file_path         TEXT,
    cloud_url         TEXT,
    generated_at      TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_deleted        INTEGER NOT NULL DEFAULT 0,
    sync_status       TEXT NOT NULL DEFAULT 'pending'
                      CHECK(sync_status IN ('pending','synced','conflict')),
    last_synced_at    TEXT,
    device_id         TEXT,
    version           INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX idx_reports_child ON reports(child_id);

-- ---

CREATE TABLE report_shares (
    id                TEXT PRIMARY KEY,
    report_id         TEXT NOT NULL REFERENCES reports(id) ON DELETE CASCADE,
    share_token       TEXT NOT NULL UNIQUE,
    expires_at        TEXT NOT NULL,
    access_count      INTEGER NOT NULL DEFAULT 0,
    max_accesses      INTEGER,
    is_revoked        INTEGER NOT NULL DEFAULT 0,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_rs_token ON report_shares(share_token);

-- ============================================================
-- Domain 6: System (local-only)
-- ============================================================

CREATE TABLE consent_log (
    id                TEXT PRIMARY KEY,
    child_id          TEXT REFERENCES children(id) ON DELETE CASCADE,
    user_id           TEXT REFERENCES users(id),
    setting           TEXT NOT NULL,
    value             TEXT NOT NULL,
    context           TEXT,
    recorded_at       TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    device_id         TEXT
);

CREATE INDEX idx_consent_child_setting ON consent_log(child_id, setting);

-- ---

CREATE TABLE sync_queue (
    id                TEXT PRIMARY KEY,
    table_name        TEXT NOT NULL,
    record_id         TEXT NOT NULL,
    operation         TEXT NOT NULL CHECK(operation IN ('insert','update','delete')),
    payload           TEXT,  -- JSON snapshot
    retry_count       INTEGER NOT NULL DEFAULT 0,
    last_error        TEXT,
    created_at        TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    processed_at      TEXT
);

CREATE INDEX idx_sq_pending ON sync_queue(processed_at) WHERE processed_at IS NULL;
CREATE INDEX idx_sq_table_record ON sync_queue(table_name, record_id);
```

---

## Sync Strategy

### Which tables sync

| Synced | Local-only |
|---|---|
| users, families, family_members, children, tag_definitions, child_tags, observations, observation_tags, voice_logs, ai_extracted_events, observation_access, medication_templates, medication_events, daily_reflections, reminders, reports | consent_log, sync_queue, family_invites, report_shares |

### Conflict resolution

- **Default:** Last-write-wins using `updated_at` timestamp
- **Detection:** `version` column — if server version differs from what client expected, mark `sync_status = 'conflict'`
- **Resolution:** For observations, conflicts are surfaced to the user rather than silently overwritten

### Offline-first flow

1. All writes go to local SQLite first
2. A row is inserted into `sync_queue` for each change
3. Sync service processes the queue when connectivity is available
4. On success: update `sync_status = 'synced'`, set `last_synced_at`, mark `sync_queue` row as processed
5. On conflict: set `sync_status = 'conflict'` for user resolution

---

## Key Query Patterns

| Query | Tables | Index |
|---|---|---|
| Timeline for child on a date | observations | `idx_obs_child_occurred` |
| Filter by category | observations | `idx_obs_child_category` |
| Daily summary counts | observations | `idx_obs_child_occurred` + GROUP BY |
| Medication adherence % | medication_events | `idx_medevt_child_taken` + `idx_medevt_status` |
| Sleep vs meltdown correlation | observations (category filter) | `idx_obs_child_category` |
| Difficult days pattern | daily_reflections | `idx_dr_child_date` |
| Pending reminders | reminders | `idx_rem_active` |
| Unsynced records | any synced table | filter on `sync_status = 'pending'` |
| Voice logs awaiting AI parse | voice_logs | `idx_vl_parse_status` |
| Tag frequency analysis | observation_tags JOIN tag_definitions | `idx_obs_tags_tag` |
| Visibility-filtered query | observations + observation_access | `idx_obs_visibility` + UNIQUE on observation_access |

---

## Screen ↔ Schema Coverage

| Screen | Tables Used |
|---|---|
| 1. Lock Screen Quick Entry | observations (entry_type=lock_screen, created_by=NULL), observation_tags |
| 2. Quick Capture Home | children, observations (entry_type=quick_tap), observation_tags, tag_definitions, child_tags |
| 3. Stress Screen | observations (entry_type=incident, incident_data JSON), observation_tags, reminders (Save and Remind) |
| 4. Voice Transcriber | observations (entry_type=voice), voice_logs, ai_extracted_events |
| 5. Daily Timeline | observations (all types), observation_tags, tag_definitions |
| 6. Pediatrician Visit Planner | reports, report_shares, observations (date range query) |
| 7. Insights Dashboard | observations, observation_tags, daily_reflections, medication_events |
| 8. Child Profile & Settings | children, tag_definitions (custom), child_tags, medication_templates |
| 9. Notification & Reminders | reminders |
| 10. Local-First & Sync Settings | sync_queue, consent_log, family_invites |

---

## Migration from v1

The v1 schema had 4 tables with INTEGER auto-increment PKs. Migration steps:

1. **Create a default family** — insert into `families`, create a `users` row for the local user, link via `family_members`
2. **children** — add `family_id`, rename `name` → `display_name`, convert `date_of_birth` to `birth_year_month` (extract year-month), generate UUID `id`, add sync columns
3. **events → observations** — rename table, map column names (`timestamp` → `occurred_at`, `raw_text` → `notes`, `input_method` → `entry_type`), expand category enum, generate UUID PKs, add new columns with defaults
4. **medications → medication_templates** — rename table, add new columns, generate UUID PKs
5. **consent_log** — add `user_id`, `context` columns, generate UUID PKs
6. **Create new tables** — all tables not in v1

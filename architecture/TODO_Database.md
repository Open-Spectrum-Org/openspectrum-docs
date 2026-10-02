# Database TODO — Missing Tables

These 9 tables are defined in `db-schema.md` but not yet registered in Dexie (`database.ts`) or typed in `types/database.ts`.

Defer until after initial app testing.

## Identity & Access

- [ ] `family_invites` — co-parent invite tokens (Screen 10)

## Observations & Logging

- [ ] `observation_access` — custom visibility grants (used when `visibility_level = 'custom'`)

## Medications

- [ ] `medication_templates` — recurring medication definitions
- [ ] `medication_events` — individual dose logs

## Reflections, Reminders, Reports

- [ ] `reminders` — follow-up reminders, scheduled check-ins, medication reminders (Screens 3, 9)
- [ ] `reports` — generated report metadata (Screen 6)
- [ ] `report_shares` — shareable links for reports (Screen 6)

## System (local-only)

- [ ] `consent_log` — append-only privacy settings audit trail
- [ ] `sync_queue` — pending sync operations queue

## Notes

- `reports.ts` query file is safe — despite the name, it only queries `observations`, `observation_tags`, `tag_definitions`, and `daily_reflections` (all implemented). It generates analytics data, not `reports` table records.
- Consider calling `navigator.storage.persist()` at app init to prevent IndexedDB eviction under storage pressure.

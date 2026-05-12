# Privacy Design — OpenSpectrum

**Status**: Draft
**Last updated**: 2026-05-12

> OpenSpectrum handles sensitive health data about children. Privacy is not a feature — it is a foundational requirement. This document must be reviewed before any data-handling feature is built.

---

## Core Privacy Commitments

1. **Local by default**: All data stays on the parent's device unless they explicitly choose otherwise
2. **No telemetry**: No usage analytics, crash reporting, or behavioral tracking in MVP
3. **No accounts required**: Parents can use the app without creating an account or providing an email
4. **You own your data**: Parents can export all their data at any time and delete it completely
5. **Transparent by design**: No hidden data flows — every data movement is explicit and user-initiated

---

## Data Storage Architecture

### MVP: Local-only SQLite

```
Parent's Device
└── openspectrum.db (SQLite)
    ├── children
    ├── events
    ├── medications
    └── consent_log
```

All data is stored in a single SQLite file on the device. There is no cloud sync, no external API calls, and no data leaving the device unless the user explicitly exports it.

**Docker deployment note**: When running via Docker, the SQLite database lives in a named Docker volume (`openspectrum-data`). It is local to the machine running Docker.

### Future: Opt-in Cloud Sync

If/when cloud sync is added, it will be:
- Explicitly opt-in (not enabled by default)
- Encrypted in transit (HTTPS/TLS only)
- Encrypted at rest on any server
- Documented with a full data processing agreement
- Governed by the consent log table

---

## Data Classification

| Data | Sensitivity | Location | Notes |
|---|---|---|---|
| Child name | High | Local only | Optional — can use a nickname |
| Date of birth | High | Local only | Optional — used for age calculations only |
| Profile notes | High | Local only | Free text, may contain diagnosis info |
| Event logs | High | Local only | Core app data |
| Medication records | High | Local only | Medical data |
| Consent settings | Medium | Local only | Audit trail of user choices |

**No "low sensitivity" data exists in OpenSpectrum.** All data is treated as sensitive.

---

## No Telemetry Policy (MVP)

The MVP will not include:
- Error reporting to external services (Sentry, Bugsnag, etc.)
- Analytics (Google Analytics, Mixpanel, etc.)
- Feature flag or A/B testing services
- Any network request made without user action

If telemetry is proposed in the future, it must be:
1. Fully opt-in (default: off)
2. Clearly documented in the privacy policy
3. Architecturally isolated from child data
4. Subject to a team vote and ADR

---

## Consent Log Design

Every privacy-sensitive setting toggle is recorded in the `consent_log` table:

```sql
INSERT INTO consent_log (child_id, setting, value)
VALUES (1, 'anonymous_sharing', 'false');
```

### Settings tracked in MVP

| Setting key | Values | Description |
|---|---|---|
| `anonymous_sharing` | `true`/`false` | Opt-in to anonymized community dataset (future) |
| `export_enabled` | `true`/`false` | Whether CSV export has been used |

### Design notes

- Append-only: rows are never updated or deleted
- This creates an audit trail: "consent was given at time X, revoked at time Y"
- The current effective value is the most recent row for that `(child_id, setting)` pair

---

## Encryption

### MVP (v0.1)

- SQLite database is **not encrypted** at the file level in MVP
- Rationale: File-level encryption adds significant setup complexity for self-hosted deployments
- Mitigation: The app is local-only, so the attack surface is the physical device

### Phase 2 Target

| Layer | Approach |
|---|---|
| At rest (local) | SQLCipher (SQLite encryption extension) — transparent, no schema changes |
| At rest (server, if added) | AES-256 encryption of the SQLite/PostgreSQL data volume |
| In transit | HTTPS/TLS enforced for all API calls if network features are added |
| Export files | Option to export as encrypted zip (password-protected) |

### Key management (future)

If at-rest encryption is added:
- The encryption key must be derived from a user-provided passphrase (PBKDF2 or Argon2)
- The key must never be stored unencrypted on disk
- Recovery must be through the passphrase only (no "forgot password" that weakens encryption)

---

## Anonymization Pipeline (Future)

For any future community data sharing feature:

1. **Consent required**: Parent explicitly opts in per child
2. **Anonymization steps**:
   - Remove child name, date of birth, and profile notes
   - Replace child_id with a random, non-reversible token (not a hash of the real ID)
   - Generalize dates to week-of-year (not exact date)
   - Strip free-text fields or run through PII detection before sharing
3. **Audit**: Every anonymized record can be traced back to the consent log entry
4. **Right to withdraw**: Parent can revoke consent and request deletion from the dataset

---

## Self-Hosting and Docker

Docker deployment is a privacy feature: it lets technically capable parents run OpenSpectrum on their own infrastructure, with full control over the data, rather than using a hosted service.

For self-hosters:
- The Docker volume should be stored on an encrypted disk
- Reverse proxy (e.g., Nginx) should be configured with HTTPS if exposed on a network
- Documentation for secure self-hosting will be provided

---

## What We Will Never Do

- Sell user data
- Share individual-level data without explicit consent
- Use child health data for advertising
- Store data on external servers without user knowledge
- Make it difficult to export or delete data

---

## Privacy Review Process

Before any feature that touches data storage, export, or network access is merged:

1. The PR description must include a "Privacy considerations" section
2. At least one maintainer must review it with privacy in mind
3. If the feature adds a new data field, the db-schema.md must be updated
4. If the feature adds a new network call, this document must be updated

---

## Open Questions

- [ ] Should the child's name be optional (allow nicknames / pseudonyms)?
- [ ] Should we add a "wipe all data" button in v0.1?
- [ ] What's the right passphrase-based encryption UX for non-technical parents?

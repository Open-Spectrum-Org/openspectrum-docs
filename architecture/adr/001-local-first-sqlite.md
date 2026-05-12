# ADR-001: Local-first SQLite for MVP storage

**Date**: 2026-05-12
**Status**: Accepted
**Deciders**: OpenSpectrum founding team

---

## Context

OpenSpectrum tracks sensitive health data about children. Parents need to trust that their data is private and under their control. We need to choose a storage strategy for the MVP.

Options considered:

1. **Cloud database (PostgreSQL on a hosted server)** — Easier to access from multiple devices, but data leaves the parent's device and requires account management
2. **Local SQLite** — Data stays on the device, no accounts needed, simple to set up, works offline
3. **IndexedDB (browser-native)** — Stays in the browser, but hard to export, no backup, limited query capabilities
4. **File-based (JSON/CSV)** — Simplest possible, but no real querying, concurrency issues

## Decision

Use **SQLite via SQLAlchemy** for MVP storage.

## Rationale

- Directly supports the "local-first, privacy-first" mission statement
- SQLite is embedded in the backend process — no separate database server needed
- SQLAlchemy makes it straightforward to migrate to PostgreSQL when multi-user support is added
- Works fully offline — no dependency on network connectivity
- Parents can back up their data by copying one file
- No authentication or account management required for MVP

## Consequences

**Positive:**
- Maximum privacy: data never leaves the device by default
- Simple developer setup (no external DB service)
- Easy backup and restore (single file)
- Trust-building with early users

**Negative:**
- Single-device only for MVP — no sync across devices
- SQLite has concurrency limits (not relevant for single-user MVP)
- When cloud sync is added, a migration path to PostgreSQL will need to be designed

## Revisit Criteria

This decision should be revisited when:
- Multi-device sync is a user requirement
- The team begins building self-hosted multi-user deployments

# API Design — OpenSpectrum MVP

**Status**: Draft
**Last updated**: 2026-05-12
**Related ADR**: [ADR-002 — FastAPI backend](./adr/002-fastapi-backend.md)

---

## Design Principles

1. **RESTful**: Standard HTTP methods and status codes
2. **Local-first**: API runs on localhost for MVP; no authentication required for single-user
3. **Self-documenting**: FastAPI generates OpenAPI docs at `/docs`
4. **Privacy-aware**: No endpoints expose data from other children; child_id is always scoped

---

## Base URL

```
http://localhost:8000
```

Interactive docs: `http://localhost:8000/docs`

---

## Endpoints

### Children

#### `GET /children`

List all child profiles.

**Response `200`:**
```json
[
  {
    "id": 1,
    "name": "Alex",
    "date_of_birth": "2019-03-15",
    "profile_notes": "Diagnosed ASD level 2. Therapist: Dr. Patel.",
    "created_at": "2026-01-10T09:00:00"
  }
]
```

---

#### `POST /children`

Create a new child profile.

**Request body:**
```json
{
  "name": "Alex",
  "date_of_birth": "2019-03-15",
  "profile_notes": "Optional notes"
}
```

**Response `201`:**
```json
{
  "id": 1,
  "name": "Alex",
  "date_of_birth": "2019-03-15",
  "profile_notes": "Optional notes",
  "created_at": "2026-05-12T10:30:00"
}
```

---

#### `GET /children/{child_id}`

Get a single child profile.

**Response `200`:** Same shape as the list item above.
**Response `404`:** `{"detail": "Child not found"}`

---

### Events

#### `POST /events`

Log a new event.

**Request body:**
```json
{
  "child_id": 1,
  "timestamp": "2026-05-12T08:30:00",
  "category": "medication",
  "raw_text": "Gave 5mg Ritalin with breakfast",
  "input_method": "text"
}
```

- `timestamp`: When the event occurred (ISO 8601). Defaults to current time if omitted.
- `category`: One of `food`, `medication`, `behavior`, `milestone`, `other`
- `input_method`: One of `text`, `voice`. Defaults to `text`.

**Response `201`:**
```json
{
  "id": 42,
  "child_id": 1,
  "timestamp": "2026-05-12T08:30:00",
  "category": "medication",
  "raw_text": "Gave 5mg Ritalin with breakfast",
  "parsed_data": null,
  "input_method": "text",
  "created_at": "2026-05-12T10:35:00"
}
```

---

#### `GET /events`

Retrieve events with optional filters.

**Query parameters:**

| Parameter | Type | Description |
|---|---|---|
| `child_id` | integer | **Required.** Filter by child. |
| `date_from` | date (YYYY-MM-DD) | Filter events on or after this date |
| `date_to` | date (YYYY-MM-DD) | Filter events on or before this date |
| `category` | string | Filter by category |
| `limit` | integer | Max results (default: 100, max: 500) |
| `offset` | integer | Pagination offset (default: 0) |

**Example:**
```
GET /events?child_id=1&date_from=2026-05-01&date_to=2026-05-12&category=medication
```

**Response `200`:**
```json
{
  "total": 12,
  "events": [
    {
      "id": 42,
      "child_id": 1,
      "timestamp": "2026-05-12T08:30:00",
      "category": "medication",
      "raw_text": "Gave 5mg Ritalin with breakfast",
      "parsed_data": null,
      "input_method": "text",
      "created_at": "2026-05-12T10:35:00"
    }
  ]
}
```

---

#### `GET /events/{event_id}`

Get a single event by ID.

**Response `200`:** Single event object (same shape as above).
**Response `404`:** `{"detail": "Event not found"}`

---

#### `DELETE /events/{event_id}`

Delete an event.

**Response `204`:** No content.
**Response `404`:** `{"detail": "Event not found"}`

---

### Insights

#### `GET /insights/correlations`

Return basic correlation data for a child's events.

**Query parameters:**

| Parameter | Type | Description |
|---|---|---|
| `child_id` | integer | **Required.** |
| `days` | integer | Lookback window in days (default: 30) |

**Response `200`:**
```json
{
  "child_id": 1,
  "period_days": 30,
  "events_per_day_by_category": {
    "2026-05-12": {"food": 3, "medication": 2, "behavior": 1},
    "2026-05-11": {"food": 2, "medication": 2, "behavior": 0}
  },
  "category_totals": {
    "food": 45,
    "medication": 28,
    "behavior": 12,
    "milestone": 2,
    "other": 1
  }
}
```

---

### Export

#### `GET /export`

Export event data for a child.

**Query parameters:**

| Parameter | Type | Description |
|---|---|---|
| `child_id` | integer | **Required.** |
| `format` | string | `csv` or `json` (default: `csv`) |
| `date_from` | date | Optional start date |
| `date_to` | date | Optional end date |

**Response `200`:**
- For `csv`: `Content-Type: text/csv`, file download
- For `json`: `Content-Type: application/json`

**CSV columns:** `id, timestamp, category, raw_text, input_method, created_at`

---

### Medications

#### `GET /children/{child_id}/medications`

List active medications for a child.

**Response `200`:**
```json
[
  {
    "id": 1,
    "child_id": 1,
    "name": "Ritalin",
    "dose": "5mg",
    "schedule": "Morning with food",
    "active": true,
    "created_at": "2026-01-15T09:00:00"
  }
]
```

---

#### `POST /children/{child_id}/medications`

Add a medication.

**Request body:**
```json
{
  "name": "Ritalin",
  "dose": "5mg",
  "schedule": "Morning with food"
}
```

**Response `201`:** Medication object.

---

### Health Check

#### `GET /health`

Liveness check for the API.

**Response `200`:**
```json
{"status": "ok", "version": "0.1.0"}
```

---

## Error Responses

All errors follow this format:

```json
{
  "detail": "Human-readable error message"
}
```

| Status | Meaning |
|---|---|
| `400` | Bad request (invalid input) |
| `404` | Resource not found |
| `422` | Validation error (FastAPI default for malformed request) |
| `500` | Server error |

---

## Versioning

No API versioning for MVP. If breaking changes are needed, a `/v2/` prefix will be introduced.

---

## Open Questions

- [ ] Should `DELETE /events` be soft-delete (set a `deleted_at` flag) for auditability?
- [ ] Should `POST /events` accept a batch array for bulk import?
- [ ] Authentication strategy when multi-user is added (JWT? Local session?)

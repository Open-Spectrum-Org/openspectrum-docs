# ADR-002: Python + FastAPI for backend

**Date**: 2026-05-12
**Status**: Accepted
**Deciders**: OpenSpectrum founding team

---

## Context

We need a backend framework for the REST API. The backend will serve data from SQLite, handle event logging, and eventually run data analysis and correlation queries.

Options considered:

1. **Python + FastAPI**
2. **Python + Flask**
3. **Node.js + Express**
4. **No separate backend (browser-only with IndexedDB)**

## Decision

Use **Python with FastAPI**.

## Rationale

- **Data science alignment**: Python is the dominant language for data analysis and ML. As OpenSpectrum grows toward correlation detection and pattern recognition, Python is the right foundation.
- **Auto-generated API docs**: FastAPI generates interactive OpenAPI docs (`/docs`) automatically — helpful for contributors and for documenting the API.
- **Pydantic validation**: Request/response validation is built-in and explicit, reducing bugs around malformed data.
- **Strong async support**: FastAPI is async-native, which will matter when data operations become heavier.
- **Flask**: Simpler, but lacks auto-docs and Pydantic integration. Adds work we'd have to do manually.
- **Node.js**: Viable, but the data science community overlap is with Python, not JavaScript.
- **No backend**: Works for a pure client-side app, but SQLite access from a browser requires WASM (complex) and loses server-side query power.

## Consequences

**Positive:**
- Natural home for future data analysis features
- Auto-generated API docs lower the contribution barrier
- Strong Python ecosystem for data processing (pandas, numpy, scipy)

**Negative:**
- Two languages in the project (Python backend, TypeScript frontend) — some contributors may only know one
- Python setup (virtual environments) can be confusing for beginners (mitigated by Docker)

## Revisit Criteria

Revisit if:
- The team grows strongly toward JavaScript/Node.js expertise
- A browser-only (no backend) architecture becomes desirable for simpler deployment

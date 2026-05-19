# Wireframes

This directory contains wireframes and UI mockups for OpenSpectrum.

## Status

**To be created.** Wireframes are a Phase 1 parallel workstream (Track B — UX).

## MVP Screens Needed

1. **Quick Log screen** — voice button + category chips + text field
2. **Timeline view** — chronological events, filterable by category
3. **Simple dashboard** — 2-3 charts (events over time, category breakdown)
4. **Settings / Privacy screen** — consent toggles, export, data management

## Suggested Tools

- **Figma** (free tier) — collaborative, shareable, good for detailed mockups
- **Excalidraw** (free, open source) — fast, hand-drawn feel, great for quick sketches
- **ASCII/Markdown mockups** — lowest barrier to contribution, works in any editor

## Contributing Wireframes

1. Create your wireframe using any tool
2. Export as PNG or PDF (or include as ASCII in a `.md` file)
3. Name files clearly: `01-quick-log.png`, `02-timeline-view.png`, etc.
4. Open a PR or post in GitHub Discussions for feedback before finalizing

## Key UX Principles

From the MVP design goals:
- **Fast**: Log an event in under 30 seconds
- **Minimal decisions per screen**: Reduce cognitive load for stressed parents
- **Mobile-first**: Most logging will happen on a phone, in the moment
- **Accessible**: High contrast, large touch targets, screen reader support

## ASCII Mockup: Quick Log Screen (Draft)

```
┌────────────────────────────────┐
│  OpenSpectrum                  │
│  Alex ▼                        │
├────────────────────────────────┤
│                                │
│  What happened?                │
│  ┌──────────────────────────┐  │
│  │  Type or speak...        │  │
│  └──────────────────────────┘  │
│                    🎤 [Voice]  │
│                                │
│  Category:                     │
│  [Food] [Medication] [Behavior]│
│  [Milestone] [Other]           │
│                                │
│  When: [Now ▼]                 │
│                                │
│  ┌──────────────────────────┐  │
│  │        Log Event         │  │
│  └──────────────────────────┘  │
│                                │
└────────────────────────────────┘
```

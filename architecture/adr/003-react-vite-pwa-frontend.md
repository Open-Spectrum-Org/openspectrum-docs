# ADR-003: React + Vite + TypeScript as PWA frontend

**Date**: 2026-05-12
**Status**: Accepted
**Deciders**: OpenSpectrum founding team

---

## Context

We need a frontend framework. The app should work on mobile and desktop, without requiring app store distribution, and support offline use.

Options considered:

1. **React + Vite + TypeScript (PWA)**
2. **React Native** (mobile app)
3. **Flutter** (cross-platform app)
4. **Vue.js**
5. **Plain HTML/CSS/JS**

## Decision

Use **React + Vite + TypeScript**, configured as a Progressive Web App (PWA).

## Rationale

- **Largest contributor pool**: React is the most widely used frontend framework. Choosing React maximizes the number of potential open-source contributors.
- **PWA over native apps**: A PWA runs in the browser, requires no app store account, can be "installed" on any device's home screen, and works offline. App stores add review delays, distribution friction, and platform lock-in — all contrary to an open-source local-first ethos.
- **Vite**: Significantly faster development server than Create React App. Better developer experience.
- **TypeScript**: Catches bugs at compile time, especially useful for a small team without extensive test coverage.
- **Vue.js**: Good framework, but smaller contributor pool than React.
- **Plain HTML/JS**: Would work for MVP but scales poorly and discourages contribution from developers comfortable with modern frameworks.
- **React Native / Flutter**: App store complexity, higher contribution barrier, harder to self-host.

## Consequences

**Positive:**
- Works on all devices without an app store
- Offline capability via service worker
- Large contributor pool familiar with React
- TypeScript catches integration errors between frontend and backend schemas

**Negative:**
- PWA has some limitations vs native apps (push notifications, some hardware APIs)
- React ecosystem churn is real — dependencies need maintenance
- TypeScript adds a learning curve for contributors who only know JavaScript

## Revisit Criteria

Revisit if:
- Native device features (background audio, lock screen widgets) become hard requirements
- The community strongly prefers a different framework

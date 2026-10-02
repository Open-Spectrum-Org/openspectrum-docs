# OpenSpectrum — Quick Start

Get the web app running locally in three steps.

---

## Prerequisites

- Node.js 18+
- npm

---

## Run locally

**Terminal 1 — Expo / Metro bundler**

```bash
cd openspectrum-app
npm install        # first time only
npx expo start --web --port 8081
```

Wait for: `Waiting on http://localhost:8081`

**Terminal 2 — Dev proxy** (required for SQLite to work in the browser)

```bash
cd openspectrum-app
node proxy.js
```

Wait for: `[proxy] Listening on http://localhost:8083`

**Open the app**

```
http://localhost:8083
```

> First load takes 15–30 seconds while Metro compiles. Hot-reload is fast after that.

---

## Why two terminals?

The app stores all data locally using SQLite via WebAssembly. The browser requires specific security headers (`COOP`/`COEP`) for this to work. The Expo dev server doesn't send them — `proxy.js` adds them.

---

## WSL2 users

Run both servers inside WSL. Access the app at `http://localhost:8083` in your **Windows** browser. Do not use a WSL IP address — cross-origin isolation won't activate on it.

---

## Troubleshooting

See [contributing/web-dev-setup.md](./contributing/web-dev-setup.md) for common errors (port conflicts, peer dependency issues, WASM resolution errors).

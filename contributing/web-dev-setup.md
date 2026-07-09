# Web Development Setup

This guide walks a new team member through getting the OpenSpectrum web app running locally from scratch.

---

## Prerequisites

You will need:

- **Node.js** 18 or later — [nodejs.org](https://nodejs.org)
- **npm** (comes with Node)
- **Git**
- A modern browser (Chrome or Edge recommended — Firefox works but OPFS support varies)

> **Windows users**: if you are running WSL2, follow the [WSL2 notes](#wsl2-notes) at the bottom of this page.

---

## 1. Clone the repository

If this is your first time:

```bash
git clone https://github.com/Open-Spectrum-Org/openspectrum-app.git
cd openspectrum-app
```

If you already have the repo and want to pull the latest:

```bash
git pull origin main
```

---

## 2. Install dependencies

```bash
npm install
```

This installs all packages including `react-native-web`, `react-dom`, and the Expo toolchain.

---

## 3. Start the development servers

You need **two terminal windows** running at the same time.

### Terminal 1 — Expo / Metro bundler

```bash
npx expo start --web --port 8081
```

Wait until you see:

```
Waiting on http://localhost:8081
```

### Terminal 2 — Dev proxy

```bash
node proxy.js
```

Wait until you see:

```
[proxy] Listening on http://localhost:8083
[proxy] Cross-origin isolation headers: ON
```

---

## 4. Open the app

Open your browser and go to:

```
http://localhost:8083
```

The first load takes 15–30 seconds while Metro compiles the JavaScript bundle. After that, hot-reload is fast.

---

## Why two servers?

The app uses **expo-sqlite** for local-first data storage. On web, expo-sqlite compiles SQLite to WebAssembly (via wa-sqlite) and runs it inside a Web Worker using the browser's Origin Private File System (OPFS).

OPFS requires the browser to be in a **cross-origin isolated** context, which means the server must send two HTTP headers on every response:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

The Expo dev server doesn't send these. The `proxy.js` script is a thin Node.js proxy that sits in front of Expo and injects them. This is a dev-only concern — a production deployment would configure these headers at the web server or CDN level.

---

## Troubleshooting

### The app shows a spinner and never loads

- Make sure **both** servers are running (Expo on 8081, proxy on 8083).
- Make sure you are accessing `http://localhost:8083`, **not** the IP address or port 8081 directly.
- Cross-origin isolation only activates on `localhost`. If you use an IP (e.g. `192.168.x.x`), the database will hang silently.

### Port already in use

```bash
# Find and kill whatever is on port 8081
npx kill-port 8081

# Or find the PID manually
lsof -ti:8081 | xargs kill
```

### npm install fails with peer dependency errors

```bash
npm install --legacy-peer-deps
```

### Metro can't resolve `.wasm` files

This is handled by `metro.config.js` in the repo root. If you see this error, make sure you pulled the latest `main` and restarted Expo.

---

## WSL2 notes

If you are on Windows using WSL2:

- Run both servers inside WSL (not in PowerShell/CMD).
- Access the app at `http://localhost:8083` in your **Windows** browser — WSL2 automatically bridges the port.
- Do **not** use the WSL IP address (e.g. `172.x.x.x`) — cross-origin isolation will not activate on that address.

---

## Project structure (app)

```
openspectrum-app/
├── app/               # Expo Router screens (tabs, modals)
├── src/
│   ├── components/    # Reusable UI components
│   ├── db/            # SQLite schema, queries, seed data
│   ├── hooks/         # React hooks (database, children, observations)
│   ├── theme/         # Colours, typography, spacing
│   ├── types/         # TypeScript types
│   └── utils/         # Date, UUID helpers
├── assets/            # Icons, images
├── metro.config.js    # Adds .wasm support to Metro bundler
├── proxy.js           # Dev proxy for cross-origin isolation headers
├── app.json           # Expo configuration
└── package.json
```

---

## Useful commands

| Command | What it does |
|---|---|
| `npx expo start --web --port 8081` | Start Metro bundler for web |
| `node proxy.js` | Start the dev proxy |
| `npx expo start --android` | Run on Android (requires emulator or device) |
| `npx expo install --check` | Check for outdated Expo dependencies |

---

*Questions? Open a [GitHub Discussion](https://github.com/Open-Spectrum-Org/openspectrum-app/discussions) or ping the team on Discord.*

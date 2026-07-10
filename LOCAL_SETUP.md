# MovieHub — Local Development Setup (Windows)

This is a **pnpm monorepo** with three main runnable pieces:

| Package | What it is | Default port |
|---------|-----------|-------------|
| `server/` (`@workspace/server`) | Node.js + Express + MongoDB backend | 5001 |
| `artifacts/movie-hub` | React 19 frontend (Vite) | 5173 |
| `artifacts/mockup-sandbox` | UI component preview sandbox (Vite) | 5174 |

---

## Prerequisites

1. **Node.js ≥ 20** — https://nodejs.org/en/download
2. **pnpm** — install globally once:
   ```
   npm install -g pnpm
   ```
3. **MongoDB** — either:
   - Free Atlas cluster: https://cloud.mongodb.com (recommended)
   - Local install: https://www.mongodb.com/try/download/community

---

## Step 1 — Configure environment variables

### Backend (`server/.env`)
The file `server/.env` has been created with local defaults.
Open it and **replace these two values** at minimum:

```
MONGODB_URI=mongodb+srv://<user>:<password>@cluster0.xxxxx.mongodb.net/moviehub?retryWrites=true&w=majority
```

Everything else has working dev defaults and can be filled in later.

### Frontend (no .env needed)
The Vite configs for `movie-hub` and `mockup-sandbox` default to sensible
local ports automatically. No extra config required.

---

## Step 2 — Install dependencies

Run this once from the project root (where `pnpm-workspace.yaml` lives):

```
pnpm install
```

> bcrypt requires native compilation. If it fails, make sure you have
> the Visual Studio C++ build tools installed:
> https://visualstudio.microsoft.com/visual-cpp-build-tools/

---

## Step 3 — Run the projects

Open **three separate terminals** in the project root and run one command per terminal:

### Terminal 1 — Backend API
```
pnpm --filter @workspace/server run dev
```
The API will be available at http://localhost:5001/api/health

### Terminal 2 — Movie Hub Frontend
```
pnpm --filter @workspace/movie-hub run dev
```
Open http://localhost:5173 in your browser.
API calls are automatically proxied to http://localhost:5001.

### Terminal 3 — Mockup Sandbox (optional)
```
pnpm --filter @workspace/mockup-sandbox run dev
```
Open http://localhost:5174 in your browser.

---

## Shortcut scripts (root package.json)

These are also available from the project root:

```
pnpm run dev:server     # same as Terminal 1 above
pnpm run dev:frontend   # same as Terminal 2 above
pnpm run dev:sandbox    # same as Terminal 3 above
```

---

## First-run notes

- **First registered user becomes admin** automatically.
- Without `MONGODB_URI` set, the server still starts and the health check
  (`GET /api/health`) passes — but all database endpoints will fail.
- The `artifacts/api-server` package is an **unused TypeScript shell** —
  the real backend is `server/`. You don't need to run `api-server`.

---

## Common issues

| Problem | Fix |
|---------|-----|
| `pnpm: command not found` | Run `npm install -g pnpm` first |
| `bcrypt` build error | Install VS C++ Build Tools (link above) |
| `EADDRINUSE: port 5001` | Another process is using port 5001 — change `PORT` in `server/.env` |
| `EADDRINUSE: port 5173` | Change `PORT` in `artifacts/movie-hub/.env` or set env var before running |
| MongoDB connection errors | Check your `MONGODB_URI` in `server/.env`; ensure IP is whitelisted in Atlas |
| `Cannot find module '@workspace/api-client-react'` | Run `pnpm install` from the root first |

---

## Required secrets for full functionality

| Feature | Required env vars |
|---------|------------------|
| Database (all DB features) | `MONGODB_URI` |
| Media uploads | `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` |
| Email / OTP | `SMTP_USER`, `SMTP_PASS` |
| Admin registration | `ADMIN_REGISTRATION_KEY` |

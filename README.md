<div align="center">

# Usage Metering & Quota Service — Frontend

**Operator console, company portal and Client Demo for the Usage Metering & Quota Service**

![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?logo=vite&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?logo=tailwindcss&logoColor=white)
![TanStack Query](https://img.shields.io/badge/TanStack_Query-5-FF4154?logo=reactquery&logoColor=white)
![nginx](https://img.shields.io/badge/nginx-1.27-009639?logo=nginx&logoColor=white)

[Backend](https://github.com/QaisRjoob/usage-metering-backend) ·
[Main project](https://github.com/QaisRjoob/usage-metering-service) ·
[User guide](docs/USER_GUIDE.md)

</div>

---

## Live demo

| What | URL |
|---|---|
| Web app | https://usage-metering-frontend-w270.onrender.com |
| API | https://usage-metering-backend.onrender.com/api/v1 |
| API docs (Swagger UI) | https://usage-metering-backend.onrender.com/docs |
| Health check | https://usage-metering-backend.onrender.com/api/v1/ready |

The login page has a **Test accounts** box for one-click sign-in. Code:
[backend](https://github.com/QaisRjoob/usage-metering-backend) ·
[main project](https://github.com/QaisRjoob/usage-metering-service).

Render's free plan sleeps after about 15 minutes without traffic, so the first request can take up
to a minute.

## Overview

The web app for the **Usage Metering & Quota Service**. It talks to the
[backend API](https://github.com/QaisRjoob/usage-metering-backend) through a same-origin `/api` proxy
(nginx in production, the Vite dev server locally), so the browser never makes cross-origin requests.
It can also run fully in the browser with a **mock adapter** that follows the same business rules, so
no backend is needed to try it.

A click-by-click tour of every page is in **[docs/USER_GUIDE.md](docs/USER_GUIDE.md)**.

## Features

| Interface | URL | What it offers |
|---|---|---|
| Public | `/login`, `/signup`, `/signup/status` | One-field sign-in, test accounts, company sign-up, request tracking and one-time key collection |
| Operator console | `/admin` | Dashboard, access requests, companies (tabs, search, drill-down), company statistics, users directory, plan catalogue, usage history, AI Advisor, header finder |
| Company portal | `/app` | Overview, users and user statistics, plans, quotas, usage history, Client Demo, AI Advisor |

- JWT session: access token in memory only, silent refresh before expiry, session restored after reload
- Role-separated layouts and routes; one role signed in at a time
- URL-driven filters, sorting and pagination validated with the same zod schemas as the API
- Light and dark theme, Arabic sample data, lazy-loaded pages, charts with Recharts
- Strict Content Security Policy and security headers in nginx

## Tech stack

| Area | Technologies |
|---|---|
| UI | React 19, React Router 7, Tailwind CSS 4, lucide icons, Recharts |
| Data | TanStack Query 5, zod 4 contracts shared with the backend |
| Build | Vite 7, TypeScript 5.9 |
| Tests | Vitest, Testing Library, jsdom |
| Serving | nginx 1.27 (static files, `/api` proxy, CSP) |

## Quick start

Requirements: Node.js 22+.

```bash
npm install
npm run dev
```

Open http://localhost:5173. This uses the in-browser **mock adapter**: no backend needed, and the
login page offers test accounts for the demo company and the operator.

### Against a real backend

Start the backend first (see its README; it listens on `:3000`), then:

```bash
npm run dev:api-mode
```

That serves http://localhost:5174 and forwards `/api` to `http://localhost:3000`. To use a hosted
backend instead, set `API_PROXY_TARGET`, e.g. `API_PROXY_TARGET=https://usage-metering-backend.onrender.com`.

### With Docker

```bash
docker build -t usage-metering-frontend --build-arg VITE_DEMO_API_KEY=umk_demo_local_only_do_not_use_in_production .
docker run --rm -p 8080:80 usage-metering-frontend
```

Open http://localhost:8080. By default nginx forwards `/api` to a backend on the host's port 3000
(`http://host.docker.internal:3000`); pass `-e API_UPSTREAM=...` to use another one.

## Configuration

**Build time** (Vite, baked into the bundle; pass as Docker build args):

| Variable | Default | Description |
|---|---|---|
| `VITE_DATA_SOURCE` | `api` in Docker | `mock` (in-browser) or `api` |
| `VITE_API_BASE_URL` | `/api/v1` | API base path (same origin) |
| `VITE_DEMO_API_KEY` | – | Shows the demo company test account and pre-fills the Client Demo |
| `VITE_DEMO_OPERATOR_TOKEN` | – | Shows an operator test account (public demos only: anyone can then act as operator) |

**Run time** (nginx container):

| Variable | Default | Description |
|---|---|---|
| `API_UPSTREAM` | `http://host.docker.internal:3000` | Backend address for `/api/*`, e.g. `https://usage-metering-backend.onrender.com` |
| `PORT` | `80` | Port nginx listens on (hosting platforms set it) |
| `REAL_IP_FROM` / `REAL_IP_FROM_V6` | loopback | Edge proxy addresses whose `X-Forwarded-For` is believed |
| `API_ORIGIN` | – | Extra origin allowed in the CSP `connect-src` (not needed with the proxy) |

## Project structure

```text
src/
  contracts/     DTOs and zod schemas (identical copy of the backend's src/contracts)
  models/        view-facing types and helpers
  services/      API client (JWT, refresh), HTTP services and the mock adapter behind one interface
  view_models/   hooks combining queries, URL state and validation
  views/         pages (operator console, company portal, public pages)
  widgets/       layout, tables, filters, charts, UI primitives
  providers/     theme, query client, session restore
  routing/       routes and role guards
```

When the API contract changes, update `src/contracts` in the backend and copy the same files here.

## Testing

| Check | Command |
|---|---|
| Tests (Vitest + Testing Library) | `npm test` |
| Typecheck | `npm run typecheck` |
| Lint | `npm run lint` |
| Production build | `npm run build` |

## Deploy (Render)

Deploy the backend first and copy its public URL. Then create a **Static Site** from this repository:

| Setting | Value |
|---|---|
| Build command | `npm ci && npm run build` |
| Publish directory | `dist` |

Set the variables (Vite bakes them in at build time):

| Variable | Value |
|---|---|
| `NODE_VERSION` | `22` |
| `VITE_DATA_SOURCE` | `api` |
| `VITE_API_BASE_URL` | `/api/v1` |
| `VITE_DEMO_API_KEY` | `umk_demo_local_only_do_not_use_in_production` when the backend seeds the demo data |
| `VITE_DEMO_OPERATOR_TOKEN` | optional: the backend's `ADMIN_API_TOKEN`, to show an operator test account |

Add these rules under *Redirects/Rewrites*, in this order:

| Source | Destination | Action |
|---|---|---|
| `/api/*` | `https://usage-metering-backend.onrender.com/api/*` | Rewrite |
| `/*` | `/index.html` | Rewrite |

The first rule proxies the API on the same origin, so the `SameSite=Strict` refresh cookie works;
the second serves the single-page app for every route.

The Docker image (nginx, with `API_UPSTREAM` pointing at the backend) also works on any Docker host.

---

<div align="center">

Final Project of the Boot.dev Plan (TypeScript), 2026-C3 · Built by [Qais Rjoob](https://github.com/QaisRjoob)

</div>

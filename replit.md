# MovieHub Enterprise

A full-stack movie streaming platform with React frontend, Node.js/Express/MongoDB backend, short-form video feed (Cinephile Shorts), social X-style feed, admin dashboard, and AI assistant.

## Run & Operate

**API Server (MovieHub backend):**
```
pnpm --filter @workspace/server run dev
```
Runs on port 8080 at `/api`. Requires `MONGODB_URI` secret to be set.

**Movie Hub Frontend:**
```
pnpm --filter @workspace/movie-hub run dev
```
Runs on port 21859 at `/`.

**Install dependencies:**
```
pnpm install
```

**Root-level commands:**
- `pnpm run typecheck` — full typecheck across TypeScript packages
- `pnpm run build` — build all packages

## Required Environment Secrets

Set these in **Replit Secrets** (the lock icon in the sidebar):

| Secret | Description |
|--------|-------------|
| `MONGODB_URI` | MongoDB Atlas connection string — **required for all DB features** |
| `JWT_SECRET` | JWT signing secret (use a long random string) |
| `JWT_REFRESH_SECRET` | Refresh token secret |
| `COOKIE_SECRET` | Cookie signing secret |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary for media uploads |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |
| `SMTP_USER` | Email address for sending OTPs |
| `SMTP_PASS` | SMTP password / app password |
| `ADMIN_REGISTRATION_KEY` | Key for registering admin accounts |

See `server/.env.example` for the full list of optional settings.

## Stack

- **Frontend:** React 19, Vite, Tailwind CSS 4, Wouter (routing), TanStack Query, Embla Carousel
- **Backend:** Node.js 24, Express 5, Mongoose 8 (MongoDB Atlas)
- **Auth:** JWT (access + refresh tokens), bcrypt, HTTP-only signed cookies, OTP flow
- **Media:** Cloudinary (images, videos, HLS streaming)
- **Email:** Nodemailer (SMTP)
- **Uploads:** Multer (memory storage → stream to Cloudinary)
- **Rate Limiting:** express-rate-limit (global, auth, upload, search tiers)
- **Logging:** Winston + daily-rotate-file
- **Workspace:** pnpm monorepo

## Where Things Live

```
server/                        ← Main backend (Node.js + Express + MongoDB)
├── app.js                     ← Express app, middleware pipeline, route mounts
├── server.js                  ← HTTP server bootstrap + graceful shutdown
├── config/
│   ├── constants.js           ← All platform enums and constants
│   ├── database.js            ← MongoDB Atlas connection with retry
│   └── cloudinary.js          ← Cloudinary config + upload presets
├── models/                    ← Mongoose models (14 models)
├── controllers/               ← Route handlers (auth, movie, series, short, post, admin...)
├── routes/                    ← Express routers (13 route files)
├── services/                  ← token, email, cloudinary, notification services
├── validators/                ← express-validator chains
├── middleware/                ← auth, adminAuth, errorHandler, rateLimiter, upload, validate
└── utils/                     ← ApiError, ApiResponse, asyncHandler, helpers, logger

artifacts/movie-hub/           ← React frontend (Vite + Tailwind)
artifacts/api-server/          ← Artifact shell — routes traffic to server/ in dev
lib/api-spec/                  ← OpenAPI spec + codegen config
lib/db/                        ← Drizzle schema (auxiliary, not used by main backend)
```

## API Routes

| Prefix | Description |
|--------|-------------|
| `GET /api/health` | Health check (no auth required) |
| `POST /api/auth/register` | Register new user |
| `POST /api/auth/login` | Login with email/username/phone + password |
| `POST /api/auth/refresh` | Rotate access + refresh tokens |
| `POST /api/auth/send-otp` | Send OTP to email/phone |
| `POST /api/auth/verify-otp` | Verify OTP and log in |
| `POST /api/auth/forgot-password` | Send password reset email |
| `POST /api/auth/reset-password` | Reset password with token |
| `GET /api/movies` | List movies (pagination, filters, sort) |
| `GET /api/movies/:id` | Get movie by ID or slug |
| `POST /api/movies/:id/like` | Toggle like (auth required) |
| `POST /api/movies/:id/watchlist` | Toggle watchlist (auth required) |
| `GET /api/series` | List TV series |
| `GET /api/series/:id/episodes` | Get all episodes grouped by season |
| `GET /api/shorts/feed` | Infinite scroll shorts feed |
| `GET /api/posts/feed` | Social feed (following-first if authenticated) |
| `GET /api/search?q=...` | Global search across all content types |
| `GET /api/search/trending` | Trending content |
| `GET /api/search/suggest?q=...` | Autocomplete suggestions |
| `GET /api/notifications` | My notifications (auth required) |
| `GET /api/profile` | My profile (auth required) |
| `GET /api/profile/watchlist` | My watchlist |
| `GET /api/profile/history` | My watch history |
| `PUT /api/settings/preferences` | Update UI preferences |
| `PUT /api/settings/notifications` | Update notification settings |
| `POST /api/upload/avatar` | Upload avatar image |
| `POST /api/upload/short` | Upload short video |
| `GET /api/admin/dashboard` | Admin dashboard stats |
| `GET /api/admin/users` | List all users (admin/mod) |
| `POST /api/admin/users/:id/ban` | Ban a user |
| `GET /api/admin/genres` | List all genres |

## Architecture Decisions

- **HTTP server starts before DB connects** — prevents workflow timeouts when MongoDB is slow; health check always passes.
- **JWT dual-token flow** — short-lived access tokens (7d) + long-lived refresh tokens (30d) stored as signed HTTP-only cookies and hashed in DB for refresh-token rotation.
- **Polymorphic Like/Comment models** — single Like and Comment collection covers movies, series, shorts, posts, and comments via `targetType` discriminator field.
- **Denormalized `genreNames` array** on Movie/Series — enables fast genre filtering without join; synced when genre data changes.
- **server/ is a standalone workspace package** (`@workspace/server`) — not compiled; runs directly via `node --watch server.js`. The `artifacts/api-server` artifact.toml routes to it.

## Product

- **Movie Hub** — Browse, search, and stream movies and TV series with genre/year/country filters
- **Cinephile Shorts** — TikTok-style vertical video feed for short-form content
- **X Feed** — Twitter-style social feed for movie fans
- **AI Assistant** — AI-powered search and movie recommendations
- **Admin Dashboard** — Content management, user moderation, analytics

## User Preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Set `MONGODB_URI` in Replit Secrets before any database-backed endpoint will work.
- `bcrypt` requires native compilation — it's listed in `onlyBuiltDependencies` in `pnpm-workspace.yaml`.
- The `server/` directory is in the pnpm workspace as `@workspace/server` but it's a plain JS package — never add a `tsconfig.json` to it.
- First registered user automatically becomes `admin`.
- CORS: add the frontend origin to the `ALLOWED_ORIGINS` environment variable in production.
- The `artifacts/api-server` TypeScript skeleton still exists but is unused — `artifact.toml` points the workflow to `@workspace/server` instead.

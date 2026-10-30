# RepoPilot

AI Repository Debugging Agent. Investigates software defects, understands unfamiliar repositories, generates code patches, and verifies fixes by executing tests. Powered by NVIDIA Nemotron via Nebius Token Factory.

Built for the Nebius x NVIDIA Global AI Hackathon. Stack: Next.js 16, TypeScript, Tailwind CSS v4, shadcn/ui, Neon PostgreSQL, Drizzle ORM, better-auth, TanStack Query, Zustand, Bun.

## Key Features

- Autonomous debugging: submit an issue, get a diagnosed root cause, a generated patch, and verified test results
- Repository directory: connect GitHub repositories via GitHub App or OAuth, view metadata and language stats
- Agent run history: list, filter, and inspect runs with status, verification status, duration, and findings
- Run detail page: investigation timeline, code diff panel, test output, verification badge (verified / unverified / no patch)
- Overview dashboard: KPI cards (verified fixes, test pass rate, average repair time, connected repositories), contribution heatmap, member status table, latest activity feed
- Benchmarks: evaluation results across SWE-bench Verified, SWE-bench Lite, HumanEval Plus, and an internal RepoDebug suite (seed data, labeled)
- Models & Usage: Nebius Token Factory provider config, default model `nvidia/nemotron-4-340b-instruct`, connection status
- Authentication: email/password with email verification, GitHub and Google OAuth, admin role plugin, rate limiting on sign-in/sign-up
- Protected routes via middleware (`src/proxy.ts`): `/overview`, `/repositories`, `/runs`, `/settings`, `/models`, `/benchmarks`

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | Next.js (App Router) | 16.4.3 |
| UI | React | 19.3.0 |
| Language | TypeScript | 5 |
| Styling | Tailwind CSS v4 | — |
| UI components | shadcn/ui, Aceternity UI | — |
| Icons | Lucide React | — |
| Animations | Framer Motion (`motion`) | 14.6.0 |
| Authentication | better-auth | 1.7.12 |
| Database | Neon serverless PostgreSQL | — |
| ORM / Migrations | Drizzle ORM, Drizzle Kit | 1.0.0-rc.5 |
| Client state | TanStack Query, Zustand | 5.x / 5.0.16 |
| Forms / validation | React Hook Form, Zod | 7.x / 4.6.7 |
| Email | Resend (with in-memory dev adapter) | 6.34.1 |
| GitHub | Octokit, @octokit/auth-app | 5.0.5 |
| AI inference | Nebius Token Factory API | — |
| Default model | nvidia/nemotron-4-340b-instruct | — |
| Package manager | Bun | 1.3.14 |
| Testing | Vitest, Playwright | 5.x / 1.64.1 |
| Lint | ESLint (eslint-config-next) | 9 |

## Architecture

```
src/
  app/                  Next.js App Router routes, layouts, route handlers
    (auth)/             sign-in, sign-up, verify-email, forgot-password, reset-password
    (dashboard)/        overview, repositories, runs, benchmarks, models, settings
    api/                route handlers: auth, github, repositories, runs, metrics, activity
    dashboard/          legacy overview page + data.json fixture
    privacy/, terms/    static pages
    layout.tsx          root layout (Inter + Geist Mono fonts, theme provider)
    globals.css         Tailwind v4 + design tokens
    page.tsx            public landing page
    proxy.ts            middleware: protected/auth route redirects
  components/
    layout/             sidebar, header, breadcrumbs, activity panel
    dashboard/          KPI cards, activity heatmap, member status table
    repositories/       GitHub connect button, connection card, commit history
    ui/                 shadcn/ui primitives + demo sections
    auth/               auth card, alert, OAuth buttons, password input
    providers/          TanStack Query provider + toaster
  lib/
    db/                 Drizzle client, schema (users, repositories, agent_runs, agent_events, github_*), queries, seed
    ai/                 AIProvider: Nebius chat/completions client with demo fallback
    api/                typed fetch client, ApiError, TanStack query keys
    authorization/      requireSession, requireUser, requireRole, assertResourceOwnership
    email/              adapters (Resend, in-memory), templates
    github/             GitHub App Octokit, sync, installations, commits, contents
    schemas/            Zod schemas: repository, agent-run, benchmark, github, settings
    stores/             Zustand: app-store, sidebar-store
    auth.ts, auth-server.ts, auth-client.ts, auth-utils.ts
    benchmarks.ts       seeded benchmark records (labeled demo data)
    utils.ts            cn() helper
  types/
    api.ts, repository.ts, agent.ts
```

Data flow:
1. Client calls Next.js route handlers (`/api/repositories`, `/api/runs`, `/api/metrics`, `/api/activity`) or Server Components fetch directly from Drizzle queries.
2. Route handlers validate input with Zod, authenticate via better-auth, invoke service/query functions, return `{ success, data, error }` JSON.
3. AI provider (`src/lib/ai/ai-provider.ts`) calls Nebius Token Factory `POST /chat/completions`; returns structured `BugDiagnosis` JSON or a realistic demo fallback when `NEBIUS_API_KEY` is unset.
4. GitHub integration (`src/lib/github/`) uses Octokit with GitHub App installation auth (or cached/persisted repos when the PEM key is unconfigured).
5. TanStack Query caches client-side server state; Zustand holds UI-only state (sidebar collapse, app preferences).

## Project Structure

```
NebiusxNvidia/
├── README.md
├── AGENTS.md
├── design.md
├── package.json
├── bun.lock
├── tsconfig.json
├── next.config.ts
├── drizzle.config.ts
├── vitest.config.ts
├── playwright.config.ts
├── eslint.config.mjs
├── components.json
├── .env.example
├── .gitignore
├── build.log
├── skills-lock.json
├── video.d.ts
├── next-env.d.ts
├── src/
│   ├── app/
│   ├── components/
│   ├── lib/
│   ├── db/
│   ├── hooks/
│   ├── types/
│   └── proxy.ts
├── tests/
│   ├── auth-lifecycle.test.ts
│   ├── auth-utils.test.ts
│   ├── authorization.test.ts
│   ├── email.test.ts
│   ├── setup.ts
│   └── e2e/
├── docs/
│   ├── project.md
│   ├── requirements.md
│   ├── demo-notes.md
│   ├── rule.md
│   └── repo.png
├── drizzle/
│   ├── 20261024053001_small_phantom_reporter/
│   └── 20261024070535_complete_roughhouse/
├── public/
│   ├── assets/
│   └── videos/
└── videos/
    └── hero-video.mp4.json
```

Excluded from tree: `node_modules/`, `.next/`, `coverage/`, `.env`, `*.tsbuildinfo`, `videos/*` (generated media).

## Prerequisites

- Bun 1.3.14 (package manager; `bun.lock` present)
- Node.js 20+ (`@types/node: ^20`)
- Neon PostgreSQL database (serverless) with `DATABASE_URL`
- For AI inference: Nebius Token Factory account and `NEBIUS_API_KEY`
- For GitHub App sync: GitHub App ID, client ID/secret, and downloaded `.pem` private key
- For email in production: Resend API key and verified sender (`RESEND_API_KEY`, `EMAIL_FROM`)

## Installation and Setup

```bash
# 1. Clone the repository
git clone <repo-url>
cd NebiusxNvidia

# 2. Install dependencies
bun install

# 3. Copy environment example
cp .env.example .env

# 4. Fill in .env with real values (see Environment Variables below)
#    Never commit .env; it is gitignored.

# 5. Push the Drizzle schema to Neon
bun run db:push

# 6. (Optional) Seed demo data
bun run db:seed

# 7. Start the dev server
bun run dev
# App runs at http://localhost:3000
```

## Environment Variables

Copy `.env.example` to `.env` and fill in real values. Secrets never go in the repository.

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | Yes | Neon PostgreSQL serverless connection string | `postgresql://user:pass@ep-x.pooler.us-east-2.aws.neon.tech/neondb?sslmode=require` |
| `BETTER_AUTH_SECRET` | Yes | Cryptographically random secret, min 32 chars in production | `openssl rand -base64 32` |
| `BETTER_AUTH_URL` | Yes | Canonical public app URL | `http://localhost:3000` |
| `TRUSTED_ORIGINS` | No | Comma-separated additional trusted origins | `https://preview.yourdomain.com` |
| `GITHUB_CLIENT_ID` | No | GitHub OAuth App client ID (sign-in) | — |
| `GITHUB_CLIENT_SECRET` | No | GitHub OAuth App client secret (sign-in) | — |
| `GOOGLE_CLIENT_ID` | No | Google OAuth client ID (sign-in) | — |
| `GOOGLE_CLIENT_SECRET` | No | Google OAuth client secret (sign-in) | — |
| `RESEND_API_KEY` | No | Resend API key for transactional email | `re_...` |
| `EMAIL_FROM` | No | Verified sender address | `resend` |
| `EMAIL_ADAPTER` | No | `"memory"` (dev default) or `"resend"` | `memory` |
| `GITHUB_APP_SLUG` | No | GitHub App slug (agent sync) | `repository-helper` |
| `GITHUB_APP_ID` | No | GitHub App ID | — |
| `GITHUB_PRIVATE_KEY` | No | GitHub App downloaded `.pem` private key | — |
| `NEBIUS_API_KEY` | No | Nebius Token Factory API key | — |
| `NEBIUS_MODEL` | No | Default model; falls back to `nvidia/nemotron-4-340b-instruct` | `nvidia/nemotron-4-340b-instruct` |

OAuth providers (GitHub, Google) are only enabled when both credentials are present. Email uses the in-memory adapter in development or when `RESEND_API_KEY` is absent; set `EMAIL_ADAPTER=resend` in production.

The GitHub App private key must be a PEM block (`-----BEGIN PRIVATE KEY-----...`); a SHA256 fingerprint is rejected by `getFormattedPrivateKey()`.

## Running the Project

| Command | Purpose |
|---------|---------|
| `bun run dev` | Start Next.js dev server with Turpack at http://localhost:3000 |
| `bun run build` | Production build (`next build`) |
| `bun run start` | Start production server (`next start`) |
| `bun run lint` | Run ESLint |
| `bun run test` | Run Vitest unit tests (`vitest run`) |
| `bun run test:watch` | Watch-mode Vitest |
| `bun run test:e2e` | Run Playwright E2E tests (starts dev server) |
| `bun run db:push` | Push Drizzle schema to Neon (`drizzle-kit push`) |
| `bun run db:seed` | Seed repositories, agent runs, and events (`tsx src/lib/db/seed.ts`) |

Dev server uses Tailwind CSS v4 via `@tailwindcss/turbopack`.

## Usage Guide

1. **Sign up / sign in** at `/sign-up` or `/sign-in`. Email verification is required before accessing the dashboard.
2. **Connect a repository** from the dashboard (`Connect GitHub` button) or `/repositories`. The GitHub App sync imports installation repositories into the `repositories` table.
3. **Start a debug session** from `/runs/new`: select a repository, branch, describe the issue, and submit. The agent diagnoses via NVIDIA Nemotron on Nebius Token Factory, generates a patch, and runs tests.
4. **Inspect a run** at `/runs/[runId]`: view the investigation timeline, code diff, findings, test command/output, and verification status.
5. **Browse runs** at `/runs` with repository, status, and verification filters.
6. **View benchmarks** at `/benchmarks` (labeled seed data for the demo).
7. **Check model configuration** at `/models` (Nebius provider, default model, connection status).
8. **Manage settings** at `/settings` (consolidated general/agent/security/integrations sections via anchors).

The AI provider runs in demo mode without `NEBIUS_API_KEY`, returning structured fallback diagnoses so the workflow is exercisable locally.

## Available Scripts

All scripts run with Bun in the repository root:

- `bun run dev` — dev server (`next dev`)
- `bun run build` — production build (`next build`)
- `bun run start` — production server (`next start`)
- `bun run lint` — ESLint
- `bun run test` — Vitest unit tests
- `bun run test:watch` — Vitest watch mode
- `bun run test:e2e` — Playwright E2E tests
- `bun run db:push` — Drizzle schema push to Neon
- `bun run db:seed` — seed demo data into the database

## API Documentation

Route handlers live in `src/app/api/`. All return JSON with shape `{ success, data?, error?, message? }`.

| Endpoint | Method | Description | Auth |
|----------|--------|-------------|------|
| `/api/repositories` | GET | List repositories (DB or seed fallback) | Session |
| `/api/repositories` | POST | Create a repository (Zod-validated) | Session |
| `/api/repositories/[repoId]` | — | Repository detail route | — |
| `/api/runs` | GET | List agent runs (filters: `repoId`, `status`) | Session |
| `/api/runs` | POST | Create a run: diagnose via Nemotron, generate patch, run tests | Session |
| `/api/runs/[runId]` | — | Run detail route | — |
| `/api/metrics` | GET | Overview metrics + activity heatmap | Session |
| `/api/activity` | GET | Latest agent events (last 10) | Session |
| `/api/auth/[...all]` | — | better-auth handlers (sign-in, sign-up, OAuth, email verification, password reset) | — |
| `/api/github/callback` | — | GitHub OAuth callback | — |
| `/api/github/install` | — | GitHub App installation flow | — |
| `/api/github/installations` | — | List installations | — |
| `/api/github/repositories` | — | Sync/list GitHub repositories | — |
| `/api/github/sync` | — | Sync installation repositories via Octokit | — |

Response example (`GET /api/runs`):

```json
{
  "success": true,
  "data": [
    {
      "id": "run-701",
      "repoId": "repo-1",
      "repoName": "prime-marketplace-3.0",
      "title": "Fix signature deserialization panic on malformed payload",
      "status": "passed",
      "verificationStatus": "verified",
      "durationMs": 42000,
      "modelUsed": "nvidia/nemotron-4-340b-instruct",
      "testCommand": "pnpm test src/crypto/verify.test.ts",
      "testExitCode": 0
    }
  ]
}
```

API errors return a non-2xx status with `{ success: false, error: "<message>" }`. `ApiError` class in `src/lib/api/errors.ts` normalizes status and payload.

## Data Storage

- Primary store: Neon serverless PostgreSQL via Drizzle ORM (`drizzle-orm/neon-http`, `@neondatabase/serverless`).
- Connection: `src/lib/db/index.ts` exports `sql` (neon) and `db` (drizzle); fallback placeholder URL used at build time when `DATABASE_URL` is absent.
- Schema (Drizzle, pg-core): `users`, `sessions`, `accounts`, `verifications` (better-auth Drizzle adapter), `repositories`, `agent_runs`, `agent_events`, `github_installations`, `github_repositories`, `github_install_states`.
- Migrations: `drizzle/` (two versioned folders). Apply with `bun run db:push`.
- Seed: `bun run db:seed` inserts sample repositories, agent runs, and events (`onConflictDoNothing`).
- Queries fall back to in-memory `initialSampleRepositories` / `initialSampleRuns` / `initialSampleEvents` during production builds or when `DATABASE_URL` is missing.
- Secrets are never stored in database records.

## Testing

Unit tests (Vitest):
- `tests/auth-lifecycle.test.ts`
- `tests/auth-utils.test.ts`
- `tests/authorization.test.ts`
- `tests/email.test.ts`
- Setup: `tests/setup.ts`

Run unit tests:
```bash
bun run test
```

E2E tests (Playwright) live in `tests/e2e/` and start the dev server via `webServer`:
```bash
bun run test:e2e
```

Type checking: `tsc --noEmit` is enforced via `isolatedModules` and `strict: true` in `tsconfig.json`.

## Deployment

Deploy target is platform-agnostic; configure `DATABASE_URL`, `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL`, and optional provider keys on the deployment host. Build with `bun run build`, run with `bun run start`.

Deployment notes from the project:
- `BETTER_AUTH_SECRET` must be at least 32 characters in production; the app throws a fatal error otherwise.
- `BETTER_AUTH_URL` must match the public origin; trusted origins can be added via `TRUSTED_ORIGINS`.
- Set `EMAIL_ADAPTER=resend` and provide `RESEND_API_KEY` + `EMAIL_FROM` for production email; otherwise email falls back to the in-memory adapter.
- GitHub App sync requires a valid PEM private key (`GITHUB_PRIVATE_KEY`) and `GITHUB_APP_ID`; without it, cached repositories are shown.
- AI inference requires `NEBIUS_API_KEY`; without it the provider returns demo fallback responses.
- Do not commit `.env`; `.env.example` is the only tracked env file.

## Troubleshooting

| Issue | Likely cause | Fix |
|-------|--------------|-----|
| `FATAL: BETTER_AUTH_SECRET is required...` | Production mode with short/missing secret | Set `BETTER_AUTH_SECRET` to ≥32 random chars |
| DB query fails at build time | `DATABASE_URL` absent during `next build` | Fallback placeholder URL is used; set `DATABASE_URL` for runtime DB access |
| GitHub sync shows cached repos only | `GITHUB_PRIVATE_KEY` is a SHA256 fingerprint, not PEM | Set the downloaded `.pem` private key, restoring `\n` escapes |
| OAuth sign-in providers missing | `GITHUB_CLIENT_ID`/`SECRET` or Google pair not set | Add both credentials for the provider to enable |
| Email not sending in dev | No `RESEND_API_KEY` | Expected; in-memory adapter is used. Set `EMAIL_ADAPTER=resend` for real email |
| AI returns demo fallback | `NEBIUS_API_KEY` unset | Expected local behavior; set the key for real Nebius inference |
| `redirect("/sign-in")` from dashboard | No valid better-auth session cookie | Sign in first; protected routes require authentication |

## Contributing

1. Fork the repository and create a branch from `main`.
2. Make changes scoped to the issue; do not add unrelated features or refactors.
3. Run `bun run lint`, `bun run test`, and `bun run build` before submitting.
4. Update `drizzle/` migrations only via `drizzle-kit`; never reset or drop an existing database.
5. Never commit `.env` or other secrets.
6. Open a pull request with a clear description of the change and verification steps.

For substantial architecture changes, discuss in an issue first. The project is private; a public license will be added before hackathon submission per the submission requirements.

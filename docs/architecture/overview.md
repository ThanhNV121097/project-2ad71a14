# Architecture Overview — hello-word-10

## 1. Scope

Project is fullstack proof page. It serves one centered greeting line. Greeting text lives in PostgreSQL, backend reads it, frontend renders it.

In scope for foundation:

- Go backend skeleton under `code/backend/`.
- PostgreSQL runtime through repository `docker-compose.yml`.
- Next.js 15 App Router frontend skeleton under `code/frontend/`.
- Build, lint, type check, vet, and test through existing `.github/workflows/ci.yml`.

Out of scope for scaffold:

- Finished greeting UI component.
- API implementation for greeting retrieval beyond health and migration-capable server shell.
- Auth, editing flows, multiple pages, analytics, themes, animations.

## 2. Stack

| Layer | Choice | Reason | Rejected alternative / tradeoff |
|---|---|---|---|
| Frontend | Next.js 15 App Router, TypeScript, Tailwind v3 | Matches project convention and supports server-rendered composition root | Plain HTML would be smaller but would not match pipeline or later story structure |
| Backend | Go 1.22 HTTP server | Existing default, small binary, easy container build | Node API would duplicate frontend runtime and break backend convention |
| Database | PostgreSQL 16 | Required by SRS; one row persists greeting text | In-memory data would violate SRS and fail end-to-end DB proof |
| Migrations | Embedded SQL files applied on backend boot | Runtime starts with empty DB; server must self-migrate | External migration job would add another moving part for one table |
| Styling | Tailwind v3 plus CSS tokens in `app/globals.css` | Matches design-system gate and keeps shared values centralized | Component-local hardcoded values fail CI token checks |
| Containers | Existing Dockerfiles and `docker-compose.yml` conventions | Deployer and CI expect `code/backend` and `code/frontend` contexts | Editing container workflow risks deploy mismatch |

## 3. Repository layout

```text
docs/
  general/SRS.md
  architecture/overview.md
  architecture/erd.md
  architecture/services.md
code/
  backend/
    cmd/api/main.go
    migrations/migrations.go
    migrations/*.sql
    go.mod
    .env.example
    Dockerfile
  frontend/
    app/layout.tsx
    app/page.tsx
    app/globals.css
    package.json
    next.config.js
    tailwind.config.ts
    postcss.config.js
    tsconfig.json
    .eslintrc.json
    .env.example
```

## 4. Backend architecture

Backend owns HTTP API, DB connection, migrations, and health reporting.

Startup order:

1. Read `DATABASE_URL`; fail fast if missing.
2. Connect to PostgreSQL using `pgx` stdlib-compatible driver.
3. Apply embedded migrations from `code/backend/migrations/` in filename order.
4. Confirm DB with `SELECT 1`.
5. Listen on `PORT`; fallback `APP_PORT`; fallback `8080`.

`/healthz` returns `200 OK` only when migrations already succeeded and current DB ping succeeds. Otherwise it returns `503` with generic error envelope.

Backend package rule: exactly one `main` package, in `code/backend/cmd/api`. Shared code should stay in non-main packages only when second file needs it. No interface until two implementations exist.

## 5. Frontend architecture

`app/page.tsx` is composition root only. It stays Server Component and lists story components. Story authors add one import and one JSX element; they do not rewrite page shell.

`app/globals.css` defines shared tokens from design system for six categories:

- Colour: `--color-bg`, `--color-text`.
- Spacing: `--space-0`.
- Typography: `--font-body`, `--font-heading`, `--text-display`, `--leading-display`, `--weight-display`.
- Radius: `--radius-sm`, `--radius-md`, `--radius-lg`, `--radius-full`.
- Shadow: `--shadow-sm`, `--shadow-md`, `--shadow-lg`.
- Motion: `--duration-fast`, `--duration-base`, `--easing`.

Unused categories keep explicit `none` or `0` values so story CSS never invents tokens.

Server/client boundary:

- Default: Server Component.
- Any file using browser APIs, event handlers, `useState`, `useEffect`, or refs must start with literal first line `"use client"`.
- `app/page.tsx` must not pass functions to children.

## 6. Data flow

```text
Browser -> Next.js page -> backend /v1/greeting -> PostgreSQL greeting row
```

Frontend reads backend URL from `NEXT_PUBLIC_API_URL`. Local browser uses `http://localhost:8080`. Docker frontend build may bake this public URL; server-to-server proxy is not used in scaffold.

Backend reads database URL from `DATABASE_URL`; it never assembles credentials from separate DB variables.

## 7. Environment variables

Root `.env.example` documents compose variables:

| Key | Used by | Required | Notes |
|---|---|---|---|
| `POSTGRES_USER` | compose postgres/backend | local only | Defaults to `app` |
| `POSTGRES_PASSWORD` | compose postgres/backend | local only | Defaults for local dev only |
| `POSTGRES_DB` | compose postgres/backend | local only | Defaults to `app` |
| `BACKEND_PORT` | compose | optional | Host port for backend |
| `FRONTEND_PORT` | compose | optional | Host port for frontend |
| `NEXT_PUBLIC_API_URL` | frontend | optional | Browser-facing backend URL |

Backend `.env.example`:

| Key | Required | Notes |
|---|---|---|
| `DATABASE_URL` | yes | Injected by runtime |
| `PORT` | yes in runtime | HTTP port; fallback exists for local |
| `APP_PORT` | optional | Secondary fallback |

Frontend `.env.example`:

| Key | Required | Notes |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | yes for API calls | Example `http://localhost:8080` |

No secrets committed. Examples contain local placeholders only.

## 8. API and persistence contracts

Detailed schema lives in `docs/architecture/erd.md`. Detailed endpoints and error envelope live in `docs/architecture/services.md`.

Foundation decision: all product API paths use `/v1/...` without `/api` prefix. Deploy proxy strips `/api` before backend receives requests.

## 9. Naming conventions

| Item | Convention |
|---|---|
| Go packages | short lowercase names |
| Go exported symbols | only when used across packages |
| SQL files | UTC timestamp prefix, `.up.sql` and `.down.sql` pairs |
| DB tables | plural snake_case |
| DB columns | snake_case |
| Frontend components | `export default function ComponentName()` |
| Frontend story files | `code/frontend/components/{PascalCase}.tsx` later |
| CSS tokens | semantic `--category-job` names |
| API JSON fields | camelCase |

## 10. Failure handling

- Startup fails if DB is unreachable or migrations fail.
- `/healthz` returns 503 when DB check fails.
- API errors use shared envelope from services doc.
- External errors shown to clients are generic; logs keep internal detail.
- Context timeouts bound DB checks and migration statements.

## 11. Observability

Use Go standard `log/slog` to log startup, migration application, health failures, and server shutdown. No analytics, tracing, or metrics needed for one proof page.

## 12. Local run

```bash
cp .env.example .env
docker compose --profile local up --build
```

Expected services:

- PostgreSQL on compose network.
- Backend on `http://localhost:8080`.
- Frontend on `http://localhost:3000`.

Health check:

```bash
curl http://localhost:8080/healthz
```

## 13. CI gate

Existing `.github/workflows/ci.yml` runs on pull requests:

- Backend: `go mod download`, `go build ./...`, `go vet ./...`, `go test ./...`.
- Frontend: `npm ci`, `npm run lint`, `npm run build`, `npm test --if-present`.
- CSS token checks: no hardcoded module values, used tokens defined, no token fallbacks.

`.github/` is read-only for agents. Do not edit workflows.

## 14. Risks and constraints

| Risk | Mitigation |
|---|---|
| Runtime DB starts empty | Backend self-migrates before listen |
| Greeting missing | Migration seeds exactly one row; story API reads it |
| Token drift | `globals.css` mirrors design-system tokens |
| Story PR conflicts | `page.tsx` kept as thin composition root |
| Hardcoded frontend copy | Story must fetch `/v1/greeting`; frontend scaffold contains no product copy |

## 15. Deliberate non-features

| Skipped | Add when |
|---|---|
| Auth | Product has user-specific data |
| Admin editing | Stakeholder asks to change greeting in app |
| Migration tool binary | Multiple services need migration control |
| Retry framework | Remote dependencies appear beyond DB |
| UI error state | SRS asks for non-default states |

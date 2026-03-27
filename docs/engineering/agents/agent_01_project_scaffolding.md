# Agent 01 — Project Scaffolding

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-01` |
| **Name** | Project Scaffolding |
| **Sprint** | S1 — Foundation |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 2 days |
| **Dependencies** | None (first agent) |
| **Blocks** | Agent 02, Agent 03, Agent 04, Agent 05 |

## Objective

Initialize the full development environment: Next.js frontend, Express backend, Docker services, CI pipeline, and developer tooling. Every subsequent agent depends on this foundation.

## Scope

- Next.js 14 App Router project with TypeScript
- Express.js API server with modular folder structure
- Docker Compose for local PostgreSQL 16 + Redis 7
- Prisma ORM initial connection
- ESLint + Prettier + Husky pre-commit hooks
- GitHub Actions CI skeleton
- Environment variable template

## Out of Scope

- Database schema (Agent 02)
- Any business logic
- Production Dockerfile (Agent 12 / T-070)

## Tasks

### T-001: Initialize Next.js + TypeScript Project

**Files**:
- `[NEW] package.json`
- `[NEW] tsconfig.json`
- `[NEW] next.config.ts`
- `[NEW] src/app/layout.tsx`
- `[NEW] src/app/page.tsx`
- `[NEW] .gitignore`
- `[NEW] .nvmrc` → `20`

**Steps**:
1. `npx -y create-next-app@latest ./ --typescript --app --eslint --src-dir --no-tailwind --import-alias "@/*"`
2. Verify `npm run dev` serves on `:3000`

**Acceptance Criteria**:
- [ ] Next.js 14 App Router project boots on `:3000`
- [ ] TypeScript strict mode enabled
- [ ] Path alias `@/*` resolves to `src/*`

---

### T-002: Add Express Backend Server

**Files**:
- `[NEW] src/server/index.ts` — Express app bootstrap
- `[NEW] src/server/routes/health.ts` — `GET /api/health`
- `[MODIFY] package.json` — add `express`, `cors`, `helmet`, `@types/express`, `tsx`
- `[NEW] tsconfig.server.json` — separate server tsconfig

**Steps**:
1. Install: `npm i express cors helmet`; `npm i -D @types/express @types/cors tsx`
2. Create Express app with `/api/health` returning `{ status: "ok", timestamp }`
3. Add dev script `"server": "tsx watch src/server/index.ts"`

**Acceptance Criteria**:
- [ ] Express runs on `:3001` with `npm run server`
- [ ] `/api/health` returns JSON with status and timestamp
- [ ] Helmet security headers applied
- [ ] CORS configured for `localhost:3000`

---

### T-003: Docker Compose for PostgreSQL + Redis

**Files**:
- `[NEW] docker-compose.yml`
- `[NEW] .env.example`
- `[NEW] .env` (gitignored)

**Steps**:
1. `docker-compose.yml` with `postgres:16` (DB: `venture_canvas`, user: `app`) and `redis:7-alpine`
2. `.env.example` with `DATABASE_URL`, `REDIS_URL`, `NEXTAUTH_SECRET`, `API_KEY_ENCRYPTION_KEY`
3. Copy to `.env` with dev-safe values

**Acceptance Criteria**:
- [ ] `docker compose up -d` starts both services
- [ ] PostgreSQL accessible on `:5432` with DB `venture_canvas`
- [ ] Redis accessible on `:6379`
- [ ] Data persisted via Docker volume
- [ ] `.env.example` documents all required environment variables

---

### T-008 (partial): ESLint + Prettier + CI Pipeline

**Files**:
- `[NEW] .eslintrc.json`
- `[NEW] .prettierrc`
- `[NEW] .husky/pre-commit`
- `[NEW] .github/workflows/ci.yml`
- `[MODIFY] package.json` — lint/format scripts

**Steps**:
1. Configure ESLint with TypeScript rules
2. Configure Prettier (single quotes, trailing commas, semi)
3. Husky pre-commit: `npm run lint && npm run format:check`
4. GitHub Actions: checkout → install → lint → type-check → build

**Acceptance Criteria**:
- [ ] `npm run lint` passes with no errors
- [ ] `npm run format:check` passes
- [ ] Pre-commit hook blocks commits with lint errors
- [ ] CI runs on every push and PR

## Final Verification

```bash
# Full agent verification sequence
docker compose up -d
npm run dev          # → localhost:3000 serves Next.js
npm run server       # → localhost:3001/api/health returns OK
npm run lint         # → passes
npm run build        # → succeeds
```

## Definition of Done

- [ ] All tasks T-001, T-002, T-003, T-008 (partial) completed
- [ ] `docker compose up && npm run dev` starts full local dev environment
- [ ] CI pipeline passes on push
- [ ] All files committed and pushed

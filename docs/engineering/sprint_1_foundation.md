# Sprint 1 — Foundation (Project Scaffolding + Database)

> **Goal**: Bootable development environment with full database schema, seeded data, and CI pipeline.

---

## T-001: Initialize Next.js + TypeScript Project

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

**Test**: `npm run build` succeeds; `localhost:3000` returns 200.

**Acceptance Criteria**:
- [ ] Next.js 14 App Router project boots on `:3000`
- [ ] TypeScript strict mode enabled
- [ ] Path alias `@/*` resolves to `src/*`

---

## T-002: Add Express Backend Server

**Files**:
- `[NEW] src/server/index.ts` — Express app bootstrap
- `[NEW] src/server/routes/health.ts` — `GET /api/health`
- `[MODIFY] package.json` — add `express`, `cors`, `helmet`, `@types/express`, `tsx`, dev script `"server": "tsx watch src/server/index.ts"`
- `[NEW] tsconfig.server.json` — separate server tsconfig

**Steps**:
1. Install deps: `npm i express cors helmet`; `npm i -D @types/express @types/cors tsx`
2. Create Express app with `/api/health` returning `{ status: "ok", timestamp }`
3. Configure CORS for `localhost:3000`

**Test**: `npm run server` starts on `:3001`; `curl localhost:3001/api/health` returns `{ "status": "ok" }`.

**Acceptance Criteria**:
- [ ] Express runs on `:3001` with `npm run server`
- [ ] `/api/health` returns JSON with status and timestamp
- [ ] Helmet security headers applied
- [ ] CORS configured for frontend origin

---

## T-003: Docker Compose for PostgreSQL + Redis

**Files**:
- `[NEW] docker-compose.yml`
- `[NEW] .env.example`
- `[NEW] .env` (gitignored)

**Steps**:
1. Create `docker-compose.yml` with `postgres:16` and `redis:7-alpine`
2. Create `.env.example` with `DATABASE_URL`, `REDIS_URL`, `NEXTAUTH_SECRET`, `API_KEY_ENCRYPTION_KEY`
3. Copy to `.env` with dev values

**Test**: `docker compose up -d` starts both services; `docker compose exec postgres psql -U app -d venture_canvas -c "SELECT 1"` returns 1.

**Acceptance Criteria**:
- [ ] PostgreSQL accessible on `:5432` with DB `venture_canvas`
- [ ] Redis accessible on `:6379`
- [ ] Data persisted via Docker volume
- [ ] `.env.example` documents all required vars

---

## T-004: Prisma Setup + Core Schema (users, workspaces, workspace_members)

**Files**:
- `[NEW] prisma/schema.prisma`
- `[MODIFY] package.json` — add `prisma`, `@prisma/client`
- `[NEW] src/shared/db/client.ts` — Prisma client singleton

**Steps**:
1. `npm i prisma @prisma/client`; `npx prisma init`
2. Define models: `User`, `Workspace`, `WorkspaceMember` per `02_database_schema.md`
3. Create Prisma client singleton with connection pooling

**Test**: `npx prisma migrate dev --name init` applies cleanly; `npx prisma studio` shows empty tables.

**Acceptance Criteria**:
- [ ] 3 tables created: `users`, `workspaces`, `workspace_members`
- [ ] Foreign keys and composite PK on `workspace_members`
- [ ] JSONB fields: `api_keys`, `llm_default_config` on workspaces
- [ ] Prisma client importable from `@/shared/db/client`

---

## T-005: Prisma Schema — Canvas, Nodes, Edges

**Files**:
- `[MODIFY] prisma/schema.prisma` — add Canvas, Node, Edge models

**Steps**:
1. Add `Canvas` model with FK to Workspace, JSONB `viewport` and `llm_config_override`
2. Add `Node` model with FK to Canvas, JSONB `config`, enum-like `type` and `state`
3. Add `Edge` model with FK to Canvas + source/target Nodes, unique constraint
4. Run migration

**Test**: `npx prisma migrate dev --name add_canvas_nodes_edges` applies; relationships navigable in Prisma Studio.

**Acceptance Criteria**:
- [ ] `canvases`, `nodes`, `edges` tables created
- [ ] `nodes.config` is JSONB with default `{}`
- [ ] `edges` has unique constraint on `(source_node_id, target_node_id)`
- [ ] Cascade delete: removing canvas removes its nodes and edges

---

## T-006: Prisma Schema — NodeDefinitions, CanvasVersions, Executions, WizardStates

**Files**:
- `[MODIFY] prisma/schema.prisma` — add remaining 4 models

**Steps**:
1. Add `NodeDefinition` with JSONB `default_llm_config`, `input_schema`, `output_schema`; self-referencing `forked_from`
2. Add `CanvasVersion` with JSONB `snapshot_data`
3. Add `Execution` with decimal `cost_usd`
4. Add `WizardState` with 1:1 relation to Node, JSONB arrays
5. Run migration

**Test**: `npx prisma migrate dev --name add_definitions_versions_executions` applies cleanly; all 10 tables visible.

**Acceptance Criteria**:
- [ ] All 10 tables from `02_database_schema.md` exist
- [ ] `node_definitions.forked_from_id` self-references
- [ ] `wizard_states` has unique constraint on `node_id` (1:1)
- [ ] `canvas_versions` indexed on `(canvas_id, created_at DESC)`

---

## T-007: Seed Script — Base Node Library + Dev User

**Files**:
- `[NEW] prisma/seed.ts`
- `[MODIFY] package.json` — add `prisma.seed` config

**Steps**:
1. Create seed script that inserts:
   - 1 dev user (`dev@upventure.com`)
   - 1 dev workspace (`upventure-dev`)
   - 5 base node definitions (NODE-PD-01, MR-01, BS-01, GTM-01, PITCH-01) with `is_base: true`, `visibility: 'public'`, placeholder prompts, and default LLM configs
2. Make idempotent (upsert)

**Test**: `npx prisma db seed` completes; re-running doesn't duplicate; `SELECT count(*) FROM node_definitions WHERE is_base = true` returns 5.

**Acceptance Criteria**:
- [ ] 5 base node definitions seeded with correct `node_id_code`
- [ ] Dev user and workspace created
- [ ] Seed is idempotent (safe to re-run)
- [ ] Wizard nodes (BS-01, GTM-01, PITCH-01) have `is_wizard: true`

---

## T-008: Repository Layer + CI Pipeline

**Files**:
- `[NEW] src/shared/db/repositories/base.repository.ts` — generic CRUD
- `[NEW] src/shared/db/repositories/user.repository.ts`
- `[NEW] src/shared/db/repositories/workspace.repository.ts`
- `[NEW] src/shared/db/repositories/canvas.repository.ts`
- `[NEW] .github/workflows/ci.yml`
- `[MODIFY] package.json` — add lint/test scripts

**Steps**:
1. Create `BaseRepository<T>` with `findById`, `findMany`, `create`, `update`, `delete`
2. Create entity-specific repos extending base with typed methods
3. Create GitHub Actions CI: checkout → install → lint → type-check → build
4. Add ESLint + Prettier config

**Test**: `npm run lint` passes; `npm run build` succeeds; push to branch triggers CI.

**Acceptance Criteria**:
- [ ] `BaseRepository` provides typed CRUD via Prisma
- [ ] Entity repos compile and export clean interfaces
- [ ] CI runs lint + type-check + build on every push
- [ ] ESLint + Prettier configured with consistent rules

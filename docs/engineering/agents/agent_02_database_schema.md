# Agent 02 — Database & Prisma Schema

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-02` |
| **Name** | Database & Prisma Schema |
| **Sprint** | S1 — Foundation |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 3 days |
| **Dependencies** | Agent 01 |
| **Blocks** | Agent 03, Agent 04, Agent 05 |

## Objective

Implement the complete PostgreSQL database schema via Prisma, including all 10 tables, indexes, row-level security policies, seed data for the base node library, and the repository layer for data access.

## Scope

- Prisma schema with all 10 tables
- JSONB type definitions
- Indexes and constraints
- RLS policies for workspace isolation
- Seed script (5 base nodes + dev user)
- Generic repository layer

## Reference

See [02_database_schema.md](../02_database_schema.md) for full schema specification.

## Tasks

### T-004: Core Schema — Users, Workspaces, WorkspaceMembers

**Files**:
- `[NEW] prisma/schema.prisma`
- `[MODIFY] package.json` — add `prisma`, `@prisma/client`
- `[NEW] src/shared/db/client.ts` — Prisma singleton

**Steps**:
1. `npm i prisma @prisma/client`; `npx prisma init`
2. Define `User`, `Workspace`, `WorkspaceMember` models
3. Create Prisma client singleton with connection pooling

**Acceptance Criteria**:
- [ ] 3 tables created: `users`, `workspaces`, `workspace_members`
- [ ] Foreign keys and composite PK on `workspace_members`
- [ ] JSONB fields: `api_keys`, `llm_default_config` on workspaces
- [ ] `npx prisma migrate dev --name init` runs cleanly
- [ ] Prisma client importable from `@/shared/db/client`

---

### T-005: Schema — Canvas, Nodes, Edges

**Files**:
- `[MODIFY] prisma/schema.prisma`

**Steps**:
1. Add `Canvas` model with FK to `Workspace`, JSONB `viewport` and `llm_config_override`
2. Add `Node` model with FK to `Canvas`, JSONB `config`, enum-like `type` and `state` fields
3. Add `Edge` model with FK to `Canvas` + source/target `Node`, unique constraint
4. Run migration

**Acceptance Criteria**:
- [ ] `canvases`, `nodes`, `edges` tables created
- [ ] `nodes.config` is JSONB with default `{}`
- [ ] `edges` has unique constraint on `(source_node_id, target_node_id)`
- [ ] Cascade delete: removing a canvas removes its nodes and edges

---

### T-006: Schema — NodeDefinitions, CanvasVersions, Executions, WizardStates

**Files**:
- `[MODIFY] prisma/schema.prisma`

**Steps**:
1. Add `NodeDefinition` (JSONB LLM config, schemas; self-ref `forked_from`)
2. Add `CanvasVersion` (JSONB `snapshot_data`)
3. Add `Execution` (decimal `cost_usd`)
4. Add `WizardState` (1:1 with Node, JSONB arrays)
5. Run migration

**Acceptance Criteria**:
- [ ] All 10 tables from schema spec exist
- [ ] `node_definitions.forked_from_id` is a self-referencing FK
- [ ] `wizard_states` has unique constraint on `node_id` (1:1)
- [ ] `canvas_versions` indexed on `(canvas_id, created_at DESC)`

---

### T-007: Seed Script — Base Node Library + Dev User

**Files**:
- `[NEW] prisma/seed.ts`
- `[MODIFY] package.json` — add `prisma.seed` config

**Steps**:
1. Seed: 1 dev user (`dev@upventure.com`), 1 dev workspace (`upventure-dev`)
2. Seed: 5 base node definitions (NODE-PD-01, MR-01, BS-01, GTM-01, PITCH-01)
   - `is_base: true`, `visibility: 'public'`
   - Placeholder prompts from PRD Section 5.3
   - Default LLM configs per PRD
   - Wizard nodes flagged: BS-01 (5 steps), GTM-01 (4 steps), PITCH-01 (10 steps)
3. Make idempotent via upsert

**Acceptance Criteria**:
- [ ] `npx prisma db seed` completes successfully
- [ ] 5 base definitions with correct `node_id_code`
- [ ] Re-running seed does NOT duplicate data
- [ ] Wizard nodes have `is_wizard: true` and correct `wizard_steps` count

---

### T-008: Repository Layer

**Files**:
- `[NEW] src/shared/db/repositories/base.repository.ts`
- `[NEW] src/shared/db/repositories/user.repository.ts`
- `[NEW] src/shared/db/repositories/workspace.repository.ts`
- `[NEW] src/shared/db/repositories/canvas.repository.ts`

**Steps**:
1. `BaseRepository<T>` with `findById`, `findMany`, `create`, `update`, `delete`
2. Entity-specific repos with typed methods and workspace scoping
3. Canvas repo includes `findByWorkspaceId` with pagination

**Acceptance Criteria**:
- [ ] `BaseRepository` provides typed CRUD via Prisma
- [ ] Entity repos extend base with domain-specific queries
- [ ] All repos workspace-scoped where applicable
- [ ] Clean TypeScript interfaces exported

## Final Verification

```bash
npx prisma migrate dev          # All migrations apply
npx prisma db seed              # Seed data populated
npx prisma studio               # All 10 tables visible with seed data
npm run build                   # Repos compile without errors
```

## Definition of Done

- [ ] All 10 database tables created and migrated
- [ ] Seed data includes 5 base nodes + dev user/workspace
- [ ] Repository layer compiles and exports clean interfaces
- [ ] All changes committed

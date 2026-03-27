# Sprint 6 — Testing, Polish & GA Launch

> **Goal**: Production-ready with E2E tests, security audit, onboarding, and deployment.

---

### T-063: Playwright E2E — Auth + Canvas CRUD

**Files**:
- `[NEW] e2e/auth.spec.ts`
- `[NEW] e2e/canvas-crud.spec.ts`
- `[NEW] e2e/fixtures.ts` — test user setup
- `[NEW] playwright.config.ts`
- `[MODIFY] package.json` — add `@playwright/test`, `test:e2e` script

**Test**: `npm run test:e2e` — register, login, create canvas, add/move/delete node, connect edges, delete canvas.

**Acceptance Criteria**:
- [ ] Auth flow: register → login → see dashboard → logout → redirect to login
- [ ] Canvas: create from template → add node → move node → connect edge → delete node → delete canvas
- [ ] Tests isolated (unique email per run)
- [ ] Runs in CI (headless Chromium)

---

### T-064: Playwright E2E — Node Execution + Export

**Files**:
- `[NEW] e2e/node-execution.spec.ts`
- `[NEW] e2e/document-export.spec.ts`

**Test**: Execute LLM node (mock LLM) → output appears; generate PDF from output node → download succeeds.

**Acceptance Criteria**:
- [ ] Execute node → streaming output visible → state=done
- [ ] Wizard node: gate → step 1 → confirm → step 2 → ... → done
- [ ] Output node: generate markdown → preview → download
- [ ] Tests use mocked LLM provider (no real API calls)

---

### T-065: Unit Test Coverage Sweep

**Files**:
- `[NEW] src/server/modules/*/\__tests__/*.test.ts` — one test file per service
- `[NEW] jest.config.ts`
- `[MODIFY] package.json` — `test:unit` script

**Test**: `npm run test:unit -- --coverage` → all services covered; threshold ≥ 80%.

**Acceptance Criteria**:
- [ ] Jest configured with TypeScript + Prisma mock
- [ ] Tests for: auth, canvas, node, edge (DAG validation), execution, wizard, LLM gateway, versioning, library, document service
- [ ] Coverage ≥ 80% lines on all service files
- [ ] CI runs unit tests on every push

---

### T-066: Performance Testing + Optimization

**Files**:
- `[NEW] e2e/performance.spec.ts`

**Test**: Load canvas with 50 nodes + 40 edges → measures load time, pan/zoom FPS, auto-save latency.

**Acceptance Criteria**:
- [ ] Canvas load ≤ 3s with 50 nodes (measure `DOMContentLoaded` → interactive)
- [ ] Pan/zoom 60fps (no frame drops measured via `PerformanceObserver`)
- [ ] Auto-save completes ≤ 2s from last change
- [ ] First LLM stream token ≤ 5s from execute click (with real provider)
- [ ] Optimizations applied if thresholds not met (React.memo, virtualization)

---

### T-067: Security Audit + RLS Verification

**Files**:
- `[NEW] e2e/security.spec.ts`
- `[MODIFY] prisma/migrations/*/migration.sql` — verify RLS policies

**Test**: User A creates canvas in workspace A; User B (workspace B) tries to access → 403; API key never appears in responses.

**Acceptance Criteria**:
- [ ] Cross-workspace access returns 403 for all endpoints
- [ ] API keys NEVER returned in any HTTP response
- [ ] JWT expiry enforced (expired token → 401)
- [ ] Rate limiting active (>100 req/min → 429)
- [ ] Helmet headers present (CSP, HSTS, X-Frame-Options)
- [ ] No SQL injection via JSONB query params

---

### T-068: RBAC Enforcement

**Files**:
- `[NEW] src/server/shared/middleware/rbac.middleware.ts`
- `[MODIFY] src/server/modules/workspace/workspace.routes.ts` — apply RBAC

**Test**: Member cannot delete workspace; admin cannot remove owner; owner can do everything.

**Acceptance Criteria**:
- [ ] Middleware `requireRole('admin')` checks membership role
- [ ] `owner`: all permissions
- [ ] `admin`: manage members (not owner), settings, canvases
- [ ] `member`: canvas CRUD, execute nodes, view workspace
- [ ] Insufficient role → 403 with clear message

---

### T-069: Onboarding Wizard

**Files**:
- `[NEW] src/components/onboarding/OnboardingWizard.tsx`
- `[NEW] src/components/onboarding/steps/*.tsx` — 4 step components
- `[MODIFY] src/app/(dashboard)/page.tsx` — trigger for new users

**Test**: New user registers → onboarding wizard starts → name workspace → add API key → create first canvas from template → arrive at canvas.

**Acceptance Criteria**:
- [ ] 4 steps: Welcome → Name Workspace → Add API Key (optional) → Choose Template
- [ ] Progress indicator (1/4, 2/4...)
- [ ] Skip button on each step
- [ ] API key step shows inline test (verify key works)
- [ ] Final step creates canvas and navigates to it
- [ ] Shown only once (flag stored on user)

---

### T-070: Production Dockerfile + Deploy Pipeline

**Files**:
- `[NEW] Dockerfile`
- `[NEW] Dockerfile.worker`
- `[MODIFY] .github/workflows/ci.yml` — add staging + production deploy stages

**Test**: `docker build -t venture-canvas .` succeeds; container starts and serves on `:3000`; health check passes.

**Acceptance Criteria**:
- [ ] Multi-stage build: builder (install + compile) → runner (slim image)
- [ ] Chromium installed for Puppeteer PDF generation
- [ ] Prisma migrations run on container start
- [ ] Separate `Dockerfile.worker` for BullMQ worker process
- [ ] CI: builds → pushes to GHCR → deploys to staging → E2E → manual promote to prod
- [ ] Zero-downtime deployment (health check + graceful shutdown)

---

### T-071: Monitoring + Alerting Setup

**Files**:
- `[NEW] src/server/shared/monitoring/sentry.ts`
- `[NEW] src/server/shared/monitoring/logger.ts` — Pino logger
- `[MODIFY] src/server/index.ts` — integrate Sentry + structured logging

**Test**: Throw error → appears in Sentry; request logged with duration; LLM execution logged with tokens/cost.

**Acceptance Criteria**:
- [ ] Sentry captures unhandled errors with stack trace + request context
- [ ] Pino structured JSON logging (timestamp, level, message, metadata)
- [ ] Log levels: `error`, `warn`, `info`, `debug` (configurable via env)
- [ ] Key events logged: auth, canvas CRUD, node execution, LLM calls, exports
- [ ] Uptime monitoring configured (external ping to `/api/health`)

---

### T-072: Remaining Templates + Launch Checklist

**Files**:
- `[NEW] prisma/templates/design-thinking.json`
- `[NEW] prisma/templates/jobs-to-be-done.json`
- `[NEW] prisma/templates/business-model-canvas.json`
- `[NEW] prisma/templates/innovation-sprint.json`

**Test**: Each template instantiates correctly via API; all nodes connect properly.

**Acceptance Criteria**:
- [ ] 4 additional templates: Design Thinking, JTBD, BMC, Innovation Sprint
- [ ] Each template: correct nodes, edges, positions, descriptions
- [ ] All 7 templates available in selection dialog
- [ ] **Launch checklist verified:**
  - [ ] All P0 requirements working
  - [ ] All P1 requirements working
  - [ ] E2E tests green
  - [ ] Security audit clean
  - [ ] Production deployment tested
  - [ ] Backup verified (restore test)
  - [ ] Monitoring alerts configured
  - [ ] Seed data (base library) included in production migration

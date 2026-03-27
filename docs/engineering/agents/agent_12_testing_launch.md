# Agent 12 — Testing, Polish & GA Launch

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-12` |
| **Name** | Testing, Polish & GA Launch |
| **Sprint** | S6 — Testing & Launch |
| **Milestone** | M3 — GA v1.0 |
| **Estimated Effort** | 8 days |
| **Dependencies** | Agent 10, Agent 11 |
| **Blocks** | None (final agent) |

## Objective

Ensure production readiness: comprehensive E2E test suite, ≥80% unit test coverage, performance validation against PRD targets, security audit with RLS verification, RBAC enforcement, onboarding wizard, production deployment pipeline, monitoring, remaining templates, and GA launch checklist.

## Tasks

### T-063: Playwright E2E — Auth + Canvas CRUD

**Files**:
- `[NEW] e2e/auth.spec.ts`
- `[NEW] e2e/canvas-crud.spec.ts`
- `[NEW] e2e/fixtures.ts`
- `[NEW] playwright.config.ts`

**Acceptance Criteria**:
- [ ] Auth: register → login → see dashboard → logout → redirect
- [ ] Canvas: create from template → add node → move → connect → delete
- [ ] Tests isolated (unique email per run)
- [ ] Runs headless in CI

---

### T-064: Playwright E2E — Execution + Export

**Files**:
- `[NEW] e2e/node-execution.spec.ts`
- `[NEW] e2e/document-export.spec.ts`

**Acceptance Criteria**:
- [ ] Execute node (mock LLM) → streaming → state=done
- [ ] Wizard: gate → step → confirm → ... → done
- [ ] Output node: generate markdown → preview → download
- [ ] No real LLM API calls in tests

---

### T-065: Unit Test Coverage ≥80%

**Files**:
- `[NEW] src/server/modules/*/__tests__/*.test.ts`
- `[NEW] jest.config.ts`

**Acceptance Criteria**:
- [ ] Jest + TypeScript + Prisma mock configured
- [ ] Tests: auth, canvas, node, edge (DAG), execution, wizard, LLM, versioning, library, documents
- [ ] Coverage ≥ 80% lines on all service files
- [ ] Part of CI

---

### T-066: Performance Testing

**Files**:
- `[NEW] e2e/performance.spec.ts`

**Acceptance Criteria**:
- [ ] Canvas load ≤ 3s with 50 nodes
- [ ] Pan/zoom 60fps (no frame drops)
- [ ] Auto-save ≤ 2s from last change
- [ ] First LLM token ≤ 5s from execute
- [ ] Optimizations applied if thresholds missed

---

### T-067: Security Audit + RLS

**Files**:
- `[NEW] e2e/security.spec.ts`

**Acceptance Criteria**:
- [ ] Cross-workspace access → 403 for all endpoints
- [ ] API keys never in HTTP responses
- [ ] Expired JWT → 401
- [ ] Rate limiting: >100 req/min → 429
- [ ] Helmet headers present
- [ ] No SQL injection via JSONB params

---

### T-068: RBAC Enforcement

**Files**:
- `[NEW] src/server/shared/middleware/rbac.middleware.ts`

**Acceptance Criteria**:
- [ ] `requireRole('admin')` middleware
- [ ] `owner`: all; `admin`: manage (not owner); `member`: canvas + execute
- [ ] Insufficient role → 403

---

### T-069: Onboarding Wizard

**Files**:
- `[NEW] src/components/onboarding/OnboardingWizard.tsx`
- `[NEW] src/components/onboarding/steps/*.tsx`

**Acceptance Criteria**:
- [ ] 4 steps: Welcome → Name Workspace → API Key → Choose Template
- [ ] Progress indicator; skip buttons
- [ ] API key step: inline verification test
- [ ] Final step creates canvas + navigates
- [ ] Shown once (user flag)

---

### T-070: Production Dockerfile + Deploy Pipeline

**Files**:
- `[NEW] Dockerfile`
- `[NEW] Dockerfile.worker`
- `[MODIFY] .github/workflows/ci.yml`

**Acceptance Criteria**:
- [ ] Multi-stage build (builder → slim runner)
- [ ] Chromium for Puppeteer PDF
- [ ] Prisma migrations on start
- [ ] Separate worker Dockerfile
- [ ] CI: build → push GHCR → staging → E2E → manual prod
- [ ] Zero-downtime with health checks

---

### T-071: Monitoring + Alerting

**Files**:
- `[NEW] src/server/shared/monitoring/sentry.ts`
- `[NEW] src/server/shared/monitoring/logger.ts`

**Acceptance Criteria**:
- [ ] Sentry captures errors with context
- [ ] Pino structured JSON logging (configurable level)
- [ ] Key events logged: auth, canvas, execution, LLM, export
- [ ] Uptime monitoring on `/api/health`

---

### T-072: Remaining Templates + Launch Checklist

**Files**:
- `[NEW] prisma/templates/design-thinking.json`
- `[NEW] prisma/templates/jobs-to-be-done.json`
- `[NEW] prisma/templates/business-model-canvas.json`
- `[NEW] prisma/templates/innovation-sprint.json`

**Acceptance Criteria**:
- [ ] 4 additional templates added (7 total)
- [ ] Each instantiates correctly

**Launch Checklist**:
- [ ] All P0 requirements working
- [ ] All P1 requirements working
- [ ] E2E tests green
- [ ] Security audit clean
- [ ] Production deployment tested
- [ ] Backup restore verified
- [ ] Monitoring alerts configured
- [ ] Base library seeded in production
- [ ] Onboarding wizard functional
- [ ] Documentation complete (API docs, user guide)

## Definition of Done

- [ ] All tests passing (E2E, unit, performance, security)
- [ ] Production deployment pipeline verified
- [ ] Launch checklist 100% complete
- [ ] Application ready for GA release

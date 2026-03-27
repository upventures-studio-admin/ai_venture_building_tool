# Agent 05 — Node System (Backend)

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-05` |
| **Name** | Node System (Backend) |
| **Sprint** | S3 — Nodes & LLM |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 5 days |
| **Dependencies** | Agent 02 |
| **Blocks** | Agent 06, Agent 07 |

## Objective

Build the complete backend for node and edge operations: CRUD APIs, BullMQ execution queue, prompt variable resolution, cascade execution logic, wizard finite state machine, and execution logging.

## Tasks

### T-023: Node CRUD API

**Files**:
- `[NEW] src/server/modules/nodes/node.service.ts`
- `[NEW] src/server/modules/nodes/node.controller.ts`
- `[NEW] src/server/modules/nodes/node.routes.ts`

**Acceptance Criteria**:
- [ ] CRUD for nodes scoped to canvas → workspace
- [ ] Delete cascades to connected edges
- [ ] `POST /nodes/:id/duplicate` copies config, offsets position by (20, 20)
- [ ] Validates `type` ∈ `{llm, input, output, custom}`

---

### T-024: Edge CRUD API + DAG Validation

**Files**:
- `[NEW] src/server/modules/nodes/edge.service.ts`
- `[NEW] src/server/modules/nodes/edge.controller.ts`
- `[NEW] src/server/shared/utils/dag.ts`

**Acceptance Criteria**:
- [ ] `POST /canvases/:id/edges` validates no cycle via topological sort
- [ ] Returns 400 with clear message on cycle detection
- [ ] `PATCH /edges/:id` updates mode (`auto|manual|visual`)
- [ ] Unique constraint prevents duplicate edges

---

### T-025: BullMQ Queue Setup

**Files**:
- `[NEW] src/server/shared/queue/queue.config.ts`
- `[NEW] src/server/shared/queue/node-execution.queue.ts`
- `[NEW] src/server/shared/queue/node-execution.worker.ts`

**Acceptance Criteria**:
- [ ] Queue `node-executions` with Redis connection
- [ ] Worker concurrency: 5
- [ ] Retry: 3 attempts, exponential backoff (2s base)
- [ ] Dead-letter queue for failed jobs
- [ ] `npm run worker` starts separate process

---

### T-026: Node Execution Service

**Files**:
- `[NEW] src/server/modules/nodes/execution.service.ts`

**Acceptance Criteria**:
- [ ] `executeNode(nodeId)` enqueues BullMQ job
- [ ] Worker: resolves prompt → calls LLM Gateway → stores output
- [ ] State transitions: `idle → running → done` / `error`
- [ ] Execution logged: model, tokens, cost, duration, state
- [ ] `POST /nodes/:id/stop` cancels running job
- [ ] `POST /nodes/:id/rerun` requires `confirmOverwrite: true`
- [ ] Timeout: 120s default

---

### T-027: Prompt Variable Resolution

**Files**:
- `[NEW] src/server/modules/nodes/prompt.resolver.ts`

**Acceptance Criteria**:
- [ ] `{input}` → node's `input_content` + concatenated upstream outputs
- [ ] `{context}` → all upstream outputs (separated by `\n---\n`)
- [ ] `{node_name}` → node label
- [ ] `{canvas_name}` → canvas name
- [ ] Multi-input: concatenates all incoming edge source outputs
- [ ] Unresolved variables logged as warnings

---

### T-028: Cascade Execution

**Files**:
- `[MODIFY] src/server/modules/nodes/execution.service.ts`
- `[MODIFY] src/server/shared/queue/node-execution.worker.ts`

**Acceptance Criteria**:
- [ ] `POST /canvases/:id/execute` topologically sorts → enqueues in order
- [ ] On completion, finds downstream `mode=auto` edges → enqueues
- [ ] `mode=manual` and `mode=visual` do NOT trigger downstream
- [ ] Progress: `{ completed: N, total: M }`

---

### T-029: Wizard FSM — Gate + Steps

**Files**:
- `[NEW] src/server/modules/nodes/wizard.service.ts`

**Acceptance Criteria**:
- [ ] Gate check validates required inputs against definition
- [ ] Gate fail → up to 6 clarifying questions returned
- [ ] Gate pass → enables step execution
- [ ] `POST /nodes/:id/wizard/advance` confirms step, executes next
- [ ] Step outputs stored in `wizard_states.step_outputs[]`
- [ ] Node stays `waiting` between steps (not `done`)
- [ ] Final confirmation → state = `done`, outputs concatenated

**Wizard FSM**:
```
[GATE_CHECK] → sufficient? → [STEP_1_RUNNING] → done → [STEP_1_WAITING]
  → user confirms → [STEP_2_RUNNING] → ... → [COMPLETED]
```

---

### T-030: Execution Log API

**Files**:
- `[NEW] src/shared/db/repositories/execution.repository.ts`

**Acceptance Criteria**:
- [ ] `GET /nodes/:id/executions` paginated (default 20, max 100)
- [ ] Fields: timestamp, model, tokens_in/out, cost_usd, duration_ms, state
- [ ] Filterable by state; sortable by `created_at` DESC

## Definition of Done

- [ ] Node + edge CRUD fully functional with DAG validation
- [ ] BullMQ queue processes executions with retry/timeout
- [ ] Cascade triggers downstream auto-mode nodes
- [ ] Wizard FSM stops between steps, requires user confirmation
- [ ] Execution history queryable

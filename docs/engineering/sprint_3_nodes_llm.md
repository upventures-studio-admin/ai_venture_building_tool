# Sprint 3 — Node System + LLM Gateway

> **Goal**: Nodes execute LLM calls with streaming, cascading, wizard gates, and cost tracking.

---

## Node System Backend (T-023 → T-030)

### T-023: Node CRUD API

**Files**:
- `[NEW] src/server/modules/nodes/node.service.ts`
- `[NEW] src/server/modules/nodes/node.controller.ts`
- `[NEW] src/server/modules/nodes/node.routes.ts`
- `[MODIFY] src/server/index.ts` — mount node routes

**Test**: `POST /canvases/:id/nodes` creates node; `PATCH /nodes/:id` updates position/config; `DELETE /nodes/:id` removes node + its edges.

**Acceptance Criteria**:
- [ ] CRUD for nodes scoped to canvas → workspace
- [ ] Delete node cascades to remove connected edges
- [ ] `POST /nodes/:id/duplicate` copies all config, offsets position by (20, 20)
- [ ] Validation: node type must be one of `llm|input|output|custom`

---

### T-024: Edge CRUD API + DAG Validation

**Files**:
- `[NEW] src/server/modules/nodes/edge.service.ts`
- `[NEW] src/server/modules/nodes/edge.controller.ts`
- `[NEW] src/server/shared/utils/dag.ts` — server-side topological sort
- `[MODIFY] src/server/modules/nodes/node.routes.ts` — edge endpoints

**Test**: Create edge → 201; create cycle → 400 `{ error: "Cycle detected" }`; `PATCH` changes mode; `DELETE` removes.

**Acceptance Criteria**:
- [ ] `POST /canvases/:id/edges` validates no cycle via topological sort
- [ ] Returns 400 with clear message if cycle detected
- [ ] `PATCH /edges/:id` updates mode (`auto|manual|visual`)
- [ ] Unique constraint prevents duplicate edges

---

### T-025: BullMQ Queue Setup

**Files**:
- `[NEW] src/server/shared/queue/queue.config.ts` — BullMQ connection
- `[NEW] src/server/shared/queue/node-execution.queue.ts` — queue definition
- `[NEW] src/server/shared/queue/node-execution.worker.ts` — worker process
- `[MODIFY] package.json` — add `bullmq`

**Test**: Enqueue a job → worker picks it up → logs processing; job with failure retries 3 times with backoff.

**Acceptance Criteria**:
- [ ] Queue `node-executions` created with Redis connection
- [ ] Worker processes jobs with concurrency 5
- [ ] Retry: 3 attempts, exponential backoff (2s base)
- [ ] Failed jobs move to dead-letter queue after max retries
- [ ] Separate `npm run worker` command to start worker process

---

### T-026: Node Execution Service

**Files**:
- `[NEW] src/server/modules/nodes/execution.service.ts`
- `[MODIFY] src/server/modules/nodes/node.routes.ts` — execution endpoints

**Test**: `POST /nodes/:id/execute` → node state changes idle→running→done; output persisted; execution record created with tokens/cost.

**Acceptance Criteria**:
- [ ] `executeNode(nodeId)` enqueues BullMQ job
- [ ] Worker: resolves prompt variables → calls LLM Gateway → stores output
- [ ] Node state updated: `idle → running → done` (or `error`)
- [ ] Execution logged: model, tokens_in, tokens_out, cost_usd, duration_ms
- [ ] `POST /nodes/:id/stop` cancels running job
- [ ] `POST /nodes/:id/rerun` requires `confirmOverwrite: true`
- [ ] Timeout at 120s → state=error

---

### T-027: Prompt Variable Resolution

**Files**:
- `[NEW] src/server/modules/nodes/prompt.resolver.ts`

**Test**: Prompt `"Analyze {input} in context of {context}"` with input="problem X" and upstream output="market data Y" → resolves to full string.

**Acceptance Criteria**:
- [ ] `{input}` replaced with node's `input_content` + concatenated upstream outputs
- [ ] `{context}` replaced with all upstream node outputs (separated by `\n---\n`)
- [ ] `{node_name}` replaced with node label
- [ ] `{canvas_name}` replaced with canvas name
- [ ] Unresolved variables left as-is with warning log
- [ ] Multi-input: concatenates all incoming edge source outputs

---

### T-028: Cascade Execution

**Files**:
- `[MODIFY] src/server/modules/nodes/execution.service.ts` — add cascade logic
- `[MODIFY] src/server/shared/queue/node-execution.worker.ts` — trigger downstream

**Test**: A→B→C with auto edges; execute A → B auto-executes after A completes → C auto-executes after B. Manual edge between B→D does NOT trigger D.

**Acceptance Criteria**:
- [ ] `POST /canvases/:id/execute` topologically sorts → enqueues in order
- [ ] After node completes, find downstream edges with `mode=auto`
- [ ] Enqueue downstream nodes in dependency order
- [ ] `mode=manual` edges do NOT trigger downstream
- [ ] `mode=visual` edges do NOT trigger downstream
- [ ] Progress reported: `{ completed: 2, total: 5 }`

---

### T-029: Wizard FSM — Gate + Steps

**Files**:
- `[NEW] src/server/modules/nodes/wizard.service.ts`
- `[MODIFY] src/server/modules/nodes/node.routes.ts` — wizard endpoints

**Test**: Wizard node BS-01: gate check with missing input → returns questions; provide answers → gate passes → step 1 executes → stops → advance → step 2 → ... → step 5 → completed.

**Acceptance Criteria**:
- [ ] `POST /nodes/:id/wizard/gate` checks required inputs against definition's gate criteria
- [ ] Gate fail → returns up to 6 clarifying questions
- [ ] Gate pass → sets `gate_status: 'passed'`, enables step execution
- [ ] `POST /nodes/:id/wizard/advance` confirms current step, executes next
- [ ] Each step output stored in `wizard_states.step_outputs[]` JSONB array
- [ ] Node stays in `waiting` state between steps (not `done`)
- [ ] Final step completion → node state = `done`, all step outputs concatenated to `output_content`

---

### T-030: Execution Log API

**Files**:
- `[MODIFY] src/server/modules/nodes/node.routes.ts` — `GET /nodes/:id/executions`
- `[NEW] src/shared/db/repositories/execution.repository.ts`

**Test**: Execute node 3 times → `GET /nodes/:id/executions` returns 3 records with timestamps, models, tokens, costs.

**Acceptance Criteria**:
- [ ] Returns paginated execution history (default 20, max 100)
- [ ] Each record: timestamp, model, tokens_in, tokens_out, cost_usd, duration_ms, state
- [ ] Filterable by state (`done|error`)
- [ ] Sortable by `created_at` (default DESC)

---

## LLM Gateway (T-031 → T-036)

### T-031: Provider — Anthropic Client

**Files**:
- `[NEW] src/server/modules/llm/providers/anthropic.provider.ts`
- `[NEW] src/server/modules/llm/types.ts` — shared types
- `[MODIFY] package.json` — add `@anthropic-ai/sdk`

**Test**: With valid API key, call Claude Sonnet → receive streamed response chunks; count tokens; invalid key returns clear error.

**Acceptance Criteria**:
- [ ] `stream(request: LLMRequest)` returns `AsyncGenerator<StreamChunk>`
- [ ] `StreamChunk`: `{ text, done, usage?: { inputTokens, outputTokens } }`
- [ ] Maps to Anthropic Messages API with system prompt
- [ ] Handles rate limit (429) as transient error
- [ ] Handles auth error (401) as permanent error

---

### T-032: Provider — OpenAI Client

**Files**:
- `[NEW] src/server/modules/llm/providers/openai.provider.ts`
- `[MODIFY] package.json` — add `openai`

**Test**: Same streaming test with GPT-4o; token counts returned.

**Acceptance Criteria**:
- [ ] Same `stream()` interface as Anthropic provider
- [ ] Maps to OpenAI Chat Completions API with streaming
- [ ] Supports GPT-4o, GPT-4 Turbo, o1/o3 models
- [ ] Token usage extracted from stream final chunk

---

### T-033: Provider — Google GenAI Client

**Files**:
- `[NEW] src/server/modules/llm/providers/google.provider.ts`
- `[MODIFY] package.json` — add `@google/generative-ai`

**Test**: Same streaming test with Gemini 1.5 Flash.

**Acceptance Criteria**:
- [ ] Same `stream()` interface
- [ ] Maps to Gemini generateContentStream API
- [ ] Supports Gemini 1.5 Pro and Flash

---

### T-034: LLM Gateway — Unified Interface + Config Resolution

**Files**:
- `[NEW] src/server/modules/llm/llm.gateway.ts`
- `[NEW] src/server/modules/llm/config.resolver.ts`

**Test**: `gateway.execute(nodeId)` resolves config (node>canvas>workspace), selects provider, decrypts API key, streams response.

**Acceptance Criteria**:
- [ ] `resolveConfig(nodeId)` merges node > canvas > workspace LLM configs
- [ ] `execute(nodeId)` selects provider based on resolved config
- [ ] Decrypts workspace API key for resolved provider
- [ ] Returns `AsyncGenerator<StreamChunk>` regardless of provider
- [ ] Missing API key → clear error: "No API key configured for {provider}"

---

### T-035: Context Window Management

**Files**:
- `[NEW] src/server/modules/llm/context.manager.ts`

**Test**: Input exceeding model's context window → auto-truncated from beginning; notification included in response metadata.

**Acceptance Criteria**:
- [ ] Token limit lookup per model (e.g. Claude Opus=200k, GPT-4o=128k)
- [ ] `estimateTokens(text)` uses tiktoken or character heuristic
- [ ] If input > limit: truncate oldest content, keep system prompt intact
- [ ] Return `{ truncated: true, originalTokens, truncatedTokens }` in metadata
- [ ] Log warning when truncation occurs

---

### T-036: Cost Tracking

**Files**:
- `[NEW] src/server/modules/llm/cost.calculator.ts`

**Test**: 1000 input tokens + 500 output tokens on GPT-4o → calculates correct USD cost per pricing table.

**Acceptance Criteria**:
- [ ] Pricing table: per-model input/output cost per 1M tokens
- [ ] `calculateCost(model, tokensIn, tokensOut)` → `number` (USD)
- [ ] Covers all supported models (Claude Opus/Sonnet/Haiku, GPT-4o/Turbo/o1, Gemini Pro/Flash)
- [ ] Pricing table easily updatable (single config file)
- [ ] Returns 0 for unknown models with warning log

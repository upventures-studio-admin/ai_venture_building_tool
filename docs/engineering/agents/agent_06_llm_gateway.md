# Agent 06 — LLM Gateway

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-06` |
| **Name** | LLM Gateway |
| **Sprint** | S3 — Nodes & LLM |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 4 days |
| **Dependencies** | Agent 03, Agent 05 |
| **Blocks** | Agent 08, Agent 09 |

## Objective

Build a unified abstraction over 3 LLM providers (Anthropic, OpenAI, Google) with streaming, 3-level config resolution, API key decryption, context window management, and cost tracking.

## Tasks

### T-031: Anthropic Provider Client

**Files**:
- `[NEW] src/server/modules/llm/providers/anthropic.provider.ts`
- `[NEW] src/server/modules/llm/types.ts`

**Acceptance Criteria**:
- [ ] `stream(request)` → `AsyncGenerator<StreamChunk>`
- [ ] `StreamChunk: { text, done, usage?: { inputTokens, outputTokens } }`
- [ ] Maps to Anthropic Messages API with system prompt
- [ ] Rate limit (429) classified as transient; auth error (401) as permanent

---

### T-032: OpenAI Provider Client

**Files**:
- `[NEW] src/server/modules/llm/providers/openai.provider.ts`

**Acceptance Criteria**:
- [ ] Same `stream()` interface as Anthropic
- [ ] Maps to Chat Completions API (streaming)
- [ ] Supports GPT-4o, GPT-4 Turbo, o1/o3
- [ ] Token usage from final stream chunk

---

### T-033: Google GenAI Provider Client

**Files**:
- `[NEW] src/server/modules/llm/providers/google.provider.ts`

**Acceptance Criteria**:
- [ ] Same `stream()` interface
- [ ] Maps to Gemini `generateContentStream`
- [ ] Supports Gemini 1.5 Pro and Flash

---

### T-034: Unified Gateway + Config Resolution

**Files**:
- `[NEW] src/server/modules/llm/llm.gateway.ts`
- `[NEW] src/server/modules/llm/config.resolver.ts`

**Acceptance Criteria**:
- [ ] `resolveConfig(nodeId)` merges: node > canvas > workspace
- [ ] `execute(nodeId)` selects provider, decrypts API key, streams
- [ ] Returns `AsyncGenerator<StreamChunk>` regardless of provider
- [ ] Missing API key → clear error message

---

### T-035: Context Window Management

**Files**:
- `[NEW] src/server/modules/llm/context.manager.ts`

**Acceptance Criteria**:
- [ ] Token limit lookup per model
- [ ] `estimateTokens(text)` via tiktoken or heuristic
- [ ] Auto-truncation (oldest content first, system prompt preserved)
- [ ] Returns `{ truncated: true, originalTokens, truncatedTokens }` metadata

---

### T-036: Cost Calculator

**Files**:
- `[NEW] src/server/modules/llm/cost.calculator.ts`

**Acceptance Criteria**:
- [ ] Pricing table: per-model $/1M tokens (input + output)
- [ ] `calculateCost(model, tokensIn, tokensOut)` → USD
- [ ] Covers all supported models
- [ ] Easily updatable config file
- [ ] Returns 0 + warning for unknown models

## Definition of Done

- [ ] All 3 providers stream responses via unified interface
- [ ] Config resolution merges 3 levels correctly
- [ ] API keys decrypted at execution time only
- [ ] Oversized inputs truncated with metadata notification
- [ ] Cost calculated and logged per execution

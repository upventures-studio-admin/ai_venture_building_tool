# Agent 07 — Real-time Layer

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-07` |
| **Name** | Real-time Layer |
| **Sprint** | S4 — Real-time & Versioning |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 3 days |
| **Dependencies** | Agent 04, Agent 05 |
| **Blocks** | Agent 08 |

## Objective

Set up Socket.IO for real-time communication: LLM streaming relay (token-by-token output in UI), node state change broadcasts, auto-save status indicator, and canvas room management.

## Tasks

### T-037: Socket.IO Server Setup

**Files**:
- `[NEW] src/server/modules/realtime/socket.server.ts`
- `[MODIFY] src/server/index.ts`

**Acceptance Criteria**:
- [ ] Socket.IO mounted on Express HTTP server
- [ ] JWT auth middleware on socket handshake
- [ ] `canvas:join` → joins room `canvas:{id}`; `canvas:leave` → leaves
- [ ] Connection/disconnection logged

---

### T-038: Node State Change Events

**Files**:
- `[NEW] src/server/modules/realtime/canvas.emitter.ts`
- `[MODIFY] src/server/shared/queue/node-execution.worker.ts`

**Acceptance Criteria**:
- [ ] `emitToCanvas(canvasId, event, payload)` helper function
- [ ] Worker emits `node:stateChange` on state transitions (idle→running, running→done/error)
- [ ] Only clients in canvas room receive events

---

### T-039: LLM Stream Relay

**Files**:
- `[MODIFY] src/server/modules/realtime/canvas.emitter.ts`
- `[MODIFY] src/server/shared/queue/node-execution.worker.ts`

**Acceptance Criteria**:
- [ ] Each LLM `StreamChunk` piped as `node:streamChunk` socket event
- [ ] Payload: `{ nodeId, executionId, chunk, done }`
- [ ] Final event includes `usage: { tokensIn, tokensOut, costUsd }`

---

### T-040: Frontend Socket Client + Stream UI

**Files**:
- `[NEW] src/hooks/useCanvasSocket.ts`
- `[MODIFY] src/components/canvas/nodes/LLMNode.tsx`

**Acceptance Criteria**:
- [ ] Hook connects on canvas mount, disconnects on unmount
- [ ] `node:streamChunk` appends text to output panel (typing effect)
- [ ] `node:stateChange` updates badge in Zustand store
- [ ] Connection status in canvas header

---

### T-041: Save Status Indicator

**Files**:
- `[NEW] src/components/canvas/panels/SaveStatus.tsx`

**Acceptance Criteria**:
- [ ] 3 states: `Saved` (green), `Saving...` (yellow pulse), `Error` (red)
- [ ] Reacts to `canvas:saved` socket event
- [ ] Updates on local changes (dirty) and server confirm (clean)

## Definition of Done

- [ ] LLM output streams token-by-token in the node UI
- [ ] Node state badges update in real-time
- [ ] Save indicator reflects server-confirmed state
- [ ] Socket reconnects gracefully on network interruption

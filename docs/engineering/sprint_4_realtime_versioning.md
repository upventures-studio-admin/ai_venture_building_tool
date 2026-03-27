# Sprint 4 — Real-time + Auto-save + Versioning

> **Goal**: Live streaming UI, auto-save within 2s, snapshot history with rollback.

---

## Real-time Layer (T-037 → T-041)

### T-037: Socket.IO Server Setup

**Files**:
- `[NEW] src/server/modules/realtime/socket.server.ts`
- `[MODIFY] src/server/index.ts` — attach Socket.IO to HTTP server
- `[MODIFY] package.json` — add `socket.io`, `socket.io-client`

**Test**: Client connects to `ws://localhost:3001`; joins room `canvas:abc`; server logs connection; disconnect cleans up.

**Acceptance Criteria**:
- [ ] Socket.IO server mounted on Express HTTP server
- [ ] JWT auth middleware on socket handshake
- [ ] `canvas:join` → joins room; `canvas:leave` → leaves room
- [ ] Connection/disconnection logged

---

### T-038: Node State Change Events

**Files**:
- `[MODIFY] src/server/modules/realtime/socket.server.ts` — emit helpers
- `[NEW] src/server/modules/realtime/canvas.emitter.ts`
- `[MODIFY] src/server/shared/queue/node-execution.worker.ts` — emit on state changes

**Test**: Execute node → client in same canvas room receives `node:stateChange` events for `running` and `done`.

**Acceptance Criteria**:
- [ ] `emitToCanvas(canvasId, event, payload)` helper
- [ ] Worker emits `node:stateChange` on idle→running, running→done, running→error
- [ ] Only clients in the canvas room receive events

---

### T-039: LLM Stream Relay to Frontend

**Files**:
- `[MODIFY] src/server/modules/realtime/canvas.emitter.ts` — stream chunk emission
- `[MODIFY] src/server/shared/queue/node-execution.worker.ts` — pipe stream chunks

**Test**: Execute LLM node → client receives `node:streamChunk` events token-by-token; final chunk has `done: true`.

**Acceptance Criteria**:
- [ ] Each `StreamChunk` from LLM Gateway piped as `node:streamChunk` event
- [ ] Payload: `{ nodeId, executionId, chunk: string, done: boolean }`
- [ ] Final event includes `usage: { tokensIn, tokensOut, costUsd }`

---

### T-040: Frontend Socket Client + Stream Display

**Files**:
- `[NEW] src/hooks/useCanvasSocket.ts`
- `[MODIFY] src/components/canvas/nodes/LLMNode.tsx` — wire stream chunks to output panel

**Test**: Execute node from UI → output panel shows text appearing token-by-token; state badge updates in real-time.

**Acceptance Criteria**:
- [ ] Hook connects on canvas mount, disconnects on unmount
- [ ] `node:streamChunk` appends text to node's output panel with typing effect
- [ ] `node:stateChange` updates node badge in Zustand store
- [ ] Connection status indicator in canvas header

---

### T-041: Save Status Indicator

**Files**:
- `[NEW] src/components/canvas/panels/SaveStatus.tsx`

**Test**: Edit node → indicator shows "Saving..." → shows "Saved ✓" after server confirms; network disconnect → shows "Offline".

**Acceptance Criteria**:
- [ ] 3 states: `Saved` (green), `Saving...` (yellow pulse), `Error` (red)
- [ ] Listens to `canvas:saved` socket event
- [ ] Updates on local change (dirty) and server confirm (clean)
- [ ] Positioned in canvas header bar

---

## Auto-save + Versioning (T-042 → T-048)

### T-042: Auto-save — Client Debounce + Server Persist

**Files**:
- `[NEW] src/hooks/useAutoSave.ts` — debounced save hook
- `[NEW] src/server/modules/canvas/autosave.service.ts`
- `[MODIFY] src/server/modules/canvas/canvas.routes.ts` — `POST /canvases/:id/auto-save`

**Test**: Move 3 nodes rapidly → only 1 save request after 2s idle; server persists; response confirms timestamp.

**Acceptance Criteria**:
- [ ] Client debounces changes with 2s delay
- [ ] Sends delta payload: `{ nodes: [changed], edges: [changed], viewport }`
- [ ] Server merges delta into database
- [ ] Returns `{ savedAt }` + emits `canvas:saved` via socket
- [ ] Retries once on network failure

---

### T-043: Auto-snapshot on Discrete Events

**Files**:
- `[NEW] src/server/modules/versioning/versioning.service.ts`
- `[NEW] src/server/modules/versioning/snapshot.builder.ts`

**Test**: Add node → snapshot created with `event_type: 'node_add'`; execute node → snapshot with `event_type: 'node_run'`.

**Acceptance Criteria**:
- [ ] Snapshot on: `node_add`, `node_remove`, `node_run`, `edge_add`, `edge_remove`
- [ ] Snapshot stored as JSON diff (delta from last full snapshot)
- [ ] Full snapshot materialized every 10th event
- [ ] Emits `canvas:versionCreated` socket event

---

### T-044: Manual Snapshots

**Files**:
- `[MODIFY] src/server/modules/versioning/versioning.service.ts`
- `[MODIFY] src/server/modules/canvas/canvas.routes.ts` — `POST /canvases/:id/versions`

**Test**: `POST /canvases/:id/versions { name: "Post-workshop" }` → creates named snapshot; appears in history.

**Acceptance Criteria**:
- [ ] Creates full snapshot (not delta)
- [ ] Optional `name` field stored
- [ ] `event_type: 'manual'`
- [ ] No retention limit (persisted indefinitely)

---

### T-045: History Panel API

**Files**:
- `[MODIFY] src/server/modules/versioning/versioning.service.ts`
- `[NEW] src/server/modules/versioning/versioning.controller.ts`
- `[NEW] src/server/modules/versioning/versioning.routes.ts`
- `[MODIFY] src/server/index.ts` — mount routes

**Test**: `GET /canvases/:id/versions?page=1&limit=20` returns paginated snapshots sorted by newest first.

**Acceptance Criteria**:
- [ ] Paginated (default 20, max 100)
- [ ] Filterable by `eventType` and `authorId`
- [ ] Returns: id, event_type, name, author, created_at (NOT snapshot_data)
- [ ] `GET /versions/:id` returns full reconstructed snapshot data

---

### T-046: Rollback

**Files**:
- `[MODIFY] src/server/modules/versioning/versioning.service.ts` — rollback logic

**Test**: Create 3 snapshots → rollback to #1 → canvas state matches snapshot #1; auto-snapshot of pre-rollback state created.

**Acceptance Criteria**:
- [ ] `POST /canvases/:id/rollback { versionId }` with confirmation
- [ ] Auto-creates snapshot of current state BEFORE rollback (`event_type: 'rollback'`)
- [ ] Restores canvas nodes, edges, viewport from target snapshot
- [ ] Broadcasts updated canvas to all connected clients
- [ ] Returns restored canvas state

---

### T-047: Frontend — History Panel + Snapshot Viewer

**Files**:
- `[NEW] src/components/canvas/panels/HistoryPanel.tsx`
- `[NEW] src/components/canvas/panels/SnapshotViewer.tsx`

**Test**: Open history → list of snapshots; click snapshot → read-only canvas view; click "Restore" → confirm → canvas restored.

**Acceptance Criteria**:
- [ ] Sidebar panel with chronological snapshot list
- [ ] Each entry: icon (auto/manual), name/event_type, author, timestamp
- [ ] Click entry → renders snapshot as read-only overlay
- [ ] "Restore this version" button with confirmation dialog
- [ ] "Save checkpoint" button for manual snapshots

---

### T-048: Snapshot Cleanup Cron

**Files**:
- `[NEW] src/server/shared/cron/snapshot-cleanup.ts`
- `[MODIFY] src/server/index.ts` — register cron

**Test**: Create auto-snapshot with `created_at` 91 days ago → cron deletes it; manual snapshot with same age → NOT deleted.

**Acceptance Criteria**:
- [ ] Runs daily at 03:00 UTC
- [ ] Deletes auto-snapshots (`event_type != 'manual'`) older than 90 days
- [ ] Manual snapshots never deleted
- [ ] Logs count of deleted snapshots

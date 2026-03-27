# Agent 08 — Auto-save & Versioning

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-08` |
| **Name** | Auto-save & Versioning |
| **Sprint** | S4 — Real-time & Versioning |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 4 days |
| **Dependencies** | Agent 06, Agent 07 |
| **Blocks** | Agent 10 |

## Objective

Implement debounced auto-save (≤2s), event-driven and periodic snapshots (delta-based with periodic full materialization), history browsing, rollback, and snapshot cleanup.

## Tasks

### T-042: Auto-save — Client Debounce + Server Persist

**Files**:
- `[NEW] src/hooks/useAutoSave.ts`
- `[NEW] src/server/modules/canvas/autosave.service.ts`
- `[MODIFY] src/server/modules/canvas/canvas.routes.ts`

**Acceptance Criteria**:
- [ ] Client debounces changes with 2s delay
- [ ] Sends delta: `{ nodes: [changed], edges: [changed], viewport }`
- [ ] Server merges into database, returns `{ savedAt }`
- [ ] Emits `canvas:saved` via socket
- [ ] Retries once on network failure

---

### T-043: Auto-snapshot on Discrete Events

**Files**:
- `[NEW] src/server/modules/versioning/versioning.service.ts`
- `[NEW] src/server/modules/versioning/snapshot.builder.ts`

**Acceptance Criteria**:
- [ ] Triggers on: `node_add`, `node_remove`, `node_run`, `edge_add`, `edge_remove`
- [ ] Stored as JSON diff (delta from last full snapshot)
- [ ] Full snapshot materialized every 10th event
- [ ] Emits `canvas:versionCreated` socket event

---

### T-044: Manual Snapshots

**Files**:
- `[MODIFY] src/server/modules/versioning/versioning.service.ts`

**Acceptance Criteria**:
- [ ] `POST /canvases/:id/versions { name? }` creates full snapshot
- [ ] `event_type: 'manual'`; no retention limit
- [ ] Optional user-defined name

---

### T-045: History Panel API

**Files**:
- `[NEW] src/server/modules/versioning/versioning.controller.ts`
- `[NEW] src/server/modules/versioning/versioning.routes.ts`

**Acceptance Criteria**:
- [ ] `GET /canvases/:id/versions` paginated, filterable by eventType/authorId
- [ ] Returns metadata only (no snapshot_data in list)
- [ ] `GET /versions/:id` returns full reconstructed snapshot

---

### T-046: Rollback

**Files**:
- `[MODIFY] src/server/modules/versioning/versioning.service.ts`

**Acceptance Criteria**:
- [ ] `POST /canvases/:id/rollback { versionId }`
- [ ] Auto-creates safety snapshot of current state first
- [ ] Restores nodes, edges, viewport from target
- [ ] Broadcasts updated canvas to all clients

---

### T-047: Frontend — History Panel + Snapshot Viewer

**Files**:
- `[NEW] src/components/canvas/panels/HistoryPanel.tsx`
- `[NEW] src/components/canvas/panels/SnapshotViewer.tsx`

**Acceptance Criteria**:
- [ ] Sidebar with chronological snapshot list
- [ ] Each entry: icon, name/event_type, author, timestamp
- [ ] Click → read-only canvas overlay
- [ ] "Restore this version" with confirm dialog
- [ ] "Save checkpoint" button for manual snapshots

---

### T-048: Snapshot Cleanup Cron

**Files**:
- `[NEW] src/server/shared/cron/snapshot-cleanup.ts`

**Acceptance Criteria**:
- [ ] Runs daily at 03:00 UTC
- [ ] Deletes auto-snapshots older than 90 days
- [ ] Manual snapshots retained indefinitely
- [ ] Logs deletion count

## Definition of Done

- [ ] Changes auto-save within 2s of last edit
- [ ] Snapshots created on discrete events + every 30s
- [ ] History panel shows browsable timeline
- [ ] Rollback restores canvas with safety snapshot
- [ ] Cleanup cron runs without errors

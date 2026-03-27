# Agent 09 — Library Service

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-09` |
| **Name** | Library Service |
| **Sprint** | S5 — Library, Docs, Templates |
| **Milestone** | M2 — Beta |
| **Estimated Effort** | 4 days |
| **Dependencies** | Agent 06 |
| **Blocks** | Agent 11 |

## Objective

Build the full node library system: backend API for CRUD, fork, import/export, search/filter, plus the frontend UI for browsing definitions, creating/editing custom nodes, saving nodes from canvas, and viewing version history.

## Tasks

### T-049: Library API — CRUD + Search

**Files**:
- `[NEW] src/server/modules/library/library.service.ts`
- `[NEW] src/server/modules/library/library.controller.ts`
- `[NEW] src/server/modules/library/library.routes.ts`

**Acceptance Criteria**:
- [ ] `GET /library/definitions` returns public + workspace-private
- [ ] Filterable: `workspaceId`, `category`, `type`, `visibility`, `search`
- [ ] `POST` creates private definition
- [ ] `PATCH` increments `version`
- [ ] Base definitions (`is_base: true`) → 403 on delete

---

### T-050: Fork + Import/Export

**Files**:
- `[MODIFY] src/server/modules/library/library.service.ts`
- `[MODIFY] src/server/modules/library/library.routes.ts`

**Acceptance Criteria**:
- [ ] `POST /:id/fork` → private copy with `forked_from_id`
- [ ] `GET /:id/export` → JSON with full schema
- [ ] `POST /import` validates JSON → creates definition
- [ ] Invalid JSON → descriptive validation errors

---

### T-051: Library Panel — API-Connected Browsing

**Files**:
- `[MODIFY] src/components/canvas/panels/LibraryPanel.tsx`
- `[NEW] src/components/canvas/panels/NodePreviewCard.tsx`

**Acceptance Criteria**:
- [ ] Fetches from API via React Query
- [ ] Category tabs, debounced search (300ms)
- [ ] Preview card: name, description, type badge, I/O summary
- [ ] Drag-to-canvas creates node with `definition_id` and pre-filled config

---

### T-052: Custom Node Editor

**Files**:
- `[NEW] src/components/library/NodeEditor.tsx`
- `[NEW] src/components/library/NodeEditor.css`
- `[NEW] src/app/(dashboard)/library/page.tsx`

**Acceptance Criteria**:
- [ ] Form: name, description, category, type, prompt (with `{variable}` helpers), LLM config, tags
- [ ] Prompt editor with syntax highlighting for template variables
- [ ] Save creates private definition; edit increments version
- [ ] Validation: name and prompt required

---

### T-053: Save Canvas Node to Library

**Files**:
- `[NEW] src/components/canvas/nodes/SaveToLibraryDialog.tsx`
- `[MODIFY] src/components/canvas/nodes/LLMNode.tsx`

**Acceptance Criteria**:
- [ ] Right-click node → "Save to Library" context menu
- [ ] Dialog pre-fills from current node config
- [ ] Saves as private definition in workspace
- [ ] Success toast with link to library page

---

### T-054: Node Versioning UI

**Files**:
- `[NEW] src/components/library/VersionHistory.tsx`

**Acceptance Criteria**:
- [ ] Version list on definition detail page
- [ ] Each: version number, timestamp, author
- [ ] Click → read-only view of that version's config
- [ ] Current version highlighted

## Definition of Done

- [ ] Library browsable with search/filter
- [ ] Custom nodes creatable/editable with versioning
- [ ] Fork, import, export all functional
- [ ] Canvas nodes saveable to library

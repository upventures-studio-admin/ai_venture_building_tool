# Agent 04 — Canvas Engine (Frontend)

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-04` |
| **Name** | Canvas Engine (Frontend) |
| **Sprint** | S2 — Auth & Canvas |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 5 days |
| **Dependencies** | Agent 02 |
| **Blocks** | Agent 07 |
| **Parallelizable with** | Agent 03 |

## Objective

Build the complete frontend canvas experience using React Flow: infinite canvas with pan/zoom, 4 custom node types (LLM, Input, Output, Custom), edge system with cycle detection, library side panel, Zustand state management, and keyboard shortcuts.

## Scope

- React Flow v12 infinite canvas
- 4 custom node components with full 7-element anatomy
- Edge rendering with directional arrows and mode colors
- Client-side DAG cycle detection
- Canvas Zustand store with undo/redo
- Library side panel (static data, API-connected in Agent 09)
- Keyboard shortcuts and confirm dialogs

## Tasks

### T-016: React Flow Canvas Setup

**Files**:
- `[NEW] src/app/(dashboard)/canvas/[id]/page.tsx`
- `[NEW] src/components/canvas/CanvasEditor.tsx`
- `[MODIFY] package.json` — add `reactflow`

**Acceptance Criteria**:
- [ ] React Flow canvas fills viewport with infinite scroll
- [ ] Pan/zoom at 60fps with mouse, trackpad, and pinch
- [ ] Background grid visible
- [ ] Controls panel (zoom in/out/fit)

---

### T-017: Canvas Zustand Store

**Files**:
- `[NEW] src/stores/canvas.store.ts`
- `[MODIFY] package.json` — add `zustand`

**Acceptance Criteria**:
- [ ] State: `nodes[]`, `edges[]`, `viewport`, `selectedNodeIds[]`
- [ ] Actions: `addNode`, `removeNode`, `updateNode`, `addEdge`, `removeEdge`, `setViewport`, `selectNode`, `clearSelection`
- [ ] Undo/redo stack (last 50 operations)
- [ ] `isDirty` flag tracks unsaved changes

---

### T-018: LLM Node Component

**Files**:
- `[NEW] src/components/canvas/nodes/LLMNode.tsx`
- `[NEW] src/components/canvas/nodes/NodeHeader.tsx`
- `[NEW] src/components/canvas/nodes/NodePorts.tsx`
- `[NEW] src/components/canvas/nodes/node-styles.css`

**Acceptance Criteria**:
- [ ] All 7 anatomy components visible (header, input, prompt, controls, output, LLM config, ports)
- [ ] State badges: idle=grey, running=animated pulse, done=green, error=red
- [ ] `◈` symbol in header for LLM type
- [ ] Draggable and resizable via corner handles

---

### T-019: Input, Output, Custom Node Components

**Files**:
- `[NEW] src/components/canvas/nodes/InputNode.tsx`
- `[NEW] src/components/canvas/nodes/OutputNode.tsx`
- `[NEW] src/components/canvas/nodes/CustomNode.tsx`

**Acceptance Criteria**:
- [ ] Input node: textarea + file upload area, `▶` symbol
- [ ] Output node: format dropdown (PDF/PPTX/MD/JSON), `◉` symbol
- [ ] Custom node: same layout as LLM with `⬡` symbol
- [ ] All share `NodeHeader` and `NodePorts` base components
- [ ] Registered in React Flow `nodeTypes` map

---

### T-020: Edge System + Cycle Detection

**Files**:
- `[NEW] src/components/canvas/edges/DataEdge.tsx`
- `[NEW] src/lib/dag-utils.ts`
- `[MODIFY] src/stores/canvas.store.ts` — validate in `addEdge`

**Acceptance Criteria**:
- [ ] Edges render as bezier curves with directional arrows
- [ ] Color by mode: auto=blue, manual=orange, visual=grey dashed
- [ ] Client-side cycle detection rejects circular connections with error toast
- [ ] `topologicalSort()` returns ordered node IDs for cascade execution

---

### T-021: Node Library Side Panel

**Files**:
- `[NEW] src/components/canvas/panels/LibraryPanel.tsx`
- `[NEW] src/components/canvas/panels/library-panel.css`

**Acceptance Criteria**:
- [ ] Slides in from left edge
- [ ] Lists 4 base types + seeded definitions (PD-01 through PITCH-01)
- [ ] Search input filters by name
- [ ] Category/phase tabs
- [ ] Drag-and-drop creates node at mouse position
- [ ] Double-click adds node at canvas center

---

### T-022: Keyboard Shortcuts + Delete Confirm

**Files**:
- `[NEW] src/hooks/useCanvasShortcuts.ts`
- `[NEW] src/components/ui/ConfirmDialog.tsx`

**Acceptance Criteria**:
- [ ] `Delete`/`Backspace` → confirm → remove selected nodes/edges
- [ ] `Cmd+Z`/`Ctrl+Z` → undo
- [ ] `Cmd+Shift+Z`/`Ctrl+Y` → redo
- [ ] `Cmd+D`/`Ctrl+D` → duplicate selected
- [ ] `Space+drag` → pan canvas
- [ ] Shortcuts suppressed when typing in input/textarea

## Final Verification

Open browser → navigate to `/canvas/test`:
- [ ] Canvas renders full-screen with grid background
- [ ] Drag node from library panel → node appears on canvas
- [ ] All 4 node types render correctly with distinct symbols
- [ ] Connect two nodes → edge with arrow appears
- [ ] Create cycle → error toast, edge rejected
- [ ] Delete key → confirm → node removed
- [ ] Cmd+Z → node restored
- [ ] Pan/zoom smooth at 60fps

## Definition of Done

- [ ] All 7 tasks (T-016 → T-022) completed
- [ ] 4 node types render correctly
- [ ] Edge system with cycle detection working
- [ ] Undo/redo functional
- [ ] No visual regressions on Chrome and Safari

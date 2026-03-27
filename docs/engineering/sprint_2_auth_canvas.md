# Sprint 2 — Auth & Workspace + Canvas Engine

> **Goal**: Users can register/login and interact with an infinite canvas with nodes and edges.

---

## Auth & Workspace (T-009 → T-015)

### T-009: Auth Service — Register + Login

**Files**:
- `[NEW] src/server/modules/auth/auth.service.ts`
- `[NEW] src/server/modules/auth/auth.controller.ts`
- `[NEW] src/server/modules/auth/auth.routes.ts`
- `[MODIFY] src/server/index.ts` — mount auth routes

**Steps**: Implement `POST /auth/register` (bcrypt hash, create user + default workspace) and `POST /auth/login` (verify, return JWT).

**Test**: `curl -X POST /api/auth/register -d '{"email":"test@test.com","password":"Test1234","name":"Test"}' ` returns user + token; login with same creds succeeds; wrong password returns 401.

**Acceptance Criteria**:
- [ ] Register creates user + default workspace + membership (role: owner)
- [ ] Login returns `{ accessToken, refreshToken }` with 15-min / 7-day expiry
- [ ] Passwords hashed with bcrypt (12 rounds)
- [ ] Duplicate email returns 409

---

### T-010: JWT Middleware + Refresh Token Rotation

**Files**:
- `[NEW] src/server/shared/middleware/auth.middleware.ts`
- `[NEW] src/server/modules/auth/token.service.ts`
- `[MODIFY] src/server/modules/auth/auth.routes.ts` — add `POST /auth/refresh`, `POST /auth/logout`, `GET /auth/me`

**Test**: Protected endpoint returns 401 without token; 200 with valid token; expired token returns 401; refresh returns new pair.

**Acceptance Criteria**:
- [ ] Middleware extracts and verifies JWT from `Authorization: Bearer`
- [ ] `req.user` populated with `{ userId, email }`
- [ ] Refresh rotation: old refresh token invalidated
- [ ] `/auth/me` returns user + workspace list

---

### T-011: Google OAuth

**Files**:
- `[MODIFY] src/server/modules/auth/auth.routes.ts` — add `POST /auth/oauth/google`
- `[MODIFY] src/server/modules/auth/auth.service.ts` — add `oauthLogin()`

**Test**: Send valid Google `idToken` → creates/finds user → returns JWT pair.

**Acceptance Criteria**:
- [ ] Verifies Google `idToken` via Google Auth Library
- [ ] Creates new user if email not found; links if found
- [ ] Returns same JWT pair as regular login
- [ ] Handles invalid/expired tokens with 401

---

### T-012: API Key Encryption Service

**Files**:
- `[NEW] src/server/shared/crypto/encryption.service.ts`

**Test**: `encrypt("sk-abc123")` → ciphertext; `decrypt(ciphertext)` → `"sk-abc123"`; different master key fails.

**Acceptance Criteria**:
- [ ] AES-256-GCM encryption with random IV per encryption
- [ ] Master key from `API_KEY_ENCRYPTION_KEY` env var
- [ ] `encrypt(plaintext)` → `iv:authTag:ciphertext` (base64)
- [ ] `decrypt(blob)` → plaintext
- [ ] Wrong key throws `DecryptionError`

---

### T-013: Workspace CRUD + API Key Storage

**Files**:
- `[NEW] src/server/modules/workspace/workspace.service.ts`
- `[NEW] src/server/modules/workspace/workspace.controller.ts`
- `[NEW] src/server/modules/workspace/workspace.routes.ts`
- `[MODIFY] src/server/index.ts` — mount workspace routes

**Test**: Create workspace → set API keys → retrieve workspace → keys NOT in response; internal `getDecryptedApiKeys()` returns plaintext.

**Acceptance Criteria**:
- [ ] `POST /workspaces` creates workspace, caller becomes owner
- [ ] `GET /workspaces/:id` returns workspace WITHOUT api_keys field
- [ ] `PUT /workspaces/:id/api-keys` encrypts and stores keys
- [ ] `PATCH /workspaces/:id` updates name, LLM config, cost limit
- [ ] Only workspace members can access their workspace (middleware check)

---

### T-014: Workspace Membership Management

**Files**:
- `[MODIFY] src/server/modules/workspace/workspace.routes.ts` — add member endpoints
- `[MODIFY] src/server/modules/workspace/workspace.service.ts` — add member logic

**Test**: Owner invites member by email → member appears in `GET /workspaces/:id`; change role; remove member → no longer has access.

**Acceptance Criteria**:
- [ ] `POST /workspaces/:id/members` invites (email + role)
- [ ] `PATCH /workspaces/:id/members/:userId` changes role
- [ ] `DELETE /workspaces/:id/members/:userId` removes member
- [ ] Only owner/admin can manage members
- [ ] Cannot remove the last owner

---

### T-015: Auth Frontend Pages

**Files**:
- `[NEW] src/app/(auth)/login/page.tsx`
- `[NEW] src/app/(auth)/register/page.tsx`
- `[NEW] src/app/(auth)/layout.tsx`
- `[NEW] src/lib/api-client.ts` — fetch wrapper with auth headers
- `[NEW] src/lib/auth-context.tsx` — React context for user/token

**Test**: Register form → submits → redirects to dashboard; Login form → submits → redirects; no token → redirected to login.

**Acceptance Criteria**:
- [ ] Login and register pages with form validation
- [ ] Google OAuth button (opens popup)
- [ ] Token stored in memory (not localStorage); refresh token in HttpOnly cookie
- [ ] Auth context provides `user`, `isAuthenticated`, `logout()`
- [ ] Protected routes redirect to `/login` if unauthenticated

---

## Canvas Engine Frontend (T-016 → T-022)

### T-016: React Flow Canvas Setup

**Files**:
- `[NEW] src/app/(dashboard)/canvas/[id]/page.tsx`
- `[NEW] src/components/canvas/CanvasEditor.tsx`
- `[MODIFY] package.json` — add `reactflow`

**Test**: Navigate to `/canvas/test` → infinite canvas renders; pan with mouse drag; zoom with scroll; pinch-to-zoom on trackpad.

**Acceptance Criteria**:
- [ ] React Flow canvas fills viewport
- [ ] Pan/zoom at 60fps
- [ ] Background grid visible
- [ ] Controls panel (zoom in/out/fit) visible

---

### T-017: Canvas Zustand Store

**Files**:
- `[NEW] src/stores/canvas.store.ts`
- `[MODIFY] package.json` — add `zustand`

**Test**: `useCanvasStore.getState().addNode(...)` adds node; `removeNode(id)` removes; `updateNodePosition(id, x, y)` updates; state changes trigger React re-render.

**Acceptance Criteria**:
- [ ] Store manages: `nodes[]`, `edges[]`, `viewport`, `selectedNodeIds[]`
- [ ] Actions: `addNode`, `removeNode`, `updateNode`, `addEdge`, `removeEdge`, `setViewport`, `selectNode`, `clearSelection`
- [ ] Undo/redo stack (last 50 operations)
- [ ] `isDirty` flag tracks unsaved changes

---

### T-018: Custom Node Component — LLM Node

**Files**:
- `[NEW] src/components/canvas/nodes/LLMNode.tsx`
- `[NEW] src/components/canvas/nodes/NodeHeader.tsx`
- `[NEW] src/components/canvas/nodes/NodePorts.tsx`
- `[NEW] src/components/canvas/nodes/node-styles.css`

**Test**: LLM node renders on canvas with header (name, type badge, state badge), input panel, prompt area, run/stop buttons, output panel, LLM selector. State badge colors match spec.

**Acceptance Criteria**:
- [ ] All 7 anatomy components rendered (header, input, prompt, controls, output, LLM config, ports)
- [ ] State badges: idle=grey, running=animated pulse, done=green, error=red
- [ ] Input/output ports visible for edge connections
- [ ] Node is draggable and resizable via handles
- [ ] `◈` symbol in header for LLM type

---

### T-019: Custom Node Components — Input, Output, Custom

**Files**:
- `[NEW] src/components/canvas/nodes/InputNode.tsx`
- `[NEW] src/components/canvas/nodes/OutputNode.tsx`
- `[NEW] src/components/canvas/nodes/CustomNode.tsx`

**Test**: Each node type renders with correct symbol (▶, ◉, ⬡); Input node has textarea for user text; Output node has format dropdown; all nodes share consistent styling.

**Acceptance Criteria**:
- [ ] Input node: textarea + file upload placeholder, `▶` symbol
- [ ] Output node: format selector (PDF/PPTX/MD/JSON), `◉` symbol
- [ ] Custom node: same as LLM node with `⬡` symbol
- [ ] All nodes share `NodeHeader` and `NodePorts` components
- [ ] Node type registered in React Flow `nodeTypes` map

---

### T-020: Edge System + Cycle Detection

**Files**:
- `[NEW] src/components/canvas/edges/DataEdge.tsx`
- `[NEW] src/lib/dag-utils.ts` — topological sort, cycle detection
- `[MODIFY] src/stores/canvas.store.ts` — addEdge with validation

**Test**: Connect two nodes → edge renders with arrow; try to create A→B→A cycle → rejected with error toast; multi-hop cycle also rejected.

**Acceptance Criteria**:
- [ ] Edges render as bezier curves with directional arrow
- [ ] Edge color reflects mode: auto=blue, manual=orange, visual=grey dashed
- [ ] `addEdge()` runs cycle detection before accepting
- [ ] `hasCycle(nodes, edges, newEdge)` returns boolean
- [ ] `topologicalSort(nodes, edges)` returns ordered node IDs

---

### T-021: Node Library Side Panel

**Files**:
- `[NEW] src/components/canvas/panels/LibraryPanel.tsx`
- `[NEW] src/components/canvas/panels/library-panel.css`

**Test**: Side panel lists available node types; search filters by name; drag node from panel → drops onto canvas at mouse position.

**Acceptance Criteria**:
- [ ] Panel slides in from left
- [ ] Lists 4 base types + seeded definitions (PD-01 through PITCH-01)
- [ ] Search input filters list
- [ ] Category/phase filter tabs
- [ ] Drag-and-drop from panel to canvas creates node
- [ ] Double-click on item adds node at canvas center

---

### T-022: Keyboard Shortcuts + Delete Confirm

**Files**:
- `[NEW] src/hooks/useCanvasShortcuts.ts`
- `[NEW] src/components/ui/ConfirmDialog.tsx`

**Test**: Select node → press Delete → confirm dialog → node removed; Cmd+Z undoes; Cmd+D duplicates; Space+drag pans.

**Acceptance Criteria**:
- [ ] `Delete` / `Backspace` → confirm dialog → remove selected node(s)/edge(s)
- [ ] `Cmd+Z` / `Ctrl+Z` → undo last action
- [ ] `Cmd+Shift+Z` / `Ctrl+Y` → redo
- [ ] `Cmd+D` / `Ctrl+D` → duplicate selected node(s)
- [ ] `Space + drag` → pan canvas
- [ ] Shortcuts don't fire when typing in text fields

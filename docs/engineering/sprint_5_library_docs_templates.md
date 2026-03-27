# Sprint 5 — Library + Documents + Templates

> **Goal**: Full node library, document export in all formats, and template system.

---

## Library Service (T-049 → T-054)

### T-049: Library API — CRUD + Search

**Files**:
- `[NEW] src/server/modules/library/library.service.ts`
- `[NEW] src/server/modules/library/library.controller.ts`
- `[NEW] src/server/modules/library/library.routes.ts`
- `[MODIFY] src/server/index.ts` — mount routes

**Test**: `GET /library/definitions?search=problem` returns NODE-PD-01; `POST` creates custom definition; `PATCH` increments version.

**Acceptance Criteria**:
- [ ] `GET /library/definitions` returns public + workspace-private definitions
- [ ] Filterable by: `workspaceId`, `category`, `type`, `visibility`, `search` (name/description)
- [ ] `POST` creates private definition in workspace
- [ ] `PATCH` increments `version` and stores new data
- [ ] Base definitions (`is_base: true`) cannot be deleted → 403

---

### T-050: Fork + Import/Export

**Files**:
- `[MODIFY] src/server/modules/library/library.service.ts`
- `[MODIFY] src/server/modules/library/library.routes.ts`

**Test**: Fork NODE-PD-01 → private copy with `forked_from_id` set; export → valid JSON; import JSON → new definition created.

**Acceptance Criteria**:
- [ ] `POST /library/definitions/:id/fork` creates private copy with `forked_from_id`
- [ ] `GET /library/definitions/:id/export` returns JSON with full schema
- [ ] `POST /library/definitions/import` validates JSON schema → creates definition
- [ ] Import rejects invalid JSON with descriptive errors
- [ ] Exported JSON includes: name, description, prompt, LLM config, I/O schemas, tags

---

### T-051: Library Panel — Browsing + Drag-to-Canvas

**Files**:
- `[MODIFY] src/components/canvas/panels/LibraryPanel.tsx` — connect to API
- `[NEW] src/components/canvas/panels/NodePreviewCard.tsx`

**Test**: Panel loads definitions from API; search filters in real-time; drag card to canvas creates node with definition's config.

**Acceptance Criteria**:
- [ ] Fetches definitions via React Query
- [ ] Category tabs: all, Phase 1–5, Custom
- [ ] Search input with debounce (300ms)
- [ ] Preview card: name, description, type badge, input/output summary
- [ ] Drag-to-canvas creates node with `definition_id` set, config pre-filled

---

### T-052: Custom Node Editor

**Files**:
- `[NEW] src/components/library/NodeEditor.tsx`
- `[NEW] src/components/library/NodeEditor.css`
- `[NEW] src/app/(dashboard)/library/page.tsx`

**Test**: Open editor → fill name, description, prompt, select LLM → save → appears in library; edit → version increments.

**Acceptance Criteria**:
- [ ] Form fields: name, description, category, type, prompt (with `{input}` `{context}` helpers), LLM config (provider, model, temperature, max_tokens), tags
- [ ] Prompt editor with syntax highlighting for `{variables}`
- [ ] "Save to Library" persists as private definition
- [ ] "Edit" on existing definition → form pre-filled → save increments version
- [ ] Validation: name required, prompt required

---

### T-053: Save Canvas Node to Library

**Files**:
- `[NEW] src/components/canvas/nodes/SaveToLibraryDialog.tsx`
- `[MODIFY] src/components/canvas/nodes/LLMNode.tsx` — add "Save to Library" menu item

**Test**: Right-click LLM node → "Save to Library" → dialog with pre-filled name/prompt → save → node appears in library panel.

**Acceptance Criteria**:
- [ ] Context menu on node header includes "Save to Library"
- [ ] Dialog pre-fills all fields from current node config
- [ ] Saves as private definition in current workspace
- [ ] Success toast with link to library page

---

### T-054: Node Versioning UI

**Files**:
- `[NEW] src/components/library/VersionHistory.tsx`

**Test**: Edit a definition 3 times → version history shows v1, v2, v3; click v1 → shows original config (read-only).

**Acceptance Criteria**:
- [ ] Version list on definition detail page
- [ ] Each version: number, timestamp, changed-by
- [ ] Click version → read-only view of that version's config
- [ ] Current version highlighted

---

## Document Generator (T-055 → T-059)

### T-055: Document API + Aggregation

**Files**:
- `[NEW] src/server/modules/documents/document.service.ts`
- `[NEW] src/server/modules/documents/document.controller.ts`
- `[NEW] src/server/modules/documents/document.routes.ts`
- `[MODIFY] src/server/index.ts` — mount routes

**Test**: Output node connected to 3 upstream nodes → `POST /nodes/:id/generate { format: "markdown" }` returns aggregated markdown.

**Acceptance Criteria**:
- [ ] Traverses all incoming edges recursively to collect upstream outputs
- [ ] Orders content by topological sort (dependency order)
- [ ] Concatenates with section headers (node name as H2)
- [ ] Returns `{ documentUrl, filename }` or inline content

---

### T-056: PDF Generator

**Files**:
- `[NEW] src/server/modules/documents/renderers/pdf.renderer.ts`
- `[NEW] src/server/modules/documents/templates/report.html` — HTML template
- `[MODIFY] package.json` — add `puppeteer`

**Test**: Generate PDF from aggregated content → valid PDF file with Upventure branding, sections, tables rendered correctly.

**Acceptance Criteria**:
- [ ] HTML template with branded header/footer
- [ ] Content injected into template, rendered via Puppeteer
- [ ] PDF uploaded to S3, signed URL returned
- [ ] Handles tables, bold, headings from markdown content
- [ ] Page numbers in footer

---

### T-057: PPTX Generator

**Files**:
- `[NEW] src/server/modules/documents/renderers/pptx.renderer.ts`
- `[MODIFY] package.json` — add `pptxgenjs`

**Test**: Pitch node output (10 slides) → generates PPTX with correct slide structure, headlines, and speaker notes.

**Acceptance Criteria**:
- [ ] Maps pitch deck structure: 10 slides with headline, content, visual suggestion, speaker notes
- [ ] Branded Upventure template (title slide, content slides)
- [ ] PPTX uploaded to S3
- [ ] Handles non-pitch content as bullet-point slides

---

### T-058: Markdown + JSON Renderers

**Files**:
- `[NEW] src/server/modules/documents/renderers/markdown.renderer.ts`
- `[NEW] src/server/modules/documents/renderers/json.renderer.ts`

**Test**: Markdown: returns `.md` string with H1/H2/H3; JSON: returns structured `{ nodes: [{ name, output }] }`.

**Acceptance Criteria**:
- [ ] Markdown: H1=canvas name, H2=node name, content in sections
- [ ] JSON: structured `{ canvasName, generatedAt, nodes: [{ id, name, type, output }] }`
- [ ] Both: copy-to-clipboard + download
- [ ] UTF-8 encoding for Italian characters

---

### T-059: Output Node Frontend

**Files**:
- `[MODIFY] src/components/canvas/nodes/OutputNode.tsx` — full implementation
- `[NEW] src/components/canvas/nodes/DocumentPreview.tsx`

**Test**: Connect upstream nodes → select PDF format → click Generate → preview renders in modal → download button works.

**Acceptance Criteria**:
- [ ] Format dropdown: PDF, PPTX, Markdown, JSON
- [ ] "Generate" button calls API
- [ ] Preview modal shows rendered document (HTML for PDF preview, slide thumbnails for PPTX)
- [ ] Download button fetches from S3 URL
- [ ] Loading state during generation
- [ ] Re-generate button (no need to re-run upstream LLM nodes)

---

## Templates (T-060 → T-062)

### T-060: Template Data Model + API

**Files**:
- `[NEW] src/server/modules/canvas/template.service.ts`
- `[NEW] prisma/templates/catena-principale.json`
- `[NEW] prisma/templates/lean-startup.json`
- `[NEW] prisma/templates/blank.json`

**Test**: `POST /workspaces/:wid/canvases { templateId: "catena-principale" }` creates canvas with 5 nodes + 4 edges pre-connected.

**Acceptance Criteria**:
- [ ] Templates stored as JSON files: `{ name, description, nodes[], edges[], metadata }`
- [ ] Template instantiation: creates canvas, creates nodes at specified positions, creates edges
- [ ] Node configs copied from template (linked to base definitions)
- [ ] At least 3 templates: Blank, Catena Principale, Lean Startup

---

### T-061: Template Selection Dialog

**Files**:
- `[NEW] src/components/dashboard/TemplateDialog.tsx`
- `[NEW] src/components/dashboard/TemplateCard.tsx`
- `[MODIFY] src/app/(dashboard)/page.tsx` — "New Canvas" button opens dialog

**Test**: Click "New Canvas" → dialog shows template cards with preview → select template → canvas created → navigate to canvas.

**Acceptance Criteria**:
- [ ] Grid of template cards with: name, description, node count, estimated time
- [ ] Visual preview: miniature node layout diagram
- [ ] Click → creates canvas via API → redirects to `/canvas/:id`
- [ ] Blank canvas always listed first

---

### T-062: Save Canvas as Template

**Files**:
- `[NEW] src/components/canvas/panels/SaveTemplateDialog.tsx`

**Test**: Canvas with nodes → "Save as Template" → name + description → saved → appears in template dialog for workspace.

**Acceptance Criteria**:
- [ ] Exports current canvas state as template JSON
- [ ] Stores in workspace scope (private to workspace)
- [ ] Template appears in selection dialog with "Custom" badge
- [ ] Node outputs NOT included in template (only structure + config)

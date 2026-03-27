# Agent 10 — Document Generator

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-10` |
| **Name** | Document Generator |
| **Sprint** | S5 — Library, Docs, Templates |
| **Milestone** | M2 — Beta |
| **Estimated Effort** | 5 days |
| **Dependencies** | Agent 08 |
| **Blocks** | Agent 12 |

## Objective

Build the document generation pipeline: aggregate upstream node outputs, render into 4+ formats (PDF, PPTX, Markdown, JSON), store in S3, and provide preview/download in the frontend Output node.

## Tasks

### T-055: Document API + Aggregation

**Files**:
- `[NEW] src/server/modules/documents/document.service.ts`
- `[NEW] src/server/modules/documents/document.controller.ts`
- `[NEW] src/server/modules/documents/document.routes.ts`

**Acceptance Criteria**:
- [ ] Traverses all incoming edges recursively to collect upstream outputs
- [ ] Orders by topological sort (dependency order)
- [ ] Concatenates with section headers (node name as H2)
- [ ] Returns `{ documentUrl, filename }` or inline content

---

### T-056: PDF Generator

**Files**:
- `[NEW] src/server/modules/documents/renderers/pdf.renderer.ts`
- `[NEW] src/server/modules/documents/templates/report.html`

**Acceptance Criteria**:
- [ ] HTML template with branded Upventure header/footer
- [ ] Puppeteer renders HTML → PDF
- [ ] Uploaded to S3 with signed download URL
- [ ] Handles tables, bold, headings from markdown
- [ ] Page numbers in footer

---

### T-057: PPTX Generator

**Files**:
- `[NEW] src/server/modules/documents/renderers/pptx.renderer.ts`

**Acceptance Criteria**:
- [ ] Maps pitch deck structure to 10 slides (headline, content, speaker notes)
- [ ] Branded template (title slide, content slides)
- [ ] Uploaded to S3
- [ ] Non-pitch content rendered as bullet-point slides

---

### T-058: Markdown + JSON Renderers

**Files**:
- `[NEW] src/server/modules/documents/renderers/markdown.renderer.ts`
- `[NEW] src/server/modules/documents/renderers/json.renderer.ts`

**Acceptance Criteria**:
- [ ] Markdown: H1=canvas, H2=node, content sections
- [ ] JSON: `{ canvasName, generatedAt, nodes: [{ id, name, type, output }] }`
- [ ] Both: copy-to-clipboard + file download
- [ ] UTF-8 encoding for Italian characters

---

### T-059: Output Node Frontend

**Files**:
- `[MODIFY] src/components/canvas/nodes/OutputNode.tsx`
- `[NEW] src/components/canvas/nodes/DocumentPreview.tsx`

**Acceptance Criteria**:
- [ ] Format dropdown: PDF, PPTX, Markdown, JSON
- [ ] "Generate" button calls API with loading state
- [ ] Preview modal (HTML for PDF, slide thumbnails for PPTX)
- [ ] Download button fetches from S3
- [ ] Re-generate without re-running upstream LLM nodes

## Definition of Done

- [ ] All 4 formats generate correctly
- [ ] Files uploaded to S3 with signed URLs
- [ ] Preview renders in browser
- [ ] Download works for all formats

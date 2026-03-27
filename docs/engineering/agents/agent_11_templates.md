# Agent 11 — Templates

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-11` |
| **Name** | Templates |
| **Sprint** | S5 — Library, Docs, Templates |
| **Milestone** | M2 — Beta |
| **Estimated Effort** | 3 days |
| **Dependencies** | Agent 09 |
| **Blocks** | Agent 12 |

## Objective

Build the template system: predefined JSON templates for common venture building flows, a selection dialog for canvas creation, and the ability to save any canvas as a reusable workspace template.

## Tasks

### T-060: Template Data Model + API

**Files**:
- `[NEW] src/server/modules/canvas/template.service.ts`
- `[NEW] prisma/templates/catena-principale.json`
- `[NEW] prisma/templates/lean-startup.json`
- `[NEW] prisma/templates/blank.json`

**Acceptance Criteria**:
- [ ] Templates stored as JSON: `{ name, description, nodes[], edges[], metadata }`
- [ ] `POST /workspaces/:wid/canvases { templateId }` instantiates template
- [ ] Creates canvas with nodes at specified positions + edges connected
- [ ] Node configs linked to base definitions
- [ ] Minimum 3 templates: Blank, Catena Principale, Lean Startup

**Template: Catena Principale**:
```
Input → NODE-PD-01 → NODE-MR-01 → NODE-BS-01 → NODE-GTM-01 → NODE-PITCH-01 → Output
```

**Template: Lean Startup**:
```
Input → NODE-PD-01 → NODE-BS-01 (steps 2-4 only) → Output
```

---

### T-061: Template Selection Dialog

**Files**:
- `[NEW] src/components/dashboard/TemplateDialog.tsx`
- `[NEW] src/components/dashboard/TemplateCard.tsx`
- `[MODIFY] src/app/(dashboard)/page.tsx`

**Acceptance Criteria**:
- [ ] Grid of template cards: name, description, node count, estimated time
- [ ] Visual preview: miniature node layout diagram
- [ ] Click → creates canvas via API → redirects to `/canvas/:id`
- [ ] Blank canvas always first

---

### T-062: Save Canvas as Template

**Files**:
- `[NEW] src/components/canvas/panels/SaveTemplateDialog.tsx`

**Acceptance Criteria**:
- [ ] Exports current canvas structure as template JSON
- [ ] Stored in workspace scope (private)
- [ ] Appears in selection dialog with "Custom" badge
- [ ] Node outputs NOT included (only structure + config)

## Definition of Done

- [ ] 3+ predefined templates instantiate correctly
- [ ] Template dialog shows visual previews
- [ ] Custom templates saveable from any canvas
- [ ] All template nodes connect properly on instantiation

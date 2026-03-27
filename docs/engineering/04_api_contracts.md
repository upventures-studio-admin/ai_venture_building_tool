# 04 — API Contracts

> Base URL: `https://api.venturecanvas.app/v1`  
> Auth: `Authorization: Bearer <JWT>` on all endpoints except `/auth/*`  
> All responses: `{ data, error?, meta? }`

---

## Authentication

| Method | Endpoint | Body | Response | Notes |
|---|---|---|---|---|
| POST | `/auth/register` | `{ email, password, name }` | `{ user, accessToken, refreshToken }` | Creates default workspace |
| POST | `/auth/login` | `{ email, password }` | `{ user, accessToken, refreshToken }` | |
| POST | `/auth/oauth/google` | `{ idToken }` | `{ user, accessToken, refreshToken }` | |
| POST | `/auth/refresh` | `{ refreshToken }` | `{ accessToken, refreshToken }` | |
| POST | `/auth/logout` | — | `204` | Invalidates refresh token |
| GET | `/auth/me` | — | `{ user, workspaces[] }` | |

## Workspaces

| Method | Endpoint | Body / Query | Response |
|---|---|---|---|
| POST | `/workspaces` | `{ name, slug }` | `{ workspace }` |
| GET | `/workspaces/:id` | — | `{ workspace, members[] }` |
| PATCH | `/workspaces/:id` | `{ name?, llmDefaultConfig?, monthlyCostLimitUsd? }` | `{ workspace }` |
| PUT | `/workspaces/:id/api-keys` | `{ anthropic?, openai?, google? }` | `204` |
| POST | `/workspaces/:id/members` | `{ email, role }` | `{ member }` |
| DELETE | `/workspaces/:id/members/:userId` | — | `204` |
| PATCH | `/workspaces/:id/members/:userId` | `{ role }` | `{ member }` |

## Canvases

| Method | Endpoint | Body / Query | Response |
|---|---|---|---|
| GET | `/workspaces/:wid/canvases` | `?page&limit` | `{ canvases[], meta }` |
| POST | `/workspaces/:wid/canvases` | `{ name, templateId? }` | `{ canvas }` (with nodes/edges if template) |
| GET | `/canvases/:id` | — | `{ canvas, nodes[], edges[] }` |
| PATCH | `/canvases/:id` | `{ name?, viewport?, llmConfigOverride? }` | `{ canvas }` |
| DELETE | `/canvases/:id` | — | `204` |
| POST | `/canvases/:id/auto-save` | `{ delta }` | `{ savedAt }` |

## Nodes

| Method | Endpoint | Body | Response |
|---|---|---|---|
| POST | `/canvases/:cid/nodes` | `{ type, label, definitionId?, positionX, positionY, config? }` | `{ node }` |
| PATCH | `/nodes/:id` | `{ label?, positionX?, positionY?, width?, height?, config?, inputContent?, notes?, isCollapsed? }` | `{ node }` |
| DELETE | `/nodes/:id` | — | `204` (cascade removes edges) |
| POST | `/nodes/:id/duplicate` | — | `{ node }` (new node, same config) |
| GET | `/nodes/:id/export` | — | `{ json }` (complete node definition) |

## Edges

| Method | Endpoint | Body | Response |
|---|---|---|---|
| POST | `/canvases/:cid/edges` | `{ sourceNodeId, targetNodeId, mode }` | `{ edge }` / `400` if cycle detected |
| PATCH | `/edges/:id` | `{ mode }` | `{ edge }` |
| DELETE | `/edges/:id` | — | `204` |

## Execution

| Method | Endpoint | Body | Response |
|---|---|---|---|
| POST | `/nodes/:id/execute` | — | `{ executionId }` (streaming via WebSocket) |
| POST | `/canvases/:id/execute` | — | `{ executionIds[] }` (cascade) |
| POST | `/nodes/:id/stop` | — | `204` |
| POST | `/nodes/:id/rerun` | `{ confirmOverwrite: true }` | `{ executionId }` |
| GET | `/nodes/:id/executions` | `?page&limit` | `{ executions[] }` |

## Wizard Operations

| Method | Endpoint | Body | Response |
|---|---|---|---|
| GET | `/nodes/:id/wizard` | — | `{ currentStep, totalSteps, stepOutputs[], gateStatus }` |
| POST | `/nodes/:id/wizard/gate` | `{ answers?: {} }` | `{ gateStatus, questions? }` |
| POST | `/nodes/:id/wizard/advance` | `{ stepConfirmation: true }` | `{ nextStep, executionId }` |
| POST | `/nodes/:id/wizard/reset` | — | `204` |

## Library

| Method | Endpoint | Body / Query | Response |
|---|---|---|---|
| GET | `/library/definitions` | `?workspaceId&category&type&search&visibility` | `{ definitions[] }` |
| GET | `/library/definitions/:id` | — | `{ definition }` |
| POST | `/library/definitions` | `{ workspaceId, name, type, defaultPrompt, ... }` | `{ definition }` |
| PATCH | `/library/definitions/:id` | partial update | `{ definition }` (increments version) |
| POST | `/library/definitions/:id/fork` | `{ workspaceId }` | `{ definition }` (private copy) |
| GET | `/library/definitions/:id/export` | — | JSON file download |
| POST | `/library/definitions/import` | `{ workspaceId, json }` | `{ definition }` |

## Versioning

| Method | Endpoint | Body / Query | Response |
|---|---|---|---|
| GET | `/canvases/:id/versions` | `?page&limit&eventType&authorId` | `{ versions[] }` |
| POST | `/canvases/:id/versions` | `{ name? }` | `{ version }` (manual snapshot) |
| GET | `/versions/:id` | — | `{ version, snapshotData }` |
| POST | `/canvases/:id/rollback` | `{ versionId }` | `{ canvas }` (restored) |
| GET | `/canvases/:id/diff` | `?versionA&versionB` | `{ added[], removed[], modified[] }` |

## Documents

| Method | Endpoint | Body | Response |
|---|---|---|---|
| POST | `/nodes/:id/generate` | `{ format: 'pdf'|'pptx'|'markdown'|'json'|'gdocs'|'gslides' }` | `{ documentUrl, filename }` |
| GET | `/nodes/:id/preview` | — | HTML preview |

---

## WebSocket Events (Socket.IO)

**Namespace**: `/canvas`  
**Room**: `canvas:{canvasId}`

### Client → Server
| Event | Payload | Description |
|---|---|---|
| `canvas:join` | `{ canvasId }` | Join room for real-time updates |
| `canvas:leave` | `{ canvasId }` | Leave room |

### Server → Client
| Event | Payload | Description |
|---|---|---|
| `node:stateChange` | `{ nodeId, state, errorMessage? }` | Node execution state changed |
| `node:streamChunk` | `{ nodeId, executionId, chunk, done }` | LLM streaming token |
| `canvas:saved` | `{ canvasId, savedAt }` | Auto-save confirmed |
| `canvas:versionCreated` | `{ canvasId, versionId, eventType }` | New snapshot |
| `user:presence` | `{ canvasId, users[] }` | Active users in canvas |
| `node:outputUpdated` | `{ nodeId, outputContent }` | Final output stored |

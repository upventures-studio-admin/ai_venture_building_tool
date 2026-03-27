# Agent 03 — Auth & Workspace

## Identity

| Field | Value |
|---|---|
| **Agent ID** | `AGENT-03` |
| **Name** | Auth & Workspace |
| **Sprint** | S2 — Auth & Canvas |
| **Milestone** | M1 — Alpha |
| **Estimated Effort** | 4 days |
| **Dependencies** | Agent 02 |
| **Blocks** | Agent 06 |
| **Parallelizable with** | Agent 04 |

## Objective

Implement the full authentication system (email/password + Google OAuth), JWT management with refresh token rotation, workspace CRUD, membership management, API key encryption vault, and frontend auth pages.

## Scope

- Register/login/logout endpoints
- Google OAuth integration
- JWT + refresh token rotation
- Workspace CRUD with membership roles (owner/admin/member)
- AES-256-GCM API key encryption service
- Frontend login/register pages + auth context

## Tasks

### T-009: Auth Service — Register + Login

**Files**:
- `[NEW] src/server/modules/auth/auth.service.ts`
- `[NEW] src/server/modules/auth/auth.controller.ts`
- `[NEW] src/server/modules/auth/auth.routes.ts`
- `[MODIFY] src/server/index.ts` — mount auth routes

**Acceptance Criteria**:
- [ ] `POST /api/auth/register` creates user + default workspace + membership (role: owner)
- [ ] `POST /api/auth/login` returns `{ accessToken, refreshToken }` (15-min / 7-day expiry)
- [ ] Passwords hashed with bcrypt (12 rounds)
- [ ] Duplicate email returns 409

---

### T-010: JWT Middleware + Refresh Token Rotation

**Files**:
- `[NEW] src/server/shared/middleware/auth.middleware.ts`
- `[NEW] src/server/modules/auth/token.service.ts`
- `[MODIFY] src/server/modules/auth/auth.routes.ts`

**Acceptance Criteria**:
- [ ] Middleware verifies JWT from `Authorization: Bearer` header
- [ ] `req.user` populated with `{ userId, email }`
- [ ] `POST /api/auth/refresh` rotates tokens (old refresh invalidated)
- [ ] `GET /api/auth/me` returns user + workspace list
- [ ] `POST /api/auth/logout` invalidates refresh token

---

### T-011: Google OAuth

**Files**:
- `[MODIFY] src/server/modules/auth/auth.routes.ts`
- `[MODIFY] src/server/modules/auth/auth.service.ts`

**Acceptance Criteria**:
- [ ] `POST /api/auth/oauth/google` verifies Google `idToken`
- [ ] Creates new user if email not found; links to existing if found
- [ ] Returns same JWT pair as regular login
- [ ] Invalid/expired Google token → 401

---

### T-012: API Key Encryption Service

**Files**:
- `[NEW] src/server/shared/crypto/encryption.service.ts`

**Acceptance Criteria**:
- [ ] AES-256-GCM with random IV per encryption
- [ ] Master key from `API_KEY_ENCRYPTION_KEY` env var
- [ ] `encrypt(plaintext)` → `iv:authTag:ciphertext` (base64)
- [ ] `decrypt(blob)` → plaintext
- [ ] Wrong master key throws `DecryptionError`

---

### T-013: Workspace CRUD + API Key Storage

**Files**:
- `[NEW] src/server/modules/workspace/workspace.service.ts`
- `[NEW] src/server/modules/workspace/workspace.controller.ts`
- `[NEW] src/server/modules/workspace/workspace.routes.ts`

**Acceptance Criteria**:
- [ ] `POST /api/workspaces` creates workspace, caller becomes owner
- [ ] `GET /api/workspaces/:id` returns workspace **without** `api_keys` field
- [ ] `PUT /api/workspaces/:id/api-keys` encrypts and stores keys
- [ ] `PATCH /api/workspaces/:id` updates name, LLM config, cost limit
- [ ] Only workspace members can access (middleware check)

---

### T-014: Workspace Membership Management

**Files**:
- `[MODIFY] src/server/modules/workspace/workspace.routes.ts`
- `[MODIFY] src/server/modules/workspace/workspace.service.ts`

**Acceptance Criteria**:
- [ ] `POST /api/workspaces/:id/members` invites by email + role
- [ ] `PATCH /api/workspaces/:id/members/:userId` changes role
- [ ] `DELETE /api/workspaces/:id/members/:userId` removes member
- [ ] Only owner/admin can manage members
- [ ] Cannot remove the last owner

---

### T-015: Auth Frontend Pages

**Files**:
- `[NEW] src/app/(auth)/login/page.tsx`
- `[NEW] src/app/(auth)/register/page.tsx`
- `[NEW] src/app/(auth)/layout.tsx`
- `[NEW] src/lib/api-client.ts`
- `[NEW] src/lib/auth-context.tsx`

**Acceptance Criteria**:
- [ ] Login and register forms with client-side validation
- [ ] Google OAuth button
- [ ] Token in memory (not localStorage); refresh in HttpOnly cookie
- [ ] Auth context: `user`, `isAuthenticated`, `logout()`
- [ ] Protected routes redirect to `/login` if unauthenticated

## Final Verification

```bash
# Register + login flow
curl -X POST localhost:3001/api/auth/register -H 'Content-Type: application/json' \
  -d '{"email":"test@test.com","password":"Test1234!","name":"Test"}'
# → 201 with user + tokens

# Protected endpoint
curl localhost:3001/api/auth/me -H 'Authorization: Bearer <token>'
# → 200 with user + workspaces

# API key encryption round-trip (internal)
# encrypt("sk-abc") → decrypt → "sk-abc" ✓
```

## Definition of Done

- [ ] All auth endpoints working (register, login, refresh, logout, OAuth)
- [ ] JWT middleware protects all non-auth routes
- [ ] Workspace CRUD + membership management functional
- [ ] API keys encrypted at rest, never exposed in responses
- [ ] Frontend login/register pages functional

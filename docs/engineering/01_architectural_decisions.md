# 01 — Architectural Decisions

## Tech Stack

| Layer | Choice | Rationale |
|---|---|---|
| **Frontend** | Next.js 14 (App Router) + TypeScript | SSR for SEO/onboarding pages, RSC for dashboard, strong typing |
| **Canvas Engine** | React Flow v12 | Mature, performant infinite canvas with built-in pan/zoom, node/edge management, 60fps on 100+ nodes |
| **State Management** | Zustand + React Query | Zustand for canvas state (local, fast); React Query for server state (cache, sync) |
| **Styling** | CSS Modules + Design Tokens | Maximum control, no runtime overhead, theme-ready for future dark mode |
| **Backend** | Node.js + Express (TypeScript) | Same language as frontend; strong async/streaming support for LLM proxying |
| **Database** | PostgreSQL 16 | JSONB for flexible node configs; strong indexing; row-level security for workspace isolation |
| **Cache / Queue** | Redis 7 + BullMQ | Auto-save debounce; execution queue with retry/backoff; pub/sub for real-time events |
| **Auth** | NextAuth.js (Auth.js) | Built-in OAuth (Google), JWT, refresh tokens; easy to extend with email/password |
| **File Storage** | S3-compatible (AWS S3 or MinIO) | Exported documents, uploaded files, template assets |
| **Real-time** | WebSocket (Socket.IO) | LLM streaming, auto-save status, presence indicators |
| **Document Gen** | pptxgenjs + Puppeteer + Google APIs | PPTX native generation; PDF via headless Chrome; Google Docs/Slides via API |
| **ORM** | Prisma | Type-safe queries, migrations, introspection; good PostgreSQL JSONB support |

---

## Architecture Pattern: Modular Monolith

> [!IMPORTANT]
> We choose a **modular monolith** over microservices for M1–M3. The codebase is organized into domain modules with clear boundaries, but deployed as a single service. This accelerates development and simplifies operations for a small team.

```
┌─────────────────────────────────────────────────┐
│                  Next.js Frontend                │
│  (Canvas UI · Dashboard · Settings · Auth Pages) │
└──────────────────────┬──────────────────────────┘
                       │ REST + WebSocket
┌──────────────────────▼──────────────────────────┐
│               Express API Server                 │
│  ┌──────────┬──────────┬──────────┬───────────┐  │
│  │  Canvas  │  Node    │ Library  │  Auth &   │  │
│  │  Module  │ Executor │ Service  │ Workspace │  │
│  ├──────────┼──────────┼──────────┼───────────┤  │
│  │ Version  │   LLM    │  Doc     │  Export   │  │
│  │ Engine   │ Gateway  │ Generator│  Service  │  │
│  └──────────┴──────────┴──────────┴───────────┘  │
└───────┬──────────┬──────────┬────────────────────┘
        │          │          │
   PostgreSQL    Redis     S3/MinIO
```

### Key Patterns

| Pattern | Where | Why |
|---|---|---|
| **Gateway Pattern** | LLM Gateway | Single interface abstracting Anthropic/OpenAI/Google; swap providers without touching node logic |
| **Event Sourcing (light)** | Versioning Engine | Canvas changes stored as events (deltas); snapshots reconstructed; supports rollback and diff |
| **Command Queue** | Node Executor | BullMQ queue for LLM executions; supports retry with exponential backoff, timeout, cascading |
| **DAG Validation** | Link System | Topological sort to detect cycles and determine cascade execution order |
| **Optimistic Updates** | Canvas UI | Immediate UI feedback; server reconciliation via auto-save; conflict resolution on error |
| **Hierarchical Config** | LLM Config | Node > Canvas > Workspace override chain for LLM settings |

---

## ADR Log

### ADR-001: React Flow over Rete.js
- **Decision**: Use React Flow v12
- **Context**: Both are capable canvas libraries. Rete.js is more low-level.
- **Rationale**: React Flow has better React integration, larger community, built-in minimap/controls, and proven performance at 100+ nodes. Rete.js offers more customization but requires significantly more boilerplate.

### ADR-002: PostgreSQL over MongoDB
- **Decision**: PostgreSQL with JSONB columns for flexible fields
- **Context**: Need relational integrity (workspace→canvas→node) AND flexible schema (node configs vary by type)
- **Rationale**: PostgreSQL gives us foreign keys, transactions, RLS for workspace isolation, AND JSONB for schema-flexible node configs. Best of both worlds vs. pure document store.

### ADR-003: Modular Monolith over Microservices
- **Decision**: Single deployable with domain modules
- **Context**: Small team (2–4 devs), shared data model, tight coupling between canvas and execution
- **Rationale**: Microservices add operational overhead (service mesh, distributed tracing, eventual consistency). Module boundaries enforce separation; extract services later if needed (LLM Gateway is the first candidate).

### ADR-004: Event Sourcing for Versioning
- **Decision**: Store canvas changes as immutable events, not full snapshots
- **Context**: PRD requires automatic snapshots every 30s, manual snapshots, rollback, and diff
- **Rationale**: Delta-based events are storage-efficient. Full snapshots are periodically materialized for fast reads. Events enable diff computation between any two points in time.

### ADR-005: BullMQ for Node Execution Queue
- **Decision**: Use BullMQ (Redis-backed) for async node execution
- **Context**: Nodes call external LLM APIs with 120s timeout, need retry, cascading, and progress tracking
- **Rationale**: BullMQ provides reliable job queues with retry strategies, concurrency control, progress events, and dead-letter queues — all needed for robust LLM execution.

### ADR-006: Wizard Nodes as Multi-Step State Machines
- **Decision**: Implement wizard nodes (BS-01, GTM-01, PITCH-01) as finite state machines
- **Context**: Wizard nodes require mandatory stop after each step, user confirmation, and context propagation
- **Rationale**: Each wizard step is a discrete state with: required inputs, LLM execution, output, and explicit user gate. State machine pattern ensures no step is skipped and context accumulates correctly.

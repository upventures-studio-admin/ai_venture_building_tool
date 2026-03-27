# 05 — Deployment Workflow

## Infrastructure Overview

```mermaid
graph TB
    subgraph "CDN / Edge"
        CF["Cloudflare CDN"]
    end
    subgraph "Compute"
        APP["App Server<br/>Next.js + Express<br/>(Railway or AWS ECS)"]
        WORKER["Queue Worker<br/>BullMQ processor<br/>(separate process)"]
    end
    subgraph "Data"
        PG["PostgreSQL 16<br/>(Railway Postgres<br/>or AWS RDS)"]
        REDIS["Redis 7<br/>(Railway Redis<br/>or ElastiCache)"]
        S3["S3-compatible<br/>Object Storage"]
    end
    CF --> APP
    APP --> PG
    APP --> REDIS
    APP --> S3
    WORKER --> PG
    WORKER --> REDIS
    WORKER --> S3
```

---

## Environments

| Environment | Purpose | URL | Database | Deployment |
|---|---|---|---|---|
| **Development** | Local dev | `localhost:3000` | Local Docker PostgreSQL + Redis | `docker compose up` |
| **Staging** | Pre-release testing | `staging.venturecanvas.app` | Isolated staging DB | Auto-deploy on `main` merge |
| **Production** | Live users | `app.venturecanvas.app` | Production DB (daily backups) | Manual promote from staging |

---

## Docker Setup

### `docker-compose.yml` (Development)
```yaml
services:
  app:
    build: .
    ports: ["3000:3000"]
    env_file: .env
    depends_on: [postgres, redis]
  worker:
    build: .
    command: npm run worker
    env_file: .env
    depends_on: [postgres, redis]
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: venture_canvas
      POSTGRES_USER: app
      POSTGRES_PASSWORD: dev_password
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
volumes:
  pgdata:
```

### `Dockerfile`
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:20-alpine AS runner
RUN apk add --no-cache chromium  # For Puppeteer PDF generation
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
COPY --from=builder /app/prisma ./prisma
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## CI/CD Pipeline (GitHub Actions)

```mermaid
graph LR
    A[Push to branch] --> B[Lint + Type Check]
    B --> C[Unit Tests]
    C --> D[Integration Tests<br/>Docker Compose]
    D --> E{Branch?}
    E -->|main| F[Build Docker Image]
    E -->|PR| G[Report Status]
    F --> H[Push to Registry]
    H --> I[Deploy to Staging]
    I --> J[E2E Tests on Staging]
    J --> K[Manual Approval]
    K --> L[Deploy to Production]
```

### Pipeline Stages

| Stage | Trigger | Actions | Duration |
|---|---|---|---|
| **Lint & Type** | All pushes | ESLint, Prettier, `tsc --noEmit` | ~1 min |
| **Unit Tests** | All pushes | Jest with coverage (≥80% threshold) | ~2 min |
| **Integration** | All pushes | Docker Compose up → Supertest API tests → down | ~5 min |
| **Build** | `main` only | Docker build, tag with SHA + `latest` | ~3 min |
| **Push** | `main` only | Push to GitHub Container Registry | ~1 min |
| **Stage Deploy** | `main` only | Deploy to staging environment | ~2 min |
| **E2E** | Post-staging | Playwright against staging URL | ~5 min |
| **Prod Deploy** | Manual trigger | Deploy to production with zero-downtime | ~3 min |

---

## Database Migrations

| Tool | Prisma Migrate |
|---|---|
| **Dev** | `npx prisma migrate dev` (auto-apply + generate) |
| **Staging/Prod** | `npx prisma migrate deploy` (apply pending migrations only) |
| **Seed** | `npx prisma db seed` — seeds base library nodes (NODE-PD-01 through PITCH-01) |
| **Pre-deploy hook** | Migration runs before new app version starts; rollback if migration fails |

---

## Monitoring & Observability

| Concern | Tool | Details |
|---|---|---|
| **APM** | Sentry | Error tracking, performance monitoring, release tracking |
| **Logging** | Pino → stdout → log aggregator | Structured JSON logs; levels: error, warn, info, debug |
| **Metrics** | Prometheus + Grafana (or Railway metrics) | Request latency, LLM execution time, queue depth, error rates |
| **Uptime** | Better Uptime or UptimeRobot | SLA ≥ 99.5% monitoring; alerts to Slack |
| **Alerts** | PagerDuty / Slack | LLM error rate > 5%, queue depth > 50, response time P95 > 5s |

---

## Backup Strategy

| Data | Method | Frequency | Retention |
|---|---|---|---|
| **PostgreSQL** | `pg_dump` automated | Daily at 03:00 UTC | 30 days |
| **PostgreSQL WAL** | Point-in-time recovery | Continuous | 7 days |
| **S3 documents** | S3 versioning | On write | 90 days |
| **Redis** | Ephemeral (queue + cache) | No backup needed | — |

---

## Security Hardening

| Layer | Measure |
|---|---|
| **Network** | HTTPS only (TLS 1.3); HSTS; rate limiting (100 req/min per user) |
| **Auth** | JWT with short expiry; HttpOnly cookies for refresh tokens; CSRF protection |
| **API Keys** | AES-256-GCM encryption at rest; never returned to frontend; environment-variable master key |
| **Headers** | Helmet.js: CSP, X-Frame-Options, X-Content-Type-Options |
| **Dependencies** | `npm audit` in CI; Dependabot for automated updates |
| **Secrets** | All secrets in environment variables or secret manager; never in code |

---

## Environment Variables

```env
# Core
DATABASE_URL=postgresql://user:pass@host:5432/venture_canvas
REDIS_URL=redis://host:6379
NEXTAUTH_SECRET=<random-32-bytes>
NEXTAUTH_URL=https://app.venturecanvas.app

# Encryption
API_KEY_ENCRYPTION_KEY=<random-32-bytes>

# Storage
S3_BUCKET=venture-canvas-docs
S3_REGION=eu-central-1
S3_ACCESS_KEY=...
S3_SECRET_KEY=...

# Google (for Docs/Slides export)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...

# Monitoring
SENTRY_DSN=...

# Feature flags
ENABLE_GOOGLE_EXPORT=false
ENABLE_COST_LIMITS=false
```

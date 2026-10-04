# Architecture

## High-Level Diagram

```
[Request]
   ↓
[Semantic cache] ── HIT → return (0 tokens)
   ↓ MISS
[Speculative draft: 2× GLM in parallel via Reactor]
   ↓
[Freestyle sandbox runs diagnostic scripts]
   ↓ (returns structured JSON, not raw logs)
[Exa external evidence — web search for related incidents/CVEs]
   ↓
[Fiduciary critic verifies draft vs. evidence]
   ↓ approved? → store + return
   ↓ rejected? → escalate to verify-tier (Sonnet) → refine
[Cache store + token meter log + Slack post via Executor MCP]
```

## Component Map

```
src/
├── app/
│   ├── api/
│   │   ├── agent/
│   │   │   ├── incident/route.ts          → POST /api/agent/incident
│   │   │   ├── incident/stream/route.ts   → POST /api/agent/incident/stream (SSE)
│   │   │   ├── audit/route.ts             → POST /api/agent/audit
│   │   │   ├── deal/route.ts              → POST /api/agent/deal
│   │   │   └── compare/route.ts           → POST /api/agent/compare?mode=X
│   │   ├── agentmail/
│   │   │   ├── inbox/route.ts             → GET /api/agentmail/inbox
│   │   │   ├── process/route.ts           → POST /api/agentmail/process
│   │   │   └── webhook/route.ts           → POST /api/agentmail/webhook
│   │   ├── coshell/
│   │   │   ├── jobs/route.ts              → GET /api/coshell/jobs
│   │   │   ├── trigger/route.ts           → POST /api/coshell/trigger
│   │   │   └── reflection/route.ts        → POST /api/coshell/reflection
│   │   ├── reactor/status/route.ts        → GET /api/reactor/status
│   │   ├── chaos/trigger/route.ts         → POST /api/chaos/trigger
│   │   ├── metrics/route.ts               → GET /api/metrics
│   │   ├── health/route.ts                → GET /api/health
│   │   └── download/
│   │       ├── chunk/[chunk]/route.ts     → GET /api/download/chunk/{aa,ab,...}
│   │       └── all/route.ts               → GET /api/download/all
│   ├── page.tsx                           → 6-tab dashboard (client component)
│   └── layout.tsx
├── components/
│   ├── chat-interface.tsx                 → Assistant UI chat
│   └── ui/                                → shadcn/ui (40+ components)
└── lib/
    ├── orchestrator/
    │   ├── index.ts                       → Main agent loop (cache → draft → sandbox → critic → escalate)
    │   └── naive-baseline.ts              → Naive comparison (no optimizations)
    ├── llm/
    │   ├── provider-bridge.ts             → GLM routing (Neon Gateway → Zhipu → z-ai fallback)
    │   └── embeddings.ts                  → text-embedding-3-small (1536 dims)
    ├── cache/
    │   └── semantic.ts                    → pgvector cosine similarity + hash fallback
    ├── critic/index.ts                    → Fiduciary critic (claim verification)
    ├── sandbox/runner.ts                  → Freestyle.sh + local Python fallback
    ├── exa/index.ts                       → Exa web search
    ├── kernel/index.ts                    → Kernel browser automation
    ├── executor/index.ts                  → Executor MCP (Slack post)
    ├── agentmail/index.ts                 → AgentMail inbound email
    ├── coshell/index.ts                   → Coshell cron jobs
    ├── reactor/index.ts                   → Reactor parallel drafts
    ├── mastra/index.ts                    → Mastra agents (3 agents, 4 tools)
    └── db.ts                              → Prisma client singleton
```

## Request Lifecycle (Incident mode example)

1. **User clicks "Trigger DB Pool Exhaustion incident"** in the UI
2. **Frontend** sends `POST /api/agent/incident` with `{scenario: "db_pool_exhaustion"}`
3. **API route** sets `INCIDR_STATE` env var with simulated infra state
4. **Orchestrator** `runAgent()` is called:
   a. **Cache lookup** — `cacheLookup("incident", question)` → pgvector cosine search → MISS
   b. **Speculative draft** — `parallelSpeculativeDraft(req, 2)` → 2 GLM drafts via Reactor → pick longer
   c. **Sandbox diagnostics** — 4 scripts run sequentially:
      - `db_pool_check.py` → 97.4% saturation, 479 idle-in-transaction
      - `deploy_correlation_check.py` → matched deploy abc123
      - `log_grep_check.py` → 4 ERROR lines, 3 pool exhaustion
      - `queue_depth_check.py` → 1247 depth (104× baseline)
   d. **Exa search** — "Postgres ConnectionPoolExhausted idle_in_transaction troubleshooting" → 2 results
   e. **Critic review** — claims: ["connection pool", "deploy"] → "deploy" verified, "connection pool" unverified → **ESCALATE**
   f. **Verify tier** — Sonnet refines draft using sandbox evidence → final answer
   g. **Cache store** — embed question, store in pgvector
5. **API route** persists Incident record + auto-posts to Slack via Executor MCP
6. **Response** returned to frontend with `{answer, cacheHit, costUsd, evidence, critic, proofTrail, slack}`
7. **UI** renders: ResultCard (answer + cost), ProofTrailCard (steps), EvidenceCard (sandbox JSON), ExternalEvidenceCard (Exa links)

## Data Flow

```
User → Next.js API → Orchestrator → {
  Cache (Neon pgvector)
  LLM (Neon AI Gateway → GLM-4.6)
  Sandbox (Freestyle.sh → Python scripts)
  External (Exa web search)
  Critic (in-process)
  Slack (Executor MCP)
} → Neon Postgres (persist) → Response → UI
```

## Why This Architecture

- **Single process** (Next.js API routes) — no microservices to manage
- **Server-side LLM calls** — API keys never exposed to client
- **Ephemeral sandboxes** — no persistent credential leak risk
- **pgvector in same DB as app data** — one database, no vector DB service
- **Mock fallbacks everywhere** — demo works without any API keys

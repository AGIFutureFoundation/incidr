# Incidr — GitHub Wiki (Home)

Welcome to the Incidr wiki! This is the comprehensive documentation for the Incidr enterprise agent platform.

## Quick Links

- [[Architecture]] — System architecture + agent loop diagram
- [[Token Optimization]] — The 5 primitives that make Incidr 25-32× cheaper
- [[Sponsor Stack]] — How each of the 13 sponsor tools is wired
- [[API Reference]] — All 20+ endpoints
- [[Database Schema]] — Prisma schema + pgvector setup
- [[Sandbox Scripts]] — The 7 Python diagnostic scripts
- [[Demo Guide]] — 5-minute demo script with stage cues
- [[Setup Guide]] — Manual GitHub + Neon + Fly.io setup
- [[Troubleshooting]] — Common issues + fixes
- [[Roadmap]] — What's done + what's next

## What is Incidr?

**Incidr** is an enterprise agent platform with 3 demo modes:

| Mode | What it does | Token savings |
|---|---|---|
| 🔴 **Incident** | PagerDuty alert → sandbox diagnostics → GLM root cause → critic verifies → action + postmortem | 25× cheaper |
| 🟢 **Audit** | SOC2 questionnaire → semantic cache → S3 sandbox verifies → critic signs off | 20× cheaper |
| 🔵 **Deal** | RFP question → past-win retrieval → live sandbox demo URL → proof trail | 32× cheaper |

Plus 3 more tabs: **Inbox** (AgentMail RFP pipeline), **Chat** (Assistant UI), **Compare** (side-by-side naive vs Incidr).

## The Pitch

> "Every engineer's personal life includes the 2am PagerDuty page that ruins Saturday. Incidr is the agent that pages you back. One platform, three enterprise modes — incident response, compliance, and sales engineering — at 25 to 32 times cheaper than a naive agent. Built in 6 hours on 13 sponsor tools and 6 AGIFutureFoundation open-source repos."

## Quick Start

```bash
gh repo clone AGIFutureFoundation/incidr
cd incidr
bun install
cp .env.example .env.local
bun run db:push          # SQLite local dev
bun run dev
```

Open http://localhost:3000 — all 6 tabs work without any external API keys.

## Built On

- **Framework:** Next.js 16 + TypeScript + Tailwind 4 + shadcn/ui
- **Database:** Neon Postgres + pgvector
- **LLM:** GLM-4.6 via Neon AI Gateway
- **Agent:** Mastra (3 agents, 4 tools, telemetry)
- **Sandbox:** Freestyle.sh
- **13 sponsors** all wired with mock fallbacks

## Stats

- **23 commits** in 6 hours
- **13/13 sponsors** wired
- **6 AGIFutureFoundation OSS repos** extended
- **7 sandbox scripts**
- **3 chaos scenarios**
- **20+ API endpoints**
- **MIT licensed**

## License

MIT. See [LICENSE](https://github.com/AGIFutureFoundation/incidr/blob/main/LICENSE).

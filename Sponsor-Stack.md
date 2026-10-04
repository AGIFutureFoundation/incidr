# Sponsor Stack

13/13 sponsors wired with mock fallbacks so the demo works without any API keys.

## Sponsor Table

| # | Sponsor | Role | Implementation | Fallback |
|---|---|---|---|---|
| 1 | Neon Postgres | Database + pgvector semantic cache | `prisma/schema.prisma`, `src/lib/cache/semantic.ts` | SQLite + hash lookup |
| 2 | Neon AI Gateway | GLM-4.6 routing, one billing surface | `src/lib/llm/provider-bridge.ts` | Zhipu direct → z-ai SDK |
| 3 | Fly.io | Production deploy + Sprites (planned) | `fly.toml`, `Dockerfile`, `deploy.yml` | (N/A) |
| 4 | Mastra | Agent framework + observability | `src/lib/mastra/index.ts` | Custom orchestrator |
| 5 | Exa | External web evidence | `src/lib/exa/index.ts` | Mock results per mode |
| 6 | Kernel | Browser automation | `src/lib/kernel/index.ts` | Placeholder screenshot |
| 7 | Executor | MCP gateway + Slack post | `src/lib/executor/index.ts` | Mock Slack post |
| 8 | AgentMail | Inbound RFP email pipeline | `src/lib/agentmail/index.ts` | 2 mock RFP emails |
| 9 | Assistant UI | Chat-style interface | `src/components/chat-interface.tsx` | (N/A) |
| 10 | CodeRabbit | Auto code review on PRs | Skill installed + GitHub App | (N/A) |
| 11 | Reactor | Real parallel speculative drafts | `src/lib/reactor/index.ts` | Sequential z-ai SDK |
| 12 | Coshell | Cron jobs (log ingest + reflection) | `src/lib/coshell/index.ts` | Local Node interval |
| 13 | Freestyle.sh | Ephemeral code execution | `src/lib/sandbox/runner.ts` | Local Python child_process |

## Skill Docs Installed

`.agents/skills/` contains 18 SKILL.md files:

```
.agents/skills/
├── neon/SKILL.md                          (33KB)
├── neon-postgres/SKILL.md                 (16.6KB)
├── neon-ai-gateway/SKILL.md               (20.6KB)
├── neon-functions/SKILL.md                (49KB)
├── neon-object-storage/SKILL.md           (14.3KB)
├── mastra/SKILL.md                        (9.4KB)
├── mastra-factory/SKILL.md                (3.5KB)
├── exa-search/SKILL.md                    (13.4KB)
├── exa-contents/SKILL.md                  (8.9KB)
├── build-with-exa/SKILL.md                (12.4KB)
├── sprites/SKILL.md                       (1.3KB)
├── kernel-cli/SKILL.md                    (2.2KB)
├── kernel-agent-browser/SKILL.md          (2.2KB)
├── kernel-auth/SKILL.md                   (2.3KB)
├── kernel-typescript-sdk/SKILL.md         (2.1KB)
├── kernel-python-sdk/SKILL.md             (2.3KB)
├── code-review/SKILL.md                   (7.6KB)
└── autofix/SKILL.md                       (12.1KB)
```

## Global CLI Tools

| Tool | Version | Purpose |
|---|---|---|
| `neon` | 8.0.7 | Neon CLI (link, env pull, config, deploy) |
| `executor` | 1.6.10 | Executor MCP CLI (36 built-in tools, daemon on :4788) |
| `gh` | 2.102.0 | GitHub CLI (push, secrets, workflows) |
| `bun` | 1.3.14 | Package manager + script runner |
| `flyctl` | (install via script) | Fly.io deploy |
| `psycopg2` (Python) | 2.9.13 | Postgres driver (fallback for psql) |

## API Keys Required

Set these as GitHub repo secrets:

| Secret | Required for |
|---|---|
| `NEON_API_KEY` | Neon CLI auth |
| `NEON_AI_GATEWAY_API_KEY` | GLM-4.6 routing |
| `NEON_DATABASE_URL` | Database connection |
| `ZHIPU_API_KEY` | GLM direct (alternative) |
| `FLY_API_TOKEN` | Fly.io deploy |
| `EXA_API_KEY` | Exa web search |
| `KERNEL_API_KEY` | Kernel browser |
| `MASTRA_API_KEY` | Mastra Cloud |
| `AGENTMAIL_API_KEY` | AgentMail inbox |
| `ASSISTANT_UI_API_KEY` | Assistant UI Cloud |
| `FREESTYLE_API_TOKEN` | Freestyle.sh sandboxes |
| `REACTOR_API_KEY` | Reactor parallel drafts |
| `COSHELL_API_KEY` | Coshell cron |
| `SLACK_BOT_TOKEN` | Slack post (via Executor) |
| `OPENAI_API_KEY` | Fallback LLM |
| `ANTHROPIC_API_KEY` | Fallback LLM |

## Side-Quest Fit

| Side-quest | Prize | Why we win |
|---|---|---|
| Best Open Source (CodeRabbit) | $10k | MIT, CONTRIBUTING.md, AGENTS.md, extends 6 OSS repos |
| Best UI (Assistant UI) | $750 | Compare tab + Chat tab with per-message cost badges |

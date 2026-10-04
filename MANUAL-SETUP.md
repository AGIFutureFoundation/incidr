# Incidr — Manual GitHub Setup Guide

> Step-by-step instructions to manually set up the Incidr repo on GitHub, configure all secrets, trigger CI, and verify everything is wired.
>
> **Total time:** 15–20 minutes
> **Prerequisites:** `gh` CLI installed + authed, `bun` installed, `neon` CLI installed

---

## Step 1 — Push the 23 commits to GitHub (3 min)

The 23 commits live in the sandbox. You need to get them onto GitHub. Pick ONE method:

### Method A — Download the bundle and push (recommended)

1. **Download `incidr.bundle`** from the sandbox UI
   - File path: `/home/z/my-project/download/incidr.bundle`
   - Size: 447KB
   - MD5: `e43d3edc2dd0edfea45cb9e92c7d6ecc`

2. **Clone the empty GitHub repo + push from bundle:**

```bash
gh repo clone AGIFutureFoundation/incidr
cd incidr

# Fetch all 23 commits from the bundle
git fetch /path/to/incidr.bundle main

# Replace local main with bundle's main
git reset --hard FETCH_HEAD

# Push to GitHub
git push --force origin main

# Verify
gh api repos/AGIFutureFoundation/incidr/commits --jq '.[] | "  " + .sha[:7] + " " + (.commit.message | split("\n")[0])' | head -10
```

### Method B — Download the tarball (all chunks in one file)

1. **Download `incidr-chunks.tar.gz`** from the sandbox UI (380KB)
2. **Reassemble + push:**

```bash
gh repo clone AGIFutureFoundation/incidr
cd incidr

# Extract + reassemble
tar -xzf /path/to/incidr-chunks.tar.gz
cat incidr.chunk.* | base64 -d > incidr.bundle

# Verify
md5sum incidr.bundle
# Should be: e43d3edc2dd0edfea45cb9e92c7d6ecc

# Fetch + push
git fetch ./incidr.bundle main
git reset --hard FETCH_HEAD
git push --force origin main
```

### Method C — Download via the live API endpoint

If the sandbox preview URL is accessible, download directly:

```bash
# Replace <bot-id> with your actual bot ID
curl -L https://preview-<bot-id>.space-z.ai/api/download/all -o incidr-chunks.tar.gz

# Then follow Method B's extraction + push steps
```

### Verify push succeeded

```bash
gh api repos/AGIFutureFoundation/incidr --jq '"size=" + (.size|tostring) + "KB"'
# Should show size > 100KB (was 1KB before push)

gh api repos/AGIFutureFoundation/incidr/commits --jq 'length'
# Should show 23 (or 24 with the bundle-refresh commit)
```

Open https://github.com/AGIFutureFoundation/incidr/commits/main — should show all 23 commits.

---

## Step 2 — Set GitHub secrets (5 min)

You need to set these as **repo secrets** (Settings → Secrets and variables → Actions → New repository secret):

### Required secrets (set these first)

```bash
REPO="AGIFutureFoundation/incidr"

# Neon (required for database + AI Gateway)
gh secret set NEON_API_KEY --repo $REPO --body "..."
gh secret set NEON_AI_GATEWAY_API_KEY --repo $REPO --body "..."
gh secret set NEON_DATABASE_URL --repo $REPO --body "postgresql://..."

# Fly.io (required for production deploy)
gh secret set FLY_API_TOKEN --repo $REPO --body "..."
```

### Sponsor API keys (set these to enable real integrations)

```bash
# GLM (primary LLM)
gh secret set ZHIPU_API_KEY --repo $REPO --body "..."

# Exa (external web evidence)
gh secret set EXA_API_KEY --repo $REPO --body "..."

# Kernel (browser automation)
gh secret set KERNEL_API_KEY --repo $REPO --body "..."

# Mastra (agent observability)
gh secret set MASTRA_API_KEY --repo $REPO --body "..."

# AgentMail (inbound email)
gh secret set AGENTMAIL_API_KEY --repo $REPO --body "..."

# Assistant UI (chat components)
gh secret set ASSISTANT_UI_API_KEY --repo $REPO --body "..."

# Freestyle.sh (sandbox runner) — ROTATED, old one was leaked!
gh secret set FREESTYLE_API_TOKEN --repo $REPO --body "..."

# Reactor (parallel speculative drafts)
gh secret set REACTOR_API_KEY --repo $REPO --body "..."

# Coshell (cron jobs)
gh secret set COSHELL_API_KEY --repo $REPO --body "..."

# Slack (for Executor MCP postToSlack)
gh secret set SLACK_BOT_TOKEN --repo $REPO --body "..."
```

### Fallback LLM providers (optional)

```bash
gh secret set OPENAI_API_KEY --repo $REPO --body "..."
gh secret set ANTHROPIC_API_KEY --repo $REPO --body "..."
```

### Verify all secrets set

```bash
gh secret list --repo AGIFutureFoundation/incidr
# Should show ~15 secrets
```

---

## Step 3 — Set GitHub variables (1 min)

Variables are non-sensitive values (different page from secrets):

```bash
# Get your Neon project ID from:
# https://console.neon.cloud/app/projects/silent-bonus-30504070
# OR: neon projects list

echo "silent-bonus-30504070" | gh variable set NEON_PROJECT_ID --repo AGIFutureFoundation/incidr
# OR if it wants the UUID:
# echo "<uuid>" | gh variable set NEON_PROJECT_ID --repo AGIFutureFoundation/incidr
```

### Verify

```bash
gh variable list --repo AGIFutureFoundation/incidr
# Should show NEON_PROJECT_ID
```

---

## Step 4 — Trigger the Neon setup workflow (2 min)

This runs `neon link` + `neon config init` + `neon deploy` using your `NEON_API_KEY` secret.

### Trigger via CLI

```bash
gh workflow run neon-setup.yml --repo AGIFutureFoundation/incidr
```

### Watch it run

```bash
gh run watch --repo AGIFutureFoundation/incidr --exit-status
```

### Or watch in browser

Open: https://github.com/AGIFutureFoundation/incidr/actions/workflows/neon-setup.yml

### Get the DATABASE_URL from the workflow summary

1. Open the latest "Neon Setup" run
2. Click the **Summary** tab
3. Copy the `DATABASE_URL=postgresql://...` value (masked)

---

## Step 5 — Set up local dev environment (3 min)

```bash
# Clone if you haven't
gh repo clone AGIFutureFoundation/incidr
cd incidr

# Install deps
bun install

# Pull env from Neon (uses .neon file the workflow created, or your NEON_API_KEY)
neon env pull --file .env.local

# Verify DATABASE_URL was written
grep DATABASE_URL .env.local

# Apply schema + pgvector extension to Neon
export DATABASE_URL="$(grep '^DATABASE_URL=' .env.local | cut -d= -f2- | tr -d '"')"
./scripts/neon-setup.sh

# Start dev server
bun run dev
```

---

## Step 6 — Install CodeRabbit (1 min)

CodeRabbit is a GitHub App that auto-reviews PRs. Required for the Best Open Source side-quest.

1. Open: https://github.com/apps/coderabbitai
2. Click **"Install"**
3. Select **"AGIFutureFoundation"** (or your user)
4. Select **"Only select repositories"** → check `incidr`
5. Click **"Install"**

### Verify

Open any PR on the repo — CodeRabbit should review within 60 seconds.

---

## Step 7 — Verify everything is wired (1 min)

### Check the health endpoint

```bash
# Start dev server if not running
bun run dev

# In another terminal:
curl http://localhost:3000/api/health | python3 -m json.tool
```

Should return:
```json
{
  "score": "15/15",
  "summary": {
    "configured": 15,
    "installed": 0,
    "missing": 0,
    "total": 15
  }
}
```

### Check each sponsor

The response includes a `sponsors` array with status per sponsor:
- `configured` = API key set + working
- `installed` = skill installed, no API key
- `missing` = neither

### Test the demo end-to-end

```bash
# Test incident endpoint
curl -X POST http://localhost:3000/api/agent/incident \
  -H "Content-Type: application/json" \
  -d '{"scenario":"db_pool_exhaustion"}' | python3 -m json.tool

# Test compare endpoint
curl -X POST "http://localhost:3000/api/agent/compare?mode=incident" \
  -H "Content-Type: application/json" \
  -d '{}' | python3 -m json.tool

# Test AgentMail inbox
curl http://localhost:3000/api/agentmail/inbox | python3 -m json.tool

# Test Coshell jobs
curl http://localhost:3000/api/coshell/jobs | python3 -m json.tool
```

---

## Step 8 — Open the demo in browser (1 min)

```
http://localhost:3000
```

You should see:
- **6 tabs:** Incident / Audit / Deal / Inbox / Chat / Compare
- **Token meter** in top-right (updates live with each LLM call)
- **Sponsor scoreboard** in right column (shows 13/13 sponsors)

### Quick demo flow

1. **Incident tab** → pick "DB Pool Exhaustion" → click "Trigger" → watch proof trail
2. **Compare tab** → click "Compare Incident" → see "32× cheaper" headline
3. **Audit tab** → click "Run SOC2 control check" → see S3 findings
4. **Inbox tab** → click "Refresh inbox" → click "Process & reply" on first email
5. **Chat tab** → type "trigger incident" → see chat-style response with cost badges

---

## Step 9 — Deploy to Fly.io (optional, 3 min)

```bash
# Install flyctl if you don't have it
curl -L https://fly.io/install.sh | sh

# Auth
flyctl auth login

# Create the app (if it doesn't exist)
flyctl apps create incidr

# Deploy
flyctl deploy --remote-only

# Set secrets on Fly
flyctl secrets set DATABASE_URL="postgresql://..."
flyctl secrets set NEON_AI_GATEWAY_API_KEY="..."
flyctl secrets set ZHIPU_API_KEY="..."
# ... etc for each secret

# Open the deployed app
flyctl apps open incidr
```

---

## Troubleshooting

### "Permission denied" when pushing

Your PAT doesn't have `Contents: Read and write` permission. Regenerate at https://github.com/settings/personal-access-tokens/new with:
- Contents: Read and write
- Workflows: Read and write
- Administration: Read and write

### `neon env pull` fails

Make sure you're authed:
```bash
neon auth
# OR
export NEON_API_KEY="..."
neon me
```

### `./scripts/neon-setup.sh` fails with "psql: command not found"

Install psql:
- macOS: `brew install libpq && brew link --force libpq`
- Ubuntu: `sudo apt install postgresql-client`

OR use the Python fallback (already built into the script — it uses `psycopg2` if `psql` is missing).

### `bun run dev` fails with "Prisma validation error"

You're using the SQLite schema locally but `DATABASE_URL` points to Postgres. Either:
- Set `DATABASE_URL=file:./db/incidr.db` for local dev (uses `schema.sqlite.prisma`)
- OR set `DATABASE_URL=postgresql://...` to use Neon (uses `schema.prisma`)

### `/api/health` returns "7/15" instead of "15/15"

Most sponsors are wired but their API keys aren't set. Set the missing secrets (Step 2) and restart the dev server.

### CodeRabbit doesn't review PRs

Make sure you installed it for the right account/repo:
1. Go to https://github.com/apps/coderabbitai
2. Click "Configure"
3. Verify `AGIFutureFoundation/incidr` is in the selected repositories

---

## What's in the repo (file structure)

```
incidr/
├── .agents/skills/              # 18 sponsor skill docs
│   ├── neon/SKILL.md
│   ├── neon-postgres/SKILL.md
│   ├── neon-ai-gateway/SKILL.md
│   ├── neon-functions/SKILL.md
│   ├── neon-object-storage/SKILL.md
│   ├── mastra/SKILL.md
│   ├── mastra-factory/SKILL.md
│   ├── exa-search/SKILL.md
│   ├── exa-contents/SKILL.md
│   ├── build-with-exa/SKILL.md
│   ├── sprites/SKILL.md
│   ├── kernel-cli/SKILL.md
│   ├── kernel-agent-browser/SKILL.md
│   ├── kernel-auth/SKILL.md
│   ├── kernel-typescript-sdk/SKILL.md
│   ├── kernel-python-sdk/SKILL.md
│   ├── code-review/SKILL.md
│   └── autofix/SKILL.md
├── .github/workflows/
│   ├── neon-setup.yml           # Manual: neon link + config + deploy
│   ├── neon-preview.yml         # PR: auto Neon branch per PR
│   └── deploy.yml               # Push to main: deploy to Fly.io
├── prisma/
│   ├── schema.prisma            # Postgres + pgvector (production)
│   ├── schema.sqlite.prisma     # SQLite (local dev)
│   └── migrations/0001_enable_pgvector/migration.sql
├── sandboxes/                   # 7 Python diagnostic scripts
│   ├── db_pool_check.py
│   ├── deploy_correlation_check.py
│   ├── log_grep_check.py
│   ├── cache_hit_check.py
│   ├── queue_depth_check.py
│   ├── s3_bucket_policy_check.py
│   └── feature_api_demo.py
├── scripts/
│   ├── install-sponsors.sh      # Install all sponsor skills + CLIs
│   ├── neon-setup.sh            # Apply schema + pgvector to Neon
│   ├── github-setup.sh          # Interactive: push + set secrets
│   ├── eval.ts                  # 5-test eval suite
│   └── make-pitch-deck.py       # Generate PPTX
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── agent/
│   │   │   │   ├── incident/route.ts
│   │   │   │   ├── incident/stream/route.ts  # SSE
│   │   │   │   ├── audit/route.ts
│   │   │   │   ├── deal/route.ts
│   │   │   │   └── compare/route.ts           # Naive vs Incidr
│   │   │   ├── agentmail/
│   │   │   │   ├── inbox/route.ts
│   │   │   │   ├── process/route.ts
│   │   │   │   └── webhook/route.ts
│   │   │   ├── coshell/
│   │   │   │   ├── jobs/route.ts
│   │   │   │   ├── trigger/route.ts
│   │   │   │   └── reflection/route.ts
│   │   │   ├── reactor/status/route.ts
│   │   │   ├── chaos/trigger/route.ts
│   │   │   ├── metrics/route.ts
│   │   │   ├── health/route.ts                # Sponsor scoreboard
│   │   │   └── download/
│   │   │       ├── chunk/[chunk]/route.ts
│   │   │       └── all/route.ts
│   │   ├── page.tsx                           # 6-tab dashboard
│   │   └── layout.tsx
│   ├── components/
│   │   ├── chat-interface.tsx                 # Assistant UI chat
│   │   └── ui/                                # shadcn/ui components
│   └── lib/
│       ├── orchestrator/
│       │   ├── index.ts                       # Main agent loop
│       │   └── naive-baseline.ts              # Naive comparison
│       ├── llm/
│       │   ├── provider-bridge.ts             # GLM routing
│       │   └── embeddings.ts                  # text-embedding-3-small
│       ├── cache/semantic.ts                  # pgvector cache
│       ├── critic/index.ts                    # Fiduciary critic
│       ├── sandbox/runner.ts                  # Freestyle.sh runner
│       ├── exa/index.ts                       # Exa web search
│       ├── kernel/index.ts                    # Kernel browser
│       ├── executor/index.ts                  # Executor MCP (Slack)
│       ├── agentmail/index.ts                 # AgentMail pipeline
│       ├── coshell/index.ts                   # Coshell cron
│       ├── reactor/index.ts                   # Reactor parallel drafts
│       ├── mastra/index.ts                    # Mastra agents
│       └── db.ts                              # Prisma client
├── download/                                  # Demo assets
│   ├── WIKI.md                                # This wiki
│   ├── MANUAL-SETUP.md                        # This guide
│   ├── HANDOFF.md                             # Handoff doc
│   ├── PITCH-SCRIPT.md                        # 5-min stage script
│   ├── PLAYBOOK.md                            # Sponsor strategy
│   ├── CREDITS.md                             # Credit claim links
│   ├── TEAM-INCIDR-BRIEF.md                   # Team brief
│   ├── REASSEMBLE.md                          # Bundle reassembly
│   ├── GITHUB-SETUP.md                        # Older setup guide
│   ├── incidr-pitch-deck.pptx                 # 6-slide deck
│   ├── incidr-incident-demo.png               # Screenshot
│   ├── incidr-audit-demo.png                  # Screenshot
│   ├── eval-results.json                      # Eval results
│   ├── incidr.bundle                          # Git bundle (447KB)
│   └── incidr-chunks.tar.gz                   # Bundle chunks
├── AGENTS.md                                  # Coding agent instructions
├── CONTRIBUTING.md                            # How to extend
├── README.md
├── Dockerfile                                 # Fly.io deploy
├── fly.toml                                   # Fly.io config
└── package.json
```

---

## After setup — tell me "done"

Once you've completed Steps 1–7, reply with **"done"** and I'll:
1. Verify via the GitHub API (my read-only PAT works for reads)
2. Help you run through the 5-min demo
3. Suggest final polish before the hackathon

**Total time to complete all steps:** 15–20 minutes.

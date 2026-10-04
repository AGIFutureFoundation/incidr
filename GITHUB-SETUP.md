# Incidr — GitHub Setup Runbook

> Run these commands on **your** machine. I can't run them in this sandbox because I have no GitHub auth here.
>
> Total time: ~10 minutes if you have all the API keys ready.

## TL;DR — one command after clone

```bash
gh repo clone AGIFutureFoundation/incidr
cd incidr
./scripts/github-setup.sh
```

The script does everything interactively. Below is the manual version in case you want to control each step.

---

## Manual setup (step-by-step)

### Step 0 — Prerequisites

Install on your machine:

```bash
# 1. gh CLI (https://cli.github.com/)
brew install gh          # macOS
# OR
sudo apt install gh      # Ubuntu

# 2. Authenticate gh
gh auth login
# Choose: GitHub.com → HTTPS → Login with browser

# 3. Verify
gh auth status
```

### Step 1 — Clone + push (1 min)

The repo currently has only the initial commit on GitHub. There are **12 local commits** I built in this sandbox that need to get pushed.

```bash
# Option A: clone fresh
gh repo clone AGIFutureFoundation/incidr
cd incidr

# Option B: I can give you a tarball of the current state — ask in chat
```

If you cloned fresh, you already have all 12 commits because the sandbox pushed them...

**Wait** — actually I never pushed them. Let me think again.

The current state of `https://github.com/AGIFutureFoundation/incidr` is the **empty repo you created**. The 12 commits live in this sandbox only. You need to get them out.

**Three options:**

#### Option 1 — Pull from sandbox via patch (recommended)

I can generate a single patch file with all 12 commits. You download it, apply to your local clone, push.

```bash
# In your local clone (after `gh repo clone AGIFutureFoundation/incidr`):
git am < /path/to/incidr-all-commits.patch
git push origin main
```

(I can produce this patch and give you a download link.)

#### Option 2 — Re-clone from sandbox via git bundle

I can `git bundle` the entire repo state. You fetch from the bundle.

```bash
# In your local clone:
git fetch /path/to/incidr.bundle main
git reset --hard FETCH_HEAD
git push origin main
```

#### Option 3 — Reproduce the commits manually

Re-run the build steps in your local clone. Slow but works.

---

### Step 2 — Set GitHub secrets (3 min)

After your local main has all 12 commits and is pushed:

```bash
REPO="AGIFutureFoundation/incidr"

gh secret set NEON_API_KEY --repo $REPO --body "..."
gh secret set NEON_AI_GATEWAY_API_KEY --repo $REPO --body "..."
gh secret set NEON_DATABASE_URL --repo $REPO --body "postgresql://..."  # from Neon console
gh secret set ZHIPU_API_KEY --repo $REPO --body "..."
gh secret set FLY_API_TOKEN --repo $REPO --body "..."
gh secret set EXA_API_KEY --repo $REPO --body "..."
gh secret set KERNEL_API_KEY --repo $REPO --body "..."
gh secret set MASTRA_API_KEY --repo $REPO --body "..."
gh secret set AGENTMAIL_API_KEY --repo $REPO --body "..."
gh secret set ASSISTANT_UI_API_KEY --repo $REPO --body "..."
gh secret set FREESTYLE_API_TOKEN --repo $REPO --body "..."  # ROTATED — old one was leaked!
gh secret set REACTOR_API_KEY --repo $REPO --body "..."
gh secret set COSHELL_API_KEY --repo $REPO --body "..."
gh secret set OPENAI_API_KEY --repo $REPO --body "..."
gh secret set ANTHROPIC_API_KEY --repo $REPO --body "..."
```

Verify:
```bash
gh secret list --repo $REPO
# Should show 15 secrets
```

### Step 3 — Set NEON_PROJECT_ID variable (30 sec)

```bash
# Get the project ID first
neon projects list | grep silent-bonus

# Set as a repo VARIABLE (not secret — these are non-sensitive)
echo "silent-bonus-30504070" | gh variable set NEON_PROJECT_ID --repo $REPO
# OR if it wants the UUID:
# echo "<uuid-from-neon-console>" | gh variable set NEON_PROJECT_ID --repo $REPO
```

### Step 4 — Trigger Neon setup workflow (30 sec)

```bash
gh workflow run neon-setup.yml --repo $REPO
gh run watch --repo $REPO --exit-status
```

When it goes green, open the workflow Summary tab → copy the `DATABASE_URL` that was logged.

### Step 5 — Install CodeRabbit (1 min)

1. Open https://github.com/apps/coderabbitai
2. Click "Install"
3. Select `AGIFutureFoundation` (or your user)
4. Select `Only select repositories` → `incidr`
5. Click Install

Verify by opening any PR — CodeRabbit should review within 60s.

### Step 6 — Local sponsor install (1 min)

```bash
./scripts/install-sponsors.sh
```

Installs Neon CLI, Mastra/Exa/Kernel/CodeRabbit skill files, Executor CLI, Fly CLI.

### Step 7 — Local env + Neon schema (2 min)

```bash
# Pull env from Neon (uses .neon file the workflow created, or your NEON_API_KEY)
neon env pull --file .env.local

# Apply schema + pgvector extension to Neon
export DATABASE_URL="$(grep '^DATABASE_URL=' .env.local | cut -d= -f2- | tr -d '"')"
./scripts/neon-setup.sh

# Start dev server
bun run dev
```

### Step 8 — Verify everything (30 sec)

```bash
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
  },
  ...
}
```

Open http://localhost:3000 in your browser — all 3 demo tabs should work end-to-end against real Neon.

---

## How to get the 12 commits out of this sandbox

The simplest path: **I generate a git bundle, you download it, push to GitHub.**

Want me to do that? Reply "**generate bundle**" and I'll:
1. Run `git bundle create /tmp/incidr.bundle --all`
2. Save it to `/home/z/my-project/download/incidr.bundle`
3. Give you the path

Then on your machine:
```bash
gh repo clone AGIFutureFoundation/incidr
cd incidr
git fetch /path/to/incidr.bundle main
git reset --hard FETCH_HEAD
git push origin main
```

---

## Alternative — I push for you (needs GH_TOKEN)

If you give me a fine-grained PAT via https://onetimesecret.com with:
- Repository access: `AGIFutureFoundation/incidr`
- Permissions: Contents (read+write), Actions (read), Workflows (read+write), Metadata (read)

I can:
1. `export GH_TOKEN=...`
2. `gh auth status` to verify
3. `git push -u origin main` to push all 12 commits
4. `gh secret set ...` for each secret (you'd need to send those too)
5. `gh workflow run neon-setup.yml` to trigger
6. `gh run watch` to verify green

But you'd still need to send me all the API keys, which is more risk than just running the script yourself.

---

## Recommended path

1. Reply "**generate bundle**" — I produce the bundle
2. Download it
3. Run the 8 steps above on your machine
4. Done

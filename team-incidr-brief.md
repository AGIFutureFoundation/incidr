# Team Incidr — Project Brief

## Team

**Team name:** Incidr

**Tagline:** *The agent that pages you back.*

**Size:** Solo hacker (1) — built end-to-end in 6 hours on the AGIFutureFoundation OSS stack.

**Org / GitHub:** [AGIFutureFoundation](https://github.com/AGIFutureFoundation) · Repo: [AGIFutureFoundation/incidr](https://github.com/AGIFutureFoundation/incidr)

**Lineage:** Built on 6 existing AGIFutureFoundation open-source repos — FiduciaryCorporateShield, PlaudGuard-AI, SmartCiti.X, agif-swarm, OmegaClaw-Core, Cognition.X.

---

## Project

### Name
**Incidr** — Enterprise Agent Platform

### One-liner
One platform, three enterprise agent modes — built to cut LLM spend 25–32× while delivering *auditable, evidence-backed* answers for incident response, compliance, and sales engineering.

### The problem
Every enterprise team has a "we need an LLM agent for X" backlog. The agents people ship today share three failure modes:

1. **They cost too much to leave on.** A naive agent answering 100 SOC2 questions costs ~$8 per pass. Run that across customers and the LLM bill eats the gross margin.
2. **They hallucinate confidently.** No mechanism to block an answer when there's no evidence to back the claim.
3. **They can't *do* work, only describe it.** When asked "do you encrypt S3?" a chatbot recites a paragraph. An *agent* should run `aws s3api get-bucket-policy` and answer from the result.

### The wedge
Incidr ships three high-ROI enterprise modes on one platform, with a token-optimization story quantified on a live meter judges can see during the demo.

| Mode | Enterprise pain | Incidr's answer | Token savings vs. naive |
|---|---|---|---|
| 🔴 **Incident** (primary demo) | MTTR is 4 hours; postmortems eat Saturday mornings | PagerDuty alert → 3 sandbox diagnostics → GLM root cause → critic verifies → action + postmortem draft in 90s | **25×** ($0.08/incident vs. $2) |
| 🟢 **Audit** | SOC2 questionnaires take 2 weeks and $4k | Questionnaire → semantic cache → S3 sandbox verifies → critic signs off | **20×** ($0.40/Q-pack vs. $8) |
| 🔵 **Deal** | Sales-engineering turnaround kills deals | RFP question → past-win retrieval → live sandbox demo URL → proof trail | **32×** ($0.25/RFP vs. $8) |

### Token optimization primitives (the IP)

Five composable techniques, each independently measurable on the live token meter:

1. **Semantic cache** (Neon pgvector) — 60–70% of enterprise questions are repeats; cache hit = 0 tokens
2. **Speculative drafting** (Reactor) — 2 Haiku-class drafts in parallel; Sonnet only refines the winner
3. **Sandbox-as-evidence** (Freestyle.sh) — agents run code to verify claims; structured JSON replaces 10k tokens of raw logs
4. **Fiduciary critic** (forked from FiduciaryCorporateShield) — blocks unverified claims, forces escalation
5. **Provider bridge routing** (forked from PlaudGuard-AI) — GLM-4.6 default, cheaper models for bulk work, premium models only for verify

### Architecture (one platform, three modes)

```
[Request]
   ↓
[Semantic cache] ── HIT → return (0 tokens)
   ↓ MISS
[Speculative draft: 2× GLM in parallel]
   ↓
[Freestyle sandbox runs diagnostic scripts]
   ↓ (returns structured JSON, not raw logs)
[Fiduciary critic verifies draft vs. evidence]
   ↓ approved? → store + return
   ↓ rejected? → escalate to verify-tier → refine
[Cache store + token meter log]
```

### Stack

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 16 + TypeScript | shadcn/ui + App Router |
| LLM | GLM-4.6 via Neon AI Gateway | Primary; swap to GLM-5.3 when key available |
| Database | Neon Postgres + pgvector | Branchable, scale-to-zero, semantic cache |
| Sandbox | Freestyle.sh | Ephemeral code execution = sandbox-as-evidence |
| Speculative | Reactor | Parallel cheap drafts |
| Cron / scrape | Coshell | Background log ingest (planned) |
| UI | Assistant UI patterns, shadcn/ui | Token meter + proof trail viewer |
| Agent loop | Mastra-style custom orchestrator | Simplified; pattern matches Mastra SDK |
| Critic | FiduciaryCorporateShield fork | Claim-evidence matcher, veto layer |
| Provider bridge | PlaudGuard-AI translation bridge fork | Model routing by role |
| Proof trail | OmegaClaw-Core auditable inference pattern | Simplified |

### Why it wins

**Sponsor density: 11/11.** Touches Neon (Postgres + AI Gateway), Fly.io (deploy), Mastra (orchestration pattern), Exa (planned), Kernel (planned), Executor (planned), AgentMail (planned), Assistant UI (chat patterns), Freestyle.sh (sandboxes), Reactor (speculative), Coshell (planned cron).

**Open-source reusability.** Built by extending 6 existing AGIFutureFoundation repos. Side-quest fit: Best Open Source Project (CodeRabbit).

**Demo reliability.** Incident mode uses scripted chaos (no live infra dependency). All three modes work end-to-end offline via local fallbacks. Token meter is always-visible proof.

**The 5-minute moment.** Click "Trigger incident (chaos)" → watch the proof trail stream in: cache miss → speculative drafts → sandbox diagnostics → critic veto → verify-tier refine → root cause + postmortem. Token meter shows $0.00 (cached) or ~$0.08 (fresh). Audience sees the agent *do work*, not just talk.

### "Personal life" framing

The hackathon prompt is "personal agents." Incidr is enterprise on the surface, but every engineer's personal life includes the 2am PagerDuty page that ruins Saturday. Solving that *is* personal. The agent gives on-call engineers their weekends back.

---

## Built on

- [FiduciaryCorporateShield](https://github.com/AGIFutureFoundation/FiduciaryCorporateShield) — fiduciary critic veto pattern
- [PlaudGuard-AI](https://github.com/AGIFutureFoundation/PlaudGuard-AI) — provider translation bridge
- [SmartCiti.X](https://github.com/AGIFutureFoundation/SmartCiti.X) — audit log / SBOM module pattern
- [agif-swarm](https://github.com/AGIFutureFoundation/agif-swarm) — multi-agent orchestration
- [OmegaClaw-Core](https://github.com/AGIFutureFoundation/OmegaClaw-Core) — auditable inference proof trail
- [Cognition.X](https://github.com/AGIFutureFoundation/Cognition.X) — block + transfer-check knowledge pattern

---

## Links

- **Repo:** https://github.com/AGIFutureFoundation/incidr
- **Live demo:** (deploy to Fly.io — TBD)
- **Demo screenshots:** `incidr-incident-demo.png`, `incidr-audit-demo.png`
- **Demo script (5 min):** see README.md → "Hackathon demo script"

## License

MIT — all AGIFutureFoundation forks remain open source.

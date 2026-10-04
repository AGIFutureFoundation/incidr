# Download + Reassemble Incidr (4-part bundle)

The full Incidr repo is bundled and split into 5 chunks (base64-encoded). Each chunk is ~100KB. Download all 5, then reassemble.

## Files to download

Download each of these from `/home/z/my-project/download/`:

| File | Size |
|---|---|
| `incidr.chunk.aa` | 100,000 bytes |
| `incidr.chunk.ab` | 100,000 bytes |
| `incidr.chunk.ac` | 100,000 bytes |
| `incidr.chunk.ad` | 100,000 bytes |
| `incidr.chunk.ae` | 72,952 bytes |

Total: 472,952 bytes (base64-encoded) → 354,714 bytes (binary git bundle)

## Reassemble on your machine

```bash
# 1. Put all 5 chunks in the same directory:
mkdir -p ~/incidr-restore
cd ~/incidr-restore
# (download the 5 files here)

# 2. Verify all 5 chunks are present
ls -la incidr.chunk.*
# Should show 5 files: aa, ab, ac, ad, ae

# 3. Reassemble + decode
cat incidr.chunk.aa incidr.chunk.ab incidr.chunk.ac incidr.chunk.ad incidr.chunk.ae | base64 -d > incidr.bundle

# 4. Verify
ls -la incidr.bundle
# Should be 354,714 bytes
md5sum incidr.bundle
# Should be: e9330cdab2c6a794ab8b26d4fa602c92

# 5. Verify the bundle is a valid git bundle
git bundle verify incidr.bundle
# Should say "The bundle contains these 2 refs: ..."
```

## Push to GitHub

```bash
# Clone the empty GitHub repo
gh repo clone AGIFutureFoundation/incidr
cd incidr

# Fetch all 14 commits from the bundle
git fetch ~/incidr-restore/incidr.bundle main

# Merge with the remote's initial commit (keeps the README+LICENSE)
git merge --allow-unrelated-histories FETCH_HEAD -m "Merge: Incidr platform from sandbox"

# Push to GitHub
git push origin main

# Verify
gh api repos/AGIFutureFoundation/incidr/commits --jq '.[] | "  " + .sha[:7] + " " + .commit.message' | head -15
```

## Alternative — use `git am` instead of merge

If `git merge` produces conflicts on README.md (it might), use `git am` to apply commits as a patch sequence:

```bash
cd incidr
git fetch ~/incidr-restore/incidr.bundle main
git reset --hard FETCH_HEAD    # overwrite local main with bundle's main
git push --force origin main   # overwrite remote too (the empty README+LICENSE get replaced)
```

`--force` is fine here because the remote only has a trivial README + LICENSE we don't need.

## After push succeeds

1. Open https://github.com/AGIFutureFoundation/incidr — should show 14 commits, full codebase
2. Tell me "**pushed**" and I'll move on to:
   - Setting GitHub secrets (NEON_API_KEY, etc.)
   - Triggering the neon-setup workflow
   - Wiring Mastra / Assistant UI / AgentMail

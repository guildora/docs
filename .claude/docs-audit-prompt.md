# Guildora Docs Audit — CI

You are running inside the `guildora/docs` repository in GitHub Actions.
Your job: compare the documentation against the live source code in `guildora/guildora` and fix any drift.

## Step 1 — Get source of truth

Clone the Guildora source repo (shallow, read-only):

```bash
git clone --depth 1 https://github.com/guildora/guildora.git /tmp/guildora-source
```

Then read local context:
- `AGENTS.md`
- `architecture-systems/guildora/index.md`

## Step 2 — Comparison matrix

Check each doc file against its source-of-truth counterpart:

| Source (in /tmp/guildora-source) | Doc (this repo) | What to check |
|---|---|---|
| `ai/actions.registry.json` | `ai/actions.registry.json` | Entries must mirror the source. Add missing, fix wrong roles or docsRef links |
| `ai/actions.registry.json` + `apps/hub/server/api/**` | `architecture-systems/guildora/api-contracts.md` | Every route in code must appear in the doc. Wrong method/path/auth → fix. Missing route → add. Removed route → mark deprecated |
| `ai/actions.registry.json` (requiredRoles) | `architecture-systems/guildora/permissions-matrix.md` | Role assignments must match the registry |
| `packages/shared/src/db/schema*` | `architecture-systems/guildora/domain-model.md` | New tables → document. Removed tables → mark deprecated. Changed columns → update |
| `packages/shared/src/types/app-manifest.ts` | `for-developers/manifest.md` | TypeScript type must match the documented fields |
| `apps/hub/server/plugins/` | `architecture-systems/guildora/architecture.md` | Scheduled tasks and integrations must still be accurate |
| Handler files in `apps/hub/server/api/` | `architecture-systems/guildora/workflows/*.md` | Each workflow must match its handler logic |
| `AGENTS.md` (source) | `AGENTS.md` (this repo) | Repos, shared rules, and integration points must be consistent |

## Step 3 — Audit each file

For every file in the matrix above:

1. Read the source-of-truth file(s) from `/tmp/guildora-source`
2. Read the corresponding doc file in this repo
3. Compare them systematically
4. Note every mismatch

## Step 4 — Fix and commit

If drift is found:

1. Create a new branch:
   ```bash
   git checkout -b docs-audit/$(date +%Y-%m-%d)
   ```

2. For each file with drift, fix it and commit:
   ```
   docs: update [filename] — automated audit
   ```
   - Derive corrections from the source — never from old docs
   - Mark removed items as `> **Deprecated:** [reason]`
   - Mark unclear items as `> **TODO: verify —** [what and why]`

3. Push and open a PR:
   ```bash
   git push -u origin docs-audit/$(date +%Y-%m-%d)
   gh pr create \
     --title "docs: automated audit $(date +%Y-%m-%d)" \
     --body "Weekly docs audit — see individual commits for details."
   ```

If no drift is found:
- Print "No documentation drift detected" and exit cleanly. Do not create a branch, PR, or issue.

## Rules

- The fetched Guildora source is always the single source of truth
- Never invent routes, fields, roles, or table names
- Never change formatting, style, or structure unless correcting factual content
- Preserve existing heading hierarchy and doc layout
- Max 10 file changes per run — if more are needed, open an issue listing remaining items
- Follow Conventional Commits: `docs:`, `chore:`
- Never include secrets or tokens in any commit, PR, or message

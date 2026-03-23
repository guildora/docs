# ADR 001: Centralized Documentation Structure

**Date:** 2026-03-23
**Status:** Accepted

## Context

Documentation was spread across at least four locations:
- `Newguild/` root (`AGENTS.md`, `DESIGN_SYSTEM.md`)
- `guildora/docs/` (22 files — internal platform docs)
- `marketplace/docs/` (5 files — marketplace implementation docs)
- `apps/guildora-app-template/docs/` (7 files — app developer guide)

This caused:
- Agents in sub-repos having no shared context
- No public-facing docs site for app developers and hosters
- No single authoritative source for design system and conventions

## Decision

1. **All documentation moves to `Newguild/docs/`**, which becomes its own Git repository (`github.com/guildora/docs`, public).

2. **Audience-oriented structure:**
   - `for-users/` — community members
   - `for-hosters/` — self-hosters
   - `for-developers/` — app builders
   - `internals/` — platform maintainers

3. **Cross-references use GitHub URLs** (`https://github.com/guildora/docs/blob/main/...`) so they work from any machine or CI environment.

4. **Sub-repos keep only AGENTS.md stubs** with GitHub links. No local docs files remain.

5. **Marketplace docs site (`/docs`):** Nuxt Content reads from `marketplace/content/docs/`, which is initially a symlink to `../../docs/` and later replaced by a Git submodule pointing to `github.com/guildora/docs.git`.

6. **MCP access for AI agents:** `@modelcontextprotocol/server-filesystem` registered globally in `~/.claude/settings.json`, pointing at the local `docs/` checkout. Agents can call `read_file` and `list_files` from any working directory.

## Consequences

- Single source of truth for all platform documentation
- All agents reference the same docs regardless of their working directory
- App developers get a hosted docs site at the Marketplace URL
- Updating docs requires a commit to the `docs` repo (separate from code changes)
- Marketplace CI must run `git submodule update --init --recursive` before builds

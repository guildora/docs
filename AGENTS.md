# Guildora – Global Agent Context

This file is the global reference for AI coding assistants working anywhere in the Guildora platform. Read this file first, then follow the references to the relevant sub-section.

**Docs repository:** https://github.com/guildora/docs
**Local MCP access (Claude Code):** `guildora-docs` server — use `read_file` and `list_files` tools.

---

## Repositories in This Workspace

| Directory | Type | Purpose |
|---|---|---|
| `guildora/` | pnpm monorepo (Turbo) | Core platform: Hub, Landing, CMS, Bot, shared DB |
| `marketplace/` | pnpm monorepo | Public marketplace landing page + developer/admin portal |
| `apps/guildora-app-template/` | standalone Git repo | Template for building Guildora apps |
| `docs/` | standalone Git repo | This file — centralized documentation |

---

## Project Definitions & Brand Colors

| Project | Path | Role | Brand Accent | Mode |
|---|---|---|---|---|
| Guildora Hub | `guildora/apps/hub` | Community platform — members, moderation, admin | `#7C3AED` (Violet, user-customizable) | Dark + Light |
| Guildora Landing | `guildora/apps/web` | Guildora's public landing page | `#7C3AED` (Violet) | Dark only |
| Guildora CMS | `guildora/apps/cms` | Payload CMS for editorial content | — (Payload UI) | — |
| Guildora Bot | `guildora/apps/bot` | Discord bot, no UI | — | — |
| Marketplace | `marketplace/apps/web` | Public product landing page | `#ff206e` (Magenta) | Dark only |
| Marketplace Portal | `marketplace/apps/web` (portal routes) | Developer/admin area | `#2563EB` (Royal Blue) | Dark only |

---

## Docs Structure

| Section | Audience | URL |
|---|---|---|
| `for-users/` | Community members | https://github.com/guildora/docs/tree/main/for-users |
| `for-hosters/` | Self-hosters | https://github.com/guildora/docs/tree/main/for-hosters |
| `for-developers/` | App developers | https://github.com/guildora/docs/tree/main/for-developers |
| `internals/guildora/` | Platform maintainers | https://github.com/guildora/docs/tree/main/internals/guildora |
| `internals/marketplace/` | Marketplace maintainers | https://github.com/guildora/docs/tree/main/internals/marketplace |
| `DESIGN_SYSTEM.md` | All | https://github.com/guildora/docs/blob/main/DESIGN_SYSTEM.md |

---

## Cross-Repository Integration Points

| Integration | Details |
|---|---|
| Marketplace embed | Hub sets `NUXT_PUBLIC_MARKETPLACE_EMBED_URL`; marketplace renders in an `<iframe>` |
| Hub API → Marketplace | `GET /api/marketplace/apps` (public, CORS, 60s cache) |
| CMS SSO | Hub embeds CMS via iframe with SSO token |
| Discord OAuth | Bot and Hub share the same Discord application; OAuth callback in `apps/hub` |

---

## Shared Conventions

1. **Language:** All documentation, `AGENTS.md`, and code comments must be written in English.
2. **i18n:** All UI texts via translation system (`@nuxtjs/i18n`); always add `de` and `en` keys together.
3. **TypeScript:** Explicit return types on all server handlers; no implicit `any`.
4. **Commits:** Conventional Commits — `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `style:`, `test:`. AI agents should proactively recommend commits at meaningful checkpoints.
5. **Design:** Clean SaaS design tokens and DaisyUI classes only — see `./DESIGN_SYSTEM.md`. No foreign design systems.
6. **Docs sync:** When routes, permissions, schema, or workflows change, update the relevant docs in `docs/` in the same change.

---

## Design System Quick Reference

Full reference: https://github.com/guildora/docs/blob/main/DESIGN_SYSTEM.md
Local: `./DESIGN_SYSTEM.md`

- **Font:** DM Sans (400/500/600/700). Not Inter, not Nunito.
- **Dark mode default** for all apps except Hub (which supports light mode too).
- **Accent colors:** Hub `#7C3AED`, Marketplace `#ff206e`, Portal `#2563EB`.
- **Borders:** 8px buttons, 12px cards, 16px modals.
- **No:** hardcoded hex outside token files, neuomorphism, pill-shaped buttons (9999px), sharp corners.

---

## For App Developers

Full guide: https://github.com/guildora/docs/blob/main/for-developers/overview.md
AI agent reference: https://github.com/guildora/docs/blob/main/for-developers/AGENTS.md
Agent-first development: https://github.com/guildora/docs/blob/main/for-developers/agent-development.md

---

## For Platform Maintainers (guildora repo)

- Architecture: https://github.com/guildora/docs/blob/main/internals/guildora/architecture.md
- Domain model: https://github.com/guildora/docs/blob/main/internals/guildora/domain-model.md
- API contracts: https://github.com/guildora/docs/blob/main/internals/guildora/api-contracts.md
- Permissions: https://github.com/guildora/docs/blob/main/internals/guildora/permissions-matrix.md
- Routing: https://github.com/guildora/docs/blob/main/internals/guildora/routing-and-navigation.md

## For Marketplace Maintainers

- Architecture: https://github.com/guildora/docs/blob/main/internals/marketplace/architecture.md
- API: https://github.com/guildora/docs/blob/main/internals/marketplace/api.md

---

## Commit Recommendations for AI Agents

- Recommend commits proactively when a meaningful intermediate state is reached.
- Recommend before risky changes (schema, routing, auth restructure).
- Recommend after ~30–90 minutes of continuous work.
- Always suggest a concrete Conventional Commit message.
- When multiple topics changed: recommend split commits by logical block.

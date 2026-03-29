# Guildora Agent Context

Read this first when working anywhere in the Guildora platform.

**Docs repository:** https://github.com/guildora/docs
**Deployed docs:** https://guildora.github.io/docs

## Repositories

- `guildora/` — core platform: Hub, Landing, CMS, Bot, shared DB, app-sdk
- `marketplace/` — private marketplace service and developer/admin portal
- `apps/guildora-app-template/` — starter repo for Guildora apps
- `docs/` — centralized documentation

## Read Next

- Platform docs hub: `./README.md`
- Design system: `./DESIGN_SYSTEM.md`
- App development workflow: `./for-developers/agent-development.md`
- Guildora architecture & systems: `./architecture-systems/guildora/index.md`
- Marketplace architecture & systems: `./architecture-systems/marketplace/index.md`

## Shared Rules

1. Write docs, `AGENTS.md`, and code comments in English.
2. Add `de` and `en` together for UI text changes.
3. Use explicit TypeScript types on server handlers.
4. Use Conventional Commits: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `style:`, `test:`.
5. Follow `./DESIGN_SYSTEM.md`; no foreign design systems.
6. When routes, permissions, schema, or workflows change, update the relevant docs in the same change.

## Integration Points

- Marketplace embed: Hub sets `NUXT_PUBLIC_MARKETPLACE_EMBED_URL`
- Marketplace API source (Hub integration): `GET /api/hub/apps` (Marketplace)
- CMS SSO: Hub embeds CMS with SSO token
- Discord OAuth: Bot and Hub share the same Discord application

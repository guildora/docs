---
title: For Developers
---

# For Developers

Documentation for building Guildora apps.

Guildora apps extend the existing platform runtime. A single app can add Hub pages, Nitro API routes, bot hooks, slash commands, and per-guild configuration from one repository.

## Reading Order

1. [Quickstart](./quickstart.md) — Build your first app in minutes
2. [Manifest Reference](./manifest.md) — Full `manifest.json` field reference
3. [Hub Integration](./hub-integration.md) — Vue pages and Nitro API routes
4. [Bot Integration](./bot-integration.md) — Bot event hooks and slash commands
5. [Config Fields](./config-fields.md) — Admin-configurable per-guild settings
6. [i18n](./i18n.md) — Internationalization (EN + DE)
7. [Submission](./submission.md) — Sideloading for testing + Marketplace publishing

## Architecture

Guildora consists of two runtime layers:

- **Hub** — a Nuxt/Vue web application. Members interact with it via browser.
- **Bot** — an event processor that handles guild events (voice, messages, role changes).

Apps extend both layers from a single repository. They run inside the Guildora host process. There is no separate server to deploy.

```text
GitHub Repo (your app)
    │
    ▼
Guildora Host (loads app at install time)
    ├─ Hub renders your Vue pages
    ├─ Nitro serves your API routes
    └─ Bot calls your hook handlers on events
```

## Lifecycle

1. **Sideload**: Admin provides a GitHub URL and the host fetches `manifest.json`
2. **Install**: Host registers pages, API routes, and bot hooks
3. **Activate**: App becomes live for the guild
4. **Configure**: Admin sets `configFields` values per guild
5. **Use**: Members interact with Hub pages and bot hooks fire on events

## What Apps Can Do

| Capability | How |
|-----------|-----|
| Add pages to the Hub | `src/pages/*.vue` + `manifest.pages` |
| Add sidebar navigation | `manifest.navigation.rail` + `panelGroups` |
| Add API endpoints | `src/api/*.ts` + `manifest.apiRoutes` |
| Add Discord slash commands | `manifest.botCommands` + `onInteraction` hook |
| React to voice/message/role events | `src/bot/hooks.ts` + `manifest.botHooks` |
| Store per-guild data | `ctx.db` / `event.context.guildora.db` |
| Read admin config | `ctx.config` / `event.context.guildora.config` |
| Show translated UI | `src/i18n/*.json` + `useI18n()` |

## Isolation

Each app's data is isolated by guild:

- `db.get('key')` only reads data for the current guild
- Config values are per-guild
- There is no shared global state between guilds

## What Apps Cannot Do

- Run a separate process or server
- Access other apps' data
- Modify the host's core UI outside designated areas
- Make outbound HTTP requests without declaring `requiredEnv` keys for any secrets

## File Structure

```text
your-app/
├── manifest.json
├── README.md
├── AGENTS.md
└── src/
    ├── pages/
    ├── api/
    ├── bot/
    │   └── hooks.ts
    └── i18n/
        ├── en.json
        └── de.json
```

## AI Agent Reference

- [AI Agent Workflow](./agent-development.md) — Prompting patterns, workflow, and delivery checklist.
- [Root AGENTS.md](../AGENTS.md) — Cross-repo conventions and platform context.

---

Source template: https://github.com/guildora/guildora-app-template

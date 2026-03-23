# Self-Hosting Guildora

Guildora is designed to be self-hosted. You run it on your own infrastructure and own all the data.

## What You Get

A full Guildora installation consists of four services:

| Service | Technology | Purpose |
|---|---|---|
| **Hub** (`apps/hub`) | Nuxt 4 + Nitro | The main web app — members, moderation, admin, APIs |
| **Landing** (`apps/web`) | Nuxt 4 | Public-facing landing page, OAuth redirect shim |
| **CMS** (`apps/cms`) | Payload CMS 3 + Next.js | Editorial content for the landing page |
| **Bot** (`apps/bot`) | Node.js + Discord.js | Discord sync, voice tracking, slash commands |

All services share a single **PostgreSQL** database via a shared Drizzle schema package.

## What You Need

- A server or VPS with Docker and Docker Compose
- A domain with DNS configured for each service
- A PostgreSQL database
- A Discord application (for OAuth and the bot)
- A reverse proxy (Caddy recommended)

## Ports (default)

| Service | Default Port |
|---|---|
| Landing (`web`) | 3000 |
| Hub | 3003 |
| CMS | 3002 |
| Bot internal sync server | 3050 |

The marketplace app runs separately — see [internals/marketplace/architecture.md](../internals/marketplace/architecture.md).

## Marketplace (optional)

The Marketplace is a standalone Nuxt 4 app that runs separately from the core Guildora services. The Hub embeds it in an `<iframe>`. You can run it on the same server or independently.

See [internals/marketplace/](../internals/marketplace/index.md) for details.

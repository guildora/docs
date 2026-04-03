# Self-Hosting Guildora

Guildora is designed to be self-hosted. You run it on your own infrastructure and own all the data.

## What You Get

A full Guildora installation consists of three services:

| Service | Technology | Purpose |
|---|---|---|
| **Hub** (`apps/hub`) | Nuxt 4 + Nitro | The main web app — members, moderation, admin, APIs, landing page editor |
| **Landing** (`apps/web`) | Nuxt 4 | Public-facing landing page, OAuth redirect shim |
| **Bot** (`apps/bot`) | Node.js + Discord.js | Discord sync, voice tracking, slash commands |

All services share a single **PostgreSQL** database via a shared Drizzle schema package.

## What You Need

- A server or VPS with Docker and Docker Compose
- A domain with DNS configured for each service
- A PostgreSQL database
- A Discord application (for OAuth and the bot)
- A reverse proxy (Caddy recommended)

## Ports (default)

| Service | Local Dev | Docker |
|---|---|---|
| Landing (`web`) | 3000 | 3000 |
| Hub | 3003 | 3003 |
| Bot internal sync server | 3050 | 3050 |

## Marketplace Integration

Marketplace is a platform-managed service and **not part of the self-hosted Guildora stack**. It is not deployed from this hoster setup.

The Hub integrates Marketplace data via the Marketplace-provided API and can embed Marketplace UI via iframe when configured.

See [architecture-systems/marketplace/](../architecture-systems/marketplace/index.md) for integration details.

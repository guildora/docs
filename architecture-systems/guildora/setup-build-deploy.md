# Setup, Build, and Deployment

## Prerequisites

- Node.js 20 or newer
- pnpm 10
- PostgreSQL
- a Discord application for OAuth and the bot
- Docker and Docker Compose for containerized deployment

## Environment Variables

Start from repository root `.env.example`.

Key groups:

- hosts and URLs:
  - `APP_HOST` + `NUXT_PUBLIC_APP_URL` for landing (`web`)
  - `HUB_HOST` + `NUXT_PUBLIC_HUB_URL` for internal app (`hub`)
  - `NUXT_PUBLIC_MARKETPLACE_EMBED_URL` for the hub marketplace iframe target
- database: `DATABASE_URL`, optional `DATABASE_SSL`
- hub auth: `NUXT_SESSION_PASSWORD`, Discord OAuth vars, `SUPERADMIN_DISCORD_ID`, optional `NUXT_AUTH_DEV_BYPASS`
- bot bridge: `BOT_INTERNAL_URL`, `BOT_INTERNAL_TOKEN`, `BOT_INTERNAL_PORT`
- bot runtime: `DISCORD_BOT_TOKEN`, `DISCORD_CLIENT_ID`, `DISCORD_GUILD_ID`, optional `AFK_VOICE_CHANNEL_ID`
- optional UI runtime toggles: `NUXT_PUBLIC_DEFAULT_THEME`, `NUXT_PUBLIC_ENABLE_PERFORMANCE_DEBUG`

Important auth rule:

- `NUXT_OAUTH_DISCORD_REDIRECT_URI` must target hub, for example `https://hub.example.com/api/auth/discord`.
- If it is accidentally pointed at landing, landing can only forward the request to hub; it does not complete OAuth itself.

## Local Development

### Install and configure

```bash
pnpm install
cp .env.example .env
```

### Prepare database

```bash
pnpm db:migrate
pnpm db:seed
```

Generate a new migration when shared schema changes:

```bash
pnpm db:generate
```

### Start applications

```bash
pnpm dev
```

Default local ports:

- web (landing): `3000`
- hub (internal): `3003`
- bot internal sync server: configured by `BOT_INTERNAL_PORT`, default `3050`

### Useful focused runs

```bash
pnpm --filter @guildora/web dev
pnpm --filter @guildora/hub dev
pnpm --filter @guildora/hub test
pnpm --filter @guildora/bot dev
```

### Deploy slash commands when needed

```bash
pnpm bot:deploy-commands
```

## Build

- root build: `pnpm build`
- web production start: `node apps/web/.output/server/index.mjs`
- hub production start: `node apps/hub/.output/server/index.mjs`
- bot production start: `pnpm --filter @guildora/bot start`

## Docker Compose

Repository includes:

- `docker-compose.yml`
- `docker-compose.override.yml`

Services:

- `web`
- `hub`
- `bot`

Operational notes:

- web depends on hub for landing content (`/api/public/landing`) and login handoff
- hub depends on bot for Discord sync operations
- bot runs on the internal network and exposes sync server only to trusted callers
- compose assumes an external `caddy` network for reverse proxying

## Deployment Considerations

- run migrations before or during deploy in a controlled step
- seed only for initial setup or explicit role reconciliation
- keep `BOT_INTERNAL_TOKEN` and `NUXT_SESSION_PASSWORD` strong and private
- ensure bot has privileged intents for members and voice states

## Known Operational Caveats

- landing rendering depends on Hub public URL reachability from `web`
- Discord sync endpoints depend on bot internal server availability and authorization
- landing's auth shim is a fallback only; production OAuth should still point directly to hub

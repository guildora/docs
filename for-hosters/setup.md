# Setup

## Prerequisites

- Node.js 20 or newer
- pnpm 10
- PostgreSQL (15+ recommended)
- A Discord application (for OAuth and the bot)
- Docker and Docker Compose (for containerized deployment)

## Clone and Install

```bash
git clone https://github.com/guildora/guildora.git
cd guildora
pnpm install
cp .env.example .env
```

Edit `.env` with your values — see [configuration.md](./configuration.md) for details.

## Database Setup

```bash
# Run migrations (creates all tables)
pnpm db:migrate

# Seed initial data (roles, settings — only for first-time setup)
pnpm db:seed
```

When the shared schema changes (for platform developers):
```bash
pnpm db:generate  # generates a new migration file
pnpm db:migrate   # applies it
```

## Local Development

Start all services:
```bash
pnpm dev
```

Or start individual services:
```bash
pnpm --filter @guildora/web dev     # landing (port 3000)
pnpm --filter @guildora/hub dev     # hub (port 3003)
pnpm --filter @guildora/bot dev     # bot
```

## Deploy Bot Slash Commands

After initial setup or when bot commands change:
```bash
pnpm bot:deploy-commands
```

## Build for Production

```bash
pnpm build
```

Start individual services:
```bash
node apps/web/.output/server/index.mjs     # landing
node apps/hub/.output/server/index.mjs     # hub
pnpm --filter @guildora/bot start          # bot
```

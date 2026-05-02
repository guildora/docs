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

## Environment Variables

See [configuration.md](./configuration.md) for the full reference. Key Discord-related variables:

| Variable | Purpose | Example |
|---|---|---|
| `DISCORD_OAUTH_CLIENT_ID` | Discord OAuth app client ID | `123456789012345678` |
| `DISCORD_OAUTH_CLIENT_SECRET` | Discord OAuth app client secret | `secret...` |
| `DISCORD_OAUTH_REDIRECT_URI` | OAuth redirect URI | `https://hub.example.com/api/auth/discord` |
| `DISCORD_BOT_TOKEN` | Discord bot token | `Bot ...` |
| `DISCORD_GUILD_ID` | Target Discord guild ID | `987654321098765432` |
| `SUPERADMIN_DISCORD_ID` | Initial superadmin Discord user ID | `111222333444555666` |

> **Setup wizard pre-fill:** When `DISCORD_BOT_TOKEN`, `DISCORD_OAUTH_CLIENT_ID`, `DISCORD_OAUTH_CLIENT_SECRET`, and `DISCORD_GUILD_ID` are set in the Hub's runtime config, the setup wizard's "Connect a Platform" step pre-fills those fields automatically.

## Setup Wizard

After starting the hub (`pnpm dev`), open `http://localhost:3003/setup` (or `/de/setup` for German). Complete all steps:

1. **Community Info** — community name, description, default locale
2. **Admin Account** — admin display name, email
3. **Connect a Platform** — Discord credentials (pre-fills from env vars if set)
4. **Review** — confirm and complete

The wizard is fully internationalized (EN/DE). The language switcher in the top-right corner changes translations. Direct navigation to `/de/setup` renders in German without redirect.

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

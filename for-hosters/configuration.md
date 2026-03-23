# Configuration

All configuration is done via environment variables. Start from `.env.example` in the repository root.

## Key Variable Groups

### Hosts and URLs

| Variable | Service | Description |
|---|---|---|
| `APP_HOST` / `NUXT_PUBLIC_APP_URL` | Landing | Public URL of the landing app |
| `HUB_HOST` / `NUXT_PUBLIC_HUB_URL` | Hub | Public URL of the hub |
| `CMS_HOST` / `NUXT_PUBLIC_CMS_URL` / `PAYLOAD_PUBLIC_SERVER_URL` | CMS | Public URL of the CMS |
| `PAYLOAD_INTERNAL_URL` | Hub → CMS | Internal URL for hub-to-CMS SSO (server-side) |
| `NUXT_PUBLIC_MARKETPLACE_EMBED_URL` | Hub | URL of the marketplace app (embedded in iframe) |

### Database

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `DATABASE_SSL` | Optional: `true` to enable SSL |

### Auth (Hub)

| Variable | Description |
|---|---|
| `NUXT_SESSION_PASSWORD` | Session encryption secret (min 32 chars, keep private) |
| `NUXT_OAUTH_DISCORD_CLIENT_ID` | Discord OAuth client ID |
| `NUXT_OAUTH_DISCORD_CLIENT_SECRET` | Discord OAuth client secret |
| `NUXT_OAUTH_DISCORD_REDIRECT_URI` | Must point to hub: `https://hub.example.com/api/auth/discord` |
| `SUPERADMIN_DISCORD_ID` | Discord user ID of the platform superadmin |
| `NUXT_AUTH_DEV_BYPASS` | Optional: dev-only auth bypass (never use in production) |

> **Important:** `NUXT_OAUTH_DISCORD_REDIRECT_URI` must point to the hub, not the landing app. The landing only has a redirect shim.

### Bot Bridge

| Variable | Description |
|---|---|
| `BOT_INTERNAL_URL` | Internal URL of the bot sync server (hub calls this) |
| `BOT_INTERNAL_TOKEN` | Shared secret between hub and bot |
| `BOT_INTERNAL_PORT` | Port for bot sync server (default: 3050) |

### CMS

| Variable | Description |
|---|---|
| `PAYLOAD_SECRET` | Payload CMS admin secret |
| `CMS_SSO_SECRET` | Shared secret for hub→CMS SSO handoff |

### Bot Runtime

| Variable | Description |
|---|---|
| `DISCORD_BOT_TOKEN` | Discord bot token |
| `DISCORD_CLIENT_ID` | Discord application client ID |
| `DISCORD_GUILD_ID` | Discord server (guild) ID |
| `AFK_VOICE_CHANNEL_ID` | Optional: AFK channel ID to exclude from voice tracking |

### Optional UI

| Variable | Default | Description |
|---|---|---|
| `NUXT_PUBLIC_DEFAULT_THEME` | `guildora-dark` | Default theme for the hub |
| `NUXT_PUBLIC_ENABLE_PERFORMANCE_DEBUG` | — | Enable performance debug overlays |

## Discord Application Setup

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a new application
3. Under **OAuth2**: add your hub URL as redirect URI (`https://hub.example.com/api/auth/discord`)
4. Under **Bot**: enable **Server Members Intent** and **Voice States Intent**
5. Copy the bot token, client ID, and client secret into your `.env`

The bot and the hub share the same Discord application.

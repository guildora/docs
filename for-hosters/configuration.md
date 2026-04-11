# Configuration

Platform connections can be configured in two ways:

1. **Admin UI (recommended):** After first login, go to Settings > Platforms to connect Discord, Matrix, or both. Credentials are stored securely in the database.
2. **Environment variables (legacy):** Set variables in `.env`. These are used as a fallback when no database entry exists.

Start from `.env.example` in the repository root.

## Key Variable Groups

### Hosts and URLs

| Variable | Service | Description |
|---|---|---|
| `APP_HOST` / `NUXT_PUBLIC_APP_URL` | Landing | Public URL of the landing app |
| `HUB_HOST` / `NUXT_PUBLIC_HUB_URL` | Hub | Public URL of the hub |
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
| `NUXT_OAUTH_DISCORD_CLIENT_ID` | Discord OAuth client ID (can also be set via Admin UI) |
| `NUXT_OAUTH_DISCORD_CLIENT_SECRET` | Discord OAuth client secret (can also be set via Admin UI) |
| `NUXT_OAUTH_DISCORD_REDIRECT_URI` | Must point to hub: `https://hub.example.com/api/auth/discord` |
| `SUPERADMIN_DISCORD_ID` | Discord user ID of the platform superadmin |
| `NUXT_AUTH_DEV_BYPASS` | Optional: dev-only auth bypass (never use in production) |

> **Important:** `NUXT_OAUTH_DISCORD_REDIRECT_URI` must point to the hub, not the landing app. The landing only has a redirect shim.

### Bot Bridge (ENV fallback)

These variables are used when no `platform_connections` DB entry exists. Once you configure platforms via the Admin UI, these are ignored.

| Variable | Description |
|---|---|
| `BOT_INTERNAL_URL` | Internal URL of the Discord bot sync server (hub calls this) |
| `BOT_INTERNAL_TOKEN` | Shared secret between hub and bot |
| `BOT_INTERNAL_PORT` | Port for bot sync server (default: 3050) |

### Discord Bot Runtime (ENV fallback)

| Variable | Description |
|---|---|
| `DISCORD_BOT_TOKEN` | Discord bot token |
| `DISCORD_CLIENT_ID` | Discord application client ID |
| `DISCORD_GUILD_ID` | Discord server (guild) ID |
| `AFK_VOICE_CHANNEL_ID` | Optional: AFK channel ID to exclude from voice tracking |

### Matrix Bot Runtime (optional)

Only needed when connecting a Matrix platform. The Matrix bot is a separate service.

| Variable | Description |
|---|---|
| `MATRIX_HOMESERVER_URL` | Full URL of the Matrix homeserver (e.g. `https://matrix.example.org`) |
| `MATRIX_ACCESS_TOKEN` | Bot user's access token |
| `MATRIX_SPACE_ID` | Matrix space ID the community is bound to |
| `BOT_INTERNAL_PORT` | Port for Matrix bot sync server (default: 3051) |
| `BOT_INTERNAL_TOKEN` | Shared secret between hub and Matrix bot |

### Optional UI

| Variable | Default | Description |
|---|---|---|
| `NUXT_PUBLIC_DEFAULT_THEME` | `guildora-dark` | Default theme for the hub |
| `NUXT_PUBLIC_ENABLE_PERFORMANCE_DEBUG` | — | Enable performance debug overlays |

## Platform Setup

### Discord

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a new application
3. Under **OAuth2**: add your hub URL as redirect URI (`https://hub.example.com/api/auth/discord`)
4. Under **Bot**: enable **Server Members Intent** and **Voice States Intent**
5. Copy the bot token, client ID, and client secret
6. **Option A (recommended):** Enter credentials in Settings > Platforms > Connect Discord
7. **Option B (legacy):** Set the variables in your `.env` file

The bot and the hub share the same Discord application.

### Matrix

1. Set up or use an existing Matrix homeserver (Synapse, Dendrite, Conduit, or a public server like matrix.org)
2. Create a bot user on the homeserver and obtain an access token
3. Create a Matrix space for your community (or use an existing one)
4. Deploy the `matrix-bot` service (see Docker setup)
5. Enter credentials in Settings > Platforms > Connect Matrix

The Matrix bot can connect to **any** Matrix homeserver — you do not need to self-host one. However, self-hosting gives you full control over user management and federation.

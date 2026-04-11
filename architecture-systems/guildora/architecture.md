# Architecture

## Monorepo Topology

The repository is a pnpm workspace orchestrated with Turbo. It contains four deployable applications and three shared packages:

- `apps/web` (public landing)
- `apps/hub` (internal app + auth + API)
- `apps/bot` (Discord runtime)
- `apps/matrix-bot` (Matrix runtime — optional, only needed when Matrix is connected)
- `packages/shared` (schema, types, utilities)
- `packages/app-sdk` (app extension SDK types)
- `packages/mcp-server` (AI-driven landing page editing)

All services use TypeScript. Hub and bot import shared schema and types from `@guildora/shared`. Web is intentionally thinner and primarily consumes Hub API content plus public runtime config.

Guildora supports **multiple communication platforms** (Discord and Matrix). Platform connections are configured via the admin UI (Settings > Platforms) or fall back to `.env` variables for backward compatibility. Each platform has its own bot runtime that exposes the same internal HTTP API contract, allowing the Hub to communicate with any connected platform through the `platformBridge` routing layer.

## Runtime Components

### Web (Landing)

- Framework: Nuxt 4
- Responsibility: render the public landing page from DB-backed content served by Hub API
- Integration: fetches landing page data from Hub's `/api/public/landing` endpoint via `useLanding.ts` composable; renders sections with `LandingBlockRenderer` and block-type-specific components in `app/components/landing/blocks/`
- Compatibility behavior: exposes `/api/auth/discord` as a redirect shim to hub so a misconfigured OAuth callback can still recover

### Hub (Internal App)

- Framework: Nuxt 4 with Nitro
- API style: file-based handlers under `apps/hub/server/api`
- Authentication: `nuxt-auth-utils` session cookies + Discord OAuth
- Styling: Tailwind + internal Ui* component wrappers + custom CSS design system
- Localization: `@nuxtjs/i18n` plus a global locale middleware backed by DB and cookie context
- Layout model:
  - `auth` layout for login
  - default layout as the authenticated internal shell

### Discord Bot

- Library: `discord.js`
- Responsibilities:
  - voice session tracking
  - slash-command setup helper
  - internal HTTP bridge for guild role/member sync
  - manifest-based bot hook registration
  - base community-role bootstrapping

### Matrix Bot (Optional)

- Library: `matrix-bot-sdk`
- Only deployed when a Matrix platform connection is configured
- Responsibilities:
  - connects to any Synapse/Dendrite/Conduit homeserver via access token
  - internal HTTP bridge with the **same API contract** as the Discord bot (port 3051)
  - space hierarchy room listing, member lookup, power level management
  - room message and member join event handling
  - app hook emission with `platform: "matrix"`

### Shared Package (`packages/shared`)

- Drizzle schema definitions
- Postgres client and migrations
- Zod app-manifest parsing
- application flow types and linearization
- application token signing and verification
- profile-name helpers
- role and locale types

### App SDK (`packages/app-sdk`)

- TypeScript type definitions for the app extension system
- bot hook payload interfaces (all include optional `platform` field)
- `BotContext` interface (includes `platform: GuildoraPlatform`)
- `HubAppContext`, `BotClient`, and `AppDb` interfaces
- `GuildoraPlatform` type: `"discord" | "matrix"`

### MCP Server (`packages/mcp-server`)

- Model Context Protocol server for AI-driven landing page editing
- Communicates with Hub API using internal token authentication
- Enables programmatic creation and editing of landing page sections

## Database Ownership

The repository uses one PostgreSQL instance:

- `public` schema for app domain (managed by Drizzle)

## Key Data Flows

### Platform Connection Management

1. Admin configures platforms via **Settings > Platforms** in the Hub admin UI.
2. Platform credentials (Discord bot token/guild ID, or Matrix homeserver URL/access token/space ID) are stored in `platform_connections`.
3. Legacy `.env` configuration continues to work as a fallback — if `platform_connections` is empty and Discord env vars are set, Discord is treated as connected.
4. Each platform connection stores a `bot_internal_url` and `bot_internal_token` for the Hub to communicate with the platform's bot service.
5. The `platformBridge` utility routes Hub requests to the correct bot service based on the platform type.

### Authentication (Hub)

The login page dynamically shows login buttons based on configured platforms (via `GET /api/auth/platforms`).

**Discord Login:**

1. User opens hub login or is redirected into `/api/auth/discord`.
2. Discord OAuth returns identity data to hub.
3. Hub loads `membership_settings` to determine login mode.
4. **Application mode** (`applications_required = true`): requires exactly one mapped community role.
5. **Auto-login mode** (`applications_required = false`): checks guild membership, optional required Discord role, and assigns the configured default community role.
6. Hub upserts `users`, creates/updates `user_platform_accounts` entry, ensures `profiles`, updates `last_login_at`.
7. Hub stores a session with canonical `permissionRoles`.
8. Auto-login errors redirect to `/login?error=<code>` with localized messages.
9. Landing can forward accidental `/api/auth/discord` hits to hub through its redirect shim.

**Matrix Login:**

1. User clicks "Login with Matrix" on the login page.
2. Hub redirects to the Matrix homeserver's SSO endpoint (`/_matrix/client/v3/login/sso/redirect`).
3. User authenticates on the homeserver (Synapse SSO, OIDC, etc.).
4. Homeserver redirects back to `/api/auth/matrix?loginToken=...`.
5. Hub exchanges the login token for an access token, resolves the MXID and profile.
6. Hub upserts `users` and `user_platform_accounts`, creates session.
7. The temporary Matrix access token is immediately revoked (not stored).

**Account Linking:**

Users can link additional platform identities from their profile. The OAuth flow uses a `link=true` state parameter to associate the new identity with the existing session user instead of creating a new account.

### Landing Content (Web + Hub)

1. Admins edit landing sections in Hub at `/settings/landing` — sections are stored in DB tables (`landing_sections`, `landing_pages`, `landing_templates`).
2. Admins publish changes, which creates a version snapshot in `landing_page_versions`.
3. Web fetches the published landing page from Hub's `/api/public/landing` endpoint via the `useLanding.ts` composable.
4. Web renders the section layout with `LandingBlockRenderer` and block-type-specific components from `app/components/landing/blocks/`.
5. If Hub is unreachable, landing falls back to an informational CTA state that still links to hub login.

### Profile Updates (Hub + Bot)

1. Hub updates `users.display_name` from structured name parts.
2. `profiles.custom_fields` stores appearance and application metadata.
3. Locale preference is stored canonically in `profiles.locale_preference`.
4. Hub can push nickname and role updates via the bot bridge.

### Discord Mirror / Admin Import (Hub + Bot)

1. Admins map community roles to Discord roles.
2. Hub fetches guild members by mapped roles through the bot bridge.
3. Mirror layer upserts local users, profiles, role assignments, and snapshots.
4. Orphan cleanup removes local users no longer represented by mapped Discord roles.

### Scheduled Tasks (Nitro Plugins)

Hub uses `defineNitroPlugin` with `setInterval`/`setTimeout` for background tasks. Each plugin polls at a short interval (e.g. 5 minutes) and checks whether the configured execution interval has elapsed.

Current scheduled tasks:

- **Archive cleanup** (`archive-cleanup.ts`): daily, removes expired application tokens and archived applications.
- **App auto-updater** (`app-auto-updater.ts`): every 6 hours, updates sideloaded apps with auto-update enabled.
- **Membership auto-sync** (`membership-auto-sync.ts`): polls every 5 min, runs import cycle at the admin-configured interval.
- **Membership auto-cleanup** (`membership-auto-cleanup.ts`): polls every 5 min, evaluates cleanup conditions and removes matching users at the admin-configured interval.

All plugins hook into `nitroApp.hooks.hook("close", ...)` for clean shutdown.

## External Integrations

- Discord OAuth for hub login
- Discord Gateway and REST APIs via `discord.js`
- Matrix Client-Server API via `matrix-bot-sdk` (optional)
- Matrix SSO for hub login (optional)
- GitHub raw/blob URLs for app sideload manifests

## Architecture Constraints

- Hub is the source of truth for internal profiles, moderation, admin workflows, and operational `/api/*` behavior.
- Web is a public landing renderer consuming Hub API, with one OAuth redirect shim, not an internal workflow host.
- Discord Bot is the source of truth for live Discord guild state and manageable Discord role operations.
- Matrix Bot is the source of truth for live Matrix space state and power level management (when connected).
- New code must be **platform-agnostic** where possible — use `platformBridge` instead of direct `botSync` calls for new features.
- Platform-specific code (Discord embeds, Matrix power levels) should be isolated behind platform checks.
- Installed apps can contribute navigation, API routes, Vue pages, and bot hooks through their manifest and bundled code. Hook payloads include an optional `platform` field so apps can respond appropriately to events from different platforms.

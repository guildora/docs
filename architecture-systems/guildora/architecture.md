# Architecture

## Monorepo Topology

The repository is a pnpm workspace orchestrated with Turbo. It contains four deployable applications and two shared packages:

- `apps/web` (public landing)
- `apps/hub` (internal app + auth + API)
- `apps/cms` (Payload authoring)
- `apps/bot` (Discord runtime)
- `packages/shared` (schema, types, utilities)
- `packages/app-sdk` (app extension SDK types)

All services use TypeScript. Hub and bot import shared schema and types from `@guildora/shared`. Web is intentionally thinner and primarily consumes Hub API content plus public runtime config.

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

### CMS

- Framework: Payload CMS 3 on Next.js
- DB schema: `payload`
- Content model: pages, media, CMS users, site settings, reusable blocks
- Access model: CMS-local roles (`editor`, `moderator`, `admin`)
- Integration surface: public content API plus `/api/sso` for signed hub-to-CMS session bootstrap

### Discord Bot

- Library: `discord.js`
- Responsibilities:
  - voice session tracking
  - slash-command setup helper
  - internal HTTP bridge for guild role/member sync
  - manifest-based bot hook registration
  - base community-role bootstrapping

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
- bot hook payload interfaces
- `BotContext` and `HubAppContext` interfaces
- `BotClient` and `AppDb` interfaces

### MCP Server (`packages/mcp-server`)

- Model Context Protocol server for AI-driven landing page editing
- Communicates with Hub API using internal token authentication
- Enables programmatic creation and editing of landing page sections

## Database Ownership

The repository uses one PostgreSQL instance with two logical areas:

- `public` schema for app domain (managed by Drizzle)
- `payload` schema for CMS-owned tables (managed by Payload)

## Key Data Flows

### Discord Login (Hub)

1. User opens hub login or is redirected into `/api/auth/discord`.
2. Discord OAuth returns identity data to hub.
3. Hub loads `membership_settings` to determine login mode.
4. **Application mode** (`applications_required = true`): requires exactly one mapped community role.
5. **Auto-login mode** (`applications_required = false`): checks guild membership, optional required Discord role, and assigns the configured default community role.
6. Hub upserts `users` and ensures `profiles`, updates `last_login_at`.
7. Hub stores a session with canonical `permissionRoles`.
8. Auto-login errors redirect to `/login?error=<code>` with localized messages.
9. Landing can forward accidental `/api/auth/discord` hits to hub through its redirect shim.

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

### CMS Access (Hub + CMS)

1. Authenticated user opens `/cms` inside hub.
2. Hub validates app-role access plus moderator CMS policy.
3. Hub signs a short-lived SSO token.
4. CMS verifies the token, upserts a CMS user, logs the user into Payload, and redirects to `/admin`.
5. Hub embeds that URL in an iframe.

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
- GitHub raw/blob URLs for app sideload manifests
- Payload CMS SSO endpoint for embedded admin access

## Architecture Constraints

- Hub is the source of truth for internal profiles, moderation, admin workflows, and operational `/api/*` behavior.
- Web is a public landing renderer consuming Hub API, with one OAuth redirect shim, not an internal workflow host.
- Bot is the source of truth for live guild state and manageable Discord role operations.
- CMS content remains isolated from internal app-domain tables.
- Installed apps can contribute navigation, API routes, Vue pages, and bot hooks through their manifest and bundled code.

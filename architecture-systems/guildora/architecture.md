# Architecture

## Monorepo Topology

The repository is a pnpm workspace orchestrated with Turbo. It contains four deployable applications and two shared packages:

- `apps/web` (public landing)
- `apps/hub` (internal app + auth + API)
- `apps/cms` (Payload authoring)
- `apps/bot` (Discord runtime)
- `packages/shared` (schema, types, utilities)
- `packages/app-sdk` (app extension SDK types)

All services use TypeScript. Hub and bot import shared schema and types from `@guildora/shared`. Web is intentionally thinner and primarily consumes CMS HTTP content plus public runtime config.

## Runtime Components

### Web (Landing)

- Framework: Nuxt 4
- Responsibility: render the public landing page from CMS content
- Integration: fetches the published `landing` page from the CMS HTTP API with locale fallback to `en`
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

## Database Ownership

The repository uses one PostgreSQL instance with two logical areas:

- `public` schema for app domain (managed by Drizzle)
- `payload` schema for CMS-owned tables (managed by Payload)

## Key Data Flows

### Discord Login (Hub)

1. User opens hub login or is redirected into `/api/auth/discord`.
2. Discord OAuth returns identity data to hub.
3. Hub upserts `users` and ensures `profiles`.
4. Hub stores a session with canonical `permissionRoles`.
5. Landing can forward accidental `/api/auth/discord` hits to hub through its redirect shim.

### Landing Content (Web + CMS)

1. Editors publish page content in CMS.
2. Landing fetches `/api/pages` from CMS with slug `landing`, status `published`, current locale, and fallback locale `en`.
3. Landing renders the returned block layout with `CmsBlockRenderer`.
4. If CMS is unreachable, landing falls back to an informational CTA state that still links to hub login.

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

## External Integrations

- Discord OAuth for hub login
- Discord Gateway and REST APIs via `discord.js`
- GitHub raw/blob URLs for app sideload manifests
- Payload CMS HTTP API for landing reads
- Payload CMS SSO endpoint for embedded admin access

## Architecture Constraints

- Hub is the source of truth for internal profiles, moderation, admin workflows, and operational `/api/*` behavior.
- Web is a public landing renderer with one OAuth redirect shim, not an internal workflow host.
- Bot is the source of truth for live guild state and manageable Discord role operations.
- CMS content remains isolated from internal app-domain tables.
- Installed apps can contribute navigation, API routes, Vue pages, and bot hooks through their manifest and bundled code.

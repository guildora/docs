# Development Workflow

## Workspace Basics

- package manager: `pnpm`
- task runner: `turbo`
- Node target: `>=20`
- main workspace scripts: root `package.json`

## Common Commands

- `pnpm dev`
- `pnpm build`
- `pnpm lint`
- `pnpm typecheck`
- `pnpm db:generate`
- `pnpm db:migrate`
- `pnpm db:seed`
- `pnpm bot:deploy-commands`

App-specific scripts:

- `pnpm --filter @guildora/web dev`
- `pnpm --filter @guildora/hub dev`
- `pnpm --filter @guildora/hub test`
- `pnpm --filter @guildora/cms dev`
- `pnpm --filter @guildora/bot dev`

## Core Engineering Conventions

### Hub API

- Nitro handlers are file-based under `apps/hub/server/api`
- HTTP method is encoded in filename suffix: `.get`, `.post`, `.put`, `.delete`
- auth via `requireSession`, `requireModeratorSession`, `requireAdminSession`, or `requireRole`
- request parsing via Zod helpers or nearby shared parsing utilities

### Web Landing

- keep landing public and thin
- treat `apps/web/server/api/auth/discord.get.ts` as a compatibility redirect shim, not the real auth implementation
- landing content should be derived from CMS publication state, not duplicated locally

### Roles

- sessions should use canonical `permissionRoles`
- `roles` exists as a legacy alias only
- community roles map to one permission role and stay synchronized

### Profile Names

- `users.displayName` is a serialized composite field
- canonical delimiter: ` | `
- use `serializeProfileName`, `parseProfileName`, `coerceProfileNameFromRaw`

### UI

- internal UI work should prefer `apps/hub/app/components/ui/*`
- internal buttons must remain design-system-aligned
- styling-only changes must not alter behavior unless intentional

### Internationalization

- all new user-facing text uses i18n keys
- keep `de` and `en` synced in the owning app
- do not switch locale before `/api/profile/locale` succeeds

### Documentation

When routes, tables, roles, or workflows change, update `docs` and `ai` in the same change.

## Current Test Coverage

Automated tests are currently concentrated in hub utilities:

- auth helper behavior
- core navigation assembly
- dev role switcher helper logic
- Discord role snapshot helpers
- locale resolution and locale key parity
- JSON response helpers

There is currently no comparable automated suite for CMS, bot runtime, or end-to-end multi-service flows.

## Recommended Change Flow

1. Update schema or shared types if the domain changes.
2. Update hub Nitro handlers and server utils if backend behavior changes.
3. Update UI in hub and or landing as needed.
4. Re-check permissions, localization, and integration boundaries.
5. Update docs and AI manifests in the same change set.

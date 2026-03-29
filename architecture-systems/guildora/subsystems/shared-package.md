# Subsystem: Shared Package

## Purpose

`packages/shared` provides the shared contract between web, hub, CMS-adjacent tooling, and the Discord bot.

## Main Responsibilities

- Drizzle schema definitions for the app domain
- Postgres client creation
- migration and seed entrypoints
- app-manifest schema and parser
- shared locale, role, user, and profile types
- application flow types and linearization utilities
- application token generation and verification
- profile-name normalization helpers

## Important Files

- `src/db/schema.ts`
- `src/db/client.ts`
- `src/db/migrate.ts`
- `src/db/seed.ts`
- `src/types/app-manifest.ts`
- `src/types/application-flow.ts`
- `src/types/locale.ts`
- `src/types/profile.ts`
- `src/types/roles.ts`
- `src/types/user.ts`
- `src/utils/application-tokens.ts`
- `src/utils/flow-linearize.ts`
- `src/utils/profile-name.ts`

## Shared Domain Guarantees

- permission roles and community roles come from one schema definition
- app manifests are parsed and normalized in one place
- display names use one serialization and parsing strategy
- locale preference and community-default locale types are shared

## App Manifest Model

The manifest schema supports:

- metadata and compatibility info
- permissions
- rail and panel navigation
- declared pages and API routes
- bot hooks
- config fields
- required env vars
- install notes and migrations

Normalization behavior:

- a legacy `railEntry` can be converted into `navigation.rail`
- legacy `panelEntries` can be converted into one normalized `panelGroups` entry

## Seeded Defaults

`db:seed` ensures default permission roles and default community roles:

- `temporaer`
- `user`
- `moderator`
- `admin`
- `superadmin`

Community roles:

- `Bewerber`
- `Anwaerter`
- `Mitglied`

## Related Package: `packages/app-sdk`

`packages/app-sdk` is a separate shared package providing TypeScript type definitions for the app extension system. It exports:

- bot hook payload interfaces (`VoiceActivityPayload`, `RoleChangePayload`, `MemberJoinPayload`, `InteractionPayload`, `MessagePayload`)
- `BotContext` and `HubAppContext` interfaces
- `BotClient` interface for Discord operations
- `AppDb` interface for scoped key-value storage

It is consumed by `apps/bot` for hook dispatch and by app developers for type-safe bot integration.

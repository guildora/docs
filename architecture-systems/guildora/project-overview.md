# Project Overview

## Purpose

Guildora is a Discord-centered community platform in a pnpm monorepo. It combines a public landing page, an internal member hub, a Payload CMS, a Discord bot, and a shared PostgreSQL-backed domain model.

Discord is both:

- the login provider for the internal hub application
- the source of guild member, role, and voice-state data

The hub is the operational center for members, moderation, admin tooling, and internal APIs. The landing app is public-facing. CMS covers authoring and publishing. The bot keeps Discord state and app state connected.

## Primary Audiences

- visitors consuming the public landing page
- members who log in via Discord and use the internal community area
- moderators who review applications, edit member data, and manage role assignments
- admins and superadmins who manage theme settings, community settings, Discord role mappings, installed apps, CMS access, and mirror flows
- CMS editors, moderators, and admins managing public content
- AI agents and developers who need a grounded model of the repository

## Main Use Cases

- public landing-page rendering from CMS content
- public login CTA that sends users into the hub login flow
- Discord OAuth login into the internal hub
- member profile editing with structured display-name handling
- visual flow-based community applications with token-gated public forms, moderator review, and automated Discord role assignment on approval
- member directory search and voice-activity overview
- self-service Discord role assignment from an admin-curated allowlist
- admin synchronization between Discord guild membership and the internal database
- configurable membership mode: application-based or open auto-login for Discord guild members
- automatic periodic user sync from Discord (auto-sync)
- automatic cleanup of users who no longer meet configurable conditions (orphans, missing roles, inactivity)
- optional extension through installed app manifests and app navigation entries
- embedded CMS admin access through hub-initiated SSO

## Scope Boundaries

Included in this repository:

- public landing UI (`apps/web`)
- internal hub UI plus Nitro APIs (`apps/hub`)
- CMS authoring and publishing (`apps/cms`)
- Discord bot runtime and internal sync bridge (`apps/bot`)
- shared PostgreSQL schema and cross-service types (`packages/shared`)
- app extension SDK types (`packages/app-sdk`)

Current product boundaries:

- `apps/web` is public and does not own member, moderator, or admin workflows
- `apps/web` does expose one compatibility Nitro route at `/api/auth/discord` that redirects to hub if OAuth is misrouted
- `apps/hub` owns the real Discord OAuth callback, session lifecycle, internal pages, and application APIs
- marketplace pages in the hub are embed or redirect shells, not a local end-to-end marketplace product
- there is no in-repo MCP server; AI metadata lives in `ai/`

## Project Vocabulary

- `permission roles`: technical access roles stored in `permission_roles` and carried in sessions as `permissionRoles`
- `community roles`: domain roles such as `Bewerber`, `Anwaerter`, `Mitglied`, mapped to one permission role
- `application flows`: visual node-based application form definitions managed by staff
- `applications`: individual submitted applications tied to a flow
- `submissions`: marketplace app submissions; table exists but local review flow is not active
- `installed apps`: app manifests stored in `installed_apps`, used for internal navigation and bot hook registration
- `CMS SSO`: short-lived handoff from hub to Payload admin via signed token

## Repository Layout

```text
apps/
  web/       Nuxt 4 public landing app plus OAuth redirect shim
  hub/       Nuxt 4 internal app + Nitro API + Discord OAuth
  cms/       Payload CMS 3 + Next.js content system
  bot/       Discord bot and internal sync server
packages/
  shared/    Drizzle schema, DB client, shared types and helpers
  app-sdk/   App extension SDK types for bot hooks and contexts
docs/        Canonical project documentation
ai/          AI-oriented skill and manifest layer
```

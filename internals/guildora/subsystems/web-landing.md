# Subsystem: Web Landing

## Purpose

`apps/web` is the public landing application.

It serves:

- CMS-backed landing page rendering
- public login CTA linking to hub login
- localized public copy (`en` and `de`)
- a compatibility `/api/auth/discord` redirect shim to hub

## Important Directories

- `app/pages`
- `app/layouts`
- `app/components/landing`
- `app/components/layout`
- `app/composables`
- `server/api`
- `i18n/locales`

## Runtime Patterns

- landing content is fetched from CMS HTTP API via `usePayload`
- landing reads the `landing` slug with `published` status and locale fallback to `en`
- landing blocks are rendered by `CmsBlockRenderer`
- if CMS is unavailable, landing falls back to an informational state instead of failing hard
- `/api/auth/discord` simply forwards the request to hub with query parameters preserved

## Boundaries

- does not host internal member, moderator, or admin flows
- does not own the real auth callback implementation
- depends on CMS content publication state
- depends on hub URL for login handoff

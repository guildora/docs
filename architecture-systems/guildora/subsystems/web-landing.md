# Subsystem: Web Landing

## Purpose

`apps/web` is the public landing application.

It serves:

- DB-backed landing page rendering via Hub API
- public login CTA linking to hub login
- localized public copy (`en` and `de`)
- a compatibility `/api/auth/discord` redirect shim to hub

## Important Directories

- `app/pages`
- `app/layouts`
- `app/components/landing` — layout wrappers
- `app/components/landing/blocks` — block-type-specific render components
- `app/components/layout`
- `app/composables` — includes `useLanding.ts`
- `server/api`
- `i18n/locales`

## Runtime Patterns

- landing content is fetched from Hub's `/api/public/landing` endpoint via the `useLanding` composable
- sections are rendered by `LandingBlockRenderer` which delegates to block-type-specific components in `app/components/landing/blocks/`
- landing content is managed by admins in Hub at `/settings/landing` and stored in DB tables (`landing_sections`, `landing_pages`, `landing_templates`)
- if Hub is unreachable, landing falls back to an informational state instead of failing hard
- `/api/auth/discord` simply forwards the request to hub with query parameters preserved

## Boundaries

- does not host internal member, moderator, or admin flows
- does not own the real auth callback implementation
- depends on Hub API for published landing content
- depends on hub URL for login handoff

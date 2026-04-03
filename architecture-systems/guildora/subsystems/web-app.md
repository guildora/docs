# Subsystem: Hub App

## Purpose

`apps/hub` is the internal operational application. It serves:

- Discord login flow
- authenticated member pages
- moderator workflows
- admin workflows
- Nitro APIs under `/api/*`
- landing page editor and footer page management (`/settings/landing`)
- marketplace iframe hosting for configured external marketplace URLs

## Important Directories

- `app/pages`
- `app/layouts`
- `app/middleware`
- `app/components`
- `app/components/ui`
- `app/composables`
- `server/api`
- `server/utils`

## Runtime Patterns

- sessions via `nuxt-auth-utils`
- login page uses the `auth` layout
- authenticated pages use the default layout as the internal shell
- middleware handles UX route gating
- server routes are the real authorization boundary
- internal navigation loads from `/api/apps/navigation`
- branding loads from `/api/internal/branding`
- locale routing is enforced by `locale.global` using cookie plus DB context

## Business Rules

- profile display names are stored centrally on `users`
- language changes persist before UI locale switch
- self-assignable Discord roles are limited to the admin allowlist
- settings pages use the `settings` middleware which allows admins and moderators with moderation rights

## Note

Public landing rendering is not part of hub and is served by `apps/web`.

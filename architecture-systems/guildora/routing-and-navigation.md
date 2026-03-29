# Routing and Navigation

## App Ownership

- `apps/web` owns public landing rendering.
- `apps/web` also owns one compatibility API route: `/api/auth/discord`, which forwards to hub.
- `apps/hub` owns login, authenticated routes, role-gated routes, and the operational API surface.
- `apps/hub` also accepts localized OAuth compatibility callbacks at `/:locale/api/auth/discord` (`en`, `de`) and forwards them to `/api/auth/discord`.

## Public Routes (Web)

- `/` renders the CMS-managed landing page.
- localized routing follows `prefix_except_default`:
  - English remains unprefixed
  - German uses `/de`

## Landing Runtime Notes

- landing fetches the `landing` page from Payload via `usePayload`
- if CMS content is unavailable, landing shows a fallback info state and a login CTA
- landing login CTA points to hub `/login?returnTo=/dashboard`

## Internal Routes (Hub)

### Public Hub Routes

- `/` redirects to `/dashboard`
- `/login` is the user-facing Discord login page

### Public Hub Routes (token-gated)

- `/apply/[flowId]/[token]` — public application form (validated via single-use token, uses `apply` layout)

### Authenticated Hub Routes

- `/dashboard`
- `/members`
- `/members/[id]` redirects to `/members?member=:id` (opens member details modal)
- `/profile`
- `/profile/customize` — profile customization (appearance, locale)
- `/profile/[id]` redirects to `/members?member=:id`
- `/apps/[appId]/[...slug]` — app-provided pages (ssr disabled)
- `/cms` — embedded CMS session bootstrap

### Moderator Hub Routes

- `/apps` — installed app listing
- `/apps/overview` — app overview dashboard
- `/applications` — application management dashboard
- `/applications/open` — open/pending applications list
- `/applications/open/[applicationId]` — view and review individual application
- `/applications/archive` — archived applications
- `/applications/archive/[applicationId]` — view archived application
- `/applications/flows` — application flow list
- `/applications/flows/new` — create new application flow
- `/applications/flows/[flowId]` — visual flow editor (Vue Flow)
- `/applications/flows/[flowId]/settings` — flow settings

### Admin Hub Routes (via Settings)

All settings routes use the `settings` middleware, which allows admins, superadmins, and moderators with at least one moderation right.

- `/settings` — settings index
- `/settings/design` — theme and design settings
- `/settings/permissions` — permission roles and community role management
- `/settings/community` — community name, locale, display name template
- `/settings/custom-fields` — custom profile field management
- `/settings/files` — file storage management
- `/settings/moderation-rights` — granular moderation rights configuration
- `/settings/apps` — app management
- `/settings/apps/review` — app submission review (redirect)
- `/settings/dev-role-switcher` — development role switcher

### Special Admin Routes

- `/apps/sideload` — local app sideloading (superadmin only)
- `/applications/config` — application module settings (admin only)

### Development Routes

- `/dev` — development dashboard
- `/dev/reset` — reset tools
- `/dev/role-switcher` — switch roles for testing

## Hub Layouts

- `auth`: public login shell with top navigation/footer
- `apply`: public application form shell
- default layout: authenticated internal shell with sidebar navigation, branding, and mobile drawer

## Hub Middleware

- `auth`: requires an active session
- `moderator`: requires `moderator`, `admin`, or `superadmin`
- `admin`: requires `admin` or `superadmin`
- `superadmin`: requires `superadmin`
- `settings`: requires `admin` or `superadmin`, or `moderator` with at least one moderation right
- `dev`: requires development mode
- `locale.global`: applies DB- and cookie-backed locale routing for hub pages
- `mandatory-fields.global`: enforces required custom field completion

## Internal Navigation Model (Hub)

Sidebar navigation merges:

1. core navigation in `apps/hub/server/utils/core-navigation.ts`
2. active installed app manifests in `apps/hub/server/utils/apps.ts`

Merged result is served by `GET /api/apps/navigation` and rendered by the default hub layout.

## Locale Handling

- project uses `prefix_except_default`
- English routes are unprefixed
- German routes use `/de/...`
- landing locale handling is managed in `apps/web`
- hub locale handling is managed in `apps/hub/app/middleware/locale.global.ts`
- hub locale preference can come from profile, community default, cookie, or current path context depending on session state

## Notes

- `/cms` is an embedded CMS session bootstrap, not a local editor.
- `/apply/:flowId/:token` is the only public hub route — all other hub routes require authentication.
- landing rendering is separate from hub and stays public.

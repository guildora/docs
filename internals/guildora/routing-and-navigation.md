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

### Authenticated Hub Routes

- `/dashboard`
- `/members`
- `/members/[id]`
- `/profile`
- `/profile/name`
- `/profile/roles`
- `/profile/design`
- `/profile/[id]` redirects to `/members/[id]`
- `/marketplace`
- `/marketplace/submit` redirects to `/marketplace`
- `/marketplace/[appId]` redirects to `/marketplace`
- `/cms`

### Moderator Hub Routes

- `/mod`

### Admin Hub Routes

- `/admin`
- `/admin/design`
- `/admin/theme` redirects to `/admin/design`
- `/admin/permissions`
- `/admin/discord-roles`
- `/admin/apps`
- `/admin/apps/review` redirects to `/admin/apps`
- `/admin/dev-role-switcher`
- `/admin/users` redirects to `/admin/dev-role-switcher`

## Hub Layouts

- `auth`: public login shell with top navigation/footer
- default layout: authenticated internal shell with sidebar navigation, branding, and mobile drawer

## Hub Middleware

- `auth`: requires an active session
- `moderator`: requires `moderator`, `admin`, or `superadmin`
- `admin`: requires `admin` or `superadmin`
- `locale.global`: applies DB- and cookie-backed locale routing for hub pages

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
- `/marketplace` is an iframe host controlled by `NUXT_PUBLIC_MARKETPLACE_EMBED_URL`.
- landing rendering is separate from hub and stays public.

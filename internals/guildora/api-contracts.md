# API Contracts

This document summarizes the current HTTP surface owned by the Nuxt apps.

Operational application APIs live under `apps/hub/server/api`.
`apps/web` only exposes one compatibility Nitro route for OAuth redirect recovery.

Auth shorthand used below:

- `public`: no session required
- `session`: any authenticated session, including `temporaer`
- `staff`: `moderator`, `admin`, or `superadmin`
- `admin`: `admin` or `superadmin`
- `superadmin`: `superadmin` only

See also [`permissions-matrix.md`](./permissions-matrix.md).

## App Ownership Summary

- `apps/hub` owns the real Discord OAuth callback, logout, internal resources, member APIs, moderator APIs, admin APIs, and CMS SSO bootstrap.
- `apps/web` owns one compatibility route at `/api/auth/discord` that redirects the request to hub.

## Auth

### Hub

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/auth/discord` | public | Discord OAuth start/callback and session creation |
| GET | `/:locale/api/auth/discord` (`en`,`de`) | public | compatibility redirect to `/api/auth/discord` preserving query params |
| POST | `/api/auth/logout` | public | clear the current session |

### Web Compatibility Shim

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/auth/discord` | public | forward misrouted OAuth requests to hub with original query params preserved |

## Frontend Support and Theme

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/theme` | public | expose public theme colors for the internal app |
| GET | `/api/internal/locale-context` | public | resolve effective locale, source, and session state |
| GET | `/api/internal/branding` | session | load internal sidebar branding and logo settings |

## Profile, Members, and Dashboard

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/profile` | session; foreign lookup is staff-only | load current profile or a specific user profile with `?id=` |
| PUT | `/api/profile` | session | update own structured profile name, appearance, locale, and custom fields |
| PUT | `/api/profile/locale` | session | atomically update the user's locale preference |
| PUT | `/api/profile/discord-roles` | session | sync self-service Discord role selection against the admin allowlist |
| GET | `/api/members` | session | list members with filters, sort, and recent voice summary |
| GET | `/api/dashboard/stats` | session | dashboard charts, summaries, and profile-change feed |

## Apps and Navigation

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/apps` | session | list installed apps visible to the current user context |
| GET | `/api/apps/navigation` | session | return merged core and app-provided sidebar navigation |
| POST | `/api/apps/:appId/activate` | admin | set installed app status to `active` |
| POST | `/api/apps/:appId/deactivate` | admin | set installed app status to `inactive` |

## Development Role Switching

These routes are intended for development or explicit debug modes.

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/dev/users` | staff or switched debug session | list switchable users |
| POST | `/api/dev/switch-user` | staff or switched debug session | replace the active session user |
| POST | `/api/dev/restore-user` | switched debug session | restore the original session user |

## CMS Session Bootstrap

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/cms/session-url` | session plus CMS access check | create a short-lived signed SSO URL for embedded Payload admin |

## Moderator Area

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/mod/community-roles` | staff | list community roles and permission roles |
| POST | `/api/mod/community-roles` | admin | create a community role without Discord mapping |
| PUT | `/api/mod/community-roles/:id` | admin | update a community role without Discord mapping |
| DELETE | `/api/mod/community-roles/:id` | admin | delete a community role without Discord mapping |
| GET | `/api/mod/users` | staff | list users for moderation |
| PUT | `/api/mod/users/:id/profile` | staff | update another user's structured name |
| PUT | `/api/mod/users/:id/community-role` | staff | set another user's community role |
| POST | `/api/mod/users/:id/sync` | staff | trigger Discord sync for a user |
| GET | `/api/mod/applications` | staff | list open community applications |
| POST | `/api/mod/applications/:id/approve` | staff | approve an application and promote the user |
| POST | `/api/mod/applications/:id/reject` | staff | reject an application |

## Admin: Theme, Community, and App Management

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/theme` | admin | read admin theme settings including logo |
| PUT | `/api/admin/theme` | admin | update theme colors, content tones, logo, and sidebar logo size |
| GET | `/api/admin/users` | admin | list users for admin tooling |
| GET | `/api/admin/apps` | admin | list installed apps with admin stats |
| POST | `/api/admin/apps/sideload` | admin | download and store an app manifest from GitHub |
| PUT | `/api/admin/apps/:appId/config` | admin | store app configuration JSON |
| PUT | `/api/admin/apps/:appId/status` | admin | set app status |
| DELETE | `/api/admin/apps/:id` | admin | delete an installed app row |
| GET | `/api/admin/community-settings` | admin | load community name and default locale |
| PUT | `/api/admin/community-settings` | admin | update community name and default locale |
| PUT | `/api/admin/cms-access` | admin | update whether moderators can open `/cms` |
| GET | `/api/admin/permissions` | admin | load permission-role, community-role, and CMS access metadata |
| POST | `/api/admin/community-roles` | admin | create a community role including optional Discord role mapping |
| PUT | `/api/admin/community-roles/:id` | admin | update a mapped community role |
| DELETE | `/api/admin/community-roles/:id` | admin | delete a mapped community role |

## Admin: Discord Role and Mirror Operations

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/discord-roles` | admin | list guild roles and currently selectable self-service role IDs |
| PUT | `/api/admin/discord-roles` | admin | replace the selectable self-service role allowlist |
| POST | `/api/admin/discord-roles/self-import` | superadmin | import the current superadmin's own Discord roles into the local snapshot state |
| GET | `/api/admin/discord/roles` | admin | fetch guild roles from the bot bridge |
| GET | `/api/admin/discord/roles/:roleId/members` | admin | fetch guild members for one Discord role |
| GET | `/api/admin/discord/members/:discordId` | admin | fetch one guild member from the bot bridge |
| POST | `/api/admin/discord/members/:discordId/remove-roles` | admin | remove specific or all manageable Discord roles |
| POST | `/api/admin/users/import` | admin | mirror mapped Discord members into local users |
| POST | `/api/admin/users/delete-orphaned` | admin | delete selected local users no longer present in mapped Discord roles |
| DELETE | `/api/admin/users/:id` | admin | remove or reconcile one user after Discord role removal |
| DELETE | `/api/admin/users/by-community-role/:communityRoleId` | admin | bulk remove or reconcile users belonging to one community role |
| POST | `/api/admin/dev/reset-mirror` | admin | reset admin mirror state used by the dev tooling |

## Notes on Current Product Boundaries

- Marketplace pages in the hub app are currently an iframe shell or redirects, not a local feature-complete marketplace product.
- The schema still contains `app_marketplace_submissions`, and app-submission checks still exist in utilities, but there is no active local submissions API in hub.
- Landing is public and intentionally thin; if OAuth requests hit landing by mistake, the web shim forwards them to hub instead of handling auth locally.

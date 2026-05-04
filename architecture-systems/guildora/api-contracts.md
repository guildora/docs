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

- `apps/hub` owns the real Discord OAuth callback, logout, internal resources, member APIs, moderator APIs, and admin APIs.
- `apps/web` owns one compatibility route at `/api/auth/discord` that redirects the request to hub.

## Auth

### Hub

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/auth/discord` | public | Discord OAuth start/callback and session creation |
| GET | `/:locale/api/auth/discord` (`en`,`de`) | public | compatibility redirect to `/api/auth/discord` preserving query params |
| GET | `/api/auth/matrix` | public | Matrix SSO login (redirect to homeserver, callback with loginToken) |
| GET | `/api/auth/platforms` | public | Get available auth platforms for login page |
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
| GET | `/api/profile` | session | load current user profile |
| PUT | `/api/profile` | session | update own structured profile name, appearance, locale, and custom fields |
| PUT | `/api/profile/locale` | session | atomically update the user's locale preference |
| PUT | `/api/profile/avatar` | session | upload a new avatar image |
| DELETE | `/api/profile/avatar` | session | remove the current avatar |
| GET | `/api/profile/custom-fields` | session | load own custom field values |
| PUT | `/api/profile/custom-fields` | session | update own custom field values |
| PUT | `/api/profile/discord-roles` | session | sync self-service Discord role selection against the admin allowlist |
| GET | `/api/members` | session | list members with filters, sort, voice summary, card role fallback, and avatar URL for member cards |
| GET | `/api/dashboard/stats` | session | dashboard charts, summaries, and profile-change feed |

## Community Settings

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/community-settings/display-name-template` | session | get the display name template fields configured for this community |

## Apps and Navigation

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/apps` | session | list installed apps visible to the current user context |
| GET | `/api/apps/navigation` | session | return merged core and app-provided sidebar navigation |
| GET | `/api/apps/:appId/_messages` | session | get app locale messages for dynamic page rendering |
| GET | `/api/apps/:appId/_source` | session | get app source code bundle |
| GET | `/api/apps/:appId/_page-source` | session | get app page SFC source |
| POST | `/api/apps/:appId/activate` | admin | set installed app status to `active` |
| POST | `/api/apps/:appId/deactivate` | admin | set installed app status to `inactive` |

## Development Role Switching

These routes are intended for development or explicit debug modes.

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/dev/users` | staff or switched debug session | list switchable users |
| POST | `/api/dev/switch-user` | staff or switched debug session | replace the active session user |
| POST | `/api/dev/restore-user` | switched debug session | restore the original session user |

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
| GET | `/api/mod/users/:id/custom-fields` | staff | get a user's custom field values |
| PUT | `/api/mod/users/:id/custom-fields` | staff | update a user's custom field values |
| POST | `/api/mod/users/:id/sync` | staff | trigger Discord sync for a user |
| POST | `/api/mod/users/batch-community-role` | staff | batch assign a community role to multiple users |
| POST | `/api/mod/users/batch-discord-roles` | staff | batch add or remove Discord roles for multiple users |
| GET | `/api/mod/tags` | staff | list community tags |
| POST | `/api/mod/tags` | staff | create a community tag |
| GET | `/api/mod/discord-roles` | staff | list Discord guild roles for bulk operations |

## Application Flows and Submissions

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/applications/flows` | staff | list all application flows |
| POST | `/api/applications/flows` | staff | create a new application flow |
| GET | `/api/applications/flows/:flowId` | staff | get flow details with graph and settings |
| PUT | `/api/applications/flows/:flowId` | staff | update flow graph and settings |
| DELETE | `/api/applications/flows/:flowId` | staff | delete an application flow |
| POST | `/api/applications/flows/:flowId/duplicate` | staff | duplicate an existing flow |
| GET | `/api/applications/open` | staff | list open/pending applications |
| GET | `/api/applications/:applicationId` | staff | get individual application details |
| POST | `/api/applications/:applicationId/approve` | staff | approve application and assign roles |
| POST | `/api/applications/:applicationId/reject` | staff | reject an application |
| POST | `/api/applications/:applicationId/retry-roles` | staff | retry failed role assignments |
| GET | `/api/applications/archive` | staff | list archived (completed) applications |
| POST | `/api/applications/archive/cleanup` | admin | clean up old archived applications |
| GET | `/api/applications/config` | admin | get application module configuration |
| PUT | `/api/applications/config` | admin | update application module configuration |
| GET | `/api/applications/moderator-notifications` | staff | get notification subscriptions for flows |
| PUT | `/api/applications/moderator-notifications` | staff | update notification subscriptions |

## Public Application Form

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| POST | `/api/apply/:flowId/validate-token` | public | validate a single-use application token |
| POST | `/api/apply/:flowId/submit` | public | submit completed application form |
| POST | `/api/apply/:flowId/upload` | public | upload file attachment for application |

## Admin: Theme, Community, and App Management

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/theme` | admin | read admin theme settings including logo |
| PUT | `/api/admin/theme` | admin | update theme colors, content tones, logo, and sidebar logo size |
| GET | `/api/admin/users` | admin | list users for admin tooling |
| GET | `/api/admin/apps` | admin | list installed apps with admin stats |
| POST | `/api/admin/apps/sideload` | admin | download and store an app manifest from GitHub |
| POST | `/api/admin/apps/local-sideload` | admin | sideload an app from local file system |
| GET | `/api/admin/apps/update-check` | admin | check for available app updates |
| PUT | `/api/admin/apps/:appId/config` | admin | store app configuration JSON |
| PUT | `/api/admin/apps/:appId/status` | admin | set app status |
| PUT | `/api/admin/apps/:appId/auto-update` | admin | toggle auto-update for an app |
| POST | `/api/admin/apps/:appId/update` | admin | manually trigger app update |
| DELETE | `/api/admin/apps/:id` | admin | delete an installed app row |
| GET | `/api/admin/community-settings` | admin | load community name and default locale |
| PUT | `/api/admin/community-settings` | admin | update community name and default locale |
| GET | `/api/admin/permissions` | admin | load permission-role, community-role, and moderation settings metadata |
| POST | `/api/admin/community-roles` | admin | create a community role including optional Discord role mapping |
| PUT | `/api/admin/community-roles/:id` | admin | update a mapped community role |
| DELETE | `/api/admin/community-roles/:id` | admin | delete a mapped community role |

## Admin: Custom Fields, Tags, and Moderation Rights

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/custom-fields` | admin | list all custom profile fields |
| POST | `/api/admin/custom-fields` | admin | create a custom profile field |
| PUT | `/api/admin/custom-fields/:id` | admin | update a custom profile field |
| DELETE | `/api/admin/custom-fields/:id` | admin | delete a custom profile field |
| GET | `/api/admin/moderation-rights` | admin | get granular moderation rights settings |
| PUT | `/api/admin/moderation-rights` | admin | update granular moderation rights settings |
| DELETE | `/api/admin/tags/:id` | admin | delete a community tag |
| POST | `/api/admin/users/batch-delete` | admin | batch delete multiple users |

## Admin: Landing Page

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/landing/page` | admin | load landing page settings |
| PUT | `/api/admin/landing/page` | admin | update landing page settings (template, custom CSS, meta) |
| GET | `/api/admin/landing/templates` | admin | list available landing templates |
| PUT | `/api/admin/landing/template` | admin | set active landing template |
| GET | `/api/admin/landing/sections` | admin | list all landing sections |
| POST | `/api/admin/landing/sections` | admin | create a new landing section |
| PUT | `/api/admin/landing/sections/:id` | admin | update a landing section |
| DELETE | `/api/admin/landing/sections/:id` | admin | delete a landing section |
| PUT | `/api/admin/landing/sections/reorder` | admin | reorder landing sections |
| GET | `/api/admin/landing/blocks` | admin | list available block types |
| POST | `/api/admin/landing/publish` | admin | publish landing page (creates version snapshot) |
| POST | `/api/admin/landing/unpublish` | admin | unpublish published sections |
| POST | `/api/admin/landing/reset` | admin | reset landing page to template defaults (snapshots current state first) |
| GET | `/api/admin/landing/versions` | admin | list landing page version history |
| GET | `/api/admin/landing/versions/:id` | admin | get a specific version snapshot |
| POST | `/api/admin/landing/versions/:id/restore` | admin | restore a specific version |
| PUT | `/api/admin/landing-access` | admin | update landing page access settings |
| GET | `/api/admin/landing/footer-pages` | admin | list footer pages |
| POST | `/api/admin/landing/footer-pages` | admin | create a footer page |
| PUT | `/api/admin/landing/footer-pages/:id` | admin | update a footer page |
| DELETE | `/api/admin/landing/footer-pages/:id` | admin | delete a footer page |
| PUT | `/api/admin/landing/footer-pages/reorder` | admin | reorder footer pages |

## Admin: Role Groups

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/role-groups` | admin | list all role groups |
| POST | `/api/admin/role-groups` | admin | create a role group |
| GET | `/api/admin/role-groups/:id` | admin | get role group with selectable roles |
| PUT | `/api/admin/role-groups/:id` | admin | update a role group |
| DELETE | `/api/admin/role-groups/:id` | admin | delete a role group |
| PUT | `/api/admin/role-groups/:id/roles` | admin | update selectable roles in a group |
| POST | `/api/admin/role-groups/:id/embed/deploy` | admin | deploy role-picker embed to Discord |
| DELETE | `/api/admin/role-groups/:id/embed` | admin | delete role-picker embed |

## Admin: Discord Role and Mirror Operations

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/discord-channels` | admin | list guild channels from the bot bridge |
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

## Admin: Membership Settings and Cleanup

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/membership-settings` | admin | load membership settings, community roles, and guild roles for config UI |
| PUT | `/api/admin/membership-settings` | admin | update membership settings (applications toggle, auto-sync, auto-cleanup, conditions) |
| GET | `/api/admin/cleanup-log` | admin | paginated audit log of auto-cleanup actions (query: `limit`, `offset`) |

### Modified Auth Behavior

When `membership_settings.applications_required = false`:

- `GET /api/auth/discord` auto-creates users on login if they are guild members
- If `required_login_role_id` is set, the user must have that Discord role
- New users receive the configured `default_community_role_id`
- Auth errors redirect to `/login?error=<code>` instead of throwing HTTP errors
- `users.last_login_at` is updated on every successful login

## Settings: File Storage

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/settings/files` | admin | list file storage settings and bucket contents |
| DELETE | `/api/settings/files/:key` | admin | delete a stored file |
| DELETE | `/api/settings/files/bucket` | admin | delete entire storage bucket |
| POST | `/api/settings/files/migrate` | admin | migrate file storage between providers |
| POST | `/api/settings/files/test-connection` | admin | test storage connection settings |

## Platform Management

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/admin/platforms` | admin | list all platform connections (without credentials) |
| POST | `/api/admin/platforms` | admin | connect a new platform (Discord or Matrix) |
| PUT | `/api/admin/platforms/:id` | admin | update platform connection config |
| DELETE | `/api/admin/platforms/:id` | admin | disconnect a platform |
| POST | `/api/admin/platforms/:id/test` | admin | test platform bot connection health |

## Initial Setup

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/setup/status` | public | check if initial platform setup is needed |
| POST | `/api/setup/platform` | public | initial platform connection setup (only works when no platforms exist) |

## Internal Landing (MCP / Internal Token Auth)

These endpoints are used by the MCP server and other internal services for programmatic landing page management.

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/internal/landing/page` | internal | load landing page settings |
| GET | `/api/internal/landing/sections` | internal | list landing sections |
| POST | `/api/internal/landing/sections` | internal | create a landing section |
| PUT | `/api/internal/landing/sections/:id` | internal | update a landing section |
| DELETE | `/api/internal/landing/sections/:id` | internal | delete a landing section |
| PUT | `/api/internal/landing/sections/reorder` | internal | reorder landing sections |
| POST | `/api/internal/landing/publish` | internal | publish landing page |

## Public Endpoints

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/public/branding` | public | public community branding (name, logo) |
| GET | `/api/public/landing` | public | published landing page data for Web app rendering |
| GET | `/api/public/footer-pages` | public | published footer pages by locale (supports single slug query) |

## Marketplace

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| GET | `/api/marketplace/apps` | session | list marketplace apps available for install |

## Feedback

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| POST | `/api/feedback` | session | submit user feedback |

## Notes on Current Product Boundaries

- The schema contains `app_marketplace_submissions` for local submission storage, but the active submission review flow runs in the separate marketplace app.
- Landing is public and intentionally thin; if OAuth requests hit landing by mistake, the web shim forwards them to hub instead of handling auth locally.

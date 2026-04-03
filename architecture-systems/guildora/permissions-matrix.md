# Permissions Matrix

This matrix is derived from current route middleware and Nitro auth utilities in `apps/hub`, plus the compatibility auth shim in `apps/web`.

## Role Vocabulary

- `temporaer`: temporary applicant-level permission role
- `user`: standard logged-in member role
- `moderator`: moderation access
- `admin`: administrative access
- `superadmin`: highest internal access

`session` means any authenticated session, including `temporaer`.

## Frontend Routes By App

| App | Route | Page middleware | Effective access |
| --- | --- | --- | --- |
| web | `/` | none | public |
| hub | `/` | `auth` + redirect | session |
| hub | `/login` | none; `auth` layout | public |
| hub | `/dashboard` | `auth` | session |
| hub | `/members` | `auth` | session |
| hub | `/members/:id` | `auth` | session |
| hub | `/profile` | `auth` | session |
| hub | `/profile/customize` | `auth` | session |
| hub | `/profile/:id` | `auth` | session |
| hub | `/apps` | `moderator` | moderator, admin, superadmin |
| hub | `/apps/overview` | `moderator` | moderator, admin, superadmin |
| hub | `/apps/sideload` | `superadmin` | superadmin |
| hub | `/apps/:appId/:slug` | none (ssr: false) | session (app-level auth) |
| hub | `/applications` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/config` | `admin` | admin, superadmin |
| hub | `/applications/flows` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/flows/new` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/flows/:flowId` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/flows/:flowId/settings` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/open` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/open/:applicationId` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/archive` | `moderator` | moderator, admin, superadmin |
| hub | `/applications/archive/:applicationId` | `moderator` | moderator, admin, superadmin |
| hub | `/apply/:flowId/:token` | none; `apply` layout | public (token-validated) |
| hub | `/cms` | `auth` | session at page level; real CMS access enforced by API |
| hub | `/settings` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/design` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/permissions` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/community` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/custom-fields` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/files` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/moderation-rights` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/apps` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/apps/review` | none (redirect) | admin, superadmin |
| hub | `/settings/landing` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/settings/dev-role-switcher` | `settings` | admin, superadmin, or moderator with moderation rights |
| hub | `/dev` | `dev` | development-only |
| hub | `/dev/reset` | `dev` | development-only |
| hub | `/dev/role-switcher` | `dev` | development-only |

## Nitro Routes

### Hub Operational API

| Route | Method | Access |
| --- | --- | --- |
| `/api/auth/discord` | GET | public |
| `/:locale/api/auth/discord` (`en`,`de`) | GET | public compatibility redirect |
| `/api/auth/logout` | POST | public |
| `/api/theme` | GET | public |
| `/api/internal/locale-context` | GET | public |
| `/api/internal/branding` | GET | session |
| `/api/profile` | GET | session |
| `/api/profile` | PUT | session |
| `/api/profile/locale` | PUT | session |
| `/api/profile/avatar` | PUT | session |
| `/api/profile/avatar` | DELETE | session |
| `/api/profile/custom-fields` | GET | session |
| `/api/profile/custom-fields` | PUT | session |
| `/api/profile/discord-roles` | PUT | session |
| `/api/community-settings/display-name-template` | GET | session |
| `/api/members` | GET | session |
| `/api/dashboard/stats` | GET | session |
| `/api/apps` | GET | session |
| `/api/apps/navigation` | GET | session |
| `/api/apps/:appId/_messages` | GET | session |
| `/api/apps/:appId/_source` | GET | session |
| `/api/apps/:appId/_page-source` | GET | session |
| `/api/apps/:appId/activate` | POST | admin, superadmin |
| `/api/apps/:appId/deactivate` | POST | admin, superadmin |
| `/api/dev/users` | GET | staff in debug or switched flow |
| `/api/dev/switch-user` | POST | staff in debug or switched flow |
| `/api/dev/restore-user` | POST | switched debug session |
| `/api/cms/session-url` | GET | moderator, admin, superadmin; moderator access depends on `moderation_settings` |
| `/api/mod/community-roles` | GET | moderator, admin, superadmin |
| `/api/mod/community-roles` | POST | admin, superadmin |
| `/api/mod/community-roles/:id` | PUT | admin, superadmin |
| `/api/mod/community-roles/:id` | DELETE | admin, superadmin |
| `/api/mod/users` | GET | moderator, admin, superadmin |
| `/api/mod/users/:id/profile` | PUT | moderator, admin, superadmin |
| `/api/mod/users/:id/community-role` | PUT | moderator, admin, superadmin |
| `/api/mod/users/:id/custom-fields` | GET | moderator, admin, superadmin |
| `/api/mod/users/:id/custom-fields` | PUT | moderator, admin, superadmin |
| `/api/mod/users/:id/sync` | POST | moderator, admin, superadmin |
| `/api/mod/users/batch-community-role` | POST | moderator, admin, superadmin |
| `/api/mod/users/batch-discord-roles` | POST | moderator, admin, superadmin |
| `/api/mod/tags` | GET | moderator, admin, superadmin |
| `/api/mod/tags` | POST | moderator, admin, superadmin |
| `/api/mod/discord-roles` | GET | moderator, admin, superadmin |
| `/api/applications/open` | GET | moderator, admin, superadmin |
| `/api/applications/:applicationId` | GET | moderator, admin, superadmin |
| `/api/applications/:applicationId/approve` | POST | moderator, admin, superadmin |
| `/api/applications/:applicationId/reject` | POST | moderator, admin, superadmin |
| `/api/applications/:applicationId/retry-roles` | POST | moderator, admin, superadmin |
| `/api/applications/archive` | GET | moderator, admin, superadmin |
| `/api/applications/archive/cleanup` | POST | admin, superadmin |
| `/api/applications/config` | GET | admin, superadmin |
| `/api/applications/moderator-notifications` | GET | moderator, admin, superadmin |
| `/api/applications/moderator-notifications` | PUT | moderator, admin, superadmin |
| `/api/applications/flows` | GET | moderator, admin, superadmin |
| `/api/applications/flows` | POST | moderator, admin, superadmin |
| `/api/applications/flows/:flowId` | GET | moderator, admin, superadmin |
| `/api/applications/flows/:flowId` | PUT | moderator, admin, superadmin |
| `/api/applications/flows/:flowId` | DELETE | moderator, admin, superadmin |
| `/api/applications/flows/:flowId/duplicate` | POST | moderator, admin, superadmin |
| `/api/apply/:flowId/validate-token` | POST | public (token-based) |
| `/api/apply/:flowId/submit` | POST | public (token-based) |
| `/api/apply/:flowId/upload` | POST | public (token-based) |
| `/api/admin/theme` | GET | admin, superadmin |
| `/api/admin/theme` | PUT | admin, superadmin |
| `/api/admin/users` | GET | admin, superadmin |
| `/api/admin/users/batch-delete` | POST | admin, superadmin |
| `/api/admin/apps` | GET | admin, superadmin |
| `/api/admin/apps/sideload` | POST | admin, superadmin |
| `/api/admin/apps/local-sideload` | POST | admin, superadmin |
| `/api/admin/apps/update-check` | GET | admin, superadmin |
| `/api/admin/apps/:appId/config` | PUT | admin, superadmin |
| `/api/admin/apps/:appId/status` | PUT | admin, superadmin |
| `/api/admin/apps/:appId/auto-update` | PUT | admin, superadmin |
| `/api/admin/apps/:appId/update` | POST | admin, superadmin |
| `/api/admin/apps/:id` | DELETE | admin, superadmin |
| `/api/admin/community-settings` | GET | admin, superadmin |
| `/api/admin/community-settings` | PUT | admin, superadmin |
| `/api/admin/cms-access` | PUT | admin, superadmin |
| `/api/admin/permissions` | GET | admin, superadmin |
| `/api/admin/community-roles` | POST | admin, superadmin |
| `/api/admin/community-roles/:id` | PUT | admin, superadmin |
| `/api/admin/community-roles/:id` | DELETE | admin, superadmin |
| `/api/admin/custom-fields` | GET | admin, superadmin |
| `/api/admin/custom-fields` | POST | admin, superadmin |
| `/api/admin/custom-fields/:id` | PUT | admin, superadmin |
| `/api/admin/custom-fields/:id` | DELETE | admin, superadmin |
| `/api/admin/moderation-rights` | GET | admin, superadmin |
| `/api/admin/moderation-rights` | PUT | admin, superadmin |
| `/api/admin/tags/:id` | DELETE | admin, superadmin |
| `/api/admin/discord-roles` | GET | admin, superadmin |
| `/api/admin/discord-roles` | PUT | admin, superadmin |
| `/api/admin/discord-roles/self-import` | POST | superadmin only |
| `/api/admin/discord/roles` | GET | admin, superadmin |
| `/api/admin/discord/roles/:roleId/members` | GET | admin, superadmin |
| `/api/admin/discord/members/:discordId` | GET | admin, superadmin |
| `/api/admin/discord/members/:discordId/remove-roles` | POST | admin, superadmin |
| `/api/admin/users/import` | POST | admin, superadmin |
| `/api/admin/users/delete-orphaned` | POST | admin, superadmin |
| `/api/admin/users/:id` | DELETE | admin, superadmin |
| `/api/admin/users/by-community-role/:communityRoleId` | DELETE | admin, superadmin |
| `/api/admin/dev/reset-mirror` | POST | admin, superadmin |
| `/api/admin/membership-settings` | GET | admin, superadmin |
| `/api/admin/membership-settings` | PUT | admin, superadmin |
| `/api/admin/landing/page` | GET | admin, superadmin |
| `/api/admin/landing/page` | PUT | admin, superadmin |
| `/api/admin/landing/templates` | GET | admin, superadmin |
| `/api/admin/landing/template` | PUT | admin, superadmin |
| `/api/admin/landing/sections` | GET | admin, superadmin |
| `/api/admin/landing/sections` | POST | admin, superadmin |
| `/api/admin/landing/sections/:id` | PUT | admin, superadmin |
| `/api/admin/landing/sections/:id` | DELETE | admin, superadmin |
| `/api/admin/landing/sections/reorder` | PUT | admin, superadmin |
| `/api/admin/landing/blocks` | GET | admin, superadmin |
| `/api/admin/landing/publish` | POST | admin, superadmin |
| `/api/admin/landing/reset` | POST | admin, superadmin |
| `/api/admin/landing/versions` | GET | admin, superadmin |
| `/api/admin/landing/versions/:id` | GET | admin, superadmin |
| `/api/admin/landing/versions/:id/restore` | POST | admin, superadmin |
| `/api/admin/landing-access` | PUT | admin, superadmin |
| `/api/admin/role-groups` | GET | admin, superadmin |
| `/api/admin/role-groups` | POST | admin, superadmin |
| `/api/admin/role-groups/:id` | GET | admin, superadmin |
| `/api/admin/role-groups/:id` | PUT | admin, superadmin |
| `/api/admin/role-groups/:id` | DELETE | admin, superadmin |
| `/api/admin/role-groups/:id/roles` | PUT | admin, superadmin |
| `/api/admin/role-groups/:id/embed/deploy` | POST | admin, superadmin |
| `/api/admin/role-groups/:id/embed` | DELETE | admin, superadmin |
| `/api/admin/discord-channels` | GET | admin, superadmin |
| `/api/admin/cleanup-log` | GET | admin, superadmin |
| `/api/settings/files` | GET | admin, superadmin |
| `/api/settings/files/:key` | DELETE | admin, superadmin |
| `/api/settings/files/bucket` | DELETE | admin, superadmin |
| `/api/settings/files/migrate` | POST | admin, superadmin |
| `/api/settings/files/test-connection` | POST | admin, superadmin |
| `/api/public/branding` | GET | public |
| `/api/public/landing` | GET | public |
| `/api/internal/landing/page` | GET | internal token |
| `/api/internal/landing/sections` | GET | internal token |
| `/api/internal/landing/sections` | POST | internal token |
| `/api/internal/landing/sections/:id` | PUT | internal token |
| `/api/internal/landing/sections/:id` | DELETE | internal token |
| `/api/internal/landing/sections/reorder` | PUT | internal token |
| `/api/internal/landing/publish` | POST | internal token |
| `/api/marketplace/apps` | GET | session |
| `/api/feedback` | POST | session |

### Web Compatibility API

| Route | Method | Access |
| --- | --- | --- |
| `/api/auth/discord` | GET | public redirect to hub |

## Important Nuances

- `/cms` page-level middleware only checks login; actual authorization is done by `/api/cms/session-url`.
- `requireSession` allows any logged-in role, including `temporaer`.
- Session payload should read `permissionRoles` first and `roles` only as compatibility fallback.
- Landing does not own real auth; its `/api/auth/discord` route is only a forwarding shim.
- The `settings` middleware allows admins and superadmins directly, plus moderators who have at least one moderation right enabled in `moderation_settings`.
- `/apply/:flowId/:token` is public but token-validated; it uses the `apply` layout, not the default authenticated layout.

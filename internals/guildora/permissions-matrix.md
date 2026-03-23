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
| hub | `/profile/name` | `auth` | session |
| hub | `/profile/roles` | `auth` | session |
| hub | `/profile/design` | `auth` | session |
| hub | `/profile/:id` | `auth` | session, then redirect |
| hub | `/marketplace` | `auth` | session |
| hub | `/marketplace/submit` | none | public redirect to `/marketplace` |
| hub | `/marketplace/:appId` | none | public redirect to `/marketplace` |
| hub | `/cms` | `auth` | session at page level; real CMS access enforced by API |
| hub | `/mod` | `moderator` | moderator, admin, superadmin |
| hub | `/admin` | `admin` | admin, superadmin |
| hub | `/admin/design` | `admin` | admin, superadmin |
| hub | `/admin/theme` | `admin` | admin, superadmin |
| hub | `/admin/permissions` | `admin` | admin, superadmin |
| hub | `/admin/discord-roles` | `admin` | admin, superadmin |
| hub | `/admin/apps` | `admin` | admin, superadmin |
| hub | `/admin/apps/review` | none | public redirect to `/admin/apps` |
| hub | `/admin/dev-role-switcher` | `admin` | admin, superadmin |
| hub | `/admin/users` | `admin` | admin, superadmin |

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
| `/api/profile` | GET | session; foreign profile lookup is staff-only |
| `/api/profile` | PUT | session |
| `/api/profile/locale` | PUT | session |
| `/api/profile/discord-roles` | PUT | session |
| `/api/members` | GET | session |
| `/api/dashboard/stats` | GET | session |
| `/api/apps` | GET | session |
| `/api/apps/navigation` | GET | session |
| `/api/apps/:appId/activate` | POST | admin, superadmin |
| `/api/apps/:appId/deactivate` | POST | admin, superadmin |
| `/api/dev/users` | GET | staff in debug or switched flow |
| `/api/dev/switch-user` | POST | staff in debug or switched flow |
| `/api/dev/restore-user` | POST | switched debug session |
| `/api/cms/session-url` | GET | moderator, admin, superadmin; moderator access depends on `cms_access_settings` |
| `/api/mod/community-roles` | GET | moderator, admin, superadmin |
| `/api/mod/community-roles` | POST | admin, superadmin |
| `/api/mod/community-roles/:id` | PUT | admin, superadmin |
| `/api/mod/community-roles/:id` | DELETE | admin, superadmin |
| `/api/mod/users` | GET | moderator, admin, superadmin |
| `/api/mod/users/:id/profile` | PUT | moderator, admin, superadmin |
| `/api/mod/users/:id/community-role` | PUT | moderator, admin, superadmin |
| `/api/mod/users/:id/sync` | POST | moderator, admin, superadmin |
| `/api/mod/applications` | GET | moderator, admin, superadmin |
| `/api/mod/applications/:id/approve` | POST | moderator, admin, superadmin |
| `/api/mod/applications/:id/reject` | POST | moderator, admin, superadmin |
| `/api/admin/theme` | GET | admin, superadmin |
| `/api/admin/theme` | PUT | admin, superadmin |
| `/api/admin/users` | GET | admin, superadmin |
| `/api/admin/apps` | GET | admin, superadmin |
| `/api/admin/apps/sideload` | POST | admin, superadmin |
| `/api/admin/apps/:appId/config` | PUT | admin, superadmin |
| `/api/admin/apps/:appId/status` | PUT | admin, superadmin |
| `/api/admin/apps/:id` | DELETE | admin, superadmin |
| `/api/admin/community-settings` | GET | admin, superadmin |
| `/api/admin/community-settings` | PUT | admin, superadmin |
| `/api/admin/cms-access` | PUT | admin, superadmin |
| `/api/admin/permissions` | GET | admin, superadmin |
| `/api/admin/community-roles` | POST | admin, superadmin |
| `/api/admin/community-roles/:id` | PUT | admin, superadmin |
| `/api/admin/community-roles/:id` | DELETE | admin, superadmin |
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

### Web Compatibility API

| Route | Method | Access |
| --- | --- | --- |
| `/api/auth/discord` | GET | public redirect to hub |

## Important Nuances

- `/cms` page-level middleware only checks login; actual authorization is done by `/api/cms/session-url`.
- `requireSession` allows any logged-in role, including `temporaer`.
- Session payload should read `permissionRoles` first and `roles` only as compatibility fallback.
- Landing does not own real auth; its `/api/auth/discord` route is only a forwarding shim.

# Workflow: Admin User Mirroring

This workflow covers the admin-side reconciliation between Discord guild membership and the local app database.

## Why It Exists

The local app stores users, profiles, community-role assignments, and Discord-role snapshots. Discord remains the source of truth for actual guild membership and current role state. The mirror workflow brings both sides back into sync.

## Inputs

- community roles with `discord_role_id` mappings
- guild roles and role members fetched through the bot bridge
- local users and role assignments

## Import Flow

Route:

- `POST /api/admin/users/import`

Flow:

1. Load active mapped community roles.
2. For each mapped Discord role, fetch guild members from the bot bridge.
3. Group members by Discord ID.
4. If a member matches exactly one mapped community role:
   - ensure local `users` row
   - ensure `profiles` row
   - assign the community role
   - sync technical permission roles
   - store Discord-role snapshot
5. If a member matches multiple mapped community roles, record a conflict instead of importing.
6. Return created, updated, conflicting, and orphan candidate results.

## Orphan Detection

An orphan candidate is a local user whose Discord ID is not present in the currently mirrored mapped-role member set.

Superadmins are excluded from orphan deletion candidates.

Route:

- `POST /api/admin/users/delete-orphaned`

This deletes only the selected orphan candidates.

## Single-User Delete/Reconcile Flow

Route:

- `DELETE /api/admin/users/:id`

Flow:

1. Optionally remove all manageable Discord roles plus mapped community-role Discord roles.
2. Fetch the guild member again from the bot.
3. If the member no longer exists in the guild, delete the local user.
4. If the member still exists and now matches exactly one mapped community role, retain and reconcile the local user.
5. If multiple mapped roles remain, return a conflict.

Constraints:

- the acting admin cannot delete themselves
- only superadmins can delete superadmin users

## Bulk Community-Role Delete/Reconcile Flow

Route:

- `DELETE /api/admin/users/by-community-role/:communityRoleId`

Flow:

1. Remove the mapped Discord role for every user currently assigned to that community role.
2. Re-fetch the guild member after removal.
3. Delete users with no remaining mapped role.
4. Retain and reconcile users with exactly one remaining mapped role.
5. Count conflicts for users with multiple remaining mapped roles.

## Auto-Sync (Scheduled)

When `membership_settings.auto_sync_enabled = true`, a Nitro plugin polls every 5 minutes and triggers a sync cycle if the configured interval has elapsed since `auto_sync_last_run`.

Plugin: `apps/hub/server/plugins/membership-auto-sync.ts`

The sync cycle runs the same import logic as `POST /api/admin/users/import`:

1. Load active mapped community roles.
2. Fetch guild members for each mapped Discord role.
3. Upsert local users with single-role matches.
4. Update `auto_sync_last_run` timestamp.

The auto-sync does **not** detect or remove orphans — that is the responsibility of auto-cleanup.

## Auto-Cleanup (Scheduled)

When `membership_settings.auto_cleanup_enabled = true`, a separate Nitro plugin polls every 5 minutes and triggers a cleanup cycle if the configured interval has elapsed since `auto_cleanup_last_run`.

Plugin: `apps/hub/server/plugins/membership-auto-cleanup.ts`
Logic: `apps/hub/server/utils/membership-cleanup.ts`

### Conditions (Per-Permission-Role)

Cleanup conditions are now configured **per permission role** via `membership_settings.cleanup_role_configs` (a `RoleCleanupConfig[]` JSONB array). Each entry specifies:

- `permissionRoleName` — which permission role this config applies to (e.g. `user`, `moderator`)
- `enabled` — whether cleanup is active for this role
- `conditions` — array of condition objects, each with type and AND/OR operator

| Condition | Check |
| --- | --- |
| `orphan` | User is not a guild member (bot bridge lookup) |
| `missingRole` | User lacks a specific Discord role (`cleanupRequiredRoleId` per config) |
| `loginInactive` | `users.last_login_at` older than `cleanupInactiveDays` per config |
| `voiceInactive` | Last `voice_sessions.ended_at` older than `cleanupNoVoiceDays` per config |

### Evaluation Logic

- Each permission role is evaluated independently against its own config.
- OR conditions: at least one must match.
- AND conditions: all must match.
- Mixed: `(any OR matches) AND (all AND conditions match)`.

### Cleanup Actions

For each user matching the conditions:

1. Remove Discord roles via bot bridge (except roles on `cleanup_role_whitelist`).
2. Log the action to `cleanup_log` (discord_id, username, reason, conditions, roles removed).
3. Delete the local user (cascades to profile, role assignments, etc.).
4. Update `auto_cleanup_last_run`.

### Protection

- **Superadmins** and **admins** are always protected.
- **Moderators** are protected by default (`cleanup_protect_moderators = true`).
- If the bot bridge is unreachable, the entire cleanup cycle is skipped (never assume orphan status).
- A module-level guard prevents concurrent cleanup cycles.

## Related Utilities

- `apps/hub/server/utils/admin-mirror.ts`
- `apps/hub/server/utils/community.ts`
- `apps/hub/server/utils/discord-roles.ts`
- `apps/hub/server/utils/membership-settings.ts`
- `apps/hub/server/utils/membership-sync.ts`
- `apps/hub/server/utils/membership-cleanup.ts`

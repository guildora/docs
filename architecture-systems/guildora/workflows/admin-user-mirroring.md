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

## Related Utilities

- `apps/hub/server/utils/admin-mirror.ts`
- `apps/hub/server/utils/community.ts`
- `apps/hub/server/utils/discord-roles.ts`

# Workflow: Discord Role Management

This workflow describes three related but distinct features:

1. admin management of role groups and the self-service Discord role allowlist
2. member self-service selection of those allowed roles
3. role-picker embeds in Discord channels

## Role Groups

Relevant tables:

- `role_groups` — named groups for organizing selectable roles
- `selectable_discord_roles` — allowlist entries, optionally assigned to a group
- `role_picker_embeds` — Discord message embeds tied to a group

Routes:

- `GET/POST /api/admin/role-groups`
- `GET/PUT/DELETE /api/admin/role-groups/:id`
- `PUT /api/admin/role-groups/:id/roles` — update selectable roles in a group
- `POST /api/admin/role-groups/:id/embed/deploy` — deploy role-picker embed to Discord
- `DELETE /api/admin/role-groups/:id/embed` — remove embed

Flow:

1. Admin creates role groups to organize selectable Discord roles.
2. Each group can contain multiple selectable Discord roles with emoji and sort order.
3. Admin can deploy a role-picker embed to a Discord channel for a group, allowing members to pick roles via reactions.
4. Deleting a group cascades to its embed; selectable roles have their `group_id` set to null.

## Admin Allowlist Flow

Relevant tables:

- `selectable_discord_roles`
- `role_groups` (optional grouping)
- `user_discord_roles` as snapshot output, not the allowlist itself

Flow:

1. Admin opens `/admin/discord-roles`.
2. The page loads:
   - community settings
   - guild roles from the bot bridge
   - existing selectable role IDs from `selectable_discord_roles`
   - role groups from `role_groups`
3. Admin saves a new set of allowed role IDs through `PUT /api/admin/discord-roles`.
4. The hub app stores a complete replacement snapshot of the allowlist.

Constraints:

- only editable, non-managed Discord roles are selectable in the UI
- the allowlist is global, not per community role
- roles can optionally be assigned to groups with emoji and sort order

## Member Self-Service Flow

Relevant route:

- `PUT /api/profile/discord-roles`

Flow:

1. Member opens `/profile/roles`.
2. The hub app loads editable roles for the current user from `/api/profile`.
3. On save, the hub app sends the selected role IDs to `/api/profile/discord-roles`.
4. The server verifies that every selected role is part of `selectable_discord_roles`.
5. The server asks the bot bridge to reconcile the member's current Discord roles against:
   - `allowedRoleIds`
   - `selectedRoleIds`
6. The local snapshot in `user_discord_roles` is replaced with the current bot-reported role set.
7. The response returns the updated editable role list plus added/removed role IDs.

## Snapshot Semantics

`user_discord_roles` is not the source of truth for guild roles. It is a local snapshot written after sync or import flows so the internal app can display the last known role set.

## Failure Modes

Common bot error codes propagated to the hub app:

- `UNAUTHORIZED`
- `MISSING_DISCORD_ID`
- `MISSING_ROLE_IDS`
- `INVALID_SELECTED_ROLE_IDS`
- `NON_EDITABLE_ALLOWED_ROLES`
- `SYNC_FAILED`

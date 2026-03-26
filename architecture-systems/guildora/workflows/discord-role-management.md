# Workflow: Discord Role Management

This workflow describes two related but distinct features:

1. admin management of the self-service Discord role allowlist
2. member self-service selection of those allowed roles

## Admin Allowlist Flow

Relevant tables:

- `selectable_discord_roles`
- `user_discord_roles` as snapshot output, not the allowlist itself

Flow:

1. Admin opens `/admin/discord-roles`.
2. The page loads:
   - community settings
   - guild roles from the bot bridge
   - existing selectable role IDs from `selectable_discord_roles`
3. Admin saves a new set of allowed role IDs through `PUT /api/admin/discord-roles`.
4. The hub app stores a complete replacement snapshot of the allowlist.

Constraints:

- only editable, non-managed Discord roles are selectable in the UI
- the allowlist is global, not per community role

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

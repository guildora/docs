# Domain Model

This document describes the application-domain schema defined in `packages/shared/src/db/schema.ts`. It covers the `public` schema used by the hub app and bot. Payload CMS tables in the `payload` schema are documented separately in [`subsystems/cms.md`](./subsystems/cms.md).

## Enums

| Enum | Values | Used by |
| --- | --- | --- |
| `absence_status` | `away`, `maintenance` | `profiles.absence_status` |
| `app_install_status` | `active`, `inactive`, `error` | `installed_apps.status` |
| `app_install_source` | `marketplace`, `sideloaded` | `installed_apps.source` |
| `app_submission_status` | `pending`, `approved`, `rejected` | `app_marketplace_submissions.status` |
| `application_flow_status` | `draft`, `active`, `inactive` | `application_flows.status` |
| `application_status` | `pending`, `approved`, `rejected` | `applications.status` |
| `editor_mode` | `simple`, `advanced` | `application_flows.editor_mode` |
| `landing_section_status` | `draft`, `published` | `landing_sections.status` |

## Tables

### `users`

Primary identity table.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `discord_id` | text unique | canonical Discord identity |
| `email` | text nullable | usually from OAuth |
| `display_name` | text | serialized profile name |
| `avatar_url` | text nullable | current avatar URL (local public path preferred) |
| `avatar_source` | text nullable | avatar origin marker (`local`, `discord`, or null) |
| `primary_discord_role_name` | text nullable | highest Discord role name snapshot at last successful login |
| `last_login_at` | timestamptz nullable | updated on each successful Discord OAuth login |
| `created_at` | timestamptz | default now |
| `updated_at` | timestamptz | auto-updated |

Relations:

- one profile
- many permission-role assignments
- one community-role assignment
- many Discord-role snapshots
- many voice sessions
- many profile change-log rows
- optional creator/reviewer references for apps and submissions

### `profiles`

Per-user profile details.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `user_id` | uuid unique FK -> `users.id` | one profile per user |
| `absence_status` | enum nullable | `away` or `maintenance` |
| `absence_message` | text nullable | freeform status note |
| `absence_until` | timestamptz nullable | scheduled end |
| `custom_fields` | jsonb | app and UI metadata |
| `locale_preference` | text nullable | canonical per-user locale preference |
| `updated_at` | timestamptz | auto-updated |

Known `custom_fields` keys used by current code:

- `applicationStatus`
- `applicationReviewedAt`
- `applicationReviewedBy`
- `appearancePreference`
- `localePreference` as a legacy read fallback

There is currently no `bio` column in the schema.

### `permission_roles`

Technical access roles.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | |
| `name` | text unique | `temporaer`, `user`, `moderator`, `admin`, `superadmin` |
| `description` | text nullable | |
| `level` | integer | hierarchy helper |

### `community_roles`

Domain-facing community roles mapped to one permission role.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | |
| `name` | text unique | examples: `Bewerber`, `Anwaerter`, `Mitglied` |
| `description` | text nullable | |
| `discord_role_id` | text unique nullable | admin-managed mapping to a Discord role |
| `permission_role_id` | integer FK -> `permission_roles.id` | required |
| `sort_order` | integer | UI ordering |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `user_permission_roles`

Join table for technical access roles.

| Field | Type | Notes |
| --- | --- | --- |
| `user_id` | uuid FK -> `users.id` | PK part |
| `permission_role_id` | integer FK -> `permission_roles.id` | PK part |
| `assigned_at` | timestamptz | |

### `user_community_roles`

One community role per user.

| Field | Type | Notes |
| --- | --- | --- |
| `user_id` | uuid unique FK -> `users.id` | one row per user |
| `community_role_id` | integer FK -> `community_roles.id` | required |
| `assigned_at` | timestamptz | |

### `role_groups`

Named groups for organizing selectable Discord roles.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `name` | text unique | group display name |
| `description` | text nullable | |
| `sort_order` | integer | UI ordering, default 0 |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `selectable_discord_roles`

Admin-curated allowlist for self-service Discord roles, optionally grouped.

| Field | Type | Notes |
| --- | --- | --- |
| `discord_role_id` | text PK | guild role ID |
| `role_name_snapshot` | text | role name at save time |
| `group_id` | uuid nullable FK -> `role_groups.id` | optional group assignment (ON DELETE SET NULL) |
| `emoji` | text nullable | display emoji |
| `sort_order` | integer | ordering within group, default 0 |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `role_picker_embeds`

Discord message embeds for role-picker interactions, one per group.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `group_id` | uuid unique FK -> `role_groups.id` | one embed per group (ON DELETE CASCADE) |
| `discord_channel_id` | text | target Discord channel |
| `discord_message_id` | text nullable | posted message ID |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `user_discord_roles`

Snapshot of Discord roles currently assigned to a user.

| Field | Type | Notes |
| --- | --- | --- |
| `user_id` | uuid FK -> `users.id` | PK part |
| `discord_role_id` | text | PK part |
| `role_name_snapshot` | text | role name at sync time |
| `synced_at` | timestamptz | |

### `voice_sessions`

Discord voice-session ledger.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `user_id` | uuid FK -> `users.id` | required |
| `channel_id` | text nullable | Discord channel ID |
| `started_at` | timestamptz | required |
| `ended_at` | timestamptz nullable | null while open |
| `duration_minutes` | integer nullable | calculated when closed |
| `created_at` | timestamptz | |

Important index behavior:

- one open session per user
- indexed lookup for open sessions

### `profile_change_log`

Audit trail for application and profile progression events.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `profile_id` | uuid FK -> `profiles.id` | required |
| `change_type` | text | domain-specific event name |
| `previous_value` | text nullable | |
| `new_value` | text nullable | |
| `is_growth` | boolean | defaults false |
| `is_departure` | boolean | defaults false |
| `changed_by` | uuid nullable FK -> `users.id` | actor when known |
| `created_at` | timestamptz | |

### `installed_apps`

Stored app manifests currently known to the platform.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `app_id` | text unique | manifest ID |
| `name` | text | |
| `version` | text | |
| `status` | enum | `active`, `inactive`, `error` |
| `source` | enum | `marketplace` or `sideloaded` |
| `verified` | boolean | trusted marker |
| `repository_url` | text nullable | source repository |
| `manifest` | jsonb | parsed against app-manifest schema |
| `config` | jsonb | app configuration blob |
| `code_bundle` | jsonb | `Record<string,string>` mapping file paths to transpiled CJS source; not null, defaults `{}` |
| `auto_update` | boolean | automatically update when new versions are available; not null, defaults false |
| `created_by` | uuid nullable FK -> `users.id` | |
| `installed_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `app_marketplace_submissions`

Marketplace-submission storage retained in the schema.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `app_id` | text | |
| `name` | text | |
| `version` | text | |
| `source_url` | text nullable | source manifest URL |
| `manifest` | jsonb | submitted manifest |
| `status` | enum | `pending`, `approved`, `rejected` |
| `automated_checks` | jsonb | check report |
| `review_notes` | text nullable | |
| `submitted_by_user_id` | uuid nullable FK -> `users.id` | |
| `reviewed_by_user_id` | uuid nullable FK -> `users.id` | |
| `reviewed_at` | timestamptz nullable | |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

This table exists, but the active local web workflow is limited. See [`workflows/marketplace-submissions.md`](./workflows/marketplace-submissions.md).

### `theme_settings`

Global internal-app theme configuration.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | effectively singleton-like |
| `color_dominant` | text | hex color |
| `color_secondary` | text | hex color |
| `color_accent` | text | hex color |
| `color_accent_content_tone` | text | `light` or `dark` |
| `color_info` | text | hex color |
| `color_info_content_tone` | text | `light` or `dark` |
| `color_success` | text | hex color |
| `color_success_content_tone` | text | `light` or `dark` |
| `color_warning` | text | hex color |
| `color_warning_content_tone` | text | `light` or `dark` |
| `color_error` | text | hex color |
| `color_error_content_tone` | text | `light` or `dark` |
| `logo_data_url` | text nullable | stored logo payload |
| `logo_mime_type` | text nullable | |
| `logo_file_name` | text nullable | |
| `sidebar_logo_size_px` | integer nullable | allowed values constrained in web utils |
| `updated_at` | timestamptz | |
| `updated_by` | uuid nullable FK -> `users.id` | |

### `app_kv`

Key-value storage for installed app data, scoped by app ID.

| Field | Type | Notes |
| --- | --- | --- |
| `app_id` | text | PK part |
| `key` | text | PK part |
| `value` | jsonb nullable | |

Composite primary key: `(app_id, key)`.

### `moderation_settings`

Controls moderator access to various admin features.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | |
| `allow_moderator_access` | boolean | defaults true |
| `allow_moderator_apps_access` | boolean | defaults true; controls moderator access to apps module |
| `mod_delete_users` | boolean | defaults false; allow moderators to delete users |
| `mod_manage_applications` | boolean | defaults false; allow moderators to manage applications |
| `mod_access_community_settings` | boolean | defaults false; allow moderators to access community settings |
| `mod_access_design` | boolean | defaults false; allow moderators to access design settings |
| `mod_access_apps` | boolean | defaults false; allow moderators to access app management |
| `mod_access_discord_roles` | boolean | defaults false; allow moderators to access Discord role settings |
| `mod_access_custom_fields` | boolean | defaults false; allow moderators to access custom field settings |
| `mod_access_permissions` | boolean | defaults false; allow moderators to access permission settings |
| `updated_at` | timestamptz | |
| `updated_by` | uuid nullable FK -> `users.id` | |

### `community_settings`

Global community-level settings used by the internal app.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | current code treats `id = 1` as the singleton |
| `community_name` | text nullable | internal sidebar branding |
| `discord_invite_code` | text nullable | Discord server invite code |
| `default_locale` | text | `en` or `de` |
| `display_name_template` | jsonb | `DisplayNameField[]` array defining which fields compose the display name; defaults `[]` |
| `updated_at` | timestamptz | |
| `updated_by` | uuid nullable FK -> `users.id` | |

### `application_flows`

Application form definitions. Two editor modes: **Simple** (linear section-based builder) and **Advanced** (visual Vue Flow graph editor). Both produce the same `flow_json` format.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `name` | text | flow display name |
| `status` | enum | `draft`, `active`, `inactive` |
| `editor_mode` | enum | `simple` (default), `advanced` — tracks which editor UI was used |
| `flow_json` | jsonb | graph definition (`ApplicationFlowGraph`) |
| `draft_flow_json` | jsonb nullable | work-in-progress draft of the flow graph |
| `settings_json` | jsonb | flow settings (`ApplicationFlowSettings`) |
| `created_by` | uuid nullable FK -> `users.id` | creator |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `application_flow_embeds`

Discord message embeds for application flows (posted by bot).

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `flow_id` | uuid FK -> `application_flows.id` | cascade delete |
| `discord_channel_id` | text | target Discord channel |
| `discord_message_id` | text nullable | posted message ID |
| `active` | boolean | defaults true |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `application_tokens`

Single-use tokens for public application form access.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `flow_id` | uuid FK -> `application_flows.id` | cascade delete |
| `discord_id` | text | applicant Discord ID |
| `discord_username` | text | applicant username |
| `discord_avatar_url` | text nullable | applicant avatar |
| `token` | text unique | random secure token |
| `expires_at` | timestamptz | token expiry |
| `used_at` | timestamptz nullable | null until submitted |
| `created_at` | timestamptz | |

### `applications`

Individual application submissions tied to a flow.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `flow_id` | uuid FK -> `application_flows.id` | cascade delete |
| `discord_id` | text | applicant Discord ID |
| `discord_username` | text | applicant username |
| `discord_avatar_url` | text nullable | applicant avatar |
| `answers_json` | jsonb | form answers keyed by node ID |
| `status` | enum | `pending`, `approved`, `rejected` |
| `roles_assigned` | jsonb | Discord role IDs assigned on approval |
| `pending_role_assignments` | jsonb | roles queued for assignment |
| `display_name_composed` | text nullable | composed display name from answers |
| `ticket_channel_id` | text nullable | Discord ticket channel ID |
| `reviewed_by` | uuid nullable FK -> `users.id` | reviewer |
| `reviewed_at` | timestamptz nullable | |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `application_file_uploads`

File attachments submitted with applications.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `application_id` | uuid nullable FK -> `applications.id` | cascade delete |
| `discord_id` | text | uploader Discord ID |
| `flow_id` | uuid FK -> `application_flows.id` | cascade delete |
| `original_filename` | text | original file name |
| `mime_type` | text | file MIME type |
| `storage_path` | text | server file path |
| `file_size` | integer | size in bytes |
| `created_at` | timestamptz | |

### `application_moderator_notifications`

Per-flow moderator notification subscriptions.

| Field | Type | Notes |
| --- | --- | --- |
| `flow_id` | uuid FK -> `application_flows.id` | PK part, cascade delete |
| `user_id` | uuid FK -> `users.id` | PK part, cascade delete |
| `enabled` | boolean | defaults true |

Composite primary key: `(flow_id, user_id)`.

### `community_custom_fields`

Admin-managed custom profile field definitions.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `key` | text unique | not null; field identifier |
| `label` | text | not null; display label |
| `description` | text nullable | help text |
| `input_type` | text | not null; field type (text, number, select, slider, etc.) |
| `options` | jsonb nullable | `string[]` for select-type fields |
| `slider_min` | integer nullable | minimum value for slider fields |
| `slider_max` | integer nullable | maximum value for slider fields |
| `slider_step` | integer nullable | step increment for slider fields |
| `required` | boolean | defaults false |
| `active` | boolean | defaults true |
| `is_default` | boolean | defaults false; system-provided default fields |
| `user_can_view` | boolean | defaults false; whether users can see this field on profiles |
| `user_can_edit` | boolean | defaults false; whether users can edit their own value |
| `mod_can_view` | boolean | defaults false; whether moderators can see this field |
| `mod_can_edit` | boolean | defaults false; whether moderators can edit this field |
| `sort_order` | integer | not null, defaults 0 |
| `created_at` | timestamptz | |
| `updated_at` | timestamptz | auto-updated |

### `community_tags`

Moderator-managed labels for community members.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `name` | text unique | not null |
| `created_at` | timestamptz | |
| `created_by` | uuid nullable FK -> `users.id` | moderator who created the tag |

### `application_access_settings`

Controls moderator access to the application module.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | |
| `allow_moderator_access` | boolean | defaults true |
| `updated_at` | timestamptz | |
| `updated_by` | uuid nullable FK -> `users.id` | |

### `membership_settings`

Singleton (id=1) controlling membership, auto-login, auto-sync, and auto-cleanup behavior.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | singleton, always 1 |
| `applications_required` | boolean | default true; when false, enables auto-login and hides applications section |
| `default_community_role_id` | integer nullable FK -> `community_roles.id` | role assigned to auto-login users (ON DELETE SET NULL) |
| `required_login_role_id` | text nullable | Discord role ID required to log in (only when applications disabled) |
| `auto_sync_enabled` | boolean | default false |
| `auto_sync_interval_hours` | integer | default 24 |
| `auto_sync_last_run` | timestamptz nullable | |
| `auto_cleanup_enabled` | boolean | default false |
| `auto_cleanup_interval_hours` | integer | default 24 |
| `auto_cleanup_last_run` | timestamptz nullable | |
| `cleanup_role_configs` | jsonb | `RoleCleanupConfig[]` — per-permission-role cleanup configuration (see below) |
| `cleanup_role_whitelist` | jsonb | array of Discord role IDs preserved during cleanup |
| `cleanup_protect_moderators` | boolean | default true |
| `updated_at` | timestamptz | |
| `updated_by` | uuid nullable FK -> `users.id` | |

`RoleCleanupConfig` shape:

```ts
interface RoleCleanupConfig {
  permissionRoleName: string;
  enabled: boolean;
  conditions: { type: "orphan" | "missingRole" | "loginInactive" | "voiceInactive"; operator: "AND" | "OR" }[];
  cleanupRequiredRoleId?: string | null;
  cleanupInactiveDays?: number | null;
  cleanupNoVoiceDays?: number | null;
}
```

### `cleanup_log`

Audit log for auto-cleanup actions. Each row represents one user removed by the cleanup cycle.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `user_id` | uuid nullable | null after user deletion |
| `discord_id` | text | snapshot, preserved after deletion |
| `discord_username` | text | snapshot, preserved after deletion |
| `reason` | text | human-readable summary of matched conditions |
| `conditions_matched` | jsonb | array of condition type strings that triggered removal |
| `roles_removed` | jsonb | array of Discord role IDs removed before deletion |
| `created_at` | timestamptz | indexed DESC for paginated queries |

### `landing_templates`

Built-in and custom landing page templates.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | text PK | template identifier (e.g. `default`) |
| `name` | text | display name |
| `description` | text nullable | |
| `preview_url` | text nullable | template preview image |
| `is_builtin` | boolean | defaults true |
| `created_at` | timestamptz | |

### `landing_pages`

Landing page configuration (singleton-like, one row per locale).

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | |
| `active_template` | text FK -> `landing_templates.id` | default `default` (ON DELETE SET DEFAULT) |
| `custom_css` | text nullable | custom CSS override |
| `locale` | text | default `en` |
| `meta_title` | text nullable | SEO title |
| `meta_description` | text nullable | SEO description |
| `published_at` | timestamptz nullable | last publish timestamp |
| `updated_at` | timestamptz | auto-updated |
| `updated_by` | uuid nullable FK -> `users.id` | |

### `landing_sections`

Individual content sections on the landing page.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `block_type` | text | section type identifier |
| `sort_order` | integer | display order |
| `visible` | boolean | defaults true |
| `status` | enum `landing_section_status` | `draft` or `published`, defaults `published` |
| `config` | jsonb | section configuration |
| `content` | jsonb | section content data |
| `updated_at` | timestamptz | auto-updated |
| `updated_by` | uuid nullable FK -> `users.id` | |

### `landing_page_versions`

Snapshots of the landing page for version history and rollback.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `snapshot` | jsonb | full page state at time of save |
| `label` | text nullable | optional version label |
| `created_at` | timestamptz | |
| `created_by` | uuid nullable FK -> `users.id` | |

## Relationship Summary

- each user has one profile and one community-role assignment
- a community role maps to exactly one permission role
- a user can hold multiple permission roles
- role groups organize selectable Discord roles; each group can have one role-picker embed
- selectable Discord roles optionally belong to a role group and define the self-service boundary for `/api/profile/discord-roles`
- role picker embeds are Discord messages tied to a role group for reaction-based role assignment
- user Discord-role snapshots are informational and synchronization-oriented
- theme, moderation settings, community settings, application access settings, and membership settings behave as singleton-style configuration tables
- membership settings references one optional default community role
- cleanup log entries reference a user ID that may become null after the user is deleted
- community custom fields define the field schema; per-user values are stored in `profiles.custom_fields`
- community tags are moderator-managed labels distinct from Discord roles
- an application flow has many tokens, applications, file uploads, embeds, and moderator notification subscriptions
- an application belongs to one flow and may have many file uploads
- landing templates have many landing pages; landing pages reference one active template
- landing sections are ordered content blocks rendered on the public landing page
- landing page versions store historical snapshots for rollback

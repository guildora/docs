# Domain Model

This document describes the application-domain schema defined in `packages/shared/src/db/schema.ts`. It covers the `public` schema used by the hub app and bot. Payload CMS tables in the `payload` schema are documented separately in [`subsystems/cms.md`](./subsystems/cms.md).

## Enums

| Enum | Values | Used by |
| --- | --- | --- |
| `absence_status` | `away`, `maintenance` | `profiles.absence_status` |
| `app_install_status` | `active`, `inactive`, `error` | `installed_apps.status` |
| `app_install_source` | `marketplace`, `sideloaded` | `installed_apps.source` |
| `app_submission_status` | `pending`, `approved`, `rejected` | `app_marketplace_submissions.status` |

## Tables

### `users`

Primary identity table.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | uuid PK | generated |
| `discord_id` | text unique | canonical Discord identity |
| `email` | text nullable | usually from OAuth |
| `display_name` | text | serialized profile name |
| `avatar_url` | text nullable | remote avatar URL |
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

### `selectable_discord_roles`

Admin-curated allowlist for self-service Discord roles.

| Field | Type | Notes |
| --- | --- | --- |
| `discord_role_id` | text PK | guild role ID |
| `role_name_snapshot` | text | role name at save time |
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

### `cms_access_settings`

Controls whether moderators can use embedded CMS access.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | |
| `allow_moderator_access` | boolean | defaults true |
| `updated_at` | timestamptz | |
| `updated_by` | uuid nullable FK -> `users.id` | |

### `community_settings`

Global community-level settings used by the internal app.

| Field | Type | Notes |
| --- | --- | --- |
| `id` | serial PK | current code treats `id = 1` as the singleton |
| `community_name` | text nullable | internal sidebar branding |
| `default_locale` | text | `en` or `de` |
| `updated_at` | timestamptz | |
| `updated_by` | uuid nullable FK -> `users.id` | |

## Relationship Summary

- each user has one profile and one community-role assignment
- a community role maps to exactly one permission role
- a user can hold multiple permission roles
- selectable Discord roles define the self-service boundary for `/api/profile/discord-roles`
- user Discord-role snapshots are informational and synchronization-oriented
- theme, CMS access, and community settings behave as singleton-style configuration tables

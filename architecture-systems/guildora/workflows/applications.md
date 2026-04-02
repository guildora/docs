# Workflow: Community Applications

This workflow covers flow-based membership applications. It is distinct from marketplace app submissions.

## Data Model

- `application_flows` — flow definitions with `editor_mode` (`simple` or `advanced`)
- `applications` — individual submitted applications
- `application_tokens` — single-use tokens for public form access
- `application_file_uploads` — file attachments submitted with applications
- `application_flow_embeds` — Discord message embeds posted by the bot
- `application_moderator_notifications` — per-flow notification subscriptions
- `application_access_settings` — controls moderator access to the applications module

## Flow Lifecycle

Each application flow has a status: `draft`, `active`, or `inactive`.

- Staff creates a flow at `/applications/flows/new`. New flows default to **Simple Mode**.
- The graph (`flow_json`) defines the applicant's journey: start → input nodes → info nodes → conditional branches → role assignment → end (or abort).
- Flows can optionally store a `draft_flow_json` for work-in-progress edits before publishing.
- Flow settings (`settings_json`) configure embed appearance, role assignments on submission/approval, welcome messages, concurrency rules, token expiry, DM templates, test mode, and archive retention.
- Only `active` flows can accept new applications.

### Editor Modes

The `editor_mode` column (enum: `simple`, `advanced`) tracks which editor UI a flow uses:

- **Simple Mode** (default for new flows): A linear, section-based form builder. Fields are grouped into named sections and reorderable via drag & drop within sections. Supports all input types, info blocks, and role assignment blocks. Does not support conditional branches or abort nodes. Internally generates a standard `ApplicationFlowGraph` (step_group nodes for sections, child nodes for fields, linear edges).
- **Advanced Mode**: The visual Vue Flow graph editor. Supports all node types including conditional branches and abort nodes. Required for non-linear flows.

**Bidirectional switching**: Simple ↔ Advanced is possible at any time. Switching from Advanced to Simple requires a compatibility check — if the graph contains conditional branches, abort nodes, or non-linear edges, the Simple tab is disabled with an explanatory hint. Switching from Simple to Advanced is always allowed.

Both modes produce the same `flow_json` format. The conversion layer (`packages/shared/src/utils/flow-simple-convert.ts`) handles `sectionsToFlowGraph()`, `flowGraphToSections()`, and `canConvertToSimple()`.

New flows are pre-filled with a gaming guild application template (2 sections, 6 fields).

### Node Types

- `start` / `end` — entry and exit points
- `input` — collects a form field (short text, long text, number, email, select, multi-select, yes/no, date, file upload, Discord username, Discord role single/multi)
- `info` — displays informational markdown with optional CTA links
- `conditional_branch` — branches the flow based on a previous input's answer
- `abort` — terminates the flow with a message (e.g. ineligible applicant)
- `role_assignment` — specifies Discord roles to assign when traversed
- `step_group` — groups input nodes visually into a step

## Application Process

### 1. Embed Posting

When a flow is activated, staff can post an embed to a Discord channel. The bot creates a message with an "Apply" button.

### 2. Token Generation (Bot)

When a Discord user clicks the embed button:

1. The bot loads the flow and verifies it is `active`.
2. If test mode is off: checks guild membership and whether the user already has approval roles.
3. Concurrency settings are enforced: `allowReapplyToSameFlow` and `allowCrossFlowApplications`.
4. A single-use `application_token` is generated with HMAC-SHA256 signing and a configurable expiry (default 60 minutes).
5. The bot replies with an ephemeral message containing a link to the public form.

### 3. Public Form

The applicant visits `/apply/:flowId/:token` in the hub.

1. Token validation: `POST /api/apply/:flowId/validate-token` verifies signature, expiry, and that the token has not been used.
2. The flow graph is linearized into sequential steps for rendering.
3. The applicant fills out form fields and optionally uploads files via `POST /api/apply/:flowId/upload`.
4. On submit: `POST /api/apply/:flowId/submit` stores the application with status `pending`, marks the token as used, assigns submission roles if configured, sends a welcome message if configured, and notifies subscribed moderators.

### 4. Review

Moderators access applications through the hub UI:

- `GET /api/applications/open` — list pending applications
- `GET /api/applications/:applicationId` — view individual application details

### 5. Approval

`POST /api/applications/:applicationId/approve`

1. Assigns configured Discord roles (from flow settings `roles.onApproval` plus roles collected from `role_assignment` nodes in the traversed path).
2. Composes a display name from flow answers if a display name template is configured.
3. Updates application status to `approved`.
4. Sends an approval DM to the applicant if configured.

### 6. Rejection

`POST /api/applications/:applicationId/reject`

1. Updates application status to `rejected`.
2. Sends a rejection DM to the applicant if configured.

### 7. Retry Role Assignments

`POST /api/applications/:applicationId/retry-roles` — retries failed Discord role assignments for an approved application.

## Archive

- `GET /api/applications/archive` — list completed (approved/rejected) applications
- `POST /api/applications/archive/cleanup` — delete archived applications older than the configured retention period

## Flow Management

- `GET /api/applications/flows` — list all flows
- `POST /api/applications/flows` — create a new flow
- `GET /api/applications/flows/:flowId` — get flow details
- `PUT /api/applications/flows/:flowId` — update flow graph and settings
- `DELETE /api/applications/flows/:flowId` — delete a flow
- `POST /api/applications/flows/:flowId/duplicate` — duplicate an existing flow

## Configuration

- `GET /api/applications/config` — get application module configuration

## Moderator Notifications

- `GET /api/applications/moderator-notifications` — get notification subscriptions
- `PUT /api/applications/moderator-notifications` — update notification subscriptions

## Access Control

`application_access_settings.allow_moderator_access` controls whether moderators can access the applications module. Admins and superadmins always have access.

## Important Constraints

- tokens are single-use and time-limited
- file uploads are scoped to a flow and linked to an application on submission
- concurrency enforcement prevents duplicate applications based on flow settings
- the flow graph is linearized at render time using conditional branching logic
- role assignment nodes collect Discord role IDs during graph traversal; these are assigned on approval
- embed posting, token generation, and ephemeral replies are handled by the bot, not the hub

## Related Routes

- `POST /api/apply/:flowId/validate-token` — public (token-based)
- `POST /api/apply/:flowId/submit` — public (token-based)
- `POST /api/apply/:flowId/upload` — public (token-based)
- `GET /api/applications/flows` — staff
- `POST /api/applications/flows` — staff
- `GET /api/applications/flows/:flowId` — staff
- `PUT /api/applications/flows/:flowId` — staff
- `DELETE /api/applications/flows/:flowId` — staff
- `POST /api/applications/flows/:flowId/duplicate` — staff
- `GET /api/applications/open` — staff
- `GET /api/applications/:applicationId` — staff
- `POST /api/applications/:applicationId/approve` — staff
- `POST /api/applications/:applicationId/reject` — staff
- `POST /api/applications/:applicationId/retry-roles` — staff
- `GET /api/applications/archive` — staff
- `POST /api/applications/archive/cleanup` — admin
- `GET /api/applications/config` — admin
- `GET /api/applications/moderator-notifications` — staff
- `PUT /api/applications/moderator-notifications` — staff

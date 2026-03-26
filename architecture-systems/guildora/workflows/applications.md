# Workflow: Community Applications

This workflow covers membership applications handled in the moderator area. It is distinct from marketplace app submissions.

## Relevant Data

- `user_community_roles`
- `community_roles`
- `user_permission_roles`
- `profiles.custom_fields.applicationStatus`
- `profiles.custom_fields.applicationReviewedAt`
- `profiles.custom_fields.applicationReviewedBy`
- `profile_change_log`

## What Counts as an Open Application

The current code treats a user as an active applicant when both are true:

1. their community role is `Bewerber`
2. `applicationStatus` is `open` or missing

The applications list is returned by `GET /api/mod/applications`.

## Approval Flow

1. A moderator or admin posts to `/api/mod/applications/:id/approve`.
2. The target user must currently be in the `Bewerber` community role.
3. The target community role defaults to `Anwaerter` unless another allowed target is provided.
4. The code rejects approving into `Bewerber`.
5. Community-role assignment is updated.
6. Technical permission roles are synchronized.
7. Profile custom fields are updated to `applicationStatus = approved`.
8. A profile change-log row is written.
9. The Discord sync bridge is triggered.

## Rejection Flow

1. A moderator or admin posts to `/api/mod/applications/:id/reject`.
2. The target user must currently be in the `Bewerber` community role.
3. The flow sets `applicationStatus = rejected`.
4. The user remains in the applicant role unless another action changes them later.
5. A profile change-log row is written.
6. Permission roles are synchronized.
7. The Discord sync bridge is triggered.

## Important Constraints

- an already rejected application cannot be rejected again
- approving a previously rejected application is blocked by the current handler logic
- the workflow is tightly coupled to the seeded role names `Bewerber` and `Anwaerter`

## Related Routes

- `GET /api/mod/applications`
- `POST /api/mod/applications/:id/approve`
- `POST /api/mod/applications/:id/reject`

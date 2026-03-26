# Workflow: Marketplace Submissions

## Current Product State

The repository still contains marketplace-related schema and utility code, but the active web product does not implement a local submission and review UI.

Current user-facing reality:

- `/marketplace` is an external embed shell
- `/marketplace/submit` redirects to `/marketplace`
- `/marketplace/:appId` redirects to `/marketplace`

## What Still Exists Locally

- `app_marketplace_submissions` table in the shared schema
- submission-check logic in `apps/hub/server/utils/apps.ts`
- installed-app management in the admin area
- GitHub manifest sideloading through `POST /api/admin/apps/sideload`

## Practical Meaning

For this repository today:

- installed apps are a real feature
- marketplace submissions are a retained data model and extension seam
- a full local marketplace review workflow is not a current web-app capability

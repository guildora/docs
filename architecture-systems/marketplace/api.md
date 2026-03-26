# API Reference

## Authentication Endpoints

### `GET /api/auth/github`

GitHub OAuth callback handler. Initiates OAuth flow or processes callback.

**Auth:** none (initiates login)
**Scopes:** `read:user`, `user:email`, `read:org`
**Behavior:** Creates/updates developer record in database, checks GitHub org membership for admin status, creates encrypted session cookie.

### `POST /api/auth/logout`

Clears the session cookie.

**Auth:** none

### `GET /api/auth/session`

Returns current session info.

**Auth:** optional (returns null if not logged in)

---

## Public Endpoints

### `GET /api/marketplace/apps`

Public marketplace listing. Returns published apps with approved submissions.

**Auth:** none

### `POST /api/submit-app`

Public app submission (no authentication required). Creates an app submission for review.

**Auth:** none

### `POST /api/feedback`

Public feedback endpoint. Creates a Linear issue with the feedback content.

**Auth:** none

---

## Hub Integration Endpoint

### `GET /api/hub/apps`

Public endpoint for Hub integration. Hub fetches approved app data through this endpoint.

**Auth:** none
**CORS:** `Access-Control-Allow-Origin: {NUXT_HUB_CORS_ORIGIN}` (default: `*`)
**Cache:** `Cache-Control: public, max-age=60, stale-while-revalidate=300`

#### Response Shape

```ts
{
  items: {
    id: string;                    // UUID (submission ID)
    appId: string;                 // manifest ID (e.g. "my-app")
    name: string;                  // display name
    version: string;               // latest approved submission version
    sourceUrl: string | null;      // repository URL
    manifest: {
      id: string;
      name: string;
      version: string;
      author: string | null;
      description: string | null;
      homepageUrl?: string;
      repositoryUrl?: string;
      license?: string;
      permissions: unknown[];
      configFields: unknown[];
      compatibility: {
        core: { minVersion: string; maxVersion?: string };
      };
    };
    reviewedAt: string | null;     // ISO timestamp
    createdAt: string;             // ISO timestamp
  }[]
}
```

#### Filtering and Mapping

- Includes only published apps with at least one approved submission.
- Uses the latest approved submission per app.
- Returns a sanitized manifest subset suitable for Hub integration.

#### Intentionally Omitted Manifest Fields

The endpoint omits platform-private manifest fields such as:

- `navigation`
- `apiRoutes`
- `botHooks`
- `pages`
- `migrations`
- `requiredEnv`
- `installNotes`

---

## Developer Endpoints (Auth Required)

All developer endpoints require `requireDeveloperSession` — a valid GitHub OAuth session.

### `GET /api/developer/apps`

List all apps registered by the authenticated developer.

### `POST /api/developer/apps`

Register a new app. Creates an `app_registry` entry linked to the developer.

### `GET /api/developer/apps/[appId]`

Get details for a specific registered app.

### `POST /api/developer/apps/[appId]/submit`

Submit a new version for review. Creates an `app_submissions` entry.

### `GET /api/developer/submissions/[id]`

Get details for a specific submission including review status.

### `GET /api/developer/notifications`

List developer notifications (submission status changes, review results).

### `PUT /api/developer/notifications/[id]/read`

Mark a specific notification as read.

### `PUT /api/developer/notifications/read-all`

Mark all notifications as read.

### `PUT /api/developer/profile`

Update developer profile (display name, email).

---

## Admin Endpoints (Admin Required)

All admin endpoints require `requireAdminSession` — GitHub org membership.

### `GET /api/admin/stats`

Dashboard statistics (total apps, submissions, pending reviews).

### `GET /api/admin/submissions`

List all submissions across all developers with filtering options.

### `GET /api/admin/submissions/[id]`

Get detailed submission info for review.

### `POST /api/admin/submissions/[id]/review`

Submit a human review (approve, reject, or request changes).

### `POST /api/admin/submissions/[id]/trigger-ai-review`

Trigger an AI-powered code review via Anthropic Claude API. Stores result in `app_reviews`.

---

## Webhook Endpoints

### `POST /api/webhooks/github`

GitHub webhook handler for repository events (primarily new releases). Validates webhook signature, stores event in `github_webhook_events`, and optionally creates a new submission.

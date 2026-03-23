# API Reference

## Hub Endpoint (external, source)

### `GET /api/marketplace/apps` (Hub)

Public endpoint in the Hub monorepo (`apps/hub/server/api/marketplace/apps.get.ts`).

**Auth:** none
**CORS:** `Access-Control-Allow-Origin: {MARKETPLACE_CORS_ORIGIN}` (default: `*`)
**Cache:** `Cache-Control: public, max-age=60, stale-while-revalidate=300`

#### Response Shape

```ts
{
  items: {
    id: string;                    // UUID (submission ID)
    appId: string;                 // manifest ID (e.g. "my-app")
    name: string;                  // display name (from submission)
    version: string;               // version string
    sourceUrl: string | null;      // origin URL of the manifest
    manifest: {
      id: string;
      name: string;
      version: string;
      author: string;
      description: string;
      homepageUrl?: string;
      repositoryUrl?: string;
      license?: string;
      permissions: {
        id: string;
        label: string;
        description: string;
        required: boolean;
      }[];
      configFields: {
        key: string;
        label: string;
        description?: string;
        type: "string" | "number" | "boolean" | "select" | "json";
        required: boolean;
        defaultValue?: unknown;
        options: string[];
      }[];
      compatibility: {
        core: { minVersion: string; maxVersion?: string };
      };
    };
    reviewedAt: string | null;     // ISO timestamp
    createdAt: string;             // ISO timestamp
  }[]
}
```

#### Intentionally Omitted Manifest Fields

The following fields are **not** returned by the Hub endpoint (internal deployment knowledge):

- `navigation` — Hub-internal routing configuration
- `apiRoutes` — internal API handler paths
- `botHooks` — Discord bot hooks
- `pages` — Hub-internal page routes
- `migrations` — DB migration SQL
- `requiredEnv` — required environment variables
- `installNotes` — installation notes for Hub admins

#### Filtering

Only submissions with `status = 'approved'` are returned, sorted by `createdAt DESC`.

---

## Error Handling (Frontend)

`pages/apps/index.vue` and `pages/apps/[id].vue` use `useFetch`. On error, an inline error message is shown. On empty result, an inline "no apps" or "not found" message is displayed.

`pages/index.vue` uses `@nuxt/content` to load landing data from `content/landing.md`.

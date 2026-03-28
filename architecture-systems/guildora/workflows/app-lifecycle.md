# Workflow: App Lifecycle

This workflow describes how installed apps currently behave in the repository.

## App Manifest Source of Truth

Manifests are validated through `packages/shared/src/types/app-manifest.ts`.

Supported manifest sections include:

- metadata
- permissions
- navigation
- pages
- API routes
- bot hooks
- config fields
- required environment variables
- install notes
- migrations
- compatibility

## Sideload Install

Route:

- `POST /api/admin/apps/sideload`

Flow:

1. Admin submits a GitHub manifest URL.
2. The server normalizes the URL.
3. The server fetches the manifest JSON.
4. The manifest is validated with `safeParseAppManifest`.
5. Manifest-declared API handlers are bundled as standalone CJS entrypoints (relative imports included).
6. If bot hooks are declared, `src/bot/hooks.ts` is bundled as a standalone CJS entrypoint (relative imports included).
7. Manifest pages are fetched as raw SFC source and stored for dynamic page rendering.
8. Optional app locale bundles (`src/i18n/en.json`, `src/i18n/de.json`) are fetched and stored when present.
9. Additional source files under `src/components/`, `src/composables/`, and `src/utils/` are auto-discovered (via GitHub Tree API or filesystem walk) and stored raw in the code bundle, enabling relative imports from Vue page SFCs. The optional `includes` manifest field can declare extra files beyond auto-discovered directories (max 50 files, 2 MB total).
10. The row is inserted or updated in `installed_apps`.
10. Status is set based on the `activate` flag.
11. The in-memory app registry is refreshed.

## Activation and Deactivation

Routes:

- `PUT /api/admin/apps/:appId/config`
- `POST /api/apps/:appId/activate`
- `POST /api/apps/:appId/deactivate`
- `PUT /api/admin/apps/:appId/status`

Only active apps are included when:

- loading app registry state
- generating app-provided navigation
- registering bot hooks

## Navigation Contribution

Active app manifests can contribute:

- rail entries
- grouped panel entries
- role-gated navigation items

The hub app merges them with core navigation in `/api/apps/navigation`.

## Bot Hook Contribution

Active app manifests can declare bot hooks. The bot loads them at startup through `loadInstalledAppHooks()`.

Current behavior:

- hooks execute from bundled `src/bot/hooks.ts` code stored in `installed_apps.code_bundle`
- invalid manifests are skipped
- hook execution is sandboxed by an error boundary so one app cannot crash the bot runtime

## Current Limits

- sideloaded API and bot bundles only support relative imports within the app repository
- runtime `require(...)` for external packages is not available in sideloaded API and bot execution
- app pages are loaded dynamically from stored SFC source; locale messages are fetched separately from stored `src/i18n/{locale}.json`
- Vue page SFCs support relative imports for `.vue`, `.ts`, `.js`, and `.json` files stored in the code bundle; client-side `.ts` is transpiled via Babel (complex TypeScript features like const enums or namespace merging are not supported)
- auto-discovered includable files are capped at 50 files and 2 MB total raw size

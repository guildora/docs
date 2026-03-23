# Architecture

## Overview

Nuxt 4 app inside a pnpm monorepo (`apps/web`).

| App | Port | Technology |
|---|---|---|
| `apps/web` | 3004 | Nuxt 4, @nuxt/content, Tailwind + DaisyUI |

No port conflicts with Guildora (3000/3002/3003/3050).

## Role in the Overall Infrastructure

```
Browser
  └─ Hub (iframe)
       └─ marketplace.domain.com (apps/web, port 3004)
            └─ /api/marketplace/apps   (Proxy → Hub API)
                  └─ hub:3003/api/marketplace/apps
                       └─ PostgreSQL app_marketplace_submissions
```

Landing page content is loaded from local `content/landing.md` via `@nuxt/content` — no external CMS dependency.

## apps/web — Nuxt Frontend

### Framework and Directory

- **Nuxt 4** with `srcDir: app/` (default in Nuxt 4)
- Vue application code: `apps/web/app/` (pages, components, layouts, assets)
- Server code: `apps/web/server/` (Nitro API routes)
- Content: `apps/web/content/` (Nuxt Content data files)
- Configuration: `apps/web/nuxt.config.ts`, `apps/web/tailwind.config.ts`

### Pages

| Route | Description |
|---|---|
| `/` | Landing Page – content from `@nuxt/content` |
| `/apps` | Marketplace listing – fetches from Hub via server proxy |
| `/apps/:id` | App detail – same fetch, filtered by `appId` |

### Data Flow

#### Landing Page (`/`)

```
SSR request for /
  └─ pages/index.vue
       └─ queryCollection('landing')
            └─ content/landing.md (YAML frontmatter)
                 └─ Typed by content.config.ts schema
                      └─ Data passed to landing components as props
```

1. `pages/index.vue` queries the `landing` content collection.
2. `@nuxt/content` reads `content/landing.md` and validates against the schema in `content.config.ts`.
3. Data is passed to landing components (`HeroSection`, `FeatureGrid`, etc.) as props.

#### Marketplace (`/apps`, `/apps/:id`)

1. Browser calls `/api/marketplace/apps` (same-origin Nitro route).
2. Nitro handler reads `config.hubApiUrl` from private runtime config.
3. Handler calls `{hubApiUrl}/api/marketplace/apps` server-side via `$fetch`.
4. Hub API returns list of approved apps (CORS headers, 60s cache).
5. Response is forwarded to the browser.

The browser never learns the internal Hub URL.

### Runtime Config (Web)

```ts
runtimeConfig: {
  hubApiUrl: "http://localhost:3003",   // server-only
  public: {
    githubUrl: "#",
    hubUrl:    "http://localhost:3003"
  }
}
```

Environment variable overrides:
- `NUXT_HUB_API_URL` → `config.hubApiUrl`
- `NUXT_PUBLIC_GITHUB_URL` → `config.public.githubUrl`
- `NUXT_PUBLIC_HUB_URL` → `config.public.hubUrl`

### Component Hierarchy

```
default.vue (Layout)
├── AppNavbar
├── <slot> (Page)
│   ├── pages/index.vue
│   │   ├── HeroSection
│   │   ├── FeatureGrid
│   │   ├── HowItWorks
│   │   ├── MarketplaceTeaser
│   │   └── SelfHostCta
│   ├── pages/apps/index.vue
│   │   ├── AppSearch
│   │   └── AppCard (n×)
│   │       └── PermissionBadge
│   └── pages/apps/[id].vue
│       └── PermissionBadge (n×)
└── AppFooter
```

## Build and Development

```sh
pnpm install
pnpm dev        # starts app (Turbo)
pnpm build      # production build
pnpm typecheck  # type-check

# Single app
pnpm --filter @marketplace/web dev
```

## Dependencies on Hub

The web app works without the Hub (empty marketplace, landing still served from content files). The server proxy returns an error shown in the UI.

## Integration with Hub

The Hub must have set:
```sh
NUXT_PUBLIC_MARKETPLACE_EMBED_URL=https://marketplace.domain.com
MARKETPLACE_CORS_ORIGIN=https://marketplace.domain.com
```

The Hub renders the marketplace app in an `<iframe>` – no Hub frontend changes required.

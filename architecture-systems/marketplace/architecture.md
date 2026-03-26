# Architecture

## Overview

Full-stack Nuxt 4 application with GitHub OAuth, own PostgreSQL database, developer portal, and admin panel.

| App | Port | Technology |
|---|---|---|
| Root app | 3004 | Nuxt 4, Drizzle ORM, nuxt-auth-utils, @nuxt/content, @nuxt/icon |
| `apps/web` | — | Public-only landing page (legacy, no auth/DB) |

No port conflicts with Guildora (3000/3002/3003/3050).

## Role in the Overall Infrastructure

```
Browser
  ├─ marketplace.domain.com (port 3004)
  │    ├─ /                         ← Landing page (@nuxt/content)
  │    ├─ /apps                     ← Public marketplace listing
  │    ├─ /login                    ← GitHub OAuth login
  │    ├─ /developer/*              ← Developer portal (auth required)
  │    ├─ /admin/*                  ← Admin panel (org members only)
  │    └─ /docs/*                   ← Documentation pages
  │
  ├─ /api/auth/github               ← GitHub OAuth callback
  ├─ /api/marketplace/apps           ← Public app listing
  ├─ /api/developer/*                ← Developer APIs (auth required)
  ├─ /api/admin/*                    ← Admin APIs (org members only)
  ├─ /api/webhooks/github            ← GitHub webhook handler
  ├─ /api/submit-app                 ← Public app submission
  └─ /api/feedback                   ← Public feedback → Linear

Hub
  └─ GET marketplace.domain.com/api/hub/apps
       └─ Marketplace API (CORS + cache)

Database
  └─ PostgreSQL (own instance, separate from Guildora)
       └─ Drizzle ORM
```

Landing page content is loaded from local `content/landing.md` via `@nuxt/content` — no external CMS dependency.

## Root App — Nuxt Full-Stack Application

### Framework and Directory

- **Nuxt 4** with `srcDir: app/` (default in Nuxt 4)
- Vue application code: `app/` (pages, components, layouts, assets, middleware, composables)
- Server code: `server/` (Nitro API routes, database, utils)
- Content: `content/` (Nuxt Content data files)
- Database migrations: `drizzle/`
- Configuration: `nuxt.config.ts`, `tailwind.config.ts`, `drizzle.config.ts`, `content.config.ts`

### Pages

| Route | Auth | Description |
|---|---|---|
| `/` | Public | Landing Page – content from `@nuxt/content` |
| `/apps` | Public | Marketplace listing – approved community extensions |
| `/apps/:id` | Public | App detail page |
| `/login` | Public | GitHub OAuth login page |
| `/developer` | Developer | Developer dashboard – registered apps overview |
| `/developer/apps` | Developer | Manage registered apps |
| `/developer/submissions` | Developer | Track submission status |
| `/developer/notifications` | Developer | View notifications |
| `/developer/profile` | Developer | Edit developer profile |
| `/admin` | Admin | Admin dashboard – review stats |
| `/admin/submissions` | Admin | Review pending submissions |
| `/docs` | Public | Documentation index |
| `/docs/:slug` | Public | Documentation page (Nuxt Content) |

### Authentication

- **GitHub OAuth** via `nuxt-auth-utils`
- Scopes: `read:user`, `user:email`, `read:org`
- Admin detection: checks membership in GitHub org (`NUXT_GITHUB_ORG_NAME`)
- Session stored in encrypted HTTP-only cookies (max age: 7 days)
- Server helpers: `requireDeveloperSession(event)`, `requireAdminSession(event)`

### Database

Own PostgreSQL database with Drizzle ORM. Tables:

| Table | Purpose |
|---|---|
| `developers` | GitHub-authenticated developer accounts |
| `app_registry` | Registered apps with repository URL and webhook config |
| `app_submissions` | Version submissions (status: pending → in_review → approved/rejected/changes_requested) |
| `app_reviews` | Human and AI reviews with verdict and feedback |
| `developer_notifications` | Developer notification inbox |
| `github_webhook_events` | Stored GitHub webhook payloads |

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

#### Marketplace (`/apps`, `/apps/:id`)

1. Browser calls `/api/marketplace/apps` (same-origin Nitro route).
2. Nitro handler queries `app_registry` + `app_submissions` for published apps with approved submissions.
3. Response forwarded to browser.

#### Developer Portal (`/developer/*`)

1. Route middleware checks GitHub OAuth session via `requireDeveloperSession`.
2. Developer CRUD operations against `app_registry`, `app_submissions`.
3. App registration creates an `app_registry` entry linked to developer.
4. Submissions go through review flow (pending → in_review → verdict).

#### Admin Panel (`/admin/*`)

1. Route middleware checks admin status via `requireAdminSession` (GitHub org membership).
2. Admin can view all submissions, trigger AI code reviews, approve/reject.
3. AI reviews call Anthropic Claude API and store results in `app_reviews`.

#### Hub Integration

1. Hub calls `GET /api/hub/apps` on Marketplace.
2. Returns published apps with latest approved submission, CORS headers, 60s cache.

#### GitHub Webhooks

1. GitHub sends webhook to `POST /api/webhooks/github` on new releases.
2. Handler validates signature, stores event, optionally creates new submission.

#### Feedback & Public Submission

1. `POST /api/feedback` — anonymous feedback, creates Linear issue.
2. `POST /api/submit-app` — public app submission (no auth required).

### Runtime Config

```ts
runtimeConfig: {
  hubCorsOrigin: "*",
  linearApiKey: "",
  databaseUrl: "",
  githubOrgName: "guildora",
  aiReviewApiKey: "",
  aiReviewModel: "claude-sonnet-4-20250514",
  session: { maxAge: 604800 },
  oauth: { github: { clientId: "", clientSecret: "" } },
  public: { githubUrl: "#" }
}
```

### Component Hierarchy

```
default.vue (Layout)
├── AppNavbar (with auth state)
├── <slot> (Page)
│   ├── pages/index.vue (Landing)
│   │   ├── HeroSection
│   │   ├── FeatureGrid
│   │   ├── HowItWorks
│   │   ├── MarketplaceTeaser
│   │   └── SelfHostCta
│   ├── pages/apps/index.vue (Marketplace)
│   │   ├── AppSearch
│   │   └── AppCard (n×)
│   │       └── PermissionBadge
│   ├── pages/developer/* (Developer Portal)
│   │   └── Developer-specific components
│   └── pages/admin/* (Admin Panel)
│       └── Admin-specific components
├── FeedbackModal
└── AppFooter
```

## apps/web — Public-Only Frontend (Legacy)

Separate Nuxt app inside `apps/web/` that serves only the public landing page and marketplace listing. Has no authentication, no database access, and no developer/admin functionality. Data comes from `@nuxt/content` and the Hub API.

This is the legacy version of the marketplace frontend. The root app now handles all functionality including the landing page.

## Build and Development

```sh
pnpm install
pnpm dev              # start dev server on port 3004
pnpm build            # production build
pnpm typecheck        # type-check
pnpm db:generate      # generate Drizzle migrations
pnpm db:migrate       # run pending migrations
pnpm db:studio        # open Drizzle Studio
```

## Integration with Hub

- Hub can embed Marketplace UI via:
  - `NUXT_PUBLIC_MARKETPLACE_EMBED_URL=https://marketplace.domain.com`
- Marketplace allows Hub API consumption via:
  - `NUXT_HUB_CORS_ORIGIN=https://hub.domain.com`

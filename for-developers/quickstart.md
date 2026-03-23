# Quickstart

Build and sideload your first Guildora app in under 15 minutes.

## Prerequisites

- A GitHub account with a public repository
- Access to a Guildora instance as Admin
- Node.js 20+ and pnpm installed locally

## 1. Clone the App Template

```bash
git clone https://github.com/guildora/guildora-app-template.git my-app
cd my-app

# Remove template history and start fresh
rm -rf .git
git init
git add .
git commit -m "Initial commit"
```

## 2. Update the Manifest

Edit `manifest.json`:

```jsonc
{
  "id": "my-app",              // unique kebab-case identifier
  "name": "My App",
  "version": "1.0.0",
  "author": "your-github-username",
  "description": "What your app does",
  "repositoryUrl": "https://github.com/you/my-app",
  "license": "MIT"
}
```

Key constraints:
- `id` must be `kebab-case`, no spaces, no uppercase
- `pages[].path` must start with `/apps/`
- `apiRoutes[].path` must start with `/api/apps/`

## 3. Add Your Logic

- **`src/pages/*.vue`** — Vue components rendered in the Hub
- **`src/api/*.ts`** — Nitro API handlers
- **`src/bot/hooks.ts`** — Bot event handlers
- **`src/i18n/`** — Translation strings (EN + DE)

See [Hub Integration](./hub-integration.md) and [Bot Integration](./bot-integration.md) for details.

## 4. Push to GitHub

```bash
git remote add origin https://github.com/you/my-app.git
git push -u origin main
```

The repository must be **public** for sideloading.

## 5. Sideload for Testing

1. Open your Guildora instance as Admin
2. Go to **Admin → Apps → Sideload**
3. Enter your GitHub repo URL: `https://github.com/you/my-app`
4. Click **Load** → review the app details
5. Click **Install** then **Activate**

Your app is now live for your guild.

## 6. Iterate

Push changes to GitHub, then in Admin → Apps → click your app → **Refresh** to pull the latest version.

## Next Steps

- [Manifest Reference](./manifest.md) — all available fields
- [Hub Integration](./hub-integration.md) — Vue pages, API routes
- [Bot Integration](./bot-integration.md) — event hooks, slash commands
- [Submission](./submission.md) — publish to the Marketplace
- [agent-development.md](./agent-development.md) — developing with AI assistance

# Manifest Reference

The `manifest.json` file at the repo root is required. It is fetched by the Guildora host during sideloading.

## Full Schema

```jsonc
{
  // ── Identity ─────────────────────────────────────────────────────────
  "id": "my-app",                    // required | kebab-case, globally unique
  "name": "My App",                  // required | display name
  "version": "1.0.0",               // required | semver
  "author": "github-username",       // required | GitHub username or org
  "description": "Short description", // required | shown in marketplace list
  "repositoryUrl": "https://github.com/...", // required for marketplace
  "license": "MIT",                  // optional

  // ── Compatibility ─────────────────────────────────────────────────────
  "compatibility": {
    "core": { "minVersion": "0.1.0" }
  },

  // ── Navigation ────────────────────────────────────────────────────────
  "navigation": {
    "rail": [/* see below */],
    "panelGroups": [/* see below */]
  },

  // ── Permissions ───────────────────────────────────────────────────────
  "permissions": [/* see below */],

  // ── Pages ─────────────────────────────────────────────────────────────
  "pages": [/* see below */],

  // ── API Routes ────────────────────────────────────────────────────────
  "apiRoutes": [/* see below */],

  // ── Bot Hooks ─────────────────────────────────────────────────────────
  "botHooks": ["onVoiceActivity"],

  // ── Bot Commands ──────────────────────────────────────────────────────
  "botCommands": [/* see below */],

  // ── Config Fields ─────────────────────────────────────────────────────
  "configFields": [/* see below */],

  // ── Environment ───────────────────────────────────────────────────────
  "requiredEnv": [
    { "key": "API_SECRET", "description": "Secret for external API" }
  ],

  // ── Install notes ─────────────────────────────────────────────────────
  "installNotes": "Plain text shown to admin after install."
}
```

## navigation.rail

```jsonc
{
  "id": "my-rail",           // unique, referenced by panelGroups
  "icon": "star",            // Material Symbols icon name
  "label": "My App",         // tooltip text
  "to": "/apps/my-app",      // landing route
  "order": 50,               // position (lower = higher, 0-100)
  "requiredRoles": ["user"]  // minimum role to see this item
}
```

## navigation.panelGroups

```jsonc
{
  "id": "my-group",          // required | unique within the app
  "railItemId": "my-rail",   // must match a rail item id
  "order": 0,                // optional | position (lower = first)
  "title": "My App",         // optional | section heading
  "items": [
    {
      "id": "my-page",
      "label": "Page Name",
      "to": "/apps/my-app/page",
      "requiredRoles": ["user"]  // optional
    }
  ]
}
```

**Note:** Use `items` (not `entries`) in `panelGroups`.

### User-without-submenu pattern

When plain users should navigate directly (no panel), but moderators see a submenu:

```jsonc
"navigation": {
  "rail": [
    { "id": "my-app", "to": "/apps/my-app", "requiredRoles": ["user"] }
  ],
  "panelGroups": [{
    "railItemId": "my-app",
    "items": [
      { "id": "my-app-main", "to": "/apps/my-app", "requiredRoles": ["moderator"] },
      { "id": "my-app-admin", "to": "/apps/my-app/admin", "requiredRoles": ["admin"] }
    ]
  }]
}
```

Result: plain users click the rail item → direct navigation. Moderators get a panel.

## permissions

```jsonc
{ "key": "read:member", "description": "Read member profiles" }
```

Available keys: `read:member`, `write:member`, `read:messages`, `write:messages`, `read:voice`, `read:roles`, `write:roles`

## pages

```jsonc
{
  "id": "my-page",                  // unique
  "path": "/apps/my-app/page",      // must start with /apps/
  "file": "src/pages/page.vue",     // relative to repo root
  "requiredRoles": ["user"]         // optional
}
```

## apiRoutes

```jsonc
{
  "id": "my-route",                        // unique
  "method": "GET",                          // GET | POST | PUT | DELETE | PATCH
  "path": "/api/apps/my-app/data",          // must start with /api/apps/
  "file": "src/api/data.get.ts",            // relative to repo root
  "requiredRoles": ["user"]                 // optional
}
```

File naming convention: `<name>.<method>.ts`

## botHooks

Array of event names. All available hooks:

- `onVoiceActivity` — member joins/leaves/moves in a voice channel
- `onRoleChange` — member's roles are updated
- `onMessage` — a message is sent in a text channel
- `onMemberJoin` — a new member joins the guild
- `onInteraction` — a slash command is used

## botCommands

Slash commands registered with Discord when the app is activated:

```jsonc
{
  "name": "my-command",           // required | slash command name (lowercase, no spaces)
  "description": "Does something", // required | shown in Discord command picker
  "nameLocalizations": {           // optional
    "de": "mein-befehl"
  },
  "descriptionLocalizations": {    // optional
    "de": "Macht etwas"
  }
}
```

`botCommands` requires `"onInteraction"` in `botHooks` and an exported `onInteraction` handler in `src/bot/hooks.ts`.

## configFields

```jsonc
// type: string
{ "key": "greeting", "type": "string", "label": "Greeting", "default": "Hello" }

// type: number
{ "key": "limit", "type": "number", "label": "Limit", "default": 10, "min": 1, "max": 100 }

// type: boolean
{ "key": "enabled", "type": "boolean", "label": "Enable", "default": true }

// type: select
{ "key": "tier", "type": "select", "label": "Tier", "options": ["a","b","c"], "default": "a" }
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `id` contains uppercase or spaces | Use only `kebab-case` |
| `pages[].path` doesn't start with `/apps/` | Must be `/apps/your-app-id/...` |
| `apiRoutes[].path` doesn't start with `/api/apps/` | Must be `/api/apps/your-app-id/...` |
| `panelGroups[].railItemId` doesn't match a `rail[].id` | IDs must match exactly |
| Bot hook name typo | Check capitalization: `onVoiceActivity` not `onVoiceactivty` |
| Using `entries` in `panelGroups` | Use `items` instead |
| Missing `onInteraction` when using slash commands | Add to `botHooks` and export from `hooks.ts` |

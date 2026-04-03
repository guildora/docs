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

  // ── Homepage ──────────────────────────────────────────────────────────
  "homepageUrl": "https://...",      // optional | project homepage

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

  // ── Includes (optional) ──────────────────────────────────────────────
  "includes": [
    "src/lib/custom-parser.ts"
  ],

  // ── Migrations ────────────────────────────────────────────────────────
  "migrations": [
    "migrations/001_init.sql"
  ],

  // ── Environment ───────────────────────────────────────────────────────
  "requiredEnv": [
    "API_SECRET"
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
{
  "id": "read:member",              // required | unique permission identifier
  "label": "Read member profiles",  // required | short display label
  "description": "Access to read member profile data", // required | longer explanation
  "required": false                 // optional | defaults to false
}
```

## pages

```jsonc
{
  "id": "my-page",                       // required | unique
  "path": "/apps/my-app/page",           // required | must start with /apps/
  "title": "My Page",                    // required | page display title
  "component": "src/pages/page.vue",     // optional | relative path to Vue SFC source
  "requiredRoles": ["user"]              // optional
}
```

## apiRoutes

```jsonc
{
  "method": "GET",                          // required | GET | POST | PUT | DELETE | PATCH
  "path": "/api/apps/my-app/data",          // required | must start with /api/apps/
  "handler": "src/api/data.get.ts",         // required | relative path to handler file
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
{
  "key": "greeting",                     // required | unique config key
  "label": "Greeting",                   // required | display label
  "description": "The welcome message",  // optional
  "type": "string",                      // required | string | number | boolean | select | json
  "required": false,                     // optional | defaults to false
  "defaultValue": "Hello",              // optional | initial value
  "options": []                          // optional | choices for select type
}
```

Examples by type:

```jsonc
// type: string
{ "key": "greeting", "type": "string", "label": "Greeting", "defaultValue": "Hello" }

// type: number
{ "key": "limit", "type": "number", "label": "Limit", "defaultValue": 10 }

// type: boolean
{ "key": "enabled", "type": "boolean", "label": "Enable", "defaultValue": true }

// type: select
{ "key": "tier", "type": "select", "label": "Tier", "options": ["a","b","c"], "defaultValue": "a" }

// type: json
{ "key": "advanced", "type": "json", "label": "Advanced Config" }
```

## includes

Optional array of repo-relative file paths to include in the code bundle beyond auto-discovered files. During sideloading, files under `src/components/`, `src/composables/`, and `src/utils/` are automatically collected. Use `includes` for files outside those directories.

```jsonc
"includes": [
  "src/lib/custom-parser.ts",
  "src/data/defaults.json"
]
```

- Max 50 entries
- Supported extensions: `.vue`, `.ts`, `.js`, `.json`
- Total raw size limit: 2 MB (across all includable files)

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `id` contains uppercase or spaces | Use only `kebab-case` |
| `pages[].path` doesn't start with `/apps/` | Must be `/apps/your-app-id/...` |
| `pages[]` missing `title` | `title` is required for every page |
| `apiRoutes[].path` doesn't start with `/api/apps/` | Must be `/api/apps/your-app-id/...` |
| Using `file` instead of `component` in pages | The field is `component` (optional) |
| Using `file` instead of `handler` in apiRoutes | The field is `handler` |
| `panelGroups[].railItemId` doesn't match a `rail[].id` | IDs must match exactly |
| Bot hook name typo | Check capitalization: `onVoiceActivity` not `onVoiceactivty` |
| Using `entries` in `panelGroups` | Use `items` instead |
| Missing `onInteraction` when using slash commands | Add to `botHooks` and export from `hooks.ts` |

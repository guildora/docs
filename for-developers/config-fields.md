# Config Fields

Config fields are settings editable by guild admins in the App Settings UI. They allow admins to customize your app's behavior per guild without changing code.

## Declaring Config Fields

In `manifest.json`:

```jsonc
"configFields": [
  {
    "key": "welcomeEnabled",
    "type": "boolean",
    "label": "Enable welcome messages",
    "description": "Send a welcome message when a new member joins.",
    "default": true
  },
  {
    "key": "welcomeMessage",
    "type": "string",
    "label": "Welcome message",
    "description": "Use {username} as a placeholder.",
    "default": "Welcome to the server, {username}!"
  },
  {
    "key": "announcementChannelId",
    "type": "string",
    "label": "Announcement channel ID",
    "description": "Discord channel ID where announcements are posted."
  },
  {
    "key": "maxMembers",
    "type": "number",
    "label": "Max tracked members",
    "default": 100,
    "min": 1,
    "max": 1000
  },
  {
    "key": "tier",
    "type": "select",
    "label": "Notification tier",
    "options": ["minimal", "standard", "verbose"],
    "default": "standard"
  }
]
```

## Accessing Config Values

### In API Handlers

```typescript
export default defineEventHandler(async (event) => {
  const { config } = event.context.guildora

  const channelId = config.announcementChannelId ?? null  // always provide a fallback
  const enabled = config.welcomeEnabled ?? true
  const message = config.welcomeMessage ?? 'Welcome!'
})
```

### In Vue Pages

```typescript
const config = useAppConfig()
const enabled = config.welcomeEnabled  // true/false
const channelId = config.announcementChannelId
```

### In Bot Hooks

```typescript
export async function onMemberJoin(payload: MemberJoinPayload, ctx: BotContext) {
  const enabled = (ctx.config.welcomeEnabled as boolean) ?? true
  const channelId = (ctx.config.announcementChannelId as string) ?? null

  if (!enabled || !channelId) return
  await ctx.bot.sendMessage(channelId, `Welcome, <@${payload.memberId}>!`)
}
```

## Rules

- **Always use fallbacks** — config values can be `undefined` if not yet set by the admin.
- **Never remove a config field key in an update** — existing guild configs reference it.
- **Never change a config field key** — treat keys as immutable once published.
- Use `description` to explain the expected format (e.g., "Discord channel ID", "seconds").

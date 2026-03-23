# Bot Integration

All bot hooks are exported from a single file: `src/bot/hooks.ts`.

## Example

```typescript
// src/bot/hooks.ts
import type {
  BotContext,
  VoiceActivityPayload,
  RoleChangePayload,
  MemberJoinPayload,
  InteractionPayload
} from '@guildora/app-sdk'

export async function onVoiceActivity(payload: VoiceActivityPayload, ctx: BotContext) {
  if (payload.action === 'join') {
    await ctx.db.set(`member:${payload.memberId}`, { lastSeen: new Date().toISOString() })
  }
}

export async function onMemberJoin(payload: MemberJoinPayload, ctx: BotContext) {
  const channelId = ctx.config.announcementChannelId as string | undefined
  if (channelId) {
    await ctx.bot.sendMessage(channelId, `Welcome, <@${payload.memberId}>!`)
  }
}

export async function onInteraction(payload: InteractionPayload, ctx: BotContext) {
  if (payload.commandName !== 'my-command') return
  await ctx.db.set(`audit:${payload.memberId}:${Date.now()}`, {
    command: 'my-command',
    at: payload.occurredAt
  })
}
```

## BotContext

```typescript
interface BotContext {
  guildId: string
  config: Record<string, any>   // configField values
  db: AppDb                      // same KV store as in API routes
  bot: BotClient                 // bot API client
}
```

## BotClient Methods

```typescript
ctx.bot.sendMessage(channelId: string, content: string): Promise<void>
ctx.bot.addRole(memberId: string, roleId: string): Promise<void>
ctx.bot.removeRole(memberId: string, roleId: string): Promise<void>
ctx.bot.getMember(memberId: string): Promise<Member>
```

## Payload Types

```typescript
interface VoiceActivityPayload {
  memberId: string
  action: 'join' | 'leave' | 'move'
  channelId: string
  previousChannelId?: string  // only on 'move'
  durationSeconds?: number    // only on 'leave' and 'move'
}

interface RoleChangePayload {
  memberId: string
  addedRoles: string[]
  removedRoles: string[]
  allRoles: string[]
}

interface MessagePayload {
  memberId: string
  channelId: string
  messageId: string
  content: string
}

interface MemberJoinPayload {
  memberId: string
  joinedAt: string  // ISO date string
}

interface InteractionPayload {
  memberId: string
  commandName: string
  occurredAt: string  // ISO date string
}
```

## Slash Commands

Declare commands in `manifest.botCommands` and handle them in `onInteraction`:

```jsonc
// manifest.json
"botCommands": [
  { "name": "my-command", "description": "Does something" }
],
"botHooks": ["onInteraction"]
```

```typescript
// src/bot/hooks.ts
export async function onInteraction(payload: InteractionPayload, ctx: BotContext) {
  if (payload.commandName !== 'my-command') return
  // handle command
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Hook not exported from `src/bot/hooks.ts` | Export must be a named export matching the hook name exactly |
| Forgetting `await` on `db.set` | Always `await` async db calls |
| Accessing `payload.durationSeconds` on 'join' | Only available on 'leave' and 'move' |
| Missing `onInteraction` export when using `botCommands` | Must be exported and listed in `botHooks` |

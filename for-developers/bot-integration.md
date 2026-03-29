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
  InteractionPayload,
  MessagePayload
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

export async function onMessage(payload: MessagePayload, ctx: BotContext) {
  if (payload.content.startsWith('!hello')) {
    await ctx.bot.sendMessage(payload.channelId, `Hello, <@${payload.memberId}>!`)
  }
}
```

## BotContext

```typescript
interface BotContext {
  config: Record<string, unknown>   // configField values set by admin
  db: AppDb                          // scoped KV store for this app
  bot: BotClient                     // Discord bot client
  botUserId: string                  // the bot's own Discord user ID
}
```

## BotClient Methods

```typescript
interface BotClient {
  sendMessage(channelId: string, content: string): Promise<void>
  createVoiceChannel(name: string, parentId: string): Promise<{ id: string; name: string } | null>
  deleteChannel(channelId: string): Promise<boolean>
  getChannel(channelId: string): Promise<{ id: string; name: string; parentId: string | null; memberCount: number | null } | null>
  setChannelName(channelId: string, name: string): Promise<boolean>
  moveMemberToChannel(memberId: string, channelId: string): Promise<boolean>
  getMemberVoiceChannelId(memberId: string): Promise<string | null>
  listVoiceChannelsByCategory(categoryId: string): Promise<Array<{ id: string; name: string; parentId: string | null; memberCount: number | null }>>
  listTextChannels(): Promise<Array<{ id: string; name: string }>>
  listAllChannels(): Promise<Array<{ id: string; name: string; type: string; parentId: string | null }>>
}
```

## Payload Types

```typescript
interface VoiceActivityPayload {
  guildId: string
  memberId: string
  action: 'join' | 'leave' | 'move'
  channelId: string | null
  previousChannelId: string | null
  durationSeconds?: number    // only on 'leave' and 'move'
  occurredAt: string          // ISO date string
}

interface RoleChangePayload {
  guildId: string
  memberId: string
  addedRoles: string[]
  removedRoles: string[]
}

interface MemberJoinPayload {
  guildId: string
  memberId: string
  username: string
  joinedAt: string | null     // ISO date string
}

interface InteractionPayload {
  guildId: string | null
  memberId: string
  commandName: string
  channelId: string | null
  occurredAt: string          // ISO date string
}

interface MessagePayload {
  guildId: string
  channelId: string
  messageId: string
  memberId: string
  content: string
  occurredAt: string          // ISO date string
  replyToMessageId?: string   // if this is a reply
  replyToUserId?: string      // author of the replied-to message
  attachments?: MessageAttachment[]
}

interface MessageAttachment {
  url: string
  contentType: string
  filename: string
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
| Using `addRole`/`removeRole` on BotClient | These methods do not exist; use the hub API for role management |

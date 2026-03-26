# Subsystem: Discord Bot

## Purpose

`apps/bot` connects the platform to Discord. It is responsible for voice tracking, guild sync operations, slash-command setup, and app hook dispatch.

## Main Entry Points

- `src/index.ts`
- `src/events/*`
- `src/utils/internal-sync-server.ts`
- `src/utils/app-hooks.ts`

## Registered Event Handlers

- `ready`
- `interactionCreate`
- `guildMemberAdd`
- `voiceStateUpdate`

## Implemented Bot Features

### Slash Command

- `setup` posts or updates an informational embed pointing users to the hub app

### Voice Tracking

- listens to `voiceStateUpdate`
- ignores bot users
- ignores unchanged voice states
- can exclude an AFK channel
- opens, closes, or splits voice sessions in `voice_sessions`
- emits manifest-based `onVoiceActivity` hooks

### Guild Member Hooks

- `guildMemberAdd` emits `onMemberJoin`
- interaction handling emits `onInteraction`

### Internal Sync Server

The bot runs an internal HTTP server guarded by `BOT_INTERNAL_TOKEN`. The hub app uses it for:

- syncing a user's nickname and manageable permission-role Discord roles
- listing guild roles
- listing members for a guild role
- fetching one guild member
- removing manageable Discord roles from a guild member
- syncing user-selected community Discord roles against an allowlist

## App Hook System

Active installed apps can declare bot hooks in their manifest. The current implementation registers lightweight logging handlers per active app for:

- `onRoleChange`
- `onMemberJoin`
- `onVoiceActivity`
- `onInteraction`

This is a manifest-driven extension seam, not a full plugin runtime.

## Important Constraints

- the bot needs guild member and voice-state intents
- the bot bridge is an internal integration surface, not a public API
- role sync only manages a defined set of role names or allowed role IDs depending on the endpoint

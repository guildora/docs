# Matrix Bot Subsystem

## Overview

The Matrix bot (`apps/matrix-bot`) is an **optional** service that connects Guildora to a Matrix homeserver. It is only needed when the community admin has configured a Matrix platform connection.

## Technology

- **Runtime:** Node.js
- **Library:** `matrix-bot-sdk`
- **Default Port:** 3051 (internal HTTP API)
- **Connects to:** Any Matrix homeserver (Synapse, Dendrite, Conduit) via access token

## Configuration

| Variable | Description |
|---|---|
| `MATRIX_HOMESERVER_URL` | Full URL of the Matrix homeserver (e.g. `https://matrix.example.org`) |
| `MATRIX_ACCESS_TOKEN` | Bot user's access token |
| `MATRIX_SPACE_ID` | Matrix space ID the community is bound to (e.g. `!abc:matrix.example.org`) |
| `BOT_INTERNAL_PORT` | HTTP API port (default: 3051) |
| `BOT_INTERNAL_TOKEN` | Shared secret for Hub communication |
| `DATABASE_URL` | PostgreSQL connection string (shared with Hub) |

## Internal HTTP API

The Matrix bot exposes the **same HTTP API contract** as the Discord bot, enabling the Hub's `platformBridge` to route requests transparently:

| Route | Method | Response |
|---|---|---|
| `/internal/health` | GET | `{ ok, status, platform: "matrix" }` |
| `/internal/guild/roles` | GET | Power level definitions mapped to role-like objects |
| `/internal/guild/channels/list` | GET | Rooms in the configured Matrix space |
| `/internal/guild/members/:mxid` | GET | Member profile (display name, avatar) |
| `/internal/sync-user` | POST | Set power level based on permission roles |

All routes require `Authorization: Bearer {BOT_INTERNAL_TOKEN}`.

## Event Handling

- **Room messages:** Emits `onMessage` app hooks for text messages in space rooms
- **Member joins:** Emits `onMemberJoin` app hooks when users join rooms in the space
- **Power level changes:** Tracked for `onRoleChange` hooks (planned)

All hook payloads include `platform: "matrix"`.

## Key Differences from Discord Bot

| Concept | Discord | Matrix |
|---|---|---|
| Server/Community | Guild (ID: snowflake) | Space (ID: `!room:server`) |
| Roles | Named roles with IDs | Power levels (0-100) |
| Channels | Channels with types | Rooms in space hierarchy |
| User ID | Snowflake number | MXID (`@user:server`) |
| Slash commands | Yes (registered via Gateway) | Not applicable (Matrix has no slash command API) |
| Rich embeds | Yes (buttons, selects) | Not applicable (Matrix uses formatted messages) |
| Voice tracking | Via Gateway events | Via Element Call room membership (limited) |
| Nickname management | Bot can set per-guild | Cannot set for other users |

## Docker Deployment

```yaml
services:
  matrix-bot:
    image: guildora/matrix-bot:latest
    restart: unless-stopped
    environment:
      - MATRIX_HOMESERVER_URL=${MATRIX_HOMESERVER_URL}
      - MATRIX_ACCESS_TOKEN=${MATRIX_ACCESS_TOKEN}
      - MATRIX_SPACE_ID=${MATRIX_SPACE_ID}
      - BOT_INTERNAL_PORT=3051
      - BOT_INTERNAL_TOKEN=${BOT_INTERNAL_TOKEN}
      - DATABASE_URL=${DATABASE_URL}
    networks:
      - internal
```

The Matrix bot is **not started by default** — it should only be deployed when a Matrix platform connection is configured.

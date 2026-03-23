# Guildora Documentation

Guildora is an open-source community platform built for Discord communities. It consists of a **Hub** (web app for members), a **Bot** (Discord event processor), a **Marketplace** (community extensions), and a **CMS** (editorial content).

This is the central documentation repository. All documentation for the Guildora platform lives here.

---

## Who are you?

### I'm a community member using Guildora
→ [for-users/](./for-users/README.md) — Learn about the platform and its features.

### I want to run my own Guildora instance
→ [for-hosters/](./for-hosters/README.md) — Setup, configuration, and deployment guides.

### I want to build apps for Guildora
→ [for-developers/](./for-developers/README.md) — App SDK, manifest reference, Hub/Bot integration, and how to publish to the Marketplace.

### I'm a platform maintainer
→ [internals/](./internals/README.md) — Architecture, domain model, API contracts, and internal workflows.

---

## Design System

→ [DESIGN_SYSTEM.md](./DESIGN_SYSTEM.md) — Tokens, typography, color palette, component standards. Single source of truth for all Guildora apps.

---

## For AI Agents

→ [AGENTS.md](./AGENTS.md) — Cross-repo conventions and platform context for AI coding assistants.

**MCP access (Claude Code):** Register the `guildora-docs` server in `~/.claude/settings.json` to access all docs via `read_file` and `list_files` tools directly from your editor.

```json
{
  "mcpServers": {
    "guildora-docs": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/Newguild/docs"]
    }
  }
}
```

---

## Architecture Decisions

→ [decisions/](./decisions/) — Architectural decision records (ADRs).

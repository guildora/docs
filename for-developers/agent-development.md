# Developing Guildora Apps with AI Agents

Guildora apps follow a strict, well-defined structure (`manifest.json`, `src/pages/`, `src/api/`, `src/bot/hooks.ts`). This predictability makes AI coding assistants highly effective — they can reason about the full app without needing to explore an unknown codebase.

## Setup: What to Give the Agent

### The AGENTS.md File

Every Guildora app repo should have an `AGENTS.md`. The app template ships with a comprehensive one at:

https://github.com/guildora/docs/blob/main/for-developers/AGENTS.md

This file contains the full manifest schema, Hub integration guide, bot hook reference, design system rules, common mistakes, and a test checklist. Copy it into your repo as the starting point.

### MCP Access to Docs (Claude Code)

If you use Claude Code, register the `guildora-docs` MCP server once globally. This gives the agent direct access to all Guildora documentation from any working directory:

```json
// ~/.claude/settings.json
{
  "mcpServers": {
    "guildora-docs": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/Newguild/docs"
      ]
    }
  }
}
```

The agent can then call:
- `read_file("/for-developers/manifest.md")` — manifest reference
- `read_file("/for-developers/hub-integration.md")` — hub pages + API routes
- `read_file("/DESIGN_SYSTEM.md")` — design tokens and component rules
- `list_files("/for-developers")` — browse available docs

---

## Workflow: Task-Oriented Prompting

### Starting a New App

```
I want to build a Guildora app that tracks member birthdays and sends a bot message on the birthday.

Read the manifest schema and hub integration docs first, then scaffold:
- manifest.json with the appropriate fields
- src/pages/index.vue (admin config page, role: admin)
- src/api/birthday.get.ts and birthday.post.ts (role: user)
- src/bot/hooks.ts with onVoiceActivity (as a check-in hook)
- src/i18n/en.json and de.json
```

### Adding a Feature

```
Add a new API route POST /api/apps/my-app/announce that allows moderators
to send a message to the announcement channel.
Check the apiRoutes convention in the manifest docs before writing any code.
```

### Fixing a Bug

```
The onVoiceActivity hook isn't being called. Check the botHooks field in my manifest.json
against the hook name conventions in the bot integration docs.
```

### Before Submitting

```
Review my app against the test checklist in the developer AGENTS.md.
Check that all manifest fields are correct, all files exist, and spacing follows the 8px grid.
```

---

## Recommended Prompting Patterns

### Start Each Session

When starting a new session on an existing app:
```
Read AGENTS.md first. Then read manifest.json to understand the app structure.
What are we working on today?
```

### Scaffold from Manifest

Let the agent derive the implementation from `manifest.json` rather than writing code first:
```
Read manifest.json, then implement all pages, API routes, and bot hooks listed in it.
Follow the patterns in AGENTS.md exactly.
```

### Commit Recommendations

Ask the agent to recommend commits at checkpoints:
```
After completing each file or logical block, suggest a commit with a Conventional Commits message.
```

---

## Design System Rules for Agents

This is the most common source of mistakes in AI-generated Guildora app code.

**App pages run inside the Hub and inherit its theme.** The agent must:

| Rule | Detail |
|---|---|
| Use CSS variables | `var(--color-text-primary)`, `var(--color-accent)`, etc. — never hex |
| Use Hub CSS classes | `card`, `btn btn-primary btn-sm`, `alert alert-error` |
| Respect spacing grid | Only tokens: `1, 2, 3, 4, 6, 8, 12, 16, 24, 32` — no `*-5` |
| No DaisyUI sub-classes | `card-body`, `stat-title`, `stat-value` are **not defined** |
| No Tailwind color utils | `text-primary`, `bg-base-200`, `border-base-300` are **unreliable** |
| No font classes | DM Sans is injected globally — do not set any font |
| No imports for composables | `useI18n`, `useAuth`, `useFetch` are globally injected |

Full reference: https://github.com/guildora/docs/blob/main/DESIGN_SYSTEM.md

---

## Test Checklist (for Agents)

Before completing a task or recommending a commit, the agent should verify:

**manifest.json**
- [ ] Valid JSON (no trailing commas, no comments)
- [ ] `id` is `kebab-case`
- [ ] All `pages[].path` start with `/apps/`
- [ ] All `apiRoutes[].path` start with `/api/apps/`
- [ ] `panelGroups[].railItemId` matches a `rail[].id`
- [ ] `onInteraction` in `botHooks` if using `botCommands`

**Code**
- [ ] All `pages[].file` paths point to existing files
- [ ] All `apiRoutes[].file` paths point to existing files
- [ ] `src/bot/hooks.ts` exports all functions listed in `botHooks`
- [ ] Config values always have fallbacks (`config.key ?? defaultValue`)
- [ ] Bot hooks use `await` on all async db calls

**Design**
- [ ] No hardcoded hex colors — CSS variables only
- [ ] No `font-nunito` or any font class
- [ ] No DaisyUI compound sub-classes
- [ ] Spacing follows 8px grid (`p-6`, `gap-4`, `space-y-6`, `mb-4`, `mt-8`)
- [ ] No `*-5` spacing tokens

**i18n**
- [ ] Both `en.json` and `de.json` exist with identical key structure
- [ ] No hardcoded UI text in Vue templates

---

## Do's and Don'ts

### Do
- Feed `AGENTS.md` to the agent at the start of every session
- Let the agent scaffold code from `manifest.json` (manifest-first approach)
- Ask for commit recommendations after each logical block
- Have the agent run through the test checklist before marking a feature done
- Use the MCP `read_file` tool to access the latest doc versions

### Don't
- Let the agent use DaisyUI compound sub-classes — they are not defined in the Hub
- Let the agent import host composables — they are globally injected, imports break
- Let the agent hardcode hex colors — only CSS variables
- Let the agent use `#ff206e` as an accent color fallback — that is the Marketplace color, not the Hub
- Let the agent add `*-5` spacing — use the 8px grid only

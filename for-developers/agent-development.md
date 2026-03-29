# AI Agent Workflow

Guildora apps follow a strict, well-defined structure (`manifest.json`, `src/pages/`, `src/api/`, `src/bot/hooks.ts`). This predictability makes AI coding assistants highly effective — they can reason about the full app without needing to explore an unknown codebase.

## Read First

- `../AGENTS.md` — cross-repo rules and platform conventions
- `./index.md` — app model, lifecycle, and reading order
- `./manifest.md` — manifest schema
- `./hub-integration.md` — pages and API routes
- `./bot-integration.md` — bot hooks and slash commands
- `./config-fields.md` — admin configuration model
- `./i18n.md` — translation rules
- `./submission.md` — sideloading and marketplace publishing
- `../DESIGN_SYSTEM.md` — shared UI rules

### App Repo AGENTS.md

Every Guildora app repo should still have its own `AGENTS.md`.

Keep it repo-specific:
- current app purpose
- important implementation constraints
- key files and workflows
- links back to the canonical docs above
- project-specific test or release rules

### Accessing Docs (Claude Code)

All Guildora documentation is deployed at **https://guildora.github.io/docs** and accessible from any system — no local clone needed.

Claude Code agents can fetch any doc page directly:
- `https://guildora.github.io/docs/for-developers/manifest.html` — manifest reference
- `https://guildora.github.io/docs/for-developers/hub-integration.html` — hub pages + API routes
- `https://guildora.github.io/docs/DESIGN_SYSTEM.html` — design tokens and component rules
- `https://guildora.github.io/docs/for-developers/` — browse available docs

---

## Task-Oriented Prompting

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
Review my app against the test checklist in this document.
Check that all manifest fields are correct, all files exist, and spacing follows the 8px grid.
```

---

## Recommended Prompting Patterns

### Start Each Session

When starting a new session on an existing app:
```
Read the app repo's AGENTS.md first. Then read manifest.json to understand the app structure.
What are we working on today?
```

### Scaffold from Manifest

Let the agent derive the implementation from `manifest.json` rather than writing code first:
```
Read manifest.json, then implement all pages, API routes, and bot hooks listed in it.
Follow the platform docs and the current repo rules exactly.
```

### Commit Recommendations

Ask the agent to recommend commits at checkpoints:
```
After completing each file or logical block, suggest a commit with a Conventional Commits message.
```

---

## Design Rules for Agents

This is the most common source of mistakes in AI-generated Guildora app code.

**App pages run inside the Hub and inherit its theme.**

| Rule | Detail |
|---|---|
| Use CSS variables | `var(--color-text-primary)`, `var(--color-accent)`, etc. — never hex |
| Use Hub CSS classes | `card`, `btn btn-primary btn-sm`, `alert alert-error` |
| **Use `.field*` for all forms** | `.field`, `.field__control`, `.field__input`, `.field__select`, `.field__textarea` — never bare `.input`/`.select`/`.textarea` |
| Respect spacing grid | Only tokens: `1, 2, 3, 4, 6, 8, 12, 16, 24, 32` — no `*-5` |
| No Tailwind color utils | `text-primary`, `bg-base-200`, `border-base-300` are **unreliable** |
| No font classes | DM Sans is injected globally — do not set any font |
| No imports for composables | `useI18n`, `useAuth`, `useFetch`, `$fetch` are globally injected |
| `useFetch()` only takes plain strings | Do NOT pass `computed()` or `ref()` as URL — the polyfill coerces objects to `[object Object]`. For dynamic URLs use `$fetch()` directly |
| `useAppConfig()` is NOT available | Access config values through your own API endpoint instead |

### Why `.field*` for forms?

The Hub's input background uses `var(--color-field-bg)` which automatically adjusts to be one surface level above the container. Using bare `.input` or native `<input>` without `.field__control` results in invisible inputs on surface-2 backgrounds (the most common card level in app pages).

**Correct form pattern:**
```html
<div class="field">
  <label class="field__label" for="ch-id">Channel ID</label>
  <div class="field__control">
    <input id="ch-id" v-model="channelId" type="text" class="field__input" />
  </div>
  <span class="field__hint">Discord channel or category ID</span>
</div>
```

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
- [ ] All `pages[].component` paths point to existing files
- [ ] All `apiRoutes[].handler` paths point to existing files
- [ ] `src/bot/hooks.ts` exports all functions listed in `botHooks`
- [ ] Config values always have fallbacks (`config.key ?? defaultValue`)
- [ ] Bot hooks use `await` on all async db calls

**Design**
- [ ] No hardcoded hex colors — CSS variables only
- [ ] No `font-nunito` or any font class
- [ ] All form inputs use `.field__control` + `.field__input` / `.field__select` / `.field__textarea` — not bare `.input` / `.select` / `.textarea`
- [ ] Spacing follows 8px grid (`p-6`, `gap-4`, `space-y-6`, `mb-4`, `mt-8`)
- [ ] No `*-5` spacing tokens

**i18n**
- [ ] Both `en.json` and `de.json` exist with identical key structure
- [ ] No hardcoded UI text in Vue templates

---

## Do's and Don'ts

### Do
- Feed the app repo's `AGENTS.md` to the agent at the start of every session
- Let the agent scaffold code from `manifest.json` (manifest-first approach)
- Ask for commit recommendations after each logical block
- Have the agent run through the test checklist before marking a feature done
- Fetch the latest docs from https://guildora.github.io/docs when needed

### Don't
- Let the agent use bare `.input`, `.select`, `.textarea` classes for form fields — use `.field__control` + `.field__input` etc.
- Let the agent import host composables — they are globally injected, imports break
- Let the agent hardcode hex colors — only CSS variables
- Let the agent use `#ff206e` as an accent color fallback — that is the Marketplace color, not the Hub
- Let the agent add `*-5` spacing — use the 8px grid only

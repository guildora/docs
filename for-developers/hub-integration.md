# Hub Integration

## Vue Pages

Pages live in `src/pages/` and are standard Vue 3 SFCs.

```vue
<template>
  <div class="p-6 md:p-8">
    <div class="mb-8">
      <h1 class="text-3xl font-bold">{{ t('app.title') }}</h1>
      <p class="mt-2 text-sm opacity-60">{{ t('app.subtitle') }}</p>
    </div>

    <div v-if="pending" class="opacity-60">Loading...</div>
    <div v-else-if="error" class="alert alert-error">Failed to load.</div>

    <div v-else class="space-y-6">
      <div class="card">
        <div class="p-6">
          <p>{{ data?.membersTracked }} members tracked</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
// Composables provided by the Guildora host — NO imports needed
const { t } = useI18n()
const { user, hasRole } = useAuth()

// Fetch from your app's API route
const { data, pending, error } = await useFetch('/api/apps/my-app/overview')
</script>
```

### Available Composables (injected by host, no import needed)

- `useI18n()` — `{ t, locale }` for translations
- `useAuth()` — `{ user, hasRole, guildId }` for current user
- `useAppConfig()` — key/value object of all configField values for this guild
- `useFetch()` — standard Nuxt `useFetch`, scoped to host origin

### Design System

App pages inherit the Hub's theme automatically. Use Hub CSS classes and variables — do **not** add external CSS.

```
var(--color-surface-2)       card/panel background
var(--color-line)            border/divider color
var(--color-text-primary)    primary text
var(--color-text-secondary)  secondary/muted text
var(--color-accent)          primary accent (guild-configurable)
var(--color-success)         success green
var(--color-warning)         warning amber
var(--color-error)           error red
```

Key Hub CSS classes:
- `<div class="card">` — card surface (automatically dark)
- `<div class="p-6">` — standard card inner padding
- `<button class="btn btn-primary btn-sm">` — primary button
- `<div class="alert alert-error">` — error alert
- `<input class="input w-full">` — text input

**Do NOT:**
- Import host composables (`useI18n`, `useAuth`, etc.) — they are globally injected
- Use DaisyUI compound sub-classes: `card-body`, `card-title`, `stats`, `stat-title`, `stat-value`
- Use Tailwind color utilities like `text-primary`, `bg-base-200` — use CSS variables
- Set any font class — DM Sans is injected globally
- Hardcode hex colors — use CSS variables

Full design rules: [DESIGN_SYSTEM.md](https://github.com/guildora/docs/blob/main/DESIGN_SYSTEM.md)

---

## Nitro API Routes

Handlers are standard Nitro `defineEventHandler` functions in `src/api/`.

```typescript
// src/api/overview.get.ts
export default defineEventHandler(async (event) => {
  const { guildId, userId, userRoles, config, db } = event.context.guildora

  const members = await db.list('member:')

  return { membersTracked: members.length }
})
```

### `event.context.guildora` Fields

| Field | Type | Description |
|---|---|---|
| `guildId` | `string` | Current guild ID |
| `userId` | `string \| undefined` | Authenticated user ID |
| `userRoles` | `string[]` | Roles of the current user |
| `config` | `Record<string, any>` | configField values for this guild |
| `db` | `AppDb` | Guild-scoped key-value store |

### AppDb API

```typescript
await db.get(key: string): Promise<any | null>
await db.set(key: string, value: any): Promise<void>
await db.delete(key: string): Promise<void>
await db.list(prefix: string): Promise<{ key: string; value: any }[]>
```

### Auth Guards

`requiredRoles` in the manifest is enforced before the handler runs. For additional checks inside the handler:

```typescript
const { userRoles } = event.context.guildora
const privileged = ['moderator', 'admin', 'superadmin']
if (!userRoles.some(r => privileged.includes(r))) {
  throw createError({ statusCode: 403, message: 'Forbidden' })
}
```

### Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing fallback for config values | Always use `config.key ?? defaultValue` |
| Returning non-serializable values | Return plain objects/arrays/primitives |
| Not handling missing `userId` | Check `userId` before using it |
| Throwing raw errors | Use `createError({ statusCode, message })` |

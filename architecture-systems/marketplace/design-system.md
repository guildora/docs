# Design System: Marketplace Dark

Full public design system reference: https://github.com/guildora/docs/blob/main/DESIGN_SYSTEM.md

## Theme

```html
<html data-theme="marketplace-dark">
```

Set in `nuxt.config.ts` → `app.head.htmlAttrs`. Dark mode only.

## Color Palette (CSS Variables)

| Variable | Value | Usage |
|---|---|---|
| `--color-surface-0` / `--color-surface-1` | `#0A0A0A` | Page background |
| `--color-surface-2` | `#111111` | Cards, elevated surfaces |
| `--color-surface-3` | `#1A1A1A` | Deeper surfaces |
| `--color-surface-4` | `#222222` | Hover states |
| `--color-accent` | `#ff206e` | Primary color, CTAs, highlights |
| `--color-accent-light` | `#ff5c94` | Hover on accent |
| `--color-accent-muted` | `rgba(255, 32, 110, 0.2)` | Accent backgrounds |
| `--color-line` | `#1F1F1F` | Dividers, borders |
| `--color-text-primary` | `#FAFAFA` | Main text |
| `--color-text-secondary` | `#A3A3A3` | Secondary text |
| `--color-text-tertiary` | `#525252` | Helper texts, labels |

### Portal Accent (Developer/Admin)

The portal layout uses `data-section="portal"` to scope a blue accent (`#2563EB`) for developer and admin sections.

## Typography

- Font: **DM Sans** (Google Fonts), fallback: `"Segoe UI"`, `"Helvetica Neue"`, sans-serif
- Weights: 400, 500, 600, 700
- Headlines: letter-spacing `-0.02em`

## Shadow Utilities

| Class | Token | Usage |
|---|---|---|
| `shadow-sm` | `var(--shadow-sm)` | Small cards |
| `shadow-md` | `var(--shadow-md)` | Standard cards |
| `shadow-lg` | `var(--shadow-lg)` | Modals, hero |
| `shadow-card` | `var(--shadow-md)` | Card shorthand |
| `shadow-elevated` | `var(--shadow-lg)` | Elevated shorthand |

## CSS Classes

### Buttons
```html
<button class="btn btn-primary">    <!-- Accent-filled, rounded -->
<button class="btn btn-secondary">  <!-- surface-3 background -->
<button class="btn btn-outline">    <!-- Transparent, border -->
<button class="btn btn-ghost">      <!-- No background, hover fade -->
<button class="btn btn-sm">         <!-- Smaller variant -->
```

Buttons have `border-radius: 0.5rem` (8px).

### Cards
```html
<div class="card p-6">  <!-- surface-2 background, 12px radius -->
```

## Layout Convention

The default layout has **no** container. Each section controls its own width:

```html
<section class="py-16">
  <div class="mx-auto max-w-7xl px-4 sm:px-6">...</div>
</section>
```

## Rules

- No light mode — `data-theme` is fixed to `marketplace-dark`
- Buttons use `border-radius: 0.5rem`, not pill shape
- Cards use `border-radius: 0.75rem` with `border: 1px solid var(--color-line)`
- No neuomorphism shadows, scanlines, or glow effects
- Do not introduce external icon libraries without prior agreement

## Sources

- Tokens and overrides: `app/assets/css/main.css` (root app) or `apps/web/app/assets/css/main.css` (legacy)
- Tailwind theme extensions: `tailwind.config.ts` (root app) or `apps/web/tailwind.config.ts` (legacy)

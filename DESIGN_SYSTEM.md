# Design System – Clean SaaS

Global design system for all Guildora platform projects. This file is the single source of truth for design tokens, component standards, and visual rules.

**Replaces:** RetroMorphism design system (deprecated).
**Figma Reference:** SaaS Landing Page Template (`96pEtcb3Or2jbaL3Bs7Nb5`)

---

## Philosophy

- Simple, reduced, minimalist — no visual noise
- SaaS quality: professional, trustworthy, modern
- Optimal UX for non-technical users
- Every element has a clear purpose — nothing without function
- **Surfaces use shadow for depth, never borders** — borders are reserved for UI chrome (tab underlines, outline buttons, dividers, spinner rings)

---

## Typography

- **Font:** DM Sans (not Inter, not Roboto, not Arial, not Nunito)
- **Weights:** 400 (body), 500 (UI), 600 (subheadings), 700 (headlines)
- **Scale:** 12 / 14 / 16 / 20 / 24 / 32 / 48 / 64px
- **Line-height:** body 1.6, headlines 1.1–1.2
- **Letter-spacing:** headlines -0.02em to -0.04em (tight, modern)
- **Max 2 font families** per project

---

## Colors

### Dark Mode (default for all apps)

The Hub uses a granular surface scale (`--color-surface-0` through `--color-surface-5`) and `--color-line` for dividers. The Marketplace uses a similar but slightly different naming. Both follow the same visual intent.

| Token | Hub Variable | Value | Usage |
|---|---|---|---|
| Background | `--color-surface-0` / `--color-surface-1` | `#0A0A0A` | Page background |
| Surface | `--color-surface-2` | `#111111` | Cards, elevated surfaces |
| Surface elevated | `--color-surface-3` | `#1A1A1A` | Deeper surfaces |
| Surface hover | `--color-surface-4` | `#222222` | Hover states |
| Surface active | `--color-surface-5` | `#2A2A2A` | Active states |
| Field background | `--color-field-bg` | `var(--color-surface-3)` | Input field background |
| Divider | `--color-line` | `#1F1F1F` | Dividers, borders |
| Text primary | `--color-text-primary` | `#FAFAFA` | Main text |
| Text secondary | `--color-text-secondary` | `#A3A3A3` | Secondary text |
| Text tertiary | `--color-text-tertiary` | `#525252` | Helper texts, labels |
| Text disabled | `--color-text-disabled` | `rgba(255,255,255,0.3)` | Disabled elements |

### Light Mode (Hub only)

| Token | Variable | Value |
|---|---|---|
| Background | `--color-surface-0` / `--color-surface-1` | `#FAFAFA` |
| Surface | `--color-surface-2` | `#FFFFFF` |
| Surface elevated | `--color-surface-3` | `#F4F4F5` |
| Divider | `--color-line` | `#E5E5E5` |
| Text primary | `--color-text-primary` | `#0A0A0A` |
| Text secondary | `--color-text-secondary` | `#525252` |
| Text tertiary | `--color-text-tertiary` | `#A3A3A3` |

### Project-Specific Accent Colors

| Project | Accent | Hex | Hover | Subtle (10% opacity) |
|---|---|---|---|---|
| Guildora Hub (default) | Violet | `#7C3AED` | `#6D28D9` | `rgba(124,58,237,0.1)` |
| Guildora Landing | Violet | `#7C3AED` | `#6D28D9` | `rgba(124,58,237,0.1)` |
| Marketplace Landing | Magenta | `#ff206e` | `#e6195f` | `rgba(255,32,110,0.1)` |
| Marketplace Portal | Royal Blue | `#2563EB` | `#1D4ED8` | `rgba(37,99,235,0.1)` |

### Semantic Colors

| Token | Hex | Usage |
|---|---|---|
| Info | `#3B82F6` | Informational states |
| Success | `#22C55E` | Positive actions, confirmations |
| Warning | `#F59E0B` | Caution states |
| Error | `#EF4444` | Errors, destructive actions |

---

## Spacing (8px Grid)

Base unit: 8px

| Token | Value |
|---|---|
| `--space-1` | 4px |
| `--space-2` | 8px |
| `--space-3` | 12px |
| `--space-4` | 16px |
| `--space-6` | 24px |
| `--space-8` | 32px |
| `--space-12` | 48px |
| `--space-16` | 64px |
| `--space-24` | 96px |
| `--space-32` | 128px |

- Sections: min 80px vertical padding (desktop), 48px (mobile)
- Cards/containers inner padding: 24px

### App Utility Mapping (Tailwind)

Use these utility classes in Hub app pages to stay aligned with the 8px grid:

- Page shell: `p-6 md:p-8`
- Header block: `mb-8`; subtitle: `mt-2`
- Vertical section stack: `space-y-6`
- Card inner spacing: `p-6`
- Card title spacing: `mb-4`
- Grid and stat gaps: `gap-4`
- Stat label spacing: `mb-2`
- Action separator: `mt-8 pt-6 border-t`
- Allowed spacing tokens (preferred): `1, 2, 3, 4, 6, 8, 12, 16, 24, 32`
- Avoid odd tokens like `*-5` unless there is a hard UX reason

---

## Border Radius

| Element | Radius |
|---|---|
| Buttons | 8px |
| Cards | 12px |
| Modals/Panels | 16px |
| Pills/Tags | 9999px |
| Input fields | 8px |

---

## Shadows

Shadows replace borders for depth — they must be strong enough to stand alone on dark backgrounds.

| Token | Dark mode value | Light mode value | Usage |
|---|---|---|---|
| `--shadow-sm` | `0 1px 4px rgba(0,0,0,0.14), 0 1px 2px rgba(0,0,0,0.09)` | `0 1px 3px rgba(0,0,0,0.07), 0 1px 2px rgba(0,0,0,0.05)` | Inputs, badges, subtle elements |
| `--shadow-md` | `0 4px 20px rgba(0,0,0,0.22)` | `0 4px 16px rgba(0,0,0,0.08)` | Cards |
| `--shadow-lg` | `0 8px 40px rgba(0,0,0,0.30)` | `0 8px 32px rgba(0,0,0,0.10)` | Elevated panels, modals, sidebars |

No neuomorphism. No hard-edged shadows. No blur radius > 40px.

---

## Component Standards

### Buttons

- **Primary:** Accent-colored background, white text
- **Secondary:** Border + transparent background
- **Ghost:** Text-only, no background
- **Padding:** 12px 20px (medium), 10px 16px (small), 14px 24px (large)
- **Radius:** 8px
- **Font-weight:** 500

### Cards

- No border — `--shadow-md` provides depth
- Background: `--color-surface-elevated`
- Radius: 12px
- Padding: 24px
- Hover: slightly elevated shadow

### Input Fields

- Height: auto (padding-based)
- Padding: `0.7rem 1rem`
- No border — `--shadow-sm` + `--color-surface-2` background provides definition
- Radius: 8px (`0.5rem`)
- Focus: shadow-based ring on the **container** via `:focus-within` — never `outline` on the inner element

### Forms

#### Field Anatomy

Every field consists of (top to bottom):
1. **Label** — uppercase, 0.75rem, 600 weight, `--color-text-secondary`
2. **Control wrapper** (`.field__control`) — shadow-based surface, contains the actual input
3. **Sub-row** (optional) — hint text left, character counter right

#### Field Gap

`gap-4` (16px) between all fields — never `gap-5` or `gap-6`.

#### States

| State | CSS modifier | Visual |
|---|---|---|
| Default | — | `--shadow-sm` on `--color-surface-2` |
| Hover | — | background `--color-surface-3` |
| Focus | — | container: `--shadow-sm, 0 0 0 2px rgba(accent, 0.30)` via `:focus-within` |
| Error | `.field--error` | container: `--shadow-sm, 0 0 0 2px rgba(239,68,68,0.30)` |
| Success | `.field--success` | container: `--shadow-sm, 0 0 0 2px rgba(34,197,94,0.30)` |
| Disabled | — | `opacity: 0.5`, `cursor: not-allowed` on inner element |

**Focus rule:** The ring always sits on `.field__control:focus-within` — never `outline` on `<input>`. This keeps surfaces border-free at all times.

**Field background rule:** Inputs always sit one surface level above their container via the `--color-field-bg` CSS variable. The variable is set automatically on `.bg-base-100/200/300`, `.surface-level-*`, and `.bg-surface-*` classes and cascades down to all `.field__control` children. Default (`:root`) is `--color-surface-2`.

#### Sub-row Elements

- **`.field__hint`** — helper text, shown unconditionally below the control; `0.75rem`, `--color-text-muted`
- **`.field__message`** — validation message (error or success text); `0.75rem`, colored by state
- **`.field__counter`** — character counter, right-aligned; `0.72rem`, `--color-text-muted`; turns `--color-warning` at 80%, `--color-error` at 100%

#### Field Structure — Input / Select / Textarea

Use the `UiInput`, `UiSelect`, `UiTextarea` components. Available props:

| Prop | Type | Purpose |
|---|---|---|
| `label` | `string` (required) | Label text (rendered uppercase) |
| `required` | `boolean` | Appends `*` to label |
| `hint` | `string` | Helper text below the field |
| `error` | `string` | Error message — activates `.field--error` ring |
| `showCounter` | `boolean` | Show character counter (requires `maxlength`) |
| `size` | `"md"\|"sm"\|"xs"` | Field size variant |

```html
<!-- UiInput with hint + error -->
<UiInput
  v-model="email"
  label="E-Mail"
  type="email"
  required
  hint="Wird nur für Benachrichtigungen genutzt"
  :error="errors.email"
/>

<!-- UiTextarea with counter -->
<UiTextarea
  v-model="bio"
  label="Bio"
  :maxlength="500"
  show-counter
  hint="Kurze Vorstellung"
/>
```

#### Field Structure — Toggle / Checkbox

Use `UiCheckbox`. The `label` prop becomes the section heading above; the `description` slot or prop becomes the inline checkbox label.

```html
<UiCheckbox
  v-model="enabled"
  label="Benachrichtigungen"
  description="E-Mail-Benachrichtigungen aktivieren"
  hint="Du kannst dies jederzeit ändern"
/>
```

#### Field Groups

Group related fields with `.field-group`. Use `.field-group__title` for a section heading (uses `border-bottom` on `--color-line` — the only allowed border in forms).

```html
<div class="field-group">
  <p class="field-group__title">Profil</p>
  <UiInput v-model="name" label="Name" />
  <UiInput v-model="email" label="E-Mail" type="email" />
</div>
```

#### Leading / Trailing Icons

Both `UiInput` and `UiTextarea` support a `#trailing` slot (and `#leading` slot) for icons:

```html
<UiInput v-model="search" label="Suche" placeholder="Suchen…">
  <template #trailing>
    <Icon name="lucide:search" />
  </template>
</UiInput>
```

#### Raw HTML Pattern (Marketplace / no Vue component)

When Vue components are not available, use the `.field*` CSS classes directly:

```html
<div class="field">
  <label class="field__label" for="name">App Name <span aria-hidden="true">*</span></label>
  <div class="field__control">
    <input id="name" class="field__input" type="text" required maxlength="150" />
  </div>
  <span class="field__hint">Maximal 150 Zeichen</span>
</div>
```

#### Rules

- Focus ring is always shadow-based on the container — no `outline` on inner elements
- No inline description paragraphs below labels — use `hint` prop or `.field__hint`
- Number / small inputs: fixed width (`w-32`); text inputs: full width (`w-full`)
- Save button separator: `mt-8 pt-6 border-t border-[var(--color-line)]`
- Card headings inside single-purpose settings cards are redundant with the page `<h1>` — omit them

### Navigation

- Sticky, backdrop-blur
- Max 6 visible nav items
- Height: ~64px

### Badges/Tags

- Small rounded pills (9999px radius)
- Accent color at 10–15% opacity as background
- Font-size: 12px

### Icons

- Lucide Icons or Hero Icons — consistent style
- Sizes: 16px / 20px / 24px

---

## Animations & Transitions

| Type | Duration | Easing |
|---|---|---|
| Standard | 150ms | ease-out |
| Cards/Hover | 200ms | ease |
| Page elements (fade-in) | 300ms | ease, staggered 50ms |

- No distracting loop animations except subtle background effects
- Scroll reveal: opacity 0→1, translateY 8px→0

---

## Layout

- **Max-width content:** 1200px, centered
- **Grid:** 12 columns (desktop), 4 columns (mobile)
- **Gutter:** 24px (desktop), 16px (mobile)
- **Sections:** alternate between surface/bg colors for natural separation
- **Hero:** always full-width statement with primary CTA + secondary CTA
- No horizontal scroll elements without explicit UX justification

---

## Accessibility & UX

- Contrast ratio min 4.5:1 for body text
- All interactive elements have focus styles
- Touch targets min 44x44px
- No color-only information without icon/text fallback
- Critical fonts preloaded, LCP optimized

---

## Forbidden Patterns

- NO generic purple gradients on white backgrounds
- NO excessive glassmorphism without context
- NO more than 2 font families per project
- NO components without hover/focus state
- NO walls of small text without visual hierarchy
- NO dark mode without full token support
- NO images/illustrations that don't match the tone
- NO hardcoded hex values outside of token/config files
- NO neuomorphism shadows (deprecated)
- NO sharp-cornered cards (border-radius: 0 is deprecated)
- NO pill-shaped buttons (9999px radius on buttons is deprecated — use 8px)
- NO borders on surfaces (cards, modals, panels, inputs, dropdowns) — use shadow instead
- NO redundant depth signals: if a shadow defines the surface, a border is noise

---

## CSS Variable Contract

All apps implement the same CSS variable names with project-specific values:

```css
:root {
  /* Surface scale (granular) */
  --color-surface-0: ...;  /* page background */
  --color-surface-1: ...;  /* page background (alias) */
  --color-surface-2: ...;  /* cards, elevated surfaces */
  --color-surface-3: ...;  /* deeper surfaces */
  --color-surface-4: ...;  /* hover states */
  --color-surface-5: ...;  /* active states */
  --color-field-bg: var(--color-surface-3);
  --color-line: ...;       /* dividers, borders */

  /* Text */
  --color-text-primary: ...;
  --color-text-secondary: ...;
  --color-text-tertiary: ...;
  --color-text-disabled: ...;

  /* Accent */
  --color-accent: ...;
  --color-accent-hover: ...;
  --color-accent-subtle: ...;

  /* Shadows */
  --shadow-sm: ...;
  --shadow-md: ...;
  --shadow-lg: ...;

  /* Typography */
  --font-display: 'DM Sans', sans-serif;
  --font-body: 'DM Sans', sans-serif;

  /* Radii */
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  --radius-full: 9999px;

  /* Transitions */
  --transition-fast: 150ms ease-out;
  --transition-base: 200ms ease;
  --transition-slow: 300ms ease;
}
```

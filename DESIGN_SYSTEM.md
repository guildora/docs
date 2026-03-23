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

| Token | Variable | Value |
|---|---|---|
| Background | `--color-bg` | `#0A0A0A` |
| Surface | `--color-surface` | `#111111` |
| Surface elevated | `--color-surface-elevated` | `#1A1A1A` |
| Border | `--color-border` | `#1F1F1F` |
| Text primary | `--color-text-primary` | `#FAFAFA` |
| Text secondary | `--color-text-secondary` | `#A3A3A3` |
| Text muted | `--color-text-muted` | `#525252` |

### Light Mode (Hub only)

| Token | Variable | Value |
|---|---|---|
| Background | `--color-bg` | `#FAFAFA` |
| Surface | `--color-surface` | `#FFFFFF` |
| Surface elevated | `--color-surface-elevated` | `#F4F4F5` |
| Border | `--color-border` | `#E5E5E5` |
| Text primary | `--color-text-primary` | `#0A0A0A` |
| Text secondary | `--color-text-secondary` | `#525252` |
| Text muted | `--color-text-muted` | `#A3A3A3` |

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

| Token | Value | Usage |
|---|---|---|
| `--shadow-sm` | `0 1px 3px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04)` | Subtle elements |
| `--shadow-md` | `0 4px 16px rgba(0,0,0,0.06)` | Cards |
| `--shadow-lg` | `0 8px 32px rgba(0,0,0,0.08)` | Elevated panels, modals |

No neuomorphism. No hard-edged shadows. No blur radius > 32px.

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

- Border: 1px solid `--color-border`
- Shadow: `--shadow-md`
- Radius: 12px
- Padding: 24px
- Hover: slightly elevated shadow

### Input Fields

- Height: auto (padding-based)
- Padding: 10px 14px
- Border: 1px solid `--color-border`
- Radius: 8px
- Focus: 2px ring with accent color at 30% opacity

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

---

## DaisyUI Theme Names

| App | Theme(s) |
|---|---|
| Guildora Hub | `guildora-dark`, `guildora-light` |
| Guildora Landing | `guildora-dark` |
| Marketplace | `marketplace-dark` |

---

## CSS Variable Contract

All apps implement the same CSS variable names with project-specific values:

```css
:root {
  --color-bg: ...;
  --color-surface: ...;
  --color-surface-elevated: ...;
  --color-border: ...;
  --color-text-primary: ...;
  --color-text-secondary: ...;
  --color-text-muted: ...;
  --color-accent: ...;
  --color-accent-hover: ...;
  --color-accent-subtle: ...;
  --shadow-sm: ...;
  --shadow-md: ...;
  --shadow-lg: ...;
  --font-display: 'DM Sans', sans-serif;
  --font-body: 'DM Sans', sans-serif;
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  --radius-full: 9999px;
  --transition-fast: 150ms ease-out;
  --transition-base: 200ms ease;
  --transition-slow: 300ms ease;
}
```

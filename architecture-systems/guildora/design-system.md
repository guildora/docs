# Guildora Design System (Internal)

This is the internal design system reference for Hub and Landing.

For the full public design system (used by app developers), see:
https://github.com/guildora/docs/blob/main/DESIGN_SYSTEM.md

## Scope

This design system governs:

- Hub: member area, moderator area, admin area (dark + light mode)
- Landing page (dark mode only)

## Theme Modes

- `guildora-dark`
- `guildora-light` (Hub only)

User preference is stored as `appearancePreference` with values: `light`, `dark`, `system`.
The active HTML theme is derived from resolved color mode.

## Theme Data Model

Theme values come from `theme_settings` and are normalized by:

- `apps/hub/server/utils/theme.ts`
- `apps/hub/utils/theme-colors.ts`

Theme configuration includes:

- dominant, secondary, accent colors
- semantic info, success, warning, and error colors
- content-tone flags for accent and semantic colors
- optional sidebar logo and its display size

Communities can customize their own colors via the dynamic theme system. Default accent: `#7C3AED` (Violet).

## Typography

- Font: **DM Sans** (400, 500, 600, 700)
- Headlines: letter-spacing `-0.02em`, line-height 1.1–1.2
- Body: line-height 1.6

## Color Palette

| Variable | Dark | Light | Usage |
|---|---|---|---|
| `--color-surface-0/1` | `#0A0A0A` | `#FAFAFA` | Page background |
| `--color-surface-2` | `#111111` | `#FFFFFF` | Cards, elevated surfaces |
| `--color-surface-3` | `#1A1A1A` | `#F4F4F5` | Deeper surfaces |
| `--color-accent` | `#7C3AED` | `#7C3AED` | Primary, CTAs |
| `--color-line` | `#1F1F1F` | `#E5E5E5` | Borders, dividers |
| `--color-text-primary` | `#FAFAFA` | `#0A0A0A` | Main text |
| `--color-text-secondary` | `#A3A3A3` | `#525252` | Secondary text |

## Shadows

| Token | Value |
|---|---|
| `--shadow-sm` | `0 1px 3px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04)` |
| `--shadow-md` | `0 4px 16px rgba(0,0,0,0.06)` |
| `--shadow-lg` | `0 8px 32px rgba(0,0,0,0.08)` |

## Border Radius

- Buttons/Inputs: `0.5rem` (8px)
- Cards: `0.75rem` (12px)
- Modals: `1rem` (16px)
- Pills/badges: `9999px`

## UI Components

Base internal UI wrappers live in `apps/hub/app/components/ui/`:

- `UiButton.vue` — Button variants (Primary, Secondary, Ghost, Outline, Info, Success, Warning, Error) and sizes (`xs`, `sm`, `md`)
- `UiInput.vue` — Input with label and optional trailing slot
- `UiSelect.vue` — Select field with label
- `UiTextarea.vue` — Multiline input with consistent field shell
- `UiCheckbox.vue` — Checkbox with label layout
- `UiFileInput.vue` — File upload field
- `UiColorInput.vue` — Combined color picker + hex text field
- `UiTag.vue` — Tag/badge display
- `UiDropdown.vue` — Dropdown container shell
- `UiOptionRow.vue` — Option rows within menus
- `UiModalTitle.vue` — Title block for modals

## Rules

- Prefer the Ui* wrappers above over raw form controls in new internal UI work
- Avoid page-specific hard-coded theme colors when a CSS variable or theme utility already exists
- Styling-only changes must not alter functional behavior
- No neuomorphism shadows, scanlines, or glow effects

## Sources

- Tokens and overrides: `apps/hub/app/assets/css/main.css`
- Tailwind theme extensions: `apps/hub/tailwind.config.ts`
- Dynamic theme logic: `apps/hub/utils/theme-colors.ts`

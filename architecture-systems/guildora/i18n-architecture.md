# i18n Architecture

This document describes active language-resolution and localization rules across web, hub, and bot.

## Supported Locales

- `en`
- `de`

## Effective Locale Resolution (Hub User Context)

Priority order:

1. `profiles.locale_preference`
2. `community_settings.default_locale`
3. fallback `en`

Relevant utilities:

- `apps/hub/utils/locale-preference.ts`
- `apps/hub/server/api/internal/locale-context.get.ts`

## Persistence Rules

- user preference stored canonically in `profiles.locale_preference`
- community default locale stored in `community_settings.default_locale`
- `profiles.custom_fields.localePreference` is legacy read fallback only

## Routing Behavior

- Nuxt i18n strategy: `prefix_except_default`
- English URLs unprefixed, German prefixed with `/de`
- hub applies locale middleware via `apps/hub/app/middleware/locale.global.ts`
- landing locale handling is independent in `apps/web`
- landing fetches published page data from Hub with the current locale

## UI Update Rule

UI must not switch locale optimistically before server confirms persistence.

Mechanism:

- update via `PUT /api/profile/locale`
- only switch locale after success

## Translation Rules

- all user-facing strings must use translation keys
- new keys must be added to both locales in the owning app:
  - hub: `apps/hub/i18n/locales/en.json` and `apps/hub/i18n/locales/de.json`
  - web: `apps/web/i18n/locales/en.json` and `apps/web/i18n/locales/de.json`
- raw backend error text should not be primary translated UI content

## Dynamic App Page Localization (Hub)

Sideloaded app pages (`/apps/:appId/...`) are rendered from stored SFC source.

- app-local messages are loaded from the sideloaded bundle at `src/i18n/{locale}.json`
- hub fetch endpoint: `GET /api/apps/:appId/_messages?locale=de|en`
- fallback order on app pages:
  1. app-local message key
  2. hub global message key
  3. raw key (default vue-i18n behavior when missing)
- app messages are scoped per rendered app page and are not merged into the global hub i18n store

## Landing Content Localization

Landing sections and footer pages store content as localized `Record<string, string>` JSONB fields (keyed by locale). The `landing_pages.enabled_locales` column controls which locales are active. The public endpoint `/api/public/landing` resolves the best locale from the request.

## Bot Localization

Bot message dictionary: `apps/bot/src/i18n/messages.ts`.

Current behavior:

- input starting with `de` resolves to German
- all others fallback to English

## Quality Checks

Current automated checks cover:

- locale-resolution utilities
- DE and EN locale key parity

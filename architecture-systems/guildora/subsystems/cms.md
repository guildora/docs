# Subsystem: CMS

## Purpose

`apps/cms` is the landing-page authoring system. It is separate from the internal hub and from the public landing renderer.

## Stack

- Payload CMS 3
- Next.js
- PostgreSQL via `@payloadcms/db-postgres`

## Payload Configuration

- config entry: `apps/cms/payload.config.ts`
- DB schema: `payload`
- locales: `en`, `de` (default `en`, fallback enabled)
- admin user collection: `users`
- internal SSO endpoint: `/api/sso`

## Collections

### `pages`

- localized title and SEO fields
- unique slug
- status: `draft` or `published`
- block-based `layout`
- public read access
- write access for CMS moderators and admins

### `media`

- public read access
- authenticated create, update, and delete
- generated `thumbnail` and `card` sizes
- localized `alt` field

### `users`

- Payload-authenticated CMS users
- CMS-local roles: `editor`, `moderator`, `admin`
- only CMS admins can manage CMS users directly

## Globals

### `site-settings`

- localized site name
- logo upload
- localized footer text
- localized social-link labels
- public read access
- write access for CMS moderators and admins

## Block Library

Current page blocks:

- `hero`
- `richText`
- `image`
- `gallery`
- `cta`
- `youtube`
- `discordInvite`

These blocks define landing content rendered in `apps/web`.

## CMS SSO

Hub creates short-lived signed tokens and opens CMS `/api/sso`.

CMS then:

- verifies token
- upserts CMS user tied to internal user ID
- assigns CMS role (`moderator` or `admin`)
- logs user into Payload and redirects to `/admin`

## Boundaries

- CMS does not own main community tables in `public`
- CMS access rules are separate from hub permission-role model
- landing content is fetched by `apps/web` via CMS HTTP API
- hub iframe embedding does not make CMS part of the hub runtime itself

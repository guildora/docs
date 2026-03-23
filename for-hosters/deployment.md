# Deployment

## Docker Compose (Recommended)

The repository ships with a `docker-compose.yml` and `docker-compose.override.yml`.

Services:
- `web` — Landing page
- `hub` — Internal Hub + API
- `cms` — Payload CMS
- `bot` — Discord bot

```bash
docker compose up -d
```

### Service Dependencies

- `web` depends on CMS (landing content) and hub (login handoff)
- `hub` depends on CMS (embedded SSO) and bot (Discord sync)
- `cms` stores uploads in the `cms_media` volume
- `bot` runs on the internal Docker network; its sync server is only accessible to the hub

### Caddy (Reverse Proxy)

The Docker Compose setup assumes an external `caddy` network for reverse proxying. Example Caddyfile:

```
hub.example.com {
    reverse_proxy hub:3003
}

example.com {
    reverse_proxy web:3000
}

cms.example.com {
    reverse_proxy cms:3002
}
```

## Migrations in Production

Run migrations as a controlled step before or during deployment:

```bash
pnpm db:migrate
```

Do not run seeds in production unless explicitly needed for role reconciliation.

## Secrets Management

Keep these values strong and private:

- `BOT_INTERNAL_TOKEN` — hub-to-bot communication
- `CMS_SSO_SECRET` — hub-to-CMS SSO
- `PAYLOAD_SECRET` — Payload admin
- `NUXT_SESSION_PASSWORD` — session encryption (min 32 chars)

## Known Operational Caveats

- Landing rendering depends on the CMS public URL being reachable from the `web` container
- CMS SSO requires both `CMS_SSO_SECRET` and a valid CMS base URL in the hub runtime config
- Discord sync endpoints depend on bot internal server availability
- The landing OAuth shim is a fallback — production OAuth should point directly to hub

## Marketplace (Separate Deployment)

The Marketplace app runs on port 3004 by default and is deployed separately. The hub embeds it via iframe. See [internals/marketplace/architecture.md](../internals/marketplace/architecture.md).

Required env vars for the marketplace:
```
NUXT_PUBLIC_HUB_URL=https://hub.example.com
NUXT_PUBLIC_GITHUB_URL=https://github.com/guildora/guildora
```

Hub side:
```
NUXT_PUBLIC_MARKETPLACE_EMBED_URL=https://marketplace.example.com
MARKETPLACE_CORS_ORIGIN=https://marketplace.example.com
```

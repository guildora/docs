# Deployment

## Docker Compose (Recommended)

The repository ships with a `docker-compose.yml` and `docker-compose.override.yml`.

Services:
- `web` — Landing page
- `hub` — Internal Hub + API + Landing page editor
- `bot` — Discord bot

```bash
docker compose up -d
```

### Service Dependencies

- `web` depends on hub (landing content via `/api/public/landing` and login handoff)
- `hub` depends on bot (Discord sync)
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
- `NUXT_SESSION_PASSWORD` — session encryption (min 32 chars)

## Known Operational Caveats

- Landing rendering depends on the Hub public URL being reachable from the `web` container
- Discord sync endpoints depend on bot internal server availability
- The landing OAuth shim is a fallback — production OAuth should point directly to hub

## Marketplace Integration (Managed Service)

Marketplace is not self-hosted in this stack. Treat it as an external platform service.

Hub side configuration remains optional and embed-focused:
```
NUXT_PUBLIC_MARKETPLACE_EMBED_URL=https://marketplace.example.com
```

For API/data contract details between Hub and Marketplace, see [architecture-systems/marketplace/api.md](../architecture-systems/marketplace/api.md).

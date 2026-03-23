# Guildora – Internal Documentation

Canonical technical documentation for the Guildora monorepo.

## Reading Order

1. [project-overview.md](./project-overview.md)
2. [architecture.md](./architecture.md)
3. [routing-and-navigation.md](./routing-and-navigation.md)
4. [domain-model.md](./domain-model.md)
5. [api-contracts.md](./api-contracts.md)
6. [permissions-matrix.md](./permissions-matrix.md)
7. [development-workflow.md](./development-workflow.md)
8. [setup-build-deploy.md](./setup-build-deploy.md)

## Core References

- Product scope and terminology: [project-overview.md](./project-overview.md)
- Runtime topology and integrations: [architecture.md](./architecture.md)
- Routes, layouts, middleware: [routing-and-navigation.md](./routing-and-navigation.md)
- Shared database schema: [domain-model.md](./domain-model.md)
- Nitro route contracts: [api-contracts.md](./api-contracts.md)
- Access rules per route: [permissions-matrix.md](./permissions-matrix.md)
- Localization rules: [i18n-architecture.md](./i18n-architecture.md)
- Internal UI design rules: [design-system.md](./design-system.md)

## Subsystems

- Hub app: [subsystems/web-app.md](./subsystems/web-app.md)
- Web landing: [subsystems/web-landing.md](./subsystems/web-landing.md)
- CMS: [subsystems/cms.md](./subsystems/cms.md)
- Discord bot: [subsystems/bot.md](./subsystems/bot.md)
- Shared package: [subsystems/shared-package.md](./subsystems/shared-package.md)

## Workflows

- Community applications: [workflows/applications.md](./workflows/applications.md)
- Discord role management: [workflows/discord-role-management.md](./workflows/discord-role-management.md)
- Admin user mirroring: [workflows/admin-user-mirroring.md](./workflows/admin-user-mirroring.md)
- CMS SSO: [workflows/cms-sso.md](./workflows/cms-sso.md)
- App lifecycle: [workflows/app-lifecycle.md](./workflows/app-lifecycle.md)
- Marketplace submissions: [workflows/marketplace-submissions.md](./workflows/marketplace-submissions.md)

## Maintenance Rules

- Update this directory when routes, tables, roles, or core workflows change.
- Prefer linking to existing documents instead of duplicating explanations.
- Mark legacy or inactive behavior explicitly instead of silently deleting history.

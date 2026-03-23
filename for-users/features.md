# Features

## Hub

The Hub is the web interface for community members. What you see depends on your role.

### For Members (`user`)
- **Profile** — View and edit your community profile, display name, and preferences
- **Member directory** — Browse other community members
- **Applications** — Apply for community membership
- **Role management** — Request Discord roles from the admin-curated list
- **Apps** — Access community-installed apps from the sidebar

### For Moderators (`moderator`)
- All member features
- **Applications review** — Review and process membership applications
- **User management** — Edit member data, manage community roles

### For Admins (`admin`)
- All moderator features
- **App management** — Install, activate, and configure community apps
- **Discord role mappings** — Define which Discord roles map to Guildora permission levels
- **Community settings** — Theme, locale, and general configuration
- **Admin sync** — Mirror Discord guild membership to the Hub database

### For Superadmins
- All admin features
- Server-level settings and platform configuration

---

## Marketplace

The Marketplace lists community apps built by third-party developers. Each app has:
- A description and version
- Declared permissions (what data it accesses)
- A link to the source repository

Apps are free and open source. To install an app, admins use the sideload feature (paste a GitHub URL) or browse approved submissions.

---

## Apps

Community apps extend the Hub with additional pages and bot functionality. Apps can:
- Add pages to the Hub sidebar navigation
- Add Nitro API routes
- React to Discord events (voice, messages, role changes, member joins)
- Store per-guild data
- Add slash commands to your Discord server

Apps are isolated per guild — data from one community's app is never visible to another.

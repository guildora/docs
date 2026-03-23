# What is Guildora?

Guildora is an open-source community platform built for Discord communities. It extends Discord with a web-based Hub that gives communities structured tooling for membership management, moderation, and community apps.

## How It Works

Guildora has two main runtime layers:

**Hub** — A web application that members access through their browser. It provides:
- A member profile and community directory
- Role management and membership applications
- Moderation tools for moderators and admins
- An integrated marketplace for community extensions (apps)

**Bot** — A Discord bot that keeps your Discord server and the Hub in sync:
- Tracks voice activity
- Syncs roles and membership between Discord and the Hub
- Powers bot hooks for community apps

## Login

You sign in to the Hub using your Discord account. No separate registration required — your Discord identity is the login.

## Community Roles

Guildora maps Discord roles to a structured permission hierarchy:

| Role | Access |
|---|---|
| `temporaer` | Guest or unverified — public pages only |
| `user` | Verified member — standard Hub access |
| `moderator` | Trusted member — can manage others, send announcements |
| `admin` | Guild admin — configures apps, manages roles and settings |
| `superadmin` | Platform owner — full access |

Your community determines how Discord roles map to these permission levels.

## Apps

Guildora communities can install apps that extend the Hub with new pages and features. Apps appear in the sidebar navigation and are configured by admins. They can also add bot commands.

Apps are published in the [Guildora Marketplace](../marketplace/overview.md) and installed directly from a GitHub URL.

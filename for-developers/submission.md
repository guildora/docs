# Submission: Sideloading & Marketplace Publishing

## Sideloading (for Testing)

Sideloading lets you load your app from a GitHub URL without going through marketplace review. Use this for development and testing.

### Steps

1. Push your code to a **public** GitHub repository
2. Ensure `manifest.json` is at the **root** of the default branch
3. In your Guildora instance, go to **Admin → Apps → Sideload**
4. Enter your repository URL: `https://github.com/username/my-app`
5. Click **Load** — the host fetches and validates `manifest.json`
6. Review the displayed app info and click **Install**
7. Click **Activate** to enable the app for your guild

### Updating a Sideloaded App

1. Push changes to GitHub
2. Go to **Admin → Apps** → click your app → **Refresh**

### Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `manifest.json not found` | File is not at repo root | Move `manifest.json` to root |
| `Invalid manifest` | JSON syntax error | Validate JSON (no trailing commas, no comments) |
| `Page file not found` | `pages[].file` path wrong | Check that file exists at that path |
| `Duplicate app ID` | Another app has the same `id` | Choose a unique `id` |

---

## Marketplace Publishing

Distribute your app through the Guildora Marketplace so any community can install it.

### Requirements Checklist

- [ ] `manifest.json` is complete: `repositoryUrl`, `license`, `description`
- [ ] Repository is public on GitHub
- [ ] App tested via sideloading on a live instance
- [ ] `README.md` explains what the app does and how to configure it
- [ ] No hardcoded guild IDs or user IDs
- [ ] All customizable behavior covered by `configFields`

### Submission

1. Ensure your repo is public and the default branch is production-ready
2. Go to the Guildora Marketplace developer portal
3. Click **Submit App**
4. Enter your repository URL
5. The review team checks:
   - Valid `manifest.json`
   - No malicious code
   - Working sideload install
   - Correct permission declarations
6. Once approved, your app appears in the Marketplace

### Versioning

Use semantic versioning in `manifest.json`:

```json
"version": "1.2.3"
```

When releasing a new version:
1. Update `"version"` in `manifest.json`
2. Commit and push
3. Guilds with your app installed see an update notification

**Tag releases in GitHub:**
```bash
git tag v1.2.3
git push --tags
```

### Backwards Compatibility Rules

- **Never remove a `configFields` key** — breaks existing guild config
- **Never change an `apiRoutes[].path`** — breaks existing integrations
- **Never rename `pages[].path`** — breaks existing navigation bookmarks
- Increment minor version for new features, patch for bug fixes, major for breaking changes

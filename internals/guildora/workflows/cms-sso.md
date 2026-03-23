# Workflow: CMS SSO

This workflow allows the internal hub app to open a logged-in Payload admin session.

## Actors

- hub app
- CMS app
- authenticated internal user

## Authorization Rules

- admins and superadmins always qualify for CMS SSO
- moderators qualify only if `cms_access_settings.allow_moderator_access` is enabled

## Hub Side

Route:

- `GET /api/cms/session-url`

Flow:

1. Require logged-in session.
2. Load current CMS access settings.
3. Resolve whether user is allowed to access CMS.
4. Resolve CMS role:
   - `admin` for internal `admin` and `superadmin`
   - `moderator` for internal `moderator`
5. Sign short-lived token with `CMS_SSO_SECRET`.
6. Return URL pointing to CMS `/api/sso` endpoint.

## CMS Side

Endpoint:

- `GET /api/sso`

Flow:

1. Verify token.
2. Upsert CMS user identified by synthetic email derived from internal user ID.
3. Assign CMS role from token.
4. Rotate CMS user password for login operation.
5. Log user into Payload.
6. Redirect to `/admin`.

## Security Notes

- token lifetime is intentionally short
- endpoint depends on `CMS_SSO_SECRET`
- hub must know a valid CMS base URL
- this is an internal trust boundary, not a public auth protocol

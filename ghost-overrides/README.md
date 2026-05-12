# Ghost Server Overrides

Files here are **not part of the Ghost theme** and are excluded from the theme zip by gulp. They are server-side overrides that get volume-mounted into the Ghost Docker container.

## Why this exists

Ghost's email newsletter system does not use the theme for rendering — emails are built entirely by Ghost's internal templates. To customize email output (e.g. replace full post content with an announcement + CTA), the internal template must be overridden via a Docker volume mount.

## Files

### `email-template.hbs`

Overrides Ghost's default newsletter email body template located inside the container at:

```
/var/lib/ghost/current/core/server/services/email-service/email-templates/template.hbs
```

**What it changes:** Replaces the full post content (`{{{html}}}`) with a single "See the full post" CTA button. Ghost's standard email wrapper (logo, post title, excerpt, feature image, footer, unsubscribe link) is unaffected — those come from the outer `email-wrapper.hbs` which is not overridden.

The button color, text color, and border radius are pulled from Ghost's newsletter design settings (configured in Ghost Admin → Newsletters → Edit).

## Deployment

1. Copy `email-template.hbs` to `/<path-to-docker-stacks>/ghost/ghost-overrides/` on the server.

2. Add to the `ghost` service's `volumes` in `compose.yml`:
   ```yaml
   - ./ghost-overrides/email-template.hbs:/var/lib/ghost/current/core/server/services/email-service/email-templates/template.hbs
   ```

3. Restart the container:
   ```
   docker compose restart ghost
   ```

## Maintenance note

If Ghost is updated to a major version, verify that the internal `template.hbs` structure hasn't changed before redeploying. The source can be checked at:

> [ghost/core/core/server/services/email-service/email-templates/template.hbs](https://github.com/TryGhost/Ghost/blob/main/ghost/core/core/server/services/email-service/email-templates/template.hbs)

# Ghost Server Overrides

Files here are **not part of the Ghost theme** and are excluded from the theme zip by gulp. They are server-side overrides that get volume-mounted into the Ghost Docker container.

## Why this exists

Ghost's email newsletter system does not use the theme for rendering — emails are built entirely by Ghost's internal templates. To customize email output (e.g. replace full post content with an announcement + CTA), the internal templates must be overridden via Docker volume mounts.

## Files

### `email-template.hbs`

Overrides Ghost's default newsletter email body template located inside the container at:

```
/var/lib/ghost/current/core/server/services/email-service/email-templates/template.hbs
```

**What it changes:** Replaces the full post content (`{{{html}}}`) with a "See the full post" CTA button. The button color, text color, and border radius are pulled from Ghost's newsletter design settings.

### `email-wrapper.hbs`

Overrides Ghost's email header/wrapper partial located inside the container at:

```
/var/lib/ghost/current/core/server/services/email-rendering/partials/email-wrapper.hbs
```

**What it changes:** Moves the excerpt from its default position (between the title and author/date line) to after the feature image. The newsletter's "Show excerpt" setting (Ghost Admin → Newsletters → Edit) still controls whether it appears — only the position changes.

## Deployment

1. Copy both files to `/opt/stacks/ghost/ghost-overrides/` on the server.

2. Add to the `ghost` service's `volumes` in `compose.yml`:
   ```yaml
   - ./ghost-overrides/email-template.hbs:/var/lib/ghost/current/core/server/services/email-service/email-templates/template.hbs
   - ./ghost-overrides/email-wrapper.hbs:/var/lib/ghost/current/core/server/services/email-rendering/partials/email-wrapper.hbs
   ```

3. Restart the container:
   ```
   docker compose restart ghost
   ```

## Maintenance note

If Ghost is updated to a major version, verify that the internal templates haven't changed before redeploying. Sources to check:

> [email-templates/template.hbs](https://github.com/TryGhost/Ghost/blob/main/ghost/core/core/server/services/email-service/email-templates/template.hbs)

> [email-rendering/partials/email-wrapper.hbs](https://github.com/TryGhost/Ghost/blob/main/ghost/core/core/server/services/email-rendering/partials/email-wrapper.hbs)

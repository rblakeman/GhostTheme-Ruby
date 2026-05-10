# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Do not run these automatically, let the developer handle that for now.

```bash
npm run dev      # build + watch with livereload (uses gulp default task)
npm run zip      # build + zip to dist/ruby.zip for Ghost upload
npm test         # run gscan theme validator
```

If `gulp` isn't found globally, run via `./node_modules/.bin/gulp <task>`.

`npm run dev` also runs the `locales` task on every build, which modifies `locales/context.json` and `locales/en.json` even though no `locales-local/` directory exists. Revert those files before committing if they change unexpectedly.

## Architecture

This is a **Ghost 5+ theme** using Handlebars templates with a Gulp/PostCSS build pipeline. However, the developer is using Ghost v6 for their project.

The developer forked this theme to add dark-mode support, access control for private posts, and clearer messaging. Tiers are for privacy only—users are manually added by admins, not paying.

### Template hierarchy

`default.hbs` is the layout shell (header, footer, `{{{body}}}`). All other templates extend it with `{{!< default}}` at the top: `post.hbs`, `index.hbs`, `page.hbs`, `author.hbs`, `tag.hbs`, `error.hbs`. Shared markup lives in `partials/`.

### Build pipeline

**CSS**: `assets/css/screen.css` is the PostCSS entry point. It uses `postcss-easy-import` to inline all `@import` statements from subdirectories (`general/`, `site/`, `blog/`, `misc/`). Output is minified to `assets/built/screen.css`. To add styles, create a new file and add an `@import` to `screen.css`.

**JS**: Gulp concatenates in this order: shared-theme-assets vendor libs → shared-theme-assets main.js → `assets/js/lib/*.js` (if exists) → `assets/js/main.js`. All are minified into `assets/built/main.min.js`. The shared-theme-assets layer provides `pagination()` and other Ghost utilities used in `main.js`.

`assets/built/` is excluded from git.

### CSS variables

`assets/css/general/basics.css` defines theme-local CSS variables (`--primary-text-color`, `--white-color`, etc.). `@tryghost/shared-theme-assets` also defines its own variables (`--color-darker-gray`, `--color-primary-text`, `--color-white`, etc.) used for headings and shared components. Both sets need to be overridden for dark mode.

### Dark mode

Dark mode is toggled by adding the `dark` class to `<html>` (`document.documentElement`). FOUC is prevented by an inline `<script>` in `<head>` (in `default.hbs`) that reads the `darkmode-prefered` cookie and sets the class before render.

The toggle buttons use class `.dm-toggle` and are rendered via two partials:
- `partials/darkmode-button.hbs` — icon-only button, placed in the mobile header brand area
- `partials/darkmode.hbs` — full toggle with the click handler and all iframe injection logic

**Ghost iframes (search, portal, comments)** are hardcoded light-mode. `darkmode.hbs` handles this by:
- **Search (sodo)**: injects a `<style>` into the iframe's `contentDocument` overriding Tailwind utility classes
- **Portal modal**: injects a `<style>` overriding Ghost Portal CSS variables (`--white`, `--black`, `--grey*`); also rewrites the email-preferences disclaimer text via `applyPortalContentOverride()` (theme-agnostic, runs in both modes)
- **Portal trigger** (floating avatar): injects a `<style>` overriding only accent/brand color variables
- **Comments**: uses `{{comments mode="auto"}}` in `post.hbs`; Ghost comments-ui reads `getComputedStyle(container).backgroundColor` of the parent of `#ghost-comments-root` to decide dark/light. `.gh-comments` must have an explicit (non-transparent) background color in CSS for this to work.

A `MutationObserver` on `document.body` watches for new iframes being added dynamically and applies overrides via `trackIframe()`.

**Portal content rewriting** (`applyPortalContentOverride`): Ghost Portal is a React SPA inside the popup iframe. Content like the email-preferences disclaimer is rendered on-demand when the user navigates to that view — it is not present in the DOM at iframe load time. The observer for content rewrites must target `doc.documentElement` (not `doc.body`), because React can replace/remount the body when navigating between portal screens. The "already observed" flag is stored on `doc.documentElement.dataset` (not `iframe.dataset`) so it resets correctly if the iframe srcdoc reloads and a fresh document is created.

### Ghost theme customization

`package.json` → `config.custom` defines the Ghost Design panel settings (`navigation_layout`, `title_font`, `body_font`, `show_related_posts`), accessed in templates as `@custom.*`.

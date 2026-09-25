# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static multi-page marketing website for F.M. srl (imbiancature, ristrutturazioni, pavimenti in resina). No build step, no package manager, no framework — plain HTML/CSS/vanilla JS served as-is. Deployed via GitHub Pages with a custom domain (see `CNAME`: `fm-srl.com`).

## Commands

There is no build/lint/test tooling in this repo. To preview locally, serve the directory with any static file server, e.g.:

```bash
python3 -m http.server 8000
```

Then browse `http://localhost:8000/`. Do not open `index.html` directly via `file://` — relative paths behave differently and some things (e.g. the Google Maps embed) expect an http(s) origin.

## Architecture and routing

The site is a set of hand-authored "pages", each its own directory with its own `index.html`, `script.js`, and `style.css`:

- `/` (root `index.html`) — homepage
- `/resine/` — resin/wood flooring page
- `/galleria/` — photo gallery with lightbox
- `/contatti/` — contact page with map embed

**Routing is just folder structure**, resolved as static files by GitHub Pages — there is no router, no server-side logic. This means:

- Internal links to other pages use trailing-slash relative paths so they resolve to that folder's `index.html`: from the root, `resine/`, `galleria/`, `contatti/`; from any subpage, `../`, `../resine/`, etc.
- Asset paths (images, fonts, favicon) live under the shared `public/` directory at the repo root, and are referenced as `public/...` from root pages and `../public/...` from subpages.
- Anchor links to homepage sections (`#servizi`, `#dicono-di-noi`) are written as `../#servizi` etc. from subpages so they navigate back to root first.
- No path anywhere uses a leading slash (`/...`) — everything is relative. Keep it that way; an absolute path would break if the site is ever served from a subpath, and is inconsistent with the rest of the codebase.

When adding a new page, follow this same pattern: create a new directory with its own `index.html`/`script.js`/`style.css`, link to shared images via `../public/...`, and update the nav (`.nav-links`) and footer links **in every existing page** — nav/footer markup is duplicated per page, not shared via includes or templating.

## Duplication is intentional (for now)

Each page's `script.js` and `style.css` are independent copies, not shared modules — there's no bundler to dedupe them. Common behaviors (mobile menu toggle, scroll-reveal `IntersectionObserver`, testimonial carousel) are reimplemented in each page's `script.js` with the same DOM IDs/classes (`#menu-toggle`, `#nav-content`, `.fade-in-up`, `#testimonial-track`, `#prev-btn`/`#next-btn`). `resine/script.js` additionally implements a lightbox (`#lightbox`, `.gallery-item`) reused with the same markup/IDs in `galleria/script.js`. When fixing a bug in one of these shared behaviors, check whether the same code exists in the other pages' `script.js`/`style.css` and fix it there too, since there is no single source of truth to edit once.

## Images

All photos/icons live under `public/`, including two large batches of raw WhatsApp export photos (`public/whatsapp/`, `public/resine/`) with spaces and parentheses in filenames (e.g. `WhatsApp Image 2026-04-04 at 18.03.16 (1).jpeg`) — keep matching the exact filename (including spaces) when referencing or renaming these, and be careful with shell quoting when scripting against them.

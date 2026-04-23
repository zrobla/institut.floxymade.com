# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static marketing site for **FLOXY MADE**, an institut de beauté in Abidjan Cocody (Côte d'Ivoire). Site is in French (`lang="fr"`, locale `fr_CI`), prices in XOF. Production URL: `https://institut.floxymade.com`. Built and maintained by the agency **Tech & Web** (`tech-and-web.com`).

## Commands

No build pipeline, no package manager, no tests. To preview locally:

```bash
python3 -m http.server 8080
```

Deployment is by FTP upload to `ftp.floxymade.com` (the live host serves files from the root of `institut.floxymade.com`). Always confirm before pushing — the user runs deploys manually.

## Architecture

### Stack
- Plain HTML5, CSS, jQuery. No framework, no bundler.
- Vendor libs are vendored under [lib/](lib/): bootstrap, jquery, owlcarousel, magnific-popup, wow, animate, superfish, sticky, easing, font-awesome, ionicons.
- Two stylesheets, **always loaded in this order**:
  1. [css/app.min.css](css/app.min.css) — bundled vendor + base theme (originally from the `art4web` template the site was forked from).
  2. [css/floxymade.css](css/floxymade.css) — FLOXY MADE overrides. Edit this file for visual changes; never touch `app.min.css` (it's a minified bundle, treat as opaque).
- Single bundled JS: [js/app.min.js](js/app.min.js) is the only script the pages load. [js/main.js](js/main.js) exists but is **not referenced** by any page — it's a readable reference of the carousel/nav/scroll logic that lives bundled inside `app.min.js`. Don't edit `main.js` expecting it to take effect; behavioral JS changes have to go through the bundle (or be added inline on the page).

### Page model
Each route is a standalone `.html` file at the repo root — no templating or includes, so the header/nav/footer are duplicated across pages. When you change shared chrome (menu, footer, favicons, SEO meta), apply the edit to every page. The set:

- [index.html](index.html) — home (uses `body.page-front`)
- Service pages: [coiffure_floxymade.html](coiffure_floxymade.html), [esthetique_floxymade.html](esthetique_floxymade.html), [onglerie_floxymade.html](onglerie_floxymade.html), [perruques_floxymade.html](perruques_floxymade.html)
- Product pages: `perruque-*.html` (one per wig SKU)
- [reservation.html](reservation.html) — booking form (uses `body.page-treatment`)
- [achille_koffi.html](achille_koffi.html) — single-image stub for the developer's contact card

The `body.page-front` vs `body.page-treatment` class distinction drives layout differences in `floxymade.css` (see line ~30) — keep it consistent when adding new pages.

### SEO is part of the contract
Every page ships with: `<title>`, description/keywords meta, full Open Graph block (`og:title`, `og:description`, `og:url`, `og:image`, `og:locale=fr_CI`, `og:site_name`), `<link rel="canonical">`, the favicon set (shortcut icon + 192/512 PNG + apple-touch-icon), and `<meta name="author" content="Tech & Web - tech-and-web.com">`. There is also one or more `application/ld+json` Schema.org blocks per page:
- Home: `BeautySalon`, `WebSite`, `ProfessionalService` (the agency credit).
- Product pages: `Product` + `BreadcrumbList`.
- Service pages: typically `Service` + `BreadcrumbList`.

When adding a new page or product, replicate this whole header block and update the schema — don't ship a page with a missing canonical or schema. When changing a price, update both the visible price *and* the `Product` JSON-LD `offers.price`.

### Site-wide content invariants
These appear in multiple pages and must stay in sync if changed:
- Phone: `+225 07 05 00 00 91`
- Email: `contact@floxymade.com`
- Address: `Cocody Angré, Près Pharmacie 8ème Tranche, Abidjan`
- Geo: `5.4012616, -3.9780579`
- Hours: Tue–Sat 08:30–19:30, Sun 13:30–19:00, **closed Monday**
- Socials: facebook.com/floxymade, instagram.com/floxymade.ci, tiktok.com/@floxy_made
- Currency for all prices: XOF

### Image directories
- [img/](img/) — site-wide assets (logo, favicons, backgrounds).
- [img/produits/](img/produits/) — wig product photos (referenced by `perruque-*.html` and their `Product` schema).
- [img/services/](img/services/), [img/portfolio/](img/portfolio/), [img/clients/](img/clients/), [img/intro-carousel/](img/intro-carousel/) — section-specific.
- [img/new-images/](img/new-images/), [img/new-images-2/](img/new-images-2/) — staging folders for asset batches the user is sorting through; don't assume contents are in use.

The additional working directories `/home/kayz/ss.art4web.co/images` and `/home/kayz/Documents/FROM MACBOOK/institut.floxymade.com/img/new-images-2` are sources the user pulls assets *from* — read-only reference, not part of the deployed site.

### robots.txt
[robots.txt](robots.txt) sets `Crawl-delay: 60` and disallows `/*calendar*` and `/*guestbook*`. There is no sitemap.xml currently.

## Conventions

- French copy throughout. HTML entities (`&eacute;`, `&agrave;`, `&amp;`) are used in `<meta>` content values for safety; UTF-8 is fine in body text.
- Recent commits show the user prefers concise French commit messages describing the *what* in user-facing terms (e.g. "Unification footers, icônes RS, bouton CTA premium, breadcrumb pages perruques"). Match that style.
- Don't introduce a build step, package.json, or framework without asking — the "edit file, FTP upload" workflow is intentional.

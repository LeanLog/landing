# LeanLog Landing Page

Marketing site source for [leanlog.ai](https://leanlog.ai). Static HTML.

## What's here

- `index.html` — homepage
- `about.html` — /about page (collective intro, credentials, CTA)
- `assets/logos/` — credentials-strip SVG logos
- `leanlog-final-light 2.png` — wordmark used in nav
- `favicon.ico` — site favicon
- `CNAME` — GitHub Pages custom-domain config (becomes a no-op once we cut over to Cloudflare Pages)
- `DEPLOY.md` — Cloudflare Pages setup walkthrough

## Local preview

```bash
npx --yes http-server . -p 8765 -c-1
```

Then visit http://localhost:8765/ and http://localhost:8765/about.html.

(The repo also works with `python -m http.server` if you have real Python on the PATH; the Microsoft Store stub doesn't.)

## Deployment

**Current:** GitHub Pages auto-deploys from `main` to `leanlog.ai`. The `refresh` branch holds the new content but does not deploy here.

**Next:** Cloudflare Pages, tracking `refresh`, serving on `<project>.pages.dev` for preview. See [`DEPLOY.md`](DEPLOY.md).

**Eventual:** flip the apex DNS off GitHub Pages onto Cloudflare. See the migration sequence in [../demos/naked-wines/DEPLOYMENT.md](../demos/naked-wines/DEPLOYMENT.md#migrating-the-marketing-site-to-cloudflare-pages).

## Conventions

- Em-dashes are banned in prose (per project convention). UI labels and section markers (`01 — partnership`) are fine since they're stylistic, not prose. Browser title bars (`LeanLog — ...`) follow the same convention.
- Logo SVGs are sourced from Wikimedia Commons. Replace with brand-kit files if and when we have them.

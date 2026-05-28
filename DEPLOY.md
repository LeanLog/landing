# Deploying the marketing site

The marketing site at [leanlog.ai](https://leanlog.ai) is served by **Cloudflare Pages** from the `main` branch of this repo. The Pages project is `leanlog-landing` in Oscar's Cloudflare account.

There is a closely related — but separate — doc at [../demos/naked-wines/DEPLOYMENT.md](../demos/naked-wines/DEPLOYMENT.md) that documents how the Naked Wines demo (`nakedwines.leanlog.ai`) is hosted. Same Cloudflare account, same patterns, but the marketing site has no auth gate, no Pages Functions, and is publicly indexable.

---

## Current state

- **Cloudflare Pages project `leanlog-landing`** auto-builds on every push to `main`. No build command — pure static HTML from the repo root.
- **Apex `leanlog.ai`** is on Cloudflare Pages. Headers show `server: cloudflare`.
- **`www.leanlog.ai`** is handled by a Cloudflare Page Rule (`www.leanlog.ai/*` → `https://leanlog.ai/$1`, 301). The redirect fires at the Cloudflare edge — no origin fetch.
- **GitHub Pages is decommissioned.** Source is set to `None` in repo settings. The `CNAME` file has been deleted from the repo.
- **Preview deploys**: pushing any non-`main` branch produces a `<branch>.leanlog-landing.pages.dev` URL with `x-robots-tag: noindex`.

## Day-to-day deploy loop

Push to `main` → Cloudflare auto-rebuilds in ~30s and the live site updates. Push to any other branch → preview URL at `<branch>.leanlog-landing.pages.dev`.

That's it. No CI to maintain, no build command, no environment variables.

---

## Critical guardrails

- **Do not touch MX, SPF, DMARC, or DKIM records** in the `leanlog.ai` Cloudflare zone. Email breaks if you fat-finger the wrong row.
- **Do not copy the auth middleware** (`functions/_middleware.js`) from the demos project. Marketing site is public.
- **Do not include `noindex` meta or `Disallow: /` robots.txt.** Marketing site should be crawlable.
- **SSL/TLS mode at the `leanlog.ai` zone is `Full (strict)`** and should stay that way. Switching to Flexible causes an infinite-redirect loop on the apex.

---

## DNS state reference

The `leanlog.ai` zone is registered with **GoDaddy** (Rodrigo's account) but DNS is delegated to **Cloudflare** (`oscar@leanlog.ai` account). All DNS edits happen in Cloudflare.

Current records:
- `A @` → Cloudflare (Pages-managed, apex pointing at the `leanlog-landing` project)
- `CNAME www` → proxied through CF; the Page Rule handles the 301 to apex regardless of the underlying target. The record may still nominally point at `rodrigo-ll.github.io` — harmless because the Page Rule fires at the edge before any origin fetch, but worth updating to `leanlog.ai` when convenient as DNS hygiene.
- `CNAME nakedwines → demos-naked-wines.pages.dev`
- `MX × 5` (Google Workspace) — **do not touch**
- SPF / DMARC — **do not touch**

---

## Rolling back to GitHub Pages

If the Cloudflare Pages project is ever lost or you need to fall back:

1. Re-enable GitHub Pages: `LeanLog/landing` → **Settings → Pages → Source = `main` (root)**, Custom domain = `leanlog.ai`. GH will recreate a `CNAME` file in the repo.
2. In Cloudflare → `leanlog.ai` zone → **DNS → Records**: remove the apex `A` records that Pages set, add GH's: `185.199.108.153`, `.109.153`, `.110.153`, `.111.153`.
3. Remove `leanlog.ai` from the `leanlog-landing` Pages project's custom domains.
4. Disable or delete the `www → apex` Page Rule. Point `CNAME www` at `rodrigo-ll.github.io`.

Total rollback: ~5 minutes plus DNS propagation.

---

## Analytics

- **Zone analytics** (requests, bandwidth, threats, cache hit ratio) are automatic — visible in the Cloudflare dashboard under the `leanlog.ai` zone → **Analytics & Logs**. No script needed.
- **Web Analytics** (page views, referrers, browser/OS) is **not enabled**. No beacon is present in the served HTML. To enable: Pages project → **Settings → Web Analytics → Enable** (auto-injects the beacon at the edge, no repo change).

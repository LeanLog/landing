# Deploying the marketing site

The marketing site at [leanlog.ai](https://leanlog.ai) is served by **Cloudflare Pages** from the `main` branch of this repo. The Pages project is `leanlog-landing` in Oscar's Cloudflare account.

There is a closely related — but separate — doc at [../demos/naked-wines/DEPLOYMENT.md](../demos/naked-wines/DEPLOYMENT.md) that documents how the Naked Wines demo (`nakedwines.leanlog.ai`) is hosted. Same Cloudflare account, same patterns, but the marketing site has no auth gate, no Pages Functions, and is publicly indexable.

---

## Current state

- **Cloudflare Pages project `leanlog-landing`** auto-builds on every push to `main`. No build command — pure static HTML from the repo root.
- **Apex `leanlog.ai` is on Cloudflare Pages.** DNS A records point at Cloudflare; response headers show `server: cloudflare`. Content matches `leanlog-landing.pages.dev`.
- **`www.leanlog.ai` still routes through GitHub Pages.** The Cloudflare `CNAME www → rodrigo-ll.github.io` record is still in place and proxied, so traffic flows: client → Cloudflare edge → GH Pages → 301 to apex. Works, but GH Pages remains a live dependency on the www path.
- **GitHub Pages source is still enabled** in `LeanLog/landing` repo settings. The repo's `CNAME` file (containing `leanlog.ai`) is the marker.
- **Preview deploys**: pushing any non-`main` branch produces a `<branch>.leanlog-landing.pages.dev` URL with `x-robots-tag: noindex`.

## Remaining work to fully decommission GitHub Pages

Don't skip the order — disabling GH Pages before fixing the `www` DNS would 404 the www subdomain.

### 1. Repoint `www.leanlog.ai` off GH Pages

Pick one of two approaches. Either is fine — redirect rule is cheaper, second custom domain is simpler conceptually.

**Option A (recommended) — CF redirect rule.** Keep `www` resolving to Cloudflare, have the edge 301 to apex.

1. Cloudflare Dashboard → `leanlog.ai` zone → **DNS → Records**.
2. Edit the `CNAME www` row: change target from `rodrigo-ll.github.io` to `leanlog.ai` (or `@`). Keep proxy **on** (orange cloud).
3. Same zone → **Rules → Redirect Rules → Create rule**.
   - Name: `www → apex`
   - When incoming requests match: `Hostname equals www.leanlog.ai`
   - Then: Dynamic redirect, expression `concat("https://leanlog.ai", http.request.uri.path)`, status 301, preserve query string on.
4. Save.

**Option B — second custom domain on Pages.**

1. Workers & Pages → `leanlog-landing` → **Custom domains → Set up a custom domain** → enter `www.leanlog.ai`. Cloudflare creates the record automatically; replace the existing `www → rodrigo-ll.github.io` if prompted.
2. Both apex and www now serve the same content. Optionally add a redirect rule to canonicalize www → apex for SEO.

### 2. Disable GitHub Pages in the repo

GitHub → `LeanLog/landing` → **Settings → Pages** → Source = **None**.

Leave it disabled for ~24h to confirm no fallback paths break, then it's permanent.

### 3. Clean up the repo

After step 2 is confirmed:

- Delete the `CNAME` file (no-op once GH Pages is off).
- This doc stays current — no further changes needed.

### 4. Verify

In a fresh incognito window:

- `https://leanlog.ai` — headers should show `server: cloudflare`, no `x-github-request-id`.
- `https://www.leanlog.ai` — should 301 to apex; redirect headers should also be Cloudflare-served, no GH headers.
- `/about` (or `/about.html`) loads, all 7 credential logos render, mobile breakpoints at 680px and 360px hold.

---

## Critical guardrails

- **Do not touch MX, SPF, DMARC, or DKIM records.** Only the `www` CNAME changes in step 1. Email breaks if you fat-finger the wrong row.
- **Do not copy the auth middleware** (`functions/_middleware.js`) from the demos project. Marketing site is public.
- **Do not include `noindex` meta or `Disallow: /` robots.txt.** Marketing site should be crawlable.
- **SSL/TLS mode at the `leanlog.ai` zone is `Full (strict)`** and should stay that way. Switching to Flexible causes an infinite-redirect loop on the apex.

---

## DNS state reference

The `leanlog.ai` zone is registered with **GoDaddy** (Rodrigo's account) but DNS is delegated to **Cloudflare** (`oscar@leanlog.ai` account). All DNS edits happen in Cloudflare.

Current records:
- `A @` → Cloudflare (Pages-managed, apex pointing at the `leanlog-landing` project)
- `CNAME www → rodrigo-ll.github.io` (proxied through CF, still routes to GH Pages — to be replaced in step 1)
- `CNAME nakedwines → demos-naked-wines.pages.dev`
- `MX × 5` (Google Workspace) — **do not touch**
- SPF / DMARC — **do not touch**

---

## Rolling back

If apex behavior regresses after the www / GH-Pages decommission:

1. Re-enable GitHub Pages in the repo (`Settings → Pages → Source = main`, `Custom domain = leanlog.ai`).
2. In Cloudflare DNS: restore `CNAME www → rodrigo-ll.github.io`, proxied.
3. If apex itself needs to fall back to GH Pages: replace the apex A records with GH's (`185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`) and remove `leanlog.ai` from the Pages project's custom domains.

Total rollback: ~5 minutes plus DNS propagation.

---

## Analytics

- **Zone analytics** (requests, bandwidth, threats, cache hit ratio) are automatic — visible in the Cloudflare dashboard under the `leanlog.ai` zone → **Analytics & Logs**. No script needed.
- **Web Analytics** (page views, referrers, browser/OS) is **not enabled**. No beacon is present in the served HTML. To enable: Pages project → **Settings → Web Analytics → Enable** (auto-injects the beacon at the edge, no repo change). Worth doing only after the `www` cutover so both hostnames are counted.

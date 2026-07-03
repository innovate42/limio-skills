---
name: limio-pages
description: Creates, updates, and publishes Limio PAGES programmatically — assembling landing pages from custom components via the Catalog API, controlling URLs with tags, and migrating existing web pages into Limio. Use when user asks to "create a page via API", "build landing pages", "generate page variants", "publish pages programmatically", "migrate a website to Limio", "copy a webpage into Limio", "bulk create pages", or discusses page assets, tags/routes, shop builds, or publishing. Do NOT use for creating the components themselves (limio-component) or deploying component code (limio-setup).
metadata:
  author: Limio
  version: 8.0.0
---

# Limio Page Assembly & Publishing

Everything the Page Builder does can be driven programmatically. This skill covers the full loop: compose a page from custom components, give it a URL, build it, publish it, and verify it — plus the migration workflow for copying existing web pages into Limio.

**Read `references/page-assembly-api.md` before making any API call** — it has the exact schemas and a complete helper library. For migrating existing sites, also read `references/migration-playbook.md`.

---

## The mental model (explain this to the user early)

Four systems cooperate. Most wasted iterations come from rebuilding the wrong one:

| System | What it holds | How to change it |
|---|---|---|
| **Catalog** | Draft records: `/pages/...`, `/tags/...`, `/offers/...` | Creation jobs (`POST /limio/jobs`), delete (`DELETE /limio/catalogs/1/items/...`) |
| **Shop build** | Compiled site bundles | `POST /api/shop/builds` → returns `buildId` |
| **Publish** | Which build each live route serves | `POST /api/publish {tags, buildId, name}` |
| **Component build** | Compiled custom-component code | git push (limio-setup skill); poll `GET /api/component/builds` |

Decision table:

- Changed **component code/CSS/limioProps schema** → component build only; pages pick it up automatically.
- Changed **prop values on a page** → shop build + publish. No component build.
- Added a **new URL** → one-time route registration (see The New-Route Trap below).

## Core workflow: create → build → publish

1. **Learn the shape from a real page first.** `GET /api/pages?path=/pages/<name>` returns the exact record to replicate (`.items[0].data`). When unsure about any schema, read a page the Page Builder made — never guess.

2. **Create the tag(s)** that define the URL. Tag path minus `/tags` = live URL. **Nested tags make nested URLs** (`/tags/customers/acme` → `/customers/acme`); create parents before children.

3. **Create the page** via a creation job. Rules that prevent failures:
   - Page `path` must be **flat** (`/pages/customer-acme`). Nested page paths fail (`ENOTDIR`) — URL hierarchy comes from tags, never page paths.
   - `assets[]` is the page: ordered components with `position: header|body|footer` and `props` matching each component's `limioProps` ids.
   - SEO lives in `attributes`: `meta_title__limio`, `meta_description__limio`, `canonical_tag__limio`.

4. **Update = delete + recreate.** `DELETE /limio/catalogs/1/items/<url-encoded path>` then re-create. This makes every script idempotent — critical because tokens expire after 1 hour and batches fail midway; a re-run must be safe.

5. **Build** all changed pages in one call: `POST /api/shop/builds {items: ["/pages/a", "/pages/b"]}`. `success: true` = accepted, not finished — the build completes asynchronously (minutes).

6. **Publish**: `POST /api/publish {tags, buildId, name}` with the `id` from step 5.

## The New-Route Trap (the #1 time sink — warn the user)

`POST /api/publish` returns `success: true` **even when it didn't publish your page**. Always check both maps in the response:

- `publishedData.pages` — live, each entry has a `pathPrefix` like `/__v/<commit>_<ts>`.
- `ommitedWithError.pages` — skipped. A **brand-new route** (never published before) always lands here with an *empty* `pathPrefix` and *no error message*: the publish API cannot register new routes.

**Fix:** a one-time bulk **Publish from the Page Builder UI** registers the route; every subsequent API publish then works. Tell the user this upfront when creating new pages: *"API for every update; one Page Builder bulk-publish whenever new URLs are introduced."* There is also no API to mint preview URLs — previewing pre-publish means the Page Builder Preview button, or local rendering (below).

## Verify before and after deploying

- **Before (the iteration killer):** render the assembled page locally — components are plain React, so esbuild-bundle them against the Storybook SDK mock, `renderToStaticMarkup` each asset with its props, inline the CSS, screenshot. Full harness in `references/migration-playbook.md`. This turns deploy-look-fix cycles into seconds.
- **Content**: automated coverage check — every source text fragment must appear in the assembled props (normalise whitespace/quotes/`&amp;` first).
- **After:** `GET /api/pages?path=...` to confirm stored props; fetch the live URL; if a component shows a spinner/error, check `GET /api/component/builds` `logErrors` (a page can be fine while one component's build failed).

## Auth for scripts

**Mint tokens programmatically — never ask the user to paste a Bearer token.** `POST <tenant>/oauth2/token` with `grant_type=client_credentials` returns a 1-hour token (docs: Authentication Overview). Client id/secret come from the `.limio.json` written by limio-setup, or env vars — if neither exists, ask the user for *credentials once* (or run limio-setup), not for tokens repeatedly.

The helper library in `references/page-assembly-api.md` does this for you: caches the token, refreshes 60s before expiry, so long batches never 401. Keep scripts idempotent (delete + recreate) anyway, so any failure is recoverable by re-running.

## Pitfall checklist (scan before every run)

| Pitfall | Symptom | Fix |
|---|---|---|
| Nested page path | Creation job fails | Flat page path + nested tag |
| New route via API publish | In `ommitedWithError`, URL 404s | One-time Page Builder bulk publish |
| Props don't match limioProps ids | Component renders its defaults | Read the component package.json; match ids exactly |
| Renamed a component prop id | Existing pages lose that prop's content | Treat prop ids as a contract; migrate page props when renaming |
| Publishing too fast after build | Route omitted / stale | Wait for build completion, re-publish |
| Token expired mid-batch | 401s halfway | Mint from client credentials with auto-refresh (don't hand-feed tokens); idempotent re-run |
| Missing npm dep in component package.json (e.g. `xss`) | Published page SSR-crashes / spinner | Declare every import in the component's `dependencies` |

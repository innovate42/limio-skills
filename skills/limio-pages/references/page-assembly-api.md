# Page Assembly API Reference

Exact schemas + a complete helper library for creating, building, and publishing Limio pages. Base URL: `https://<tenant>.prod.limio.com`. Endpoints under `/api/...` are the public Commerce API; endpoints under `/limio/...` are the Catalog Admin API (the same API the Limio app uses) — both take `Authorization: Bearer <token>`.

## The page record

```json
{
  "name": "Spring Campaign",
  "record_type": "page",
  "baseTemplate": "/config/templates/pages/default",
  "path": "/pages/spring-campaign",
  "tags": ["/tags/spring-campaign"],
  "isAuthenticated": false,
  "offers": [],
  "attributes": {
    "meta_title__limio": "Spring Campaign | Acme",
    "meta_description__limio": "Save 20% this spring.",
    "primary_color__limio": "#E0431C"
  },
  "pageStyle": "html,body{margin:0;padding:0;background:#fff;}",
  "assets": []
}
```

- `path` — flat, under `/pages`. Never nest.
- `tags` — live URLs. `/tags/spring-campaign` → `/spring-campaign`; `/tags/customers/acme` → `/customers/acme`.
- `pageStyle` — page-level CSS; use a reset when the page is 100% custom components.
- `attributes` — SEO keys as above; carry them from the source page when migrating.

## The asset entry (one component placement)

```json
{
  "path": "/custom-components-2/my-hero",
  "id": "<fresh-uuid>",
  "position": "body",
  "asset": { "contentType": "text/javascript", "url": "/public/my-hero" },
  "contentType": "text/javascript",
  "url": "/public/my-hero",
  "props": { "heading": "Spring sale", "body": "<p>20% off.</p>" }
}
```

- `contentType`/`url` are intentionally duplicated at both levels — include both.
- `position`: `header` | `body` | `footer`; array order = render order within a position.
- `props` keys must exactly match the component's `limioProps` ids. Rich-text props take HTML strings.
- Read a Page Builder-made page (`GET /api/pages?path=...` → `.items[0].data`) whenever unsure — never guess schema.

## Endpoints

| Action | Call |
|---|---|
| Read page | `GET /api/pages?path=/pages/<name>` |
| Create tag/page | `POST /limio/jobs` `{jobType:"creation", updatePath, itemData}` |
| Poll job | `GET /limio/jobs/<id>` → `{state: "completed" \| "failed", failedReason}` |
| Delete item | `DELETE /limio/catalogs/1/items/<encodeURIComponent(path-without-leading-slash)>` |
| Build | `POST /api/shop/builds` `{items:["/pages/..."]}` → `{id: buildId}` (async) |
| Publish | `POST /api/publish` `{tags, buildId, name}` → check BOTH `publishedData.pages` and `ommitedWithError.pages` |
| Component builds | `GET /api/component/builds` → latest build; poll `buildStatus`, inspect `logErrors` |

`updatePath` for jobs is the catalog **tree path of the parent**: `/limio/catalogs/1/tree/pages` for pages; `/limio/catalogs/1/tree/tags` for a top-level tag; `/limio/catalogs/1/tree/tags/customers` for a child of `/tags/customers`.

## Helper library (drop-in)

```javascript
// lib.mjs — Limio page assembly helpers. Node 18+.
// Auth is fully programmatic: tokens are minted from client credentials and
// auto-refreshed — never ask the user to paste a Bearer token.
import { randomUUID } from "node:crypto"

const TENANT = process.env.LIMIO_TENANT       // e.g. https://acme.prod.limio.com
// client credentials: from env or the .limio.json written by limio-setup
const CLIENT_ID = process.env.LIMIO_CLIENT_ID
const CLIENT_SECRET = process.env.LIMIO_CLIENT_SECRET

let _tok = null
async function token() {
  if (_tok && Date.now() < _tok.expiresAt - 60_000) return _tok.value // refresh 60s early
  const r = await fetch(`${TENANT}/oauth2/token`, {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: new URLSearchParams({ grant_type: "client_credentials", client_id: CLIENT_ID, client_secret: CLIENT_SECRET }),
  })
  if (!r.ok) throw new Error(`token mint failed ${r.status}: ${await r.text()}`)
  const { access_token, expires_in } = await r.json()
  _tok = { value: access_token, expiresAt: Date.now() + (expires_in || 3600) * 1000 }
  return _tok.value
}
const H = async () => ({ Authorization: `Bearer ${await token()}`, "Content-Type": "application/json" })

export const custom = (name, position, props = {}) => ({
  path: `/custom-components-2/${name}`, id: randomUUID(), position,
  asset: { contentType: "text/javascript", url: `/public/${name}` },
  contentType: "text/javascript", url: `/public/${name}`, props,
})

export async function job(body) {
  const r = await fetch(`${TENANT}/limio/jobs`, { method: "POST", headers: await H(), body: JSON.stringify(body) })
  if (!r.ok) throw new Error(`job submit ${r.status}: ${await r.text()}`)
  const { id } = await r.json()
  for (let i = 0; i < 60; i++) {
    await new Promise((res) => setTimeout(res, 500))
    const s = await (await fetch(`${TENANT}/limio/jobs/${id}`, { headers: await H() })).json()
    if (s.state === "completed") return
    if (s.state === "failed") throw new Error(s.failedReason || "job failed")
  }
  throw new Error("job timed out")
}

// Create a (possibly nested) tag, building the parent chain: /tags/customers/acme
export async function ensureTag(tagPath, displayName) {
  const segs = tagPath.split("/").filter(Boolean)          // ["tags","customers","acme"]
  for (let i = 2; i <= segs.length; i++) {
    const path = "/" + segs.slice(0, i).join("/")
    await job({
      jobType: "creation",
      updatePath: "/limio/catalogs/1/tree/" + segs.slice(0, i - 1).join("/"),
      itemData: { name: segs[i - 1], record_type: "tag", path,
        baseTemplate: "/config/templates/tags/default",
        attributes: { display_name__limio: i === segs.length ? displayName : segs[i - 1] } },
    })
  }
}

// Idempotent page upsert: delete + recreate, then build. Safe to re-run.
export async function upsertPage({ name, path, tag, attributes = {}, assets, pageStyle }) {
  await ensureTag(tag, name)
  const enc = encodeURIComponent(path.replace(/^\//, ""))
  await fetch(`${TENANT}/limio/catalogs/1/items/${enc}`, { method: "DELETE", headers: await H() })
  await job({ jobType: "creation", updatePath: "/limio/catalogs/1/tree/pages",
    itemData: { name, record_type: "page", baseTemplate: "/config/templates/pages/default",
      path, tags: [tag], isAuthenticated: false, offers: [], attributes,
      pageStyle: pageStyle ?? "html,body{margin:0;padding:0;background:#fff;}", assets } })
}

export async function buildPages(paths) {
  const r = await (await fetch(`${TENANT}/api/shop/builds`, { method: "POST", headers: await H(),
    body: JSON.stringify({ items: paths }) })).json()
  if (!r.id) throw new Error("build not accepted: " + JSON.stringify(r))
  return r.id // buildId for publish
}

export async function publish(tags, buildId, name) {
  const r = await (await fetch(`${TENANT}/api/publish`, { method: "POST", headers: await H(),
    body: JSON.stringify({ tags, buildId, name }) })).json()
  const live = Object.keys(r.publishedData?.pages || {})
  const omitted = Object.keys(r.ommitedWithError?.pages || {})
  return { live, omitted } // omitted ⇒ new route needs one-time Page Builder bulk publish
}
```

Usage:

```javascript
import { custom, upsertPage, buildPages, publish } from "./lib.mjs"

await upsertPage({
  name: "Spring Campaign", path: "/pages/spring-campaign", tag: "/tags/spring-campaign",
  attributes: { meta_title__limio: "Spring Campaign | Acme" },
  assets: [
    custom("my-nav", "header", {}),
    custom("my-hero", "body", { heading: "Spring sale", primaryCta: "Buy now", primaryCtaHref: "/checkout" }),
    custom("my-footer", "footer", {}),
  ],
})
const buildId = await buildPages(["/pages/spring-campaign"])
const { omitted } = await publish(["/tags/spring-campaign"], buildId, "Spring Campaign")
if (omitted.length) console.log("One-time Page Builder bulk-publish needed for:", omitted)
```

## Variant factories

For N landing-page variants, keep ONE pure assembler `(model) => assets[]` and loop `upsertPage` over models, then build all paths in a single `buildPages` call and publish all tags together. Never inline page content into the deploy script — keep content models (JSON) separate from assembly so variants are data, not code.

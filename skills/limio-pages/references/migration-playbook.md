# Migration Playbook: Copy a Web Page into Limio

The pipeline for copying pages from an existing site (Webflow, WordPress, anything) into Limio with **zero content loss** and minimal deploy iterations:

**Extract → Map → Assemble → Verify locally → Deploy → Compare.**

Each step produces a re-runnable artifact. Never hand-transcribe content — hand-transcription is where quotes, tables, and paragraphs silently vanish.

## 1. Extract faithfully

Walk the page's main content container in **DOM order** (jsdom) and convert blocks to clean HTML. Pick the container with the most `<p>` descendants. Fidelity traps that lose content if unhandled:

| Trap | Handling |
|---|---|
| `<blockquote>` pull-quotes | Emit as blockquotes; a walker reading only `<p>` drops them (classic bug: a dangling "As X put it:" with no quote) |
| Tables | Emit `<table>` rows — don't flatten |
| Bold-lead pseudo-headings (`<strong>The Challenge</strong>Text...`) | If a paragraph starts with a short bold span (≤9 words), split into `<h3>` + `<p>` — otherwise it renders as a run-on |
| Zero-width chars (`​`–`‍`, `﻿`) | Strip; drop now-empty blocks (Webflow pads rich text with them) |
| Eyebrow/kicker labels | Short label above a heading, often mis-attached to the *previous* section — carry forward as the next section's `kicker` |
| Hero image | First content-area raster (skip logo SVGs); also capture per-section screenshots |
| Inline links | Keep `<a>` inline in the HTML — they map to rich-text props, never to separate button props |
| Quote attributions | Split trailing "— Name, Title" onto a `<cite>` line |

Extractor core (jsdom; silence its CSS parser with `VirtualConsole`):

```javascript
const clean = (s) => (s || "").replace(/[​-‍﻿]/g, "").replace(/\s+/g, " ").trim()
function extract(container) {
  let html = "", inList = false
  const endList = () => { if (inList) { html += "</ul>"; inList = false } }
  const walk = (el) => {
    for (const n of el.children) {
      const t = n.tagName.toLowerCase(), txt = clean(n.textContent)
      if (/^h[1-6]$/.test(t)) { endList(); if (txt) html += `<${t}>${esc(txt)}</${t}>` }
      else if (t === "p" && txt) {
        const lead = n.firstElementChild
        const bold = lead && /^(STRONG|B)$/.test(lead.tagName) ? clean(lead.textContent) : ""
        if (bold && txt.startsWith(bold) && bold.split(" ").length <= 9) {
          endList(); html += `<h3>${esc(bold)}</h3>`
          const rest = clean(txt.slice(bold.length)); if (rest) html += `<p>${esc(rest)}</p>`
        } else { endList(); html += `<p>${esc(txt)}</p>` }
      }
      else if (t === "blockquote" && txt) { endList(); html += `<blockquote>${esc(txt)}</blockquote>` }
      else if (t === "li" && txt) { if (!inList) { html += "<ul>"; inList = true } html += `<li>${esc(txt)}</li>` }
      else if (t === "table") { endList(); /* emit rows/cells */ }
      else if (["ul","ol","div","section","span","figure"].includes(t)) walk(n)
    }
  }
  walk(container); endList(); return html
}
```

Save each page as a JSON model (`{title, desc, h1, heroSub, heroImg, sections:[...]}`) — every later step reads this file.

## 2. Map sections to components

| Source pattern | Component archetype |
|---|---|
| Headline + sub + CTAs (+ product shot + logo strip) | Hero |
| Copy beside a screenshot | Split panel (alternate `imageSide`) |
| 3+ titled cards | Card grid (5 cards → force 3 columns for 3+2) |
| 2 short label headings each followed by bullets | ONE panel whose rich-text body has `<h4>` groups + `<ul>` — not a flat mega-list |
| Big numbers + captions | Stats band |
| Quotation + attribution | Quote band |
| Long article | Rich-text prose |
| Customer-story teasers | Case-study cards (image+logo+links), not paragraphs |
| Closing strip | CTA band |

Two rules that prevent the most rework: **rich text beats arrays** (headings/paragraphs/bullets go in ONE richtext prop; arrays only for repeated structured cards), and **buttons are props, links are content** (standalone CTA = label+href props; in-sentence link = inline `<a>`).

Keep the assembler **pure** (`model → assets[]`, no network) so local render and deploy share one source of truth. Preserve the source URL structure with nested tags, and carry `<title>`/meta description into `meta_title__limio`/`meta_description__limio`.

## 3. Verify locally before deploying (the iteration killer)

Custom components are plain React — render assembled pages locally against the Storybook SDK mock and screenshot them beside the original. This is what turns ten deploys per page into one.

```javascript
// render.mjs — local SSR harness (npm i esbuild react react-dom jsdom playwright-core)
import esbuild from "esbuild"
import { createRequire } from "node:module"
const require = createRequire(import.meta.url)

// 1. Generate an entry that maps component names -> require(componentDir)
//    and exposes: renderAsset(name, props) => sdk.__mockConfig.propsOverride = props;
//    ReactDOMServer.renderToStaticMarkup(<Component/>)
await esbuild.build({
  entryPoints: ["out/entry.js"], bundle: true, format: "cjs", platform: "node",
  outfile: "out/bundle.cjs",
  alias: { "@limio/sdk": "<repo>/__mocks__/@limio/sdk.js" },     // the Storybook mock
  loader: { ".css": "empty", ".js": "jsx", ".svg": "text", ".png": "dataurl" },
  jsx: "automatic",
})
const { renderAsset } = require("./out/bundle.cjs")
// 2. html = assets.map(a => renderAsset(nameOf(a), a.props)).join("")
// 3. Inline each used component's index.css into a <style> tag; write page.html
// 4. Screenshot page.html and the live source URL side by side (set img.loading='eager',
//    scroll to bottom to defeat lazy-loading, then full-page screenshot)
```

The mock must expose a mutable `__mockConfig.propsOverride` consumed by `useComponentProps` (the standard Storybook mock pattern) so each asset renders with its real page props.

## 4. Content-loss check (run it, don't eyeball it)

```javascript
const norm = (s) => s.toLowerCase().replace(/&amp;/g, "&").replace(/[‘’]/g, "'")
  .replace(/[“”]/g, '"').replace(/\s+/g, " ").trim()
const hay = norm(JSON.stringify(assets))
const missing = sourceFragments(model).filter((f) => !hay.includes(norm(f)))
```

- Fragments = every heading, paragraph, list item, quote, stat, card title/body from the model (≥2 words).
- Normalise entities (`&` vs `&amp;`), curly quotes, whitespace — or you chase false positives.
- Target **zero missing** per page; record intentional restructures (e.g. teaser paragraphs replaced by richer cards) as explicit exceptions rather than lowering the bar.

## 5. Deploy & compare

Deploy via `page-assembly-api.md`, then side-by-side the **deployed** page vs the source at desktop + mobile widths. Structural issues (flattened groups, misplaced eyebrows, orphan grid rows) survive text checks and only show up visually. Give the highest-traffic pages a second, closer pass.

Images may keep pointing at the source site's CDN during migration (shop CSP allows any `https:` image) — honest side-by-sides now, asset re-hosting later.

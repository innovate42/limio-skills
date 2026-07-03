# Marketing & Landing-Page Components

Design rules for components meant to compose into many marketing/landing pages (campaign variants, migrated sites). Different discipline from commerce components: optimise for a **small catalog** that expresses any page, with props fillable without reading the source.

## The catalog (≈12 components covers a whole marketing site)

Nav, Footer, Hero (headline/sub/CTA pair/product image/logo strip), Split panel (rich text beside image, `imageSide` left|right, stacks on mobile), Card grid, Stats band, Quote band, Rich-text article, Case-study cards, CTA band, Logo marquee, FAQ. Resist per-page components — extend only for genuinely new *patterns*.

## Prop design rules (highest-leverage first)

1. **One rich-text body beats an array of items.** If a section's copy is headings/paragraphs/bullets, model it as ONE `richtext` prop and style `h3`/`h4`/`ul`/`blockquote` in the component CSS. Grouped lists ("Manage" + bullets, "Expand" + bullets) become `<h4>` + `<ul>` inside that body. Reserve `list` props for genuinely repeated *structured* units (icon + title + body + link cards, logo rows, stats). A list whose items are just `{title, body}` should be rich text instead.

2. **Buttons are prop pairs; links are content.** Standalone CTA → `buttonLabel` + `buttonHref` string props (render nothing when either is empty). A link inside a sentence stays an inline `<a>` in rich text — never a separate prop.

3. **Same furniture on every section component:** optional `kicker` (small uppercase eyebrow), `heading`, and a `theme` picklist (`light`/`dark`/brand) where designs use multiple grounds. Consistency here is what makes assembled pages feel designed.

4. **Columns are a preference prop.** Grids default to responsive auto-fit but expose `columns` (`auto`/`2`/`3`/`4`) — 5 cards usually want 3+2, not 4+1. Implement as **CSS classes, never inline styles**, so media queries still collapse on tablet/mobile:

   ```css
   .grid--c3 { grid-template-columns: repeat(3, minmax(0,1fr)); }
   @media (max-width: 900px) { .grid--c3 { grid-template-columns: repeat(2, minmax(0,1fr)); } }
   @media (max-width: 600px) { .grid--c3 { grid-template-columns: 1fr; } }
   ```

5. **Icons: named inline-SVG set + URL override.** Ship a small `Icon` component keyed by name (`icon: "chart"`) — inline SVGs inherit `currentColor` and need no external files — plus `iconImg` (URL) that overrides it for specific assets.

6. **Defaults must demo.** Every prop default should render a presentable section the moment the component lands in the Page Builder — real copy, not placeholders or empty strings.

7. **Prop ids are a contract.** Page assets store props keyed by `limioProps` id; renaming an id silently orphans that content on every existing page. Rename only with a migration of page props.

## Consistency across the family

Components bundle independently — don't import shared stylesheets. Repeat one small token block at the top of every component's CSS and derive everything from it:

```css
.cmp { --ink:#2b2145; --muted:#6d6d72; --accent:#d14424;
       --surface:#fbf3ec; --border:rgba(43,33,69,.1);
       --head:"DisplayFont",ui-sans-serif,system-ui,sans-serif; }
```

One type scale + one accent + one border treatment, copied verbatim, makes twelve components read as one system. Namespace every class (`.lpn__`, `.ldg__`) — pages stack many components and unprefixed classes collide.

## SSR reality check

Components server-render during shop builds:

- Guard `window`/`document` behind effects or existence checks.
- **Declare every npm import in the component's `package.json` `dependencies`** — an undeclared dep (classically `"xss"` when using `sanitiseHTML`) works in Storybook and then **crashes the published page's server render** (page shows a spinner/error).
- `loading="lazy"` on images is fine — but remember it when screenshotting.

## Verification loop for one-shot quality

1. Storybook stories per variant (limio-story skill).
2. Local full-page SSR render with real page props before deploying (see limio-pages skill → migration-playbook).
3. After git deploy: poll `GET /api/component/builds`; even on `SUCCEEDED`, check `logErrors` — one component can fail while the batch passes.
4. Changing component code/CSS/limioProps needs a component build only; pages pick it up automatically. Changing page prop *values* needs a shop build + publish instead.

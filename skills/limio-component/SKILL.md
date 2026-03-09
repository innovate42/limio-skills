---
name: limio-component
description: Creates Limio custom React components following official SDK guidelines and best practices. Use when user asks to "create a Limio component", "build a subscription component", "make offer cards", mentions "limioProps", "@limio/sdk", "useCampaign", "useBasket", "useUser", or discusses building React components for the Limio subscription platform. Do NOT use for Storybook setup, story creation, or deployment - those have dedicated skills.
metadata:
  author: Limio
  version: 8.0.0
---

# Limio Component Creation

Use this skill when creating custom React components for the Limio subscription management platform. Components are created in `./components/` relative to the project root.

**IMPORTANT:** This skill is self-contained — it contains all the documentation you need for building components. Do NOT explore the filesystem or search for existing component patterns. Use the templates, SDK reference, and examples provided below to create components directly. Full SDK docs are available at https://docs.limio.com/developers/limio-sdk

**CRITICAL — Credential Safety:** `.limio.json` contains OAuth client credentials (client ID + client secret). It MUST be in `.gitignore` and MUST NEVER be committed to git. When committing or staging files, NEVER include `.limio.json`. If you see it in `git status` as untracked or modified, verify it is gitignored before proceeding. When creating a new project, always add `.limio.json` to the **root** `.gitignore`.

## Component Location

**Create components in `./components/<component-name>/` relative to the project root.** If the directory doesn't exist, create it.

```
/components/
├── component-name/
│   ├── package.json
│   ├── index.js
│   ├── componentStaticProps.js
│   └── index.css
├── another-component/
│   └── ...
```

## Component Structure

Each component folder contains:

```
component-name/
├── package.json           # Dependencies + limioProps config
├── index.js               # Main React component
├── componentStaticProps.js # Props hook setup
└── index.css              # Styles (optional)
```

## package.json Format

```json
{
  "name": "@limio/component-name",
  "version": "1.0.0",
  "description": "Component description",
  "main": "./index.js",
  "dependencies": {},
  "peerDependencies": {
    "react": "*"
  },
  "limioProps": []
}
```

## Dependencies

You can import **any public npm library** in the dependencies. Limio's build system will bundle them.

**Prefer SDKs/libraries over custom code** — makes components easier to maintain and more reliable.

**Common libraries:**
- `ramda` — Functional utilities (groupBy, prop, etc.)
- `xss` — HTML sanitization (required for rich text)
- `@mui/material` — Material UI (use **5.16.12** for React 19 compatibility)
- `@emotion/react` / `@emotion/styled` — Required for MUI
- `date-fns` or `dayjs` — Date formatting

## componentStaticProps.js

**Important:** Use default import for package.json, not `import * as`.

```javascript
import { useComponentProps } from "@limio/sdk"
import { getPropsFromPackageJson } from "@limio/components/helpers"
import packageData from "./package.json"

const defaultComponentProps = getPropsFromPackageJson(packageData)

export function useStaticProps() {
    return useComponentProps(defaultComponentProps)
}
```

**Note:** In the Storybook playground, `getPropsFromPackageJson` is mocked from `@limio/sdk` instead. Both import paths work — use `@limio/components/helpers` for production components.

## Self-Contained Components

Components must be **self-contained** — do NOT import from `../source/utils/` or other internal paths. These rely on modules (like `@limio/shop/src/shop/appConfig.js`) that are not mocked in Storybook and will break.

Instead, use SDK utilities (`formatCurrency`, `formatDate`, `checkActiveOffers`, `getCurrentOffer`, `useSchedule`, etc.) or write small inline helpers within the component itself.

---

## limioProps Types

Quick summary of each type with examples. For the full reference with all options, see `references/prop-types.md`.

### String
```json
{ "id": "headline", "label": "Headline", "type": "string", "default": "Welcome" }
```

### Boolean
```json
{ "id": "showImage", "label": "Show image", "type": "boolean", "default": true }
```

### Number
```json
{ "id": "cardWidth", "label": "Card width", "type": "number", "default": "2" }
```

### Rich Text (HTML)
```json
{ "id": "description", "label": "Description", "type": "richtext", "default": "<p>Content</p>" }
```

### Color
```json
{ "id": "primaryColor", "label": "Primary color", "type": "color", "default": "#635BFF" }
```

### DateTime
```json
{ "id": "expiryDateTime", "label": "Expiry", "type": "datetime", "default": "2025-12-10T11:30:42.809Z" }
```

### Picklist (Dropdown)
```json
{
  "id": "theme",
  "label": "Theme",
  "type": "picklist",
  "options": [
    { "id": "light", "label": "Light", "value": "light" },
    { "id": "dark", "label": "Dark", "value": "dark" }
  ],
  "default": "light"
}
```

### List (Array of Objects)
```json
{
  "id": "groupLabels",
  "label": "Group Labels",
  "type": "list",
  "fields": {
    "name": { "id": "id", "label": "ID", "type": "string" },
    "url": { "id": "label", "label": "Label", "type": "string" },
    "thumbnail": { "id": "thumbnail", "label": "Thumbnail", "type": "string", "format": "uri", "purpose": "image" }
  },
  "default": [
    { "id": "monthly", "label": "Monthly" },
    { "id": "annual", "label": "Annual" }
  ]
}
```

**Important:** List items are `{id, label}` objects, not plain strings.

For detailed prop types reference including schema type and naming conventions, see `references/prop-types.md`.

---

## SDK Hooks Quick Reference

Brief imports and usage for each hook. For the full SDK reference with complete return shapes, see `references/sdk-hooks-quickref.md`.

### useCampaign
```javascript
import { useCampaign } from "@limio/sdk"

const { offers, campaign, addOns, tag, groupValues } = useCampaign()
```
- `campaign` — Page metadata: `{ name, path, attributes }`
- `offers` — Array of subscription products
- `addOns` — Array of optional products/upsells
- `tag` — Entry tracking tag (e.g., "/tags/dummytag")
- `groupValues` — Array of `{ label, id }` for offer categorization

### groupOffers Utility
```javascript
import { groupOffers } from "@limio/sdk"

const grouped = groupOffers(offers, groupLabels)
// Returns: Array<{ groupId, id, label, offers, thumbnail }>
```

### useBasket
```javascript
import { useBasket } from "@limio/sdk"
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"

const {
  orderItems,           // Current basket items
  basketLoading,        // Boolean for async operations
  initiateCheckout,
  addOfferToBasket,
  removeFromBasket,
  navigateToCheckout,
  // ... see references/sdk-hooks-quickref.md for full list
} = useBasket()
```

### getCurrentBasketId
```javascript
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"
```

### useUser
```javascript
import { useUser } from "@limio/sdk"

const { attributes, loginStatus, loaded, token } = useUser()
```

### useSubscriptions
```javascript
import { useSubscriptions } from "@limio/sdk"

const { subscriptions } = useSubscriptions()
```

### useLimioContext
```javascript
import { useLimioContext } from "@limio/sdk"

const { isInPageBuilder } = useLimioContext() || {}
```

### useCheckout
```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })
const order = useCheckoutSelector((state) => state.order)
const orderTotals = useCheckoutSelector((state) => state.display.orderTotal)
```

For full SDK reference including all hook return shapes, utility functions, and subscription helpers, see `references/sdk-hooks-quickref.md`.

---

## Add to Basket Pattern

The standard pattern for adding offers to the basket:

```javascript
const handleAddToBasket = async (offer) => {
  const checkoutId = getCurrentBasketId()
  if (!checkoutId) {
    await initiateCheckout({ order: { orderItems: [{ offer }] } })
  } else {
    await addOfferToBasket({ offer })
  }
  if (pageOptions?.pushToCheckout) {
    await navigateToCheckout()
  }
}
```

---

## Component Template

```javascript
import React, { useState, useMemo } from "react"
import { useCampaign, useBasket, useLimioContext } from "@limio/sdk"
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"
import { useStaticProps } from "./componentStaticProps"
import { groupBy, prop } from "ramda"
import xss from "xss"
import "./index.css"

const sanitizeString = (str) => xss(str || "")
const groupOffers = groupBy(prop("group__limio"))

const MyComponent = () => {
    const { offers } = useCampaign() || {}
    const { isInPageBuilder } = useLimioContext() || {}
    const props = useStaticProps() || {}

    const { heading = "Default", groupLabels = [], showGroupedOffers = false } = props

    const groupedOffers = useMemo(() => {
        if (!offers || !Array.isArray(offers)) return {}
        return groupOffers(
            offers.map(offer => ({
                ...offer,
                group__limio: offer?.data?.attributes?.group__limio || "default",
            }))
        )
    }, [offers])

    const validLabels = useMemo(() => {
        const groups = Object.keys(groupedOffers)
        if (groupLabels?.length > 0) {
            return groupLabels.filter(item => groups.includes(item.id))
        }
        return groups.map(g => ({ id: g, label: g }))
    }, [groupLabels, groupedOffers])

    const [selectedGroup, setSelectedGroup] = useState(validLabels[0]?.id || "")

    const displayedOffers = useMemo(() => {
        if (showGroupedOffers && selectedGroup && groupedOffers[selectedGroup]) {
            return groupedOffers[selectedGroup]
        }
        return offers || []
    }, [showGroupedOffers, selectedGroup, groupedOffers, offers])

    if (!offers?.length) return null

    return (
        <section>
            <h1>{heading}</h1>
            {displayedOffers.map((offer, i) => (
                <OfferCard key={offer?.id || i} offer={offer} />
            ))}
        </section>
    )
}

export default MyComponent
```

---

## CSS Patterns

Use plain CSS with **CSS custom properties** for white-labelling. Map color limioProps to CSS variables via `style`:

```javascript
<div className="my-component" style={{ "--my-primary": primaryColor, "--my-danger": dangerColor }}>
```

```css
.my-component {
    --my-primary: #635BFF;
    --my-text: #1a1f36;
    --my-text-muted: #697386;
    --my-border: #e3e8ee;
    --my-bg: #f6f9fc;
    --my-card: #ffffff;

    background: var(--my-bg);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    color: var(--my-text);
    -webkit-font-smoothing: antialiased;
}
```

Key CSS patterns:
- Prefix all classes with a short component abbreviation (e.g. `ad-`, `oc-`, `sp-`) to avoid collisions
- Use `border: 1px solid var(--border)` + `border-radius: 10px` + subtle `box-shadow` for cards
- 13px uppercase `letter-spacing: 0.06em` for section titles
- `flex` with `justify-content: space-between` for detail rows
- Always include `@media (max-width: 600px)` responsive breakpoint
- Reset box-sizing: `.my-component *, .my-component *::before, .my-component *::after { box-sizing: border-box; }`

---

## Skeleton & Error States

Modern Limio components define `.Skeleton` and `.Error` static methods for loading and error states:

```javascript
const MyComponent = () => {
    // ... component code
}

MyComponent.Skeleton = () => (
    <div className="mc-skeleton">
        <div className="mc-skeleton-line" style={{ width: "60%", height: 20 }} />
        <div className="mc-skeleton-line" style={{ width: "100%", height: 16 }} />
    </div>
)

MyComponent.Error = () => (
    <div className="mc-error">
        <p>Something went wrong. Please try refreshing.</p>
    </div>
)

export default MyComponent
```

Wrap components with `ErrorBoundary` for graceful failure:
```javascript
import { ErrorBoundary } from "@limio/sdk"

<ErrorBoundary fallback={<MyComponent.Error />}>
    <MyComponent />
</ErrorBoundary>
```

---

## Error Handling for Basket Operations

Wrap async basket operations in try/catch for resilience:

```javascript
const handleAddToBasket = async (offer) => {
    try {
        const checkoutId = getCurrentBasketId()
        if (!checkoutId) {
            await initiateCheckout({ order: { orderItems: [{ offer }] } })
        } else {
            await addOfferToBasket({ offer })
        }
        if (pageOptions?.pushToCheckout) {
            await navigateToCheckout()
        }
    } catch (error) {
        console.error("Failed to add to basket:", error)
    }
}
```

---

## useCheckout (from @limio/internal-checkout-sdk)

For components in the checkout/cart flow, use `useCheckout` from `@limio/internal-checkout-sdk`:

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })
const order = useCheckoutSelector((state) => state.order)
const { orderItems } = order
```

This gives access to checkout state including `order`, `paidSchedule`, `schedule`, `locale`, and `nextActions` (for upgrades/downgrades/cross-sells).

---

## Handlebars Templating in Offer Attributes

Offer attributes can contain Handlebars templates. Use `parseString` to render them:

```javascript
import { parseString, encodeDates } from "@limio/shop/src/helpers/string"

const displayText = parseString(
    offer.data.attributes.display_description__limio,
    offer,
    encodeDates
)
```

limioProps can also use templates: `"default": "{{data.attributes.display_description__limio}}"`.

---

## Best Practices

1. **Use SDK utilities** — `sanitiseHTML`, `formatCurrency`, `formatDate`, `useOfferInfo`, `checkActiveOffers`, `groupOffers`, `getCurrentOffer`, `useSchedule`, `useSubInfo`, etc. Don't reimplement what the SDK provides.
2. **Self-contained** — Do NOT import from `../source/utils/` or other internal paths. Use SDK utilities or inline helpers.
3. **Null safety** — Always use optional chaining and defaults:
   ```javascript
   const { offers } = useCampaign() || {}
   const attributes = offer?.data?.attributes || {}
   ```
4. **All text configurable** — Every heading, label, button text, and URL should be a `limioProp` so the component is fully white-label.
5. **Color props** — Use `"type": "color"` in limioProps (no special suffix needed on the ID). Pass through CSS custom properties.
6. **List props** — Items are `{id, label}` objects.
7. **Picklist options** — Use `options` array with `{id, label, value}`.
8. **Page Builder compatibility** — Components with `position: fixed/absolute` must fall back to `position: relative` when `isInPageBuilder` is true.
9. **Loading states** — Handle `basketLoading` to prevent double submissions.
10. **Sanitize HTML** — Use `xss` library or `sanitiseHTML` from SDK for rich text content.
11. **MUI version** — Use 5.16.12 for React 19 compatibility.
12. **Contrast colors** — When using configurable background colors for buttons, calculate contrasting text color for accessibility.
13. **Subscription references** — Use `subscription.reference` or `subscription.id` for linking. Pass as URL query params (e.g. `?subRef=...`).
14. **Subscription offers access** — Always use `subscription.offers[]` array to access offers. Do NOT use `subscription.data.offer` (legacy). A subscription can have multiple offers (standard + discount), so filter where `record_subtype` is NOT `"discount"` and check `start`/`end` dates to find the current active standard offer.

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Imports not resolving | Missing webpack aliases | Ensure `main.js` has aliases for `@limio/sdk`, `@limio/shop`, `@limio/internal-checkout-sdk` |
| SDK hooks return undefined | Hook not mocked in SDK mock | Check `packages/limio/sdk/src/context.js` has the hook exported |
| `sanitiseHTML` not found | Not mocked in Storybook SDK | Use `xss` npm package as fallback in components, or add mock to SDK |
| Component breaks in Storybook | Importing from `../source/utils/` | Make component self-contained; use SDK utilities or inline helpers |
| `position: fixed` breaks Page Builder | No Page Builder fallback | Check `isInPageBuilder` from `useLimioContext()` and use `position: relative` |
| `.limio.json` in git status | Not gitignored | Add `.limio.json` to root `.gitignore` immediately |
| MUI styles broken | Wrong MUI version | Pin `@mui/material` to `5.16.12` |
| List prop values incorrect | Using plain strings | List items must be `{id, label}` objects |

---

For related workflows, see skills: `limio-story` (creating stories), `limio-storybook` (playground setup), `limio-setup` (credentials & deploy).

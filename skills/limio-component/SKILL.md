---
name: limio-component
description: This skill should be used when the user asks to "create a Limio component", "build a subscription component", "make offer cards", "set up limio", "limio setup", "configure limio", "connect to limio", "launch storybook", "start storybook", "run storybook", "open storybook", mentions "limioProps", "Limio SDK", "@limio/sdk", "useCampaign", "useBasket", "useUser", or discusses building React components for the Limio subscription platform.
version: 5.2.0
---

# Limio Custom Component Creation

Use this skill when creating custom components for the Limio subscription management platform.

**IMPORTANT:** This skill contains all the documentation you need for building components. Do NOT explore the filesystem or search for existing component patterns. Use the templates, SDK reference, and examples provided below to create components directly. The one exception is checking whether Storybook is already set up (see Storybook section).

**Official SDK docs:** https://docs.limio.com/developers/limio-sdk

## Skill Version Check

**Current version: 5.2.0** — Before running any workflow, verify the skill is up to date:

1. Read the first 5 lines of this skill file and check for `version: 5.2.0`
2. If the version is missing or lower than `5.2.0`, the skill cache is stale. Copy the latest from the source repo:
   ```bash
   cp /Users/benny/dev/limio-skills/skills/limio-component/SKILL.md /Users/benny/.claude/skills/limio-component/SKILL.md
   ```
3. Also check for these markers in the skill itself — if any are missing, the skill is outdated:
   - `Addon Version Checking` (section heading)
   - `SETTINGS_PANEL_ID` (in the manager.js template)
   - `/api/deploy` (in the middleware.js template)
   - `Skill Version Check` (this section)
   - `ClaudeOverlay` (in the preview.js / claude-overlay.js template)
   - `NewComponent` (in the NewComponent.stories.js template)

If the skill was updated, re-read it before continuing.

## Full Workflow

1. **Create the component** in `./components/` (using the reference sections below)
2. **Check for Storybook:** Look for `component-playground/.storybook/main.js`
3. **If no Storybook exists:** Set up the playground (see "Storybook Setup" section)
4. **Check for Claude Prompt addon:** Look for `component-playground/.storybook/addon-prompt/manager.js`
5. **If no addon exists:** Set up the Claude Prompt addon (see "Claude Prompt Addon" section)
6. **If addon exists, check if outdated:** grep for `SETTINGS_PANEL_ID` in `manager.js` and `/api/deploy` in `middleware.js` — if either is missing, replace both files with the current templates below (see "Addon Version Checking")
7. **Check for Claude Overlay:** Look for `component-playground/.storybook/claude-overlay.js` — if missing, create it (see "Claude Overlay" section) and update `preview.js` to import and render it
8. **Check for New Component story:** Look for `component-playground/src/stories/NewComponent.stories.js` — if missing, create it (see "New Component Builder" section)
9. **Create a story** for the component with multiple variations
10. **Install dependencies** if needed: `cd component-playground && npm install`
11. **Start Storybook:** `cd component-playground && npx storybook dev -p 6006` (run in background)
12. **Start the prompt watcher:** `node component-playground/scripts/watch-prompts.js` (run in background)
13. **Show the user** the running Storybook and mention the Claude Prompt panel
14. **When prompt watcher exits** (prompt received), read `.prompt.json`, apply changes, then restart the watcher

## Limio Setup Workflow

When the user asks to "set up Limio", "configure Limio", or "connect to Limio":

1. **Check for Storybook:** Look for `component-playground/.storybook/main.js`
2. **If no Storybook exists:** Set up the full playground (see "Storybook Setup" section)
3. **Check for Claude Prompt addon:** Look for `component-playground/.storybook/addon-prompt/manager.js`
4. **If no addon exists:** Set up the Claude Prompt addon (see "Claude Prompt Addon" section)
5. **If addon exists, check if outdated:** grep for `SETTINGS_PANEL_ID` in `manager.js` and `/api/deploy` in `middleware.js` — if either is missing, replace both files with the current templates below (see "Addon Version Checking")
6. **Check for Claude Overlay:** Look for `component-playground/.storybook/claude-overlay.js` — if missing, create it and update `preview.js`
7. **Check for New Component story:** Look for `component-playground/src/stories/NewComponent.stories.js` — if missing, create it
8. **Install dependencies:** `cd component-playground && npm install`
9. **Start Storybook:** `cd component-playground && npx storybook dev -p 6006`
10. **Tell the user:** "Storybook is running at http://localhost:6006 — open **Tools > Limio Setup** in the sidebar to connect your Limio account."
11. **Start the prompt watcher loop** (see "Start Storybook" section)

This flow ensures that even a brand-new project with no Storybook gets everything bootstrapped in one command.

## Launch Storybook Workflow

When the user asks to "launch storybook", "start storybook", "run storybook", or "open storybook":

1. **Check for Storybook:** Look for `component-playground/.storybook/main.js`
2. **If no Storybook exists:** Set up the full playground (see "Storybook Setup" section)
3. **Check for Claude Prompt addon:** Look for `component-playground/.storybook/addon-prompt/manager.js`
4. **If no addon exists:** Set up the Claude Prompt addon (see "Claude Prompt Addon" section)
5. **If addon exists, check if outdated:** grep for `SETTINGS_PANEL_ID` in `manager.js` and `/api/deploy` in `middleware.js` — if either is missing, replace both files with the current templates below (see "Addon Version Checking")
6. **Check for Claude Overlay:** Look for `component-playground/.storybook/claude-overlay.js` — if missing, create it and update `preview.js`
7. **Check for New Component story:** Look for `component-playground/src/stories/NewComponent.stories.js` — if missing, create it
8. **Install dependencies if needed:** `cd component-playground && npm install`
9. **Start Storybook:** `cd component-playground && npx storybook dev -p 6006` (run in background)
10. **Start the prompt watcher:** `node component-playground/scripts/watch-prompts.js` (run in background)
11. **Tell the user:** "Storybook is running at http://localhost:6006" and list any available stories

This is the quick-launch path — it skips component creation and just starts the dev environment.

## Component Location

**Create components in `./components/` relative to the project root.** If the directory doesn't exist, create it.

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

**Prefer SDKs/libraries over custom code** - makes components easier to maintain and more reliable.

**Common libraries:**
- `ramda` - Functional utilities (groupBy, prop, etc.)
- `xss` - HTML sanitization (required for rich text)
- `@mui/material` - Material UI (use **5.16.12** for React 19 compatibility)
- `@emotion/react` / `@emotion/styled` - Required for MUI
- `date-fns` or `dayjs` - Date formatting

## componentStaticProps.js

**Important:** Use default import for package.json, not `import * as`.

```javascript
import { useComponentProps, getPropsFromPackageJson } from "@limio/sdk"
import packageData from "./package.json"

const defaultComponentProps = getPropsFromPackageJson(packageData)

export function useStaticProps() {
    return useComponentProps(defaultComponentProps)
}
```

## Self-Contained Components

Components must be **self-contained** — do NOT import from `../source/utils/` or other internal paths. These rely on modules (like `@limio/shop/src/shop/appConfig.js`) that are not mocked in Storybook and will break.

Instead, use SDK utilities (`formatCurrency`, `formatDate`, `checkActiveOffers`, `getCurrentOffer`, `useSchedule`, etc.) or write small inline helpers within the component itself.

---

## limioProps Types

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
{ "id": "description__limio_richtext", "label": "Description", "type": "richText", "default": "<p>Content</p>" }
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

---

## Limio SDK - Page/Campaign

### useCampaign
Returns page/campaign data including offers.

```javascript
import { useCampaign } from "@limio/sdk"

const { offers, campaign, addOns, tag, groupValues } = useCampaign()
```

**Returns:**
- `campaign` - Page metadata: `{ name, path, attributes }`
- `offers` - Array of subscription products
- `addOns` - Array of optional products/upsells
- `tag` - Entry tracking tag (e.g., "/tags/dummytag")
- `groupValues` - Array of `{ label, id }` for offer categorization

### groupOffers Utility
```javascript
import { groupOffers } from "@limio/sdk"

const grouped = groupOffers(offers, groupLabels)
// Returns: Array<{ groupId, id, label, offers, thumbnail }>
```

---

## Offer Object Structure

```javascript
offer = {
  id: "unique-id",
  name: "Offer Name",
  path: "/offers/offer-name",
  parent_path: "/pages/page-name",
  type: "item",
  data: {
    attributes: {
      // Display
      display_name__limio: "Premium Plan",
      display_price__limio: "<span>$9.99</span>/mo",
      detailed_display_price__limio: "Billed annually at $119.88",
      offer_features__limio: "<ul><li>Feature 1</li></ul>",
      cta_text__limio: "Subscribe Now",
      checkout_description__limio: "Premium subscription",

      // Grouping & Flags
      group__limio: "monthly",
      best_value__limio: true,
      badge_text__limio: "Most Popular",

      // Commerce
      payment_types__limio: ["card", "paypal"],
      allowed_countries__limio: ["US", "GB"],
      allow_multibuy__limio: false,
      autoRenew__limio: true,

      // Cross-sell/Upsell
      cross_sell_addons__limio: [...],
      cross_sell_offers__limio: [...],
      upsell_offers__limio: [...],

      // Term
      term__limio: { renewal_type, renewal_trigger },
      initial_term__limio: { renewal_type, renewal_trigger }
    },
    price: [{
      name: "Monthly charge",
      value: 9.99,
      currencyCode: "USD",
      type: "recurring",
      trigger: "subscription_start",
      repeat_interval: 1,
      repeat_interval_type: "months"
    }],
    products: [{
      path: "/products/product-name",
      name: "Product",
      attributes: { display_name, product_code }
    }],
    attachments: [{
      type: "image",
      url: "https://..."
    }]
  }
}
```

---

## Limio SDK - Basket

### useBasket
```javascript
import { useBasket } from "@limio/sdk"
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"

const {
  orderItems,           // Current basket items
  basketLoading,        // Boolean for async operations
  formattedTotal,       // e.g., "£10.00"
  pageOptions,          // Page config settings
  expiresAt,            // Basket expiration timestamp
  initiateCheckout,
  addOfferToBasket,
  removeFromBasket,
  updateItemQuantity,
  swapOffer,
  clearOrderItems,
  navigateToCheckout,
  redeemPromoCode,
  removePromoCode,
  updateCustomField,
  setCheckoutDisabled,
  validateBasket,
  updateBasketDetails,
  selectOfferForSubscriptionUpdate,
} = useBasket()
```

### Add to Basket Pattern
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

### Key Methods
- `initiateCheckout({ order: { orderItems: [{ offer }] } })` - Create new basket
- `addOfferToBasket({ offer, quantity?, type?, parentId? })` - Add to existing basket
- `removeFromBasket({ id })` - Remove by OrderItem ID
- `updateItemQuantity(itemId, quantity)` - Update quantity
- `swapOffer(itemId, offer)` - Replace item with different offer
- `clearOrderItems()` - Empty basket
- `navigateToCheckout()` - Go to checkout page
- `redeemPromoCode(promoCode)` - Apply discount
- `removePromoCode(promoCode)` - Remove discount
- `updateBasketDetails(details)` - Update basket metadata
- `selectOfferForSubscriptionUpdate(offer)` - Designate offer for subscription change

---

## Limio SDK - User

### useUser
```javascript
import { useUser } from "@limio/sdk"

const { attributes, subscriptions, loginStatus, loaded, token } = useUser()
```

**Returns:**
- `attributes` - User identity: `{ email, email_verified, firstName, lastName, sub, crm_id, ... }`
- `subscriptions` - Array of user's subscriptions
- `loginStatus` - "logged-in" or other states
- `loaded` - Boolean for data availability
- `token` - JWT access token

### useSubscriptions
```javascript
import { useSubscriptions } from "@limio/sdk"

const { subscriptions } = useSubscriptions()
```

**Subscription object:**
```javascript
{
  name: "Premium",
  status: "active",              // "active" | "cancelled" | etc.
  id: "sub-...",
  reference: "1KPEEEJ8RNF8",    // Customer-facing ref
  created: "2024-01-15T...",
  record_type: "subscription",
  mode: "production",
  offers: [                       // Array of offers — the documented access pattern
    {
      data: {
        start: "2024-01-15T...",
        end: null,                // null if still active
        record_subtype: "base",   // "discount" = discount offer; anything else (or absent) = standard offer
        offer: {                  // Full offer object with data.attributes etc.
          data: {
            attributes: { display_name__limio, price__limio, term__limio, ... },
            products: [{ name: "Product Name", attributes: { display_name__limio, product_code__limio } }]
          }
        }
      }
    }
  ],
  schedule: [                    // Payment schedule
    {
      id: "schedule-...",        // Unique ID — use as React key
      data: { date, amount, currency, description, type: "payment" },
      status: "active"           // "active" | "pending" | "pending-external" | "cancelled"
    }
  ]
}
```

**Important:** Always access offers via `subscription.offers[]` — this is the documented pattern. A subscription can have multiple offers (e.g. a standard offer + a discount offer). Do NOT use `subscription.data.offer` as that is a legacy field. To get the current standard offer, filter `subscription.offers` where `record_subtype` is NOT `"discount"` and check `start`/`end` dates.

### useSubInfo
```javascript
import { useSubInfo } from "@limio/sdk"

const { status, isGift, quantity, hasLapsed, hasPendingChange } = useSubInfo(subscription)
```

### useSchedule
```javascript
import { useSchedule } from "@limio/sdk"

const { nextPaymentAmount, renewalPrice, termStartDate, termEndDate } = useSchedule(subscription)
// Returns formatted values: "£9.99", "14 Dec 2024"
```

### useUserInvoices
```javascript
const { invoices, revalidate, mutate } = useUserInvoices()
```

### Subscription Utility Functions
```javascript
import {
  getCurrentAddress,      // (type, addresses) => address object
  getPriceFromSchedule,   // (schedule, country?) => { value, currencyCode }
  getCurrentOffer,        // (subscription) => offer
  getPeriodForOffer,      // (offer) => "1 month" | "1 year" | "N/A"
  getRenewalDateForUserSubscription,  // (subscription) => formatted date
  getPriceForUserSubscription,        // (subscription) => formatted price
} from "@limio/sdk"
```

### Subscription References
Use `subscription.reference` or `subscription.id` for linking between pages. Pass as URL query params (e.g. `?subRef=...` or `?subId=...`).

---

## Limio SDK - Pricing

### useCheckout
```javascript
import { useCheckout } from "@limio/sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: true })
const checkoutState = useCheckoutSelector(state => state) || {}
const { order, paidSchedule, schedule, locale } = checkoutState
const orderTotals = useCheckoutSelector((state) => state.display.orderTotal)
```

**orderTotals object:**
- `orderSubtotal` - Before discounts/tax
- `orderTotal` - Final total
- `currency` - "USD", "GBP", etc.
- `taxSummary` - Array of `{ taxCode, taxAmount, taxRate }`

### usePreview
```javascript
import { usePreview } from "@limio/sdk"

const { loadingPreview, isTaxPreviewCountry, taxCalculated } = usePreview()
```

---

## Limio SDK - Context

### useLimioContext
```javascript
import { useLimioContext } from "@limio/sdk"

const { isInPageBuilder } = useLimioContext() || {}
```

### Page Builder Compatibility

When `isInPageBuilder` is true, the component is being rendered in the Limio Page Builder editor. Components using `position: fixed` or `position: absolute` can break out of their designated section and interfere with the Page Builder UI.

**Always ensure components stay within their section bounds in Page Builder**, even if they're designed to float/stick in production:

```javascript
const { isInPageBuilder } = useLimioContext() || {}

const headerClasses = [
    "header",
    isInPageBuilder ? "header--static" : ""
].filter(Boolean).join(" ")

return <header className={headerClasses}>...</header>
```

```css
.header {
    position: fixed;  /* Floats in production */
    top: 0;
    z-index: 1000;
}

.header--static {
    position: relative;  /* Stays in section in Page Builder */
}
```

---

## SDK Utilities

All imported from `@limio/sdk`:

### HTML Sanitization
```javascript
import { sanitiseHTML } from "@limio/sdk"

<div dangerouslySetInnerHTML={{ __html: sanitiseHTML(offer.data.attributes.offer_features__limio) }} />
```

**Note:** `sanitiseHTML` uses DOMPurify and adds security attributes like `rel="noopener noreferrer"` to links. Prefer this over the `xss` npm package for rich text content. For Storybook compatibility (where `sanitiseHTML` may not be mocked), you can fall back to `xss` as a dependency.

### Error Boundary
```javascript
import { ErrorBoundary } from "@limio/sdk"

<ErrorBoundary ErrorUI={({ error }) => <p>Error: {error.message}</p>}>
    <RiskyChild />
</ErrorBoundary>
```

Also available as HOC: `withErrorBoundary(Component, ErrorUI)`

### Date & Currency Formatting
```javascript
import { formatDate, formatCurrency, formatCurrencyForCurrentLocale } from "@limio/sdk"

formatDate("2024-01-15T00:00:00Z", "DATE_FULL")   // "January 15, 2024"
formatCurrency("20.00", "GBP")                      // "£20.00"
formatCurrencyForCurrentLocale(20, "GBP")            // Locale-aware
```

`formatDate` formats: `"DATE_EN"`, `"DATE_FULL"`, `"DATE_SHORT"`, `"DATE_MED"`

### Display Price Formatting
```javascript
import { formatDisplayPrice } from "@limio/sdk"

formatDisplayPrice("{{currencySymbol}}{{amount}}/mo", offer.data.attributes.price__limio)
```

Placeholders: `{{currencyCode}}`, `{{currencySymbol}}`, `{{currencySymbolNative}}`, `{{amount}}`, `{{integerValue}}`, `{{decimalValue}}`, `{{formattedPrice}}`, `{{formattedPriceComma}}`

### Offer Info Helper
```javascript
import { useOfferInfo } from "@limio/sdk"

const info = useOfferInfo(offer)
// { allowMultibuy, offerDescription, hasRecurringCharge, isDelivery, productNames, isAutoRenew, offerImage, usesExternalPrice, isGift, displayName }
```

### Active Offer Filtering
```javascript
import { checkActiveOffers } from "@limio/sdk"

const activeOffers = checkActiveOffers(subscription.offers, false)
// Filters by start/end dates, sorted by start date
```

### Address Utilities
```javascript
import { addressSummary, formatCountry, getAddressMetadata, getCountryMetadata } from "@limio/sdk"

addressSummary(address)      // Formatted address string or "N/A"
formatCountry("GB")          // "United Kingdom"
getAddressMetadata("GB")     // { requiredAddressFields, addressFieldsToRender }
getCountryMetadata("GB")     // { name, "alpha-2", "alpha-3", "country-code" }
```

### App Settings
```javascript
import { LimioAppSettings } from "@limio/sdk"
const dateFormat = LimioAppSettings.getDateFormat()
```

### DateTime (Luxon)
```javascript
import { DateTime } from "@limio/sdk"
```

### Invoice Fetcher
```javascript
import { LimioFetchers } from "@limio/sdk"
const blob = await LimioFetchers.invoiceFetch(path, token)
```

---

## Offer Attachments

Offers can have image attachments. To find and display an offer's image:

```javascript
const attachments = offer?.data?.attachments || []

// Find image attachment
const imageAttachment = attachments.find(a =>
    a.type === "image" || (a.url && /\.(jpg|jpeg|png|gif|svg|webp)$/i.test(a.url))
)

// Use in component
{imageAttachment && (
    <img src={imageAttachment.url} alt={displayName} />
)}
```

---

## Common Utilities

### Contrast Color
When using configurable background colors for buttons, calculate contrasting text color:

```javascript
const getContrastColor = (hexColor) => {
    if (!hexColor) return "#000000"
    const hex = hexColor.replace("#", "")
    const r = parseInt(hex.substr(0, 2), 16)
    const g = parseInt(hex.substr(2, 2), 16)
    const b = parseInt(hex.substr(4, 2), 16)
    const luminance = (0.299 * r + 0.587 * g + 0.114 * b) / 255
    return luminance > 0.5 ? "#000000" : "#FFFFFF"
}

// Usage
<button style={{
    backgroundColor: accentColor,
    color: getContrastColor(accentColor)
}}>
    {ctaText}
</button>
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

## Best Practices

1. **Use SDK utilities** — `sanitiseHTML`, `formatCurrency`, `formatDate`, `useOfferInfo`, `checkActiveOffers`, `groupOffers`, `getCurrentOffer`, `useSchedule`, `useSubInfo`, etc. Don't reimplement what the SDK provides.
2. **Self-contained** — Do NOT import from `../source/utils/` or other internal paths. Use SDK utilities or inline helpers.
3. **Null safety** — Always use optional chaining and defaults
   ```javascript
   const { offers } = useCampaign() || {}
   const attributes = offer?.data?.attributes || {}
   ```
4. **All text configurable** — Every heading, label, button text, and URL should be a `limioProp` so the component is fully white-label.
5. **Color props** — Use color type limioProps for any configurable color. Pass through CSS custom properties.
6. **List props** — Items are `{id, label}` objects
7. **Picklist options** — Use `options` array with `{id, label, value}`
8. **Page Builder compatibility** — Components with `position: fixed/absolute` must fall back to `position: relative` when `isInPageBuilder` is true
9. **Loading states** — Handle `basketLoading` to prevent double submissions
10. **Sanitize HTML** — Use `xss` library or `sanitiseHTML` from SDK for rich text content
11. **MUI version** — Use 5.16.12 for React 19 compatibility
12. **Always create stories** — Every component should have a Storybook story with variations
13. **Subscription references** — Use `subscription.reference` or `subscription.id` for linking. Pass as URL query params (e.g. `?subRef=...`).
14. **Subscription offers access** — Always use `subscription.offers[]` array to access offers. Do NOT use `subscription.data.offer` (legacy). A subscription can have multiple offers (standard + discount), so filter where `record_subtype` is NOT `"discount"` and check `start`/`end` dates to find the current active standard offer.

---

## Storybook Setup (One-time)

If `component-playground/.storybook/main.js` does **not** exist, create the full Storybook playground. If it already exists, skip to "Creating a Story".

### Directory Structure

```
component-playground/
├── .storybook/
│   ├── main.js
│   ├── preview.js
│   ├── middleware.js
│   ├── claude-overlay.js
│   └── addon-prompt/
│       ├── manager.js
│       └── preset.js
├── packages/
│   └── limio/
│       ├── sdk/
│       │   ├── index.js
│       │   └── src/
│       │       └── context.js
│       ├── shop/
│       │   └── src/
│       │       └── shop/
│       │           └── checkout/
│       │               └── basket.js
│       └── internal-checkout-sdk/
│           └── index.js
├── scripts/
│   └── watch-prompts.js
├── src/
│   └── stories/
│       ├── LimioSetup.stories.js
│       └── NewComponent.stories.js
├── .prompt.json          (transient — add to .gitignore)
├── .prompt-status.json   (transient — add to .gitignore)
└── package.json
```

### component-playground/package.json

```json
{
  "name": "@limio/component-playground",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "ramda": "^0.28.0",
    "xss": "^1.0.15"
  },
  "scripts": {
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build"
  },
  "devDependencies": {
    "@storybook/addon-essentials": "^8.0.0",
    "@storybook/addon-interactions": "^8.0.0",
    "@storybook/addon-links": "^8.0.0",
    "@storybook/addon-webpack5-compiler-babel": "^1.0.0",
    "@storybook/blocks": "^8.0.0",
    "@storybook/react": "^8.0.0",
    "@storybook/react-webpack5": "^8.0.0",
    "storybook": "^8.0.0"
  }
}
```

### component-playground/.storybook/main.js

```javascript
import path, { dirname, join } from "path"

function getAbsolutePath(value) {
    return dirname(require.resolve(join(value, "package.json")))
}

const config = {
    stories: ["../src/**/*.stories.@(js|jsx|ts|tsx)"],
    addons: [
        getAbsolutePath("@storybook/addon-webpack5-compiler-babel"),
        getAbsolutePath("@storybook/addon-essentials"),
        getAbsolutePath("@storybook/addon-interactions"),
        path.resolve(__dirname, "addon-prompt"),
    ],
    framework: {
        name: getAbsolutePath("@storybook/react-webpack5"),
        options: {},
    },
    webpackFinal: async (config) => {
        config.resolve.alias = {
            ...config.resolve.alias,
            "@limio/sdk": path.resolve(__dirname, "..", "packages", "limio", "sdk"),
            "@limio/sdk/components": path.resolve(__dirname, "..", "packages", "limio", "sdk", "src", "components"),
            "@limio/shop": path.resolve(__dirname, "..", "packages", "limio", "shop"),
            "@limio/internal-checkout-sdk": path.resolve(__dirname, "..", "packages", "limio", "internal-checkout-sdk"),
        }
        return config
    }
}

export default config
```

### component-playground/.storybook/preview.js

```javascript
import React from "react"
import { ClaudeOverlay } from "./claude-overlay"

const preview = {
    parameters: {
        layout: "fullscreen",
        controls: {
            matchers: {
                color: /(background|color)$/i,
                date: /Date$/i,
            },
        },
    },
    decorators: [
        (Story) => (
            <>
                <Story />
                <ClaudeOverlay />
            </>
        ),
    ],
}

export default preview
```

### component-playground/.storybook/claude-overlay.js

This overlay shows a polished loading screen while Claude Code is working. It polls `/api/prompt-status` and displays animated phases, a pulsing ring logo, and success/error states with smooth transitions.

```javascript
import React, { useState, useEffect, useRef } from "react"

const PHASES = [
    { message: "Prompt received..." },
    { message: "Reading component files..." },
    { message: "Making magic..." },
    { message: "Writing code changes..." },
    { message: "Sprinkling some pixels..." },
    { message: "Almost there..." },
]

const COMPLETED_MESSAGES = [
    "Changes applied!",
    "All done — check it out!",
    "Component updated!",
]

function useStatusPoller() {
    const [status, setStatus] = useState({ state: "listening", message: "" })
    const prevState = useRef("listening")

    useEffect(() => {
        let active = true
        const poll = async () => {
            try {
                const res = await fetch("/api/prompt-status")
                if (res.ok && active) {
                    const data = await res.json()
                    setStatus(data)
                    prevState.current = data.state
                }
            } catch {}
        }
        poll()
        const id = setInterval(poll, 800)
        return () => { active = false; clearInterval(id) }
    }, [])

    return status
}

function PulsingRing() {
    return (
        <div style={styles.ringContainer}>
            <div style={{ ...styles.ring, ...styles.ring1 }} />
            <div style={{ ...styles.ring, ...styles.ring2 }} />
            <div style={{ ...styles.ring, ...styles.ring3 }} />
            <div style={styles.ringCenter}>
                <span style={styles.ringLogo}>C</span>
            </div>
        </div>
    )
}

export function ClaudeOverlay() {
    const status = useStatusPoller()
    const [phaseIndex, setPhaseIndex] = useState(0)
    const [visible, setVisible] = useState(false)
    const [exiting, setExiting] = useState(false)
    const phaseTimer = useRef(null)

    const isActive = status.state === "working" || status.state === "queued" || status.state === "received"
    const isCompleted = status.state === "completed"
    const isError = status.state === "error"

    // Show overlay when active
    useEffect(() => {
        if (isActive) {
            setVisible(true)
            setExiting(false)
            setPhaseIndex(0)
        }
    }, [isActive])

    // Cycle through phases while working
    useEffect(() => {
        if (isActive) {
            phaseTimer.current = setInterval(() => {
                setPhaseIndex(prev => (prev + 1) % PHASES.length)
            }, 2800)
            return () => clearInterval(phaseTimer.current)
        }
    }, [isActive])

    // Handle completed: show briefly then fade out
    useEffect(() => {
        if (isCompleted && visible) {
            clearInterval(phaseTimer.current)
            const timeout = setTimeout(() => {
                setExiting(true)
                setTimeout(() => {
                    setVisible(false)
                    setExiting(false)
                }, 600)
            }, 2000)
            return () => clearTimeout(timeout)
        }
    }, [isCompleted, visible])

    // Handle error: show briefly then fade out
    useEffect(() => {
        if (isError && visible) {
            clearInterval(phaseTimer.current)
            const timeout = setTimeout(() => {
                setExiting(true)
                setTimeout(() => {
                    setVisible(false)
                    setExiting(false)
                }, 600)
            }, 3000)
            return () => clearTimeout(timeout)
        }
    }, [isError, visible])

    if (!visible) return null

    const phase = PHASES[phaseIndex]
    const completedMsg = COMPLETED_MESSAGES[Math.floor(Math.random() * COMPLETED_MESSAGES.length)]
    const displayMessage = status.message || (isCompleted ? completedMsg : isError ? "Something went wrong" : phase.message)

    return (
        <div style={{
            ...styles.overlay,
            opacity: exiting ? 0 : 1,
            transition: "opacity 0.6s ease",
        }}>
            <style>{keyframes}</style>
            <div style={{
                ...styles.dialog,
                animation: exiting ? "claude-slideDown 0.5s ease forwards" : "claude-slideIn 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards",
            }}>
                <div style={styles.dialogInner}>
                    {isActive && <PulsingRing />}
                    {isCompleted && (
                        <div style={styles.successIcon}>
                            <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                                <circle cx="24" cy="24" r="24" fill="#10B981" />
                                <path d="M15 24.5L21 30.5L33 18.5" stroke="white" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round" style={{ strokeDasharray: 30, strokeDashoffset: 0, animation: "claude-checkDraw 0.5s ease 0.2s both" }} />
                            </svg>
                        </div>
                    )}
                    {isError && (
                        <div style={styles.errorIcon}>
                            <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                                <circle cx="24" cy="24" r="24" fill="#EF4444" />
                                <path d="M17 17L31 31M31 17L17 31" stroke="white" strokeWidth="3" strokeLinecap="round" />
                            </svg>
                        </div>
                    )}

                    <div style={styles.messageArea}>
                        <p style={{
                            ...styles.message,
                            color: isCompleted ? "#10B981" : isError ? "#EF4444" : "#1a1f36",
                        }}>
                            {displayMessage}
                        </p>
                    </div>
                </div>
            </div>
        </div>
    )
}

const keyframes = `
@keyframes claude-slideIn {
    from { opacity: 0; transform: translateY(30px) scale(0.95); }
    to   { opacity: 1; transform: translateY(0) scale(1); }
}
@keyframes claude-slideDown {
    from { opacity: 1; transform: translateY(0) scale(1); }
    to   { opacity: 0; transform: translateY(-20px) scale(0.95); }
}
@keyframes claude-pulse {
    0%, 100% { transform: scale(1); opacity: 0.3; }
    50%      { transform: scale(1.6); opacity: 0; }
}
@keyframes claude-pulse2 {
    0%, 100% { transform: scale(1); opacity: 0.2; }
    50%      { transform: scale(1.8); opacity: 0; }
}
@keyframes claude-pulse3 {
    0%, 100% { transform: scale(1); opacity: 0.15; }
    50%      { transform: scale(2); opacity: 0; }
}
@keyframes claude-spin {
    from { transform: translate(-50%, -50%) rotate(0deg); }
    to   { transform: translate(-50%, -50%) rotate(360deg); }
}
@keyframes claude-checkDraw {
    from { stroke-dashoffset: 30; }
    to   { stroke-dashoffset: 0; }
}
`

const styles = {
    overlay: {
        position: "fixed",
        inset: 0,
        zIndex: 999999,
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        background: "rgba(15, 23, 42, 0.4)",
        backdropFilter: "blur(8px)",
        WebkitBackdropFilter: "blur(8px)",
    },
    dialog: {
        background: "#FFFFFF",
        borderRadius: "24px",
        padding: "40px 48px",
        minWidth: "380px",
        maxWidth: "440px",
        boxShadow: "0 25px 60px rgba(0, 0, 0, 0.15), 0 0 0 1px rgba(0, 0, 0, 0.05)",
        textAlign: "center",
        fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif',
    },
    dialogInner: {
        display: "flex",
        flexDirection: "column",
        alignItems: "center",
        gap: "20px",
    },
    ringContainer: {
        position: "relative",
        width: "80px",
        height: "80px",
    },
    ring: {
        position: "absolute",
        inset: 0,
        borderRadius: "50%",
        border: "2px solid #635BFF",
    },
    ring1: { animation: "claude-pulse 2s ease-in-out infinite" },
    ring2: { animation: "claude-pulse2 2s ease-in-out 0.4s infinite" },
    ring3: { animation: "claude-pulse3 2s ease-in-out 0.8s infinite" },
    ringCenter: {
        position: "absolute",
        top: "50%",
        left: "50%",
        transform: "translate(-50%, -50%)",
        width: "48px",
        height: "48px",
        borderRadius: "14px",
        background: "linear-gradient(135deg, #d4a574 0%, #c4956a 100%)",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        boxShadow: "0 4px 12px rgba(196, 149, 106, 0.3)",
    },
    ringLogo: {
        color: "#fff",
        fontSize: "20px",
        fontWeight: "700",
    },
    messageArea: {
        display: "flex",
        flexDirection: "column",
        alignItems: "center",
        gap: "8px",
        minHeight: "60px",
        justifyContent: "center",
    },
    message: {
        fontSize: "16px",
        fontWeight: "600",
        margin: 0,
        lineHeight: 1.4,
        letterSpacing: "-0.01em",
        transition: "color 0.3s ease",
    },
    successIcon: {
        animation: "claude-slideIn 0.4s ease",
    },
    errorIcon: {
        animation: "claude-slideIn 0.4s ease",
    },
}
```

### component-playground/packages/limio/sdk/index.js

```javascript
export * from "./src/context"

export function getPropsFromPackageJson(packageData) {
    const limioProps = packageData.limioProps || []
    const defaults = {}
    limioProps.forEach(prop => {
        if (prop.default !== undefined) {
            defaults[prop.id] = prop.default
        }
    })
    return defaults
}
```

### component-playground/packages/limio/sdk/src/context.js

```javascript
import * as React from "react"

const LimioContext = React.createContext({})
export const ComponentContext = React.createContext({})

// ===== Mock Data =====

const mockOffers = [
    {
        id: "offer-monthly-001", name: "Monthly Plan", path: "/offers/monthly", type: "item",
        data: {
            attributes: {
                display_name__limio: "Monthly", display_price__limio: "<p>$9.99/mo</p>",
                detailed_display_price__limio: "<p>Billed monthly</p>", cta_text__limio: "Subscribe",
                group__limio: "monthly", best_value__limio: false,
                offer_features__limio: "<ul><li>Unlimited access</li><li>Cancel anytime</li></ul>",
                payment_types__limio: ["card"], checkout_description__limio: "Monthly subscription",
                price__limio: [{ type: "recurring", value: 9.99, currencyCode: "USD" }],
                term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" },
            },
            price: [{ value: 9.99, currencyCode: "USD", type: "recurring", trigger: "subscription_start", repeat_interval: 1, repeat_interval_type: "months" }],
            products: [{ path: "/products/standard", name: "Standard", attributes: { display_name__limio: "Standard Plan", product_code__limio: "STANDARD" } }],
            attachments: []
        }
    },
    {
        id: "offer-annual-002", name: "Annual Plan", path: "/offers/annual", type: "item",
        data: {
            attributes: {
                display_name__limio: "Annual", display_price__limio: "<p><s>$119.88</s> $99.99/yr</p>",
                detailed_display_price__limio: "<p>Billed annually — save 17%</p>", cta_text__limio: "Subscribe & Save",
                group__limio: "annual", best_value__limio: true, badge_text__limio: "Best Value",
                offer_features__limio: "<ul><li>Unlimited access</li><li>Priority support</li><li>Cancel anytime</li></ul>",
                payment_types__limio: ["card", "paypal"], checkout_description__limio: "Annual subscription",
                price__limio: [{ type: "recurring", value: 99.99, currencyCode: "USD" }],
                term__limio: { type: "years", length: 1, renewal_trigger: "auto", renewal_type: "term" },
            },
            price: [{ value: 99.99, currencyCode: "USD", type: "recurring", trigger: "subscription_start", repeat_interval: 1, repeat_interval_type: "years" }],
            products: [{ path: "/products/standard", name: "Standard", attributes: { display_name__limio: "Standard Plan", product_code__limio: "STANDARD" } }],
            attachments: []
        }
    },
    {
        id: "offer-premium-003", name: "Premium Monthly", path: "/offers/premium", type: "item",
        data: {
            attributes: {
                display_name__limio: "Premium", display_price__limio: "<p>$19.99/mo</p>",
                detailed_display_price__limio: "<p>Billed monthly</p>", cta_text__limio: "Go Premium",
                group__limio: "monthly", best_value__limio: false,
                offer_features__limio: "<ul><li>Everything in Standard</li><li>Advanced analytics</li><li>API access</li><li>Dedicated support</li></ul>",
                payment_types__limio: ["card", "paypal"], checkout_description__limio: "Premium monthly subscription",
                price__limio: [{ type: "recurring", value: 19.99, currencyCode: "USD" }],
                term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" },
            },
            price: [{ value: 19.99, currencyCode: "USD", type: "recurring", trigger: "subscription_start", repeat_interval: 1, repeat_interval_type: "months" }],
            products: [{ path: "/products/premium", name: "Premium", attributes: { display_name__limio: "Premium Plan", product_code__limio: "PREMIUM" } }],
            attachments: []
        }
    }
]

const mockBasketItems = [
    {
        name: "Monthly Plan", id: "basket-item-001",
        offer: mockOffers[0], details: "",
        price: { summary: { headline: "<p>$9.99/mo</p>" }, currency: "USD", amount: 9.99 },
        products: mockOffers[0].data.products
    }
]

const mockUser = {
    username: "mock-user-001",
    attributes: { email: "alex@example.com", email_verified: true, firstName: "Alex", lastName: "Johnson", sub: "mock-user-001" },
    subscriptions: [
        {
            name: "Pro Plan Monthly", status: "active", record_type: "subscription",
            id: "sub-001", reference: "REF001", created: "2024-01-15T00:00:00Z", mode: "production",
            offers: [{
                name: "Pro Plan", quantity: 1,
                data: {
                    start: "2024-01-15T00:00:00Z", record_subtype: "base",
                    offer: {
                        data: {
                            attributes: { display_name__limio: "Pro Plan", price__limio: [{ type: "recurring", value: 9.99, currencyCode: "USD" }], term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" } },
                            products: [{ name: "Pro Access", attributes: { display_name__limio: "Pro Access", product_code__limio: "STANDARD" } }]
                        }
                    }
                },
                price: { summary: { headline: "$9.99/mo" }, currency: "USD", amount: 9.99 }, products: []
            }],
            schedule: [
                { id: "sched-001", data: { date: "2024-01-15T00:00:00Z", amount: "9.99", currency: "USD", type: "payment", description: "Pro Plan — Monthly" }, status: "active" },
                { id: "sched-002", data: { date: "2024-02-15T00:00:00Z", amount: "9.99", currency: "USD", type: "payment", description: "Pro Plan — Monthly" }, status: "active" },
                { id: "sched-003", data: { date: "2027-07-15T00:00:00Z", amount: "9.99", currency: "USD", type: "payment", description: "Pro Plan — Monthly" }, status: "active" }
            ]
        },
        {
            name: "Enterprise Annual", status: "active", record_type: "subscription",
            id: "sub-002", reference: "REF002", created: "2024-03-15T09:30:00Z", mode: "production",
            offers: [{
                name: "Enterprise Plan", quantity: 1,
                data: {
                    start: "2024-03-15T09:30:00Z", record_subtype: "base",
                    offer: {
                        data: {
                            attributes: { display_name__limio: "Enterprise Plan", price__limio: [{ type: "recurring", value: 499, currencyCode: "USD" }], term__limio: { type: "years", length: 1, renewal_trigger: "auto", renewal_type: "term" } },
                            products: [{ name: "Enterprise Access", attributes: { display_name__limio: "Enterprise Access", product_code__limio: "ENTERPRISE" } }]
                        }
                    }
                },
                price: { summary: { headline: "$499/year" }, currency: "USD", amount: 499 }, products: []
            }],
            schedule: [
                { id: "sched-010", data: { date: "2024-03-15T09:30:00Z", amount: "499.00", currency: "USD", type: "payment", description: "Enterprise Plan — Annual" }, status: "active" },
                { id: "sched-011", data: { date: "2027-03-15T09:30:00Z", amount: "499.00", currency: "USD", type: "payment", description: "Enterprise Plan — Annual" }, status: "active" }
            ]
        },
        {
            name: "Starter Monthly", status: "cancelled", record_type: "subscription",
            id: "sub-003", reference: "REF003", created: "2023-06-01T08:00:00Z", mode: "production",
            offers: [{
                name: "Starter Plan", quantity: 1,
                data: {
                    start: "2023-06-01T08:00:00Z", end: "2023-12-01T08:00:00Z", record_subtype: "base",
                    offer: {
                        data: {
                            attributes: { display_name__limio: "Starter Plan", price__limio: [{ type: "recurring", value: 4.99, currencyCode: "USD" }], term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" } },
                            products: [{ name: "Starter Access", attributes: { display_name__limio: "Starter Access", product_code__limio: "STARTER" } }]
                        }
                    }
                },
                price: { summary: { headline: "$4.99/mo" }, currency: "USD", amount: 4.99 }, products: []
            }],
            schedule: [
                { id: "sched-020", data: { date: "2023-06-01T08:00:00Z", amount: "4.99", currency: "USD", type: "payment", description: "Starter Plan — Monthly" }, status: "active" },
                { id: "sched-021", data: { date: "2023-11-01T08:00:00Z", amount: "4.99", currency: "USD", type: "payment", description: "Starter Plan — Monthly" }, status: "cancelled" }
            ]
        }
    ],
    loginStatus: "logged-in", loaded: true, token: "mock-jwt-token"
}

const dummyContext = {
    pageBuilder__limio: false,
    shop: {
        campaign: { name: "Demo Campaign", path: "/campaigns/demo", attributes: { push_to_checkout__limio: true } },
        offers: mockOffers,
        addOns: [],
        tag: "/tags/demo",
        basketItems: mockBasketItems,
        addToBasket: (offer) => console.log("Added to basket:", offer),
    },
    user: mockUser
}

// ===== Hooks =====

export function useCampaign() {
    React.useContext(LimioContext)
    const { campaign, offers, addOns } = dummyContext.shop
    return { campaign, offers, addOns }
}

export function useBasket() {
    React.useContext(LimioContext)
    const { basketItems, addToBasket } = dummyContext.shop
    return {
        orderItems: basketItems, basketLoading: false, formattedTotal: "$9.99",
        initiateCheckout: async (data) => console.log("Checkout initiated:", data),
        addOfferToBasket: async (data) => console.log("Added:", data),
        removeFromBasket: async (data) => console.log("Removed:", data),
        navigateToCheckout: async () => console.log("Navigate to checkout"),
        clearOrderItems: () => console.log("Cart cleared"),
    }
}

export function useUser() {
    React.useContext(LimioContext)
    return mockUser
}

export function useSubscriptions() {
    React.useContext(LimioContext)
    return { subscriptions: mockUser.subscriptions }
}

export function useLimioContext() {
    React.useContext(LimioContext)
    return { isInPageBuilder: false }
}

export function useComponentProps(defaultProps) {
    const context = React.useContext(ComponentContext)
    return React.useMemo(() => ({ ...defaultProps, ...context }), [context, defaultProps])
}

export function useCheckout() {
    return {
        useCheckoutSelector: (callback) => callback({
            order: { orderDate: new Date().toISOString(), basketItems: mockBasketItems, orderItems: mockBasketItems, customerDetails: { firstName: "Alex", lastName: "Johnson", email: "alex@example.com" } },
            display: { orderTotal: { orderSubtotal: "$9.99", orderTotal: "$9.99", currency: "USD", taxSummary: [] } }
        })
    }
}

export function groupOffers(offers = [], groupLabels = []) {
    const groups = {}
    for (const offer of offers) {
        const group = offer?.data?.attributes?.group__limio || "other"
        groups[group] = groups[group] || []
        groups[group].push(offer)
    }
    return Object.keys(groups).map(groupId => {
        const match = groupLabels.find(g => g.id === groupId) || { id: groupId, label: groupId, thumbnail: "" }
        return { groupId, id: groupId, label: match.label, offers: groups[groupId], thumbnail: match.thumbnail }
    })
}

export function formatCurrencyForCurrentLocale(amount, currency) {
    return new Intl.NumberFormat("en-US", { style: "currency", currency }).format(amount)
}

export function ErrorBoundary({ children }) {
    return <>{children}</>
}

// ===== Provider =====

export function LimioProvider({ children, value = dummyContext }) {
    return <LimioContext.Provider value={value}>{children}</LimioContext.Provider>
}
```

### component-playground/packages/limio/shop/src/shop/checkout/basket.js

```javascript
export function getCurrentBasketId() {
    return "mock-basket-id"
}
```

### component-playground/packages/limio/internal-checkout-sdk/index.js

```javascript
import { useCheckout } from "@limio/sdk"
export { useCheckout }
```

After creating all files, install dependencies:
```bash
cd component-playground && npm install
```

---

## Claude Prompt Addon (Create or Update)

If `component-playground/.storybook/addon-prompt/manager.js` does **not** exist, create the Claude Prompt addon. If it exists but is outdated (see "Addon Version Checking" below), replace both `manager.js` and `middleware.js` with the current templates. This adds a panel to Storybook where users can type prompts that Claude Code picks up and processes automatically.

### Addon Version Checking

When the addon already exists, check these markers to determine if it needs updating:

| Feature | File | Marker |
|---------|------|--------|
| Settings panel | manager.js | `SETTINGS_PANEL_ID` |
| Deploy endpoint | middleware.js | `/api/deploy` |
| Limio config | middleware.js | `readLimioConfig` |
| Build status | middleware.js | `/api/build-status` |
| Loading overlay | claude-overlay.js | `ClaudeOverlay` |
| New component tool | src/stories/NewComponent.stories.js | `Tools/New Component` |
| Overlay decorator | preview.js | `ClaudeOverlay` |

If **any** marker is missing from its respective file, replace **both** `manager.js` and `middleware.js` with the current templates below. If `claude-overlay.js` is missing, create it and update `preview.js`. If `NewComponent.stories.js` is missing, create it. This ensures all features stay in sync.

### How it works

1. User types a prompt in the Storybook "Claude Prompt" panel and clicks "Send to Claude"
2. Storybook's Express middleware writes the prompt to `component-playground/.prompt.json`
3. The watcher script (`scripts/watch-prompts.js`) detects the change and exits
4. Claude Code gets notified, reads the prompt, applies changes to the component
5. Storybook hot-reloads with the updated component

### component-playground/.storybook/middleware.js

```javascript
const fs = require("fs")
const path = require("path")
const { execSync } = require("child_process")

const PROMPT_FILE = path.resolve(__dirname, "..", ".prompt.json")
const STATUS_FILE = path.resolve(__dirname, "..", ".prompt-status.json")
const PROJECT_ROOT = path.resolve(__dirname, "..", "..")
const CONFIG_FILE = path.join(PROJECT_ROOT, ".limio.json")

// --- Limio config + token management ---

function readLimioConfig() {
    try {
        if (!fs.existsSync(CONFIG_FILE)) return null
        const raw = JSON.parse(fs.readFileSync(CONFIG_FILE, "utf8"))
        if (!raw.tenant || !raw.clientId || !raw.clientSecret) return null
        return raw
    } catch {
        return null
    }
}

function getLimioBaseUrl(config) {
    const region = (config.region || "eu").toLowerCase()
    if (region === "us") return `https://${config.tenant}.prod-us.limio.com`
    if (region === "dev") return `https://${config.tenant}.dev.limio.com`
    return `https://${config.tenant}.prod.limio.com`
}

let tokenCache = { token: null, expiresAt: 0 }

async function getAccessToken(config) {
    const now = Date.now()
    if (tokenCache.token && tokenCache.expiresAt > now + 60000) {
        return tokenCache.token
    }
    const baseUrl = getLimioBaseUrl(config)
    const params = new URLSearchParams({
        grant_type: "client_credentials",
        client_id: config.clientId,
        client_secret: config.clientSecret,
    })
    const res = await fetch(`${baseUrl}/oauth2/token`, {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: params.toString(),
    })
    if (!res.ok) {
        const text = await res.text().catch(() => "")
        throw new Error(`Auth failed (${res.status}): ${text}`)
    }
    const data = await res.json()
    tokenCache = {
        token: data.access_token,
        expiresAt: now + (data.expires_in || 3600) * 1000,
    }
    return tokenCache.token
}

// --- Helpers ---

function readBody(req) {
    return new Promise((resolve) => {
        let body = ""
        req.on("data", chunk => { body += chunk })
        req.on("end", () => {
            try { resolve(JSON.parse(body)) } catch { resolve({}) }
        })
    })
}

function sendJson(res, statusCode, data) {
    res.statusCode = statusCode
    res.setHeader("Content-Type", "application/json")
    res.end(JSON.stringify(data))
}

module.exports = function expressMiddleware(app) {
    // Auto-reset stale prompt status from interrupted sessions
    try {
        if (fs.existsSync(STATUS_FILE)) {
            const status = JSON.parse(fs.readFileSync(STATUS_FILE, "utf8"))
            if (["queued", "received", "working"].includes(status.state)) {
                fs.writeFileSync(STATUS_FILE, JSON.stringify(
                    { state: "listening", message: "", timestamp: new Date().toISOString() }, null, 2
                ))
            }
        }
    } catch {}

    app.use("/api/prompt", async (req, res, next) => {
        if (req.method === "POST") {
            const body = await readBody(req)
            const { prompt, component, storyId, mode } = body
            if (!prompt) return sendJson(res, 400, { error: "prompt is required" })
            const data = { prompt, component: component || "unknown", storyId: storyId || "", mode: mode || "edit", timestamp: new Date().toISOString() }
            try {
                fs.writeFileSync(PROMPT_FILE, JSON.stringify(data, null, 2))
                const statusData = { state: "queued", message: "Prompt sent — waiting for Claude Code...", timestamp: new Date().toISOString() }
                fs.writeFileSync(STATUS_FILE, JSON.stringify(statusData, null, 2))
                sendJson(res, 200, { success: true })
            } catch (err) {
                console.error("Error writing prompt:", err)
                sendJson(res, 500, { error: err.message })
            }
        } else if (req.method === "GET") {
            try {
                if (fs.existsSync(PROMPT_FILE)) {
                    sendJson(res, 200, JSON.parse(fs.readFileSync(PROMPT_FILE, "utf8")))
                } else {
                    sendJson(res, 200, { prompt: "", component: "", storyId: "", timestamp: "" })
                }
            } catch { sendJson(res, 200, { prompt: "", component: "", storyId: "", timestamp: "" }) }
        } else {
            next()
        }
    })

    app.use("/api/prompt-status", async (req, res, next) => {
        if (req.method === "POST") {
            const body = await readBody(req)
            const { state, message } = body
            if (!state) return sendJson(res, 400, { error: "state is required" })
            try {
                const data = { state, message: message || "", timestamp: new Date().toISOString() }
                fs.writeFileSync(STATUS_FILE, JSON.stringify(data, null, 2))
                sendJson(res, 200, { success: true })
            } catch (err) {
                sendJson(res, 500, { error: err.message })
            }
        } else if (req.method === "GET") {
            try {
                if (fs.existsSync(STATUS_FILE)) {
                    sendJson(res, 200, JSON.parse(fs.readFileSync(STATUS_FILE, "utf8")))
                } else {
                    sendJson(res, 200, { state: "listening", message: "" })
                }
            } catch { sendJson(res, 200, { state: "listening", message: "" }) }
        } else {
            next()
        }
    })

    app.use("/api/deploy", async (req, res, next) => {
        if (req.method === "POST") {
            const body = await readBody(req)
            const { component } = body
            if (!component) return sendJson(res, 400, { error: "component is required" })
            try {
                const componentDir = path.join("components", component)
                const componentPath = path.join(PROJECT_ROOT, componentDir)
                if (!fs.existsSync(componentPath)) return sendJson(res, 400, { error: `Component folder not found: ${componentDir}` })
                execSync(`git add ${componentDir}/`, { cwd: PROJECT_ROOT })
                const storiesDir = path.join(PROJECT_ROOT, "component-playground", "src", "stories")
                if (fs.existsSync(storiesDir)) {
                    const storyFiles = fs.readdirSync(storiesDir).filter(f => f.endsWith(".stories.js") || f.endsWith(".stories.jsx"))
                    for (const file of storyFiles) {
                        const content = fs.readFileSync(path.join(storiesDir, file), "utf8")
                        if (content.includes(component)) {
                            execSync(`git add component-playground/src/stories/${file}`, { cwd: PROJECT_ROOT })
                        }
                    }
                }
                execSync(`git commit -m "Deploy component: ${component}"`, { cwd: PROJECT_ROOT })
                const commitHash = execSync("git rev-parse HEAD", { cwd: PROJECT_ROOT }).toString().trim()
                execSync("git push", { cwd: PROJECT_ROOT })
                sendJson(res, 200, { success: true, message: `Deployed ${component} successfully`, commitHash })
            } catch (err) {
                console.error("Deploy error:", err.message)
                sendJson(res, 500, { error: err.message })
            }
        } else if (req.method === "GET") {
            try {
                const branch = execSync("git rev-parse --abbrev-ref HEAD", { cwd: PROJECT_ROOT }).toString().trim()
                const status = execSync("git status --porcelain", { cwd: PROJECT_ROOT }).toString().trim()
                sendJson(res, 200, { branch, clean: status.length === 0, status })
            } catch (err) {
                sendJson(res, 500, { error: err.message })
            }
        } else {
            next()
        }
    })

    // --- Limio connection status ---
    app.use("/api/limio/status", async (req, res, next) => {
        if (req.method !== "GET") return next()
        const config = readLimioConfig()
        if (!config) return sendJson(res, 200, { configured: false })
        try {
            await getAccessToken(config)
            sendJson(res, 200, { configured: true, tenant: config.tenant, region: config.region || "eu", baseUrl: getLimioBaseUrl(config) })
        } catch (err) {
            sendJson(res, 200, { configured: false, error: `Credentials invalid: ${err.message}` })
        }
    })

    // --- Save Limio credentials ---
    app.use("/api/limio/setup", async (req, res, next) => {
        if (req.method !== "POST") return next()
        const body = await readBody(req)
        const { tenant, region, clientId, clientSecret } = body
        if (!tenant || !clientId || !clientSecret) {
            return sendJson(res, 400, { error: "tenant, clientId, and clientSecret are required" })
        }
        const config = { tenant, region: region || "eu", clientId, clientSecret }
        try {
            tokenCache = { token: null, expiresAt: 0 }
            await getAccessToken(config)
        } catch (err) {
            return sendJson(res, 400, { error: `Authentication failed: ${err.message}` })
        }
        try {
            fs.writeFileSync(CONFIG_FILE, JSON.stringify(config, null, 2))
            sendJson(res, 200, { success: true, tenant, region: config.region, baseUrl: getLimioBaseUrl(config) })
        } catch (err) {
            sendJson(res, 500, { error: `Failed to save config: ${err.message}` })
        }
    })

    // --- Build status proxy ---
    app.use("/api/build-status", async (req, res, next) => {
        if (req.method !== "GET") return next()
        const url = new URL(req.url, "http://localhost")
        const commitHash = url.searchParams.get("commitHash")
        if (!commitHash) return sendJson(res, 400, { error: "commitHash is required" })
        const config = readLimioConfig()
        if (!config) return sendJson(res, 400, { error: "Limio not configured" })
        try {
            const token = await getAccessToken(config)
            const baseUrl = getLimioBaseUrl(config)
            const apiRes = await fetch(`${baseUrl}/api/component/builds?commitHash=${encodeURIComponent(commitHash)}`, {
                headers: { Authorization: `Bearer ${token}` },
            })
            if (!apiRes.ok) {
                const text = await apiRes.text().catch(() => "")
                return sendJson(res, apiRes.status, { found: false, error: `Limio API error (${apiRes.status}): ${text}` })
            }
            const data = await apiRes.json()
            if (!data || (Array.isArray(data) && data.length === 0)) {
                return sendJson(res, 200, { found: false })
            }
            const build = Array.isArray(data) ? data[0] : data
            sendJson(res, 200, {
                found: true,
                buildStatus: build.status || build.buildStatus || "UNKNOWN",
                buildComplete: ["SUCCEEDED", "FAILED", "ERROR"].includes((build.status || build.buildStatus || "").toUpperCase()),
                logErrors: build.logErrors || build.errors || null,
                startTime: build.startTime || build.createdAt || null,
                endTime: build.endTime || build.completedAt || null,
            })
        } catch (err) {
            sendJson(res, 500, { error: err.message })
        }
    })
}
```

### component-playground/.storybook/addon-prompt/preset.js

```javascript
module.exports = {
    managerEntries(entry = []) {
        return [...entry, require.resolve("./manager")]
    },
}
```

### component-playground/.storybook/addon-prompt/manager.js

The manager.js file registers two panels: **Claude Prompt** (for sending prompts) and **Limio Settings** (for configuring Limio credentials). It includes real-time backend status polling, deploy functionality with Limio build tracking, and the Limio connection settings form.

The full file is extensive (~540 lines). Key features to include when creating it:

- **PromptPanel** — textarea + send button, real-time backend status polling (`/api/prompt-status`), deploy button with build status tracking, `+ New` button to navigate to new-component story
- **SettingsPanel** — Limio credential form with tenant, region (EU/US/Dev), client ID, client secret; URL preview that shows the correct domain per region; connects via `/api/limio/setup`
- **Region dropdown** must include all three options:
  ```jsx
  <option value="eu">EU (Europe)</option>
  <option value="us">US (United States)</option>
  <option value="dev">Dev (Development)</option>
  ```
- **URL preview** must handle all three regions:
  ```jsx
  {region === "us" ? `${tenant}.prod-us.limio.com` : region === "dev" ? `${tenant}.dev.limio.com` : `${tenant}.prod.limio.com`}
  ```
- **Registration:** `addons.register` with both `PANEL_ID` (Claude Prompt) and `SETTINGS_PANEL_ID` (Limio Settings)

Use the actual file at `component-playground/.storybook/addon-prompt/manager.js` as the source of truth — it evolves faster than this template.

### component-playground/scripts/watch-prompts.js

```javascript
const fs = require("fs")
const path = require("path")

const PROMPT_FILE = path.resolve(__dirname, "..", ".prompt.json")

if (!fs.existsSync(PROMPT_FILE)) {
    fs.writeFileSync(PROMPT_FILE, JSON.stringify({ prompt: "", component: "", storyId: "", timestamp: "" }, null, 2))
}

const initialContent = fs.readFileSync(PROMPT_FILE, "utf8")
const initialTimestamp = JSON.parse(initialContent).timestamp || ""

console.log("Watching for prompts from Storybook...")
console.log(`Prompt file: ${PROMPT_FILE}`)

const check = () => {
    try {
        const content = fs.readFileSync(PROMPT_FILE, "utf8")
        const data = JSON.parse(content)
        if (data.timestamp && data.timestamp !== initialTimestamp && data.prompt) {
            console.log("\n===PROMPT_RECEIVED===")
            console.log(JSON.stringify(data, null, 2))
            console.log("===END_PROMPT===")
            process.exit(0)
        }
    } catch {}
}

const interval = setInterval(check, 500)
process.on("SIGINT", () => { clearInterval(interval); process.exit(0) })
process.on("SIGTERM", () => { clearInterval(interval); process.exit(0) })
```

### Register the addon in main.js

Add `path.resolve(__dirname, "addon-prompt")` to the `addons` array in `.storybook/main.js` (see the main.js template above which already includes it).

### Add `.prompt.json` to `.gitignore`

Append to `component-playground/.gitignore`:
```
.prompt.json
.prompt-status.json
```

---

## Prompt Watcher Workflow

After starting Storybook, start the prompt watcher as a **background task**:

```bash
node component-playground/scripts/watch-prompts.js
```

When the watcher exits (a prompt was received from the Storybook panel):

1. **Update status to "working":**
   ```bash
   node component-playground/scripts/update-prompt-status.js working "Reading component files..."
   ```
2. **Read the prompt file:** `component-playground/.prompt.json`
3. **Parse the JSON** — it contains `{ prompt, component, storyId, timestamp }`
4. **Read the target component files:** `components/<component>/index.js`, `index.css`, `package.json`
5. **Update status as you work:**
   ```bash
   node component-playground/scripts/update-prompt-status.js working "Applying changes to <component>..."
   ```
6. **Apply the requested changes** to the component based on the prompt
7. **Update status to "completed":**
   ```bash
   node component-playground/scripts/update-prompt-status.js completed "Changes applied — check Storybook"
   ```
8. **Restart the watcher** as a new background task to listen for the next prompt

If you need user input or permission:
```bash
node component-playground/scripts/update-prompt-status.js permission_needed "Need approval to modify package.json dependencies"
```

If something goes wrong:
```bash
node component-playground/scripts/update-prompt-status.js error "Could not find component 'foo'"
```

### Status States

| State | Shown as | Meaning |
|-------|----------|---------|
| `listening` | Green "Listening" badge | Watcher is running, ready for prompts |
| `queued` | Yellow banner | Prompt saved, waiting for watcher to pick up |
| `received` | Purple banner | Watcher picked up the prompt |
| `working` | Purple banner + message | Claude Code is actively making changes |
| `permission_needed` | Yellow banner | Claude Code needs user input |
| `completed` | Green banner | Changes applied successfully |
| `error` | Red banner | Something went wrong |

### Status Update Script

```bash
node component-playground/scripts/update-prompt-status.js <state> <message>
```

The prompt file format:
```json
{
    "prompt": "Make the hero section taller and change the gradient to blue-to-green",
    "component": "win-back",
    "storyId": "win-back--default",
    "timestamp": "2025-01-15T10:30:00.000Z"
}
```

**Important:** After processing a prompt, always update the status to "completed" and restart the watcher script so the next prompt can be captured.

---

## Limio Setup Story (One-time)

If `component-playground/src/stories/LimioSetup.stories.js` does **not** exist, create it. This provides a guided onboarding wizard at **Tools > Limio Setup** in the Storybook sidebar.

The wizard has three steps:
1. **Welcome** — branded header + "Get Started" button
2. **Enter Credentials** — form with tenant, region (EU/US/Dev), client ID, client secret; live URL preview; POSTs to `/api/limio/setup`
3. **Connected** — success confirmation with "Build a Component" and "Browse Components" action buttons

On mount it auto-detects existing config via `GET /api/limio/status` and skips to Step 3 if already configured.

Use the actual file at `component-playground/src/stories/LimioSetup.stories.js` as the source of truth.

---

## New Component Builder (One-time)

If `component-playground/src/stories/NewComponent.stories.js` does **not** exist, create it. This provides a dedicated page at **Tools > New Component** in the Storybook sidebar where users can name a component, describe what they want, and submit it to Claude Code for creation.

The `+ New` button in the Claude Prompt panel header also navigates to this page.

### component-playground/src/stories/NewComponent.stories.js

```javascript
import React, { useState, useEffect, useCallback, useRef } from "react"

const toKebabCase = (str) =>
    str
        .replace(/([a-z])([A-Z])/g, "$1-$2")
        .replace(/[\s_]+/g, "-")
        .replace(/[^a-z0-9-]/gi, "")
        .toLowerCase()

const BuilderPage = () => {
    const [componentName, setComponentName] = useState("")
    const [prompt, setPrompt] = useState("")
    const [submitting, setSubmitting] = useState(false)
    const [submitted, setSubmitted] = useState(false)
    const [error, setError] = useState(null)
    const [prefilled, setPrefilled] = useState(false)
    const textareaRef = useRef(null)

    // Prefill from .prompt.json on mount
    useEffect(() => {
        const prefill = async () => {
            try {
                const res = await fetch("/api/prompt")
                if (!res.ok) return
                const data = await res.json()
                if (data.mode === "create" && data.prompt) {
                    setPrompt(data.prompt)
                    if (data.component && data.component !== "unknown") {
                        setComponentName(data.component)
                    }
                    setPrefilled(true)
                }
            } catch {}
        }
        prefill()
    }, [])

    const kebab = toKebabCase(componentName)

    const handleSubmit = useCallback(async () => {
        if (!componentName.trim() || !prompt.trim() || submitting) return
        setSubmitting(true)
        setError(null)
        try {
            const res = await fetch("/api/prompt", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({
                    prompt: prompt.trim(),
                    component: kebab || componentName.trim(),
                    storyId: "tools-new-component--builder",
                    mode: "create",
                }),
            })
            if (res.ok) {
                setSubmitted(true)
            } else {
                const data = await res.json().catch(() => ({}))
                setError(data.error || "Failed to send prompt")
                setSubmitting(false)
            }
        } catch (err) {
            setError("Connection error — is Storybook middleware running?")
            setSubmitting(false)
        }
    }, [componentName, prompt, kebab, submitting])

    const handleKeyDown = useCallback(
        (e) => {
            if ((e.metaKey || e.ctrlKey) && e.key === "Enter") {
                e.preventDefault()
                handleSubmit()
            }
        },
        [handleSubmit]
    )

    const handleReset = () => {
        setComponentName("")
        setPrompt("")
        setSubmitting(false)
        setSubmitted(false)
        setError(null)
        setPrefilled(false)
        // Reset status to listening
        fetch("/api/prompt-status", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ state: "listening", message: "" }),
        }).catch(() => {})
    }

    const isDisabled = submitting || submitted

    return (
        <div style={s.page}>
            <div style={s.card}>
                <div style={s.header}>
                    <div style={s.logo}>C</div>
                    <div>
                        <h1 style={s.title}>New Component</h1>
                        <p style={s.subtitle}>
                            Describe what you want and Claude will build it.
                        </p>
                    </div>
                </div>

                {prefilled && !submitted && (
                    <div style={s.prefillBanner}>
                        <span style={{ fontSize: "13px" }}>&#9889;</span>
                        <span>Pre-filled from your last prompt</span>
                    </div>
                )}

                {submitted && (
                    <div style={s.successBanner}>
                        <span style={{ fontSize: "14px" }}>&#10003;</span>
                        <span>
                            Prompt sent — Claude Code is building{" "}
                            <strong>{kebab || componentName}</strong>. Watch the
                            overlay for progress.
                        </span>
                    </div>
                )}

                {error && (
                    <div style={s.errorBanner}>
                        <span style={{ fontSize: "14px" }}>&#10007;</span>
                        <span>{error}</span>
                    </div>
                )}

                <div style={s.field}>
                    <label style={s.label}>Component Name</label>
                    <input
                        style={{
                            ...s.input,
                            ...(isDisabled ? { opacity: 0.5 } : {}),
                        }}
                        type="text"
                        value={componentName}
                        onChange={(e) => setComponentName(e.target.value)}
                        placeholder='e.g. "Pricing Table" or "Hero Banner"'
                        disabled={isDisabled}
                    />
                    {componentName && (
                        <div style={s.kebabPreview}>
                            <span style={s.kebabLabel}>folder:</span>
                            <code style={s.kebabValue}>
                                components/{kebab}/
                            </code>
                        </div>
                    )}
                </div>

                <div style={s.field}>
                    <label style={s.label}>Prompt</label>
                    <textarea
                        ref={textareaRef}
                        style={{
                            ...s.textarea,
                            ...(isDisabled ? { opacity: 0.5 } : {}),
                        }}
                        value={prompt}
                        onChange={(e) => setPrompt(e.target.value)}
                        onKeyDown={handleKeyDown}
                        placeholder="Describe the component you want to build. Include details about layout, features, styling, and any specific Limio SDK hooks to use..."
                        disabled={isDisabled}
                        rows={8}
                    />
                </div>

                <div style={s.footer}>
                    {!submitted ? (
                        <button
                            style={{
                                ...s.button,
                                ...(!componentName.trim() ||
                                !prompt.trim() ||
                                submitting
                                    ? s.buttonDisabled
                                    : {}),
                            }}
                            onClick={handleSubmit}
                            disabled={
                                !componentName.trim() ||
                                !prompt.trim() ||
                                submitting
                            }
                        >
                            {submitting
                                ? "Sending..."
                                : "Build Component"}
                        </button>
                    ) : (
                        <button style={s.resetButton} onClick={handleReset}>
                            Build Another
                        </button>
                    )}
                    <span style={s.hint}>
                        <strong>Cmd+Enter</strong> to submit
                    </span>
                </div>
            </div>
        </div>
    )
}

const s = {
    page: {
        minHeight: "100vh",
        background: "#f8f9fb",
        display: "flex",
        alignItems: "flex-start",
        justifyContent: "center",
        padding: "60px 20px",
        fontFamily:
            '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif',
        WebkitFontSmoothing: "antialiased",
    },
    card: {
        background: "#fff",
        borderRadius: "16px",
        padding: "40px",
        width: "100%",
        maxWidth: "580px",
        boxShadow:
            "0 1px 3px rgba(0,0,0,0.04), 0 4px 16px rgba(0,0,0,0.06)",
        display: "flex",
        flexDirection: "column",
        gap: "24px",
    },
    header: {
        display: "flex",
        alignItems: "center",
        gap: "14px",
    },
    logo: {
        width: "40px",
        height: "40px",
        borderRadius: "12px",
        background: "linear-gradient(135deg, #d4a574 0%, #c4956a 100%)",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        color: "#fff",
        fontSize: "18px",
        fontWeight: "700",
        flexShrink: 0,
    },
    title: {
        fontSize: "20px",
        fontWeight: "700",
        color: "#1a1f36",
        margin: 0,
        lineHeight: 1.3,
    },
    subtitle: {
        fontSize: "13px",
        color: "#697386",
        margin: "2px 0 0",
        lineHeight: 1.4,
    },
    prefillBanner: {
        display: "flex",
        alignItems: "center",
        gap: "8px",
        background: "#F5F3FF",
        borderRadius: "10px",
        padding: "10px 14px",
        border: "1px solid #DDD6FE",
        fontSize: "12px",
        fontWeight: "500",
        color: "#5B21B6",
    },
    successBanner: {
        display: "flex",
        alignItems: "center",
        gap: "8px",
        background: "#F0FDF4",
        borderRadius: "10px",
        padding: "10px 14px",
        border: "1px solid #BBF7D0",
        fontSize: "12px",
        fontWeight: "500",
        color: "#166534",
        lineHeight: 1.5,
    },
    errorBanner: {
        display: "flex",
        alignItems: "center",
        gap: "8px",
        background: "#FEF2F2",
        borderRadius: "10px",
        padding: "10px 14px",
        border: "1px solid #FECACA",
        fontSize: "12px",
        fontWeight: "500",
        color: "#991B1B",
    },
    field: {
        display: "flex",
        flexDirection: "column",
        gap: "6px",
    },
    label: {
        fontSize: "13px",
        fontWeight: "600",
        color: "#1a1f36",
    },
    input: {
        padding: "10px 12px",
        borderRadius: "8px",
        border: "1px solid #e3e8ee",
        fontSize: "14px",
        fontFamily: "inherit",
        color: "#1a1f36",
        outline: "none",
        transition: "border-color 0.15s ease, box-shadow 0.15s ease",
    },
    kebabPreview: {
        display: "flex",
        alignItems: "center",
        gap: "6px",
        fontSize: "12px",
        color: "#697386",
    },
    kebabLabel: {
        fontWeight: "500",
    },
    kebabValue: {
        fontFamily:
            '"SF Mono", SFMono-Regular, Consolas, "Liberation Mono", Menlo, monospace',
        color: "#635BFF",
        fontSize: "12px",
    },
    textarea: {
        padding: "12px",
        borderRadius: "8px",
        border: "1px solid #e3e8ee",
        fontSize: "14px",
        fontFamily: "inherit",
        color: "#1a1f36",
        lineHeight: 1.6,
        resize: "vertical",
        outline: "none",
        transition: "border-color 0.15s ease, box-shadow 0.15s ease",
        minHeight: "160px",
    },
    footer: {
        display: "flex",
        alignItems: "center",
        gap: "14px",
    },
    button: {
        padding: "10px 24px",
        borderRadius: "8px",
        border: "none",
        background: "#635BFF",
        color: "#fff",
        fontSize: "14px",
        fontWeight: "600",
        cursor: "pointer",
        fontFamily: "inherit",
        transition: "opacity 0.15s ease",
    },
    buttonDisabled: {
        opacity: 0.5,
        cursor: "not-allowed",
    },
    resetButton: {
        padding: "10px 24px",
        borderRadius: "8px",
        border: "1px solid #e3e8ee",
        background: "#fff",
        color: "#1a1f36",
        fontSize: "14px",
        fontWeight: "600",
        cursor: "pointer",
        fontFamily: "inherit",
    },
    hint: {
        fontSize: "12px",
        color: "#a3acb9",
    },
}

export default {
    title: "Tools/New Component",
    parameters: {
        layout: "fullscreen",
        previewTabs: { "storybook/docs/panel": { hidden: true } },
    },
}

export const Builder = {
    render: () => <BuilderPage />,
}
```

Key features:
- **Component name input** — auto-converts to kebab-case and shows folder path preview
- **Prompt textarea** — describe the component, Cmd+Enter to submit
- **Prefill support** — auto-fills from `.prompt.json` if a previous create prompt exists
- **Mode: "create"** — the prompt is sent with `mode: "create"` so the watcher can distinguish from edit prompts
- **Status banners** — shows success, error, and prefill states

---

## Creating a Story

After creating a component, **always** create a story file at `component-playground/src/stories/<ComponentName>.stories.js`.

### Story Template

```javascript
import React from "react"
import { LimioProvider, ComponentContext } from "@limio/sdk"
import MyComponent from "../../../components/component-name/index"

export default {
    title: "Component Name",
    component: MyComponent,
    parameters: { layout: "fullscreen" },
    decorators: [
        (Story, context) => (
            <LimioProvider>
                <ComponentContext.Provider value={context.args}>
                    <Story />
                </ComponentContext.Provider>
            </LimioProvider>
        )
    ]
}

// Default — uses limioProps defaults from the component's package.json
export const Default = {
    args: {
        // Copy each limioProps entry: use its "id" as key, "default" as value
        heading: "Choose Your Plan",
        primaryColor__limio_color: "#635BFF",
        showFeatures: true,
    }
}

// Create 2-4 additional variations showcasing different configurations
export const DarkTheme = {
    args: {
        ...Default.args,
        primaryColor__limio_color: "#1a1a2e",
    }
}

export const MinimalContent = {
    args: {
        ...Default.args,
        showFeatures: false,
    }
}
```

### Story Creation Rules

1. **Args come from limioProps** — Map each `limioProps` entry in the component's `package.json` to a story arg using its `id` as the key and `default` as the value
2. **Create meaningful variations** — Each story should demonstrate a different visual state: different themes, with/without optional sections, different content lengths, etc.
3. **Spread defaults for variations** — Use `...Default.args` and override only what changes
4. **3-5 stories per component** — Default + 2-4 variations
5. **Name stories descriptively** — `DarkTheme`, `WithBadges`, `MinimalContent`, `LongContent`, `CustomBranding`, etc.
6. **Import path** — Components are at `../../../components/<name>/index` relative to the stories directory

---

## Start Storybook & Get Feedback

After creating the component and its story:

```bash
cd component-playground && npx storybook dev -p 6006
```

Start this as a **background task** so it keeps running.

Then start the **prompt watcher** as a second background task:

```bash
node component-playground/scripts/watch-prompts.js
```

**After starting Storybook, tell the user:**
- Storybook is running at **http://localhost:6006**
- List each story variation you created and what it demonstrates
- The **Claude Prompt** panel is available in the Storybook addons panel (bottom tabs) — they can type prompts there and Claude Code will automatically pick them up
- If Limio is not yet configured, mention: "Open **Tools > Limio Setup** in the sidebar to connect your Limio account"
- Keep Storybook and the watcher running while iterating on feedback

**When the watcher background task completes** (prompt received):
1. Read `component-playground/.prompt.json` to get the prompt
2. Read the target component's files (`components/<component>/index.js`, `index.css`, `package.json`)
3. Apply the requested changes
4. Restart the watcher as a new background task
5. Tell the user the changes are applied and Storybook should hot-reload

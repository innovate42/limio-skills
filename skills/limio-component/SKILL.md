---
name: limio-component
description: This skill should be used when the user asks to "create a Limio component", "build a subscription component", "make offer cards", mentions "limioProps", "Limio SDK", "@limio/sdk", "useCampaign", "useBasket", "useUser", or discusses building React components for the Limio subscription platform.
version: 4.0.0
---

# Limio Custom Component Creation

Use this skill when creating custom components for the Limio subscription management platform.

**IMPORTANT:** This skill contains all the documentation you need for building components. Do NOT explore the filesystem or search for existing component patterns. Use the templates, SDK reference, and examples provided below to create components directly. The one exception is checking whether Storybook is already set up (see Storybook section).

**Official SDK docs:** https://docs.limio.com/developers/limio-sdk

## Full Workflow

1. **Create the component** in `./components/` (using the reference sections below)
2. **Check for Storybook:** Look for `component-playground/.storybook/main.js`
3. **If no Storybook exists:** Set up the playground (see "Storybook Setup" section)
4. **Create a story** for the component with multiple variations
5. **Install dependencies** if needed: `cd component-playground && npm install`
6. **Start Storybook:** `cd component-playground && npx storybook dev -p 6006`
7. **Show the user** the running Storybook and ask for feedback

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
│   └── preview.js
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
├── src/
│   └── stories/
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
}

export default preview
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

**After starting Storybook, tell the user:**
- Storybook is running at **http://localhost:6006**
- List each story variation you created and what it demonstrates
- Ask if they want any changes to the component or additional variations
- Keep Storybook running while iterating on feedback

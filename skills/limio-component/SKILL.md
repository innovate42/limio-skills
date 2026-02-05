---
name: limio-component
description: This skill should be used when the user asks to "create a Limio component", "build a subscription component", "make offer cards", mentions "limioProps", "Limio SDK", "@limio/sdk", "useCampaign", "useBasket", "useUser", or discusses building React components for the Limio subscription platform.
version: 2.1.0
---

# Limio Custom Component Creation

Use this skill when creating custom components for the Limio subscription management platform.

**IMPORTANT:** This skill contains all the documentation you need. Do NOT explore the filesystem or search for existing component patterns. Use the templates, SDK reference, and examples provided below to create components directly.

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

---

## Limio SDK - User

### useUser
```javascript
import { useUser } from "@limio/sdk"

const { attributes, subscriptions, loginStatus, loaded, token } = useUser()
```

**Returns:**
- `attributes` - User identity: `{ email, auth_time, sub, crm_id, ... }`
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
  status: "active",
  schedule: [{ date, type, amount, currency, description }],
  offers: [...],
  record_type: "subscription",
  id, ref, reference,
  created: "2024-01-15T...",
  mode: "production"
}
```

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

### Utility Functions
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

---

## Limio SDK - Pricing

### useCheckout
```javascript
import { useCheckout } from "@limio/sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: true })
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

### formatCurrencyForCurrentLocale
```javascript
import { formatCurrencyForCurrentLocale } from "@limio/sdk"

formatCurrencyForCurrentLocale(9.99, "USD") // "$9.99"
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

// Conditional class for sticky header
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

## HTML Sanitization

Always sanitize HTML from Limio attributes:

```javascript
import xss from "xss"

const sanitizeString = (str) => xss(str || "")

<div dangerouslySetInnerHTML={{ __html: sanitizeString(offer.data.attributes.display_price__limio) }} />
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

1. **Use public npm libraries** - Don't reinvent the wheel
2. **Prefer SDKs** - Well-maintained libraries over custom code
3. **Null safety** - Always use optional chaining and defaults
   ```javascript
   const { offers } = useCampaign() || {}
   const attributes = offer?.data?.attributes || {}
   ```
4. **List props** - Items are `{id, label}` objects
5. **Picklist options** - Use `options` array with `{id, label, value}`
6. **Page Builder compatibility** - Components with `position: fixed/absolute` must fall back to `position: relative` when `isInPageBuilder` is true, so they stay within their section
7. **Loading states** - Handle `basketLoading` to prevent double submissions
8. **Sanitize HTML** - Always use `xss` library for rich text content
9. **MUI version** - Use 5.16.12 for React 19 compatibility

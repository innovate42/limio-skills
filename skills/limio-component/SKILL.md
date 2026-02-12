---
name: limio-component
description: This skill should be used when the user asks to "create a Limio component", "build a subscription component", "make offer cards", mentions "limioProps", "Limio SDK", "@limio/sdk", "useCampaign", "useBasket", "useUser", or discusses building React components for the Limio subscription platform.
version: 3.0.0
---

# Limio Custom Component Creation

Use this skill when creating custom components for the Limio subscription management platform.

**IMPORTANT:** This skill contains all the documentation you need for building components. Do NOT explore the filesystem or search for existing component patterns. Use the templates, SDK reference, and examples provided below to create components directly. The one exception is checking whether Storybook is already set up (see Storybook section).

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
10. **Always create stories** - Every component should have a Storybook story with variations

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
    attributes: { email: "user@example.com", email_verified: true, sub: "mock-user-001" },
    subscriptions: [{
        name: "Monthly Plan", status: "active", record_type: "subscription",
        id: "sub-001", reference: "REF001", created: "2024-01-15T00:00:00Z",
        offers: [{ name: "Monthly Plan", quantity: 1, price: { summary: { headline: "$9.99/mo" }, currency: "USD", amount: 9.99 }, products: [] }],
        schedule: [{ data: { date: "2024-02-15T00:00:00Z", amount: "9.99", currency: "USD", type: "payment", description: "Monthly Plan" }, status: "pending" }]
    }],
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
            order: { orderDate: new Date().toISOString(), basketItems: mockBasketItems, orderItems: mockBasketItems, customerDetails: { firstName: "Test", lastName: "User", email: "user@example.com" } },
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

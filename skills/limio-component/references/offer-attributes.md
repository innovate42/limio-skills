# Offer Object — Complete Structure

This reference documents the full shape of a Limio offer object, including all known attributes.

---

## Full Offer Shape

```javascript
offer = {
  id: "unique-id",
  name: "Offer Name",
  path: "/offers/offer-name",
  parent_path: "/pages/page-name",
  type: "item",
  data: {
    attributes: { /* see below */ },
    price: [ /* see Price Array */ ],
    products: [ /* see Products Array */ ],
    attachments: [ /* see Attachments */ ]
  }
}
```

---

## All Attributes (`offer.data.attributes`)

### Display Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `display_name__limio` | string | Display name shown to users (e.g., "Premium Plan") |
| `display_price__limio` | string (HTML) | Formatted price display (e.g., `"<span>$9.99</span>/mo"`) |
| `detailed_display_price__limio` | string (HTML) | Extended price description (e.g., "Billed annually at $119.88") |
| `offer_features__limio` | string (HTML) | Feature list (e.g., `"<ul><li>Feature 1</li></ul>"`) |
| `cta_text__limio` | string | Call-to-action button text (e.g., "Subscribe Now") |
| `checkout_description__limio` | string | Description shown during checkout |
| `checkout__limio` | object | Checkout configuration |

### Grouping & Badge Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `group__limio` | string | Group identifier for offer categorization (e.g., "monthly", "annual") |
| `best_value__limio` | boolean | Whether this offer is flagged as best value |
| `badge_text__limio` | string | Badge/label text (e.g., "Most Popular", "Best Value") |

### Commerce Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `payment_types__limio` | string[] | Accepted payment methods (e.g., `["card", "paypal"]`) |
| `allowed_countries__limio` | string[] | Country codes where offer is available (e.g., `["US", "GB"]`) |
| `allow_multibuy__limio` | boolean | Whether multiple quantities are allowed |
| `autoRenew__limio` | boolean | Whether the subscription auto-renews |
| `push_to_checkout__limio` | boolean | Whether to push directly to checkout on add |
| `block_multiple__limio` | boolean | Whether to prevent multiple subscriptions |

### Cross-sell / Upsell / Upgrade Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `upgrade_offers__limio` | `{path, id, label}[]` | Offers the subscriber can upgrade to |
| `upsell_offers__limio` | `{items: {path, id}[], item_type, item_label}` | Upsell offers (object with items array) |
| `cross_sell_addons__limio` | `{items: {path, id}[], item_type, item_label}` | Add-on cross-sells (object with items array) |
| `cross_sell_add_ons__limio` | `string` | Label tag for cross-sell add-on matching |
| `upsell_display_name__limio` | `string (HTML)` | Display name for upsell context |
| `upsell_display_description__limio` | `string (HTML)` | Description for upsell context |
| `upgrade_cta__limio` | `string` | CTA text for upgrade button (e.g., "Upgrade") |
| `downgrade_cta__limio` | `string` | CTA text for downgrade button (e.g., "Downgrade") |

**These are references, not full offer objects.** Resolve them against the offers from `useCampaign()`:

```javascript
const { offers } = useCampaign()
const attributes = offer?.data?.attributes || {}

// Resolve upgrade offers
const upgradeRefs = attributes.upgrade_offers__limio || []
const upgradeOffers = upgradeRefs
  .map(ref => offers.find(o => o.id === ref.id || o.path === ref.path))
  .filter(Boolean)

// Resolve upsell offers (note: object with .items array)
const upsellRefs = attributes.upsell_offers__limio?.items || []
const upsellOffers = upsellRefs
  .map(ref => offers.find(o => o.id === ref.id || o.path === ref.path))
  .filter(Boolean)

// Resolve cross-sell add-ons (note: object with .items array)
const crossSellRefs = attributes.cross_sell_addons__limio?.items || []
const { addOns } = useCampaign()
const crossSellAddOns = crossSellRefs
  .map(ref => addOns.find(a => a.id === ref.id || a.path === ref.path))
  .filter(Boolean)
```

### Term Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `term__limio` | object | Term configuration: `{ renewal_type, renewal_trigger }` |
| `initial_term__limio` | object | Initial term configuration: `{ renewal_type, renewal_trigger }` |

### Special Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `is_gift__limio` | boolean | Whether this is a gift subscription |
| `student__limio` | boolean | Whether this is a student offer |
| `trial__limio` | boolean/object | Trial configuration |
| `price__limio` | array | Price information array (similar to `data.price`) |

---

## Price Array (`offer.data.price`)

```javascript
price: [
  {
    name: "Monthly charge",
    value: 9.99,
    currencyCode: "USD",
    type: "recurring",           // "recurring" | "one-time"
    trigger: "subscription_start",
    repeat_interval: 1,
    repeat_interval_type: "months"  // "months" | "years" | "weeks" | "days"
  }
]
```

An offer can have multiple price entries (e.g., a setup fee + recurring charge).

**Price fields:**
- `name` — Human-readable name for this charge
- `value` — Numeric amount
- `currencyCode` — ISO currency code (e.g., "USD", "GBP", "EUR")
- `type` — `"recurring"` for subscription charges, `"one-time"` for single charges
- `trigger` — When the charge occurs (e.g., `"subscription_start"`)
- `repeat_interval` — Number of intervals between charges
- `repeat_interval_type` — Unit of interval (`"months"`, `"years"`, `"weeks"`, `"days"`)

---

## Products Array (`offer.data.products`)

```javascript
products: [
  {
    path: "/products/product-name",
    name: "Product",
    attributes: {
      display_name__limio: "Product Display Name",
      product_code__limio: "PRODUCT_CODE"
    }
  }
]
```

An offer can include multiple products.

---

## Attachments (`offer.data.attachments`)

```javascript
attachments: [
  {
    type: "image",
    url: "https://cdn.example.com/image.jpg"
  }
]
```

### Finding an Offer's Image

```javascript
const attachments = offer?.data?.attachments || []

const imageAttachment = attachments.find(a =>
    a.type === "image" || (a.url && /\.(jpg|jpeg|png|gif|svg|webp)$/i.test(a.url))
)

// Use in component
{imageAttachment && (
    <img src={imageAttachment.url} alt={displayName} />
)}
```

---

## useOfferInfo Helper

The `useOfferInfo` hook from `@limio/sdk` extracts commonly needed info from an offer object:

```javascript
import { useOfferInfo } from "@limio/sdk"

const info = useOfferInfo(offer)
```

**Returns:**
- `allowMultibuy` — Boolean, whether multibuy is allowed
- `offerDescription` — Checkout description string
- `hasRecurringCharge` — Boolean, whether offer has recurring charges
- `isDelivery` — Boolean, whether offer involves physical delivery
- `productNames` — Array of product name strings
- `isAutoRenew` — Boolean, whether subscription auto-renews
- `offerImage` — Image URL from attachments (or null)
- `usesExternalPrice` — Boolean, whether pricing is external
- `isGift` — Boolean, whether it's a gift offer
- `displayName` — Display name string

---

## Common Access Patterns

### Getting attributes safely
```javascript
const attributes = offer?.data?.attributes || {}
const displayName = attributes.display_name__limio || "Untitled"
const features = attributes.offer_features__limio || ""
const ctaText = attributes.cta_text__limio || "Subscribe"
```

### Getting the first price
```javascript
const price = offer?.data?.price?.[0]
const amount = price?.value || 0
const currency = price?.currencyCode || "USD"
```

### Checking for best value
```javascript
const isBestValue = offer?.data?.attributes?.best_value__limio === true
const badgeText = offer?.data?.attributes?.badge_text__limio
```

### Getting the group
```javascript
const group = offer?.data?.attributes?.group__limio || "default"
```

### Resolving upgrade/upsell offers
```javascript
// upgrade_offers__limio is an array of { path, id, label }
// upsell_offers__limio is an object: { items: [{ path, id }], item_type, item_label }
// These are references — resolve them against campaign offers:
const { offers } = useCampaign()

const upgradeRefs = offer?.data?.attributes?.upgrade_offers__limio || []
const upgradeOffers = upgradeRefs
  .map(ref => offers.find(o => o.id === ref.id || o.path === ref.path))
  .filter(Boolean)

const upsellItems = offer?.data?.attributes?.upsell_offers__limio?.items || []
const upsellOffers = upsellItems
  .map(ref => offers.find(o => o.id === ref.id || o.path === ref.path))
  .filter(Boolean)
```

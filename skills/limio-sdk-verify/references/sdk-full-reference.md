# Limio SDK — Complete Reference

All hooks and utilities available from the Limio SDK, with full signatures, return shapes, and correct import paths.

---

## Import Paths

| Module | Import Path |
|--------|-------------|
| All hooks & utilities | `@limio/sdk` |
| Basket ID helper | `@limio/shop/src/shop/checkout/basket` |
| Checkout SDK | `@limio/internal-checkout-sdk` |

---

## Hooks

### useCampaign

Returns page/campaign data including offers, add-ons, and grouping metadata.

```javascript
import { useCampaign } from "@limio/sdk"

const { offers, campaign, addOns, tag, groupValues } = useCampaign()
```

**Return shape:**

| Field | Type | Description |
|-------|------|-------------|
| `offers` | `Array<Offer>` | Subscription offer objects |
| `campaign` | `Object` | Page metadata: `{ name, path, attributes }` |
| `addOns` | `Array<Offer>` | Optional products/upsells (same shape as offers) |
| `tag` | `string` | Entry tracking tag (e.g., `"/tags/dummytag"`) |
| `groupValues` | `Array<{ label, id }>` | Offer categorization labels |

**campaign.attributes** includes:
- `push_to_checkout__limio` — Whether to auto-redirect to checkout

---

### useBasket

Manages the shopping basket/cart state and operations.

```javascript
import { useBasket } from "@limio/sdk"

const {
  orderItems,
  basketLoading,
  formattedTotal,
  pageOptions,
  expiresAt,
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

**State fields:**

| Field | Type | Description |
|-------|------|-------------|
| `orderItems` | `Array` | Current basket items |
| `basketLoading` | `boolean` | True during async basket operations |
| `formattedTotal` | `string` | Formatted total (e.g., `"$10.00"`) |
| `pageOptions` | `Object` | Page config; includes `pushToCheckout` |
| `expiresAt` | `string` | Basket expiration timestamp |

**Methods:**

| Method | Signature | Description |
|--------|-----------|-------------|
| `initiateCheckout` | `({ order: { orderItems: [{ offer }] } })` | Create new basket with initial offer |
| `addOfferToBasket` | `({ offer, quantity?, type?, parentId? })` | Add offer to existing basket |
| `removeFromBasket` | `({ id })` | Remove item by OrderItem ID |
| `updateItemQuantity` | `(itemId, quantity)` | Update item quantity |
| `swapOffer` | `(itemId, offer)` | Replace item with different offer |
| `clearOrderItems` | `()` | Empty the basket |
| `navigateToCheckout` | `()` | Navigate to checkout page |
| `redeemPromoCode` | `(promoCode)` | Apply discount code |
| `removePromoCode` | `(promoCode)` | Remove discount code |
| `updateBasketDetails` | `(details)` | Update basket metadata |
| `updateCustomField` | `(field, value)` | Update a custom field on the basket |
| `setCheckoutDisabled` | `(boolean)` | Enable/disable checkout button |
| `validateBasket` | `()` | Run basket validation |
| `selectOfferForSubscriptionUpdate` | `(offer)` | Designate offer for subscription change |

**DEPRECATED:** `addToBasket` and `basketItems` — use `initiateCheckout`/`addOfferToBasket` and `orderItems` instead.

---

### getCurrentBasketId

Helper to check whether a basket/checkout session already exists.

```javascript
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"

const checkoutId = getCurrentBasketId()
// Returns string (basket ID) or null/undefined if no basket exists
```

---

### useUser

Returns authenticated user data.

```javascript
import { useUser } from "@limio/sdk"

const { attributes, subscriptions, loginStatus, loaded, token } = useUser()
```

**Return shape:**

| Field | Type | Description |
|-------|------|-------------|
| `attributes` | `Object` | User identity (see below) |
| `subscriptions` | `Array` | User's subscriptions |
| `loginStatus` | `string` | `"logged-in"` or other states |
| `loaded` | `boolean` | Whether user data has loaded |
| `token` | `string` | JWT access token |

**attributes object:**
- `email` — User email
- `email_verified` — Boolean
- `firstName` — First name
- `lastName` — Last name
- `sub` — Subject identifier
- `crm_id` — CRM identifier

---

### useSubscriptions

Returns the user's subscription data.

```javascript
import { useSubscriptions } from "@limio/sdk"

const { subscriptions } = useSubscriptions()
```

**Subscription object shape:**

```javascript
{
  name: "Premium",
  status: "active",              // "active" | "cancelled" | "pending" | etc.
  id: "sub-...",
  reference: "1KPEEEJ8RNF8",    // Customer-facing reference
  created: "2024-01-15T...",
  record_type: "subscription",
  mode: "production",
  offers: [
    {
      data: {
        start: "2024-01-15T...",
        end: null,                // null if still active
        record_subtype: "base",   // "discount" = discount offer; anything else = standard
        offer: {                  // Full offer object
          data: {
            attributes: {
              display_name__limio: "Premium Plan",
              price__limio: [...],
              term__limio: { ... },
              // ... all standard offer attributes
            },
            products: [
              {
                name: "Product Name",
                attributes: {
                  display_name__limio: "Product Display Name",
                  product_code__limio: "PRODUCT_CODE"
                }
              }
            ]
          }
        }
      }
    }
  ],
  schedule: [
    {
      id: "schedule-...",        // Unique ID — use as React key
      data: {
        date: "2024-02-15T...",
        amount: 9.99,
        currency: "USD",
        description: "Monthly charge",
        type: "payment"          // "payment" | "refund" | etc.
      },
      status: "active"           // "active" | "pending" | "pending-external" | "cancelled"
    }
  ]
}
```

**IMPORTANT:** Always access offers via `subscription.offers[]`. Do NOT use `subscription.data.offer` — that is a legacy field. A subscription can have multiple offers (standard + discount). To find the current standard offer, filter where `record_subtype` is NOT `"discount"` and check `start`/`end` dates.

---

### useLimioContext

Returns Limio platform context, most importantly whether the component is rendered inside the Page Builder editor.

```javascript
import { useLimioContext } from "@limio/sdk"

const { isInPageBuilder } = useLimioContext() || {}
```

| Field | Type | Description |
|-------|------|-------------|
| `isInPageBuilder` | `boolean` | True when rendered in Limio Page Builder editor |

When `isInPageBuilder` is true, components with `position: fixed` or `position: absolute` should fall back to `position: relative`.

---

### useCheckout

Returns checkout state and a selector hook for accessing nested checkout data. Import from `@limio/internal-checkout-sdk`.

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })
const order = useCheckoutSelector((state) => state.order)
const { orderItems } = order
const checkoutState = useCheckoutSelector(state => state) || {}
const { order: fullOrder, paidSchedule, schedule, locale, nextActions } = checkoutState
const orderTotals = useCheckoutSelector((state) => state.display.orderTotal)
```

The `nextActions` object provides `crossSells`, `upgrades`, and `downgrades` arrays for subscription change flows.

**orderTotals object:**

| Field | Type | Description |
|-------|------|-------------|
| `orderSubtotal` | `string` | Before discounts/tax |
| `orderTotal` | `string` | Final total |
| `currency` | `string` | ISO currency code (e.g., `"USD"`, `"GBP"`) |
| `taxSummary` | `Array<{ taxCode, taxAmount, taxRate }>` | Tax breakdown |

---

### usePreview

Returns preview/tax calculation state.

```javascript
import { usePreview } from "@limio/sdk"

const { loadingPreview, isTaxPreviewCountry, taxCalculated } = usePreview()
```

| Field | Type | Description |
|-------|------|-------------|
| `loadingPreview` | `boolean` | True while preview is loading |
| `isTaxPreviewCountry` | `boolean` | Whether tax preview applies to current country |
| `taxCalculated` | `boolean` | Whether tax has been calculated |

---

### useSubInfo

Returns computed information about a single subscription.

```javascript
import { useSubInfo } from "@limio/sdk"

const { status, isGift, quantity, hasLapsed, hasPendingChange } = useSubInfo(subscription)
```

| Field | Type | Description |
|-------|------|-------------|
| `status` | `string` | Computed subscription status |
| `isGift` | `boolean` | Whether this is a gift subscription |
| `quantity` | `number` | Subscription quantity |
| `hasLapsed` | `boolean` | Whether subscription has lapsed |
| `hasPendingChange` | `boolean` | Whether there is a pending change |

---

### useSchedule

Returns formatted schedule/payment information for a subscription.

```javascript
import { useSchedule } from "@limio/sdk"

const { nextPaymentAmount, renewalPrice, termStartDate, termEndDate } = useSchedule(subscription)
```

Returns pre-formatted strings (e.g., `"£9.99"`, `"14 Dec 2024"`).

| Field | Type | Description |
|-------|------|-------------|
| `nextPaymentAmount` | `string` | Formatted next payment amount |
| `renewalPrice` | `string` | Formatted renewal price |
| `termStartDate` | `string` | Formatted term start date |
| `termEndDate` | `string` | Formatted term end date |

---

### useUserInvoices

Returns user invoice data with SWR-style revalidation.

```javascript
import { useUserInvoices } from "@limio/sdk"

const { invoices, revalidate, mutate } = useUserInvoices()
```

| Field | Type | Description |
|-------|------|-------------|
| `invoices` | `Array` | User's invoices |
| `revalidate` | `Function` | Re-fetch invoices |
| `mutate` | `Function` | Optimistic update |

---

### useOfferInfo

Returns computed information extracted from an offer object.

```javascript
import { useOfferInfo } from "@limio/sdk"

const info = useOfferInfo(offer)
```

**Return shape:**

| Field | Type | Description |
|-------|------|-------------|
| `allowMultibuy` | `boolean` | Whether multibuy is allowed |
| `offerDescription` | `string` | Checkout description |
| `hasRecurringCharge` | `boolean` | Whether offer has recurring charges |
| `isDelivery` | `boolean` | Whether offer involves physical delivery |
| `productNames` | `Array<string>` | Product name strings |
| `isAutoRenew` | `boolean` | Whether subscription auto-renews |
| `offerImage` | `string \| null` | Image URL from attachments |
| `usesExternalPrice` | `boolean` | Whether pricing is external |
| `isGift` | `boolean` | Whether it's a gift offer |
| `displayName` | `string` | Display name |

---

## Utility Functions

All imported from `@limio/sdk` unless otherwise noted.

### sanitiseHTML

Sanitizes HTML using DOMPurify. Adds security attributes like `rel="noopener noreferrer"` to links. Preferred over the `xss` npm package when available.

```javascript
import { sanitiseHTML } from "@limio/sdk"

<div dangerouslySetInnerHTML={{ __html: sanitiseHTML(htmlContent) }} />
```

**Note:** For Storybook compatibility (where `sanitiseHTML` may not be mocked), fall back to the `xss` npm package as a dependency.

---

### formatCurrency

Formats a numeric value with a currency symbol.

```javascript
import { formatCurrency } from "@limio/sdk"

formatCurrency("20.00", "GBP")   // "£20.00"
formatCurrency("9.99", "USD")    // "$9.99"
formatCurrency("15.00", "EUR")   // "€15.00"
```

---

### formatCurrencyForCurrentLocale

Locale-aware currency formatting using the browser's current locale.

```javascript
import { formatCurrencyForCurrentLocale } from "@limio/sdk"

formatCurrencyForCurrentLocale(20, "GBP")  // e.g., "£20.00" or "20,00 £"
```

---

### formatDate

Formats a date string using predefined format names.

```javascript
import { formatDate } from "@limio/sdk"

formatDate("2024-01-15T00:00:00Z", "DATE_FULL")    // "January 15, 2024"
formatDate("2024-01-15T00:00:00Z", "DATE_EN")      // "15/01/2024"
formatDate("2024-01-15T00:00:00Z", "DATE_SHORT")   // "Jan 15, 2024"
formatDate("2024-01-15T00:00:00Z", "DATE_MED")     // "15 Jan 2024"
```

**Format names:** `"DATE_EN"`, `"DATE_FULL"`, `"DATE_SHORT"`, `"DATE_MED"`

---

### formatDisplayPrice

Formats a price string using template placeholders.

```javascript
import { formatDisplayPrice } from "@limio/sdk"

formatDisplayPrice("{{currencySymbol}}{{amount}}/mo", offer.data.attributes.price__limio)
```

**Available placeholders:**
- `{{currencyCode}}` — ISO code (e.g., `"USD"`)
- `{{currencySymbol}}` — Symbol (e.g., `"$"`)
- `{{currencySymbolNative}}` — Native symbol
- `{{amount}}` — Formatted amount (e.g., `"9.99"`)
- `{{integerValue}}` — Integer part (e.g., `"9"`)
- `{{decimalValue}}` — Decimal part (e.g., `"99"`)
- `{{formattedPrice}}` — Locale-formatted price
- `{{formattedPriceComma}}` — Comma-formatted price

---

### ErrorBoundary

React error boundary component for graceful error handling.

```javascript
import { ErrorBoundary } from "@limio/sdk"

<ErrorBoundary ErrorUI={({ error }) => <p>Error: {error.message}</p>}>
    <RiskyChild />
</ErrorBoundary>
```

Also available as a higher-order component:

```javascript
import { withErrorBoundary } from "@limio/sdk"

const SafeComponent = withErrorBoundary(MyComponent, ErrorFallback)
```

---

### checkActiveOffers

Filters a subscription's offers array to return only currently active offers, sorted by start date.

```javascript
import { checkActiveOffers } from "@limio/sdk"

const activeOffers = checkActiveOffers(subscription.offers, false)
// Second parameter: whether to include discount offers
```

---

### getCurrentOffer

Returns the current active standard offer from a subscription.

```javascript
import { getCurrentOffer } from "@limio/sdk"

const currentOffer = getCurrentOffer(subscription)
```

---

### groupOffers

Groups offers by their `group__limio` attribute, matching against provided group labels.

```javascript
import { groupOffers } from "@limio/sdk"

const grouped = groupOffers(offers, groupLabels)
```

**Parameters:**
- `offers` — Array of offer objects (default: `[]`)
- `groupLabels` — Array of `{ id, label, thumbnail }` objects (default: `[]`)

**Returns:** `Array<{ groupId, id, label, offers, thumbnail }>`

Each group object:
- `groupId` — Group identifier string
- `id` — Same as groupId
- `label` — Display label (from groupLabels match, or groupId as fallback)
- `offers` — Array of offers in this group
- `thumbnail` — Thumbnail URL from matching groupLabel (or empty string)

---

### getCurrentAddress

Returns the current address of a given type from an addresses array.

```javascript
import { getCurrentAddress } from "@limio/sdk"

const billingAddress = getCurrentAddress("billing", addresses)
const shippingAddress = getCurrentAddress("shipping", addresses)
```

---

### getPriceFromSchedule

Extracts price information from a subscription's schedule.

```javascript
import { getPriceFromSchedule } from "@limio/sdk"

const { value, currencyCode } = getPriceFromSchedule(schedule, country)
```

---

### getPeriodForOffer

Returns a human-readable billing period string for an offer.

```javascript
import { getPeriodForOffer } from "@limio/sdk"

const period = getPeriodForOffer(offer)
// Returns: "1 month" | "1 year" | "N/A"
```

---

### addressSummary

Formats an address object into a display string.

```javascript
import { addressSummary } from "@limio/sdk"

addressSummary(address)  // Formatted address string, or "N/A" if empty
```

---

### formatCountry

Converts a country code to a country name.

```javascript
import { formatCountry } from "@limio/sdk"

formatCountry("GB")  // "United Kingdom"
formatCountry("US")  // "United States"
```

---

### getAddressMetadata

Returns metadata about what address fields are required and displayed for a given country.

```javascript
import { getAddressMetadata } from "@limio/sdk"

const { requiredAddressFields, addressFieldsToRender } = getAddressMetadata("GB")
```

---

### getCountryMetadata

Returns ISO metadata for a country code.

```javascript
import { getCountryMetadata } from "@limio/sdk"

const meta = getCountryMetadata("GB")
// { name: "United Kingdom", "alpha-2": "GB", "alpha-3": "GBR", "country-code": "826" }
```

---

### LimioAppSettings

Provides access to application-level settings.

```javascript
import { LimioAppSettings } from "@limio/sdk"

const dateFormat = LimioAppSettings.getDateFormat()
```

---

### DateTime

Re-exported Luxon `DateTime` class for date manipulation.

```javascript
import { DateTime } from "@limio/sdk"

const now = DateTime.now()
const formatted = DateTime.fromISO("2024-01-15").toFormat("dd MMM yyyy")
```

---

### LimioFetchers

Provides authenticated fetch helpers for Limio APIs.

```javascript
import { LimioFetchers } from "@limio/sdk"

const blob = await LimioFetchers.invoiceFetch(path, token)
```

---

## Component Props Utilities

### useComponentProps

Hook that returns the current component props, merging defaults from package.json with any overrides from the Limio Page Builder.

```javascript
import { useComponentProps } from "@limio/sdk"

const props = useComponentProps(defaultProps)
```

---

### getPropsFromPackageJson

Extracts the `limioProps` array from a package.json object and converts it into a default props map. Import from `@limio/components/helpers`.

```javascript
import { getPropsFromPackageJson } from "@limio/components/helpers"
import packageData from "./package.json"

const defaultComponentProps = getPropsFromPackageJson(packageData)
```

**IMPORTANT:** Use a default import for `package.json`, not `import * as`.

---

### Standard componentStaticProps.js Pattern

```javascript
import { useComponentProps } from "@limio/sdk"
import { getPropsFromPackageJson } from "@limio/components/helpers"
import packageData from "./package.json"

const defaultComponentProps = getPropsFromPackageJson(packageData)

export function useStaticProps() {
    return useComponentProps(defaultComponentProps)
}
```

---

## Offer Object Shape

Full structure of an offer object as returned by `useCampaign()`.

```javascript
{
  id: "unique-id",
  name: "Offer Name",
  path: "/offers/offer-name",
  parent_path: "/pages/page-name",
  type: "item",
  data: {
    attributes: {
      // Display
      display_name__limio: "Premium Plan",
      display_price__limio: "<span>$9.99</span>/mo",       // HTML
      detailed_display_price__limio: "Billed annually",     // HTML
      offer_features__limio: "<ul><li>Feature</li></ul>",   // HTML
      cta_text__limio: "Subscribe Now",
      checkout_description__limio: "Premium subscription",

      // Grouping & badges
      group__limio: "monthly",
      best_value__limio: true,
      badge_text__limio: "Most Popular",

      // Commerce
      payment_types__limio: ["card", "paypal"],
      allowed_countries__limio: ["US", "GB"],
      allow_multibuy__limio: false,
      autoRenew__limio: true,
      push_to_checkout__limio: false,
      block_multiple__limio: false,

      // Cross-sell
      cross_sell_addons__limio: [],
      cross_sell_offers__limio: [],
      upsell_offers__limio: [],

      // Term
      term__limio: { renewal_type: "...", renewal_trigger: "..." },
      initial_term__limio: { renewal_type: "...", renewal_trigger: "..." },

      // Special
      is_gift__limio: false,
      student__limio: false,
      trial__limio: false,
      price__limio: [...]
    },
    price: [
      {
        name: "Monthly charge",
        value: 9.99,
        currencyCode: "USD",
        type: "recurring",
        trigger: "subscription_start",
        repeat_interval: 1,
        repeat_interval_type: "months"
      }
    ],
    products: [
      {
        path: "/products/product-name",
        name: "Product",
        attributes: {
          display_name__limio: "Product Display Name",
          product_code__limio: "PRODUCT_CODE"
        }
      }
    ],
    attachments: [
      {
        type: "image",
        url: "https://cdn.example.com/image.jpg"
      }
    ]
  }
}
```

---

## Subscription Object Shape

Full structure of a subscription object as returned by `useSubscriptions()`.

```javascript
{
  name: "Premium",
  status: "active",
  id: "sub-...",
  reference: "1KPEEEJ8RNF8",
  created: "2024-01-15T...",
  record_type: "subscription",
  mode: "production",
  offers: [
    {
      data: {
        start: "2024-01-15T...",
        end: null,
        record_subtype: "base",
        offer: {
          data: {
            attributes: { /* standard offer attributes */ },
            products: [{ name: "...", attributes: { ... } }]
          }
        }
      }
    }
  ],
  schedule: [
    {
      id: "schedule-...",
      data: {
        date: "2024-02-15T...",
        amount: 9.99,
        currency: "USD",
        description: "Monthly charge",
        type: "payment"
      },
      status: "active"
    }
  ]
}
```

**Access patterns:**
- Current standard offer: `getCurrentOffer(subscription)` or filter `subscription.offers` where `record_subtype !== "discount"`
- Active offers: `checkActiveOffers(subscription.offers, false)`
- Schedule payments: `subscription.schedule.filter(s => s.status === "active")`
- Customer reference: `subscription.reference`
- Subscription ID for linking: `subscription.reference` or `subscription.id` (pass as URL query param)

---

## Correct Import Summary

```javascript
// Hooks
import {
  useCampaign,
  useBasket,
  useUser,
  useSubscriptions,
  useLimioContext,
  useCheckout,
  usePreview,
  useSubInfo,
  useSchedule,
  useUserInvoices,
  useOfferInfo,
} from "@limio/sdk"

// Component props
import {
  useComponentProps,
  getPropsFromPackageJson,
} from "@limio/sdk"

// Utility functions
import {
  sanitiseHTML,
  formatCurrency,
  formatDate,
  formatDisplayPrice,
  formatCurrencyForCurrentLocale,
  ErrorBoundary,
  withErrorBoundary,
  checkActiveOffers,
  getCurrentOffer,
  groupOffers,
  getCurrentAddress,
  getPriceFromSchedule,
  getPeriodForOffer,
  addressSummary,
  formatCountry,
  getAddressMetadata,
  getCountryMetadata,
  LimioAppSettings,
  DateTime,
  LimioFetchers,
} from "@limio/sdk"

// Basket helper (separate import path)
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"
```

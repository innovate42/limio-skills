# Limio SDK Hooks — Full Reference

All hooks are imported from `@limio/sdk` unless otherwise noted.

---

## useCampaign

Returns page/campaign data including offers.

```javascript
import { useCampaign } from "@limio/sdk"

const { offers, campaign, addOns, tag, groupValues } = useCampaign()
```

**Full return shape:**
- `campaign` — Page metadata object:
  - `name` — Campaign/page name
  - `path` — Campaign path (e.g., "/pages/pricing")
  - `attributes` — Page-level attributes including `push_to_checkout__limio`
- `offers` — Array of offer objects (see `references/offer-attributes.md` for full shape)
- `addOns` — Array of optional products/upsells (same shape as offers)
- `tag` — Entry tracking tag string (e.g., "/tags/dummytag")
- `groupValues` — Array of `{ label, id }` objects for offer categorization

---

## groupOffers

Utility function for grouping offers by their `group__limio` attribute.

```javascript
import { groupOffers } from "@limio/sdk"

const grouped = groupOffers(offers, groupLabels)
```

**Parameters:**
- `offers` — Array of offer objects (default: `[]`)
- `groupLabels` — Array of `{ id, label, thumbnail }` objects (default: `[]`)

**Returns:** `Array<{ groupId, id, label, offers, thumbnail }>`

Each group object contains:
- `groupId` — The group identifier string
- `id` — Same as groupId
- `label` — Display label (from groupLabels match, or groupId as fallback)
- `offers` — Array of offers in this group
- `thumbnail` — Thumbnail URL from matching groupLabel (or empty string)

---

## useBasket

Manages the shopping basket/cart.

```javascript
import { useBasket } from "@limio/sdk"
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"

const {
  orderItems,           // Current basket items array
  basketLoading,        // Boolean — true during async operations
  formattedTotal,       // Formatted total string, e.g., "$10.00"
  pageOptions,          // Page config settings (includes pushToCheckout)
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

### Basket Methods

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
| `selectOfferForSubscriptionUpdate` | `(offer)` | Designate offer for subscription change |

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

---

## useUser

Returns authenticated user data.

```javascript
import { useUser } from "@limio/sdk"

const { attributes, subscriptions, loginStatus, loaded, token } = useUser()
```

**Return shape:**
- `attributes` — User identity object:
  - `email` — User email
  - `email_verified` — Boolean
  - `firstName` — First name
  - `lastName` — Last name
  - `sub` — Subject identifier
  - `crm_id` — CRM identifier
- `subscriptions` — Array of user's subscriptions (same as `useSubscriptions().subscriptions`)
- `loginStatus` — `"logged-in"` or other states
- `loaded` — Boolean indicating data availability
- `token` — JWT access token

---

## useSubscriptions

Returns user's subscription data.

```javascript
import { useSubscriptions } from "@limio/sdk"

const { subscriptions } = useSubscriptions()
```

**Subscription object shape:**

```javascript
{
  name: "Premium",
  status: "active",              // "active" | "cancelled" | etc.
  id: "sub-...",
  reference: "1KPEEEJ8RNF8",    // Customer-facing reference
  created: "2024-01-15T...",
  record_type: "subscription",
  mode: "production",
  offers: [                       // Array of offers — the documented access pattern
    {
      data: {
        start: "2024-01-15T...",
        end: null,                // null if still active
        record_subtype: "base",   // "discount" = discount offer; anything else = standard offer
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

**IMPORTANT:** Always access offers via `subscription.offers[]` — this is the documented pattern. A subscription can have multiple offers (e.g. a standard offer + a discount offer). Do NOT use `subscription.data.offer` as that is a legacy field. To get the current standard offer, filter `subscription.offers` where `record_subtype` is NOT `"discount"` and check `start`/`end` dates.

---

## useSubInfo

Returns computed info about a single subscription.

```javascript
import { useSubInfo } from "@limio/sdk"

const { status, isGift, quantity, hasLapsed, hasPendingChange } = useSubInfo(subscription)
```

---

## useSchedule

Returns formatted schedule/payment info for a subscription.

```javascript
import { useSchedule } from "@limio/sdk"

const { nextPaymentAmount, renewalPrice, termStartDate, termEndDate } = useSchedule(subscription)
// Returns formatted values: "£9.99", "14 Dec 2024"
```

---

## useUserInvoices

Returns user invoice data.

```javascript
import { useUserInvoices } from "@limio/sdk"

const { invoices, revalidate, mutate } = useUserInvoices()
```

---

## useCheckout

Returns checkout state and pricing. Import from `@limio/internal-checkout-sdk` (not `@limio/sdk`).

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })
const order = useCheckoutSelector((state) => state.order)
const { orderItems } = order
const checkoutState = useCheckoutSelector(state => state) || {}
const { order: fullOrder, paidSchedule, schedule, locale, nextActions } = checkoutState
const orderTotals = useCheckoutSelector((state) => state.display.orderTotal)
```

The `nextActions` object contains `crossSells`, `upgrades`, and `downgrades` arrays for subscription change flows.

**orderTotals object:**
- `orderSubtotal` — Before discounts/tax
- `orderTotal` — Final total
- `currency` — "USD", "GBP", etc.
- `taxSummary` — Array of `{ taxCode, taxAmount, taxRate }`

---

## usePreview

Returns preview/tax calculation state.

```javascript
import { usePreview } from "@limio/sdk"

const { loadingPreview, isTaxPreviewCountry, taxCalculated } = usePreview()
```

---

## useLimioContext

Returns Limio platform context.

```javascript
import { useLimioContext } from "@limio/sdk"

const { isInPageBuilder } = useLimioContext() || {}
```

When `isInPageBuilder` is true, the component is rendered in the Limio Page Builder editor. Components using `position: fixed` or `position: absolute` should fall back to `position: relative` to stay within their section bounds.

---

## SDK Utilities

All imported from `@limio/sdk`:

### sanitiseHTML
```javascript
import { sanitiseHTML } from "@limio/sdk"

<div dangerouslySetInnerHTML={{ __html: sanitiseHTML(offer.data.attributes.offer_features__limio) }} />
```
Uses DOMPurify and adds security attributes like `rel="noopener noreferrer"` to links. Prefer this over the `xss` npm package. For Storybook compatibility (where `sanitiseHTML` may not be mocked), you can fall back to `xss` as a dependency.

### ErrorBoundary
```javascript
import { ErrorBoundary } from "@limio/sdk"

<ErrorBoundary ErrorUI={({ error }) => <p>Error: {error.message}</p>}>
    <RiskyChild />
</ErrorBoundary>
```
Also available as HOC: `withErrorBoundary(Component, ErrorUI)`

### formatDate
```javascript
import { formatDate } from "@limio/sdk"

formatDate("2024-01-15T00:00:00Z", "DATE_FULL")   // "January 15, 2024"
```
Formats: `"DATE_EN"`, `"DATE_FULL"`, `"DATE_SHORT"`, `"DATE_MED"`

### formatCurrency
```javascript
import { formatCurrency } from "@limio/sdk"

formatCurrency("20.00", "GBP")  // "£20.00"
```

### formatCurrencyForCurrentLocale
```javascript
import { formatCurrencyForCurrentLocale } from "@limio/sdk"

formatCurrencyForCurrentLocale(20, "GBP")  // Locale-aware formatting
```

### formatDisplayPrice
```javascript
import { formatDisplayPrice } from "@limio/sdk"

formatDisplayPrice("{{currencySymbol}}{{amount}}/mo", offer.data.attributes.price__limio)
```
Placeholders: `{{currencyCode}}`, `{{currencySymbol}}`, `{{currencySymbolNative}}`, `{{amount}}`, `{{integerValue}}`, `{{decimalValue}}`, `{{formattedPrice}}`, `{{formattedPriceComma}}`

### useOfferInfo
```javascript
import { useOfferInfo } from "@limio/sdk"

const info = useOfferInfo(offer)
// Returns: { allowMultibuy, offerDescription, hasRecurringCharge, isDelivery, productNames, isAutoRenew, offerImage, usesExternalPrice, isGift, displayName }
```

### checkActiveOffers
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

### LimioAppSettings
```javascript
import { LimioAppSettings } from "@limio/sdk"

const dateFormat = LimioAppSettings.getDateFormat()
```

### DateTime (Luxon)
```javascript
import { DateTime } from "@limio/sdk"
```

### LimioFetchers
```javascript
import { LimioFetchers } from "@limio/sdk"

const blob = await LimioFetchers.invoiceFetch(path, token)
```

---

## Subscription Utility Functions

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

**IMPORTANT:** `subscription.offers[]` is the documented pattern for accessing offers on a subscription. Do NOT use `subscription.data.offer` — that is a legacy field.

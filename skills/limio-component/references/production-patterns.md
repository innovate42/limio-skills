# Production Component Patterns

Real-world patterns extracted from production Limio components. Use these as canonical references when building components.

---

## Import Map — Mandatory Reference

Every hook and utility MUST be imported from the correct package. Using the wrong package causes runtime errors.

| Function | Package | Notes |
|----------|---------|-------|
| `useCampaign` | `@limio/sdk` | Returns `{ offers, campaign, addOns, tag, groupValues }` |
| `useBasket` | `@limio/sdk` | Returns basket state + all mutation methods |
| `useUser` | `@limio/sdk` | Returns `{ attributes, subscriptions, loginStatus, loaded, token }` |
| `useSubscriptions` | `@limio/sdk` | Returns `{ subscriptions }` |
| `useLimioContext` | `@limio/sdk` | Returns `{ isInPageBuilder }` |
| `usePreview` | `@limio/sdk` | Tax preview state |
| `useUserInvoices` | `@limio/sdk` | Returns `{ invoices, revalidate, mutate, nextPage }` |
| `useUserAccountInformation` | `@limio/sdk` | Returns `{ accountInformation, revalidate, mutate }` |
| `getCurrentOffer` | `@limio/sdk` | **IMPORT this — never reimplement** |
| `checkActiveOffers` | `@limio/sdk` | Filter active offers by date |
| `groupOffers` | `@limio/sdk` | Group offers by `group__limio` |
| `useOfferInfo` | `@limio/sdk` | Computed offer metadata |
| `useSubInfo` | `@limio/sdk` | Computed subscription metadata |
| `useSchedule` | `@limio/sdk` | Formatted schedule data |
| `formatCurrency` | `@limio/sdk` | `formatCurrency("20.00", "GBP")` → `"£20.00"` |
| `formatDate` | `@limio/sdk` | `formatDate(isoString, "DATE_MED")` |
| `sanitiseHTML` | `@limio/sdk` | DOMPurify-based sanitizer |
| `getCurrentAddress` | `@limio/sdk` | `getCurrentAddress("billing", addresses)` |
| `addressSummary` | `@limio/sdk` | Formatted address string |
| `getNextSchedule` | `@limio/sdk` | Next upcoming payment from schedule |
| `getRenewalDateForUserSubscription` | `@limio/sdk` | Formatted renewal date |
| `getPriceForUserSubscription` | `@limio/sdk` | Formatted subscription price |
| `getSubscriptionCurrency` | `@limio/sdk` | Currency code for subscription |
| `useCheckout` | `@limio/internal-checkout-sdk` | Returns `{ useCheckoutSelector }` — **NOT from @limio/sdk** |
| `useLimioUserSubscription` | `@limio/internal-checkout-sdk` | Returns `{ userSubscription }` by ID |
| `useLimioUserSubscriptions` | `@limio/internal-checkout-sdk` | Returns `{ userSubscriptions }` |
| `useLimioUserSubscriptionPaymentMethods` | `@limio/internal-checkout-sdk` | Returns `{ payment_methods, revalidate }` |
| `useLimioUserSubscriptionAddresses` | `@limio/internal-checkout-sdk` | Returns `{ addresses, revalidate }` |
| `useComponentProps` | `@limio/sdk` | Merge default props with campaign overrides |
| `getPropsFromPackageJson` | `@limio/sdk` | Extract defaults from package.json limioProps |
| `getCurrentBasketId` | `@limio/shop/src/shop/checkout/basket` | Current basket session ID |

**Sub-path imports also work:** `@limio/sdk/offers`, `@limio/sdk/address`, `@limio/sdk/subscription`, `@limio/sdk/date`, `@limio/sdk/price`, `@limio/sdk/checkout`, `@limio/sdk/user`, `@limio/sdk/page`, `@limio/sdk/basket`, `@limio/sdk/zuora`.

---

## Common Hallucinations — NEVER Use These

These functions DO NOT EXIST. The AI frequently invents them.

| Hallucinated | Correct Alternative |
|-------------|-------------------|
| `useUserSubscriptions()` | `useSubscriptions()` from `@limio/sdk` |
| `useCheckoutSelector` as standalone import | `const { useCheckoutSelector } = useCheckout()` — it's a method returned by `useCheckout()` |
| `useCheckout()` from `@limio/sdk` | `useCheckout()` from `@limio/internal-checkout-sdk` |
| `selectOfferForSubscriptionUpdate` from `useCheckout` | `selectOfferForSubscriptionUpdate` from `useBasket()` |
| `navigateToCheckout` from `useCheckout` | `navigateToCheckout` from `useBasket()` |
| `initiateCheckout` from `useCheckout` | `initiateCheckout` from `useBasket()` |
| `useUserSubscriptionPaymentMethods` | `useLimioUserSubscriptionPaymentMethods` (note the `Limio` prefix) |
| `useUserSubscriptionAddresses` | `useLimioUserSubscriptionAddresses` (note the `Limio` prefix) |
| `useLimioUserSubscriptionSchedules` | Does not exist — use `subscription.schedule[]` directly |
| `useLimioUserSubscriptionNextInvoice` | Does not exist — use `useUserInvoices()` |
| `useLimioUserSubscriptionInvoices` | Does not exist — use `useUserInvoices()` |
| `useLimioUserSubscriptionUsage` | Does not exist |
| Custom `getCurrentOffer` function | Import `getCurrentOffer` from `@limio/sdk` — it handles all edge cases |
| `offer.attributes.x` | `offer.data.attributes.x` — always `.data.attributes` |
| `subscription.data.attributes.status__limio` | `subscription.status` — status is top-level |
| `subscription.data.attributes.display_name__limio` | Use `getCurrentOffer(subscription)?.data?.attributes?.display_name__limio` |

---

## Pattern 1: Subscription Upgrade/Downgrade Flow

From production `update-subscription-offers` component. This is the canonical pattern.

```javascript
import { useCheckout, useLimioUserSubscription } from "@limio/internal-checkout-sdk"
import { useBasket, getCurrentOffer } from "@limio/sdk"

function UpgradeOffers() {
  // Step 1: Get checkout state (useCheckout is from @limio/internal-checkout-sdk)
  const { useCheckoutSelector } = useCheckout()

  // Step 2: Get basket actions (useBasket is from @limio/sdk)
  const { basketLoading, selectOfferForSubscriptionUpdate, navigateToCheckout } = useBasket()

  // Step 3: Read upgrade/downgrade offers from checkout nextActions
  const { upgrades = [], downgrades = [] } = useCheckoutSelector(
    (state) => state.nextActions
  )

  // Step 4: Get current subscription for comparison
  const subscriptionId = useCheckoutSelector(
    (state) => state.order.forSubscription?.id
  )
  const { userSubscription } = useLimioUserSubscription(subscriptionId)
  const currentOffer = getCurrentOffer(userSubscription) // IMPORT this, never reimplement

  // Step 5: Handle offer selection
  const selectOffer = async (offer) => {
    await selectOfferForSubscriptionUpdate({
      orderItemActionType: "add",
      offer: offer,
      type: offer.record_type || "offer",
      quantity: 1
    })

    await navigateToCheckout({
      journey: {
        checkout: offer.data?.attributes?.update_configuration__limio || "/update"
      }
    })
  }

  return (
    <div>
      {/* Current plan */}
      {currentOffer && (
        <div>
          <h3>{currentOffer.data.attributes.display_name__limio}</h3>
          <div dangerouslySetInnerHTML={{
            __html: sanitiseHTML(currentOffer.data.attributes.display_price__limio || "")
          }} />
        </div>
      )}

      {/* Downgrade options */}
      {downgrades.map((offer) => (
        <div key={offer.id}>
          <h3>{offer.data.attributes.display_name__limio}</h3>
          <button disabled={basketLoading} onClick={() => selectOffer(offer)}>
            {offer.data.attributes.downgrade_cta__limio || "Downgrade"}
          </button>
        </div>
      ))}

      {/* Upgrade options */}
      {upgrades.map((offer) => (
        <div key={offer.id}>
          <h3>{offer.data.attributes.display_name__limio}</h3>
          <button disabled={basketLoading} onClick={() => selectOffer(offer)}>
            {offer.data.attributes.upgrade_cta__limio || "Upgrade"}
          </button>
        </div>
      ))}
    </div>
  )
}
```

**Key rules:**
- `useCheckout` → from `@limio/internal-checkout-sdk` — gives `useCheckoutSelector`
- `useBasket` → from `@limio/sdk` — gives `selectOfferForSubscriptionUpdate`, `navigateToCheckout`
- `nextActions.upgrades` and `nextActions.downgrades` are **already-resolved full offer objects** from the backend
- CTA text comes from `offer.data.attributes.upgrade_cta__limio` / `downgrade_cta__limio`
- `getCurrentOffer` is **imported** from `@limio/sdk`, never reimplemented

---

## Pattern 2: Cross-Sell Display

From production `cross-sell` component. Cross-sells live on `orderItem.crossSell[]` in the checkout state.

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"
import { useBasket, formatCurrency } from "@limio/sdk"

function CrossSell() {
  const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })
  const order = useCheckoutSelector((state) => state.order)
  const { orderItems } = order
  const { basketLoading, addOfferToBasket } = useBasket()

  // Deduplicate cross-sell offers: exclude already-in-basket and duplicates
  const crossSellOffers = orderItems.reduce((uniqueOffers, orderItem) => {
    const crossSellArray = orderItem.crossSell || []
    crossSellArray.forEach((offer) => {
      const alreadyInBasket = orderItems.some((item) => item.offer?.id === offer.id)
      const alreadyAdded = uniqueOffers.some((u) => u.id === offer.id)
      if (!alreadyInBasket && !alreadyAdded) uniqueOffers.push(offer)
    })
    return uniqueOffers
  }, [])

  if (!crossSellOffers.length) return null

  const addCrossSell = async (offer) => {
    const isAddOn = offer.record_type === "add_on"
    const quantity = offer.data?.attributes?.default_quantity_options__limio?.minimum_quantity ?? 1

    let parentId
    if (isAddOn) {
      const parentItem = orderItems
        .filter((item) => item.crossSell)
        .find((item) => item.crossSell.some((sub) => sub.id === offer.id))
      parentId = parentItem?.id
    }

    await addOfferToBasket({ offer, quantity: Number(quantity), parentId })
  }

  return (
    <div>
      {crossSellOffers.map((offer) => {
        const price = offer.data.attributes.price__limio?.[0]
        return (
          <div key={offer.id}>
            <span>{offer.data.attributes.cross_sell_display_name__limio}</span>
            {price && <span>{formatCurrency(price.value, price.currencyCode)}</span>}
            <p>{offer.data.attributes.cross_sell_display_description__limio}</p>
            <button disabled={basketLoading} onClick={() => addCrossSell(offer)}>Add</button>
          </div>
        )
      })}
    </div>
  )
}
```

**Key rules:**
- Cross-sells come from `orderItem.crossSell[]` on each item in the checkout state
- Add-ons need `parentId` linking to their parent orderItem
- Use `cross_sell_display_name__limio` and `cross_sell_display_description__limio` (not regular display_name)
- Always deduplicate against items already in basket

---

## Pattern 3: Cart / Basket Display

From production `basket` component. Shows order items with quantities, prices, and totals.

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"
import { useBasket, sanitiseHTML, formatCurrency } from "@limio/sdk"

function Cart() {
  const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })

  // Order items from checkout state
  const order = useCheckoutSelector((state) => state.order)
  const { orderItems = [] } = order

  // Order totals from checkout display
  const orderTotals = useCheckoutSelector((state) => state.display?.orderTotal) || {}
  // orderTotals shape: { orderSubtotal, orderTotal, currency, taxSummary: [{ taxCode, taxAmount, taxRate }] }

  // Basket actions
  const { basketLoading, removeFromBasket, updateItemQuantity, navigateToCheckout } = useBasket()

  const handleRemove = async (itemId) => {
    await removeFromBasket({ id: itemId })
  }

  const handleQuantityChange = async (itemId, newQuantity) => {
    await updateItemQuantity(itemId, newQuantity)
  }

  if (!orderItems.length) {
    return <p>Your cart is empty.</p>
  }

  return (
    <div>
      {orderItems.map((item) => {
        const offer = item.offer
        const attrs = offer?.data?.attributes || {}
        const productName = offer?.data?.products?.[0]?.attributes?.display_name__limio
        const offerImage = offer?.data?.attachments?.find((a) => a.type?.includes("image"))
        const allowMultibuy = attrs.allow_multibuy__limio

        return (
          <div key={item.id}>
            {offerImage && <img src={offerImage.url} alt="" />}
            <div>
              <h3>{productName || attrs.display_name__limio}</h3>
              {attrs.display_price__limio && (
                <div dangerouslySetInnerHTML={{ __html: sanitiseHTML(attrs.display_price__limio) }} />
              )}
            </div>
            <input
              type="number"
              value={item.quantity || 1}
              disabled={!allowMultibuy || basketLoading}
              onChange={(e) => handleQuantityChange(item.id, Number(e.target.value))}
            />
            <button disabled={basketLoading} onClick={() => handleRemove(item.id)}>Remove</button>
          </div>
        )
      })}

      {/* Order totals */}
      <div>
        {orderTotals.orderSubtotal && <p>Subtotal: {orderTotals.orderSubtotal}</p>}
        {orderTotals.taxSummary?.map((tax, i) => (
          <p key={i}>{tax.taxCode}: {formatCurrency(tax.taxAmount, orderTotals.currency)}</p>
        ))}
        {orderTotals.orderTotal && <p>Total: {orderTotals.orderTotal}</p>}
      </div>

      <button disabled={basketLoading} onClick={() => navigateToCheckout()}>
        Checkout
      </button>
    </div>
  )
}
```

**Key rules:**
- Order items: `useCheckoutSelector((state) => state.order.orderItems)`
- Order totals: `useCheckoutSelector((state) => state.display.orderTotal)`
- `orderTotal` shape: `{ orderSubtotal, orderTotal, currency, taxSummary }`
- Product name: `item.offer?.data?.products?.[0]?.attributes?.display_name__limio`
- Image: `item.offer?.data?.attachments?.find(a => a.type?.includes("image"))`
- Quantity editable only when `offer.data.attributes.allow_multibuy__limio` is true
- Remove by `item.id` (the orderItem ID, not the offer ID)

---

## Pattern 4: Payment Method Display

From production components. Payment methods come per-subscription.

```javascript
import { useLimioUserSubscriptionPaymentMethods } from "@limio/internal-checkout-sdk"

function PaymentMethodDisplay({ subscriptionId }) {
  const { payment_methods, revalidate } = useLimioUserSubscriptionPaymentMethods(subscriptionId)

  if (!payment_methods?.length) {
    return <p>No payment method on file.</p>
  }

  const pm = payment_methods[0]
  // Payment method data lives in pm.data — structure varies by provider:
  // Zuora: pm.data.zuora.result.CreditCardType, pm.data.zuora.result.CreditCardMaskNumber
  // External: pm.data.integrationData.self_service.brand, .last4, .label
  // Generic: pm.data.card.brand, pm.data.card.last4, pm.data.card.exp_month, pm.data.card.exp_year

  const brand = pm.data?.card?.brand
    || pm.data?.integrationData?.self_service?.brand
    || pm.data?.method
    || "Card"

  const last4 = pm.data?.card?.last4
    || pm.data?.integrationData?.self_service?.last4
    || ""

  return (
    <div>
      {brand && <span>{brand}</span>}
      {last4 && <span>ending in {last4}</span>}
    </div>
  )
}
```

**Key rules:**
- Hook is `useLimioUserSubscriptionPaymentMethods` (with `Limio` prefix) from `@limio/internal-checkout-sdk`
- Takes `subscriptionId` as argument
- Returns `{ payment_methods, revalidate }`
- Payment data structure varies by provider — always use optional chaining
- NEVER hardcode card info like "•••• 4242" or "Visa ending in 1234"

---

## Pattern 5: Address Display

```javascript
import { useLimioUserSubscriptionAddresses } from "@limio/internal-checkout-sdk"
import { getCurrentAddress, addressSummary, formatCountry } from "@limio/sdk"

function AddressDisplay({ subscriptionId }) {
  const { addresses, revalidate } = useLimioUserSubscriptionAddresses(subscriptionId)

  const deliveryAddress = getCurrentAddress("delivery", addresses)
  const billingAddress = getCurrentAddress("billing", addresses)

  // Address data shape: address.data.{ firstName, lastName, address1, address2, city, state, postalCode, country }
  const renderAddress = (addr) => {
    if (!addr?.data?.address1) return <p>No address on file.</p>
    const d = addr.data
    return (
      <div>
        {d.firstName && <span>{d.firstName} {d.lastName}</span>}
        <span>{d.address1}</span>
        {d.address2 && <span>{d.address2}</span>}
        <span>{d.city}, {d.state} {d.postalCode}</span>
        {d.country && <span>{formatCountry(d.country)}</span>}
      </div>
    )
  }

  return (
    <div>
      <h3>Delivery Address</h3>
      {renderAddress(deliveryAddress)}

      <h3>Billing Address</h3>
      {renderAddress(billingAddress)}
    </div>
  )
}
```

**Key rules:**
- Hook is `useLimioUserSubscriptionAddresses` (with `Limio` prefix) from `@limio/internal-checkout-sdk`
- Use `getCurrentAddress(type, addresses)` from `@limio/sdk` to extract by type ("billing" or "delivery")
- Address fields are in `address.data.{field}` — NOT top-level
- Use `formatCountry(countryCode)` to display country names
- Use `addressSummary(address)` for a one-line formatted string

---

## Pattern 6: Subscription Info Display

```javascript
import { useSubscriptions, getCurrentOffer, useSchedule, useSubInfo, formatDate, sanitiseHTML } from "@limio/sdk"

function SubscriptionInfo() {
  const { subscriptions } = useSubscriptions()

  return (
    <div>
      {(subscriptions || []).map((sub) => {
        const currentOffer = getCurrentOffer(sub) // IMPORTED — never reimplement
        const attrs = currentOffer?.data?.attributes || {}
        const { status, isGift, quantity, hasLapsed } = useSubInfo(sub)
        const { nextPaymentAmount, renewalPrice, termStartDate, termEndDate } = useSchedule(sub)

        return (
          <div key={sub.id}>
            {/* Top-level subscription fields */}
            <p>Reference: {sub.reference}</p>
            <p>Status: {sub.status}</p>
            <p>Name: {sub.name}</p>

            {/* Offer attributes (from getCurrentOffer) */}
            {attrs.display_name__limio && <h3>{attrs.display_name__limio}</h3>}
            {attrs.display_price__limio && (
              <div dangerouslySetInnerHTML={{ __html: sanitiseHTML(attrs.display_price__limio) }} />
            )}
            {attrs.offer_features__limio && (
              <div dangerouslySetInnerHTML={{ __html: sanitiseHTML(attrs.offer_features__limio) }} />
            )}

            {/* Schedule info (from useSchedule) */}
            {nextPaymentAmount && <p>Next payment: {nextPaymentAmount}</p>}
            {termEndDate && <p>Renewal date: {termEndDate}</p>}
          </div>
        )
      })}
    </div>
  )
}
```

**Key rules:**
- `subscription.status`, `subscription.name`, `subscription.id`, `subscription.reference` are **top-level** fields
- `__limio` attributes live on **OFFERS**, not subscriptions
- Use `getCurrentOffer(subscription)` (imported) to get the current active standard offer
- Use `useSchedule(subscription)` for formatted payment amounts and dates
- Use `useSubInfo(subscription)` for computed flags (isGift, hasLapsed, etc.)

---

## Pattern 7: Customer Info Display

```javascript
import { useUser } from "@limio/sdk"

function CustomerInfo() {
  const { attributes, loginStatus } = useUser()

  if (loginStatus !== "logged-in" || !attributes) {
    return null
  }

  return (
    <div>
      {attributes.firstName && <p>Name: {attributes.firstName} {attributes.lastName}</p>}
      {attributes.email && <p>Email: {attributes.email}</p>}
    </div>
  )
}
```

**Key rules:**
- User data comes from `useUser()` — returns `{ attributes, loginStatus, loaded, token }`
- Name fields: `attributes.firstName`, `attributes.lastName`
- Email: `attributes.email`
- NEVER hardcode dummy user data. Use conditional rendering if field is undefined.

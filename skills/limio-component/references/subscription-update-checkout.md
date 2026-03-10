# Subscription Update Checkout

This reference documents the full flow for subscription updates (upgrades, downgrades, add-on changes) using the Limio SDK.

---

## Overview

Subscription update checkouts let existing subscribers change their plan. The system uses `order_type: "update_subscription"` which routes mutations through the subscription update API rather than standard acquisition flows.

**Key principle:** Only one offer can be active at a time — selecting a new offer resets the checkout and re-resolves server-side state (effective dates, proration, removal side-effects).

---

## Workflow

1. Call `initiateCheckout` with subscription ID and `order_type: "update_subscription"`
2. Redirect to update checkout page using the returned `checkoutId`
3. Subscriber selects a new offer via `selectOfferForSubscriptionUpdate`
4. Server resolves effective dates and proration
5. Updated basket returned with next actions

---

## Step 1: Initiate Update Checkout

```javascript
import { useBasket } from "@limio/sdk"

const { initiateCheckout } = useBasket()

const handleUpdateSubscription = async (subscriptionId) => {
  const basket = await initiateCheckout({
    order: {
      order_type: "update_subscription",
      forSubscription: { id: subscriptionId }
    }
  })
  // basket.order.checkoutId contains the update session ID
  // Redirect to update checkout page with checkoutId and ownerId
}
```

**Important:** Always pass the `ownerId` query parameter through redirects to identify the subscription being updated.

---

## Step 2: Select an Offer for Update

```javascript
import { useBasket } from "@limio/sdk"

const { selectOfferForSubscriptionUpdate, clearOrderItems } = useBasket()

const handleSelectOffer = async (offer) => {
  // Clear existing selection first to prevent stale server state
  clearOrderItems()

  await selectOfferForSubscriptionUpdate({
    offer,     // ElasticOffer — the target plan
    quantity: 1 // Typically 1 for plan changes
  })
}
```

### selectOfferForSubscriptionUpdate

**Signature:**
```typescript
selectOfferForSubscriptionUpdate(
  orderItem: { offer: ElasticOffer, quantity?: number }
): Promise<void>
```

**Behavior:** Selecting a new offer automatically resets the checkout and triggers server-side recalculation of effective dates, proration, and removal side-effects.

### clearOrderItems

Call `clearOrderItems()` **before** `selectOfferForSubscriptionUpdate` when subscribers change their selection. This synchronous operation empties local basket state to prevent stale server state.

---

## Step 3: Access Update Checkout State

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })
const checkoutState = useCheckoutSelector(state => state) || {}
const { order, paidSchedule, schedule, locale, nextActions } = checkoutState

// nextActions contains upgrade/downgrade/cross-sell options
const { upgrades, downgrades, crossSells } = nextActions || {}
```

Server responses for update checkouts may include fields absent from standard checkouts:
- Effective dates for the plan change
- Proration details
- Next actions (upgrades, downgrades, cross-sells)

---

## updateItemQuantity in Update Context

In subscription update checkouts, `updateItemQuantity` automatically routes through the subscription update API for proration recalculation (rather than standard basket endpoints):

```javascript
const { updateItemQuantity } = useBasket()

await updateItemQuantity(itemId, newQuantity)
// Triggers server-side proration recalculation
```

---

## Key Notes

- The SDK automatically detects `order_type: "update_subscription"` and routes basket operations through the subscription update API
- Always pass `ownerId` as a query parameter when redirecting to the update checkout page
- Selecting a new offer resets the entire checkout — the server re-resolves all pricing, effective dates, and proration
- Use `clearOrderItems()` before `selectOfferForSubscriptionUpdate()` when changing the selected offer
- `nextActions` from `useCheckout` provides available upgrades, downgrades, and cross-sells for the current subscription

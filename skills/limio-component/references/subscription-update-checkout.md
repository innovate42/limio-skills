# Subscription Update Checkout

This reference documents the full flow for subscription updates (upgrades, downgrades, add-on changes) using the Limio SDK.

---

## Overview

Subscription update checkouts let existing subscribers change their plan. The system uses `order_type: "update_subscription"` which routes mutations through the subscription update API rather than standard acquisition flows.

**Key principle:** Only one subscription offer change can happen at a time — selecting a new offer resets the checkout and re-resolves server-side state (effective dates, proration, removal of incompatible add-ons).

---

## Workflow

1. User clicks "Update" on a subscription
2. Frontend calls `initiateCheckout` with `order_type: "update_subscription"` and the subscription ID
3. **Backend resolves** the subscription's current offer, fetches `upgrade_offers__limio` and `downgrade_offers__limio` from its attributes, and returns them as full offer objects in `nextActions`
4. Frontend redirects to update page, renders upgrade/downgrade options from `nextActions`
5. User selects a new offer → `selectOfferForSubscriptionUpdate`
6. Frontend navigates to checkout → user submits

---

## Step 1: Initiate Update Checkout

```javascript
import { useBasket } from "@limio/sdk"

const { initiateCheckout } = useBasket()

const handleUpdateSubscription = async (subscriptionId) => {
  const { id: basketId } = await initiateCheckout({
    order: {
      order_type: "update_subscription",
      forSubscription: { id: subscriptionId }
    }
  })
  // Redirect to update offer selection page with basket ID
  window.location.href = `${updatePageUrl}?basket=${basketId}`
}
```

**Important:** Always pass the `ownerId` query parameter through redirects to identify the subscription being updated.

---

## Step 2: Read Available Upgrades & Downgrades

After `initiateCheckout`, the backend resolves the subscription's current offer and fetches the upgrade/downgrade offers configured on it (from `upgrade_offers__limio` and `downgrade_offers__limio` attributes). These are returned as **full offer objects** in `nextActions` — you do NOT need to resolve references yourself.

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"
import { useLimioUserSubscription, getCurrentOffer } from "@limio/sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })

// nextActions contains already-resolved full offer objects
const { upgrades = [], downgrades = [], crossSells = [], subscriptionAddOns = [] } =
  useCheckoutSelector((state) => state.nextActions) || {}

// Get the current subscription and its active offer for comparison
const subscriptionId = useCheckoutSelector((state) => state.order.forSubscription?.id)
const { subscription } = useLimioUserSubscription(subscriptionId)
const currentOffer = getCurrentOffer(subscription)
```

**`nextActions` shape:**
- `upgrades` — Array of full offer objects the subscriber can upgrade to
- `downgrades` — Array of full offer objects the subscriber can downgrade to
- `crossSells` — Array of compatible add-on offers
- `subscriptionAddOns` — Array of add-ons already owned on this subscription

---

## Step 3: Select an Offer for Update

When the user clicks an upgrade or downgrade:

```javascript
import { useBasket } from "@limio/sdk"

const { selectOfferForSubscriptionUpdate, clearOrderItems, navigateToCheckout } = useBasket()

const handleSelectOffer = async (offer) => {
  // Clear existing selection first to prevent stale server state
  clearOrderItems()

  // Select the new offer — note the full orderItem shape
  await selectOfferForSubscriptionUpdate({
    orderItemActionType: "add",
    offer: offer,
    type: offer.record_type || "offer",
    quantity: 1
  })

  // Navigate to checkout page (path configured per offer)
  const checkoutPath = offer.data?.attributes?.update_configuration__limio || "/update"
  await navigateToCheckout({
    journey: { checkout: checkoutPath }
  })
}
```

### selectOfferForSubscriptionUpdate

**Signature:**
```typescript
selectOfferForSubscriptionUpdate(orderItem: {
  orderItemActionType: "add",
  offer: ElasticOffer,
  type: string,          // "offer" or offer.record_type
  quantity: number
}): Promise<void>
```

**Behavior:**
- Sends a PUT to the subscription checkout handler
- Backend automatically removes the old subscription offer and adds the new one
- Backend removes any add-ons incompatible with the new offer
- Server recalculates effective dates, proration, and pricing
- Only one offer change at a time — selecting again replaces the previous selection

### clearOrderItems

Call `clearOrderItems()` **before** `selectOfferForSubscriptionUpdate` when subscribers change their selection. This synchronous operation empties local basket state to prevent stale server state.

---

## Step 4: Checkout Confirmation

On the checkout page, display the selected offer and submit:

```javascript
import { useCheckout } from "@limio/internal-checkout-sdk"

const { useCheckoutSelector } = useCheckout({ redirectOnFailure: false })
const orderItems = useCheckoutSelector((state) => state.order.orderItems) || []

// Filter to show only items being added (not removals of old offers)
const addedOffer = orderItems.find(
  (item) => item.orderItemActionType !== "remove" && item.type === "offer"
)
const addedAddOns = orderItems.filter(
  (item) => item.orderItemActionType !== "remove" && item.type === "add_on"
)
```

The order is submitted via `OrderForm` with `actions.sendOrder` — the backend processes add/remove actions, creates new subscription_offer records, and sets end dates on old ones.

---

## Offer Attributes for Update Flows

These attributes on the offer control the update flow:

| Attribute | Type | Description |
|-----------|------|-------------|
| `upgrade_offers__limio` | `{path, id, label}[]` | Offers available as upgrades |
| `downgrade_offers__limio` | `{path, id, label}[]` | Offers available as downgrades |
| `update_configuration__limio` | `string` | Checkout page path for update flow (e.g., `"/update"`) |
| `upgrade_cta__limio` | `string` | CTA text for upgrade button |
| `downgrade_cta__limio` | `string` | CTA text for downgrade button |

**Note:** `upgrade_offers__limio` and `downgrade_offers__limio` are references configured on the offer. The backend resolves them into full offer objects and returns them via `nextActions` — the frontend does NOT need to fetch or resolve them.

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
- `nextActions` from `useCheckout` provides **already-resolved full offer objects** for upgrades, downgrades, and cross-sells — no need to resolve references from attributes
- Selecting a new offer resets the entire checkout — the server re-resolves all pricing, effective dates, and proration
- The backend auto-removes the old subscription offer and any incompatible add-ons when a new offer is selected
- Use `clearOrderItems()` before `selectOfferForSubscriptionUpdate()` when changing the selected offer
- Display only `orderItemActionType !== "remove"` items in the cart UI — removals are handled server-side

# Express Checkout (Apple Pay & Google Pay)

This reference documents how to implement express checkout flows using `@limio/payment-sdk`.

---

## Apple Pay

**Requires:** Limio v107+

### Imports

```javascript
import { OrderForm, ExpressApplePayButton } from "@limio/payment-sdk"
```

### OrderForm

Wraps checkout flow — validates order, submits, and redirects on completion.

```jsx
<OrderForm
  orderCompleteURL="/complete"
  onError={(error) => console.error(error.message, error.code)}
>
  <ExpressApplePayButton onClick={() => setError(null)} />
</OrderForm>
```

**Props:**
- `orderCompleteURL` (optional) — Redirect destination after success; defaults to `"/complete"`. Checkout ID is appended automatically.
- `onError` — Callback receiving error object with `message` and `code`

### ExpressApplePayButton

Renders the Apple Pay button that processes wallet payments.

**Props:**
- `onClick` (optional but recommended) — Pre-trigger callback for UI updates/error reset
- `children` (optional) — Child components rendered inside button

### Customer Details

- **Authenticated users:** First name, last name, email pulled from auth token
- **Anonymous users:** Details sourced from Apple Pay sheet

### Limitations

- Users cannot modify details during checkout
- Digital offers only (no delivery address support)
- Zuora only supports Apple Pay with credit cards on specific gateway integrations
- Apple Pay on web works exclusively in Safari
- Recurring transactions are processed as credit card transactions

---

## Google Pay

**Requires:** Limio v102+

### Option 1: GooglePayButton Component

```javascript
import { GooglePayButton } from "@limio/payment-sdk"
```

```jsx
<GooglePayButton
  offer={selectedOffer}
  buttonType="subscribe"
  buttonColor="black"
  buttonRadius={4}
  orderCompleteUrl="/complete"
  redirectUrl="/checkout"  // Fallback redirect
/>
```

**Props:**
- `offer` — The selected offer object
- `buttonType` — Button label type (e.g., `"subscribe"`, `"buy"`, `"pay"`)
- `buttonColor` — `"black"` | `"white"`
- `buttonRadius` — Border radius in pixels
- `orderCompleteUrl` — Redirect on success
- `redirectUrl` — Fallback redirect if Google Pay is unavailable

Renders a `<div data-testid="google-pay-container">`. Falls back to an unstyled button that adds the item to basket if Google Pay is unavailable.

### Option 2: useGooglePay Hook

For custom button placement and offer selection flows.

```javascript
import { useGooglePay } from "@limio/payment-sdk"

const { createButton, setOffer, client, initialized } = useGooglePay(
  initialOffer,
  "/complete" // orderCompleteURL
)
```

**Return shape:**
- `createButton(containerRef, fallbackButton, options)` — Renders Google Pay button at the ref, or fallback if unavailable
  - `containerRef` — Target HTML element ref
  - `fallbackButton` — React component fallback
  - `options` — `{ phoneRequired?: boolean, buttonConfig: { buttonColor, buttonType, buttonRadius } }`
- `setOffer(offer)` — Update the offer for selectable offer flows
- `client` — Google Pay window object
- `initialized` — Boolean, true when script is loaded

### Testing

- Set mode key to `"TEST"` in Limio App Google Settings
- Works in recent Chrome browsers
- Test cards available via Google Pay documentation
- iframes receive automatic payment allowance

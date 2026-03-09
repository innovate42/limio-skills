---
name: limio-sdk-verify
description: Audits existing Limio component code against official SDK documentation. Use when user asks to "verify SDK usage", "audit Limio code", "check SDK compliance", "review Limio imports", "is this using the SDK correctly", "check limio best practices", or reviews existing component code for correctness. Do NOT use for creating components (use limio-component), creating stories (use limio-story), or deploying (use limio-setup).
metadata:
  author: Limio
  version: 8.0.0
---

# Limio SDK Verification

Use this skill to audit existing Limio component code for correct SDK usage, deprecated patterns, security issues, and best practice compliance. This is a **developer-facing** skill for verifying that components use `@limio/sdk` correctly.

**IMPORTANT:** When the MCP tool `mcp__claude_ai_Limio_External_Docs__searchDocumentation` is available, use it to cross-reference findings against the live Limio documentation at https://docs.limio.com/developers/limio-sdk. This provides the most up-to-date SDK guidance and can catch issues not covered by static rules.

**CRITICAL — Credential Safety:** `.limio.json` contains OAuth client credentials (client ID + client secret). It MUST be in `.gitignore` and MUST NEVER be committed to git. Flag this immediately if found tracked or staged.

---

## Verification Checklist

When auditing a Limio component, check each of the following areas in order:

### 1. Imports

- [ ] All SDK hooks imported from `@limio/sdk` (not relative paths)
- [ ] `getCurrentBasketId` imported from `@limio/shop/src/shop/checkout/basket`
- [ ] No imports from `../source/utils/` or other internal paths
- [ ] `package.json` uses default import: `import packageData from "./package.json"` (NOT `import * as packageData`)
- [ ] `useComponentProps` imported from `@limio/sdk`, `getPropsFromPackageJson` from `@limio/components/helpers`
- [ ] No unused SDK imports

### 2. Hooks Usage

- [ ] All hooks called with null safety: `const { offers } = useCampaign() || {}`
- [ ] `useLimioContext()` called with `|| {}` fallback
- [ ] `useStaticProps()` called with `|| {}` fallback
- [ ] Hooks called at top level of component (not inside conditionals or loops)
- [ ] No deprecated hook patterns

### 3. Data Access

- [ ] Offer attributes accessed via `offer?.data?.attributes?.` with optional chaining
- [ ] Subscription offers accessed via `subscription.offers[]` array (NOT `subscription.data.offer`)
- [ ] Price accessed via `offer?.data?.price?.[0]` with null check
- [ ] Products accessed via `offer?.data?.products` with null check
- [ ] Group accessed via `offer?.data?.attributes?.group__limio`

### 4. Security

- [ ] Rich text/HTML sanitized with `xss` library or `sanitiseHTML` from SDK
- [ ] No unsanitized `dangerouslySetInnerHTML`
- [ ] `.limio.json` not committed or staged in git
- [ ] No hardcoded credentials or tokens

### 5. CSS Patterns

- [ ] All CSS classes prefixed with component abbreviation (e.g., `oc-`, `sp-`, `ad-`) to avoid collisions
- [ ] CSS custom properties used for configurable colors (e.g., `--my-primary`)
- [ ] Color limioProps mapped to CSS variables via `style` attribute
- [ ] Responsive breakpoint included: `@media (max-width: 600px)`
- [ ] Box-sizing reset scoped to component

### 6. limioProps

- [ ] All user-facing text defined as limioProps (not hardcoded)
- [ ] List prop items use `{id, label}` objects (not plain strings)
- [ ] Picklist options use `{id, label, value}` objects
- [ ] Color props use `"type": "color"` (no special suffix on ID)
- [ ] Rich text props use `"type": "richtext"` (no special suffix on ID)
- [ ] Props have sensible defaults
- [ ] No legacy `__limio_richtext` or `__limio_color` suffixes on prop IDs

### 7. Basket & Checkout

- [ ] Uses `initiateCheckout` (NOT deprecated `addToBasket`)
- [ ] Uses `orderItems` (NOT deprecated `basketItems`)
- [ ] Checks `getCurrentBasketId()` before deciding `initiateCheckout` vs `addOfferToBasket`
- [ ] Handles `basketLoading` state to prevent double submissions
- [ ] Checks `pageOptions?.pushToCheckout` for auto-navigation

### 8. Page Builder Compatibility

- [ ] Components with `position: fixed` or `position: absolute` check `isInPageBuilder` from `useLimioContext()`
- [ ] Falls back to `position: relative` when `isInPageBuilder` is true

---

## Deprecated Methods & Correct Replacements

These are the most common deprecated patterns. Flag any occurrence and provide the replacement.

| Deprecated | Replacement | Notes |
|-----------|-------------|-------|
| `addToBasket(offer)` | `initiateCheckout({ order: { orderItems: [{ offer }] } })` | Must also check `getCurrentBasketId()` first |
| `basketItems` | `orderItems` | From `useBasket()` return value |
| `subscription.data.offer` | `subscription.offers[]` | Legacy field — subscriptions can have multiple offers |
| `import * as packageData from "./package.json"` | `import packageData from "./package.json"` | Must use default import |
| `id: "foo__limio_richtext"` | `id: "foo"` with `type: "richtext"` | Legacy suffix — type field is sufficient |
| `id: "foo__limio_color"` | `id: "foo"` with `type: "color"` | Legacy suffix — type field is sufficient |
| `type: "richText"` (camelCase) | `type: "richtext"` (lowercase) | Modern convention uses lowercase |

### Detailed Replacement Patterns

**addToBasket → initiateCheckout:**

```javascript
// WRONG (deprecated)
const { addToBasket } = useBasket()
addToBasket(offer)

// CORRECT
const { initiateCheckout, addOfferToBasket, navigateToCheckout, pageOptions } = useBasket()
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

**subscription.data.offer → subscription.offers[]:**

```javascript
// WRONG (legacy)
const offerData = subscription.data.offer

// CORRECT
import { checkActiveOffers, getCurrentOffer } from "@limio/sdk"

const activeOffers = checkActiveOffers(subscription.offers, false)
const currentOffer = getCurrentOffer(subscription)
// Or manually filter:
const standardOffers = subscription.offers?.filter(
  o => o.data?.record_subtype !== "discount"
) || []
```

**import * as packageData → default import:**

```javascript
// WRONG
import * as packageData from "./package.json"

// CORRECT
import packageData from "./package.json"
```

---

## Common Anti-Patterns & Fixes

### 1. Missing Null Safety on Hooks

```javascript
// ANTI-PATTERN
const { offers } = useCampaign()        // crashes if useCampaign() returns null/undefined

// FIX
const { offers } = useCampaign() || {}
```

### 2. Internal Path Imports

```javascript
// ANTI-PATTERN
import { formatPrice } from "../source/utils/helpers"

// FIX — use SDK utilities
import { formatCurrency, formatDisplayPrice } from "@limio/sdk"
```

### 3. Unsanitized HTML

```javascript
// ANTI-PATTERN
<div dangerouslySetInnerHTML={{ __html: offer.data.attributes.offer_features__limio }} />

// FIX
import xss from "xss"
<div dangerouslySetInnerHTML={{ __html: xss(offer?.data?.attributes?.offer_features__limio || "") }} />
```

### 4. Hardcoded Text

```javascript
// ANTI-PATTERN
<h1>Choose Your Plan</h1>
<button>Subscribe Now</button>

// FIX — use limioProps
const { heading = "Choose Your Plan", ctaText = "Subscribe Now" } = useStaticProps() || {}
<h1>{heading}</h1>
<button>{ctaText}</button>
```

### 5. Missing basketLoading Guard

```javascript
// ANTI-PATTERN
<button onClick={() => handleAddToBasket(offer)}>Buy</button>

// FIX
const { basketLoading } = useBasket()
<button onClick={() => handleAddToBasket(offer)} disabled={basketLoading}>
  {basketLoading ? "Processing..." : ctaText}
</button>
```

### 6. No isInPageBuilder Check

```javascript
// ANTI-PATTERN
<div style={{ position: "fixed", bottom: 0 }}>Sticky CTA</div>

// FIX
const { isInPageBuilder } = useLimioContext() || {}
<div style={{ position: isInPageBuilder ? "relative" : "fixed", bottom: isInPageBuilder ? "auto" : 0 }}>
  Sticky CTA
</div>
```

### 7. Wrong MUI Version

```json
// ANTI-PATTERN
"@mui/material": "^6.0.0"

// FIX — use 5.16.12 for React 19 compatibility
"@mui/material": "5.16.12"
```

### 8. Plain Strings in List Props

```json
// ANTI-PATTERN
"default": ["Monthly", "Annual"]

// FIX
"default": [{ "id": "monthly", "label": "Monthly" }, { "id": "annual", "label": "Annual" }]
```

### 9. Missing CSS Class Prefixing

```css
/* ANTI-PATTERN — collision risk */
.card { ... }
.header { ... }

/* FIX — prefix with component abbreviation */
.oc-card { ... }
.oc-header { ... }
```

### 10. No Responsive Breakpoint

```css
/* ANTI-PATTERN — no mobile styles */
.oc-grid { display: grid; grid-template-columns: repeat(3, 1fr); }

/* FIX */
.oc-grid { display: grid; grid-template-columns: repeat(3, 1fr); }
@media (max-width: 600px) {
  .oc-grid { grid-template-columns: 1fr; }
}
```

### 11. Hardcoded Colors Instead of CSS Custom Properties

```css
/* ANTI-PATTERN */
.oc-button { background: #635BFF; }

/* FIX */
.oc-button { background: var(--oc-primary, #635BFF); }
```

```javascript
// Map color limioProps to CSS variables
<div className="oc-wrapper" style={{ "--oc-primary": primaryColor }}>
```

---

## Quick Reference — Correct Patterns

### componentStaticProps.js

```javascript
import { useComponentProps } from "@limio/sdk"
import { getPropsFromPackageJson } from "@limio/components/helpers"
import packageData from "./package.json"

const defaultComponentProps = getPropsFromPackageJson(packageData)

export function useStaticProps() {
    return useComponentProps(defaultComponentProps)
}
```

### Hook Initialization

```javascript
const { offers, campaign, addOns, groupValues } = useCampaign() || {}
const { orderItems, basketLoading, initiateCheckout, addOfferToBasket, navigateToCheckout, pageOptions } = useBasket()
const { attributes, loginStatus, loaded } = useUser()
const { subscriptions } = useSubscriptions()
const { isInPageBuilder } = useLimioContext() || {}
const props = useStaticProps() || {}
```

### Add to Basket

```javascript
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"

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

### Safe Attribute Access

```javascript
const attributes = offer?.data?.attributes || {}
const displayName = attributes.display_name__limio || "Untitled"
const features = attributes.offer_features__limio || ""
const price = offer?.data?.price?.[0]
const amount = price?.value || 0
const currency = price?.currencyCode || "USD"
```

### Subscription Offers

```javascript
import { checkActiveOffers, getCurrentOffer } from "@limio/sdk"

// Get active standard offers (not discounts)
const activeOffers = checkActiveOffers(subscription.offers, false)

// Or get the current offer directly
const currentOffer = getCurrentOffer(subscription)
```

### HTML Sanitization

```javascript
import xss from "xss"

const sanitizeString = (str) => xss(str || "")
// or
import { sanitiseHTML } from "@limio/sdk"
<div dangerouslySetInnerHTML={{ __html: sanitiseHTML(htmlContent) }} />
```

---

## Verification Output Format

When reporting verification results, use this format:

```
## SDK Verification Report

### Summary
- Issues found: N
- Warnings: N
- Status: PASS / NEEDS FIXES

### Issues (must fix)
1. [FILE:LINE] Description — Replacement

### Warnings (should fix)
1. [FILE:LINE] Description — Recommendation

### Passed Checks
- ✓ Imports correct
- ✓ Null safety present
- ...
```

---

## Reference Documents

- `references/common-mistakes.md` — Detailed before/after examples of every common mistake
- `references/sdk-full-reference.md` — Complete SDK hook and utility reference with all return shapes

---

## Related Skills

- `limio-component` — Create new components
- `limio-story` — Create Storybook stories
- `limio-storybook` — Set up the Storybook playground
- `limio-setup` — Credentials and deployment

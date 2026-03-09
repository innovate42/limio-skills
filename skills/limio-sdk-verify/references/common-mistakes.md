# Common Mistakes — Detailed Examples

Comprehensive before/after examples of common mistakes in Limio components. Each section includes the incorrect code, an explanation of why it is wrong, and the corrected version.

---

## 1. Using `subscription.data.offer` Instead of `subscription.offers[]`

A subscription can have multiple offers (e.g., a standard offer plus a discount offer). The `subscription.data.offer` field is legacy and only returns a single offer.

**Before (wrong):**
```javascript
const MySubscription = ({ subscription }) => {
  const offer = subscription.data.offer
  const displayName = offer.data.attributes.display_name__limio

  return <h2>{displayName}</h2>
}
```

**After (correct):**
```javascript
import { checkActiveOffers, getCurrentOffer } from "@limio/sdk"

const MySubscription = ({ subscription }) => {
  // Use the offers array — the documented access pattern
  const activeOffers = checkActiveOffers(subscription.offers, false)
  const currentOffer = getCurrentOffer(subscription)

  // Or manually filter for the standard (non-discount) offer
  const standardOffers = subscription.offers?.filter(
    o => o.data?.record_subtype !== "discount"
  ) || []
  const currentStandard = standardOffers[0]
  const offerData = currentStandard?.data?.offer
  const displayName = offerData?.data?.attributes?.display_name__limio || "Unknown"

  return <h2>{displayName}</h2>
}
```

---

## 2. Missing Null Safety on Hooks

SDK hooks can return `null` or `undefined` in certain contexts (e.g., during server-side rendering, before data loads, or in Storybook without mocks). Destructuring without a fallback causes a runtime crash.

**Before (wrong):**
```javascript
const MyComponent = () => {
  const { offers } = useCampaign()
  const { isInPageBuilder } = useLimioContext()
  const { heading } = useStaticProps()

  return <h1>{heading}</h1>
}
```

**After (correct):**
```javascript
const MyComponent = () => {
  const { offers } = useCampaign() || {}
  const { isInPageBuilder } = useLimioContext() || {}
  const props = useStaticProps() || {}
  const { heading = "Default Heading" } = props

  return <h1>{heading}</h1>
}
```

---

## 3. Importing from `../source/utils/` Instead of SDK

Components that import from internal project paths are not self-contained. These paths rely on modules like `@limio/shop/src/shop/appConfig.js` that are not available in Storybook and will cause build failures.

**Before (wrong):**
```javascript
import { formatPrice } from "../source/utils/pricing"
import { validateEmail } from "../source/utils/validation"
import { trackEvent } from "../source/utils/analytics"
```

**After (correct):**
```javascript
// Use SDK utilities
import { formatCurrency, formatDisplayPrice, formatDate } from "@limio/sdk"

// For things not in the SDK, write small inline helpers
const validateEmail = (email) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
```

---

## 4. Using `addToBasket` Instead of `initiateCheckout` Pattern

The `addToBasket` method is deprecated. The correct pattern uses `initiateCheckout` for new baskets and `addOfferToBasket` for existing baskets.

**Before (wrong):**
```javascript
const { addToBasket } = useBasket()

const handleClick = (offer) => {
  addToBasket(offer)
}
```

**After (correct):**
```javascript
import { getCurrentBasketId } from "@limio/shop/src/shop/checkout/basket"

const {
  initiateCheckout,
  addOfferToBasket,
  navigateToCheckout,
  basketLoading,
  pageOptions,
} = useBasket()

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

## 5. Not Handling `basketLoading`

Without checking `basketLoading`, users can click the add-to-basket button multiple times during an async operation, leading to duplicate items or race conditions.

**Before (wrong):**
```javascript
const { initiateCheckout } = useBasket()

return (
  <button onClick={() => handleAddToBasket(offer)}>
    Subscribe
  </button>
)
```

**After (correct):**
```javascript
const { initiateCheckout, basketLoading } = useBasket()
const { ctaText = "Subscribe" } = useStaticProps() || {}

return (
  <button
    onClick={() => handleAddToBasket(offer)}
    disabled={basketLoading}
  >
    {basketLoading ? "Processing..." : ctaText}
  </button>
)
```

---

## 6. Missing `isInPageBuilder` Check for Fixed/Absolute Positioned Components

Components using `position: fixed` or `position: absolute` will break the Limio Page Builder editor by overlapping the builder controls and escaping their section bounds.

**Before (wrong):**
```javascript
const StickyBanner = () => {
  return (
    <div style={{ position: "fixed", bottom: 0, left: 0, right: 0, zIndex: 1000 }}>
      <p>Special offer — subscribe now!</p>
    </div>
  )
}
```

**After (correct):**
```javascript
import { useLimioContext } from "@limio/sdk"

const StickyBanner = () => {
  const { isInPageBuilder } = useLimioContext() || {}

  return (
    <div
      style={{
        position: isInPageBuilder ? "relative" : "fixed",
        bottom: isInPageBuilder ? "auto" : 0,
        left: isInPageBuilder ? "auto" : 0,
        right: isInPageBuilder ? "auto" : 0,
        zIndex: isInPageBuilder ? "auto" : 1000,
      }}
    >
      <p>Special offer — subscribe now!</p>
    </div>
  )
}
```

---

## 7. Using `import * as packageData` Instead of Default Import

The `import * as` syntax creates a module namespace object, which does not match what `getPropsFromPackageJson` expects. Use a default import.

**Before (wrong):**
```javascript
import * as packageData from "./package.json"
import { useComponentProps, getPropsFromPackageJson } from "@limio/sdk"

const defaultComponentProps = getPropsFromPackageJson(packageData)
```

**After (correct):**
```javascript
import packageData from "./package.json"
import { useComponentProps } from "@limio/sdk"
import { getPropsFromPackageJson } from "@limio/components/helpers"

const defaultComponentProps = getPropsFromPackageJson(packageData)
```

---

## 8. Not Sanitizing HTML (Missing xss / sanitiseHTML)

Limio offer attributes like `offer_features__limio`, `display_price__limio`, and `detailed_display_price__limio` contain HTML. Rendering them without sanitization is an XSS vulnerability.

**Before (wrong):**
```javascript
const OfferCard = ({ offer }) => {
  const features = offer?.data?.attributes?.offer_features__limio

  return (
    <div dangerouslySetInnerHTML={{ __html: features }} />
  )
}
```

**After (correct):**
```javascript
import xss from "xss"

const sanitizeString = (str) => xss(str || "")

const OfferCard = ({ offer }) => {
  const features = offer?.data?.attributes?.offer_features__limio

  return (
    <div dangerouslySetInnerHTML={{ __html: sanitizeString(features) }} />
  )
}
```

Or using the SDK utility (preferred when available):
```javascript
import { sanitiseHTML } from "@limio/sdk"

<div dangerouslySetInnerHTML={{ __html: sanitiseHTML(features) }} />
```

---

## 9. Hardcoded Text Instead of limioProps

All user-facing text must be configurable via limioProps so that the component is white-label and editable in the Limio Page Builder without code changes.

**Before (wrong):**
```javascript
const PricingHeader = () => {
  return (
    <div>
      <h1>Choose Your Plan</h1>
      <p>Select the plan that works best for you</p>
      <button>Subscribe Now</button>
    </div>
  )
}
```

**After (correct):**

In `package.json`:
```json
{
  "limioProps": [
    { "id": "heading", "label": "Heading", "type": "string", "default": "Choose Your Plan" },
    { "id": "subheading", "label": "Subheading", "type": "string", "default": "Select the plan that works best for you" },
    { "id": "ctaText", "label": "CTA text", "type": "string", "default": "Subscribe Now" }
  ]
}
```

In `index.js`:
```javascript
const PricingHeader = () => {
  const {
    heading = "Choose Your Plan",
    subheading = "Select the plan that works best for you",
    ctaText = "Subscribe Now",
  } = useStaticProps() || {}

  return (
    <div>
      <h1>{heading}</h1>
      <p>{subheading}</p>
      <button>{ctaText}</button>
    </div>
  )
}
```

---

## 10. Wrong MUI Version (Not 5.16.12)

MUI v6+ and some v5 versions have breaking changes or incompatibilities with React 19 in the Limio environment. Pin to `5.16.12`.

**Before (wrong):**
```json
{
  "dependencies": {
    "@mui/material": "^6.0.0",
    "@emotion/react": "^11.0.0",
    "@emotion/styled": "^11.0.0"
  }
}
```

**After (correct):**
```json
{
  "dependencies": {
    "@mui/material": "5.16.12",
    "@emotion/react": "^11.0.0",
    "@emotion/styled": "^11.0.0"
  }
}
```

---

## 11. Plain Strings in List Props Instead of `{id, label}` Objects

List-type limioProps expect arrays of objects with `id` and `label` fields. Using plain strings will not render correctly in the Limio Page Builder editor.

**Before (wrong):**
```json
{
  "id": "groupLabels",
  "label": "Group Labels",
  "type": "list",
  "fields": {
    "name": { "id": "id", "label": "ID", "type": "string" },
    "url": { "id": "label", "label": "Label", "type": "string" }
  },
  "default": ["Monthly", "Annual", "Weekly"]
}
```

**After (correct):**
```json
{
  "id": "groupLabels",
  "label": "Group Labels",
  "type": "list",
  "fields": {
    "name": { "id": "id", "label": "ID", "type": "string" },
    "url": { "id": "label", "label": "Label", "type": "string" }
  },
  "default": [
    { "id": "monthly", "label": "Monthly" },
    { "id": "annual", "label": "Annual" },
    { "id": "weekly", "label": "Weekly" }
  ]
}
```

---

## 12. Missing CSS Class Prefixing (Collision Risk)

Limio components run alongside other components on the same page. Generic class names like `.card`, `.header`, or `.button` will collide with other components and global styles.

**Before (wrong):**
```css
.card {
  border: 1px solid #e3e8ee;
  border-radius: 10px;
  padding: 24px;
}

.header {
  font-size: 24px;
  font-weight: 700;
}

.button {
  background: #635BFF;
  color: white;
}
```

**After (correct):**
```css
.oc-card {
  border: 1px solid var(--oc-border, #e3e8ee);
  border-radius: 10px;
  padding: 24px;
}

.oc-header {
  font-size: 24px;
  font-weight: 700;
}

.oc-button {
  background: var(--oc-primary, #635BFF);
  color: white;
}
```

Use a short abbreviation derived from the component name (e.g., `oc-` for "offer-cards", `sp-` for "subscription-panel", `ad-` for "account-dashboard").

---

## 13. Missing Responsive Breakpoint

Components must be usable on mobile devices. Without a responsive breakpoint, grid layouts and fixed-width elements will overflow or become unusable on smaller screens.

**Before (wrong):**
```css
.oc-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.oc-card {
  min-width: 320px;
}
```

**After (correct):**
```css
.oc-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.oc-card {
  min-width: 0; /* allow shrinking in grid */
}

@media (max-width: 600px) {
  .oc-grid {
    grid-template-columns: 1fr;
    gap: 16px;
  }
}
```

---

## 14. Not Using CSS Custom Properties for Configurable Colors

Hardcoding color values means the component cannot be white-labelled in the Page Builder. Colors should flow from limioProps through CSS custom properties.

**Before (wrong):**
```css
.oc-button {
  background: #635BFF;
  color: white;
}

.oc-badge {
  background: #E8E5FF;
  color: #635BFF;
}
```

```javascript
<div className="oc-wrapper">
  <button className="oc-button">Subscribe</button>
</div>
```

**After (correct):**
```css
.oc-wrapper {
  --oc-primary: #635BFF;
  --oc-primary-light: #E8E5FF;
}

.oc-button {
  background: var(--oc-primary);
  color: white;
}

.oc-badge {
  background: var(--oc-primary-light);
  color: var(--oc-primary);
}
```

```javascript
const { primaryColor = "#635BFF" } = useStaticProps() || {}

<div className="oc-wrapper" style={{ "--oc-primary": primaryColor }}>
  <button className="oc-button">Subscribe</button>
</div>
```

In `package.json`:
```json
{
  "limioProps": [
    { "id": "primaryColor", "label": "Primary color", "type": "color", "default": "#635BFF" }
  ]
}
```

---

## 16. Using Legacy `__limio_richtext` or `__limio_color` Suffixes on Prop IDs

The `__limio_richtext` and `__limio_color` suffixes on prop IDs are legacy. Modern components use the `type` field alone to determine the Page Builder control.

**Before (wrong):**
```json
{
  "limioProps": [
    { "id": "description__limio_richtext", "label": "Description", "type": "richText", "default": "<p>Content</p>" },
    { "id": "primaryColor__limio_color", "label": "Primary color", "type": "color", "default": "#635BFF" }
  ]
}
```

**After (correct):**
```json
{
  "limioProps": [
    { "id": "description", "label": "Description", "type": "richtext", "default": "<p>Content</p>" },
    { "id": "primaryColor", "label": "Primary color", "type": "color", "default": "#635BFF" }
  ]
}
```

Note: the type should be lowercase `"richtext"`, not `"richText"`.

---

## 15. Committing `.limio.json`

The `.limio.json` file contains OAuth client credentials (client ID and client secret). Committing it exposes secrets in the repository.

**Detection:**
```bash
git status          # Check if .limio.json appears as tracked/staged
git log --all --diff-filter=A -- .limio.json   # Check if ever committed
```

**Fix:**
1. Add to root `.gitignore`:
   ```
   .limio.json
   ```
2. If already tracked, remove from git (without deleting the file):
   ```bash
   git rm --cached .limio.json
   ```
3. If already pushed, rotate the client secret in the Limio admin panel.

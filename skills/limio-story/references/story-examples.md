# Story Examples

Complete, working story examples for different Limio component types. Use these as templates when creating stories.

---

## Offer Card Story (useCampaign + useBasket)

This example is for a pricing/offer card component that displays subscription offers and an add-to-basket button.

```javascript
import React from "react"
import OfferCards from "../../components/offer-cards"

export default {
  title: "Components/Offer Cards",
  component: OfferCards,
  decorators: [
    (Story) => (
      <div style={{ padding: "2rem", maxWidth: "1200px", margin: "0 auto" }}>
        <Story />
      </div>
    ),
  ],
  argTypes: {
    headline: {
      control: "text",
      description: "Main heading above the offer cards",
    },
    subheadline: {
      control: "text",
      description: "Supporting text below the headline",
    },
    showGroupToggle: {
      control: "boolean",
      description: "Show monthly/annual toggle",
    },
    ctaText: {
      control: "text",
      description: "Button text override (uses offer CTA if empty)",
    },
    primaryColor: {
      control: "color",
      description: "Primary accent color",
    },
    backgroundColor: {
      control: "color",
      description: "Section background color",
    },
    textColor: {
      control: "color",
      description: "Main text color",
    },
    groupLabels: {
      control: "object",
      description: "Group label configuration",
    },
    theme: {
      control: { type: "select" },
      options: ["light", "dark"],
      description: "Color theme",
    },
  },
}

const Template = (args) => <OfferCards {...args} />

// Default — uses 3 offers from the SDK mock (Monthly $9.99, Annual $99.99, Premium $19.99)
export const Default = Template.bind({})
Default.args = {
  headline: "Choose Your Plan",
  subheadline: "Start your free trial today. Cancel anytime.",
  showGroupToggle: false,
  ctaText: "",
  primaryColor: "#635BFF",
  backgroundColor: "#ffffff",
  textColor: "#1a1f36",
  groupLabels: [
    { id: "monthly", label: "Monthly" },
    { id: "annual", label: "Annual" },
  ],
  theme: "light",
}

// Single offer — tests layout with just one card
export const SingleOffer = Template.bind({})
SingleOffer.args = {
  ...Default.args,
  headline: "Our Plan",
  subheadline: "Simple, transparent pricing.",
}
SingleOffer.decorators = [
  (Story) => {
    // Note: The SDK mock always provides 3 offers. To show a single offer,
    // the component itself should accept a maxOffers or offerCount prop,
    // or you can document this as "shows first offer only when layout is single".
    return (
      <div style={{ padding: "2rem", maxWidth: "480px", margin: "0 auto" }}>
        <Story />
      </div>
    )
  },
]

// With group toggle — monthly/annual switcher using groupLabels
export const WithGroupToggle = Template.bind({})
WithGroupToggle.args = {
  ...Default.args,
  headline: "Choose Your Plan",
  subheadline: "Save 17% with annual billing.",
  showGroupToggle: true,
  groupLabels: [
    { id: "monthly", label: "Monthly" },
    { id: "annual", label: "Annual" },
  ],
}

// Dark theme — dark background with light text
export const DarkTheme = Template.bind({})
DarkTheme.args = {
  ...Default.args,
  headline: "Choose Your Plan",
  subheadline: "Start your free trial today.",
  primaryColor: "#7C6AFF",
  backgroundColor: "#0f172a",
  textColor: "#f1f5f9",
  theme: "dark",
}
DarkTheme.decorators = [
  (Story) => (
    <div
      style={{
        padding: "2rem",
        maxWidth: "1200px",
        margin: "0 auto",
        background: "#0f172a",
        minHeight: "100vh",
      }}
    >
      <Story />
    </div>
  ),
]

// Mobile view — narrow viewport to test responsive layout
export const MobileView = Template.bind({})
MobileView.args = {
  ...Default.args,
  headline: "Choose Your Plan",
}
MobileView.parameters = {
  viewport: {
    defaultViewport: "mobile1",
  },
}
```

---

## Subscription Dashboard Story (useUser + useSubscriptions)

This example is for a subscription management component that displays a user's active and past subscriptions.

```javascript
import React from "react"
import SubscriptionDashboard from "../../components/subscription-dashboard"

export default {
  title: "Components/Subscription Dashboard",
  component: SubscriptionDashboard,
  decorators: [
    (Story) => (
      <div style={{ padding: "2rem", maxWidth: "960px", margin: "0 auto" }}>
        <Story />
      </div>
    ),
  ],
  argTypes: {
    headline: {
      control: "text",
      description: "Dashboard heading",
    },
    showPaymentHistory: {
      control: "boolean",
      description: "Show payment schedule section",
    },
    showManageButton: {
      control: "boolean",
      description: "Show manage/cancel subscription button",
    },
    manageButtonText: {
      control: "text",
      description: "Text for the manage button",
    },
    cancelButtonText: {
      control: "text",
      description: "Text for the cancel button",
    },
    emptyStateMessage: {
      control: "text",
      description: "Message shown when user has no subscriptions",
    },
    primaryColor: {
      control: "color",
      description: "Primary accent color",
    },
    dangerColor: {
      control: "color",
      description: "Color for cancel/danger actions",
    },
    activeStatusLabel: {
      control: "text",
      description: "Label for active status badge",
    },
    cancelledStatusLabel: {
      control: "text",
      description: "Label for cancelled status badge",
    },
  },
}

const Template = (args) => <SubscriptionDashboard {...args} />

// Default — SDK mock provides 3 subscriptions:
// - "Pro Plan Monthly" (active, $9.99/mo, ref: REF001)
// - "Enterprise Annual" (active, $499/yr, ref: REF002)
// - "Starter Monthly" (cancelled, $4.99/mo, ref: REF003)
export const Default = Template.bind({})
Default.args = {
  headline: "My Subscriptions",
  showPaymentHistory: true,
  showManageButton: true,
  manageButtonText: "Manage",
  cancelButtonText: "Cancel Subscription",
  emptyStateMessage: "You don't have any subscriptions yet.",
  primaryColor: "#635BFF",
  dangerColor: "#dc2626",
  activeStatusLabel: "Active",
  cancelledStatusLabel: "Cancelled",
}

// Cancelled subscription emphasis — useful for testing the cancelled state UI.
// The SDK mock already includes a cancelled subscription (Starter Monthly, REF003).
// This variation adjusts labels to focus on the cancellation messaging.
export const CancelledSubscription = Template.bind({})
CancelledSubscription.args = {
  ...Default.args,
  headline: "Subscription History",
  cancelledStatusLabel: "Cancelled on Dec 1, 2023",
  showManageButton: false,
}

// No subscriptions — tests the empty state.
// Since the SDK mock always returns subscriptions, the component should handle
// an empty array gracefully. If using useSubscriptions directly, this variation
// documents the expected empty-state behavior.
export const NoSubscriptions = Template.bind({})
NoSubscriptions.args = {
  ...Default.args,
  headline: "My Subscriptions",
  emptyStateMessage: "You don't have any active subscriptions. Browse our plans to get started.",
}
NoSubscriptions.decorators = [
  (Story) => (
    <div style={{ padding: "2rem", maxWidth: "960px", margin: "0 auto" }}>
      {/* Component should show empty state when subscriptions array is empty */}
      <Story />
    </div>
  ),
]

// Loading state — simulates the loading state before user data arrives
export const LoadingState = Template.bind({})
LoadingState.args = {
  ...Default.args,
  headline: "My Subscriptions",
}
LoadingState.decorators = [
  (Story) => (
    <div style={{ padding: "2rem", maxWidth: "960px", margin: "0 auto" }}>
      {/* Component should show skeleton/spinner when loaded is false */}
      <Story />
    </div>
  ),
]

// Payment history focus — expanded payment schedule view
export const WithPaymentHistory = Template.bind({})
WithPaymentHistory.args = {
  ...Default.args,
  headline: "Billing & Subscriptions",
  showPaymentHistory: true,
  showManageButton: true,
}
```

---

## Simple Content Story (no SDK hooks, just limioProps)

This example is for a content/marketing component that only uses limioProps (no SDK hooks like useCampaign or useUser).

```javascript
import React from "react"
import HeroBanner from "../../components/hero-banner"

export default {
  title: "Components/Hero Banner",
  component: HeroBanner,
  decorators: [
    (Story) => (
      <div style={{ maxWidth: "1400px", margin: "0 auto" }}>
        <Story />
      </div>
    ),
  ],
  argTypes: {
    headline: {
      control: "text",
      description: "Main hero heading",
    },
    subheadline: {
      control: "text",
      description: "Supporting text below headline",
    },
    description: {
      control: "text",
      description: "Rich text body content (HTML)",
    },
    ctaText: {
      control: "text",
      description: "Call-to-action button text",
    },
    ctaUrl: {
      control: "text",
      description: "CTA button link URL",
    },
    showCta: {
      control: "boolean",
      description: "Show or hide the CTA button",
    },
    primaryColor: {
      control: "color",
      description: "Primary accent / button color",
    },
    backgroundColor: {
      control: "color",
      description: "Section background color",
    },
    textColor: {
      control: "color",
      description: "Heading and body text color",
    },
    textAlignment: {
      control: { type: "select" },
      options: ["left", "center", "right"],
      description: "Text alignment within the banner",
    },
    paddingSize: {
      control: "number",
      description: "Vertical padding in pixels",
    },
  },
}

const Template = (args) => <HeroBanner {...args} />

// Default — standard hero with typical content
export const Default = Template.bind({})
Default.args = {
  headline: "Stream Without Limits",
  subheadline: "Thousands of movies, shows, and originals. One simple price.",
  description:
    "<p>Join millions of subscribers enjoying premium content on any device. Start your free trial today and cancel anytime.</p>",
  ctaText: "Start Free Trial",
  ctaUrl: "/subscribe",
  showCta: true,
  primaryColor: "#635BFF",
  backgroundColor: "#ffffff",
  textColor: "#1a1f36",
  textAlignment: "center",
  paddingSize: 80,
}

// Long content — tests layout with very long text
export const LongContent = Template.bind({})
LongContent.args = {
  ...Default.args,
  headline:
    "The Ultimate Streaming Experience for the Whole Family with Unlimited Downloads and Premium Features",
  subheadline:
    "Access our complete library of over 50,000 titles including award-winning originals, blockbuster movies, live sports, and exclusive documentaries — all in stunning 4K HDR quality with Dolby Atmos sound on up to 5 devices simultaneously.",
  description:
    "<p>Our premium subscription gives you access to everything in our catalog. Download content for offline viewing, create up to 6 user profiles, and enjoy parental controls for a safe family experience.</p><p>Plus, get early access to new releases and exclusive behind-the-scenes content. No ads, no interruptions — just pure entertainment.</p><ul><li>Unlimited streaming on all devices</li><li>Download for offline viewing</li><li>6 user profiles</li><li>4K HDR + Dolby Atmos</li><li>No ads ever</li></ul>",
  ctaText: "Subscribe Now and Save 20% on Your First Year",
}

// Minimal content — only essential fields, everything else empty/off
export const MinimalContent = Template.bind({})
MinimalContent.args = {
  headline: "Subscribe Today",
  subheadline: "",
  description: "",
  ctaText: "",
  ctaUrl: "",
  showCta: false,
  primaryColor: "#635BFF",
  backgroundColor: "#f8f9fb",
  textColor: "#1a1f36",
  textAlignment: "center",
  paddingSize: 40,
}

// Custom colors — branded color scheme
export const CustomColors = Template.bind({})
CustomColors.args = {
  ...Default.args,
  headline: "Join the Movement",
  subheadline: "Premium content, redefined.",
  primaryColor: "#e11d48",
  backgroundColor: "#1e1b4b",
  textColor: "#e0e7ff",
  ctaText: "Get Started",
}
CustomColors.decorators = [
  (Story) => (
    <div
      style={{
        maxWidth: "1400px",
        margin: "0 auto",
        background: "#1e1b4b",
        minHeight: "60vh",
      }}
    >
      <Story />
    </div>
  ),
]

// Right-aligned — tests text alignment option
export const RightAligned = Template.bind({})
RightAligned.args = {
  ...Default.args,
  textAlignment: "right",
  headline: "Your Entertainment, Your Way",
  subheadline: "Personalized recommendations powered by AI.",
}

// Mobile view — responsive layout at mobile viewport
export const MobileView = Template.bind({})
MobileView.args = {
  ...Default.args,
  paddingSize: 40,
}
MobileView.parameters = {
  viewport: {
    defaultViewport: "mobile1",
  },
}
```

---

## Tool Stories Reference

The Storybook playground also includes two built-in tool stories created by the `limio-storybook` skill. These live under the **Tools** category in the sidebar and are not component stories — they are interactive utilities:

- **`LimioSetup.stories.js`** — A guided onboarding wizard at **Tools > Limio Setup** that handles tenant credentials, region selection, and connection validation. Created during Storybook setup.

- **`NewComponent.stories.js`** — A component builder interface at **Tools > New Component** where users name a component, describe what they want, and submit it to Claude Code for creation. Also created during Storybook setup.

These tool stories are managed by the `limio-storybook` skill and should not be modified by the `limio-story` skill. They are mentioned here only for awareness — if a user asks about them, point them to the `limio-storybook` skill.

---

## Mock Data Reference

The SDK mocks (set up by `limio-storybook`) provide the following data by default. Stories do not need to recreate this data — it is available automatically via the SDK hooks.

### Offers (from useCampaign)

Three offers are available:

| Offer | display_name__limio | price | group__limio | best_value__limio |
|-------|-------------------|-------|-------------|------------------|
| Monthly Plan | "Monthly" | $9.99/mo | "monthly" | false |
| Annual Plan | "Annual" | $99.99/yr | "annual" | true (badge: "Best Value") |
| Premium Monthly | "Premium" | $19.99/mo | "monthly" | false |

Each offer includes full attributes: `display_price__limio` (HTML), `detailed_display_price__limio` (HTML), `cta_text__limio`, `offer_features__limio` (HTML list), `payment_types__limio`, `checkout_description__limio`, `price__limio` (array with `{ type, value, currencyCode }`), and `term__limio` (with `type`, `length`, `renewal_trigger`, `renewal_type`).

### User (from useUser)

```
username: "mock-user-001"
email: "alex@example.com"
firstName: "Alex"
lastName: "Johnson"
loginStatus: "logged-in"
loaded: true
```

### Subscriptions (from useSubscriptions)

Three subscriptions are available:

| Subscription | Status | Price | Reference | Created |
|-------------|--------|-------|-----------|---------|
| Pro Plan Monthly | active | $9.99/mo | REF001 | 2024-01-15 |
| Enterprise Annual | active | $499/yr | REF002 | 2024-03-15 |
| Starter Monthly | cancelled | $4.99/mo | REF003 | 2023-06-01 |

Each subscription includes an `offers` array (with `data.start`, `data.end`, `data.record_subtype`, and nested offer attributes) and a `schedule` array (with payment dates, amounts, and statuses).

### Basket (from useBasket)

One basket item by default:
- Monthly Plan offer, $9.99 USD
- `basketLoading: false`
- All basket methods (`addOfferToBasket`, `removeFromBasket`, `navigateToCheckout`, etc.) log to console

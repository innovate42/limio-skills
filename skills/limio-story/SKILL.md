---
name: limio-story
description: Creates Storybook stories for Limio components with meaningful variations and realistic mock data. Use when user asks to "create a story", "add storybook stories", "write stories for component", "story for component", "create variations", mentions ".stories.js", or discusses story variations for a Limio component. Both developers and non-technical staff can describe desired variations. Do NOT use for setting up Storybook (use limio-storybook) or creating components (use limio-component).
metadata:
  author: Limio
  version: 8.0.0
---

# Limio Story Creation

This skill creates Storybook stories for Limio components with meaningful variations and realistic mock data. It is designed for both **developers** and **non-technical staff** — anyone can describe the variations they want and Claude will translate them into proper story files.

## Story Creation Workflow

1. **Read the component's `package.json`** to get the `limioProps` array — this defines all configurable props, their types, and defaults
2. **Read the component's `index.js`** to understand what the component renders and how it uses props
3. **Determine which SDK hooks the component uses** — look for `useCampaign`, `useBasket`, `useUser`, `useSubscriptions`, `useCheckout`, `useLimioContext`, etc.
4. **Create story file** at `component-playground/src/stories/<ComponentName>.stories.js` using PascalCase naming
5. **Map limioProps to Storybook args** with appropriate controls (see Args Mapping Rules below)
6. **Create 3-5 meaningful variations** based on component type (see Variation Patterns below)
7. If Storybook is running, stories appear immediately via hot reload — no restart needed

## Story Template

```javascript
import React from "react"
import Component from "../../components/component-name"

export default {
  title: "Components/Component Name",
  component: Component,
  decorators: [
    (Story) => (
      <div style={{ padding: "2rem", maxWidth: "1200px", margin: "0 auto" }}>
        <Story />
      </div>
    ),
  ],
  argTypes: {
    // Map from limioProps — see rules below
  },
}

const Template = (args) => <Component {...args} />

export const Default = Template.bind({})
Default.args = {
  // Default values from limioProps
}
```

## Args Mapping Rules

Map each limioProp type to its corresponding Storybook control:

| limioProps type | Storybook argType | Notes |
|-----------------|-------------------|-------|
| `string` | `{ control: "text" }` | Plain text input |
| `boolean` | `{ control: "boolean" }` | Toggle switch |
| `number` | `{ control: "number" }` | Numeric input |
| `richText` | `{ control: "text" }` | HTML string — user edits raw HTML |
| `color` | `{ control: "color" }` | Color picker |
| `datetime` | `{ control: "text" }` | ISO date string |
| `picklist` | `{ control: { type: "select" }, options: [...values] }` | Dropdown from picklist options |
| `list` | `{ control: "object" }` | JSON editor for array of objects |

### Example Mapping

Given this limioProp:
```json
{
  "id": "theme",
  "label": "Theme",
  "type": "picklist",
  "options": [
    { "id": "light", "label": "Light", "value": "light" },
    { "id": "dark", "label": "Dark", "value": "dark" }
  ],
  "default": "light"
}
```

The argType becomes:
```javascript
argTypes: {
  theme: {
    control: { type: "select" },
    options: ["light", "dark"],
    description: "Theme",
    defaultValue: "light",
  },
}
```

## Non-Technical Users

Non-technical staff can describe variations in plain language. Claude will translate these into proper story variations:

- **"Show me what it looks like with 1 offer, 3 offers, and 6 offers"** — Creates SingleOffer, ThreeOffers, and SixOffers variations with different mock offer arrays
- **"Create a dark theme version and a light theme version"** — Creates LightTheme and DarkTheme variations with appropriate color props
- **"Show a version with a really long headline"** — Creates LongHeadline variation with edge-case text
- **"Make a mobile-sized preview"** — Creates MobileView variation using Storybook viewport parameters
- **"What does it look like empty?"** — Creates EmptyState variation with no data
- **"Show the loading state"** — Creates LoadingState variation if the component supports it

Just describe what you want to see — no code knowledge required.

## Variation Patterns (by component type)

Choose variations that make sense for the component being documented:

### Offer / Pricing Cards
- **Default** — 3 offers (from SDK mock)
- **SingleOffer** — Just one offer
- **MultipleOffers** — 5-6 offers to test grid layout
- **WithGroupToggle** — Grouped offers with monthly/annual toggle
- **WithBadge** — Offer with `best_value__limio` and badge text
- **Promotional** — Sale pricing with strikethrough
- **DarkTheme** — Dark background, light text
- **MobileView** — Narrow viewport via `parameters.viewport`

### Subscription Management
- **Default** — Active subscriptions (from SDK mock: 2 active, 1 cancelled)
- **ActiveSubscription** — Single active subscription
- **CancelledSubscription** — Subscription with `status: "cancelled"`
- **MultipleSubscriptions** — Several subscriptions in different states
- **WithPendingChanges** — Subscription with pending schedule changes
- **NoSubscriptions** — Empty subscriptions array
- **LoadingState** — Simulated loading state

### User Account
- **LoggedIn** — Full user profile with attributes
- **LoggedOut** — `loginStatus` not `"logged-in"`, no user data
- **LoadingState** — `loaded: false`

### Content / Marketing
- **Default** — Standard content with typical lengths
- **LongContent** — Very long headlines and body text
- **MinimalContent** — Minimal/empty optional fields
- **WithImages** — All image slots filled
- **CustomColors** — Non-default color props
- **DarkTheme** — Dark background variation

### General (any component)
- **Default** — Uses limioProp defaults exactly
- **Minimal** — Only required props, everything else empty/off
- **FullyLoaded** — Every prop filled with realistic data
- **EdgeCases** — Empty strings, very long strings, special characters
- **MobileView** — Using `parameters: { viewport: { defaultViewport: "mobile1" } }`

## Important Rules

1. **Story file location:** Always `component-playground/src/stories/<ComponentName>.stories.js`
2. **PascalCase file names:** `PricingCards.stories.js`, not `pricing-cards.stories.js`
3. **Descriptive variation names:** Use meaningful names like `SingleOffer`, `DarkTheme`, `CancelledSubscription` — never `Variation1`, `Variation2`
4. **Always include a Default story:** The Default story should use the exact limioProp defaults from `package.json`
5. **Component import path:** `../../components/component-name` (from `src/stories/` up to `component-playground/`, then into `components/`)
6. **Mock data from SDK:** For components using `useCampaign`, the mock provides 3 offers by default. For components using `useUser`, the mock provides a logged-in user with 3 subscriptions (2 active, 1 cancelled). These mocks are already wired up — no additional mock setup needed in the story file.
7. **Props pass through `useStaticProps`:** The component reads args via the `ComponentContext` provider. Storybook args are passed as props to the component, which merges them with defaults in `useStaticProps`.
8. **Rich text values:** Use HTML strings for richText props (e.g., `"<h1>Welcome</h1><p>Get started today</p>"`)
9. **List values:** Use arrays of `{ id, label }` objects for list props
10. **Viewport parameters:** For mobile variations, use `parameters: { viewport: { defaultViewport: "mobile1" } }`

## Full Examples

See `references/story-examples.md` for complete, working story examples for different component types:
- Offer card story (useCampaign + useBasket)
- Subscription dashboard story (useUser + useSubscriptions)
- Simple content story (no SDK hooks, just limioProps)

## Related Skills

- `limio-component` — Create the components themselves
- `limio-storybook` — Set up and launch the Storybook playground
- `limio-setup` — Credentials and deployment

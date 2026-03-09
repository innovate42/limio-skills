# Limio Skills for Claude Code

A Claude Code plugin providing a suite of skills for building, verifying, and deploying custom components on the [Limio](https://limio.com) subscription management platform.

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add innovate42/limio-skills
/plugin install limio-skills
```

## Available Skills

### `limio-component` — Create Components

**Audience:** Developers

Creates Limio custom React components following official SDK guidelines and best practices.

**Triggers when you:**
- Ask to "create a Limio component" or "build a subscription component"
- Mention `@limio/sdk`, `useCampaign`, `useBasket`, `useUser`, `limioProps`
- Discuss building React components for the Limio platform

**What it provides:**
- Component file structure (`package.json`, `index.js`, `componentStaticProps.js`, `index.css`)
- Full `@limio/sdk` hook reference
- limioProps configuration for all field types
- Add-to-basket patterns, CSS best practices, Page Builder compatibility
- Component template with best practices built in

---

### `limio-sdk-verify` — Audit SDK Usage

**Audience:** Developers

Audits existing Limio component code against official SDK documentation.

**Triggers when you:**
- Ask to "verify SDK usage", "audit Limio code", "check SDK compliance"
- Ask "is this using the SDK correctly" or "review Limio imports"
- Review existing component code for correctness

**What it provides:**
- Comprehensive verification checklist (imports, hooks, data access, security)
- Deprecated method detection with correct replacements
- Common anti-pattern identification with fixes
- Live doc verification via MCP when available

---

### `limio-story` — Create Stories

**Audience:** Developers + Non-technical staff

Creates Storybook stories for Limio components with meaningful variations.

**Triggers when you:**
- Ask to "create a story", "add storybook stories", "story for component"
- Mention `.stories.js` or discuss story variations

**What it provides:**
- Story template with LimioProvider and ComponentContext decorators
- Automatic args mapping from `limioProps` in `package.json`
- 3-5 variation patterns per component type
- Non-technical users can describe variations in plain language

---

### `limio-storybook` — Set Up Storybook Playground

**Audience:** Developers

Sets up and launches the full Storybook component playground with mocked SDK.

**Triggers when you:**
- Ask to "set up storybook", "launch storybook", "start storybook", "run storybook"
- Discuss the `component-playground` environment

**What it provides:**
- Full `component-playground/` directory setup with Storybook 8
- Mocked `@limio/sdk` hooks with realistic sample data
- **Claude Prompt addon** — type prompts in Storybook, Claude auto-applies changes
- **Claude Overlay** — animated loading/deploy status screens
- **New Component builder** — create components from the Storybook UI
- Addon version checking and auto-update
- Prompt watcher feedback loop

---

### `limio-setup` — Connect & Deploy

**Audience:** Developers + Non-technical staff

Connects to a Limio tenant, manages credentials, and deploys components.

**Triggers when you:**
- Ask to "set up limio", "configure limio", "connect to limio"
- Ask to "deploy component" or "push to limio"
- Discuss Limio credentials or deployment

**What it provides:**
- Guided credential setup (via Storybook wizard or manual)
- Smart git deployment with conflict resolution
- Build status tracking from Limio API
- Region support: EU, US, Dev

---

## Quick Start

### Set up Limio
```
set up limio
```
Bootstraps the Storybook playground, starts the dev server, and opens the setup wizard.

### Build a component
```
Create a pricing card component that displays offers with a monthly/annual toggle
```
Creates the component, sets up Storybook (if needed), generates stories, and launches the dev environment.

### Verify SDK usage
```
Verify this component follows Limio SDK best practices
```
Audits your code against official documentation and reports issues.

### Create stories
```
Create storybook stories for the pricing-cards component with dark theme and mobile variations
```
Generates story files with meaningful variations based on the component's limioProps.

### Deploy
```
Deploy the pricing-cards component to Limio
```
Commits, pushes, and tracks the Limio build.

## Supported Regions

| Region | Domain |
|--------|--------|
| EU (default) | `{tenant}.prod.limio.com` |
| US | `{tenant}.prod-us.limio.com` |
| Dev | `{tenant}.dev.limio.com` |

## Documentation

- [Limio Custom Components Docs](https://docs.limio.com/developers/custom-components/custom-components)
- [@limio/sdk Reference](https://docs.limio.com/developers/limio-sdk/getting-started)
- [Development Guidelines](https://docs.limio.com/developers/custom-components/development-guidelines)

## License

MIT

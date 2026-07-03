# Limio Skills for Claude Code

A Claude Code plugin that gives Claude **six specialised skills** for the [Limio](https://limio.com) subscription management platform — from creating components to assembling them into live pages on your tenant.

| Skill | What it does | Audience |
|-------|-------------|----------|
| [`limio-component`](#limio-component--create-components) | Creates custom React components with the `@limio/sdk` | Developers |
| [`limio-sdk-verify`](#limio-sdk-verify--audit-sdk-usage) | Audits existing code against SDK best practices | Developers |
| [`limio-story`](#limio-story--create-stories) | Generates Storybook stories with meaningful variations | Developers + Non-technical |
| [`limio-storybook`](#limio-storybook--set-up-storybook-playground) | Sets up the full Storybook component playground | Developers |
| [`limio-setup`](#limio-setup--connect--deploy) | Connects to a Limio tenant, manages credentials, and deploys | Developers + Non-technical |
| `limio-pages` | Assembles components into pages via the API — landing-page factories, publishing, and site migrations | Developers |

All six skills are installed together as a single plugin. Each one activates automatically based on what you ask Claude to do — there's nothing extra to configure or enable per-skill.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- A Limio tenant (for deployment — not required for local development)

## Installation

### First-time setup

Two steps — add the marketplace, then install the plugin:

```shell
/plugin marketplace add innovate42/limio-skills
```

```shell
/plugin install limio-skills@limio-skills
```

After installing, run `/reload-plugins` to activate. All five skills start working immediately — just ask Claude to do something Limio-related and the right skill kicks in.

### Updating to the latest version

If you already have the plugin installed and want to pull the latest changes:

```shell
/plugin marketplace update limio-skills
```

This fetches the latest version from the `production` branch. Run `/reload-plugins` afterwards to pick up the changes.

You can also enable auto-updates so the plugin stays current automatically — open `/plugin`, go to the **Marketplaces** tab, select `limio-skills`, and choose **Enable auto-update**.

### Verifying installation

Run `/plugin` and check the **Installed** tab. You should see `limio-skills` listed with all five skills.

### Uninstalling

```shell
/plugin uninstall limio-skills@limio-skills
```

## Available Skills

### `limio-component` — Create Components

**Audience:** Developers

Creates Limio custom React components following official SDK guidelines and best practices.

**Example prompts:**
```
Create a pricing card component that shows offers with a monthly/annual toggle
```
```
Build a subscription management component using useCampaign and useBasket
```

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

**Example prompts:**
```
Verify this component follows Limio SDK best practices
```
```
Audit the pricing-cards component for deprecated SDK usage
```
```
Is my component using the SDK correctly?
```

**What it provides:**
- Comprehensive verification checklist (imports, hooks, data access, security)
- Deprecated method detection with correct replacements
- Common anti-pattern identification with fixes
- Live doc verification via MCP when available

---

### `limio-story` — Create Stories

**Audience:** Developers + Non-technical staff

Creates Storybook stories for Limio components with meaningful variations.

**Example prompts:**
```
Create storybook stories for the pricing-cards component
```
```
Add stories with dark theme and mobile variations
```
```
I want a story showing what the card looks like with a long title and no discount
```

**What it provides:**
- Story template with LimioProvider and ComponentContext decorators
- Automatic args mapping from `limioProps` in `package.json`
- 3-5 variation patterns per component type
- Non-technical users can describe variations in plain language

---

### `limio-storybook` — Set Up Storybook Playground

**Audience:** Developers

Sets up and launches the full Storybook component playground with mocked SDK.

**Example prompts:**
```
Set up storybook
```
```
Launch the storybook playground
```
```
Start storybook
```

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

**Example prompts:**
```
Set up limio
```
```
Connect to my Limio tenant
```
```
Deploy the pricing-cards component to Limio
```

**What it provides:**
- Guided credential setup (via Storybook wizard or manual)
- Smart git deployment with conflict resolution
- Build status tracking from Limio API
- Region support: EU, US, Dev

---

## Quick Start

### 1. Set up Limio
```
set up limio
```
Bootstraps the Storybook playground, starts the dev server, and opens the setup wizard.

### 2. Build a component
```
Create a pricing card component that displays offers with a monthly/annual toggle
```
Creates the component, sets up Storybook (if needed), generates stories, and launches the dev environment.

### 3. Verify SDK usage
```
Verify this component follows Limio SDK best practices
```
Audits your code against official documentation and reports issues.

### 4. Create stories
```
Create storybook stories for the pricing-cards component with dark theme and mobile variations
```
Generates story files with meaningful variations based on the component's limioProps.

### 5. Deploy
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

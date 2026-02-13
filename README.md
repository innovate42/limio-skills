# Limio Skills for Claude Code

A Claude Code plugin providing skills for building custom components on the [Limio](https://limio.com) subscription management platform.

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add innovate42/limio-skills
/plugin install limio-skills
```

## Available Skills

### `limio-component`

Helps you build custom React components for Limio's Page Builder with full SDK integration.

**Triggers automatically when you:**
- Ask to "create a Limio component"
- Mention `@limio/sdk`, `useCampaign`, `useBasket`, `useUser`
- Discuss building subscription/offer components
- Reference `limioProps`

**What it provides:**
- Component file structure and boilerplate
- Full `@limio/sdk` hook reference (useCampaign, useBasket, useUser, etc.)
- limioProps configuration for all field types
- Add-to-basket patterns
- Page Builder compatibility guidance
- Common utilities and best practices
- **Claude Prompt panel** — a Storybook addon that lets you type prompts directly in the Storybook UI and Claude Code automatically picks them up, applies changes, and Storybook hot-reloads

## Example Usage

```
Create a pricing card component that displays offers from the campaign
with a monthly/annual toggle and add to basket functionality
```

The skill will guide Claude to:
1. Create the component files:
   - `package.json` with limioProps config
   - `index.js` React component using SDK hooks
   - `componentStaticProps.js` for props handling
   - `index.css` with styling
2. Set up a Storybook playground (if not already configured) with mocked `@limio/sdk` hooks
3. Set up the Claude Prompt addon (if not already configured) for interactive prompting from Storybook
4. Generate a story with multiple variations based on the component's limioProps
5. Launch Storybook and the prompt watcher so you can preview and iterate interactively

## Storybook Playground

The skill automatically sets up a `component-playground/` directory with:
- Storybook 8 configured with webpack aliases to mock `@limio/sdk`
- Full mock implementations of all SDK hooks (`useCampaign`, `useBasket`, `useUser`, `useCheckout`, etc.) with realistic sample data
- Stories for each component with multiple variations
- **Claude Prompt addon** — a panel in Storybook where you can type change requests that Claude Code processes automatically

On subsequent runs, the existing playground is reused and only new stories are added.

## Claude Prompt Addon

The Claude Prompt addon adds an interactive feedback loop between Storybook and Claude Code:

1. Open the **Claude Prompt** panel in Storybook's addon tabs
2. Type a change request (e.g., "Make the header taller and change the gradient to blue")
3. Click **Send to Claude** (or press Cmd+Enter)
4. Claude Code automatically detects the prompt, reads the target component files, applies the changes, and Storybook hot-reloads

The addon auto-detects which component you're viewing and includes status feedback (Working, Completed, Error) so you know what Claude Code is doing. The prompt watcher runs as a background task alongside Storybook.

## Documentation

- [Limio Custom Components Docs](https://docs.limio.com/developers/custom-components/custom-components)
- [@limio/sdk Reference](https://docs.limio.com/developers/limio-sdk/getting-started)

## License

MIT

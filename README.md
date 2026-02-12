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
3. Generate a story with multiple variations based on the component's limioProps
4. Launch Storybook so you can preview and iterate on the component immediately

## Storybook Playground

The skill automatically sets up a `component-playground/` directory with:
- Storybook 8 configured with webpack aliases to mock `@limio/sdk`
- Full mock implementations of all SDK hooks (`useCampaign`, `useBasket`, `useUser`, `useCheckout`, etc.) with realistic sample data
- Stories for each component with multiple variations

On subsequent runs, the existing playground is reused and only new stories are added.

## Documentation

- [Limio Custom Components Docs](https://docs.limio.com/developers/custom-components/custom-components)
- [@limio/sdk Reference](https://docs.limio.com/developers/limio-sdk/getting-started)

## License

MIT

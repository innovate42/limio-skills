# Limio Skills for Claude Code

A Claude Code plugin providing skills for building custom components on the [Limio](https://limio.com) subscription management platform.

## Installation

Add the marketplace and install the plugin:

```
/plugin install limio/limio-skills
```

Or add via marketplace:

```
/plugin marketplace add limio/limio-skills
/plugin install limio-skills@limio/limio-skills
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

The skill will guide Claude to create:
- `package.json` with limioProps config
- `index.js` React component using SDK hooks
- `componentStaticProps.js` for props handling
- `index.css` with styling

## Documentation

- [Limio Custom Components Docs](https://docs.limio.com/developers/custom-components/custom-components)
- [@limio/sdk Reference](https://docs.limio.com/developers/sdk)

## License

MIT

# Storybook Configuration Templates

These are the core configuration files for the component-playground Storybook setup.

## component-playground/package.json

```json
{
  "name": "@limio/component-playground",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "ramda": "^0.28.0",
    "xss": "^1.0.15"
  },
  "scripts": {
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build"
  },
  "devDependencies": {
    "@storybook/addon-essentials": "^8.0.0",
    "@storybook/addon-interactions": "^8.0.0",
    "@storybook/addon-links": "^8.0.0",
    "@storybook/addon-webpack5-compiler-babel": "^1.0.0",
    "@storybook/blocks": "^8.0.0",
    "@storybook/react": "^8.0.0",
    "@storybook/react-webpack5": "^8.0.0",
    "storybook": "^8.0.0"
  }
}
```

## component-playground/.storybook/main.js

```javascript
import path, { dirname, join } from "path"

function getAbsolutePath(value) {
    return dirname(require.resolve(join(value, "package.json")))
}

const config = {
    stories: ["../src/**/*.stories.@(js|jsx|ts|tsx)"],
    addons: [
        getAbsolutePath("@storybook/addon-webpack5-compiler-babel"),
        getAbsolutePath("@storybook/addon-essentials"),
        getAbsolutePath("@storybook/addon-interactions"),
        path.resolve(__dirname, "addon-prompt"),
    ],
    framework: {
        name: getAbsolutePath("@storybook/react-webpack5"),
        options: {},
    },
    webpackFinal: async (config) => {
        config.resolve.alias = {
            ...config.resolve.alias,
            "@limio/sdk": path.resolve(__dirname, "..", "packages", "limio", "sdk"),
            "@limio/sdk/components": path.resolve(__dirname, "..", "packages", "limio", "sdk", "src", "components"),
            "@limio/shop": path.resolve(__dirname, "..", "packages", "limio", "shop"),
            "@limio/internal-checkout-sdk": path.resolve(__dirname, "..", "packages", "limio", "internal-checkout-sdk"),
        }
        return config
    }
}

export default config
```

## component-playground/.storybook/preview.js

```javascript
import React from "react"
import { ClaudeOverlay } from "./claude-overlay"

const preview = {
    parameters: {
        layout: "fullscreen",
        controls: {
            matchers: {
                color: /(background|color)$/i,
                date: /Date$/i,
            },
        },
    },
    decorators: [
        (Story) => (
            <>
                <Story />
                <ClaudeOverlay />
            </>
        ),
    ],
}

export default preview
```

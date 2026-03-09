---
name: limio-storybook
description: Sets up and launches the Limio Storybook component playground with mocked SDK, Claude Prompt addon, and interactive development workflow. Use when user asks to "set up storybook", "launch storybook", "start storybook", "run storybook", "open storybook", "configure storybook playground", or discusses the component-playground environment. Do NOT use for creating components (use limio-component) or creating stories (use limio-story).
metadata:
  author: Limio
  version: 8.0.0
---

# Limio Storybook Playground

The component-playground provides a Storybook environment with mocked Limio SDK hooks, a Claude Prompt addon for interactive development, and deployment capabilities.

## Setup Workflow

When setting up Storybook from scratch (no `component-playground/.storybook/main.js` exists):

1. **Credential safety check:** Verify `.limio.json` is in the root `.gitignore`
2. **Create the full playground structure** — see `references/storybook-configs.md` for all file templates
3. **Create SDK mocks** — see `references/sdk-mocks.md` for mock implementations
4. **Create Claude Overlay** — see `references/claude-overlay.md` for the overlay component
5. **Create Claude Prompt Addon** — see `references/addon-prompt.md` for manager.js, preset.js, middleware.js
6. **Create tool stories** — see `references/tool-stories.md` for LimioSetup and NewComponent stories
7. **Create prompt watcher** — see `references/watch-prompts.md` for the watcher script
8. **Install dependencies:** `cd component-playground && npm install`
9. **Start Storybook:** `cd component-playground && npx storybook dev -p 6006` (background)
10. **Start prompt watcher:** `node component-playground/scripts/watch-prompts.js` (background)

## Launch Workflow

When Storybook already exists (quick-launch):

1. **Credential safety check:** Verify `.limio.json` is in root `.gitignore`
2. **Check addon is up to date** — see "Addon Version Checking" below
3. **Install dependencies if needed:** `cd component-playground && npm install`
4. **Start Storybook:** `cd component-playground && npx storybook dev -p 6006` (background)
5. **Start prompt watcher:** `node component-playground/scripts/watch-prompts.js` (background)
6. **Tell the user:** "Storybook is running at http://localhost:6006"

## Directory Structure

```
component-playground/
├── .storybook/
│   ├── main.js              # Webpack config with SDK aliases
│   ├── preview.js            # Decorators with ClaudeOverlay
│   ├── middleware.js          # Express API endpoints
│   ├── claude-overlay.js      # Loading/deploy overlay components
│   └── addon-prompt/
│       ├── manager.js         # Claude Prompt + Settings panels
│       └── preset.js          # Addon registration
├── packages/limio/            # Mocked SDK packages
│   ├── sdk/
│   ├── shop/
│   └── internal-checkout-sdk/
├── scripts/
│   └── watch-prompts.js       # File watcher for prompt loop
├── src/stories/               # Story files
│   ├── LimioSetup.stories.js
│   └── NewComponent.stories.js
└── package.json
```

## Addon Version Checking

When the addon already exists, check these markers to determine if it needs updating:

| Feature | File | Marker |
|---------|------|--------|
| Settings panel | manager.js | `SETTINGS_PANEL_ID` |
| Deploy endpoint | middleware.js | `/api/deploy` |
| Limio config | middleware.js | `readLimioConfig` |
| Build status | middleware.js | `/api/build-status` |
| Loading overlay | claude-overlay.js | `ClaudeOverlay` |
| New component tool | NewComponent.stories.js | `Tools/New Component` |
| Overlay decorator | preview.js | `ClaudeOverlay` |
| Deploy overlay | claude-overlay.js | `DeployOverlay` |
| Deploy status | middleware.js | `/api/deploy-overlay` |
| Smart deploy | middleware.js | `smart-deploy` |
| Credential safety | root .gitignore | `.limio.json` |

If **any** marker is missing, replace both `manager.js` and `middleware.js` with current templates.

## Prompt Feedback Loop

After starting Storybook and the watcher:

1. User types prompt in Storybook's "Claude Prompt" panel → clicks "Send to Claude"
2. Express middleware writes to `.prompt.json`
3. Watcher script detects change → exits
4. Claude Code reads prompt, applies changes
5. Update status: `node component-playground/scripts/update-prompt-status.js working "Applying changes..."`
6. Apply changes to component
7. Update status: `node component-playground/scripts/update-prompt-status.js completed "Changes applied"`
8. Restart watcher for next prompt

### Status States
| State | Meaning |
|-------|---------|
| `listening` | Ready for prompts |
| `queued` | Prompt saved, waiting |
| `received` | Watcher picked up prompt |
| `working` | Claude making changes |
| `completed` | Changes applied |
| `error` | Something went wrong |

## .gitignore Updates

Root `.gitignore` — add: `.limio.json`
`component-playground/.gitignore` — add: `.prompt.json`, `.prompt-status.json`, `.deploy-status.json`

## Related Skills
- `limio-component` — Create components
- `limio-story` — Create stories
- `limio-setup` — Credentials and deployment

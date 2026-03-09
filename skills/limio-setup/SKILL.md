---
name: limio-setup
description: Connects to a Limio tenant, manages credentials, and deploys components to Limio. Use when user asks to "set up limio", "limio setup", "configure limio", "connect to limio", "deploy component", "push to limio", "limio credentials", "limio deployment", or discusses connecting to a Limio environment or deploying custom components. Suitable for both developers and non-technical staff.
metadata:
  author: Limio
  version: 8.0.0
---

# Limio Setup & Deployment

Connect to your Limio tenant, manage credentials, and deploy custom components.

## CRITICAL — Credential Safety

`.limio.json` contains OAuth client credentials (client ID + client secret). It MUST be in `.gitignore` and MUST NEVER be committed to git. When committing or staging files, NEVER include `.limio.json`. Always verify it is in the **root** `.gitignore` before proceeding with any workflow.

## Limio Setup Workflow

When the user asks to "set up Limio", "configure Limio", or "connect to Limio":

1. **Credential safety check:** Verify `.limio.json` is in the **root** `.gitignore`. If missing, add it immediately.
2. **Check for Storybook:** Look for `component-playground/.storybook/main.js`
3. **If no Storybook exists:** Tell the user to run the `limio-storybook` skill first, or set it up now (the setup wizard lives inside Storybook)
4. **Start Storybook** if not running: `cd component-playground && npx storybook dev -p 6006`
5. **Tell the user:** "Storybook is running at http://localhost:6006 — open **Tools > Limio Setup** in the sidebar to connect your Limio account."
6. The Storybook wizard will guide them through:
   - Entering tenant name
   - Selecting region (EU / US / Dev)
   - Entering client ID and client secret
   - Validating the connection
7. Credentials are saved to `.limio.json` at project root

### For Non-Technical Users

You can set up Limio by simply saying "set up Limio" or "connect to Limio". Claude will:
1. Start the development environment for you
2. Open a setup wizard in your browser
3. Guide you through entering your Limio credentials
4. Validate the connection

You'll need your Limio **tenant name**, **client ID**, and **client secret** from your Limio admin.

## Manual Credential Setup

If you prefer to set up credentials without Storybook:

Create `.limio.json` in the project root:
```json
{
  "tenant": "your-tenant-name",
  "region": "eu",
  "clientId": "your-client-id",
  "clientSecret": "your-client-secret"
}
```

Region options: `"eu"` (Europe), `"us"` (United States), `"dev"` (Development)

URL mapping:
- EU: `https://<tenant>.prod.limio.com`
- US: `https://<tenant>.prod-us.limio.com`
- Dev: `https://<tenant>.dev.limio.com`

**Then immediately verify** `.limio.json` is in `.gitignore`.

## Deploy Workflow

Components are deployed by committing and pushing to the git remote. Limio's CI/CD automatically builds components from the repository.

### Quick Deploy (via Storybook)

1. Click the **Deploy** button in the Storybook "Claude Prompt" panel
2. The middleware handles: git add → commit → push (with smart conflict resolution)
3. A rocket animation shows deployment progress
4. Build status is polled from Limio's API
5. On success, a "Open Page Builder" link appears

### Manual Deploy

For detailed manual deploy steps, see `references/deploy-workflow.md`.

### Deploy Flow Summary

1. Validate component folder exists in `components/`
2. Fetch remote, check ahead/behind
3. If behind: stash → pull --rebase → pop
4. Stage component folder + related story files
5. Commit: `"Deploy component: <name>"`
6. Push (with retry on failure)
7. Poll Limio build API for status

### Build Status

After pushing, the Limio platform automatically triggers a component build. The Storybook middleware proxies build status from `<limio-base-url>/api/component/builds`.

Build statuses: `IN_PROGRESS`, `SUCCEEDED`, `FAILED`, `ERROR`

## Troubleshooting

### Connection Failed
- Verify tenant name is correct (just the name, not the full URL)
- Check region matches your Limio environment
- Ensure client ID and secret are for the correct tenant
- Try refreshing: delete `.limio.json` and re-enter credentials

### Deploy Failed
- Check git status: `git status`
- Resolve any merge conflicts
- Ensure you have push access to the remote
- Check if the branch has an upstream: `git push -u origin <branch>`

### Build Failed
- Check Limio build logs in the admin panel
- Common issues: missing dependencies, import errors, syntax errors
- Verify component follows Limio SDK guidelines (use `limio-sdk-verify` skill)

## Related Skills
- `limio-component` — Create components
- `limio-story` — Create stories
- `limio-storybook` — Set up Storybook playground
- `limio-sdk-verify` — Verify SDK usage

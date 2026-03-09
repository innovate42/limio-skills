# Prompt Watcher

The prompt watcher script monitors `.prompt.json` for new prompts submitted from the Storybook Claude Prompt panel and exits when one is detected, allowing Claude Code to pick it up.

## scripts/watch-prompts.js

```javascript
const fs = require("fs")
const path = require("path")

const PROMPT_FILE = path.resolve(__dirname, "..", ".prompt.json")

if (!fs.existsSync(PROMPT_FILE)) {
    fs.writeFileSync(PROMPT_FILE, JSON.stringify({ prompt: "", component: "", storyId: "", timestamp: "" }, null, 2))
}

const initialContent = fs.readFileSync(PROMPT_FILE, "utf8")
const initialTimestamp = JSON.parse(initialContent).timestamp || ""

console.log("Watching for prompts from Storybook...")
console.log(`Prompt file: ${PROMPT_FILE}`)

const check = () => {
    try {
        const content = fs.readFileSync(PROMPT_FILE, "utf8")
        const data = JSON.parse(content)
        if (data.timestamp && data.timestamp !== initialTimestamp && data.prompt) {
            console.log("\n===PROMPT_RECEIVED===")
            console.log(JSON.stringify(data, null, 2))
            console.log("===END_PROMPT===")
            process.exit(0)
        }
    } catch {}
}

const interval = setInterval(check, 500)
process.on("SIGINT", () => { clearInterval(interval); process.exit(0) })
process.on("SIGTERM", () => { clearInterval(interval); process.exit(0) })
```

## Prompt Watcher Workflow

After starting Storybook, start the prompt watcher as a **background task**:

```bash
node component-playground/scripts/watch-prompts.js
```

When the watcher exits (a prompt was received from the Storybook panel):

1. **Update status to "working":**
   ```bash
   node component-playground/scripts/update-prompt-status.js working "Reading component files..."
   ```
2. **Read the prompt file:** `component-playground/.prompt.json`
3. **Parse the JSON** — it contains `{ prompt, component, storyId, mode, timestamp }`
4. **Read the target component files:** `components/<component>/index.js`, `index.css`, `package.json`
5. **Update status as you work:**
   ```bash
   node component-playground/scripts/update-prompt-status.js working "Applying changes to <component>..."
   ```
6. **Apply the requested changes** to the component based on the prompt
7. **Update status to "completed":**
   ```bash
   node component-playground/scripts/update-prompt-status.js completed "Changes applied — check Storybook"
   ```
8. **Restart the watcher** as a new background task to listen for the next prompt

### Error Handling

If you need user input or permission:
```bash
node component-playground/scripts/update-prompt-status.js permission_needed "Need approval to modify package.json dependencies"
```

If something goes wrong:
```bash
node component-playground/scripts/update-prompt-status.js error "Could not find component 'foo'"
```

## Status Update Script Usage

```bash
node component-playground/scripts/update-prompt-status.js <state> <message>
```

The `update-prompt-status.js` script is a simple utility that writes a status JSON file read by the Storybook overlay. It accepts two arguments: the state name and a human-readable message.

## Prompt File Format (.prompt.json)

```json
{
    "prompt": "Make the hero section taller and change the gradient to blue-to-green",
    "component": "win-back",
    "storyId": "win-back--default",
    "mode": "edit",
    "timestamp": "2025-01-15T10:30:00.000Z"
}
```

Fields:
- `prompt` — The user's natural language request
- `component` — The target component folder name (kebab-case)
- `storyId` — The Storybook story ID that was active when the prompt was sent
- `mode` — Either `"edit"` (modify existing component) or `"create"` (build new component)
- `timestamp` — ISO timestamp used by the watcher to detect new prompts

## Restarting the Watcher

After processing a prompt, always:
1. Update status to `"completed"`
2. Restart the watcher script as a new background task so the next prompt can be captured

The watcher is designed to run once and exit — it does not loop internally. Each prompt cycle requires a fresh watcher instance.

# Claude Prompt Addon Templates

The Claude Prompt addon adds a panel to Storybook where users can type prompts that Claude Code picks up and processes automatically. It also includes a Settings panel for configuring Limio credentials and deployment.

## addon-prompt/preset.js

```javascript
module.exports = {
    managerEntries(entry = []) {
        return [...entry, require.resolve("./manager")]
    },
}
```

## addon-prompt/manager.js

The manager.js file registers two panels: **Claude Prompt** (for sending prompts) and **Limio Settings** (for configuring Limio credentials). It includes real-time backend status polling, deploy functionality with Limio build tracking, and the Limio connection settings form. The deploy flow posts status updates to `/api/deploy-overlay` for the overlay to display.

The full file is extensive (~540 lines). Key features:

- **PromptPanel** — textarea + send button, real-time backend status polling (`/api/prompt-status`), deploy button with build status tracking, `+ New` button to navigate to new-component story
- **SettingsPanel** — Limio credential form with tenant, region (EU/US/Dev), client ID, client secret; URL preview that shows the correct domain per region; connects via `/api/limio/setup`
- **Region dropdown** must include all three options:
  ```jsx
  <option value="eu">EU (Europe)</option>
  <option value="us">US (United States)</option>
  <option value="dev">Dev (Development)</option>
  ```
- **URL preview** must handle all three regions:
  ```jsx
  {region === "us" ? `${tenant}.prod-us.limio.com` : region === "dev" ? `${tenant}.dev.limio.com` : `${tenant}.prod.limio.com`}
  ```
- **Registration:** `addons.register` with both `PANEL_ID` (Claude Prompt) and `SETTINGS_PANEL_ID` (Limio Settings)

Use the actual file at `component-playground/.storybook/addon-prompt/manager.js` as the source of truth — it evolves faster than this template.

## .storybook/middleware.js

```javascript
const fs = require("fs")
const path = require("path")
const { execSync } = require("child_process")

const PROMPT_FILE = path.resolve(__dirname, "..", ".prompt.json")
const STATUS_FILE = path.resolve(__dirname, "..", ".prompt-status.json")
const DEPLOY_STATUS_FILE = path.resolve(__dirname, "..", ".deploy-status.json")
const PROJECT_ROOT = path.resolve(__dirname, "..", "..")
const CONFIG_FILE = path.join(PROJECT_ROOT, ".limio.json")

// --- Limio config + token management ---

function readLimioConfig() {
    try {
        if (!fs.existsSync(CONFIG_FILE)) return null
        const raw = JSON.parse(fs.readFileSync(CONFIG_FILE, "utf8"))
        if (!raw.tenant || !raw.clientId || !raw.clientSecret) return null
        return raw
    } catch {
        return null
    }
}

function getLimioBaseUrl(config) {
    const region = (config.region || "eu").toLowerCase()
    if (region === "us") return `https://${config.tenant}.prod-us.limio.com`
    if (region === "dev") return `https://${config.tenant}.dev.limio.com`
    return `https://${config.tenant}.prod.limio.com`
}

let tokenCache = { token: null, expiresAt: 0 }

function invalidateToken() {
    tokenCache = { token: null, expiresAt: 0 }
}

async function getAccessToken(config, forceRefresh) {
    const now = Date.now()
    if (!forceRefresh && tokenCache.token && tokenCache.expiresAt > now + 60000) {
        return tokenCache.token
    }
    const baseUrl = getLimioBaseUrl(config)
    const params = new URLSearchParams({
        grant_type: "client_credentials",
        client_id: config.clientId,
        client_secret: config.clientSecret,
    })
    const res = await fetch(`${baseUrl}/oauth2/token`, {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: params.toString(),
    })
    if (!res.ok) {
        const text = await res.text().catch(() => "")
        throw new Error(`Auth failed (${res.status}): ${text}`)
    }
    const data = await res.json()
    tokenCache = {
        token: data.access_token,
        expiresAt: now + (data.expires_in || 3600) * 1000,
    }
    return tokenCache.token
}

async function limioApiFetch(config, url, options) {
    let token = await getAccessToken(config)
    let res = await fetch(url, { ...options, headers: { ...options?.headers, Authorization: `Bearer ${token}` } })
    if (res.status === 401) {
        token = await getAccessToken(config, true)
        res = await fetch(url, { ...options, headers: { ...options?.headers, Authorization: `Bearer ${token}` } })
    }
    return res
}

// --- Helpers ---

function readBody(req) {
    return new Promise((resolve) => {
        let body = ""
        req.on("data", chunk => { body += chunk })
        req.on("end", () => {
            try { resolve(JSON.parse(body)) } catch { resolve({}) }
        })
    })
}

function sendJson(res, statusCode, data) {
    res.statusCode = statusCode
    res.setHeader("Content-Type", "application/json")
    res.end(JSON.stringify(data))
}

module.exports = function expressMiddleware(app) {
    // Auto-reset stale prompt status from interrupted sessions
    try {
        if (fs.existsSync(STATUS_FILE)) {
            const status = JSON.parse(fs.readFileSync(STATUS_FILE, "utf8"))
            if (["queued", "received", "working"].includes(status.state)) {
                fs.writeFileSync(STATUS_FILE, JSON.stringify(
                    { state: "listening", message: "", timestamp: new Date().toISOString() }, null, 2
                ))
            }
        }
    } catch {}

    app.use("/api/prompt", async (req, res, next) => {
        if (req.method === "POST") {
            const body = await readBody(req)
            const { prompt, component, storyId, mode } = body
            if (!prompt) return sendJson(res, 400, { error: "prompt is required" })
            const data = { prompt, component: component || "unknown", storyId: storyId || "", mode: mode || "edit", timestamp: new Date().toISOString() }
            try {
                fs.writeFileSync(PROMPT_FILE, JSON.stringify(data, null, 2))
                const statusData = { state: "queued", message: "Prompt sent — waiting for Claude Code...", timestamp: new Date().toISOString() }
                fs.writeFileSync(STATUS_FILE, JSON.stringify(statusData, null, 2))
                sendJson(res, 200, { success: true })
            } catch (err) {
                console.error("Error writing prompt:", err)
                sendJson(res, 500, { error: err.message })
            }
        } else if (req.method === "GET") {
            try {
                if (fs.existsSync(PROMPT_FILE)) {
                    sendJson(res, 200, JSON.parse(fs.readFileSync(PROMPT_FILE, "utf8")))
                } else {
                    sendJson(res, 200, { prompt: "", component: "", storyId: "", timestamp: "" })
                }
            } catch { sendJson(res, 200, { prompt: "", component: "", storyId: "", timestamp: "" }) }
        } else {
            next()
        }
    })

    app.use("/api/prompt-status", async (req, res, next) => {
        if (req.method === "POST") {
            const body = await readBody(req)
            const { state, message } = body
            if (!state) return sendJson(res, 400, { error: "state is required" })
            try {
                const data = { state, message: message || "", timestamp: new Date().toISOString() }
                fs.writeFileSync(STATUS_FILE, JSON.stringify(data, null, 2))
                sendJson(res, 200, { success: true })
            } catch (err) {
                sendJson(res, 500, { error: err.message })
            }
        } else if (req.method === "GET") {
            try {
                if (fs.existsSync(STATUS_FILE)) {
                    sendJson(res, 200, JSON.parse(fs.readFileSync(STATUS_FILE, "utf8")))
                } else {
                    sendJson(res, 200, { state: "listening", message: "" })
                }
            } catch { sendJson(res, 200, { state: "listening", message: "" }) }
        } else {
            next()
        }
    })

    // --- smart-deploy: fetches, pulls/rebases if behind, handles conflicts ---
    app.use("/api/deploy", async (req, res, next) => {
        if (req.method === "POST") {
            const body = await readBody(req)
            const { component } = body
            if (!component) return sendJson(res, 400, { error: "component is required" })

            const exec = (cmd) => execSync(cmd, { cwd: PROJECT_ROOT, encoding: "utf8", timeout: 30000 }).trim()
            const steps = []

            try {
                // 1. Validate component exists
                const componentDir = path.join("components", component)
                const componentPath = path.join(PROJECT_ROOT, componentDir)
                if (!fs.existsSync(componentPath)) {
                    return sendJson(res, 400, { error: `Component folder not found: ${componentDir}` })
                }

                // 2. Get current branch and check tracking
                const branch = exec("git rev-parse --abbrev-ref HEAD")
                let hasUpstream = false
                try {
                    exec(`git rev-parse --abbrev-ref ${branch}@{upstream}`)
                    hasUpstream = true
                } catch {}
                steps.push(`branch: ${branch}, upstream: ${hasUpstream}`)

                // 3. Fetch remote
                if (hasUpstream) {
                    try { exec("git fetch origin"); steps.push("fetched origin") } catch (err) { steps.push(`fetch warning: ${err.message}`) }
                }

                // 4. Check ahead/behind
                let behind = 0, ahead = 0
                if (hasUpstream) {
                    try {
                        const counts = exec(`git rev-list --left-right --count ${branch}...origin/${branch}`)
                        const parts = counts.split(/\s+/)
                        ahead = parseInt(parts[0], 10) || 0
                        behind = parseInt(parts[1], 10) || 0
                        steps.push(`ahead: ${ahead}, behind: ${behind}`)
                    } catch { steps.push("could not determine ahead/behind") }
                }

                // 5. If behind, stash → pull --rebase → pop
                let didStash = false
                if (behind > 0) {
                    const dirtyStatus = exec("git status --porcelain")
                    if (dirtyStatus.length > 0) {
                        exec("git stash push -m \"deploy-auto-stash\" --include-untracked")
                        didStash = true
                        steps.push("stashed uncommitted changes")
                    }
                    try {
                        exec("git pull --rebase origin " + branch)
                        steps.push("pulled and rebased")
                    } catch (pullErr) {
                        try { exec("git rebase --abort") } catch {}
                        if (didStash) { try { exec("git stash pop") } catch {} }
                        return sendJson(res, 409, { error: "Merge conflict while pulling remote changes. Please resolve manually.", details: pullErr.message, steps })
                    }
                    if (didStash) {
                        try { exec("git stash pop"); steps.push("restored stashed changes") } catch (popErr) {
                            return sendJson(res, 409, { error: "Pulled successfully but local changes conflict with remote. Run 'git stash pop' and resolve.", details: popErr.message, steps })
                        }
                    }
                }

                // 6. Stage component + related stories
                exec(`git add ${componentDir}/`)
                steps.push(`staged ${componentDir}/`)
                const storiesDir = path.join(PROJECT_ROOT, "component-playground", "src", "stories")
                if (fs.existsSync(storiesDir)) {
                    const storyFiles = fs.readdirSync(storiesDir).filter(f => f.endsWith(".stories.js") || f.endsWith(".stories.jsx"))
                    for (const file of storyFiles) {
                        const content = fs.readFileSync(path.join(storiesDir, file), "utf8")
                        if (content.includes(component)) {
                            exec(`git add component-playground/src/stories/${file}`)
                            steps.push(`staged story: ${file}`)
                        }
                    }
                }

                // 7. Check for no-op
                const staged = exec("git diff --cached --name-only")
                if (!staged) {
                    return sendJson(res, 200, { success: true, message: `No changes to deploy for ${component} — already up to date`, steps, noChanges: true })
                }

                // 8. Commit + push (with retry)
                exec(`git commit -m "Deploy component: ${component}"`)
                const commitHash = exec("git rev-parse HEAD")
                steps.push(`committed: ${commitHash.substring(0, 8)}`)

                const pushCmd = hasUpstream ? "git push" : `git push -u origin ${branch}`
                try {
                    exec(pushCmd)
                    steps.push("pushed")
                } catch (pushErr) {
                    steps.push(`push failed, retrying: ${pushErr.message}`)
                    try {
                        exec("git pull --rebase origin " + branch)
                        exec(pushCmd)
                        steps.push("retried pull --rebase + push: success")
                    } catch (retryErr) {
                        try { exec("git rebase --abort") } catch {}
                        return sendJson(res, 409, { error: "Push failed after retry. Remote may have conflicting changes.", details: retryErr.message, steps, commitHash })
                    }
                }

                sendJson(res, 200, { success: true, message: `Deployed ${component} successfully`, commitHash, steps, pulled: behind > 0 })
            } catch (err) {
                console.error("Deploy error:", err.message)
                sendJson(res, 500, { error: err.message, steps })
            }
        } else if (req.method === "GET") {
            try {
                const branch = execSync("git rev-parse --abbrev-ref HEAD", { cwd: PROJECT_ROOT }).toString().trim()
                const status = execSync("git status --porcelain", { cwd: PROJECT_ROOT }).toString().trim()
                let ahead = 0, behind = 0
                try {
                    const counts = execSync(`git rev-list --left-right --count ${branch}...origin/${branch}`, { cwd: PROJECT_ROOT, encoding: "utf8" }).trim()
                    const parts = counts.split(/\s+/)
                    ahead = parseInt(parts[0], 10) || 0
                    behind = parseInt(parts[1], 10) || 0
                } catch {}
                sendJson(res, 200, { branch, clean: status.length === 0, status, ahead, behind })
            } catch (err) {
                sendJson(res, 500, { error: err.message })
            }
        } else {
            next()
        }
    })

    // --- Limio connection status ---
    app.use("/api/limio/status", async (req, res, next) => {
        if (req.method !== "GET") return next()
        const config = readLimioConfig()
        if (!config) return sendJson(res, 200, { configured: false })
        try {
            await getAccessToken(config)
            sendJson(res, 200, { configured: true, tenant: config.tenant, region: config.region || "eu", baseUrl: getLimioBaseUrl(config) })
        } catch (err) {
            sendJson(res, 200, { configured: false, error: `Credentials invalid: ${err.message}` })
        }
    })

    // --- Save Limio credentials ---
    app.use("/api/limio/setup", async (req, res, next) => {
        if (req.method !== "POST") return next()
        const body = await readBody(req)
        const { tenant, region, clientId, clientSecret } = body
        if (!tenant || !clientId || !clientSecret) {
            return sendJson(res, 400, { error: "tenant, clientId, and clientSecret are required" })
        }
        const config = { tenant, region: region || "eu", clientId, clientSecret }
        try {
            invalidateToken()
            await getAccessToken(config, true)
        } catch (err) {
            return sendJson(res, 400, { error: `Authentication failed: ${err.message}` })
        }
        try {
            fs.writeFileSync(CONFIG_FILE, JSON.stringify(config, null, 2))
            sendJson(res, 200, { success: true, tenant, region: config.region, baseUrl: getLimioBaseUrl(config) })
        } catch (err) {
            sendJson(res, 500, { error: `Failed to save config: ${err.message}` })
        }
    })

    // --- Build status proxy ---
    // Fetches the latest build from Limio. If deployedAfter is provided,
    // only considers builds that started after that timestamp (handles
    // GH→CodeCommit mirroring where local commit hashes don't match).
    app.use("/api/build-status", async (req, res, next) => {
        if (req.method !== "GET") return next()
        const url = new URL(req.url, "http://localhost")
        const deployedAfter = url.searchParams.get("deployedAfter")
        const config = readLimioConfig()
        if (!config) return sendJson(res, 400, { error: "Limio not configured" })
        try {
            const baseUrl = getLimioBaseUrl(config)
            const apiRes = await limioApiFetch(config, `${baseUrl}/api/component/builds`)
            if (!apiRes.ok) {
                const text = await apiRes.text().catch(() => "")
                return sendJson(res, apiRes.status, { found: false, error: `Limio API error (${apiRes.status}): ${text}` })
            }
            const data = await apiRes.json()
            if (!data) {
                return sendJson(res, 200, { found: false })
            }
            const build = Array.isArray(data) ? data[0] : data
            if (!build || !build.startTime) {
                return sendJson(res, 200, { found: false })
            }
            // If deployedAfter is set, only match builds that started after the deploy
            if (deployedAfter) {
                const buildStart = new Date(build.startTime).getTime()
                const deployTime = new Date(deployedAfter).getTime()
                if (buildStart < deployTime) {
                    return sendJson(res, 200, { found: false })
                }
            }
            sendJson(res, 200, {
                found: true,
                buildStatus: build.status || build.buildStatus || "UNKNOWN",
                buildComplete: ["SUCCEEDED", "FAILED", "ERROR"].includes((build.status || build.buildStatus || "").toUpperCase()),
                logErrors: build.logErrors || build.errors || null,
                startTime: build.startTime || build.createdAt || null,
                endTime: build.endTime || build.completedAt || null,
            })
        } catch (err) {
            sendJson(res, 500, { error: err.message })
        }
    })

    // --- Deploy overlay status ---
    app.use("/api/deploy-overlay", async (req, res, next) => {
        if (req.method === "POST") {
            const body = await readBody(req)
            const { state, message, component, progress } = body
            if (!state) return sendJson(res, 400, { error: "state is required" })
            try {
                const data = { state, message: message || "", component: component || "", progress: progress || 0, timestamp: new Date().toISOString() }
                fs.writeFileSync(DEPLOY_STATUS_FILE, JSON.stringify(data, null, 2))
                sendJson(res, 200, { success: true })
            } catch (err) {
                sendJson(res, 500, { error: err.message })
            }
        } else if (req.method === "GET") {
            try {
                if (fs.existsSync(DEPLOY_STATUS_FILE)) {
                    sendJson(res, 200, JSON.parse(fs.readFileSync(DEPLOY_STATUS_FILE, "utf8")))
                } else {
                    sendJson(res, 200, { state: "idle", message: "", component: "", progress: 0 })
                }
            } catch { sendJson(res, 200, { state: "idle", message: "", component: "", progress: 0 }) }
        } else {
            next()
        }
    })
}
```

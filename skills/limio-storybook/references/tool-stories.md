# Tool Stories

These stories provide built-in tools in the Storybook sidebar under the "Tools" category.

## src/stories/LimioSetup.stories.js

The LimioSetup story provides a guided onboarding wizard at **Tools > Limio Setup** in the Storybook sidebar.

The wizard has three steps:
1. **Welcome** — branded header with Limio logo + "Get Started" button
2. **Enter Credentials** — form with tenant, region (EU/US/Dev), client ID, client secret; live URL preview that shows the correct domain per region; POSTs to `/api/limio/setup`
3. **Connected** — success confirmation with tenant name, "Build a Component" and "Browse Components" action buttons

Key behaviors:
- On mount it auto-detects existing config via `GET /api/limio/status` and skips to Step 3 if already configured
- Region dropdown includes EU, US, and Dev options
- URL preview updates live: `tenant.prod.limio.com` (EU), `tenant.prod-us.limio.com` (US), `tenant.dev.limio.com` (Dev)
- Validates credentials by attempting authentication before saving
- Shows error messages for invalid credentials

Use the actual file at `component-playground/src/stories/LimioSetup.stories.js` as the source of truth — it evolves faster than this description.

Story export:
```javascript
export default {
    title: "Tools/Limio Setup",
    parameters: {
        layout: "fullscreen",
        previewTabs: { "storybook/docs/panel": { hidden: true } },
    },
}

export const Setup = {
    render: () => <SetupWizard />,
}
```

## src/stories/NewComponent.stories.js

The NewComponent story provides a dedicated page at **Tools > New Component** where users can name a component, describe what they want, and submit it to Claude Code for creation.

The `+ New` button in the Claude Prompt panel header also navigates to this page.

```javascript
import React, { useState, useEffect, useCallback, useRef } from "react"

const toKebabCase = (str) =>
    str
        .replace(/([a-z])([A-Z])/g, "$1-$2")
        .replace(/[\s_]+/g, "-")
        .replace(/[^a-z0-9-]/gi, "")
        .toLowerCase()

const BuilderPage = () => {
    const [componentName, setComponentName] = useState("")
    const [prompt, setPrompt] = useState("")
    const [submitting, setSubmitting] = useState(false)
    const [submitted, setSubmitted] = useState(false)
    const [error, setError] = useState(null)
    const [prefilled, setPrefilled] = useState(false)
    const textareaRef = useRef(null)

    // Prefill from .prompt.json on mount
    useEffect(() => {
        const prefill = async () => {
            try {
                const res = await fetch("/api/prompt")
                if (!res.ok) return
                const data = await res.json()
                if (data.mode === "create" && data.prompt) {
                    setPrompt(data.prompt)
                    if (data.component && data.component !== "unknown") {
                        setComponentName(data.component)
                    }
                    setPrefilled(true)
                }
            } catch {}
        }
        prefill()
    }, [])

    const kebab = toKebabCase(componentName)

    const handleSubmit = useCallback(async () => {
        if (!componentName.trim() || !prompt.trim() || submitting) return
        setSubmitting(true)
        setError(null)
        try {
            const res = await fetch("/api/prompt", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({
                    prompt: prompt.trim(),
                    component: kebab || componentName.trim(),
                    storyId: "tools-new-component--builder",
                    mode: "create",
                }),
            })
            if (res.ok) {
                setSubmitted(true)
            } else {
                const data = await res.json().catch(() => ({}))
                setError(data.error || "Failed to send prompt")
                setSubmitting(false)
            }
        } catch (err) {
            setError("Connection error — is Storybook middleware running?")
            setSubmitting(false)
        }
    }, [componentName, prompt, kebab, submitting])

    const handleKeyDown = useCallback(
        (e) => {
            if ((e.metaKey || e.ctrlKey) && e.key === "Enter") {
                e.preventDefault()
                handleSubmit()
            }
        },
        [handleSubmit]
    )

    const handleReset = () => {
        setComponentName("")
        setPrompt("")
        setSubmitting(false)
        setSubmitted(false)
        setError(null)
        setPrefilled(false)
        // Reset status to listening
        fetch("/api/prompt-status", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ state: "listening", message: "" }),
        }).catch(() => {})
    }

    const isDisabled = submitting || submitted

    return (
        <div style={s.page}>
            <div style={s.card}>
                <div style={s.header}>
                    <div style={s.logo}>C</div>
                    <div>
                        <h1 style={s.title}>New Component</h1>
                        <p style={s.subtitle}>
                            Describe what you want and Claude will build it.
                        </p>
                    </div>
                </div>

                {prefilled && !submitted && (
                    <div style={s.prefillBanner}>
                        <span style={{ fontSize: "13px" }}>&#9889;</span>
                        <span>Pre-filled from your last prompt</span>
                    </div>
                )}

                {submitted && (
                    <div style={s.successBanner}>
                        <span style={{ fontSize: "14px" }}>&#10003;</span>
                        <span>
                            Prompt sent — Claude Code is building{" "}
                            <strong>{kebab || componentName}</strong>. Watch the
                            overlay for progress.
                        </span>
                    </div>
                )}

                {error && (
                    <div style={s.errorBanner}>
                        <span style={{ fontSize: "14px" }}>&#10007;</span>
                        <span>{error}</span>
                    </div>
                )}

                <div style={s.field}>
                    <label style={s.label}>Component Name</label>
                    <input
                        style={{
                            ...s.input,
                            ...(isDisabled ? { opacity: 0.5 } : {}),
                        }}
                        type="text"
                        value={componentName}
                        onChange={(e) => setComponentName(e.target.value)}
                        placeholder='e.g. "Pricing Table" or "Hero Banner"'
                        disabled={isDisabled}
                    />
                    {componentName && (
                        <div style={s.kebabPreview}>
                            <span style={s.kebabLabel}>folder:</span>
                            <code style={s.kebabValue}>
                                components/{kebab}/
                            </code>
                        </div>
                    )}
                </div>

                <div style={s.field}>
                    <label style={s.label}>Prompt</label>
                    <textarea
                        ref={textareaRef}
                        style={{
                            ...s.textarea,
                            ...(isDisabled ? { opacity: 0.5 } : {}),
                        }}
                        value={prompt}
                        onChange={(e) => setPrompt(e.target.value)}
                        onKeyDown={handleKeyDown}
                        placeholder="Describe the component you want to build. Include details about layout, features, styling, and any specific Limio SDK hooks to use..."
                        disabled={isDisabled}
                        rows={8}
                    />
                </div>

                <div style={s.footer}>
                    {!submitted ? (
                        <button
                            style={{
                                ...s.button,
                                ...(!componentName.trim() ||
                                !prompt.trim() ||
                                submitting
                                    ? s.buttonDisabled
                                    : {}),
                            }}
                            onClick={handleSubmit}
                            disabled={
                                !componentName.trim() ||
                                !prompt.trim() ||
                                submitting
                            }
                        >
                            {submitting
                                ? "Sending..."
                                : "Build Component"}
                        </button>
                    ) : (
                        <button style={s.resetButton} onClick={handleReset}>
                            Build Another
                        </button>
                    )}
                    <span style={s.hint}>
                        <strong>Cmd+Enter</strong> to submit
                    </span>
                </div>
            </div>
        </div>
    )
}

const s = {
    page: {
        minHeight: "100vh",
        background: "#f8f9fb",
        display: "flex",
        alignItems: "flex-start",
        justifyContent: "center",
        padding: "60px 20px",
        fontFamily:
            '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif',
        WebkitFontSmoothing: "antialiased",
    },
    card: {
        background: "#fff",
        borderRadius: "16px",
        padding: "40px",
        width: "100%",
        maxWidth: "580px",
        boxShadow:
            "0 1px 3px rgba(0,0,0,0.04), 0 4px 16px rgba(0,0,0,0.06)",
        display: "flex",
        flexDirection: "column",
        gap: "24px",
    },
    header: {
        display: "flex",
        alignItems: "center",
        gap: "14px",
    },
    logo: {
        width: "40px",
        height: "40px",
        borderRadius: "12px",
        background: "linear-gradient(135deg, #d4a574 0%, #c4956a 100%)",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        color: "#fff",
        fontSize: "18px",
        fontWeight: "700",
        flexShrink: 0,
    },
    title: {
        fontSize: "20px",
        fontWeight: "700",
        color: "#1a1f36",
        margin: 0,
        lineHeight: 1.3,
    },
    subtitle: {
        fontSize: "13px",
        color: "#697386",
        margin: "2px 0 0",
        lineHeight: 1.4,
    },
    prefillBanner: {
        display: "flex",
        alignItems: "center",
        gap: "8px",
        background: "#F5F3FF",
        borderRadius: "10px",
        padding: "10px 14px",
        border: "1px solid #DDD6FE",
        fontSize: "12px",
        fontWeight: "500",
        color: "#5B21B6",
    },
    successBanner: {
        display: "flex",
        alignItems: "center",
        gap: "8px",
        background: "#F0FDF4",
        borderRadius: "10px",
        padding: "10px 14px",
        border: "1px solid #BBF7D0",
        fontSize: "12px",
        fontWeight: "500",
        color: "#166534",
        lineHeight: 1.5,
    },
    errorBanner: {
        display: "flex",
        alignItems: "center",
        gap: "8px",
        background: "#FEF2F2",
        borderRadius: "10px",
        padding: "10px 14px",
        border: "1px solid #FECACA",
        fontSize: "12px",
        fontWeight: "500",
        color: "#991B1B",
    },
    field: {
        display: "flex",
        flexDirection: "column",
        gap: "6px",
    },
    label: {
        fontSize: "13px",
        fontWeight: "600",
        color: "#1a1f36",
    },
    input: {
        padding: "10px 12px",
        borderRadius: "8px",
        border: "1px solid #e3e8ee",
        fontSize: "14px",
        fontFamily: "inherit",
        color: "#1a1f36",
        outline: "none",
        transition: "border-color 0.15s ease, box-shadow 0.15s ease",
    },
    kebabPreview: {
        display: "flex",
        alignItems: "center",
        gap: "6px",
        fontSize: "12px",
        color: "#697386",
    },
    kebabLabel: {
        fontWeight: "500",
    },
    kebabValue: {
        fontFamily:
            '"SF Mono", SFMono-Regular, Consolas, "Liberation Mono", Menlo, monospace',
        color: "#635BFF",
        fontSize: "12px",
    },
    textarea: {
        padding: "12px",
        borderRadius: "8px",
        border: "1px solid #e3e8ee",
        fontSize: "14px",
        fontFamily: "inherit",
        color: "#1a1f36",
        lineHeight: 1.6,
        resize: "vertical",
        outline: "none",
        transition: "border-color 0.15s ease, box-shadow 0.15s ease",
        minHeight: "160px",
    },
    footer: {
        display: "flex",
        alignItems: "center",
        gap: "14px",
    },
    button: {
        padding: "10px 24px",
        borderRadius: "8px",
        border: "none",
        background: "#635BFF",
        color: "#fff",
        fontSize: "14px",
        fontWeight: "600",
        cursor: "pointer",
        fontFamily: "inherit",
        transition: "opacity 0.15s ease",
    },
    buttonDisabled: {
        opacity: 0.5,
        cursor: "not-allowed",
    },
    resetButton: {
        padding: "10px 24px",
        borderRadius: "8px",
        border: "1px solid #e3e8ee",
        background: "#fff",
        color: "#1a1f36",
        fontSize: "14px",
        fontWeight: "600",
        cursor: "pointer",
        fontFamily: "inherit",
    },
    hint: {
        fontSize: "12px",
        color: "#a3acb9",
    },
}

export default {
    title: "Tools/New Component",
    parameters: {
        layout: "fullscreen",
        previewTabs: { "storybook/docs/panel": { hidden: true } },
    },
}

export const Builder = {
    render: () => <BuilderPage />,
}
```

Key features:
- **Component name input** — auto-converts to kebab-case and shows folder path preview
- **Prompt textarea** — describe the component, Cmd+Enter to submit
- **Prefill support** — auto-fills from `.prompt.json` if a previous create prompt exists
- **Mode: "create"** — the prompt is sent with `mode: "create"` so the watcher can distinguish from edit prompts
- **Status banners** — shows success, error, and prefill states

# Claude Overlay Component

The Claude Overlay shows a polished loading screen while Claude Code is working. It polls `/api/prompt-status` and displays animated phases, a pulsing ring logo, and success/error states with smooth transitions. It also includes a deploy overlay with rocket animation and progress bar that polls `/api/deploy-overlay` to show deployment status.

## component-playground/.storybook/claude-overlay.js

```javascript
import React, { useState, useEffect, useRef } from "react"

const PHASES = [
    { message: "Prompt received..." },
    { message: "Reading component files..." },
    { message: "Making magic..." },
    { message: "Writing code changes..." },
    { message: "Sprinkling some pixels..." },
    { message: "Almost there..." },
]

const COMPLETED_MESSAGES = [
    "Changes applied!",
    "All done — check it out!",
    "Component updated!",
]

function useStatusPoller() {
    const [status, setStatus] = useState({ state: "listening", message: "" })
    const prevState = useRef("listening")

    useEffect(() => {
        let active = true
        const poll = async () => {
            try {
                const res = await fetch("/api/prompt-status")
                if (res.ok && active) {
                    const data = await res.json()
                    setStatus(data)
                    prevState.current = data.state
                }
            } catch {}
        }
        poll()
        const id = setInterval(poll, 800)
        return () => { active = false; clearInterval(id) }
    }, [])

    return status
}

function useDeployPoller() {
    const [deploy, setDeploy] = useState({ state: "idle", message: "", progress: 0 })

    useEffect(() => {
        let active = true
        const poll = async () => {
            try {
                const res = await fetch("/api/deploy-overlay")
                if (res.ok && active) {
                    setDeploy(await res.json())
                }
            } catch {}
        }
        poll()
        const id = setInterval(poll, 1000)
        return () => { active = false; clearInterval(id) }
    }, [])

    return deploy
}

function PulsingRing() {
    return (
        <div style={styles.ringContainer}>
            <div style={{ ...styles.ring, ...styles.ring1 }} />
            <div style={{ ...styles.ring, ...styles.ring2 }} />
            <div style={{ ...styles.ring, ...styles.ring3 }} />
            <div style={styles.ringCenter}>
                <span style={styles.ringLogo}>C</span>
            </div>
        </div>
    )
}

function RocketIcon({ launching }) {
    return (
        <div style={{
            position: "relative",
            width: "80px",
            height: "80px",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
        }}>
            {/* Exhaust particles */}
            {launching && (
                <>
                    <div style={{ position: "absolute", bottom: "2px", left: "50%", transform: "translateX(-50%)", width: "20px", height: "30px", overflow: "hidden" }}>
                        <div style={{ position: "absolute", width: "4px", height: "4px", borderRadius: "50%", background: "#F59E0B", left: "4px", animation: "deploy-particle 0.8s ease-out infinite" }} />
                        <div style={{ position: "absolute", width: "3px", height: "3px", borderRadius: "50%", background: "#EF4444", left: "10px", animation: "deploy-particle 0.8s ease-out 0.2s infinite" }} />
                        <div style={{ position: "absolute", width: "3px", height: "3px", borderRadius: "50%", background: "#F97316", left: "7px", animation: "deploy-particle 0.8s ease-out 0.4s infinite" }} />
                    </div>
                    {/* Glow under rocket */}
                    <div style={{
                        position: "absolute",
                        bottom: "0",
                        left: "50%",
                        transform: "translateX(-50%)",
                        width: "40px",
                        height: "20px",
                        borderRadius: "50%",
                        background: "radial-gradient(ellipse, rgba(249, 115, 22, 0.4) 0%, transparent 70%)",
                        animation: "deploy-glow 0.6s ease-in-out infinite alternate",
                    }} />
                </>
            )}
            <svg
                width="48"
                height="48"
                viewBox="0 0 48 48"
                fill="none"
                style={{
                    animation: launching ? "deploy-hover 1.2s ease-in-out infinite" : "none",
                    filter: launching ? "drop-shadow(0 4px 12px rgba(13, 159, 110, 0.3))" : "none",
                }}
            >
                {/* Rocket body */}
                <path d="M24 6C24 6 18 14 18 26L21 30H27L30 26C30 14 24 6 24 6Z" fill="#0d9f6e" />
                {/* Nose cone */}
                <path d="M24 6C24 6 21 12 21 16L24 10L27 16C27 12 24 6 24 6Z" fill="#10B981" />
                {/* Window */}
                <circle cx="24" cy="20" r="3" fill="#ECFDF5" stroke="#0d9f6e" strokeWidth="0.5" />
                <circle cx="24" cy="20" r="1.5" fill="#6EE7B7" />
                {/* Fins */}
                <path d="M18 24L14 30L18 28Z" fill="#059669" />
                <path d="M30 24L34 30L30 28Z" fill="#059669" />
                {/* Exhaust nozzle */}
                <path d="M21 30L22 34H26L27 30" fill="#6B7280" />
                {/* Flame */}
                {launching && (
                    <>
                        <path d="M22.5 34L24 42L25.5 34" fill="#F59E0B" style={{ animation: "deploy-flame 0.3s ease-in-out infinite alternate" }} />
                        <path d="M23 34L24 39L25 34" fill="#EF4444" style={{ animation: "deploy-flame 0.3s ease-in-out 0.15s infinite alternate" }} />
                    </>
                )}
            </svg>
        </div>
    )
}

function ProgressBar({ progress, status }) {
    const barColor = status === "error" ? "#EF4444" : status === "success" ? "#10B981" : "#0d9f6e"
    return (
        <div style={styles.progressContainer}>
            <div style={styles.progressTrack}>
                <div style={{
                    ...styles.progressFill,
                    width: `${Math.min(100, Math.max(0, progress))}%`,
                    background: barColor,
                    transition: "width 0.8s cubic-bezier(0.4, 0, 0.2, 1)",
                }}>
                    {progress > 15 && progress < 100 && (
                        <div style={styles.progressShimmer} />
                    )}
                </div>
            </div>
            <span style={styles.progressLabel}>{Math.round(progress)}%</span>
        </div>
    )
}

function DeployOverlay() {
    const deploy = useDeployPoller()
    const [visible, setVisible] = useState(false)
    const [exiting, setExiting] = useState(false)
    const [limioBaseUrl, setLimioBaseUrl] = useState("")

    const isActive = deploy.state === "pushing" || deploy.state === "building"
    const isSuccess = deploy.state === "success"
    const isError = deploy.state === "error" || deploy.state === "timeout"

    useEffect(() => {
        fetch("/api/limio/status")
            .then(r => r.json())
            .then(data => { if (data.baseUrl) setLimioBaseUrl(data.baseUrl) })
            .catch(() => {})
    }, [])

    useEffect(() => {
        if (isActive) {
            setVisible(true)
            setExiting(false)
        }
    }, [isActive])

    const dismiss = () => {
        setExiting(true)
        setTimeout(() => {
            setVisible(false)
            setExiting(false)
            fetch("/api/deploy-overlay", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ state: "idle", message: "", progress: 0 }),
            }).catch(() => {})
        }, 600)
    }

    useEffect(() => {
        if (isError && visible) {
            const timeout = setTimeout(dismiss, 4000)
            return () => clearTimeout(timeout)
        }
    }, [isError, visible])

    if (!visible) return null

    return (
        <div style={{
            ...styles.overlay,
            opacity: exiting ? 0 : 1,
            transition: "opacity 0.6s ease",
        }}>
            <style>{deployKeyframes}</style>
            <div style={{
                ...styles.dialog,
                animation: exiting ? "claude-slideDown 0.5s ease forwards" : "claude-slideIn 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards",
            }}>
                <div style={styles.dialogInner}>
                    {isActive && <RocketIcon launching={true} />}
                    {isSuccess && (
                        <div style={styles.successIcon}>
                            <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                                <circle cx="24" cy="24" r="24" fill="#10B981" />
                                <path d="M15 24.5L21 30.5L33 18.5" stroke="white" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round" style={{ strokeDasharray: 30, strokeDashoffset: 0, animation: "claude-checkDraw 0.5s ease 0.2s both" }} />
                            </svg>
                        </div>
                    )}
                    {isError && (
                        <div style={styles.errorIcon}>
                            <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                                <circle cx="24" cy="24" r="24" fill="#EF4444" />
                                <path d="M17 17L31 31M31 17L17 31" stroke="white" strokeWidth="3" strokeLinecap="round" />
                            </svg>
                        </div>
                    )}

                    {(isActive || isSuccess) && (
                        <ProgressBar progress={deploy.progress || 0} status={deploy.state} />
                    )}

                    <div style={styles.messageArea}>
                        <p style={{
                            ...styles.message,
                            color: isSuccess ? "#10B981" : isError ? "#EF4444" : "#1a1f36",
                        }}>
                            {deploy.message || "Deploying..."}
                        </p>
                        {deploy.component && isActive && (
                            <p style={{ fontSize: "13px", color: "#697386", margin: "4px 0 0", fontWeight: "500" }}>
                                {deploy.component}
                            </p>
                        )}
                    </div>

                    {isSuccess && (
                        <div style={{ display: "flex", gap: "10px", marginTop: "4px" }}>
                            {limioBaseUrl && (
                                <a
                                    href={`${limioBaseUrl}/catalog/pages2`}
                                    target="_blank"
                                    rel="noopener noreferrer"
                                    style={{
                                        display: "inline-flex",
                                        alignItems: "center",
                                        gap: "6px",
                                        padding: "10px 20px",
                                        borderRadius: "10px",
                                        background: "#0d9f6e",
                                        color: "#fff",
                                        fontSize: "14px",
                                        fontWeight: "600",
                                        textDecoration: "none",
                                        cursor: "pointer",
                                        transition: "transform 0.15s",
                                    }}
                                >
                                    Open Page Builder &#8599;
                                </a>
                            )}
                            <button
                                onClick={dismiss}
                                style={{
                                    display: "inline-flex",
                                    alignItems: "center",
                                    padding: "10px 16px",
                                    borderRadius: "10px",
                                    border: "1px solid #e3e8ee",
                                    background: "#fff",
                                    color: "#697386",
                                    fontSize: "14px",
                                    fontWeight: "600",
                                    cursor: "pointer",
                                    fontFamily: "inherit",
                                }}
                            >
                                Dismiss
                            </button>
                        </div>
                    )}
                </div>
            </div>
        </div>
    )
}

export function ClaudeOverlay() {
    const status = useStatusPoller()
    const [phaseIndex, setPhaseIndex] = useState(0)
    const [visible, setVisible] = useState(false)
    const [exiting, setExiting] = useState(false)
    const phaseTimer = useRef(null)

    const isActive = status.state === "working" || status.state === "queued" || status.state === "received"
    const isCompleted = status.state === "completed"
    const isError = status.state === "error"

    // Show overlay when active, hide when back to listening
    useEffect(() => {
        if (isActive) {
            setVisible(true)
            setExiting(false)
            setPhaseIndex(0)
        } else if (status.state === "listening" && visible && !exiting) {
            setExiting(true)
            setTimeout(() => {
                setVisible(false)
                setExiting(false)
            }, 600)
        }
    }, [isActive, status.state])

    useEffect(() => {
        if (isActive) {
            phaseTimer.current = setInterval(() => {
                setPhaseIndex(prev => (prev + 1) % PHASES.length)
            }, 2800)
            return () => clearInterval(phaseTimer.current)
        }
    }, [isActive])

    useEffect(() => {
        if (isCompleted && visible) {
            clearInterval(phaseTimer.current)
            const timeout = setTimeout(() => {
                setExiting(true)
                setTimeout(() => {
                    setVisible(false)
                    setExiting(false)
                }, 600)
            }, 2000)
            return () => clearTimeout(timeout)
        }
    }, [isCompleted, visible])

    useEffect(() => {
        if (isError && visible) {
            clearInterval(phaseTimer.current)
            const timeout = setTimeout(() => {
                setExiting(true)
                setTimeout(() => {
                    setVisible(false)
                    setExiting(false)
                }, 600)
            }, 3000)
            return () => clearTimeout(timeout)
        }
    }, [isError, visible])

    const phase = PHASES[phaseIndex]
    const completedMsg = COMPLETED_MESSAGES[Math.floor(Math.random() * COMPLETED_MESSAGES.length)]
    const displayMessage = status.message || (isCompleted ? completedMsg : isError ? "Something went wrong" : phase.message)

    return (
        <>
            {/* Prompt overlay */}
            {visible && (
                <div style={{
                    ...styles.overlay,
                    opacity: exiting ? 0 : 1,
                    transition: "opacity 0.6s ease",
                }}>
                    <style>{keyframes}</style>
                    <div style={{
                        ...styles.dialog,
                        animation: exiting ? "claude-slideDown 0.5s ease forwards" : "claude-slideIn 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards",
                    }}>
                        <div style={styles.dialogInner}>
                            {isActive && <PulsingRing />}
                            {isCompleted && (
                                <div style={styles.successIcon}>
                                    <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                                        <circle cx="24" cy="24" r="24" fill="#10B981" />
                                        <path d="M15 24.5L21 30.5L33 18.5" stroke="white" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round" style={{ strokeDasharray: 30, strokeDashoffset: 0, animation: "claude-checkDraw 0.5s ease 0.2s both" }} />
                                    </svg>
                                </div>
                            )}
                            {isError && (
                                <div style={styles.errorIcon}>
                                    <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                                        <circle cx="24" cy="24" r="24" fill="#EF4444" />
                                        <path d="M17 17L31 31M31 17L17 31" stroke="white" strokeWidth="3" strokeLinecap="round" />
                                    </svg>
                                </div>
                            )}
                            <div style={styles.messageArea}>
                                <p style={{
                                    ...styles.message,
                                    color: isCompleted ? "#10B981" : isError ? "#EF4444" : "#1a1f36",
                                }}>
                                    {displayMessage}
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
            )}

            {/* Deploy overlay */}
            <DeployOverlay />
        </>
    )
}

const keyframes = `
@keyframes claude-slideIn {
    from { opacity: 0; transform: translateY(30px) scale(0.95); }
    to   { opacity: 1; transform: translateY(0) scale(1); }
}
@keyframes claude-slideDown {
    from { opacity: 1; transform: translateY(0) scale(1); }
    to   { opacity: 0; transform: translateY(-20px) scale(0.95); }
}
@keyframes claude-pulse {
    0%, 100% { transform: scale(1); opacity: 0.3; }
    50%      { transform: scale(1.6); opacity: 0; }
}
@keyframes claude-pulse2 {
    0%, 100% { transform: scale(1); opacity: 0.2; }
    50%      { transform: scale(1.8); opacity: 0; }
}
@keyframes claude-pulse3 {
    0%, 100% { transform: scale(1); opacity: 0.15; }
    50%      { transform: scale(2); opacity: 0; }
}
@keyframes claude-checkDraw {
    from { stroke-dashoffset: 30; }
    to   { stroke-dashoffset: 0; }
}
`

const deployKeyframes = `
${keyframes}
@keyframes deploy-hover {
    0%, 100% { transform: translateY(2px); }
    50%      { transform: translateY(-4px); }
}
@keyframes deploy-flame {
    from { transform: scaleY(0.8) scaleX(0.9); }
    to   { transform: scaleY(1.2) scaleX(1.1); }
}
@keyframes deploy-particle {
    0%   { transform: translateY(0); opacity: 1; }
    100% { transform: translateY(20px); opacity: 0; }
}
@keyframes deploy-glow {
    from { opacity: 0.3; transform: translateX(-50%) scale(0.9); }
    to   { opacity: 0.6; transform: translateX(-50%) scale(1.1); }
}
@keyframes deploy-shimmer {
    from { transform: translateX(-100%); }
    to   { transform: translateX(200%); }
}
`

const styles = {
    overlay: {
        position: "fixed",
        inset: 0,
        zIndex: 999999,
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        background: "rgba(15, 23, 42, 0.4)",
        backdropFilter: "blur(8px)",
        WebkitBackdropFilter: "blur(8px)",
    },
    dialog: {
        background: "#FFFFFF",
        borderRadius: "24px",
        padding: "40px 48px",
        minWidth: "380px",
        maxWidth: "440px",
        boxShadow: "0 25px 60px rgba(0, 0, 0, 0.15), 0 0 0 1px rgba(0, 0, 0, 0.05)",
        textAlign: "center",
        fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif',
    },
    dialogInner: {
        display: "flex",
        flexDirection: "column",
        alignItems: "center",
        gap: "20px",
    },
    ringContainer: {
        position: "relative",
        width: "80px",
        height: "80px",
    },
    ring: {
        position: "absolute",
        inset: 0,
        borderRadius: "50%",
        border: "2px solid #635BFF",
    },
    ring1: { animation: "claude-pulse 2s ease-in-out infinite" },
    ring2: { animation: "claude-pulse2 2s ease-in-out 0.4s infinite" },
    ring3: { animation: "claude-pulse3 2s ease-in-out 0.8s infinite" },
    ringCenter: {
        position: "absolute",
        top: "50%",
        left: "50%",
        transform: "translate(-50%, -50%)",
        width: "48px",
        height: "48px",
        borderRadius: "14px",
        background: "linear-gradient(135deg, #d4a574 0%, #c4956a 100%)",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        boxShadow: "0 4px 12px rgba(196, 149, 106, 0.3)",
    },
    ringLogo: {
        color: "#fff",
        fontSize: "20px",
        fontWeight: "700",
    },
    messageArea: {
        display: "flex",
        flexDirection: "column",
        alignItems: "center",
        gap: "8px",
        minHeight: "60px",
        justifyContent: "center",
    },
    message: {
        fontSize: "16px",
        fontWeight: "600",
        margin: 0,
        lineHeight: 1.4,
        letterSpacing: "-0.01em",
        transition: "color 0.3s ease",
    },
    successIcon: {
        animation: "claude-slideIn 0.4s ease",
    },
    errorIcon: {
        animation: "claude-slideIn 0.4s ease",
    },
    progressContainer: {
        width: "100%",
        display: "flex",
        alignItems: "center",
        gap: "12px",
    },
    progressTrack: {
        flex: 1,
        height: "8px",
        borderRadius: "4px",
        background: "#E5E7EB",
        overflow: "hidden",
        position: "relative",
    },
    progressFill: {
        height: "100%",
        borderRadius: "4px",
        position: "relative",
        overflow: "hidden",
    },
    progressShimmer: {
        position: "absolute",
        inset: 0,
        background: "linear-gradient(90deg, transparent, rgba(255,255,255,0.3), transparent)",
        animation: "deploy-shimmer 1.5s ease-in-out infinite",
    },
    progressLabel: {
        fontSize: "13px",
        fontWeight: "700",
        color: "#697386",
        minWidth: "36px",
        textAlign: "right",
    },
}
```

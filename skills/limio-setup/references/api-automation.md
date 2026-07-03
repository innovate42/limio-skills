# API Automation: tokens, the build pipeline, and what triggers what

Guidance for standalone scripts (page factories, migrations, CI) — not just the Storybook middleware.

## Minting a Bearer token for scripts

Client-credentials flow against the tenant:

```javascript
async function getToken({ baseUrl, clientId, clientSecret }) {
  const res = await fetch(`${baseUrl}/oauth2/token`, {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: new URLSearchParams({ grant_type: "client_credentials", client_id: clientId, client_secret: clientSecret }),
  })
  const { access_token, expires_in } = await res.json()
  return { token: access_token, expiresAt: Date.now() + (expires_in || 3600) * 1000 }
}
```

Rules that save batches:

- **Tokens last 1 hour.** Mint one per run; for runs that might exceed ~50 minutes, re-mint proactively (e.g. when `expiresAt - Date.now() < 60_000`) or on the first 401.
- **Design every script idempotently** (delete + recreate, or upsert semantics) so a token expiring mid-batch is fixed by simply re-running.
- Credentials live in `.limio.json` (gitignored) or environment variables — never in code.

## The component build pipeline (what happens after `git push`)

```
git push → GitHub → mirrored to AWS CodeCommit → CodeBuild compiles all components → available to Page Builder & published pages
```

Practical consequences:

- `GET /api/component/builds` reports the **most recent** build. When verifying your deploy, match on your commit hash (or on `startTime` after your push) rather than assuming the latest build is yours. Mirroring means local commit hashes may differ from the built hash — filtering by time is the robust option.
- Poll until `buildStatus` leaves `IN_PROGRESS`. Terminal states: `SUCCEEDED`, `FAILED`, `FAULT`, `TIMED_OUT`, `STOPPED`.
- **Check `logErrors` even on `SUCCEEDED`.** Individual components can fail to compile (syntax error, missing dependency) while the overall build passes — the failing component then serves its previous version or an error placeholder on live pages.

## What triggers what (pin this)

| You changed | Required step | NOT required |
|---|---|---|
| Component code / CSS / `limioProps` schema | Component build (git push, poll builds) | Page rebuild — pages pick up the new build automatically |
| Prop **values** on a page | Shop build + publish (`POST /api/shop/builds` → `POST /api/publish`) | Component build |
| A brand-new page URL (new tag) | One-time Page Builder bulk publish to register the route | — API publish handles all subsequent updates |
| Nothing (just want it live) | Publish an existing successful build | Rebuilding |

For creating and publishing pages programmatically, use the **limio-pages** skill.

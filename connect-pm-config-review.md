# Connect & Package Manager Config Review

Holistic review of `connect/rstudio-connect.gcfg` and `packagemanager/rstudio-pm.gcfg`
against the official Posit documentation (kapa) and product release notes (2024–2026),
plus lessons carried over from the recent Workbench `extras/conf` sync.

Date: 2026-06-18 · Branch: `review-connect-pm-configs`

---

## TL;DR — findings and disposition

| # | Item | Product | Severity | Disposition |
|---|------|---------|----------|-------------|
| 1 | Live `{{ REPLACEME }}` placeholders on **uncommented** required keys will hard-fail startup | Both | High | **Applied** |
| 2 | Package Manager config has **no authentication guidance at all** | PM | High | Deferred (doc only) |
| 3 | Adopt Workbench `TODO:` placeholder convention in place of `{{ REPLACEME }}` | Both | Medium | **Applied** |
| 4 | Postgres password should be a separate encrypted value, not inline in the URL | PM | Medium | **Applied** |
| 5 | Document PKCE (default-on, 2026.01) and HSTS max-age (2024.09) | Connect | Low | **Applied** (HSTS + PKCE comment) |
| 6 | `Lifetime = 12h` is stricter than product default (24h) — confirm intent | Connect | Low (info) | No change (intentional) |

> All `{{ REPLACEME }}` tokens have been removed from both files. Required keys are now
> commented out with example values + `TODO:` notes, so each file is valid as-shipped.
> Item 2 (PM auth template) was intentionally deferred and remains documented below only.

No **deprecated or removed** settings are present in either file — both are clean against
the 2024–2026 deprecation list (verified: `Server.RVersion*`, `Packages.External`,
`Server.ViewerKiosk`, `Metrics.*`, etc. are all absent).

---

## Verified-current (no change needed)

**Connect**
- `[Authentication] APIKeyAuth = true`, `Inactivity = 8h` — match product defaults.
- `[HTTPS] MinimumTLS = 1.2` — matches the documented recommendation. TLS 1.3 is the
  default *maximum* since 2024.06; RSA/3DES cipher suites already dropped from defaults
  upstream, so no cipher config is required here.
- `[OAuth2] OpenIDConnectIssuer`, `RequireUsernameClaim`, `GroupsAutoProvision` — all
  current setting names; none renamed/deprecated.
- `[Authorization] ViewersCanOnlySeeThemselves = true` — current, good hardening default.
- `[RPackageRepository "CRAN"]` block syntax — still correct.

**Package Manager**
- `[HTTPS]` (Certificate/Key/Listen/Permanent/MinimumTLS) and `[HttpRedirect]` — correct.
- `[Postgres] URL` with `sslmode=require` — valid; see #4 for the hardening nuance.
- `[Proxy]`, `[Git]`, `[Manifest]` air-gapped blocks — current.

---

## Findings & proposals

### 1. Live placeholders on required keys (High) — the key Workbench lesson

The recent Workbench sync deliberately **commented out** every setting that needs a
real value (e.g. `#auth-openid=1`, `#ssl-certificate=...`) and appended a `# TODO:`,
so a freshly-dropped config never half-activates a feature or crashes on a literal
placeholder. The Connect/PM files still ship the **opposite** pattern: required keys
are left uncommented with `{{ REPLACEME }}`, which fails validation/startup verbatim.

Affected **Connect** keys (uncommented today):
- `Server.Address`, `Server.SenderEmail`
- `[SMTP] Host`
- `[HTTPS] Certificate`, `Key`
- `[R] Executable` (×2), `[Python] Executable` (×2), `[Quarto] Executable` (×2)
- `[OAuth2] ClientId`, `ClientSecret`, `OpenIDConnectIssuer`

Affected **PM** keys (uncommented today):
- `Server.Address`
- `[HTTPS] Certificate`, `Key`

> **Proposal:** comment these out and pair each with a `# TODO:` note (mirroring the
> Workbench convention), so the baseline file is valid as-shipped and features are
> opt-in. `Enabled = true` toggles (Python/Quarto) can stay, but their `Executable`
> lines should be commented until a real path is supplied.

### 2. Package Manager has no authentication section (High)

Per docs, PM supports **API Token** auth (default, easiest), **OpenID Connect**
(`[OpenIDConnect]` — Okta/Entra/Auth0/Ping/generic), and **OIDC Identity Federation**
(`[IdentityFederation]`, e.g. GitHub Actions for CI/CD). Repos are unauthenticated by
default; `Authentication.NewReposAuthByDefault` and `Authentication.RequireAuthForUI`
control opt-in. The current config documents **none** of this.

> **Proposal:** add commented `[Authentication]` + `[OpenIDConnect]` template blocks
> with the recommended production guidance (token for simple installs, OIDC for
> enterprise SSO, group→scope mapping), consistent with how Connect's auth is templated.

### 3. Adopt `# TODO:` convention over `{{ REPLACEME }}` (Medium)

Workbench standardized on concrete example values + inline `# TODO:` annotations and
dropped the `{{ REPLACEME }}` token. Note: `.gcfg` uses `;`/`#` for comments, so the
Connect/PM comment *character* (`;`) is already idiomatic and should stay — this is
only about the **placeholder token style**, applied alongside #1.

### 4. Postgres password handling (Medium, PM)

The template embeds credentials inline:
`postgres://username:password@hostname:5432/db?sslmode=require`. Docs recommend a
separate **base64-encrypted** `Postgres.Password` value and `sslmode=verify-full` for
production. Worth updating the example and adding an `rstudio-pm encrypt` note (mirrors
the encrypted-secret guidance Workbench now carries for `openid-client-secret`).

### 5. Newer Connect options worth a comment (Low)

- **PKCE** — `OAuth2.UsePKCE` is **on by default since 2026.01** (backported to OAuth in
  2024.11). No action required; worth a one-line comment noting it's auto-enabled and
  only set `UsePKCE = false` for IdPs that don't support it.
- **HSTS max-age** — `HTTPS.StrictTransportSecurityMaxAge` added in 2024.09 to tune the
  HSTS `max-age` when `Permanent = true`. Optional commented addition next to `Permanent`.

### 6. Session lifetime is stricter than default (Low / informational)

`[Authentication] Lifetime = 12h` vs the product default `24h` (`Inactivity = 8h`
matches). This is a reasonable opinionated hardening choice — flagging only so it's a
conscious decision, not an accident.

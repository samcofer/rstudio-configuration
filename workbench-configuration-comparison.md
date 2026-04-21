# Posit Workbench Configuration Comparison: Defaults vs. Enhanced

This document walks through every meaningful difference between the stock Posit Workbench default configuration files (what ships out of the box) and the enhanced configuration templates in this repo. I've broken things down file-by-file, called out the functional changes, and then covered the broader pattern around comment verbosity. At the end, there's a section on optional CLI-driven walkthrough workflows that could complement these config files to help folks get through the more complex setup scenarios (SSL, SAML, OIDC, SCIM, load balancing, Slurm, etc.) without guessing at values.

Full disclosure, the "defaults" referenced here are the files in the `workbench-defaults/` directory, and the "enhanced" versions are in the `workbench/` directory.

---

## Table of Contents

1. [Global Changes: Comment Verbosity and Documentation Style](#global-changes-comment-verbosity-and-documentation-style)
2. [File-by-File Comparison](#file-by-file-comparison)
   - [rserver.conf](#rserverconf)
   - [rsession.conf](#rsessionconf)
   - [launcher.conf](#launcherconf)
   - [jupyter.conf](#jupyterconf)
   - [vscode.conf](#vscodeconf)
   - [positron.conf](#positronconf)
   - [positron-user-settings.json](#positron-user-settingsjson)
   - [vscode-user-settings.json](#vscode-user-settingsjson)
3. [Files Only in Enhanced Configuration](#files-only-in-enhanced-configuration)
4. [Files Only in Defaults](#files-only-in-defaults)
5. [Optional CLI Walkthrough Arguments](#optional-cli-walkthrough-arguments)
   - [Configure SSL/TLS](#configure-ssltls)
   - [Configure SAML SSO](#configure-saml-sso)
   - [Configure OpenID Connect (OIDC)](#configure-openid-connect-oidc)
   - [Configure JIT User Provisioning](#configure-jit-user-provisioning)
   - [Configure SCIM User Provisioning](#configure-scim-user-provisioning)
   - [Configure Load Balancing](#configure-load-balancing)
   - [Configure Slurm Launcher](#configure-slurm-launcher)

---

## Global Changes: Comment Verbosity and Documentation Style

The single biggest difference across all files is the comment strategy. The defaults ship with minimal, terse comments. The enhanced configs follow a consistent documentation pattern throughout:

1. **File header**: Every file starts with a `# Posit Workbench [Component] Configuration File` header and a `# Full Configuration Reference:` link to the relevant docs page.

2. **Section dividers**: Each logical group of settings gets a visual block:
   ```
   #-----------------------------------------------------------------------------------------#
   # Section Title
   #
   # https://docs.posit.co/...
   # Multi-line explanation of what these settings do, why you'd change them,
   # and cross-references to related settings in other files.
   #-----------------------------------------------------------------------------------------#
   ```

3. **Inline TODO markers**: Anywhere a customer-specific value is needed, there's a readable example value (e.g., `https://idp.your-domain.com`, `3.XX.XX`, `distro-version-here`) with a `# TODO:` comment explaining what to replace.

4. **Documentation URLs**: Every section links to the specific admin guide page, not just the top-level reference. This makes it so someone new to Workbench can jump straight to the right docs without digging.

5. **Cross-file references**: Comments explicitly call out when a setting must match a corresponding value in another file (e.g., `launcher-port` in `rserver.conf` must match `port` in `launcher.conf`).

The defaults have generic comments like `# Server Configuration File` and `# The server user here must match the server user configured in launcher.conf.` The enhanced versions are significantly more verbose, but with purpose: they function as inline documentation for the admin deploying Workbench.

---

## File-by-File Comparison

### rserver.conf

This is by far the largest change. The defaults ship with ~35 lines. The enhanced version is ~247 lines. Here's what's different functionally:

**Settings removed:**

| Setting | Default Value | Notes |
|---------|--------------|-------|
| `server-user=rstudio-server` | Removed | Not needed in `rserver.conf`, it's inferred from the launcher config |
| `launcher-use-ssl=0` | Removed | Replaced with a commented `#launcher-use-ssl=1` in the SSL section |
| `load-balancing-enabled=0` | Removed | Replaced with a commented `#load-balancing-enabled=1` in the HA section |

**Settings added (active):**

```ini
# PAM session profile (default is "su")
auth-pam-sessions-profile=su

# Admin Dashboard (not enabled by default in stock config)
admin-enabled=1
admin-group=posit-admins
admin-superuser-group=posit-super-admin
admin-monitor-log-use-server-time-zone=1
```

The Admin Dashboard settings are the most impactful active change. In the defaults, the admin dashboard is effectively off. The enhanced config enables it and sets up group-based access with `posit-admins` and `posit-super-admin` groups. The `admin-monitor-log-use-server-time-zone=1` setting makes log timestamps in the dashboard match the server's timezone instead of UTC, which is a quality-of-life improvement for troubleshooting.

**Settings added (commented, ready to uncomment):**

The enhanced config includes fully documented, commented-out sections for:

- **SSL/TLS**: `ssl-enabled`, `ssl-certificate`, `ssl-certificate-key`, `ssl-protocols`, `ssl-hsts-max-age`, `ssl-hsts-include-subdomains`
- **SAML SSO**: `auth-saml`, `auth-saml-sp-attribute-username`, `auth-saml-idp-post-binding`, `auth-saml-metadata-url`, `auth-saml-metadata-path`
- **OpenID Connect**: `auth-openid`, `auth-openid-issuer`, `auth-openid-username-claim`
- **JIT User Provisioning**: `user-provisioning-enabled`, `user-provisioning-register-on-first-login`, `user-provisioning-start-uid`, `group-provisioning-start-gid`
- **SCIM Provisioning**: `user-provisioning-enabled` (with docs links to Okta and Azure setup)
- **Session Diagnostics**: `rsession-diagnostics-dir`, `rsession-diagnostics-enabled`, `rsession-diagnostics-strace-enabled`
- **Security Hardening**: `auth-required-user-group`, `auth-timeout-minutes`, `www-enable-origin-check`, `www-allow-origin`, `www-same-site`, `www-frame-origin`
- **Load Balancing**: `load-balancing-enabled`
- **Health Check Endpoint**: `server-health-check-enabled`
- **Auditing**: `audit-data-path`, `audit-r-sessions`, `audit-r-console` and associated format/retention settings

---

### rsession.conf

The defaults ship this file essentially empty (just a comment header). The enhanced version adds:

**Settings added (active):**

```ini
session-timeout-minutes=120
session-timeout-kill-hours=0
```

`session-timeout-minutes=120` suspends inactive sessions after 2 hours. `session-timeout-kill-hours=0` means suspended sessions are never forcibly killed, they stay suspended indefinitely. These are sensible production defaults that prevent runaway session accumulation while not destroying user state.

**Settings added (commented, ready to uncomment):**

```ini
# Posit Connect integration
#default-rsconnect-server=https://connect.your-domain.com

# Lock down CRAN repo editing in the UI
#allow-r-cran-repos-edit=0

# Security hardening
#allow-external-publish=0
#restrict-directory-view=1
```

The `default-rsconnect-server` setting pre-configures the Connect publishing target so users don't have to manually add it. `allow-r-cran-repos-edit=0` enforces Package Manager usage through the GUI (users can still change repos from the R console). The security settings disable publishing to external services like RPubs/shinyapps.io and restrict file browser navigation to home directories only.

---

### launcher.conf

**Settings changed:**

| Setting | Default | Enhanced | Impact |
|---------|---------|----------|--------|
| `address` | `localhost` | `127.0.0.1` | Functionally equivalent, but explicit IPv4 avoids any ambiguity with dual-stack DNS resolution |

**Settings added (active):**

```ini
enable-cgroups=1
```

This is a significant change. Enabling cgroups v2 resource limits means sessions launched through the Local plugin will actually have CPU and memory constraints enforced at the OS level. Without this, the resource profiles in `launcher.local.resources.conf` are suggestions, not limits. With cgroups enabled, they're hard limits. Requires cgroups v2 support on the host.

**Settings added (commented):**

- Debug logging: `enable-debug-logging=1`
- Custom paths: `scratch-path`, `logging-dir`
- Launcher SSL: `enable-ssl`, `certificate-file`, `certificate-key-file`

**Settings removed:**

The default's commented-out Kubernetes and Slurm cluster examples are removed. The enhanced config focuses on the Local cluster and leaves Kubernetes/Slurm configuration to dedicated walkthrough workflows (see [CLI section](#optional-cli-walkthrough-arguments) below).

---

### jupyter.conf

**Settings changed:**

| Setting | Default | Enhanced | Impact |
|---------|---------|----------|--------|
| `notebooks-enabled` | `0` | Commented out (`# notebooks-enabled=1`) | Classic Notebooks disabled by default in both, but enhanced makes it easy to enable |
| `session-cull-minutes` | `0` (disabled) | `120` (2 hours) | Idle Jupyter sessions are now auto-terminated after 2 hours instead of running forever |

**Settings added:**

```ini
#jupyter-exe=/opt/python/3.XX.XX/bin/python
```

The `jupyter-exe` TODO gives admins a clear prompt to set the right Python path.

---

### vscode.conf

**Settings changed:**

| Setting | Default | Enhanced | Impact |
|---------|---------|----------|--------|
| `args` | `--host=0.0.0.0 ` (trailing space) | `--host=0.0.0.0` (no trailing space) | Cosmetic cleanup |

**Settings added:**

```ini
user-data-dir=~/.vscode-server
```

`user-data-dir=~/.vscode-server` explicitly sets where VS Code user data lives. This is the default behavior, but making it explicit means admins can see and change it without hunting through docs.

**Settings added (commented):**

```ini
#session-timeout-kill-hours=24
```

---

### positron.conf

The default is a single line: `enabled=1`. The enhanced version keeps that same active setting but adds documented, commented sections for session timeout and cluster configuration:

```ini
enabled=1

#session-timeout-kill-hours=0
#default-session-cluster=Local
```

No functional change, just documentation.

---

### positron-user-settings.json

**Settings unchanged:**

All the original settings (terminal profile, auto-update, Quarto path, Python interpreter exclusions, Conda disable) are preserved identically.

**Metadata added:**

```json
{
  "_comment": "Positron Default User Settings Template - /etc/rstudio/positron-user-settings.json",
  "_documentation": "https://docs.posit.co/ide/server-pro/admin/positron_sessions/user_settings.html",
  "_note": "These settings are merged into each user's Positron settings.json on first launch. Users CAN override these after initial merge..."
}
```

The `_comment`, `_documentation`, and `_note` fields are JSON metadata that Positron ignores but serve as inline documentation for whoever is editing the file. This clarifies the important distinction that these are _default_ settings (users can override) vs. _enforced_ settings (users cannot override, configured separately in `positron-enforced-settings.json`).

---

### vscode-user-settings.json

**Settings changed:**

| Setting | Default | Enhanced | Impact |
|---------|---------|----------|--------|
| `terminal.integrated.defaultProfile.linux` | `"/bin/bash"` | `"bash"` | Uses shell name instead of full path |

**Settings added:**

```json
{
  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 1000
}
```

`files.autoSave` with a 1-second delay is particularly important because VS Code sessions are _killed_ (not suspended) when they time out. Without auto-save, users can lose unsaved work.

Same JSON metadata fields (`_comment`, `_documentation`, `_note`) are added as in the Positron settings.

---

## Files Only in Enhanced Configuration

These files exist in `workbench/` but have no counterpart in the defaults. Each one addresses a configuration area that the stock install either doesn't ship a file for or leaves entirely to the admin to create from scratch.

---

### launcher-env

Environment variables injected into every launcher session. The defaults don't ship this file at all, so Python's `requests` library (and anything that uses it) won't trust the system CA bundle in self-signed certificate environments unless the admin knows to set this up manually.

The enhanced config includes commented entries for both RHEL and Ubuntu/Debian:

```ini
# RHEL Environment setting to make Python use the system certificate store,
# useful in self-signed certificates scenarios
# JobType: any
# Workbench: any
# Environment: REQUESTS_CA_BUNDLE=/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem


# Ubuntu/Deb Environment setting to make Python use the system certificate store,
# useful in self-signed certificates scenarios
# JobType: any
# Workbench: any
# Environment: REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
```

To activate, uncomment the `Environment:` line for your distro. The `JobType: any` and `Workbench: any` directives mean the variable applies to all session types across all Workbench nodes.

**Reference:** https://docs.posit.co/ide/server-pro/admin/job_launcher/configuration.html#launcher-env-conf

---

### launcher.local.profiles.conf

Per-user and per-group resource limits for the Local Job Launcher plugin. Without this file, all users get unlimited access to system resources (bounded only by what the OS allows). With it, you get tiered resource governance.

Sections are processed top-to-bottom, with later matches overriding earlier ones. Section types: `[*]` (all users), `[@groupname]` (Unix groups), `[username]` (individual users).

**Active settings (global defaults):**

```ini
[*]
default-cpus=1.0
default-mem-mb=2048
max-cpus=8
max-mem-mb=16384
resource-profiles=default,small,medium,large
allow-custom-resources=0
```

This gives every user 1 CPU and 2GB RAM by default, caps them at 8 CPUs and 16GB, and restricts them to the named resource profiles defined in `launcher.local.resources.conf`. `allow-custom-resources=0` means users cannot enter arbitrary CPU/memory values, they must pick from the defined profiles.

**Commented examples:**

```ini
# Data science team with moderate resources
# [@data-science]
# default-cpus=1.0
# default-mem-mb=2048
# max-cpus=8.0
# max-mem-mb=16384
# resource-profiles=default,small,medium,large
# allow-custom-resources=1

# Power users with full profile access and custom resources
# [@posit-admins]
# default-cpus=1.0
# default-mem-mb=2048
# resource-profiles=default,small,medium,large
# allow-custom-resources=1

# Example individual user override
# [jsmith]
# default-cpus=1.0
# default-mem-mb=2048
# max-cpus=24
# max-mem-mb=65536
# resource-profiles=default,medium
# allow-custom-resources=1
```

**Reference:** https://docs.posit.co/ide/server-pro/admin/job_launcher/local_plugin.html#local-profiles

---

### launcher.local.resources.conf

Named resource profiles (CPU + memory bundles) that users select from the session launcher UI. These are the profiles referenced in `launcher.local.profiles.conf` via the `resource-profiles=` setting. Without this file, users see no resource options when starting a session.

**Active settings:**

```ini
[default]
name = Default (1 CPUs, 4GB RAM)
cpus=1
mem-mb=4096

[small]
name = Small (1 CPU, 2GB RAM)
cpus=1
mem-mb=2048

[medium]
name = Medium (2 CPUs, 8GB RAM)
cpus=4
mem-mb=8192

[large]
name = Large (4 CPUs, 16GB RAM)
cpus=8
mem-mb=16384
```

Note that the `name` field is what users see in the session launcher dropdown. The section header (e.g., `[default]`, `[medium]`) is the internal identifier referenced in the profiles config. These resource limits are only enforced at the OS level if `enable-cgroups=1` is set in `launcher.conf`.

**Reference:** https://docs.posit.co/ide/server-pro/admin/job_launcher/local_plugin.html#local-resource-profiles

---

### openid-client-secret

Stores the OIDC client credentials that Workbench uses to authenticate with the identity provider. The defaults don't ship this file, so admins configuring OIDC have to create it from scratch and figure out the format, permissions, and encryption steps on their own.

**Template contents:**

```ini
# client-id=your-client-id-here # TODO: Update with your OpenID Connect client ID
# client-secret=your-encrypted-secret-here # TODO: Update with your encrypted OpenID Connect client secret
```

**Setup steps:**

1. Get the client ID and client secret from your IdP (Okta, Azure Entra ID, etc.)
2. Encrypt the client secret: `sudo rstudio-server encrypt-password` (paste the secret when prompted)
3. Uncomment both lines and fill in the values
4. Set file permissions: `sudo chmod 600 /etc/rstudio/openid-client-secret`

The encrypted secret replaces the plaintext value, so the file never contains the raw credential at rest.

**Reference:** https://docs.posit.co/ide/server-pro/admin/authenticating_users/openid_connect_authentication.html#configuring-workbench-for-openid-connect

---

### positron-enforced-settings.json

Template for Positron IDE settings that users _cannot_ override. This is distinct from `positron-user-settings.json` (which sets defaults that users can change). The file ships empty (no actual settings enforced), but includes JSON metadata explaining how to use it:

```json
{
  "_comment": "Positron Enforced Settings - /etc/rstudio/positron-enforced-settings.json",
  "_documentation": "https://docs.posit.co/ide/server-pro/admin/positron_sessions/user_settings.html#enforced-settings",
  "_note": "These settings are ENFORCED and users CANNOT override them. Register this file in /etc/rstudio/profiles using 'positron-enforced-settings = /etc/rstudio/positron-enforced-settings.json' under the [*] section. Restart Workbench after changes: sudo rstudio-server restart. Example settings: extensions.autoUpdate, extensions.autoCheckUpdates, telemetry.telemetryLevel"
}
```

To actually enforce settings, add them as key-value pairs in this JSON file, then make sure the `profiles` file (see below) registers it. Common use cases: locking down extension auto-update, disabling telemetry, enforcing a specific theme or editor configuration for compliance.

**Reference:** https://docs.posit.co/ide/server-pro/admin/positron_sessions/user_settings.html#enforced-settings

---

### profiles

Workbench profiles configuration that maps enforced settings files to user/group scopes. This is the glue that connects `positron-enforced-settings.json` to actual users. Without it, the enforced settings file is ignored even if it exists.

**Active settings:**

```ini
[*]
positron-enforced-settings = /etc/rstudio/positron-enforced-settings.json
```

The `[*]` section applies to all users. The file also includes commented examples for group-based and per-user scoping:

```ini
# Group-based enforced settings
#[@data-science]
#positron-enforced-settings = /etc/rstudio/data-science-enforced-settings.json

# Per-user enforced settings
#[jsmith]
#positron-enforced-settings = /etc/rstudio/jsmith-enforced-settings.json
```

Sections are processed top-to-bottom with later matches overriding earlier ones, same precedence model as the launcher profiles. Restart Workbench after changes: `sudo rstudio-server restart`.

**Reference:** https://docs.posit.co/ide/server-pro/admin/positron_sessions/user_settings.html#enforced-settings

---

### repos.conf

R package repository configuration that sets the default CRAN mirror for all R sessions. Without this file, R sessions use whatever CRAN mirror is configured in the user's `.Rprofile` or the system default (typically `https://cloud.r-project.org`). With it, you can point everyone at an internal Posit Package Manager instance or the public Package Manager for pre-built Linux binaries.

**Commented settings:**

```ini
# Posit Public Package Manager
#CRAN=https://packagemanager.posit.co/cran/__linux__/distro-version-here/latest # TODO: Replace distro-version-here with your Linux distro/version, eg: ubuntu22

# Internal Package Manager
#CRAN=https://packagemanager.company.com/cran/__linux__/ubuntu22/latest
```

The `__linux__` placeholder in the URL is resolved by Package Manager to the correct binary path for your distro. Supported distro values: `centos7`, `centos8`, `rhel9`, `ubuntu18`, `ubuntu20`, `ubuntu22`, `ubuntu24`, `opensuse15`, `opensuse42`, `debian11`, `debian12`.

Pointing CRAN at Package Manager with binary repos is one of the highest-impact quality-of-life improvements for R users, since it eliminates compilation time for most package installs.

**Reference:** https://solutions.posit.co/envs-pkgs/rsw_defaults/

---

## Files Only in Defaults

These files ship with the product but are not included in the enhanced configuration:

| File | Purpose | Why Excluded |
|------|---------|--------------|
| `database.conf` | PostgreSQL/SQLite database configuration | Only needed for load-balanced deployments, covered in the CLI walkthrough |
| `logging.conf` | Logging level and output configuration | Generally left at defaults unless actively troubleshooting |
| `notifications.conf` | User session notification messages | Deployment-specific, no general-purpose template makes sense |
| `r-versions` | R installation path registry | Machine-specific, auto-detected in most cases |
| `vscode.extensions.conf` | VS Code extension auto-install list | Ships with Quarto/Shiny/Publisher already, usually left as-is |
| `launcher.pem` / `launcher.pub` | Launcher encryption keypair | Auto-generated, should not be templated |
| `fonts/` / `themes/` | UI customization assets | Deployment-specific branding |

---

## Optional CLI Walkthrough Arguments

The `rstudio-server` CLI already provides a set of administrative commands. Below are proposed interactive walkthrough workflows that could be added as `rstudio-server` subcommands to complement the configuration templates in this repo. Each walkthrough would prompt the admin for site-specific values and modify the relevant configuration files.

For reference, here are the existing `rstudio-server` CLI commands:

| Category | Commands |
|----------|----------|
| Server Management | `start`, `stop`, `restart`, `reload`, `status`, `offline`, `online` |
| Diagnostics | `verify-installation`, `test-config`, `run-diagnostics`, `version` |
| User Management | `add-user`, `list-users`, `lock-user`, `unlock-user`, `set-admin` |
| SCIM Tokens | `user-service generate-token`, `user-service list-tokens`, `user-service revoke-token` |
| Sessions | `active-sessions`, `suspend-session`, `suspend-all`, `force-suspend-session`, `force-suspend-all`, `kill-session`, `kill-all` |
| Load Balancing | `list-nodes`, `node-status`, `reset-cluster`, `delete-node` |
| Security | `encrypt-password` |

---

### Configure SSL/TLS

**Proposed command:** `rstudio-server configure-ssl`

**What it would do:**

This walkthrough configures HTTPS termination directly on Workbench. It would prompt for certificate and key paths, validate file permissions, update the relevant config files, and optionally configure HSTS.

**Interactive prompts:**

1. Path to SSL certificate (PEM format): `/path/to/posit.crt`
2. Path to SSL private key (no passphrase): `/path/to/posit.key`
3. Enable HSTS? (y/n)
4. Restrict to TLS 1.2+ only? (y/n)
5. Also enable SSL for the Launcher? (y/n)

**Configuration file changes:**

`/etc/rstudio/rserver.conf`:
```ini
ssl-enabled=1
ssl-certificate=/etc/rstudio/posit.crt
ssl-certificate-key=/etc/rstudio/posit.key
ssl-protocols=TLSv1.2 TLSv1.3
ssl-hsts-max-age=31536000
ssl-hsts-include-subdomains=1

# Update callback address to use HTTPS
launcher-sessions-callback-address=https://your.workbench.fqdn

# Enable SSL for launcher communication
launcher-use-ssl=1
```

`/etc/rstudio/launcher.conf` (if launcher SSL enabled):
```ini
enable-ssl=1
certificate-file=/etc/rstudio/posit.crt
certificate-key-file=/etc/rstudio/posit.key
```

**Validation steps the walkthrough would perform:**

- Verify certificate file exists and is PEM format
- Verify key file exists and has no passphrase
- Set ownership to `rstudio-server:rstudio-server`
- Set `.crt` to `644`, `.key` to `600`
- Verify the certificate CN/SAN matches the hostname used in `launcher-sessions-callback-address`
- Run `rstudio-server test-config` to validate
- Prompt for `rstudio-server restart`

**Important notes:**

- Workbench does not support passphrase-protected SSL certificates. If the key has a passphrase, the walkthrough should offer to strip it with `openssl rsa -in original.key -out new.key`.
- If intermediate certificates are needed, they must be concatenated into the `.crt` file: `cat intermediate.crt >> posit.crt`.
- If load balancing is enabled or being enabled, SSL changes require `rstudio-server reset-cluster`.

---

### Configure SAML SSO

**Proposed command:** `rstudio-server configure-saml`

**What it would do:**

Walk through SAML IdP integration, including metadata configuration, attribute mapping, and post-binding settings. SAML becomes the exclusive authentication method once enabled, PAM login is disabled.

**Interactive prompts:**

1. IdP metadata source: URL or local file?
   - If URL: `https://idp.example.com/saml/metadata`
   - If file: `/etc/rstudio/metadata.xml`
2. Username attribute in SAML assertions (default: `NameID`): e.g., `NameID`, `Username`, `email`
3. IdP requires HTTP POST binding? (y/n) (common for Azure AD)
4. Enable JIT user provisioning? (y/n) (see [JIT section](#configure-jit-user-provisioning))

**Configuration file changes:**

`/etc/rstudio/rserver.conf`:
```ini
auth-saml=1
auth-saml-sp-attribute-username=NameID

# For internet-connected systems
auth-saml-metadata-url=https://idp.example.com/saml/metadata

# OR for air-gapped deployments
# auth-saml-metadata-path=/etc/rstudio/metadata.xml

# Required for Azure AD and many other IDPs
auth-saml-idp-post-binding=1
```

**Post-configuration steps:**

- The walkthrough would output the Workbench SP metadata URL for the admin to register with their IdP: `https://<workbench-hostname>/saml/metadata`
- If JIT was selected, the walkthrough chains into the JIT configuration flow
- Run `rstudio-server test-config` and `rstudio-server restart`

**Additional SAML attribute mappings (for JIT provisioning):**

| rserver.conf Setting | Default | Description |
|---------------------|---------|-------------|
| `auth-saml-sp-attribute-username` | `Username` | Maps to local account username |
| `auth-saml-sp-attribute-email` | None | Email address |
| `auth-saml-sp-attribute-name` | None | Full name |
| `auth-saml-sp-attribute-groups` | None | Group membership |
| `auth-saml-sp-attribute-posix-id` | None | POSIX UID |
| `auth-saml-sp-attribute-homedir` | None | Home directory path |

---

### Configure OpenID Connect (OIDC)

**Proposed command:** `rstudio-server configure-oidc`

**What it would do:**

Walk through OIDC integration with an identity provider. Like SAML, OIDC becomes the exclusive auth method. The walkthrough handles the issuer URL, client credentials, and username claim configuration.

**Interactive prompts:**

1. OpenID issuer URL (must be HTTPS): `https://op.example.com`
2. Client ID from your IdP registration
3. Client secret from your IdP registration (will be encrypted)
4. Username claim (default: `preferred_username`): e.g., `preferred_username`, `email`, `sub`
5. Enable JIT user provisioning? (y/n)

**Configuration file changes:**

`/etc/rstudio/rserver.conf`:
```ini
auth-openid=1
auth-openid-issuer=https://op.example.com
auth-openid-username-claim=preferred_username
```

`/etc/rstudio/openid-client-secret`:
```ini
client-id=your-client-id-here
client-secret=<encrypted-output-from-encrypt-password>
```

**Walkthrough actions:**

- Encrypt the client secret automatically using `rstudio-server encrypt-password`
- Write the `openid-client-secret` file with `600` permissions
- Output the callback URL for the admin to register with their IdP: `https://<workbench-hostname>/openid/callback`
- Validate that the issuer URL has a reachable `/.well-known/openid-configuration` endpoint
- Run `rstudio-server test-config` and prompt for restart

**Important notes:**

- The username claim value must match a valid Linux account username (max 32 characters, alphanumeric, underscores, dashes).
- Users still need local system accounts. OIDC handles authentication only, not account creation (unless paired with JIT or SCIM).

---

### Configure JIT User Provisioning

**Proposed command:** `rstudio-server configure-jit`

**What it would do:**

Enable automatic local account creation on first SSO login. Requires SAML or OIDC to already be configured. The walkthrough sets the UID/GID ranges and configures PAM for home directory creation.

**Interactive prompts:**

1. Starting UID for provisioned users (default: `2000`)
2. Starting GID for provisioned groups (default: `2000`)
3. Configure `pam_mkhomedir` for automatic home directory creation? (y/n)

**Configuration file changes:**

`/etc/rstudio/rserver.conf`:
```ini
user-provisioning-enabled=1
user-provisioning-register-on-first-login=1
user-provisioning-start-uid=2000
group-provisioning-start-gid=2000
```

If `pam_mkhomedir` is selected, the walkthrough would add to the PAM profile (e.g., `/etc/pam.d/su` since `auth-pam-sessions-profile=su`):
```
session required pam_mkhomedir.so skel=/etc/skel umask=0022
```

**Important notes:**

- JIT and SCIM are mutually exclusive. If SCIM is already enabled, the walkthrough should warn and exit.
- JIT does not support user deactivation or lifecycle management. If those are needed, use SCIM instead.
- The UID/GID ranges should not overlap with existing system or LDAP/AD ranges.

---

### Configure SCIM User Provisioning

**Proposed command:** `rstudio-server configure-scim`

**What it would do:**

Enable SCIM-based user lifecycle management driven by an external IdP (Okta, Azure Entra ID, etc.). SCIM supports full create/update/deactivate flows, unlike JIT. Requires HTTPS with a CA-signed certificate.

**Interactive prompts:**

1. SCIM token name (descriptive, e.g., "Okta Production")
2. Token expiration: default (365 days) or no expiry?
3. Starting UID for provisioned users (default: `2000`)
4. Starting GID for provisioned groups (default: `2000`)

**Configuration file changes:**

`/etc/rstudio/rserver.conf`:
```ini
user-provisioning-enabled=1
user-provisioning-start-uid=2000
group-provisioning-start-gid=2000
```

**Walkthrough actions:**

- Verify that SSL is enabled (SCIM requires HTTPS with a CA-signed certificate)
- Generate a SCIM bearer token: `sudo rstudio-server user-service generate-token "Okta Production"`
- Display the token value (once, it cannot be retrieved again)
- Display the SCIM endpoint URL: `https://<workbench-hostname>/scim/v2`
- Output IdP-specific setup instructions based on the IdP type:

**For Okta:**
1. Navigate to your application > Provisioning > To App > Edit
2. Enable Create Users, Update User Attributes, Deactivate Users
3. Set SCIM Connector Base URL to `https://<workbench-hostname>/scim/v2`
4. Set Authentication Mode to HTTP Header
5. Paste the bearer token into the Authorization field
6. Click Test Connector Configuration

**For Azure Entra ID:**
1. Navigate to Enterprise Application > Provisioning
2. Set Provisioning Mode to Automatic
3. Set Tenant URL to `https://<workbench-hostname>/scim/v2`
4. Paste the bearer token as Secret Token
5. Click Test Connection
6. Under Mappings, ensure `userPrincipalName` maps to `userName`
7. Set Provisioning Status to On

**Token management commands:**
```bash
# List all tokens
sudo rstudio-server user-service list-tokens

# Revoke a specific token
sudo rstudio-server user-service revoke-token "Okta Production"

# Revoke all tokens
sudo rstudio-server user-service revoke-token --all
```

---

### Configure Load Balancing

**Proposed command:** `rstudio-server configure-load-balancing`

**What it would do:**

Walk through setting up a multi-node Workbench deployment with PostgreSQL and shared storage. This is the most complex walkthrough because it touches the most files and has the most prerequisites.

**Interactive prompts:**

1. PostgreSQL connection details:
   - Host: `postgres.example.com`
   - Port: `5432`
   - Database name: `rstudio`
   - Username: `rstudio`
   - Password: (will be encrypted)
   - SSL mode: `allow`, `require`, `verify-full`
2. Shared storage path (NFS mount): `/nfs/workbench/shared-storage`
3. This node's hostname/FQDN: `workbench-node1.example.com`
4. Load balancing strategy: `sessions` (default) or `round-robin`

**Configuration file changes:**

`/etc/rstudio/rserver.conf`:
```ini
load-balancing-enabled=1
server-shared-storage-path=/nfs/workbench/shared-storage
```

`/etc/rstudio/database.conf`:
```ini
provider=postgresql
password=<encrypted-output-from-encrypt-password>
connection-uri=postgresql://rstudio@postgres.example.com:5432/rstudio?sslmode=allow
```

`/etc/rstudio/load-balancer`:
```ini
balancer=sessions
www-host-name=workbench-node1.example.com
```

`/etc/rstudio/launcher.conf` (update address to listen on all interfaces):
```ini
address=0.0.0.0
```

**Validation steps:**

- Verify the shared storage path is mounted and writable
- Test PostgreSQL connectivity with the provided credentials
- Encrypt the database password via `rstudio-server encrypt-password`
- Set `database.conf` permissions to `600`
- Verify the hostname resolves from other nodes
- Run `rstudio-server test-config`
- Prompt for restart

**Prerequisites the walkthrough would verify:**

- PostgreSQL database exists and is accessible (SQLite is not supported for load-balanced deployments)
- Shared storage is mounted (POSIX-compliant: NFS, EFS, Azure Files)
- User accounts and UIDs are synchronized across all nodes
- Clocks are synchronized (NTP)
- Same Workbench version on all nodes

**Important notes:**

- More than two nodes per cluster requires an Advanced license.
- If SSL is enabled or being enabled alongside load balancing, run `rstudio-server reset-cluster` after changes.
- The `load-balancer` file is the only config file that should differ between nodes (each node has its own `www-host-name`).

---

### Configure Slurm Launcher

**Proposed command:** `rstudio-server configure-slurm`

**What it would do:**

Add a Slurm cluster to the Job Launcher configuration and create the necessary Slurm plugin config files. This assumes Slurm is already installed and the Workbench server has access to the Slurm control plane.

**Interactive prompts:**

1. Slurm cluster name (default: `Slurm`)
2. Slurm service user (must have cluster-admin privileges): `slurm`
3. Path to Slurm binaries (if not on PATH): `/usr/bin`
4. Enable GPU requests? (y/n)
   - If yes, GPU types (comma-separated): `tesla,v100`
5. Allowed partitions (comma-separated, or blank for all): `general,gpu,highmem`
6. Default CPUs for sessions: `1`
7. Default memory (MB) for sessions: `2048`
8. Max CPUs per user: `16`
9. Max memory (MB) per user: `32768`
10. Job expiry hours (default: `24`): how long completed jobs stay visible

**Configuration file changes:**

Append to `/etc/rstudio/launcher.conf`:
```ini
[cluster]
name=Slurm
type=Slurm
```

Create `/etc/rstudio/launcher.slurm.conf`:
```ini
slurm-service-user=slurm
slurm-bin-path=/usr/bin
job-expiry-hours=24
enable-gpus=1
gpu-types=tesla,v100
```

Create `/etc/rstudio/launcher.slurm.profiles.conf`:
```ini
[*]
default-cpus=1
default-mem-mb=2048
max-cpus=16
max-mem-mb=32768
allowed-partitions=general,gpu,highmem
```

**Validation steps:**

- Verify `scontrol`, `sbatch`, `squeue` are accessible at the specified path
- Verify the Slurm service user exists and can run Slurm commands
- Verify the launcher's `server-user` can sudo to the Slurm service user (or is in the right group)
- Run `scontrol show config` to verify Slurm cluster connectivity
- Run `rstudio-server test-config`
- Prompt for launcher restart: `sudo systemctl restart rstudio-launcher`

**Important notes:**

- The Slurm plugin runs commands as `slurm-service-user`, which must have admin-level Slurm privileges. This is typically the `slurm` user or a user in the `slurm` group.
- If Workbench is in a load-balanced configuration, the Slurm cluster config must be identical on all nodes, but the launcher only needs to run on nodes that will submit jobs.
- The `allowed-partitions` setting in the profiles file controls which Slurm partitions users can target. Leave it empty to allow all partitions.
- `allow-requeue=0` is the default and recommended for Workbench sessions, since re-queued sessions can cause unexpected behavior.

---

## Summary

The enhanced configuration templates do three things that the defaults don't:

1. **Activate sensible production defaults** that the stock install leaves off (admin dashboard, session timeouts, cgroups resource limits, auto-save for VS Code).

2. **Pre-stage commented configuration blocks** for every major feature so admins can uncomment and fill in values rather than hunting through docs to find the right setting names.

3. **Provide inline documentation** with direct links to the relevant admin guide pages, making each config file a self-contained reference for the feature it controls.

The proposed CLI walkthroughs would take this further by making the "uncomment and fill in" process interactive, with validation, encryption, and file permission management handled automatically.

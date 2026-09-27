---
name: letsencrypt-windows
description: >-
  Issue, renew, and maintain Let's Encrypt TLS certificates on Windows via ACME.
  Routes among win-acme, Posh-ACME, and Certify The Web; covers HTTP-01/DNS-01,
  staging, rate limits, Certificate Store, IIS binding, and file export.
  Use when the user mentions Let's Encrypt, SSL/TLS, HTTPS, ACME, Windows
  certificate renewal, win-acme, Posh-ACME, Certify The Web, IIS HTTPS, or
  Windows 证书 / 续期 / HTTPS 证书.
---

# Let's Encrypt on Windows

You are a specialist for Let's Encrypt and ACME on **Windows**. Identify context first, then pick challenge type, ACME client, and installation target. Do not assume Linux Certbot or cloud CDN upload workflows.

For lifetimes, rate limits, staging, and challenge theory, read [reference.md](reference.md).

## Out of scope

- Cloud CDN / load-balancer certificate upload (any vendor)
- Vendor-specific DNS plugin runbooks as the skill's primary path
- Private ops notes under `docs/inner/` — do not load or cite them

If the user needs cloud upload after issuance, stop after a successful local install/export and say that upload is a separate concern.

## How to approach tasks

Work this checklist in order:

1. **Deployment context** — IIS site, other Windows service, file export only, or unclear
2. **Challenge type** — HTTP-01 (port 80 reachable), DNS-01 (wildcard, no HTTP-01 path), or TLS-ALPN-01 (rare)
3. **ACME client** — see routing below
4. **Lifetime** — prefer 90-day unless the user has full automation and asks for short-lived profiles (see reference)
5. **Installation target** — Certificate Store, IIS binding, and/or PEM/PFX files
6. **Environment** — use staging for first/test runs; production only when ready

## ACME client routing

| Scenario | Prefer | Notes |
|----------|--------|-------|
| Local IIS, interactive setup | **win-acme** (`wacs.exe`) or **Certify The Web** | Default win-acme path targets IIS; creates its own renewal task |
| Scripted / PowerShell automation, custom DNS plugin | **Posh-ACME** | `New-PACertificate`, `Submit-Renewal`; good for CI-like scripts |
| User already standardized on one client | Keep that client | Do not migrate without an explicit ask |
| Linux-style Certbot on Windows | Avoid as default | Prefer native Windows clients; Certbot via WSL only if user insists |

Do not download random ACME binaries from untrusted mirrors. Prefer official project releases or PowerShell Gallery (`Install-Module Posh-ACME`).

## Challenge selection (Windows)

| Challenge | Use when | Watch outs |
|-----------|----------|------------|
| HTTP-01 | Single hostnames; port 80 reachable from the public internet | IIS/site must serve `/.well-known/acme-challenge/` before HTTPS redirects; firewall allows 80 |
| DNS-01 | Wildcards (`*.example.com`); HTTP-01 impossible | Needs DNS API credentials or manual TXT; allow propagation time |
| TLS-ALPN-01 | Only 443 available, client supports it | Uncommon; confirm client support before promising |

## Workflows

### A. win-acme → IIS (typical site)

1. Download the official release zip; extract to a stable path (e.g. `%ProgramFiles%\win-acme`).
2. Run `wacs.exe` elevated.
3. Create a new certificate (simple mode for standard IIS; advanced for wildcards / non-IIS).
4. Complete HTTP-01 (default) or configure DNS validation when required.
5. Confirm IIS bindings updated and HTTPS works.
6. Rely on win-acme's scheduled renewal; verify with a renewal test if the user wants certainty (`wacs.exe` renew / test options per current docs).

### B. Posh-ACME → files and optional store

```powershell
Install-Module Posh-ACME -Scope CurrentUser -Force
Import-Module Posh-ACME

# First attempt: staging
Set-PAServer LE_STAGE
# Production when validated:
# Set-PAServer LE_PROD

# HTTP-01 example (plugin choice depends on how challenges are served)
New-PACertificate 'example.com' -AcceptTOS -Contact 'admin@example.com'

# DNS-01 example — use the DNS plugin that matches the user's DNS host
# New-PACertificate 'example.com','*.example.com' -AcceptTOS -Contact 'admin@example.com' `
#   -DnsPlugin <PluginName> -PluginArgs @{ ... }

# Optional: install into Windows Certificate Store when supported by your Posh-ACME version
# New-PACertificate 'example.com' -AcceptTOS -Contact 'admin@example.com' -Install

Get-PACertificate 'example.com'   # FullChainFile, KeyFile, PfxFile, NotAfter
```

Renewal (native):

```powershell
Import-Module Posh-ACME
Set-PAServer LE_PROD
Submit-Renewal -AllOrders   # or target a specific order
# Re-apply installation target after renew if your deploy step is external
```

### C. Certify The Web → IIS

Use when the user prefers a GUI. Guide them through: add managed certificate → choose site/domains → validation → apply to IIS → enable the built-in renew task. Prefer their existing Certify install over introducing a second client.

### D. Installation targets after issuance

| Target | Action |
|--------|--------|
| **IIS** | Prefer win-acme / Certify binding updates; or bind the store cert / PFX in IIS Manager |
| **Certificate Store** | Install PFX or use client `-Install` / installer plugins; note store location (LocalMachine\My vs CurrentUser) |
| **Files** | Keep `fullchain` + private key (or PFX) outside git; ACL-restrict the key |
| **Cloud / CDN** | Out of scope — hand off after local success |

After any renew, confirm the **installation target** was updated. Local ACME renewal does not update external systems.

## Security rules

- Never write private keys, PFX passwords, DNS API tokens, or AccessKeys into the repo, skill docs, or chat in cleartext.
- Do not commit `%LOCALAPPDATA%\Posh-ACME\`, win-acme config with secrets, or exported `*.key` / `*.pfx`.
- ACME contact email is for account and expiry notices only.
- Prefer asking the user to paste secrets into secure local config / secret stores rather than into the transcript.

## Production checklist

Copy and track:

```
Task progress:
- [ ] Challenge type chosen and feasible on this host
- [ ] Staging issuance succeeded (when testing)
- [ ] Production cert issued
- [ ] Installation target applied (Store / IIS / files)
- [ ] HTTPS verified for each hostname
- [ ] Native renewal path confirmed (client task or Submit-Renewal)
- [ ] Expiry monitoring noted (do not rely only on the ACME client)
- [ ] CAA allows Let's Encrypt if CAA exists
```

## Guidelines

- Default lifetime: **90 days**; renew around the client's normal window (~30 days before expiry).
- Prefer **one cert with all SANs** over many single-name certs (rate limits).
- Use **staging** before burning production rate-limit budget.
- Wildcard certificates require **DNS-01**.
- When stuck, diagnose validation (HTTP path, DNS TXT, CAA) before switching clients.

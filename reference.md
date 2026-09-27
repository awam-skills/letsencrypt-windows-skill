# Let's Encrypt / ACME reference (Windows skill)

Agent-facing detail for [SKILL.md](SKILL.md). Keep Windows-specific procedures in the skill; use this file for shared ACME constraints.

## Certificate lifetimes

| Profile | Lifetime | Typical renew | Notes |
|---------|----------|---------------|-------|
| Default | 90 days | ~60 days (⅔ of lifetime) | Recommended default |
| Short-lived | 6 days | ~4 days | Needs reliable automation; no practical revocation window |
| 45-day (opt-in) | 45 days | ~30 days | Confirm current Let's Encrypt policy before promising |

Prefer 90-day unless the user has full automation and explicitly wants short-lived certs.

## Challenge types

### HTTP-01

- Proves control by serving a token at `http://<host>/.well-known/acme-challenge/<token>` on **port 80**.
- Must not be broken by forced HTTPS redirects before the challenge path is served.
- Behind reverse proxies / CDNs: the HTTP challenge must still reach a responder the CA can see (often hard on pure CDN + private origin — prefer DNS-01).

### DNS-01

- Proves control with a TXT record at `_acme-challenge.<name>`.
- Required for wildcards.
- Propagation delays are the usual failure mode; increase wait/sleep in the client when needed.
- Manual DNS is fine for one-offs; API plugins for renewal.

### TLS-ALPN-01

- Uses TLS on port 443 with a special ALPN protocol.
- Only when the chosen Windows client supports it and HTTP-01/DNS-01 are worse fits.

## Staging vs production

| Server | Purpose |
|--------|---------|
| Staging (`LE_STAGE` / client staging flag) | Test validation and client config; separate rate-limit pool; **not** trusted by browsers |
| Production (`LE_PROD`) | Real certificates |

Always validate new automation against staging first when feasible.

### Posh-ACME

```powershell
Set-PAServer LE_STAGE   # test
Set-PAServer LE_PROD    # real
```

### win-acme / Certify

Use each product's staging / test mode (UI or CLI flag) before production issuance. Exact flag names change by version — check current docs when unsure.

## Rate limits (high level)

Limits change; treat these as planning guidance, not a contract:

- Prefer staging while iterating.
- Avoid `--force` / force-renewal when the existing cert is still valid.
- Batch hostnames into one certificate (SAN) instead of one cert per hostname when possible.
- Duplicate certs for the same exact set of names are limited per week.
- Certificates per registered domain are limited per week.

If production returns rate-limit errors, wait for the sliding window or use staging / another ACME CA only with explicit user approval.

Issued cert history (public): `https://crt.sh/?q=<domain>`

## Windows client map

| Client | Strengths | Renewal |
|--------|-----------|---------|
| [win-acme](https://www.win-acme.com/) | IIS-first, interactive, scheduled task | Built-in scheduled renewal |
| [Posh-ACME](https://poshac.me/) | PowerShell, many DNS plugins, scripting | `Submit-Renewal` |
| [Certify The Web](https://certifytheweb.com/) | GUI, IIS-oriented | Built-in renew service/task |

Certbot remains the common Linux reference client; on Windows prefer the clients above unless the user requires Certbot/WSL.

## Certificate files

Typical artifacts (names vary by client):

| Artifact | Use |
|----------|-----|
| Certificate (leaf) | Rarely used alone |
| Chain / intermediates | With leaf for full trust path |
| Full chain | Preferred for most TLS servers |
| Private key | Protect with tight ACLs; never commit |
| PFX / PKCS#12 | Convenient for Windows store / IIS import |

Posh-ACME order data commonly lives under `%LOCALAPPDATA%\Posh-ACME\`. win-acme stores renewal definitions under its install/config directory.

## Common failures

### HTTP-01 connection / timeout

- Port 80 closed on host or edge firewall
- Wrong site binding / hostname
- HTTPS redirect applies to the challenge path
- Another process bound to port 80 when using self-hosted validation

### DNS-01 failure

- TXT not visible yet (`nslookup` / `Resolve-DnsName` against public resolvers)
- Wrong zone (apex vs subdomain)
- API token lacks write permission
- CAA forbids Let's Encrypt (see below)

### Renewed locally but HTTPS still shows old cert

- Installation target not updated (IIS binding, store, or external system)
- Process/service not reloaded after binding change
- Clients caching the old cert (less common for leaf expiry)

### CAA blocking issuance

```
example.com.  CAA 0 issue "letsencrypt.org"
example.com.  CAA 0 issuewild "letsencrypt.org"
```

Include `issuewild` when requesting wildcards.

## Production monitoring

- Confirm the client's native renew mechanism is present (scheduled task / service / documented `Submit-Renewal` job).
- Add independent expiry monitoring when the cert matters (external check or calendar).
- Optional: SSL Labs or equivalent after first production deploy.
- Do not put private keys in chat, tickets, or git.

## External docs

- Let's Encrypt: https://letsencrypt.org/docs/
- ACME challenge types: https://letsencrypt.org/docs/challenge-types/
- Posh-ACME: https://poshac.me/
- win-acme: https://www.win-acme.com/
- Certify The Web: https://certifytheweb.com/

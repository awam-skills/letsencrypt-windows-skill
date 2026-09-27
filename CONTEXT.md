# Let's Encrypt on Windows

Agent skill for issuing, renewing, and maintaining Let's Encrypt TLS certificates on Windows via ACME. Cloud-vendor certificate upload and provider-specific DNS automation are out of this context.

## Language

**Skill Document**:
The `SKILL.md` and other agent-facing skill instructions written in English for model consumption.
_Avoid_: bilingual skill body, Chinese-only skill body

**Description Field**:
The YAML `description` may mix Chinese and English trigger phrases for discovery.
_Avoid_: English-only description when Chinese trigger terms help discovery

**Domain Expert Skill**:
A skill that first identifies deployment context, challenge type, and ACME client, then applies the matching Windows workflow—not a single fixed cloud playbook.
_Avoid_: CDN playbook skill, Aliyun-only skill, ops-script-only skill

**Out of Scope (this skill)**:
Aliyun DNS-01 plugin automation and Aliyun CDN certificate upload/binding. Those belong in separate ops docs or another skill, not here.
_Avoid_: treating `docs/inner` CDN runbooks as this skill's primary path

**Inner Ops Docs**:
`docs/inner/` holds private/historical Windows+Aliyun CDN runbooks. The skill must not cite them as authority or load them into the agent path.
_Avoid_: skill primary docs, canonical reference for this skill

**Installation Target**:
Where a issued certificate is applied after ACME succeeds: Windows Certificate Store, IIS site binding, or exported files (PEM/PFX). Cloud/CDN upload is an external concern, not an Installation Target of this skill.
_Avoid_: CDN binding, Aliyun upload as an installation target

**ACME Client Routing**:
Choosing a Windows ACME client by scenario (e.g. IIS-oriented GUI/CLI vs scriptable PowerShell) rather than mandating a single default forever.
_Avoid_: single-client-only skill, Certbot-as-Windows-default

**Skill Reference**:
English progressive-disclosure file(s) (e.g. `reference.md`) holding ACME lifetimes, staging, rate limits, and challenge details; `SKILL.md` keeps the decision tree and Windows workflows.
_Avoid_: dumping all theory into SKILL.md, relying only on external URLs

**Auto-Discovery**:
The skill is discoverable from ambient context via its description (no `disable-model-invocation: true`).
_Avoid_: explicit-only loading for this skill

**Renewal Guidance**:
Document each client's native renewal mechanism; do not mandate creating Windows Task Scheduler jobs unless the user asks.
_Avoid_: always-create scheduled task, copy-paste Task Scheduler XML as required

**Secret Handling**:
Never write private keys, API secrets, or AccessKeys into the repo, docs, or chat in cleartext. ACME contact email is for account/expiry notices only.
_Avoid_: pasting secrets into SKILL examples, committing Posh-ACME or win-acme private key paths' contents

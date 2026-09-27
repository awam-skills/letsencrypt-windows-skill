# Exclude cloud CDN upload and vendor DNS playbooks from this skill

This repository may contain private Windows+cloud ops notes, but the agent skill is a general Let's Encrypt / ACME expert for Windows (issue, renew, install locally). We deliberately keep cloud CDN certificate upload and vendor-specific DNS automation out of `SKILL.md` so the skill stays portable, AI-reusable, and not coupled to one cloud account's runbook. Those concerns belong in separate ops docs or another skill.

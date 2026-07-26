# Changelog

## Unreleased

- GitFlow-style branch workflow: `develop` integration branch, Actions gates for PR topology + version lockstep (`docs/git-flow.md`)

## 0.2.1

- Skill accuracy pass from live APIM probes: nested vs top-level `branchId`,
  `inspect_actions` is necessary≠sufficient, idempotency-replay sharp edge,
  `user_declined` elicitation note
- Document prod-APIM-only wiring; private-repo install premise
- Align keywords; add `homepage`/`repository`; add LICENSE
- Bump versions in all three lockstep places; fix CLAUDE.md maintenance note

## 0.2.0

- Wire six APIM MCP bundles in `plugins/danely-platform/.mcp.json` (OAuth, no static headers)
- Document Settings → Connectors custom-connector URLs for staff testing
- Bump `danely-platform` to 0.2.0

## 0.1.0

- Initial marketplace scaffold
- `danely-platform` plugin with `platform-conventions` skill

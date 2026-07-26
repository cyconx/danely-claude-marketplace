# Branch workflow (GitFlow-style)

| Branch | Role | How it updates |
|--------|------|----------------|
| `main` | Stable / installable marketplace | **Only** via PR `develop` → `main` |
| `develop` | Integration | **Only** via PR from `feature/*`, `fix/*`, `chore/*`, `docs/*`, or `hotfix/*` |
| `feature/*` (etc.) | Work in progress | Direct pushes OK on the feature branch |

## Day-to-day

```text
git fetch origin
git checkout develop
git pull
git checkout -b feature/short-description

# … commit …

git push -u origin HEAD
gh pr create --base develop --title "…" --body "…"
```

After the feature PR merges to `develop`, promote a release:

```text
gh pr create --base main --head develop --title "release: …" --body "…"
```

Claude Code `/plugin marketplace add` tracks this repo; prefer releasing from `main` so staff auto-update to tagged/promoted versions only.

## Automation in this repo

- [`.github/workflows/gitflow-gate.yml`](.github/workflows/gitflow-gate.yml) — fails PRs that violate the topology above.
- [`.github/workflows/marketplace-hygiene.yml`](.github/workflows/marketplace-hygiene.yml) — fails if the three `version` fields drift.

## Branch protection (requires GitHub Team / Pro on private repos)

Private repositories on the free plan cannot enable classic branch protection or rulesets via the API/UI (GitHub returns 403). Until the `cyconx` org (or this repo) has **Team/Pro**, the Actions gates above are advisory for anyone who can push — **do not push directly to `main` or `develop`**.

When protection is available, enable both rules (Settings → Branches, or the `gh` snippets below) and mark the `GitFlow gate` / `Plugin versions match` checks as **required**:

### `main`

- Require a pull request before merging
- Require status checks: `Enforce GitFlow PR topology`, `Plugin versions match`
- Do not allow bypass without admin
- Restrict who can push (empty = no direct pushes)
- Optionally: require linear history **off** if you prefer merge commits for release PRs

### `develop`

- Require a pull request before merging
- Require status checks: `Enforce GitFlow PR topology`, `Plugin versions match`
- Same push restrictions

```bash
# After Team/Pro is enabled — classic branch protection examples:
gh api -X PUT repos/cyconx/danely-claude-marketplace/branches/main/protection \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["Enforce GitFlow PR topology", "Plugin versions match"]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 0
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF

gh api -X PUT repos/cyconx/danely-claude-marketplace/branches/develop/protection \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["Enforce GitFlow PR topology", "Plugin versions match"]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 0
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF
```

(Adjust review count / CODEOWNERS when you want human approval.)

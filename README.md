# backendforth/.github

Org-wide GitHub Actions reusable workflows for Slack PR notifications.

## Workflows

### `pr-slack-summary.yml` — PR open + merge

**On open** — fixed English template (always the same sections):

1. PR title `#number`
2. Status: `Open` / `Conflicts`
3. Opener · latest commit · tags · related issues
4. Commit count
5. Files changed · lines added/deleted
6. Summary (AI)
7. Breaking changes (always shown; `_None_` if empty)
8. Documentation (only when breaking changes exist and links were found)
9. View PR · Start review

**Dependabot PRs:** AI prompt prioritizes breaking changes and documentation links. A deterministic pass also scans the PR body for major semver bumps (`from X.x.x to Y.x.x` where Y > X) and extracts `https` URLs, merged with the AI output.

**On merge** — compact line only:

```
MERGED — PR #123 by `author`
PR title (linked)
[View repo] [View PR]
```

#### Caller (per repo)

```yaml
name: PR Slack notify

on:
  pull_request:
    types: [opened, ready_for_review, closed]
    branches: [main]   # + variant/document-level in next-sanity-starter

jobs:
  notify:
    uses: backendforth/.github/.github/workflows/pr-slack-summary.yml@main
    secrets: inherit
```

### `pr-slack-monthly-digest.yml` — monthly overview

Runs for the **previous calendar month**. Posts totals, AI overview, highlights, contributors (humans, Dependabot, bots, Cursor/Claude/Copilot co-authors), dependency notes, and a PR index.

#### Caller (per repo)

```yaml
name: PR monthly Slack digest

on:
  schedule:
    - cron: "0 6 1 * *"   # 1st of month, 06:00 UTC
  workflow_dispatch:

jobs:
  digest:
    uses: backendforth/.github/.github/workflows/pr-slack-monthly-digest.yml@main
    secrets: inherit
```

## Org secrets

| Secret | Purpose |
|---|---|
| `SLACK_WEBHOOK_URL` | Incoming Webhook → `#github-logs` |
| `OPENROUTER_API_KEY` | AI summaries (set ~$5 credit cap) |

Scope: all enrolled repos (`next-sanity-starter`, `bef-website-2026`, `farbstudio.de`, `mammalsandcomputers`).

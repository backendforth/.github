# backendforth/.github

Org-wide GitHub Actions reusable workflows for Slack PR notifications.

## Workflows

### `pr-slack-summary.yml` — PR open + merge

**On open** — fixed English template (always the same sections):

1. `➔ New PR · owner/repo` (code backticks, green attachment bar)
2. `➔` PR title `#number` (code backticks, linked to PR)
3. Status · opener · commit · tags · issues · files · lines
4. **Summary** — from PR body (`## Summary` …), code block, not AI
5. **Breaking changes** — always shown; code block
6. **Documentation** — only when breaking changes + links exist
7. View PR · Start review

Green (`#2EB67D`) left bar via Slack attachment. Dividers top/bottom for separation.

**On merge** — compact card, purple (`#9B59B6`) attachment bar:

```
⤴ Merged PR · owner/repo
⤴ PR title #123 (linked)
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

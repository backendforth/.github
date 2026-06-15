# backendforth/.github

Org-wide GitHub Actions reusable workflows for Slack PR notifications.

## Workflows

### `pr-slack-summary.yml` — PR open + merge

**On open** — fixed English template (always the same sections):

1. `➔ New PR` + PR title `#number` (plain label, title+# linked in code backticks)
2. `owner/repo` (code backticks)
3. Status · opener · commit · tags · issues · files · lines
4. **Summary** — from PR body (`## Summary` …), code block, not AI
5. **Breaking changes** — always shown; code block
6. **Documentation** — only when breaking changes + links exist
7. View PR · Start review

**Dependabot PRs:** `*Claude review*` section @-mentions Claude (`<@U0BAPTE85TL>`). `/invite @Claude` in `#github-logs` required.

Attachment bar colors: green (`#2EB67D`) for human PRs, blue (`#439FE0`) for Dependabot. Divider only before Summary.

**On merge** — compact card; purple (`#9B59B6`) for human merges, blue for Dependabot:

```
⤴ Merged PR title #123 (linked, code)
owner/repo
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
| `SLACK_WEBHOOK_URL` | Incoming Webhook → `#github-logs` (fallback when no bot token) |
| `OPENROUTER_API_KEY` | Dependabot breaking-change analysis |
| `SLACK_BOT_TOKEN` | Optional — `xoxb-…` with `chat:write` + `reactions:write` (posts + emoji reactions) |
| `SLACK_CHANNEL_ID` | Optional — `#github-logs` channel ID (`C…`) |

When `SLACK_BOT_TOKEN` and `SLACK_CHANNEL_ID` are set, messages post via `chat.postMessage` and get an emoji reaction: `:eyes:` (new PR), `:package:` (Dependabot), `:white_check_mark:` (merged). With webhook only, posting works but reactions do not (webhooks do not return a message timestamp).

Scope: all enrolled repos (`next-sanity-starter`, `bef-website-2026`, `farbstudio.de`, `mammalsandcomputers`).

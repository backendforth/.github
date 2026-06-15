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

**Dependabot PRs:** OpenRouter analyzes breaking changes and documentation links. Optional second message @-mentions Claude via the Slack Web API (see secrets below).

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
| `SLACK_WEBHOOK_URL` | Incoming Webhook → `#github-logs` |
| `OPENROUTER_API_KEY` | Dependabot breaking-change analysis |
| `SLACK_BOT_TOKEN` | Optional — Bot token (`xoxb-…`) for real `@Claude` mentions |
| `SLACK_CHANNEL_ID` | Optional — Channel ID for `#github-logs` (e.g. `C012…`) |
| `SLACK_CLAUDE_USER_ID` | Optional override if auto-lookup of `@Claude` fails |

### Claude @-mention (Dependabot PRs)

Incoming webhooks cannot trigger `@Claude` — plain `@Claude` text is not a real mention. For Dependabot opens, the workflow posts the card via webhook, then sends a **second message** via `chat.postMessage` with `<@ClaudeUserId>`.

**One-time setup**

1. `/invite @Claude` in `#github-logs`
2. Enable **Developer mode** in Slack (Preferences → Advanced)
3. Copy **Channel ID**: `#github-logs` → channel details → bottom of dialog → `C…`
4. Create a small Slack app (or reuse one) with scopes `chat:write`, `users:read` → install to workspace → copy **Bot User OAuth Token** (`xoxb-…`)
5. Org secrets: `SLACK_BOT_TOKEN`, `SLACK_CHANNEL_ID` (optional: `SLACK_CLAUDE_USER_ID` if lookup fails)

Claude must be in the channel and your account connected in the Claude Slack app. Claude may ignore mentions from other bots in some workspaces — if the ping appears but Claude stays silent, reply manually in the thread once to confirm routing.

Scope: all enrolled repos (`next-sanity-starter`, `bef-website-2026`, `farbstudio.de`, `mammalsandcomputers`).

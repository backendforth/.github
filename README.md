# backendforth/.github

Org-wide GitHub Actions reusable workflows for Slack PR notifications.

## Workflows

### `pr-slack-summary.yml` — PR open + merge

**On open** — fixed English template (always the same sections):

1. `➔ New PR` label (code backticks)
2. `➔` PR title `#number` (code backticks)
3. Status: `Open` / `Conflicts`
4. Opener · latest commit · tags · related issues
5. Commit count
6. Files changed · lines added/deleted
7. **Summary** — extracted from the PR body (`## Summary`, else `## Description`, else first paragraph); shown in a code block. Not AI-generated.
8. **Breaking changes** — always shown (`_None_` if empty); code block
9. **Documentation** — only when breaking changes exist and links were found (Dependabot focus)
10. View PR · Start review

Messages are wrapped with leading/trailing dividers and a `repo · PR #N` context line for separation in busy channels.

**Dependabot PRs:** OpenRouter analyzes breaking changes and documentation links only (prompts never reference Slack or notification channels). A deterministic pass also scans the PR body for major semver bumps and `https` URLs, merged with the AI output.

**On merge** — compact card with code labels:

```
⤴ Merged PR
⤴ PR #123 by author
PR title (linked)
[View repo] [View PR]
```

Also wrapped with dividers and repo context.

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

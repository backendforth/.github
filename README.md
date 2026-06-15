# backendforth/.github

Org-wide GitHub Actions reusable workflows. Repos under `backendforth/*` opt in
by adding a thin caller workflow that points at the canonical workflow files
here.

## Reusable workflows

### `pr-slack-summary.yml` — PR → Slack with AI summary

Posts to `#github-logs` when a PR opens or merges. Drafts and fork PRs are
skipped. One OpenRouter call per PR (on open only); merge posts a compact
status line without re-summarizing.

#### Message layout

**On open** — title, repo, author, AI summary, breaking changes (if any),
status footer, Review button:

```
*PR title* (link)
`repo` · `author`
─────────────────
2–3 sentence AI summary

📥 Status: Offen          [Review PR →]
```

**On merge** — one line, no AI summary:

```
MERGED — PR #123 by `author`
chore(ci): smoke-test Slack PR notification pipeline
[View repo]  [View PR]
```

#### How to enroll a repo

1. Make sure the two org secrets are accessible to this repo (see below).
2. Drop this caller workflow into the repo at
   `.github/workflows/pr-slack-notify.yml`:

   ```yaml
   name: PR Slack notify

   on:
     pull_request:
       types: [opened, ready_for_review, closed]
       branches:
         - main
         # add other long-lived branches the repo ships, e.g.:
         # - variant/document-level

   jobs:
     notify:
       uses: backendforth/.github/.github/workflows/pr-slack-summary.yml@main
       secrets: inherit
   ```

3. Open a test PR — within ~30 s a message with summary + `📥 Status: Offen`
   appears in `#github-logs`. Merge the PR → `MERGED — PR #N by author` with
   repo and PR links (no second summary).

#### Org secrets it expects

Set both at the org level (Settings → Secrets and variables → Actions →
Organization secrets) with repository access limited to the repos that opt in.

| Secret | Where it comes from | Notes |
|---|---|---|
| `SLACK_WEBHOOK_URL` | Slack → Apps → Incoming Webhooks → pointed at `#github-logs` | Treat like a secret — anyone with this URL can post into the channel. |
| `OPENROUTER_API_KEY` | <https://openrouter.ai/keys> | Create a dedicated key named `github-actions-pr-summary` with a small credit cap (e.g. $5) — the workflow uses well under $0.10 / month across four repos. |

#### Optional input

The default model is `google/gemini-3.1-flash-lite`. Override per-repo if
needed:

```yaml
jobs:
  notify:
    uses: backendforth/.github/.github/workflows/pr-slack-summary.yml@main
    secrets: inherit
    with:
      model: anthropic/claude-haiku-4.5
```

#### What the open message contains

1. PR title (link) + repo + author
2. AI-generated 2–3 sentence summary (OpenRouter, default `google/gemini-3.1-flash-lite`)
3. Breaking-changes section (only when flagged)
4. Status footer `📥 Offen` + Review button

Merge posts only a one-line status update — no second summary.

## License

Internal infrastructure — no license file. Workflows here are intended only
for `backendforth/*` repositories.

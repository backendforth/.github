# backendforth/.github

Org-wide GitHub Actions reusable workflows. Repos under `backendforth/*` opt in
by adding a thin caller workflow that points at the canonical workflow files
here.

## Reusable workflows

### `pr-slack-summary.yml` — PR → Slack with AI summary

Posts an AI-summarized Slack message to a single channel for every PR opened,
marked ready-for-review, or merged on a watched branch. Drafts and fork PRs are
skipped. Powered by an OpenRouter call (default model
`google/gemini-3.1-flash-lite`) feeding a Slack Block Kit message via incoming
webhook.

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

3. Open a test PR — within ~30 s a `📥 New PR · <repo>` message should appear
   in `#github-logs`. Merge the PR → a second `✅ Merged · <repo>` message
   (without the Review button) follows.

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

#### What the message contains

1. Header — repo name, opened or merged
2. PR title (clickable) + author
3. AI-generated 2–3 sentence summary
4. Breaking-changes section (only shown when the model flags at least one)
5. "Review PR →" button (only on non-merged PRs)

The model is prompted to flag major version bumps in Dependabot grouped PRs
and `BREAKING CHANGE` / `feat!` / `fix!` markers in feature PRs.

## License

Internal infrastructure — no license file. Workflows here are intended only
for `backendforth/*` repositories.

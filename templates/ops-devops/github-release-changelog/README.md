# GitHub release changelog post

Posts a release's tag, name, URL, and notes to Slack when GitHub publishes a release. Created, edited, deleted, and prerelease actions are ignored. Notes are truncated to 2500 characters so the Slack message stays under the API limit.

## Credentials

- GitHub: access token, OAuth2, or a GitHub App that can manage webhooks on the repository.
- Slack: bot token with `chat:write`.

## Setup

1. Import `workflow.json`.
2. On **GitHub Trigger**, set the owner and repository. The event list is `release`.
3. On **Post changelog**, set the Slack channel. The template uses `#ops-alerts`.
4. Attach both credentials and activate the workflow so n8n registers the webhook.

## Customize

- **Format changelog** builds the Slack text. Change the truncation length or add `release.author.login`.
- Point the Slack node at a changelog channel that is separate from the pull-request alert channel.
- To include prereleases, add `prereleased` next to `published` in **Published release** (use OR by switching the combinator, or a second IF).

## Test

Publish a GitHub release that has a short body. The execution should pass **Published release** and the Slack message should contain the tag, the release URL, and the notes. Editing the release afterwards should not post again.

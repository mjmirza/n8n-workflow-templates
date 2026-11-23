# GitHub pull request opened to Slack

Posts a Slack message when a pull request is opened on a repository you administer. Draft pull requests are ignored. Other pull request actions (synchronize, closed, edited) do not notify.

## Credentials

- GitHub: personal access token, OAuth2, or a GitHub App. The account must be allowed to create repository webhooks (repository admin, or organization owner).
- Slack: bot token with `chat:write`. Invite the bot to the destination channel.

## Setup

1. Import `workflow.json`.
2. On **GitHub Trigger**, set **Repository Owner** and **Repository Name**. Authentication defaults to an access token.
3. On **Post to Slack**, set **Channel** to the channel that should receive the alert. The template uses `#ops-alerts`.
4. Attach the GitHub and Slack credentials.
5. Activate the workflow. n8n registers the GitHub webhook. GitHub sends a ping; the trigger ignores that ping.

## Customize

- The **Opened and not a draft** node keeps `action` equal to `opened` and `pull_request.draft` false. Add another condition for labels or base branch.
- Edit the Slack **Text** to include reviewers, the base branch, or a team mention.
- Duplicate the Slack node and point it at a second channel if product and engineering should both hear about the same repository.

## Test

Open a non-draft pull request. The execution should show the GitHub payload, pass the IF node, and post a message with the repository, PR number, title, author, and URL.

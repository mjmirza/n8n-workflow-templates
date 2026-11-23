# Error workflow to Slack

An Error Trigger workflow that posts the failed workflow name, execution id, last node, error message, and execution URL to Slack.

n8n runs this workflow only when another workflow is configured to use it as its error workflow. Importing the file does not attach it for you.

## Credentials

- Slack: bot token with `chat:write`. Invite the bot to the alert channel.

## Setup

1. Import `workflow.json` and activate it. Error workflows must be active.
2. On **Post error to Slack**, set the channel. The template uses `#ops-alerts`.
3. Attach the Slack credential.
4. Open each workflow you want to monitor. In its settings, set **Error Workflow** to this workflow.
5. Use **Execute workflow** on this workflow once. With no incoming error, the Error Trigger supplies the built-in example payload so you can confirm the Slack message shape.

## Customize

- **Build error message** reads `execution.error.message`, `execution.lastNodeExecuted`, `execution.url`, `execution.id`, and `workflow.name`. Add `execution.error.stack` only if that channel should see stack traces.
- Add an IF node on `workflow.name` when one channel should receive production failures and another should receive the rest.

## Test

Execute this workflow manually and confirm the example error reaches Slack. Then force a failure in a workflow that points at this error workflow (for example an HTTP Request to an invalid host) and confirm a second message names that workflow.

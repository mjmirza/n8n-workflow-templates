# Pipedrive deal won Slack alert

Posts one Slack message when a Pipedrive deal moves into **Won**. Updates to a deal that was already won, and every open or lost change, stop at the IF node.

## Credentials

- Pipedrive API token on **Pipedrive Deal Event**.
- Slack OAuth2 on **Announce Win**.

## Setup

1. Import the workflow and connect both credentials.
2. Set `slackChannel` in **Settings** to a channel the Slack app has joined, such as `#sales-wins`.
3. Activate the workflow. n8n registers the Pipedrive webhook for all deal actions. The production URL must be reachable by Pipedrive.
4. Mark a test deal won. The message includes title, value, currency, organization or person, owner, and the Pipedrive deal id.

## Customize

- Mention an owner by mapping Pipedrive user ids to Slack member ids in a Code node before **Announce Win**.
- Add a Google Sheets append beside the Slack node if finance wants a win log.
- The trigger listens to every deal action on purpose. Narrowing it to `change` misses a deal that is created already won. The IF is what defines a win.

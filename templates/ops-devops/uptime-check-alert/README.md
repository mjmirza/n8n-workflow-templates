# Uptime check with Slack alert

Every five minutes, requests each configured URL and posts to Slack when the HTTP status is outside 200-299. Healthy checks stay quiet.

## Credentials

- Slack: bot token with `chat:write`. Invite the bot to the alert channel.

The HTTP Request node does not send authentication. Put a signed health URL in the target list, or add Header Auth on **Check URL** if the endpoint requires it.

## Setup

1. Import `workflow.json`.
2. Open **Build targets** and replace the sample URLs with the health endpoints you run. Each entry needs `name` and `url`.
3. On **Alert Slack**, set the channel. The template uses `#ops-alerts`.
4. Attach the Slack credential.
5. Execute once manually and confirm a down URL alerts and a 200 does not.
6. Activate the workflow.

## Customize

- Change the Schedule Trigger from every 5 minutes to another interval.
- **Check URL** times out after 8 seconds and still returns a status code on HTTP errors, so a 500 is an alert rather than a failed execution. Connection failures still fail the execution; pair this workflow with the error-workflow template if you want those in Slack too.
- Add `headers` on the HTTP Request node for a shared health token.

## Test

Point one target at a URL that returns 200 and another at a URL that returns 404. Run the workflow manually. Only the failing target should produce a Slack message.

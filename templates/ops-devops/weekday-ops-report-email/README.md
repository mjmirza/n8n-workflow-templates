# Weekday ops report email

Every weekday at 08:00, fetches a JSON status document and emails a short report. A Statuspage-style summary (`status`, `components`, `incidents`) is rendered as a component list. Any other JSON object is included as formatted text. The subject starts with `ALERT` when the HTTP status is not successful, the overall indicator is not `none`, a component is not `operational`, or any incident is present. Otherwise the subject starts with `OK`.

## Credentials

- SMTP, on the Send Email node.
- No credential is set on the HTTP Request. Add Header Auth there if the status URL is private.

## Setup

1. Import `workflow.json`.
2. On **Report config**, set `reportUrl`, `fromEmail`, and `toEmail`.
3. Attach an SMTP credential to **Send report**.
4. Set the workflow timezone if 08:00 should be local time rather than the instance timezone. The trigger runs Monday through Friday.
5. Execute manually, check the message, then activate.

## Customize

- **Format report** understands the public Statuspage summary shape used by `https://status.example.com/api/v2/summary.json`. Point `reportUrl` at your own status JSON and extend the code node for fields that shape does not have.
- Add CC in the Send Email options.
- Change **Weekdays 08:00** to a daily trigger if the report should include weekends.

## Test

Set `reportUrl` to a status endpoint you can reach, run the workflow manually, and confirm the mailbox receives one plain-text message whose subject is `OK ops report` or `ALERT ops report`.

# Support ticket triage

Classifies an incoming support ticket with a language model, pages Slack when the priority is high or urgent, and appends every ticket to Google Sheets.

## Use case

A form, helpdesk, or internal tool POSTs a ticket. The workflow returns a category, priority, owning team, short summary, and whether a person should take it. High and urgent tickets are also posted to a Slack channel. Every ticket is logged, including the low-priority ones.

## Required credentials

Attach these in the n8n editor after import. The workflow file does not contain credentials or secret values.

| Node | Credential |
| --- | --- |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |
| Alert Slack | Slack bot token (`slackApi`) with `chat:write` |
| Log Ticket | Google Sheets OAuth2 (`googleSheetsOAuth2Api`) |

## Setup

1. Import `workflow.json` with the workflow menu, **Import from File**.
2. Open **Config**. Paste the spreadsheet id (the long id in the sheet URL) into `spreadsheetId`. Set `sheetName` and `slackChannel` (channel name only, no `#`).
3. On the sheet, row 1 must be exactly: `TicketId`, `ReceivedAt`, `RequesterEmail`, `Subject`, `Category`, `Priority`, `Team`, `Summary`, `NeedsHuman`.
4. Invite the Slack app to that channel.
5. Select the credential on each of the three nodes above, then activate the workflow.

## Input

`POST` JSON to the production URL of **Ticket Webhook** (path `support-ticket-triage`):

```json
{
  "ticketId": "T-1044",
  "requesterEmail": "amina@example.com",
  "subject": "Charged twice for April",
  "body": "I see two charges of 49 USD on my card ending 4242. Please refund one of them."
}
```

`subject` and `body` are required for a useful classification. `description`, `message`, `email`, and `from` are accepted as aliases. If `ticketId` is missing, the execution id is used.

The HTTP response is the normalized ticket plus `category`, `priority`, `team`, `summary`, and `needsHuman`.

## How to customize

- Edit the prompt on **Classify Ticket** to add a product line or a language.
- Edit the allowed values in **Validate Classification**. Anything outside the lists is coerced to `general` / `normal` so a drifted model response cannot invent a queue name.
- **Urgent or high?** uses OR. Add `needsHuman` is true if you also want those tickets to page Slack.
- Change the model on **OpenAI Chat Model**. Leave temperature at 0 for stable labels.
- The Slack node runs beside the sheet append. A Slack failure still fails the execution, which keeps a paging outage visible.

## Test

With the workflow active, send the sample body to the webhook URL shown on **Ticket Webhook**. Confirm a row in the sheet and, for this billing sample, a Slack message. Then send a "how do I change my password" ticket and confirm it is logged as account/normal without a Slack page.

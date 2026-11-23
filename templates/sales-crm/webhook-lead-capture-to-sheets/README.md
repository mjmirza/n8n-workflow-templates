# Webhook lead capture to Google Sheets

Records an inbound sales lead in a Google Sheet. Point a website form, a landing-page tool, or another system at the webhook. The workflow normalizes common field names, rejects a payload without a usable email, appends the lead, and returns JSON.

## Credentials

- Google Sheets OAuth2, on **Append Lead**. The file does not contain a credential id or secret. After import, select the Google account that can edit the spreadsheet.

## Setup

1. Create a spreadsheet with a tab named `Leads` and a header row: Received At, Email, First Name, Last Name, Company, Job Title, Phone, Website, Source, Page URL, Message.
2. Import `workflow.json`.
3. Open **Settings** and paste the spreadsheet ID (the long id in the sheet URL) into `spreadsheetId`.
4. Activate the workflow and copy the production URL for `POST /webhook/sales-lead`.

## Payload

Send JSON. These names are accepted, plus `fullName` or `name` when first and last are not separate:

```json
{
  "email": "ada@acme.example",
  "firstName": "Ada",
  "lastName": "Lovelace",
  "company": "Acme",
  "jobTitle": "VP Sales",
  "phone": "+1-555-0100",
  "website": "https://acme.example",
  "source": "webinar",
  "pageUrl": "https://acme.example/demo",
  "message": "Looking at pricing this quarter"
}
```

A missing or blank email returns HTTP 422. A accepted lead returns HTTP 200 with the normalized email and timestamp.

## Customize

- Change `sheetName` in Settings only as a reminder. The append node uses the tab name `Leads`; edit that node if the tab is named differently.
- Add a column in the sheet, then add the same column in **Append Lead**.
- To notify a channel as well, copy the Slack pattern from the lead-scoring template and connect it beside **Append Lead**.

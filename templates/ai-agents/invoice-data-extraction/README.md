# Invoice data extraction

Reads invoice text, extracts the supplier, number, dates, currency, amounts, and a line-item summary, and writes the row to Google Sheets only when the totals agree.

## Use case

Accounts payable receives invoices as pasted text or as the body of a mail that was already converted to text. This workflow structures that text. Before anything is stored, a code step checks that the total is present and that subtotal plus tax matches the total within five cents. A mismatch is posted to Slack and returned to the caller instead of being appended.

## Required credentials

| Node | Credential |
| --- | --- |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |
| Save Invoice | Google Sheets OAuth2 (`googleSheetsOAuth2Api`) |
| Ask For Review | Slack bot token (`slackApi`) with `chat:write` |

## Setup

1. Import `workflow.json`.
2. In **Config**, set `spreadsheetId`, `sheetName`, and `slackChannel`.
3. Header row: `Vendor`, `InvoiceNumber`, `InvoiceDate`, `DueDate`, `Currency`, `Subtotal`, `Tax`, `Total`, `LineItems`, `SourceName`.
4. Attach credentials and activate. The webhook path is `invoice-extract`.

## Input

```json
{
  "sourceName": "northwind-april.txt",
  "invoiceText": "Northwind LLC
Invoice NW-441
Date 2026-04-02
Due 2026-04-16
Widget 2 x 40.00 = 80.00
Subtotal 80.00
Tax 8.00
Total USD 88.00"
}
```

`text` is accepted as an alias of `invoiceText`. Input is capped at 20,000 characters. An empty body returns HTTP 422 and does not call the model.

A valid invoice returns the extracted fields plus `valid: true` after the sheet append. An invalid one returns `valid: false` and `issues`, and Slack gets the same issue list.

## How to customize

- Add attributes on **Extract Invoice** (purchase order, IBAN, tax id). Attribute types are string, number, date, or boolean. Line items are a single string because this node does not extract arrays. Split that string in a Code node if you want one row per line.
- **Check Totals** treats a missing subtotal or tax as acceptable, and only compares them when both are present. Change `0.05` if your currency needs a wider tolerance.
- The model temperature is 0. Keep it there. Amounts drift when the temperature is high.
- Point the webhook at whatever already has the text. Add an Extract From File node in front if you start from PDF uploads.

## Test

POST the sample. Expect one sheet row with total 88 and `valid: true`. Change the printed total to 90, POST again, and expect Slack plus no new row.

# Meeting transcript to action items

Turns a transcript into a summary, a decision list, and one Google Sheets row per action item, and posts the same summary to Slack.

## Use case

After a customer call or a planning meeting, paste the transcript (from your recorder, or from a doc export) and get owners and due dates instead of a wall of text. The webhook response contains the structured result so the caller can store it too.

## Required credentials

| Node | Credential |
| --- | --- |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |
| Post Summary | Slack bot token (`slackApi`) with `chat:write` |
| Log Action Item | Google Sheets OAuth2 (`googleSheetsOAuth2Api`) |

## Setup

1. Import `workflow.json`.
2. In **Config**, set `spreadsheetId`, `sheetName`, and `slackChannel`.
3. Create the header row: `Title`, `Attendees`, `Summary`, `Decisions`, `Task`, `Owner`, `DueDate`, `Priority`.
4. Attach credentials, invite the Slack app, and activate. The webhook path is `meeting-actions`.

## Input

```json
{
  "title": "Billing launch check",
  "attendees": ["Amina", "Leo"],
  "transcript": "Amina: We will ship the tax fix on Thursday. Leo: I will send the revised quote to Northwind tomorrow."
}
```

`attendees` may be a string or an array. `text` is accepted as an alias of `transcript`. Transcripts are truncated at 24,000 characters.

The response includes `summary`, `decisions`, `actionItems`, and the Slack text. A meeting with no actions still writes one sheet row with an empty task so the summary is not lost.

## How to customize

- Change the schema example on **Action Schema** if you need a field such as `status` or `team`. Then add that column on the sheet and on **Log Action Item**.
- The prompt refuses owners who are not in the transcript or the attendee list. Add role names there if your meetings use roles instead of names.
- Priority values other than low, normal, and high are coerced to normal in **Package Meeting**.
- **Split Action Items** emits one item per entry. The sheet node maps the meeting title from **Package Meeting**, so the split does not have to carry it.

## Test

POST the sample. Expect one Slack message, one sheet row for the quote, and a JSON body whose `actionItems[0].owner` is Leo. POST a transcript with no commitments and expect a single sheet row with an empty task.

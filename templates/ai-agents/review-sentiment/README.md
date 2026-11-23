# Sentiment analysis of reviews

Scores unscored product reviews in a Google Sheet and posts negative reviews to Slack.

## Use case

Reviews land in a sheet from a form, an app store export, or a shop. Once an hour this workflow fills `Sentiment`, `Strength`, and `Confidence` for up to 25 rows that do not have a sentiment yet. Negative rows are copied to Slack with the product name and the review text so someone can answer them. Positive and neutral rows are only written back to the sheet.

## Required credentials

| Node | Credential |
| --- | --- |
| Read Reviews and Update Review | Google Sheets OAuth2 (`googleSheetsOAuth2Api`) |
| OpenAI Chat Model | OpenAI API (`openAiApi`). Keep temperature at 0. |
| Alert Negative | Slack bot token (`slackApi`) with `chat:write` |

The sentiment node routes items to three outputs in this order: Positive, Neutral, Negative. All three outputs update the sheet. Only the negative output also posts to Slack. Strength and confidence are model estimates, which is why they are stored beside the label instead of being treated as a measurement.

## Setup

1. Import `workflow.json`.
2. Create a sheet with this header row: `ReviewId`, `Product`, `ReviewText`, `Sentiment`, `Strength`, `Confidence`, `ReviewedAt`.
3. In **Config**, set `spreadsheetId`, `sheetName`, and `slackChannel`.
4. Leave `Sentiment` blank on rows that still need scoring. `ReviewId` has to be unique because the update matches on it.
5. Attach credentials. Run **Manual Trigger** before trusting the hourly schedule.

## How to customize

- The categories are the node default: Positive, Neutral, Negative. Add `options.categories` on **Score Sentiment** if you need a different set, and add a matching output connection for each extra category. Output order follows the category list.
- **Unscored Reviews** caps a run at 25 rows so one execution cannot drain a large backlog and a rate limit. Raise the cap if the sheet is small.
- Rows with a sentiment already set are skipped, so a re-run does not page Slack again for the same review.
- Edit the Slack text if you want a link back to the row. The sheet update does not clear `ReviewText`.

## Test

Add two rows with fresh `ReviewId` values, one clearly positive and one clearly negative, and leave `Sentiment` empty. Run the manual trigger. Confirm both labels land on the sheet and only the negative review is posted. Run again and confirm Slack stays quiet.

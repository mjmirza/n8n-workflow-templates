# Blog post to social posts

Fetches an article, or accepts HTML you already have, and writes a LinkedIn post, an X post, and a two-sentence newsletter blurb. The variants are posted to Slack and appended to Google Sheets.

## Use case

A published post still has to be rewritten three times. This workflow does that first pass and leaves the copy in Slack so a person can edit it before it goes anywhere public. It does not publish to LinkedIn or X.

## Required credentials

| Node | Credential |
| --- | --- |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |
| Post Variants | Slack bot token (`slackApi`) with `chat:write` |
| Log Variants | Google Sheets OAuth2 (`googleSheetsOAuth2Api`) |

**Fetch Article** is an HTTP Request node with no credential. The article URL has to be public.

## Setup

1. Import `workflow.json`.
2. In **Config**, set `spreadsheetId`, `sheetName`, and `slackChannel`.
3. Header row: `SourceUrl`, `Title`, `LinkedIn`, `X`, `Newsletter`, `Hashtags`.
4. Attach credentials and activate. The webhook path is `blog-to-social`.

## Input

Either a public URL:

```json
{ "url": "https://example.com/blog/onboarding-day-two" }
```

Or the article itself, when the page is gated or already extracted:

```json
{ "title": "Day-two onboarding", "html": "<article><p>Send one job, not twelve links.</p></article>" }
```

The response JSON contains `linkedin`, `x`, `xLength`, `newsletter`, `hashtags`, and `sourceUrl`. If the extracted text is under 80 characters the webhook returns HTTP 422 and does not call the model.

## How to customize

- **Extract Article** looks for `article`, then falls back to `body`. Add a `cssSelector` for your template (for example `main`) and prefer that field in **Clean From Fetch**.
- **Fit To Channels** trims X to 280 characters on a word boundary if the model runs long. Change 280 if you also want a thread instead of a single post.
- The prompt forbids claims that are not in the article. Add a sentence if you want a fixed call to action or a required hashtag.
- Sheets is in the path before the HTTP reply, so the caller only gets a 200 after the row is written. Slack is parallel and does not block that reply.

## Test

POST a short public article you control. Confirm three variants in Slack, one sheet row, and `xLength` at or under 280. POST `{ "html": "too short" }` and confirm a 422 with no sheet row.

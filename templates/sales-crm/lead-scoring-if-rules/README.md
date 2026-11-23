# Lead scoring with IF tier rules

Scores a new lead from firmographic and intent fields, then routes it with IF nodes. A score at or above `hotThreshold` (default 70) is hot and is posted to Slack. A score at or above `warmThreshold` (default 40) is warm. Everything else is cold. Every accepted lead is appended to Google Sheets and returned on the webhook.

## Credentials

- Google Sheets OAuth2 on **Append Scored Lead**.
- Slack OAuth2 on **Notify Hot Lead**. The Slack app must be able to post in the channel.

No tokens are stored in this template.

## Setup

1. Create a tab named `Scored Leads` with the lead-capture columns plus Employee Count, Annual Revenue, Score, Tier, Reasons.
2. Paste the spreadsheet ID into **Settings**.
3. Set `slackChannel` to a channel the app has joined, including the `#`.
4. Adjust the two thresholds if your motion uses different cutoffs.
5. Activate and POST to `/webhook/score-lead`.

## Scoring rules

The **Score Lead** code node applies the points below, then clamps the result to 0–100. The IF nodes only compare that number with the thresholds, so changing a rule does not require rewiring.

| Signal | Points |
| --- | --- |
| Executive title (CEO, founder, chief, owner, partner) | +35 |
| Director, VP, or head title | +25 |
| Manager or lead title | +10 |
| 1000+ / 200+ / 50+ employees | +25 / +20 / +12 |
| Any smaller employee count provided | +4 |
| Annual revenue 10M+ / 1M+ | +20 / +12 |
| Source referral, partner, or customer | +20 |
| Source webinar, demo, or event | +12 |
| Source paid, ads, or linkedin | +6 |
| Website present / phone present | +5 / +5 |
| Message mentions budget, timeline, pricing, purchase, or evaluating | +10 |
| Free consumer email domain | -15 |

## Sample

```json
{
  "email": "ada@acme.example",
  "firstName": "Ada",
  "lastName": "Lovelace",
  "company": "Acme",
  "jobTitle": "VP Sales",
  "employeeCount": 400,
  "annualRevenue": 12000000,
  "source": "referral",
  "website": "https://acme.example",
  "phone": "+1-555-0100",
  "message": "Evaluating pricing this quarter"
}
```

That payload scores as hot with the default thresholds. A gmail address and no company detail scores cold and does not ping Slack.

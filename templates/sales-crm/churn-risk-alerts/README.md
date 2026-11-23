# Churn-risk alerts from a customer sheet

Weekday mornings, score every row on a Customers sheet and alert customer success only when an account is medium or high risk. Low-risk rows produce no message, so a healthy book stays quiet.

## Credentials

- Google Sheets OAuth2 on **Read Customers** and **Log Alerts**.
- Slack OAuth2 on **Post Risk Digest**.
- Gmail OAuth2 on **Email Risk Digest**. Leave `alertEmail` blank and disable the Gmail node if you only want Slack.

## Customer sheet

Tab name `Customers`, one row per account:

| Column | Meaning |
| --- | --- |
| email | Required. Rows without it are ignored. |
| company | Display name |
| plan | `monthly` adds risk when the account is also idle |
| mrr | Number, used as a weight when other risk already exists |
| last_login_at | ISO date or a value Sheets stores that n8n can parse |
| nps | 0–10 |
| open_tickets | Count |
| owner_email | Copied onto the alert log |

Create a second tab, `Churn Alerts`, with the headers written by **Log Alerts**: Alerted At, Email, Company, Plan, MRR, Days Since Login, Risk Score, Risk Tier, Reasons, Owner Email.

## Scoring

| Signal | Points |
| --- | --- |
| No login in 45+ / 21+ / 10+ days | +40 / +25 / +10 |
| Last login missing or unparseable | +15 |
| NPS 6 or below / 7–8 | +25 / +8 |
| 5+ / 2+ open tickets | +20 / +10 |
| Monthly plan and idle 21+ days | +10 |
| MRR 2000+ and the score is already at least 25 | +10 |

35–59 is medium. 60+ is high. The Slack message and the email list every alerted account, high first.

## Setup

1. Paste the spreadsheet ID into **Settings** and set `slackChannel` and `alertEmail`.
2. The schedule is `30 8 * * 1-5` in the n8n instance timezone.
3. Run **Score Accounts** manually once with the sheet populated before you leave it active, and confirm the Churn Alerts tab gained only the risky rows.

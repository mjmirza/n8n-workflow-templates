# Weekly Pipedrive pipeline digest

Monday at 08:00, load every open Pipedrive deal and the stage list, then email and Slack a pipeline summary: deal count, value by currency, count and value by stage, how many deals have not been updated in `staleDays`, and the five largest open deals.

An empty pipeline still sends the digest, with zero deals, so a missed run is obvious.

## Credentials

- Pipedrive API token on **Get Open Deals** and **Get Stages**.
- Slack OAuth2 on **Post Digest**.
- Gmail OAuth2 on **Email Digest**.

## Setup

1. Set `digestEmail`, `slackChannel`, and `staleDays` in **Settings**.
2. Confirm the n8n timezone. The trigger is weekly, Monday, 08:00 in that timezone.
3. Run the workflow once by executing the schedule node before relying on Monday. **Get Open Deals** pages through every open deal.

## Customize

- Stage names come from Pipedrive. A deal whose stage was deleted is labeled `Stage <id>`.
- Value is summed in each deal's own currency. Mixed-currency pipelines are not converted.
- To limit the digest to one pipeline, set a stage filter on **Get Open Deals** or drop other stages inside **Build Digest**.

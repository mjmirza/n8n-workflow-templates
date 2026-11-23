# Stale Pipedrive deal reminders

Each weekday at 09:00 the workflow loads every open Pipedrive deal and posts a Slack reminder for deals whose `update_time` is older than `staleDays` (default 14). Deals are grouped by owner, oldest first, with up to eight lines per owner.

## Credentials

Pipedrive API token on **Get Open Deals** and **Create Follow-up Task**. Slack OAuth2 on **Post Reminder**.

## Setup

1. Import and connect Pipedrive and Slack.
2. Set `slackChannel`. Change `staleDays` if 14 is the wrong idle window.
3. Leave `createTasks` off until you want the side effect. When it is on, every stale deal gets a Pipedrive task due tomorrow, every weekday, which will pile up.
4. Activate. The schedule uses the workflow timezone from n8n settings. Set that timezone before you rely on 09:00.

## What "stale" means

A deal counts when it is open and the later of its update time is at least `staleDays` before the run. Deals with an unparseable timestamp are skipped rather than treated as stale. A morning with nothing idle sends no Slack message.

## Customize

- Point `filters` on **Get Open Deals** at one pipeline by adding `stage_id`, or switch the status filter if you also want lost deals reviewed.
- Replace the Slack text in **Find Stale Deals** if the channel wants value totals per owner. The `deals` array on that item has `value`, `currency`, and `ageDays`.

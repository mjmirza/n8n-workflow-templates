# Meeting follow-up tasks from Google Calendar

Weekdays at 17:30, read today's events on a Google Calendar. For each event that has already ended, create a HubSpot follow-up task due tomorrow for every external attendee, and upsert that attendee as a contact. One Slack message lists what was created.

## Who is skipped

- The calendar owner (`self`) and the organizer
- Attendees who declined
- Anyone whose email domain equals `internalDomain` (set this to your company domain, without the `@`)
- The same event id + email already tasked earlier today. That memory lives in workflow static data and is pruned after 20 days.

Events that have not ended yet are ignored, so a 17:30 run does not task a dinner still on the calendar. All-day events are included once their end date is in the past.

## Credentials

- Google Calendar OAuth2 on **Today's Events**.
- HubSpot private app token on **Upsert Attendee** and **Create Follow-up Task**.
- Slack OAuth2 on **Post Summary**.

## Setup

1. Set `calendarId` to `primary` or another calendar id, `internalDomain` to your company domain, `slackChannel`, and the HubSpot `leadStatus` used for upserts.
2. The cron is `30 17 * * 1-5` in the n8n timezone.
3. Execute the schedule once on a day that already has a finished external meeting. Confirm one HubSpot task per external attendee and a single Slack summary. Run it again and confirm the same meetings are not tasked a second time.

## Customize

- Change the due date expression on **Create Follow-up Task** if follow-ups should be due the same day.
- The task body includes the calendar description. Strip that in **Expand Meetings** if descriptions contain private notes you do not want on the HubSpot engagement.

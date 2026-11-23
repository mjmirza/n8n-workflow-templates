# Calendly booking to HubSpot

When someone books a Calendly event, the workflow upserts the invitee as a HubSpot contact, writes a meeting engagement on that contact, and posts the booking to Slack.

## Credentials

- Calendly personal access token (`calendlyApi`) on **Calendly Booking**. Use User scope. The token needs webhook subscription access.
- HubSpot private app token on **Upsert Invitee** and **Log Meeting**, with contact write and engagement write scopes.
- Slack OAuth2 on the two Slack nodes.

## Setup

1. Import and connect the three credentials.
2. Set `slackChannel` and the HubSpot `leadStatus` in **Settings**.
3. Activate the workflow while n8n is reachable on a public HTTPS URL. Calendly will not deliver to localhost.
4. Book a test event on the connected Calendly user. The invitee email must be present, which Calendly includes by default.

## What gets stored

- Contact: email, name split on the first space, company when a routing question contains "company" or "organization", and the lead status from Settings.
- Meeting: Calendly event name, start, end (or start plus 30 minutes when Calendly omits the end), and the join URL when the location has one.
- The meeting is associated to the HubSpot contact id returned by the upsert.

## Customize

- Add `invitee.canceled` on the trigger and a second branch that creates a note instead of a meeting if you want cancellation alerts.
- Organization scope watches every user in the Calendly account. Leave scope on User unless that is what you want.

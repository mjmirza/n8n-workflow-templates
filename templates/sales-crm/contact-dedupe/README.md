# HubSpot contact dedupe

Accepts a contact payload, searches HubSpot by email, and chooses one action:

- **create** when no contact uses that email
- **update** when the contact exists and an incoming first name, last name, company, job title, phone, or website differs
- **skip** when the contact exists and those fields already match

Create and update both go through HubSpot's email upsert. Skip does not call the write API. Every decision is appended to a Google Sheet and returned on the webhook, so a form can show the caller what happened.

Blank incoming fields do not count as a change. Sending a phone-less payload will not erase a phone that HubSpot already has, because the comparison ignores empty incoming values and the upsert only sends the normalized fields that are present. If you need true field clearing, do that in HubSpot directly.

## Credentials

- HubSpot private app token on **Search HubSpot** and **Upsert Contact**. Search uses the CRM v3 search API with the same private app.
- Google Sheets OAuth2 on **Log Decision**.

## Setup

1. Create a tab named `Dedupe Log` with headers: Received At, Email, Action, Reason, Changes, HubSpot Id.
2. Paste the spreadsheet ID into **Settings**.
3. Activate and POST to `/webhook/crm-contact` with the same shape as the lead-capture template.

## Sample decisions

The first POST for `ada@acme.example` returns `"action": "create"`. An identical second POST returns `"action": "skip"`. A third POST with a new `jobTitle` returns `"action": "update"` and lists `jobTitle: old → new` in `changes`.

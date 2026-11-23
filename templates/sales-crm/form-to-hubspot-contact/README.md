# Form submission to HubSpot contact

Creates or updates a HubSpot contact from a form POST. HubSpot's contact upsert is keyed by email, so a person who submits twice stays one contact. The response includes the HubSpot `vid` and whether HubSpot treated the record as new.

## Credentials

HubSpot private app token (`hubspotAppToken`) on **Upsert Contact**. The private app needs the `crm.objects.contacts` write scope. This file does not contain the token.

## Setup

1. Import the workflow and select the HubSpot credential.
2. In **Settings**, set `leadStatus` to a value that exists in your HubSpot lead status property, or clear it to leave the property untouched when the form omits it.
3. Activate and send form posts to `POST /webhook/hubspot-lead-form`.

## Payload

```json
{
  "email": "ada@acme.example",
  "firstName": "Ada",
  "lastName": "Lovelace",
  "company": "Acme",
  "jobTitle": "VP Sales",
  "phone": "+1-555-0100",
  "website": "https://acme.example"
}
```

`name` is split into first and last when the separate fields are absent. A payload without an email returns HTTP 422 and does not call HubSpot.

## Customize

- Map a hidden form field such as `utm_campaign` by adding it in **Normalize Form** and, if HubSpot should store it, a custom property through the HubSpot node.
- Add the lead-scoring workflow's Slack node after **Upsert Contact** when only some forms should alert the floor.

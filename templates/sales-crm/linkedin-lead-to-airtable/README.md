# LinkedIn lead to Airtable

Stores a LinkedIn lead in Airtable and skips the row when the email is already there. Two payload shapes are accepted.

## Flattened lead

Use this when a lead form tool, or your own bridge, already expanded the answers:

```json
{
  "firstName": "Ada",
  "lastName": "Lovelace",
  "emailAddress": "ada@acme.example",
  "companyName": "Acme",
  "title": "VP Sales",
  "phoneNumber": "+1-555-0100",
  "linkedinProfile": "https://www.linkedin.com/in/ada",
  "formName": "Demo request",
  "campaignName": "Q4 pipeline"
}
```

`email`, `Email`, and `workEmail` are also accepted.

## LinkedIn lead notification

If the body has no email but has `leadGenFormResponse` (the URN LinkedIn sends on a lead gen webhook), the workflow GETs `https://api.linkedin.com/rest/leadFormResponses/{urn}` and picks the first answer that looks like an email. Other answers are not mapped to company or title, because the response identifies questions by id. Use the flattened shape when you need those fields filled reliably.

The LinkedIn HTTP node uses a Header Auth credential. Set the header name to `Authorization` and the value to `Bearer` plus a token that can read lead form responses. Also required by LinkedIn: `LinkedIn-Version: 202511` and `X-Restli-Protocol-Version: 2.0.0`, which the node already sends. Bump the version header when LinkedIn retires it.

## Credentials

- Header Auth on **Get Lead Response**, only needed for the URN shape.
- Airtable personal access token on **Find Existing Lead** and **Create Lead**. Scopes: `data.records:read`, `data.records:write`, `schema.bases:read`.
- Slack OAuth2 on the two Slack nodes.

## Airtable

**Settings** needs `baseId` (starts with `app`) and `tableId` (table id or table name). The table needs these fields:

Name, Email, Company, Job Title, Phone, LinkedIn URL, Form, Campaign, Submitted At.

Duplicate detection is `{Email} = incoming email`. The Email field must be named exactly `Email`.

## Responses

- HTTP 201 and a Slack "LinkedIn lead" message when a row is created.
- HTTP 200 with `duplicate: true` when Email already exists. Slack gets a short duplicate note and no new row is written.
- HTTP 422 when neither an email nor a fetchable lead URN is present.

# Stripe invoice when a Pipedrive deal is won

When a Pipedrive deal first moves to Won, this workflow invoices the attached person in Stripe and writes the hosted invoice URL back onto the deal.

1. Ignore the event unless the current status is won and the previous status was not.
2. Load the Pipedrive person. A deal with no person is reported in Slack and stops.
3. Take the primary email. Convert the deal value to Stripe's minor units (no ×100 for zero-decimal currencies such as JPY). A missing email or a zero value is reported in Slack and stops.
4. Reuse a Stripe customer with that email, or create one.
5. Create a pending invoice item, create a send-invoice due in `daysUntilDue` days (default 14), finalize it, and send it.
6. Add a Pipedrive note on the deal with the invoice number and hosted URL, and post the same link to Slack.

The Stripe invoice metadata field `pipedrive_deal_id` stores the deal id.

## Credentials

- Pipedrive API token on the trigger, **Get Person**, and **Note on Deal**.
- Stripe secret key on the Stripe customer node and the invoice HTTP nodes. The HTTP nodes use the Stripe credential directly. Use a restricted key that can write customers and invoices if you do not want the full secret in n8n.
- Slack OAuth2 on the three Slack nodes.

## Setup

1. Import and connect Pipedrive, Stripe, and Slack.
2. Set `slackChannel` and `daysUntilDue`.
3. Activate on a public HTTPS n8n URL so Pipedrive can deliver the webhook.
4. Test with a deal that has a person, a real email you control, and a small value. Stripe will email that person. Use a Stripe test key until the amounts look right.

## Customize

- Add tax rates or a product price by editing **Create Invoice Item**. The amount today is the whole deal value, one line.
- Collection method is `send_invoice`, so Stripe emails the invoice instead of charging a card on file. Switch the body to `charge_automatically` only if those customers have a default payment method.

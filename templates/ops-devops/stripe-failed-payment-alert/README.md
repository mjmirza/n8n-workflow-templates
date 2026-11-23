# Stripe failed payment alert

Listens for Stripe `invoice.payment_failed` and `charge.failed` events and posts the event type, customer, amount, currency, failure reason, and a Stripe URL to Slack.

Amounts for decimal currencies are converted from Stripe's minor units. Zero-decimal currencies (JPY, KRW, VND, and the other codes listed in **Normalize payment**) are left as Stripe sent them.

A single failed invoice can emit both events. Remove the event you do not want on the Stripe Trigger if that double message is noisy.

## Credentials

- Stripe API secret key, used by the Stripe Trigger to create the webhook endpoint. n8n stores the signing secret on the credential when the trigger activates.
- Slack: bot token with `chat:write`.

## Setup

1. Import `workflow.json`.
2. Attach the Stripe credential on **Stripe Trigger**. Leave the events as `invoice.payment_failed` and `charge.failed`, or narrow them.
3. On **Alert Slack**, set the channel. The template uses `#ops-alerts`.
4. Attach the Slack credential.
5. Activate the workflow. In Stripe Dashboard, confirm the endpoint n8n created.

## Customize

- **Normalize payment** maps both event shapes onto `customer`, `amount`, `currency`, `reason`, and `url`. Add `obj.customer_name` or the invoice id if finance wants them in the message.
- Change the Slack text if the channel should mention an on-call group.

## Test

In Stripe test mode, use the Dashboard or the Stripe CLI to send `invoice.payment_failed` (or fail a test-card payment). The Slack message should name the customer, the amount, and the reason, and it should not contain the raw API key.

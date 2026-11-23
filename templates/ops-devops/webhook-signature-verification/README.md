# Webhook signature verification

Verifies an HMAC-SHA256 signature over the raw request body before any later node treats the payload as trusted. The signing secret stays in an n8n Crypto credential. It is not stored in this workflow.

Callers send:

```
X-Signature-256: sha256=<hex hmac of the raw body>
```

A bare hex digest is also accepted. Comparison is constant-time. A missing or wrong signature gets HTTP 401 and `{"ok": false}`. A match gets HTTP 200 and `{"ok": true}`. The verified JSON body is left on **Check signature** for you to branch from.

## Credentials

- Crypto credential on **HMAC SHA256**. Create it under Credentials, choose Crypto, and set the HMAC secret to the same value the caller uses. Do not put that value in the workflow.

## Setup

1. Import `workflow.json`.
2. Create the Crypto credential and select it on **HMAC SHA256**.
3. Leave **Signed webhook** on path `signed-events`, response mode **Using Respond to Webhook Node**, and **Raw Body** enabled. Raw Body is what makes the HMAC cover the exact bytes the caller signed.
4. Activate the workflow.

## Customize

- Insert your own nodes between **Accepted event** and **Respond 200**. **Accepted event** carries the parsed JSON body.
- **Check signature** looks at header `x-signature-256`. Change `headerName` in that code node if your provider uses another header.
- This pattern is the generic `sha256=` hex HMAC. Stripe and GitHub each add their own timestamp or prefix; use their trigger nodes when you are verifying those products, and use this workflow for your own callers.

## Test

```bash
SECRET='the-value-in-the-crypto-credential'
BODY='{"hello":"world"}'
SIG=$(printf '%s' "$BODY" | openssl dgst -sha256 -hmac "$SECRET" | awk '{print $2}')

curl -sS -D -   -H 'Content-Type: application/json'   -H "X-Signature-256: sha256=$SIG"   --data "$BODY"   https://<your-n8n-host>/webhook/signed-events
```

The matching signature returns HTTP 200. Changing one byte of the body, or omitting the header, returns HTTP 401.

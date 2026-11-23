# Human approval before sending email

Drafts an email with a model, posts it to Slack with Send and Discard buttons, and uses Gmail to send only after someone approves.

## Use case

A teammate, a form, or another workflow asks for an email to a customer. The text should be drafted automatically, but it should not leave the building until a person has seen it. Slack's **Send and Wait for Response** operation pauses the execution. **Send Email** is on the approved branch only.

The webhook answers as soon as the request is accepted (`responseMode` is `onReceived`), because the approval can sit for a while. The result is posted back to Slack, not returned on the original HTTP call.

## Required credentials

| Node | Credential |
| --- | --- |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |
| Wait For Approval, Confirm Sent, Confirm Discarded, Reject Bad Address | Slack bot token (`slackApi`) with `chat:write`. The same credential can be selected on each Slack node. |
| Send Email | Gmail OAuth2 (`gmailOAuth2`) with send access |

The workflow has to be active. Slack resumes the waiting execution by calling back into n8n, which does not happen while the workflow is inactive.

## Setup

1. Import `workflow.json`.
2. Set `slackChannel` in **Config** to a channel the app is in.
3. On **Wait For Approval**, confirm the operation is **Message → Send and Wait for Response**, response type **Approval**, with both buttons.
4. Attach credentials and activate. The webhook path is `email-approval`.

## Input

```json
{
  "to": "accounts@northwind.example",
  "subject": "April invoice",
  "purpose": "Send the April total they asked for.",
  "context": "Invoice NW-441, total 88 USD, due 2026-04-16.",
  "requester": "leo"
}
```

`to` has to look like an email address. Anything else is posted to Slack and the model is not called. The prompt is not allowed to invent refunds, discounts, or legal outcomes.

## How to customize

- **Read Decision** treats both `data.approved` and `approved` as the decision, because the wait node has used each shape. A missing flag counts as not approved, so a timeout or a discard does not send.
- Add a wait limit under the options of **Wait For Approval** if a draft should expire instead of waiting indefinitely. The default is to wait until someone clicks.
- Edit the prompt on **Draft Email** for tone. The subject and body that get sent are the approved text from **Assemble Draft**, not a second generation.
- Gmail send does not add a signature. Put the mailbox signature in the prompt if you want one, and approve that text like the rest of the draft.

## Test

POST the sample to the production webhook. Confirm the HTTP call returns quickly, Slack shows the draft with both buttons, and Gmail Sent is unchanged. Click **Discard** and confirm the discarded note. POST again, click **Send email**, and confirm the message is in Sent and Slack says it was sent.

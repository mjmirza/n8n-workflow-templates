# Gmail reply drafts

When unread mail arrives, drafts a reply in the same thread and posts the draft to Slack. The message is not sent.

## Use case

A shared mailbox or a founder inbox gets mail that deserves a human reply, but the first draft is repetitive. This workflow writes that draft, files it under the Gmail thread, and tells Slack so someone can edit and send it. Automated senders (noreply, mailer-daemon, notification addresses) are skipped before the model is called.

## Required credentials

| Node | Credential |
| --- | --- |
| Gmail Trigger, Create Gmail Draft, Mark As Read | Gmail OAuth2 (`gmailOAuth2`) with read, compose, and modify |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |
| Notify Slack | Slack bot token (`slackApi`) with `chat:write` |

Gmail's draft create uses the same OAuth credential as the trigger. Grant compose access or the draft step will fail after the model has already run.

## Setup

1. Import `workflow.json`.
2. Set `slackChannel` in **Config**.
3. Attach credentials on all five nodes that need them (the three Gmail nodes can share one credential).
4. On **Gmail Trigger**, run a manual execution once and confirm a draft appears in Gmail rather than in Sent.
5. Activate the workflow. The trigger polls every minute. Change **pollTimes** if that is too chatty.

The Gmail filter is `is:unread -category:promotions -category:social`. After a draft is stored, **Mark As Read** runs so the next poll does not write a second draft for the same message.

## How to customize

- Edit the prompt on **Write Reply**. It is instructed not to invent order numbers or refunds and to ask a question when the snippet is thin. Gmail's simple payload does not include the full body. Switch the trigger off of simple mode and map the plain-text body if you need the whole letter.
- **Worth a draft?** drops mail with no parseable address and mail from automated senders. Add domains to the regular expression in **Prepare Message**.
- Remove **Mark As Read** if you would rather keep the unread flag and deduplicate another way.
- There is no send step on purpose. Pair this with the approval-before-send template if a draft should wait for a Slack approval before it goes out.

## Test

Send a normal email to the connected mailbox from a personal address. Wait for the poll or execute the trigger. Confirm a draft threaded under that message, a Slack post that says the draft was not sent, and that Sent mail did not change. Send a noreply notification and confirm it produces no draft.

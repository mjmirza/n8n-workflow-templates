# Unread email digest to Slack

Once a day, reads recent unread Gmail, writes one digest, and posts it to Slack.

## Use case

Instead of triaging a full inbox before stand-up, the operator gets a single Slack message: what needs a reply today, what is a receipt or a calendar note, and what can wait. The messages are left unread.

## Required credentials

| Node | Credential |
| --- | --- |
| Get Unread | Gmail OAuth2 (`gmailOAuth2`) with read access |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |
| Post Digest and Post Empty Digest | Slack bot token (`slackApi`) with `chat:write` |

Nothing secret is stored in the workflow file.

## Setup

1. Import `workflow.json`.
2. In **Config**, set `slackChannel` to the channel name without `#`. Change `gmailQuery` if the default Gmail search is wrong for this mailbox. The default is unread mail from the last day, excluding Promotions and Social.
3. Attach credentials. Invite the Slack app to the channel.
4. Run **Manual Trigger** once and read the Slack message before you rely on the schedule.
5. Activate the workflow. **Schedule Trigger** fires every day at 08:00 in the timezone configured on the n8n instance.

## How to customize

- Change the schedule interval on **Schedule Trigger**. `triggerAtHour` is 8.
- **Get Unread** stops at 20 messages. Raise `limit` if the mailbox is busier, and keep the cap in **Build Digest Source** in step with it.
- Edit the prompt on **Summarize Inbox** to match the voice you want (for example, always lead with customer mail).
- To mark mail as read after it is summarized, add a Gmail **Mark as Read** node after **Post Digest**. Leaving mail unread is deliberate so the digest is not the only copy.
- An empty result posts a one-line "no unread mail" message so a broken Gmail credential is easier to tell apart from a quiet morning: a credential error fails the execution, a quiet morning still posts.

## Test

Execute the workflow manually with a mailbox that has at least one unread message from the last day. Confirm one Slack message, not one per email. Archive or mark that mail read, run again, and confirm the empty-digest message.

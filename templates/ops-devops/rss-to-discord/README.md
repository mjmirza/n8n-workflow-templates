# RSS feed to Discord

Checks a feed once an hour and sends each new item to a Discord channel. The first execution records every current link and only posts items published in the last 24 hours, so activating the workflow does not dump the whole backlog into the channel.

Seen links are stored in workflow static data (the last 300 links).

## Credentials

- Discord bot token with Send Messages permission in the target channel. The bot must be in the server.

## Setup

1. Import `workflow.json`.
2. On **Feed URL**, replace `https://hnrss.org/frontpage` with your feed.
3. On **Send to Discord**, set **Server** and **Channel** by ID. Developer mode in Discord shows the ids when you copy them.
4. Attach the Discord bot credential.
5. Execute manually once and review which items were posted.
6. Activate the workflow.

## Customize

- Change the Schedule Trigger interval if the feed updates faster or slower than hourly.
- In **Keep unseen items**, change the `24 * 60 * 60 * 1000` window used on the first run, and the `300` cap on remembered links.
- The Discord **Message** is the item title and link. Add `contentSnippet` if you want a short excerpt. Discord rejects messages over 2000 characters; the code node already truncates.

## Test

Run the workflow twice manually against a live feed. The first run posts only recent items. The second run should post nothing unless a new item appeared. Deactivate and duplicate the workflow if you want a second feed; static data is per workflow.

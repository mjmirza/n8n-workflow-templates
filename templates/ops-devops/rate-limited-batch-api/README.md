# Rate-limited batch API calls

Once an hour, loads a page of pending jobs and calls a process endpoint for each job in batches of 5, waiting one second between batches. HTTP errors are collected. Slack is notified only when the list request fails or at least one process call returns a non-2xx status.

Expected list response: a JSON array, or an object with a `jobs` or `data` array. Each element needs an `id`. Each id is sent as JSON `{"id": "..."}` to `POST {baseUrl}/v1/jobs/{id}/process`.

## Credentials

- Header Auth credential used by both HTTP Request nodes. Put the API header name and value in that credential, then select it on **List pending jobs** and **Process job**.
- Slack: bot token with `chat:write`, used only for failures.

## Setup

1. Import `workflow.json`.
2. On **Config**, set `baseUrl` to the API origin, without a trailing path.
3. Create a Header Auth credential and select it on both HTTP Request nodes.
4. On **List request failed** and **Process calls failed**, set the Slack channel. The template uses `#ops-alerts`.
5. Run it manually against a test account before activating the hourly trigger.

## Customize

- **Loop over jobs** batch size is 5. **Wait 1 second** is the pause between batches. Lower the batch size or raise the wait if the API allows fewer calls.
- **Prepare jobs** is where the list and process paths are built. Change those strings if your API is not `/v1/jobs`.
- The process call sends the job id only. Add fields in **Process job** JSON body if the API needs more.
- Both HTTP nodes return the status code instead of failing the workflow, so one bad job does not abandon the rest of the batch.

## Test

Point `baseUrl` at a test API that returns two or three jobs, with Header Auth attached. Run the workflow manually. Executions should show batches of at most 5, a one-second wait between them, and no Slack message when every process call returns 2xx. Force one process URL to return 500 and confirm a single Slack summary lists the failed count.

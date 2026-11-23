# Error handling patterns

A workflow that only describes the successful path will fail silently or stop in a place where nobody is looking. Decide, for each node, whether a failure should stop the execution, take an error branch, or raise a page. Use the same pattern across templates so operators know where to look.

## Choose the node error behavior

Open a node, then open its **Settings**. The **On Error** choice has three outcomes:

- **Stop workflow** is the default. The execution is marked failed, and later nodes do not run. Use this when continuing would write half-finished data.
- **Continue** lets the main output run even though the node failed. The item then carries an error object. Use this only when a later node explicitly checks that error. Continuing by accident hides failures.
- **Continue (using error output)** sends failed items down a separate error output, and successful items down the main output. Use this when one bad row should be recorded and the rest of the batch should continue.

Prefer the error output over a bare Continue. The canvas shows the failure path, and successful items are not mixed with error items on the same connection.

## Branch on HTTP status instead of treating every non-200 as fatal

For an HTTP Request node that calls an API you expect to throttle or validate:

1. Add the option **Response** → **Include Response Headers and Status**.
2. Turn on **Never Error** if you want 4xx responses as data rather than as a node failure. Leave it off if a 4xx should still fail the node and follow **On Error**.
3. Add an **IF** node on the status. A typical split is `statusCode` smaller than 400 on the true branch, and everything else on the false branch.

A 404 that means "record not found, create it" is a business result, not an outage. A 401 is a credential problem and should stop. A 429 or a 500 is a candidate for the retry pattern described in the retries guide. Handle those three differently instead of sending every failure to the same Slack channel with the same text.

## Fail on purpose with Stop and Error

Some failures are rules, not node crashes. Example: the form payload has no email address, or a payment amount is negative. Add a **Stop and Error** node on that branch and set the error message to a sentence an operator can act on: `Form payload is missing email`. The execution is marked failed, and any error workflow attached to this workflow will run.

Use Stop and Error when you want the execution log to show a failure. Use an IF branch that simply ends when the skipped case is normal, such as "no rows today", so you do not page anyone.

## Attach one error workflow

An error workflow runs after an execution fails. One error workflow can serve many workflows.

1. Create a workflow whose first node is **Error Trigger**. Name it `Error Handler`.
2. Add the nodes that should react. A minimal handler posts to a Slack or email node with the workflow name, the last node that ran, the error message, and a link to the execution.
3. Save it.
4. On each workflow you want to watch, open **Workflow settings** (under the menu, **Settings**) and set **Error workflow** to `Error Handler`.

The Error Trigger receives JSON shaped like this for a failure after the workflow has started:

```json
{
  "execution": {
    "id": "231",
    "url": "https://n8n.example.com/execution/231",
    "error": { "message": "Form payload is missing email" },
    "lastNodeExecuted": "Stop and Error",
    "mode": "trigger"
  },
  "workflow": { "id": "1", "name": "Intake form" }
}
```

`execution.url` is present when the execution was stored. Build the Slack text from `$json.workflow.name`, `$json.execution.lastNodeExecuted`, `$json.execution.error.message`, and `$json.execution.url`.

If the trigger node itself fails while the workflow is activating, the payload is different: the detail is under `trigger.error`, and there is often no execution id, because the run never started. Read both shapes in the handler, or the activation failure will produce a second error inside the error workflow.

## Keep the handler smaller than the workflow it watches

The error workflow should notify and, when it is safe, record. Do not call the same API that just failed, or a failure will loop. Do not try to "fix" the data in the handler. Fix the source workflow, then replay the failed execution from **Executions** if the data is still there.

Check the handler with a deliberate Stop and Error, not by waiting for a real outage. Execute the source workflow, confirm a failed execution, and confirm the handler ran and the message contains the node name and the URL.

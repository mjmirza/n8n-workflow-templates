# Testing workflows

Test a template before you activate it, and test it again after you change a mapping. n8n runs a manual execution differently from a production one: pinned data and the webhook test URL exist only for the manual case. A workflow that succeeds when you press **Execute workflow** can still be wrong once a real trigger fires.

## Use one known input

Pick a payload you understand. For a webhook or app trigger, save one realistic body with private fields replaced by obvious stand-ins (`user@example.com`, an id of `1001`). Two ways to replay it:

- **Pin data** on the trigger node. Open the node output, pin the current output, and edit the pinned JSON so it matches your fixture. Later manual runs reuse the pin and do not call the outside service.
- **Execute step** on a node further down the canvas. n8n uses the incoming items already produced by the previous nodes. You can re-run the mapping node without repeating the trigger.

Clear the pin before you export a template, unless the pinned JSON is the documented sample and contains nothing private. Pinned data is stored in the workflow file.

## Listen on the test URL, then on the production URL

A Webhook node shows two URLs.

- The **Test URL** contains `/webhook-test/`. It receives events only while you are listening from the editor.
- The **Production URL** contains `/webhook/`. It receives events only after you save and set the workflow to **Active**.

Exercise both. Send the fixture to the test URL and confirm each node's output. Activate the workflow, send the same fixture to the production URL, and open **Executions**. The production run is the one that will happen when you are not looking at the canvas. If you put the test URL into the external service's settings, that service will appear to send events and n8n will never record them.

Manual executions of a Schedule Trigger run once, immediately. They do not prove the cron expression. Read the schedule back in the node's parameters and compare it with the timezone in `GENERIC_TIMEZONE` on a self-hosted instance. A schedule of `0 9 * * 1-5` is 09:00 in that timezone, not necessarily in yours.

## Check the data, not only the green check

Open the output of every node for the trial run.

- Counts: a list endpoint that should return 100 rows but returns one row is often a pagination or Split Out mistake.
- Fields the next node reads: if an expression says `{{ $json.customer.email }}`, that path must exist on the item, not only in the API docs.
- Types: ids that arrive as numbers and later get compared to strings will fail an **IF** that looked fine in a pin you typed by hand.
- Side effects: for any node that writes, posts, or sends, point the trial at a sandbox account, a test channel, or a test spreadsheet. Say so in the template README.

When a node fails, read the error on that node before you change anything downstream. The execution view names the node and shows the input it actually received.

## Load a previous execution

From **Executions**, open a past run and copy its data back into the editor when you need to reproduce a failure. That is more reliable than guessing a payload. After you fix the workflow, run the same payload once and confirm the node that failed now produces the output you expect.

## A short checklist before you call a template done

1. Manual run with the fixture succeeds, and each node output matches the README.
2. Production path succeeds once: active workflow, real trigger, new row in **Executions**.
3. A known bad payload takes the error branch or the Stop and Error node, and does not write a partial record.
4. Pins and execution data you used for debugging are cleared if they contain anything you would not commit.
5. Credentials used in the test are named sandbox or test, and the README tells the next person which credential type to attach.

Repeat the manual run after any edit to a Code node or an expression. Those edits do not change the canvas layout, so they are easy to save without noticing that the output shape changed.

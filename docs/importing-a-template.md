# Import a workflow template

Templates in this repository are ordinary n8n workflow files. Each one is a JSON document that describes nodes, connections, and settings. Importing creates a new inactive workflow in your instance. It does not copy credentials, and it does not turn the workflow on.

Import into a development instance or a personal project first. Confirm the workflow with sample data, then copy it to the instance that receives real traffic.

## Import from the editor

1. Open n8n and go to **Workflows**.
2. Create a workflow or open an empty one.
3. Open the workflow menu (the three-dot control beside the name) and choose **Import from File**.
4. Select the template's `.json` file.
5. Wait until the canvas finishes loading, then choose **Save**.

n8n keeps the workflow name stored in the file. Rename it if your project already has a workflow with that name, so execution history stays easy to tell apart.

The same menu offers **Import from URL** when the file is reachable over HTTP. Point it at the raw file URL from this repository, not at the GitHub HTML page. A raw URL ends with the `.json` file name and returns JSON when you open it in a browser.

## Import from the command line

On a self-hosted instance you can import without the editor. From a shell that has the `n8n` command, or from a one-off container on the same database:

```bash
n8n import:workflow --input=./templates/notify-on-form-submit/workflow.json
```

To import every file in a directory, add `--separate`:

```bash
n8n import:workflow --separate --input=./templates
```

The CLI writes the workflow into the database of whatever instance you pointed it at. Check `DB_TYPE` and the `DB_POSTGRESDB_*` variables before you run it against a shared server. Importing into the wrong database is the usual way a test workflow shows up in production.

## Reattach credentials

A template may name a credential on a node, for example `Slack account`. That name is a label. The credential id inside the file belongs to the instance that exported it, so your instance will show the node as missing a credential.

Open each node that has a credentials warning:

1. In **Credential to connect with**, choose **Create new**, or select a credential that already exists on your instance.
2. Use the credential type named in the template's own README. A Header Auth credential will not satisfy a node that expects a Slack API credential.
3. Save the node, then save the workflow.

Do this for every warning before the first execution. A workflow can look complete on the canvas and still fail on the first node that needs a secret.

## Replace values that are specific to the author

Search the canvas for anything the template README lists as a setting you must change. Typical spots are a **Set** or **Edit Fields** node named `Config`, and parameters on the first HTTP or app node:

- Base URLs such as `https://api.example.com`
- Channel, list, spreadsheet, or mailbox ids
- Recipient addresses used only as examples
- Timezone or locale strings

Edit those fields in place. Leave the node structure alone until you have seen one successful run.

## Run once before you activate

Most templates include a **Manual Trigger** or a note describing how to fire the real trigger.

1. If the trigger is a Webhook node, choose **Listen for test event** and send one request to the **Test URL**. The test URL contains `/webhook-test/`. The **Production URL**, which contains `/webhook/`, stays silent until the workflow is active.
2. If the trigger is manual, choose **Execute workflow**.
3. Open each node and read its output. Confirm the fields the next node expects are actually present.
4. If a node has pinned data, that pin is sample input for manual runs only. Review it. If it contains data you do not want stored in the workflow, clear the pin from the node's output menu before you save.

When the trial run matches the README, turn the workflow **Active**. Active is the toggle at the top of the editor. Scheduled triggers and production webhooks do not fire while it is off.

After the first real event, open **Executions** and confirm the run is a success. If it failed, the execution view shows the node and the error message. Fix that node, save, and let the trigger fire again. Keep the workflow inactive while you are still changing credentials or endpoint URLs.

# Versioning workflows in git

n8n keeps workflows in its database. Git does not see them until you export a JSON file and commit it. This repository is a collection of those exports, plus a README beside each template. Treat the file as the source you review, and the database as a runtime copy.

## Export a file you can diff

From the editor, open the workflow menu and choose **Download**. The file is the workflow JSON.

From a self-hosted shell, the CLI writes the same kind of document:

```bash
n8n export:workflow --id=42 --output=./templates/notify-on-form-submit/workflow.json
```

`--all` exports every workflow. Do that into a scratch directory and copy out only the file you mean to publish. Exporting everything and committing the directory will publish personal experiments and other people's drafts.

Format the JSON with a stable pretty-printer before you commit (two-space indent, trailing newline) so the next export does not rewrite every line.

## Strip instance-specific fields

An export from a live instance contains fields that should not travel with a template:

| Field | What to do |
| --- | --- |
| `id`, `versionId` | Remove them so an import creates a new workflow instead of trying to adopt an id from your database. |
| `meta.instanceId` | Remove it. It identifies your instance. |
| `active` | Remove it, or set it so the import does not arrive already running. |
| `pinData` | Remove it unless it is a small, fake fixture the README describes. |
| `credentials` on a node | Keep the credential type and the human name. The id will not match the next instance; the importer reselects a credential. Never paste secret values into this object. |
| `staticData` | Set it to `null` unless the workflow truly needs a stored cursor and that cursor is not customer data. |

Search the file for `Bearer `, tokens, and email addresses that are not obvious examples. Node parameters are the usual place a copied URL with an API key hides.

A cleaned file still has `name`, `nodes`, `connections`, `settings`, and `meta`. `settings.executionOrder` should stay `v1` for workflows built with the current execution order. Node `type` and `typeVersion` must be the values n8n wrote. Do not hand-edit `typeVersion` to an older number to "simplify" the file; the node parameters for that version will not match.

## Lay files out so a change is reviewable

Use one directory per template:

```text
templates/notify-on-form-submit/workflow.json
templates/notify-on-form-submit/README.md
```

The README covers the use case, the credentials, the setup steps, and what to edit. The JSON is only the workflow.

Commit a behavior change separately from a layout change. Dragging nodes rewrites every `position` array and hides a real parameter edit in the same diff. Move nodes, commit, then change parameters.

When you review a pull request, read the `parameters` objects and the `connections` map. Ignore `position` unless the change claims to rewire the flow. In `connections`, the key is the source node name, and each destination names the next node. A renamed node has to change in both places. If you rename a node and forget `connections`, the import loads a workflow that stops halfway.

## Do not commit the credential export

`n8n export:credentials` is a backup of an instance, encrypted with that instance's `N8N_ENCRYPTION_KEY`. It is not a template and it does not belong next to workflow JSON. People who import the workflow create their own credentials from the README.

## Built-in source control is a different path

n8n Enterprise can push workflows to a git repository from the instance. That feature is for operating one environment. This library is for files other people import by hand. Keep publishing to this repository as a cleaned download or CLI export, even if your own instance also uses source control. The two copies will drift if you commit the raw source-control payload without the cleanup above.

After you change the file, import it into a second project or a local instance and run the test in the README. A JSON file that passes a visual diff can still fail to import if a connection names a node you renamed.

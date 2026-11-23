# Contributing

This repository collects n8n workflows other people can import. A contribution is either a template (a workflow JSON file plus a short README) or a guide under `docs/`. Keep a pull request to one template or one guide so it can be reviewed on its own.

## What to add

A template belongs here when it does a job a stranger can recognize, runs without private endpoints, and includes enough written steps to configure it. A workflow that only makes sense inside one company, or that calls an internal hostname, should stay in that company's repository.

A guide belongs in `docs/` when it explains one task: importing, credentials, a node pattern, or how to run n8n. Write concrete steps and examples. A paragraph that only links at the official docs, with no steps of its own, is not a guide.

Do not add credentials, tokens, private URLs, or exports from `n8n export:credentials`. Do not commit `.env` files.

## Template layout

One directory per template:

```text
templates/<slug>/workflow.json
templates/<slug>/README.md
```

`<slug>` is lowercase words separated by hyphens, matching the workflow's purpose (`notify-on-form-submit`). The README is the part a person reads before they import. Use these sections, in this order:

1. **Use case.** What starts the workflow, what it does, and what it deliberately does not do.
2. **Required credentials.** Credential type, where to create it, and which node uses it. Name scopes or roles when the service has them.
3. **Setup.** Import steps that are specific to this workflow, then every value the user must replace.
4. **Customize.** The nodes and fields that are safe to change, and the ones that break the flow if they are renamed.
5. **Test.** A sample input and the output that shows the run worked.

Link node names in the README to the names in the JSON. If you rename a node, change both.

## Workflow JSON conventions

Export from the editor with **Download**, or with `n8n export:workflow --id=<id> --output=workflow.json`. Then edit the file so it is a template rather than a backup of your instance.

Required top-level shape:

```json
{
  "name": "Notify Slack when a form is submitted",
  "nodes": [],
  "connections": {},
  "pinData": {},
  "settings": { "executionOrder": "v1" },
  "staticData": null,
  "meta": { "templateCredsSetupCompleted": false }
}
```

`nodes` and `connections` are the workflow. Everything else follows these rules:

- `name` is a sentence a person can scan in the workflow list. Do not leave the default "My workflow".
- Remove `id`, `versionId`, `active`, and `meta.instanceId`. Those belong to the instance that exported the file.
- `pinData` is `{}` unless you ship a small fake fixture and the README explains it. Never pin a real customer payload.
- `settings.executionOrder` stays `v1`.
- `staticData` is `null` unless the workflow stores a cursor and that value is safe to publish.
- Each node keeps the `type`, `typeVersion`, `parameters`, `name`, `position`, and `id` that n8n wrote. Generate a new UUID for `id` if you built the node by hand. Do not change `typeVersion` without re-saving the node in the editor so parameters match that version.
- Node `name` values are unique. `connections` keys are source node names, and each entry points at a target `node` name. After a rename, both must match.
- A `credentials` object on a node may keep `name` so the importer sees what to attach. It must not contain secret material. Assume the id will be replaced on import.
- Parameters must not contain API keys, bearer tokens, or passwords. Put a placeholder expression that reads from a `Config` node (`{{ $('Config').item.json.baseUrl }}`) and document the field in the README.
- Pretty-print with two-space indentation and a trailing newline so later exports stay reviewable.

Before you open the pull request, import the cleaned JSON into a fresh workflow, attach test credentials, and run the test from the README. A file that looks right and fails to import is not done.

## Guides

Guides are Markdown files in `docs/`. One topic per file. Start with the task, then the steps, then the mistakes that waste an afternoon. Include the node names, expression syntax, or commands a reader would type. Do not document a secret by inventing a realistic token. Refer to environment variables by name.

## Pull requests

Describe the use case in the pull request body in a few sentences. Say which n8n version you used to export or to test. If you changed a template that was already in the repository, say what the behavior change is, not only which nodes you touched.

A review looks at the README against the JSON: every credential named in prose exists as a credential slot on a node, every value the user must change appears in the README, and a search of the diff finds no secrets.

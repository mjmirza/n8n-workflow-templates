# Environment variables and secrets

n8n uses environment variables for two very different jobs. One job is configuring the instance: database, public URL, encryption, and process mode. The other job, which is easy to get wrong, is feeding values into a workflow. Keep those jobs apart. Instance configuration belongs on the host. Workflow inputs belong in nodes or in n8n's own variable store.

## Configure the instance from the environment

Set these on the process that runs n8n (the container environment, a systemd unit, or your process manager). Do not put them in a workflow.

| Variable | Role |
| --- | --- |
| `N8N_ENCRYPTION_KEY` | Encrypts credentials at rest. Every main process, worker, and webhook process must use the same value. |
| `DB_TYPE` and `DB_POSTGRESDB_*` | Point n8n at PostgreSQL. Required once you leave the default SQLite database. |
| `N8N_WEBHOOK_URL` | Public base URL used to build webhook and OAuth redirect links. On versions before 2.30.0 the name is `WEBHOOK_URL`. |
| `GENERIC_TIMEZONE` and `TZ` | Timezone for schedule triggers, and the timezone of the container clock. |
| `EXECUTIONS_MODE` | `regular` for a single process, or `queue` when workers take executions. |

Generate an encryption key on the machine that will store it, and keep the value in your secret manager:

```bash
openssl rand -hex 32
```

If the key changes, n8n cannot decrypt credentials that were saved with the old key. Treat it as permanent. Back it up with the same care as the database.

Some variables accept a `_FILE` suffix so the process reads the value from a file path instead of the environment block. That keeps the secret out of `docker inspect` output. Use it when your orchestrator can mount a file and you would otherwise pass the key as a plain environment entry.

A Compose file in git should reference names, not values:

```yaml
environment:
  N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
  DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}
  N8N_WEBHOOK_URL: ${N8N_WEBHOOK_URL}
```

Provide the values in the environment of the deploy job, or in a file that is not committed. Add `.env` to `.gitignore` if you keep a local env file for Docker Compose.

## Do not read host environment variables from workflows

n8n blocks `$env` in expressions and `process.env` in the Code node by default (`N8N_BLOCK_ENV_ACCESS_IN_NODE`). Leave that block in place.

Anyone who can edit a workflow can read every variable the process exposes. That includes `N8N_ENCRYPTION_KEY` and the database password if those sit in the same environment. Opening the block to pass a single base URL is not a good trade.

## Pass per-environment values in the workflow

For a base URL, a list id, or a page size, put the values in a **Set** or **Edit Fields** node named `Config` at the start of the workflow. Later nodes read them by name:

```text
{{ $('Config').item.json.baseUrl }}/v1/orders
```

Copying the workflow to another instance is then a matter of editing `Config`. The rest of the canvas stays the same, and a diff of the template shows exactly which settings an operator is expected to change.

On plans that include n8n Variables, you can store the same settings outside any one workflow and read them with `$vars`. A variable is still not a place for a raw API token. Tokens belong in credentials, where they are encrypted and can be reused without appearing in execution data.

## External secret stores

Enterprise installations can connect an external secrets backend and map those secrets onto credential fields. The workflow still references a credential. The secret is resolved when the credential is used. That is the supported way to keep a token in a vault and out of both the workflow JSON and the n8n database.

## What to check before you share a workflow

- Execution data can contain headers and request bodies. Turn off saving of manual execution data you do not need, and avoid logging credentials in a Code node with `console.log`.
- A pinned output on a node is stored in the workflow. Clear pins that contain private responses before you export.
- Search the exported JSON for `Bearer `, `api_key`, and `password` before you commit. Those strings should not appear. Credential fields should be ids and names only.

Instance variables configure n8n. Credentials hold secrets. A `Config` node holds the plain settings that change between environments.

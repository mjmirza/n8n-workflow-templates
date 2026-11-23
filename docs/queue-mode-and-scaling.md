# Queue mode and scaling

A single n8n process both serves the editor and runs workflows. That is the `regular` execution mode, and it is the right start. Queue mode splits the work: one main process accepts triggers, Redis holds a queue of execution ids, and worker processes run the workflows. You add capacity by adding workers, not by making the editor process larger.

Use PostgreSQL in queue mode. n8n does not support queue mode on SQLite. Every process must reach the same Postgres database and the same Redis.

## What runs where

1. The main process serves the UI and the API, holds schedules, and receives webhook requests. It creates an execution row and pushes the id to Redis. It does not run the workflow when mode is `queue`.
2. A worker pulls that id, loads the workflow from the database, runs it, writes the result back, and tells Redis it finished.
3. Optional webhook processes answer production webhook traffic so the main process can stay responsive for people using the editor. They also need `EXECUTIONS_MODE=queue` and access to Redis.

Workers do not serve the editor. Do not point your browser at a worker and expect the UI.

## Settings every process shares

Set these on main, on every worker, and on every webhook process:

| Variable | Value |
| --- | --- |
| `EXECUTIONS_MODE` | `queue` |
| `N8N_ENCRYPTION_KEY` | The same key the main process uses. Workers cannot decrypt credentials without it. |
| `DB_TYPE` | `postgresdb`, plus the same `DB_POSTGRESDB_*` values |
| `QUEUE_BULL_REDIS_HOST` | Redis hostname |
| `QUEUE_BULL_REDIS_PORT` | `6379` unless Redis listens elsewhere |

Add `QUEUE_BULL_REDIS_PASSWORD` and `QUEUE_BULL_REDIS_USERNAME` when Redis requires them. `QUEUE_BULL_REDIS_DB` defaults to `0`. `QUEUE_BULL_REDIS_TIMEOUT_THRESHOLD` is how long n8n waits, in milliseconds, when Redis is unreachable before it exits (default `10000`).

Set the public URL on the main process: `N8N_WEBHOOK_URL` on n8n 2.30.0 and later, `WEBHOOK_URL` before that.

## Start the processes

Redis has to be up first. Then the main process, then workers:

```bash
n8n start
n8n worker --concurrency=5
```

In Docker the worker command replaces the default start command:

```text
n8n worker --concurrency=5
```

The image entrypoint accepts `worker` and `webhook` as commands. Pass the same environment block you pass to main, including the encryption key and the database variables.

`--concurrency` is how many jobs one worker runs at once. The default is 10. n8n recommends 5 or higher. A fleet of workers each set to 1 opens a lot of database connections and still processes slowly. Prefer fewer workers at concurrency 5 or more, and watch the Postgres connection count as you add them.

`N8N_GRACEFUL_SHUTDOWN_TIMEOUT` (default 30 seconds) is how long a worker spends finishing current jobs when it is told to stop. Set it above your typical job length if you deploy often, or jobs will be marked failed mid-run when the process exits.

Turn on `QUEUE_HEALTH_CHECK_ACTIVE` if you want `/healthz` and `/healthz/readiness` on the worker. Point the orchestrator at readiness so it does not send work to a worker that cannot see Redis or Postgres.

## Webhook processes and the load balancer

Start extra webhook processes with `n8n webhook` when production webhook volume is the bottleneck. Put them behind a load balancer. Send these paths to the webhook pool:

- `/webhook/` for production webhook triggers
- `/webhook-waiting/` for nodes that wait for a human reply

Send everything else, including `/webhook-test/` and the editor, to the main process. Manual test URLs must reach main. Do not put the main process in the webhook pool if you are trying to protect the editor from traffic.

Set `N8N_DISABLE_PRODUCTION_MAIN_PROCESS=true` on main when webhook processes own production traffic, so main does not also run those requests.

If you change the webhook path with `N8N_ENDPOINT_WEBHOOK`, update the load balancer paths to match.

## Binary data and large webhook responses

In queue mode the worker and the process that accepted the webhook are different machines. Binary data stored on a local disk is invisible to the other process. Use the database, or object storage (S3 or Azure) when your license includes it, or mount the same volume at the same path on every process. The default keeps binary data in the database.

A **Respond to Webhook** body travels back through Redis. `N8N_WEBHOOK_RESPONSE_RELAY_SIZE_MAX` caps that message, in MiB, and defaults to 64. From n8n 2.34.0, workers can set `N8N_WEBHOOK_RESPONSE_RELAY_OFFLOAD_ENABLED=true` and a shared `N8N_DEFAULT_BINARY_DATA_MODE` (`s3` or `azure` are the ones n8n recommends for large bodies) so an oversized body is stored and then streamed to the client. Upgrade main and webhook processes to 2.34.0 or later before you enable offload. An older front process will return status 200 and will not delete the stored body.

## One deployment per Redis prefix

If two n8n environments share a Redis instance, set a different `N8N_REDIS_KEY_PREFIX` on each environment, and use that same prefix on every process of that environment. Before n8n 2.3.0 the prefix does not cover pub/sub channels, so those versions need a separate Redis per deployment.

Multi-main high availability (`N8N_MULTI_MAIN_SETUP_ENABLED=true`) is an Enterprise feature. All main processes must be on the same version, in queue mode, behind a load balancer with sticky sessions. For a single team, one main process and a couple of workers is the setup to get right first.

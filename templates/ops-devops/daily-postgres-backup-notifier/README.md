# Daily Postgres backup notifier

Runs every day at 06:15 UTC, reads `pg_stat_archiver`, and posts the result to Slack. The message is marked STALE when there has never been a successful WAL archive, the last archive is older than 24 hours, or the last archive failure is newer than the last success.

This checks continuous archiving, which is the signal that WAL shipping for point-in-time recovery is moving. It does not run `pg_dump`.

## Credentials

- Postgres: host, database, user, password, and SSL mode stored in an n8n Postgres credential. The user needs `pg_monitor` or `pg_read_all_stats` so `pg_stat_archiver` returns real values instead of nulls.
- Slack: bot token with `chat:write`.

## Setup

1. Import `workflow.json`.
2. Attach the Postgres credential on **Read archive stats**. The query uses the database selected in that credential.
3. On **Post daily status**, set the Slack channel. The template uses `#ops-alerts`.
4. Confirm the workflow timezone. The schedule is 06:15 in the workflow timezone; the template leaves that at the instance default, so set the workflow to UTC if you want 06:15 UTC.
5. Execute manually once, then activate.

## Customize

Replace the SQL on **Read archive stats** if backups are recorded in your own table instead of WAL archiving:

```sql
SELECT
  id,
  status,
  started_at,
  finished_at,
  bytes,
  destination,
  error,
  EXTRACT(EPOCH FROM (now() - finished_at)) AS seconds_since_last_archive
FROM backup_runs
ORDER BY started_at DESC
LIMIT 1;
```

If you switch to that query, adjust **Build status message** to treat `status` other than `succeeded` as unhealthy and to read `error` and `destination`.

Change `86400` in **Build status message** to use a different freshness window.

## Test

Run the workflow manually against a database you can read. A server that has never archived a WAL should post a STALE message. A server with `last_archived_time` inside the last day and no later failure should post OK.

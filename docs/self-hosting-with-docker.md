# Self-host n8n with Docker

Docker is the usual way to run n8n yourself. The container listens on port 5678, stores its state in a volume, and reads configuration from environment variables. Use PostgreSQL once more than one person depends on the instance. The default SQLite database is a single file in the n8n volume and is a poor fit for queue mode or for anything you intend to back up with normal database tools.

Pin an image tag. `n8nio/n8n` tracks the current stable release, and `n8nio/n8n:next` is the beta line. In production, pin the version you have actually imported your workflows into, for example `n8nio/n8n:2.48.8`, and upgrade on purpose. Check the current stable version in the n8n Docker installation docs before you copy a tag from a guide.

## Compose file

The file below starts PostgreSQL and one n8n process. Values come from the environment that runs Compose. Do not write passwords or the encryption key into the file you commit.

```yaml
services:
  postgres:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d ${POSTGRES_DB} -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 20

  n8n:
    image: n8nio/n8n:2.48.8
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
      DB_POSTGRESDB_USER: ${POSTGRES_USER}
      DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}
      GENERIC_TIMEZONE: ${GENERIC_TIMEZONE}
      TZ: ${GENERIC_TIMEZONE}
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
      N8N_HOST: ${N8N_HOST}
      N8N_PORT: 5678
      N8N_PROTOCOL: https
      N8N_WEBHOOK_URL: ${N8N_WEBHOOK_URL}
      WEBHOOK_URL: ${N8N_WEBHOOK_URL}
      N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS: "true"
      N8N_PROXY_HOPS: "1"
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres_data:
  n8n_data:
```

`N8N_WEBHOOK_URL` is the public base URL, including the `https://` scheme and a trailing slash. n8n versions before 2.30.0 read `WEBHOOK_URL` for the same purpose, so the file sets both to the same value. `N8N_PROXY_HOPS` should be `1` when a single reverse proxy sits in front of n8n and terminates TLS. Raise it only if you know you have more than one proxy adding forwarded headers.

`GENERIC_TIMEZONE` controls schedule triggers. `TZ` controls the container clock. Set both to the same IANA name, such as `Europe/Berlin`.

Generate `N8N_ENCRYPTION_KEY` with `openssl rand -hex 32` and store it outside the repository. The `n8n_data` volume still matters when you use PostgreSQL: n8n keeps the encryption key file and other instance data there. Mounting a fresh volume without setting `N8N_ENCRYPTION_KEY` creates a new key and makes previously stored credentials unreadable.

## First start

From the directory that contains the Compose file, with the variables exported in your shell or in an untracked env file:

```bash
docker compose up -d
docker compose ps
docker compose logs -f n8n
```

Open `http://localhost:5678` if you published the port on the same machine, or the public URL if a proxy already points at it. Create the owner account on the first page. That account exists only in this database.

Put a reverse proxy in front of the port for any instance that is not on localhost. Terminate TLS at the proxy, forward to `n8n:5678`, and set `N8N_WEBHOOK_URL` to the public origin. Webhook providers and OAuth redirects call that origin, not the Docker network name.

## Upgrade

1. Read the release notes between your pinned tag and the tag you want.
2. Back up the PostgreSQL volume and the `n8n_data` volume, and confirm you still have `N8N_ENCRYPTION_KEY`.
3. Change the image tag, then run `docker compose pull` and `docker compose up -d`.
4. Open the editor, run one inactive test workflow, and check **Executions**.

`docker compose down` stops containers and keeps named volumes. `docker compose down -v` deletes those volumes, including the database and the n8n data directory. Do not add `-v` as a routine restart.

## A single-container trial

For a local trial with SQLite and no PostgreSQL:

```bash
docker volume create n8n_data
docker run --rm --name n8n -p 5678:5678 \
  -e GENERIC_TIMEZONE=Europe/Berlin \
  -e TZ=Europe/Berlin \
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n:2.48.8
```

That is enough to import a template and try it. Move to the Compose file before you store credentials you would be unhappy to lose.

# Personal Firefly III

A self-hosted [Firefly III](https://www.firefly-iii.org/) setup for managing my personal finances, deployed with Docker Compose. It runs the core app, the data importer, and a scheduled job for recurring tasks.

## Tech Stack

* **Core:** [Firefly III](https://www.firefly-iii.org/) (`fireflyiii/core`)
* **Importer:** Firefly III Data Importer (`fireflyiii/data-importer`)
* **Scheduler:** Alpine-based cron container
* **Database:** External database on the `database_net` Docker network
* **Infrastructure:** Docker Compose on a self-hosted VPS, with Apache APISIX on the `apisix_frontend` network

## Quick Start

Instructions to get the environment running.

```bash
# Clone the repository
git clone https://github.com/JosaphatCornelius/personal-firefly-iii.git
cd personal-firefly-iii

# Create the external networks (once)
docker network create apisix_frontend
docker network create database_net

# Create your env files (see below), then start the stack
docker compose up -d
```

*Note: Ensure you have Docker and Docker Compose installed, and a database already running and reachable on `database_net`.*

### Environment files

The compose file expects two env files that are not stored in the repository:

* `.env`: Firefly III core configuration (database connection, `APP_KEY`, `TZ`, and `STATIC_CRON_TOKEN`). See the [official `.env.example`](https://github.com/firefly-iii/firefly-iii/blob/main/.env.example) for the available options.
* `.env.importer`: Data Importer configuration, such as the Firefly III URL and access token.

`STATIC_CRON_TOKEN` must be exactly 32 characters long. The cron container uses it to call Firefly III's cron endpoint every day at 03:00.

## Usage

The stack runs three services:

| Service | Container | Purpose |
| --- | --- | --- |
| `app` | `firefly_iii_core` | The Firefly III web app. |
| `importer` | `firefly_iii_importer` | Imports transactions into Firefly III. |
| `cron` | `firefly_iii_cron` | Triggers recurring transactions and other scheduled jobs daily at 03:00. |

No ports are published by default, since the app is reached through the APISIX gateway. To access it directly, for example on a local machine, uncomment the `ports` section under `app` in `docker-compose.yml`:

```yaml
ports:
  - 80:8080
```

Uploaded attachments are stored in the `firefly_iii_upload` Docker volume.

## Contributing & License

This is a personal setup, but suggestions are welcome. Please open an issue to discuss any changes.

Firefly III is developed by its own contributors under the AGPL-3.0 license. This repository only contains my deployment configuration.

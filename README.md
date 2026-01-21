# Authentik Docker Compose Setup

This repository contains a `docker-compose.yml` file for setting up [Authentik](https://goauthentik.io/), an open-source identity provider, along with its dependencies (PostgreSQL and Redis). This setup is configured to work with Traefik for reverse proxying and SSL termination.

## Services Included

*   **`postgresql`**: A PostgreSQL database instance used by Authentik.
*   **`redis`**: A Redis instance used by Authentik for caching and session management.
*   **`server`**: The main Authentik application server.
*   **`worker`**: The Authentik worker process for background tasks.

## Prerequisites

Before you begin, ensure you have the following installed:

*   [Docker](https://docs.docker.com/get-docker/)
*   [Docker Compose](https://docs.docker.com/compose/install/) (usually comes with Docker Desktop)
*   A Traefik instance configured with a `traefik-public` network and a `cloudflare` certresolver, as indicated by the labels in the `server` service.

## Configuration

This setup relies on environment variables defined in a `.env` file at the root of this directory. An example `.env` file is provided, which you should copy and fill out.

**`.env` variables to configure:**

*   `PG_PASS`: Password for the PostgreSQL user.
*   `PG_USER`: (Optional) PostgreSQL username. Defaults to `authentik`.
*   `PG_DB`: (Optional) PostgreSQL database name. Defaults to `authentik`.
*   `REDIS_PASSWORD`: Password for the Redis instance.
*   `AUTHENTIK_SECRET_KEY`: A strong, random secret key for Authentik.
*   `AUTHENTIK_BOOTSTRAP_PASSWORD`: The initial password for the `akadmin` user in Authentik.
*   `AUTHENTIK_LOG_LEVEL`: (Optional) Log level for Authentik. Defaults to `info`.
*   `AUTHENTIK_EMAIL__HOST`: SMTP host for email sending.
*   `AUTHENTIK_EMAIL__PORT`: SMTP port (default 587).
*   `AUTH_EMAIL_USER`: Username for SMTP authentication.
*   `AUTHENTIK_EMAIL__PASSWORD`: Password for SMTP authentication.
*   `AUTH_EMAIL_FROM`: Sender email address.
*   `AUTHENTIK_HOST`: The name to access authentik in browser

**Example `.env` (create this file):**

```dotenv
PG_PASS=your_postgres_password
REDIS_PASSWORD=your_redis_password
AUTHENTIK_SECRET_KEY=a_very_long_and_random_string_of_characters
AUTHENTIK_BOOTSTRAP_PASSWORD=your_initial_admin_password
AUTHENTIK_EMAIL__HOST=smtp.example.com
AUTH_EMAIL_USER=your_smtp_username
AUTHENTIK_EMAIL__PASSWORD=your_smtp_password
AUTH_EMAIL_FROM=authentik@example.com
AUTHENTIK_HOST=authentik.yourdomain.com
```

## Usage

1.  **Create your `.env` file**: Copy the example above and fill in your desired values.
2.  **Start the services**:
    ```bash
    docker-compose up -d
    ```
    This command will pull the necessary Docker images, create the volumes and networks, and start all services in detached mode.

3.  **Check logs (optional)**:
    ```bash
    docker-compose logs -f
    ```
    This will show the logs from all services, which can be useful for debugging.

4.  **Stop the services**:
    ```bash
    docker-compose down
    ```
    This will stop and remove the containers, but keep the volumes.

5.  **Stop and remove everything (including data volumes)**:
    ```bash
    docker-compose down -v
    ```
    **Use this with caution as it will delete your database and Redis data!**

## Accessing Authentik

The `server` service is exposed via Traefik. Based on the `traefik.http.routers.authentik.rule` label in the `docker-compose.yml`, Authentik should be accessible at:

`https://authentik.yourdomain.com`

Make sure your DNS is configured correctly to point `authentik.yourdomain.com` to your Traefik instance.

## Networks and Volumes

**Networks:**

*   `traefik-public`: An external network used for Traefik integration.
*   `authentik-internal`: An internal overlay network for communication between Authentik services.

**Volumes:**

*   `authentik-database`: Stores PostgreSQL data.
*   `authentik-redis`: Stores Redis data.
*   `authentik-media`: Stores Authentik media files.
*   `authentik-certs`: Stores Authentik certificates.
*   `authentik-templates`: Stores Authentik templates.

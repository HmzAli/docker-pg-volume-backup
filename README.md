# Cold backup for Postgres db volume in docker
This an example of how to perform cold backup of postgres docker volume with pgBackrest.
This backup is purely for development convinence; an alternative way to backup/restore a db without using pg_dump. 

_*NOTE:* this is not ment for production where hot backup, and better setup, is more sensical with pgBackrest. _

For more guideance on how to use pgBackrest, check the official documentation: [https://hub.docker.com/u/pgbackrest](https://pgbackrest.org/user-guide-index.html)

## STEP 1: Create Docker image for pgbackrest
You may also use a pre-built image from Docker.

```Dockerfile
FROM debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y pgbackrest postgresql-client && \
    rm -rf /var/lib/apt/lists/*

# Create default directories
RUN mkdir -p /var/lib/pgbackrest /var/lib/postgresql/data

# Set default environment variable for pgBackRest
ENV PGBACKREST_LOG_PATH=/var/log/pgbackrest

# Entrypoint: run pgBackRest by default
ENTRYPOINT ["pgbackrest"]
```

## STEP 2: Build the pgbackrest image

```
docker build -t my-pgbackrest -f Dockerfile .
```

## STEP 3: Run a temp container and create a stanza
The stanza must be created in offline mode with `--no-online` flag

```
docker run --rm \
  -e PGBACKREST_PG1_PATH=/var/lib/postgresql/data \
  -v [your-pg-db-volume]:/var/lib/postgresql/data \
  -v ~/pgbackups:/var/lib/pgbackrest \
  pgrest \
  stanza-create --stanza=dev --no-online
```

## STEP 4: Stop your database to run cold backup.

```
docker compose down db
```

## STEP 5: Run the backup

```
docker run --rm \
  -e PGBACKREST_PG1_PATH=/var/lib/postgresql/data \
  -v [your-pg-db-volume]:/var/lib/postgresql/data \
  -v ~/pgbackups:/var/lib/pgbackrest \
  pgrest \
  backup --stanza=dev --no-online
```

## STEP 4: Restart the DB

```
docker compose up db
```

## Volume restoration

To restore the volume, you will the following conditions met:
1. The DB is stopped (down)
2. The volume is empty

Once those 2 are met, you can restore the DB with the following command:
```
docker run --rm \
  -e PGBACKREST_PG1_PATH=/var/lib/postgresql/data \
  -v [your-pg-db-volume]:/var/lib/postgresql/data \
  -v ~/pgbackups:/var/lib/pgbackrest \
  pgrest \
  restore --stanza=dev
```





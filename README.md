# atanstack-db

Centralized local database runtime for atanstack services.

## Services

- `corePostgres` (transactional workloads)
- `timescale` (telemetry/time-series workloads)

## Default mappings

- `atanstack-api-auth` -> `auth_db` on core Postgres (`localhost:5433`)
- `atanstack-api-data` -> `data_db` on Timescale (`localhost:5434`)

## Usage

```bash
cp .env.docker.example .env.docker
docker compose -f docker-compose.dev.yml up -d
```

If you already had an older `.env.docker`, re-sync it after infra contract changes:

```bash
cp .env.docker.example .env.docker
```

## Initialize Databases and Tables

This repository starts database containers and creates base databases via `POSTGRES_DB`:

- `auth_db` on `corePostgres`
- `data_db` on `timescale`

Application tables are created by each service, not by `atanstack-db`.

### 1) Start centralized databases

From `atanstack-db`:

```bash
cp .env.docker.example .env.docker
docker compose -f docker-compose.dev.yml up -d
```

### 2) Initialize auth tables (Prisma)

From `atanstack-api-auth`:

```bash
cp .env.docker.example .env.docker
pnpm install
pnpm db:generate
pnpm db:push
```

What this does:
- connects to `DATABASE_URL` (should point to `localhost:5433/auth_db` or `host.docker.internal:5433/auth_db` in Docker),
- creates/updates auth tables in `auth_db`.

### 3) Initialize data tables (sqlx startup schema)

From `atanstack-api-data`:

```bash
cp .env.docker.example .env.docker
cargo run
```

What this does:
- service starts and connects to Timescale (`data_db`),
- runs schema bootstrap (`schema::apply_schema`) at startup,
- creates/updates telemetry/device/alerts tables and Timescale extension usage.

### 4) Verify tables exist

Check auth DB tables:

```bash
docker exec -it atanstack-core-postgres psql -U postgres -d auth_db -c "\\dt"
```

Check data DB tables:

```bash
docker exec -it atanstack-timescale psql -U postgres -d data_db -c "\\dt"
```

Optional Timescale check:

```bash
docker exec -it atanstack-timescale psql -U postgres -d data_db -c "SELECT extname FROM pg_extension WHERE extname='timescaledb';"
```

### 5) Reinitialize from scratch (destructive)

```bash
docker compose -f docker-compose.dev.yml down -v
docker compose -f docker-compose.dev.yml up -d
```

Then rerun:
- `pnpm db:push` in `atanstack-api-auth`
- `cargo run` in `atanstack-api-data`

## Notes

- This repo owns local DB infrastructure for service development.
- During migration windows, service-local DB fallbacks may still exist, but centralized DB is the default path.

## Troubleshooting

- `api-auth` fails with `Can't reach database server at postgres:5432`:
  - your `atanstack-api-auth/.env.docker` is still using old service-local host values.
  - update it to use centralized DB host/port (`host.docker.internal:5433`).
- `corePostgres` exits immediately on Postgres 18 images:
  - keep the volume mount at `/var/lib/postgresql` (not `/var/lib/postgresql/data`) for the official Postgres 18 image behavior.

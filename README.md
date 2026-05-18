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

## Notes

- This repo owns local DB infrastructure for service development.
- During migration windows, service-local DB fallbacks may still exist, but centralized DB is the default path.

# PostgreSQL Docker Deployment

PostgreSQL 17 with performance tuning, deployed via Docker Compose.

## Quick Start

```bash
# 1. Copy and configure environment variables
cp .env.example .env
vim .env  # Set POSTGRES_PASSWORD, POSTGRES_DB

# 2. Start
docker compose up -d

# 3. Check status
docker compose ps
docker compose logs -f postgres
```

## Connection

- **URL**: `postgresql://postgres:your_password@<host>:5432/mydb`
- **Port**: 5432

## Commands

```bash
# Stop
docker compose down

# Stop and remove data (DESTROYS ALL DATA)
docker compose down -v

# Restart
docker compose restart

# View logs
docker compose logs -f postgres

# Connect via psql
docker exec -it postgres psql -U postgres -d mydb

# Backup database
docker exec postgres pg_dump -U postgres mydb > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore database
cat backup.sql | docker exec -i postgres psql -U postgres -d mydb

# Check active connections
docker exec postgres psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"

# Check database size
docker exec postgres psql -U postgres -c "SELECT pg_size_pretty(pg_database_size('mydb'));"

# Check slow queries
docker exec postgres psql -U postgres -c "SELECT pid, now() - pg_stat_activity.query_start AS duration, query FROM pg_stat_activity WHERE state != 'idle' ORDER BY duration DESC LIMIT 5;"

# Check table sizes
docker exec postgres psql -U postgres -d mydb -c "SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) FROM pg_catalog.pg_statio_user_tables ORDER BY pg_total_relation_size(relid) DESC LIMIT 10;"
```

## Performance Tuning

Adjust in `.env` based on available RAM:

| RAM | shared_buffers | effective_cache_size | work_mem |
|-----|---------------|---------------------|----------|
| 4GB | 1GB | 3GB | 16MB |
| 8GB | 2GB | 6GB | 32MB |
| 16GB | 4GB | 12GB | 64MB |
| 32GB | 8GB | 24GB | 128MB |

## Init Scripts

Place `.sql` or `.sh` files in `init/` directory. They run automatically on first startup (when data volume is empty).

## Backup Strategy

```bash
# Automated daily backup (add to crontab)
0 3 * * * docker exec postgres pg_dump -U postgres mydb | gzip > /backups/pg_$(date +\%Y\%m\%d).sql.gz

# Keep last 7 days
0 4 * * * find /backups -name "pg_*.sql.gz" -mtime +7 -delete
```

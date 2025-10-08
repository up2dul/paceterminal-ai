# Docker Setup Guide

This guide explains how to run PACETERMINAL AI using Docker Compose.

## Prerequisites

- Docker Desktop or Docker Engine installed
- Docker Compose v2.0+ installed
- API keys for OpenAI and Tavily

## Quick Start

1. **Copy the environment file:**
   ```bash
   cp .env.docker .env
   ```

2. **Edit `.env` and add your API keys:**
   - Add your `OPENAI_API_KEY`
   - Add your `TAVILY_API_KEY` (optional, for web search features)

3. **Start all services:**
   ```bash
   docker-compose up -d
   ```

4. **Run database migrations:**
   ```bash
   docker-compose exec app uv run prisma db push
   ```

5. **Access the application:**
   - API: http://localhost:8080
   - API Docs: http://localhost:8080/docs
   - Flower (Celery monitoring): http://localhost:5555

## Services

The docker-compose setup includes the following services:

### Core Services
- **app** - FastAPI application (port 8080)
- **worker** - Celery worker for background tasks
- **postgres** - PostgreSQL database (port 5432)
- **redis** - Redis for Celery broker/backend (port 6379)

### Optional Services
- **beat** - Celery Beat for scheduled tasks
- **flower** - Web UI for monitoring Celery tasks (port 5555)

## Common Commands

### Start services
```bash
docker-compose up -d
```

### Stop services
```bash
docker-compose down
```

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f app
docker-compose logs -f worker
```

### Restart a service
```bash
docker-compose restart app
```

### Run commands in a container
```bash
# Run Prisma migrations
docker-compose exec app uv run prisma db push

# Access Python shell
docker-compose exec app uv run python

# Run linting
docker-compose exec app uv run ruff format .
docker-compose exec app uv run ruff check . --fix
```

### Stop and remove all data
```bash
docker-compose down -v
```

## Environment Variables

Key environment variables (set in `.env` file):

| Variable | Description | Required |
|----------|-------------|----------|
| `OPENAI_API_KEY` | OpenAI API key for AI features | Yes |
| `TAVILY_API_KEY` | Tavily API key for web search | No |
| `DEBUG` | Enable debug mode | No (default: False) |
| `ALLOWED_ORIGINS` | CORS allowed origins | No (default: localhost:3000) |
| `RATE_LIMIT_CHAT` | Rate limit for chat endpoint | No (default: 10/minute) |
| `RATE_LIMIT_ANALYSIS` | Rate limit for analysis endpoint | No (default: 3/minute) |

Database and Redis URLs are automatically configured in `docker-compose.yml` for local development.

## Troubleshooting

### Database connection issues
If you see database connection errors, ensure PostgreSQL is healthy:
```bash
docker-compose ps postgres
docker-compose logs postgres
```

### Worker not processing tasks
Check if Redis and the worker are running:
```bash
docker-compose ps redis worker
docker-compose logs worker
```

### Port conflicts
If ports 5432, 6379, 8080, or 5555 are already in use, you can modify them in `docker-compose.yml`.

### Rebuild containers after code changes
```bash
docker-compose up -d --build
```

## Development Workflow

For local development with hot-reload:

1. Start only the dependencies:
   ```bash
   docker-compose up -d postgres redis
   ```

2. Run the app locally:
   ```bash
   uv run uvicorn app.main:app --reload
   ```

3. Run the worker locally:
   ```bash
   uv run celery -A app.celery worker --pool=threads -c 4
   ```

## Production Considerations

For production deployment:

1. **Change default passwords** in `docker-compose.yml`
2. **Use secrets management** for API keys instead of `.env` file
3. **Configure proper CORS origins** in `ALLOWED_ORIGINS`
4. **Set DEBUG=False**
5. **Use a reverse proxy** (nginx/traefik) in front of the app
6. **Enable SSL/TLS** for secure connections
7. **Configure proper backup** for PostgreSQL volumes
8. **Monitor resource usage** and adjust worker count as needed
9. **Consider using managed services** for PostgreSQL and Redis

## Scaling

To scale the worker service:
```bash
docker-compose up -d --scale worker=3
```

This will run 3 worker instances in parallel.

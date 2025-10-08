# PACETERMINAL AI

AI chat API with OpenAI integration.

## Setup

### Local Development

Install uv and copy .env.example to .env and fill the keys.

```bash
uv sync
uv run prisma generate
uv run uvicorn main:app --reload
```

### Docker Setup

For a complete setup with all services (PostgreSQL, Redis, Celery):

```bash
# Copy environment file and add your API keys
cp .env.docker .env

# Start all services
docker-compose up -d

# Run database migrations
docker-compose exec app uv run prisma db push
```

See [DOCKER.md](DOCKER.md) for detailed Docker setup instructions.

## Deploy

```bash
flyctl deploy
```

# ConnectMe 

This repository contains a small microservice suite used by the ConnectMe project. The following services are included (each service has its own README with more details):

# Quick start

1. Create an `.env` (or export env vars) with required secrets, for example:

```
DB_USER=postgres
DB_PASSWORD=postgres
TOKEN_SIGNING_KEY=<base64-key>
BOT_TOKEN=<discord-bot-token>
```

2. Run everything with Docker Compose from the repository root:

```bash
docker-compose up --build
```

## Services & infrastructure

- PostgreSQL (db)
- Redis (redis) - used by `account-linking-service` to store temporary codes
- Kafka (broker) - used to propagate linking confirmation events between the Discord bot and the linking service
- Nginx (nginx) - reverse proxy and CORS config, exposed on host port 8080

# Environment variables

The `docker-compose.yml` contains the full list of environment variables used by services. Important ones:

- `DB_USER`, `DB_PASSWORD` - Postgres credentials
- `TOKEN_SIGNING_KEY` - Base64 signing key used to create JWT tokens in `auth-service`
- `BOT_TOKEN` - Discord bot token for `discord-bot-service`

Endpoints summary

## Auth Service

- POST /api/auth/register - body: `{ username, email, password }` - register new user
- POST /api/auth/login - body: `{ email, password }` - returns `{ accessToken, refreshToken }`
- POST /api/auth/refresh - body: `{ refreshToken }` - returns `{ accessToken }`

User Service

- GET /api/users/{username} - returns public profile `{ uuid, username, createdAt }`

## Account Linking Service

- POST /api/linking?provider=<provider> - authenticated; returns `{ code }` (6-digit), code stored in Redis for ~5 minutes
- GET /api/linking/links - authenticated; returns linked accounts for the user
- Kafka topic `account.linking` - external providers (or the Discord bot) send `{ id, username, code, provider }` to confirm linking

## Discord Bot

- Slash command `/link code:<code>` — the bot sends a Kafka event to `account.linking` with `{ id, username, code, provider: "discord" }` and replies `In process...`



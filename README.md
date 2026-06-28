# ConnectMe — Project Overview

This repository contains a microservice suite used by the ConnectMe project. The following services are included (each service has its own README with more details):

- [auth-service/README.md](auth-service/README.md) — Authentication: register, login, refresh tokens (port 9090)
- [user-service/README.md](user-service/README.md) — User profiles (port 9091)
- [account-linking-service/README.md](account-linking-service/README.md) — Account linking flow, Redis + Kafka (port 9093)
- [discord-bot-service/README.md](discord-bot-service/README.md) — Discord bot that produces linking Kafka events (port 9094)

Quick start

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

Services & infrastructure

- PostgreSQL (db) — User and linking databases
- Redis (redis) — used by `account-linking-service` to store temporary codes
- Kafka (broker) — used to propagate linking confirmation events between the Discord bot and the linking service
- Nginx (nginx) — reverse proxy and CORS config, exposed on host port 8080

Environment variables

The `docker-compose.yml` contains the full list of environment variables used by services. Important ones:

- `DB_USER`, `DB_PASSWORD` — Postgres credentials
- `TOKEN_SIGNING_KEY` — Base64 signing key used to create JWT tokens in `auth-service`
- `BOT_TOKEN` — Discord bot token for `discord-bot-service`

Endpoints summary

Auth Service (`auth-service` — see [auth-service/README.md](auth-service/README.md))

- POST /api/auth/register — body: `{ username, email, password }` — register new user
- POST /api/auth/login — body: `{ email, password }` — returns `{ accessToken, refreshToken }`
- POST /api/auth/refresh — body: `{ refreshToken }` — returns `{ accessToken }`

User Service (`user-service` — see [user-service/README.md](user-service/README.md))

- GET /api/users/{username} — returns public profile `{ uuid, username, createdAt }`

Account Linking Service (`account-linking-service` — see [account-linking-service/README.md](account-linking-service/README.md))

- POST /api/linking?provider=<provider> — authenticated; returns `{ code }` (6-digit), code stored in Redis for ~5 minutes
- GET /api/linking/links — authenticated; returns linked accounts for the user
- Kafka topic `account.linking` — external providers (or the Discord bot) send `{ id, username, code, provider }` to confirm linking

Discord Bot (`discord-bot-service` — see [discord-bot-service/README.md](discord-bot-service/README.md))

- Slash command `/link code:<code>` — the bot sends a Kafka event to `account.linking` with `{ id, username, code, provider: "discord" }` and replies `In process...`

Notes & next steps

- Each service directory contains a `Dockerfile` and `build.gradle.kts` — services build with Gradle and run inside containers in `docker-compose`.
- For development you may build and run services individually using `./gradlew build` and `docker build` in each service folder.
- If you want, I can: run `docker-compose up --build` to smoke-test, or commit these README files to git for you.

# ConnectMe

ConnectMe is a microservice-based platform consisting of authentication, user management, account linking, and Discord integration services.

Each service contains its own README with implementation details.

## Quick Start

Run the entire stack from the repository root:

```bash
docker compose up --build
```

The API gateway will be available at:

```
http://localhost:8080
```

## Architecture

### Infrastructure

| Component  | Purpose                                                                 |
| ---------- | ----------------------------------------------------------------------- |
| PostgreSQL | Separate database for each service (`auth-db`, `user-db`, `linking-db`) |
| Redis      | Temporary storage for account linking codes                             |
| Kafka      | Event broker for account linking                                        |
| Nginx      | Reverse proxy and CORS handling                                         |

### Services

| Service                 | Host Port |
| ----------------------- | --------- |
| auth-service            | 9090      |
| user-service            | 9091      |
| account-linking-service | 9093      |
| discord-bot-service     | 9094      |

## Environment Variables

All required variables are defined in `docker-compose.yml`.

Key variables:

| Variable            | Description                                  |
| ------------------- | -------------------------------------------- |
| `AUTH_DB_*`         | Auth service database credentials            |
| `USER_DB_*`         | User service database credentials            |
| `LINK_DB_*`         | Account linking service database credentials |
| `TOKEN_SIGNING_KEY` | JWT signing key                              |
| `BOT_TOKEN`         | Discord bot token                            |
| `SPRING_MAIL_*`     | Email configuration for auth-service         |

## API Overview

### Auth Service

| Method | Endpoint                             | Description                      |
| ------ | ------------------------------------ | -------------------------------- |
| POST   | `/api/auth/register`                 | Register a new user              |
| POST   | `/api/auth/login`                    | Obtain access and refresh tokens |
| POST   | `/api/auth/refresh`                  | Refresh access token             |
| POST   | `/api/auth/email/verify?code=<code>` | Verify email address             |

### User Service

| Method | Endpoint                | Description             |
| ------ | ----------------------- | ----------------------- |
| GET    | `/api/users/{username}` | Get public user profile |

### Account Linking Service

| Method | Endpoint                           | Description             |
| ------ | ---------------------------------- | ----------------------- |
| POST   | `/api/linking?provider=<provider>` | Generate a linking code |
| GET    | `/api/linking/links`               | List linked accounts    |

Kafka topic:

```
account.linking
```

Event payload:

```json
{
  "id": "...",
  "username": "...",
  "code": "123456",
  "provider": "discord"
}
```

### Discord Bot

Discord command:

```text
/link code:<code>
```

Publishes an event to the `account.linking` Kafka topic.



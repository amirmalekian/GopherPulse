# GopherPulse

> A Go-powered social networking API for sharing posts, following people, and keeping up with your community.

GopherPulse is a backend-focused social networking project built with **Go**. It provides the core building blocks of a social platform, including user registration and authentication, user profiles, following relationships, post management, and personalized feeds.

The project is designed around a modular Go backend, PostgreSQL for persistent data, and optional Redis caching. It also includes API documentation and a lightweight React frontend scaffold.

## Features

- **User accounts** — register users, activate accounts through invitation tokens, and retrieve user profiles.
- **JWT authentication** — issue and validate JSON Web Tokens for protected API endpoints.
- **Secure password storage** — hash passwords with bcrypt.
- **Social graph** — follow and unfollow other users.
- **Post management** — create, retrieve, update, and delete posts with titles, content, and tags.
- **Personalized feed** — fetch posts from followed users with pagination, sorting, date-range, text-search, and tag filters.
- **Post details** — retrieve a post together with its comments.
- **Access control** — enforce post ownership and role-level checks for protected operations.
- **Rate limiting** — protect the API with a configurable fixed-window rate limiter.
- **Optional Redis cache** — cache user data to reduce repeated database reads.
- **Email integration** — send account-activation emails through the Mailtrap provider wired into the API; a SendGrid implementation is also present in the codebase.
- **API documentation** — generate and serve Swagger/OpenAPI documentation.
- **Operational visibility** — structured logging, a health-check endpoint, and runtime/database statistics.

## Tech Stack

| Area | Technology | Purpose |
|---|---|---|
| Language | Go 1.22+ | Backend implementation |
| HTTP routing | [Chi](https://github.com/go-chi/chi) | Lightweight HTTP router and middleware |
| Database | [PostgreSQL](https://www.postgresql.org/) | Persistent relational data |
| Database access | `database/sql`, `lib/pq` | PostgreSQL connectivity |
| Cache | [Redis](https://redis.io/) | Optional user-data caching |
| Redis client | `go-redis` v8 | Redis integration |
| Authentication | JWT (`golang-jwt/jwt`) | Token-based authentication |
| Password hashing | `bcrypt` (`golang.org/x/crypto`) | Password security |
| Validation | `go-playground/validator` | Request payload validation |
| Logging | [Zap](https://github.com/uber-go/zap) | Structured application logging |
| API docs | [Swaggo](https://github.com/swaggo/swag) | Swagger/OpenAPI generation |
| Email | Mailtrap (active provider), SendGrid (available implementation) | Account activation emails |
| Infrastructure | Docker & Docker Compose | Local development services |
| Frontend scaffold | React, TypeScript, Vite, React Router | Optional web client |

## Architecture

The backend follows a modular structure that keeps the API entry point, infrastructure concerns, and data-access code separated.

```text
.
├── cmd/
│   ├── api/                 # HTTP API application and handlers
│   └── migrate/             # Database migrations and seed command
├── internal/
│   ├── auth/                # JWT authentication
│   ├── db/                  # Database connection setup
│   ├── env/                 # Environment-variable helpers
│   ├── mailer/              # Email provider integrations
│   ├── ratelimiter/         # Request rate limiting
│   └── store/
│       └── cache/           # Redis cache implementation
├── docs/                    # Generated Swagger documentation
├── scripts/                 # Development and utility scripts
├── web/                     # React + TypeScript frontend scaffold
├── Dockerfile
├── docker-compose.yml
├── Makefile
└── go.mod
```

## Getting Started

### Prerequisites

- Go **1.22** or later
- Docker and Docker Compose
- [golang-migrate](https://github.com/golang-migrate/migrate) CLI for database migrations
- A Mailtrap API token and a verified sender address (the API initializes the Mailtrap client at startup)
- Node.js and npm (only if you want to run the frontend scaffold)

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/GopherPulse.git
cd GopherPulse
```

### 2. Start PostgreSQL and Redis

The included Compose file provides PostgreSQL, Redis, and Redis Commander for local development.

```bash
docker compose up -d db redis
```

To also start Redis Commander:

```bash
docker compose up -d redis-commander
```

Redis Commander is exposed locally at `http://localhost:8081`.

### 3. Configure environment variables

The Makefile includes a root-level `.envrc` file. Create that file and adjust the values for your environment:

```bash
# Server
export ADDR=":8080"
export EXTERNAL_URL="http://localhost:8080"
export FRONTEND_URL="http://localhost:5173"
export ENV="development"

# PostgreSQL
export DB_ADDR="postgres://admin:adminpassword@localhost:5432/socialnetwork?sslmode=disable"
export DB_MAX_OPEN_CONNS="30"
export DB_MAX_IDLE_CONNS="30"
export DB_MAX_IDLE_TIME="15m"

# Redis (disabled by default)
export REDIS_ADDR="localhost:6379"
export REDIS_PW=""
export REDIS_DB="0"
export REDIS_ENABLED="false"

# Authentication — replace these values outside local development
export AUTH_TOKEN_SECRET="change-this-secret"
export AUTH_BASIC_USER="admin"
export AUTH_BASIC_PASS="change-this-password"

# Rate limiting
export RATE_LIMITER_REQUESTS_COUNT="20"
export RATE_LIMITER_ENABLED="true"

# Email provider configuration
export FROM_EMAIL="noreply@example.com"
export MAILTRAP_API_KEY="your-mailtrap-api-key"
# Only needed if you switch the active mailer implementation in cmd/api/main.go
export SENDGRID_API_KEY=""
```

The database values above match the local Docker Compose defaults. A valid `MAILTRAP_API_KEY` and sender address are required for the API to start because the default application setup initializes Mailtrap during startup. **Do not use the sample credentials or secrets in production.** Keep real credentials out of version control.

If you use `direnv`, allow the `.envrc` file in the project directory. Otherwise, export the variables in your shell before running the API.

### 4. Run database migrations

```bash
make migrate-up
```

To roll back the latest migration, use the migration CLI directly:

```bash
migrate -path=./cmd/migrate/migrations -database="$DB_ADDR" down 1
```

### 5. (Optional) Seed the database

```bash
make seed
```

### 6. Run the API

```bash
go run ./cmd/api
```

The API listens on `http://localhost:8080` by default.

Check the health endpoint:

```bash
curl http://localhost:8080/v1/health
```

### 7. Run the frontend scaffold (optional)

```bash
cd web
npm install
npm run dev
```

The Vite development server uses `http://localhost:5173` by default.

## API Overview

The API is versioned under the `/v1` base path.

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/v1/authentication/user` | Register a user |
| `POST` | `/v1/authentication/token` | Create a JWT access token |
| `PUT` | `/v1/users/activate/{token}` | Activate a user account |
| `GET` | `/v1/users/{id}` | Get a user profile |
| `PUT` | `/v1/users/{userID}/follow` | Follow a user |
| `PUT` | `/v1/users/{userID}/unfollow` | Unfollow a user |
| `GET` | `/v1/users/feed` | Get the authenticated user's feed |
| `POST` | `/v1/posts` | Create a post |
| `GET` | `/v1/posts/{id}` | Get a post and its comments |
| `PATCH` | `/v1/posts/{id}` | Update a post |
| `DELETE` | `/v1/posts/{id}` | Delete a post |
| `GET` | `/v1/health` | Check API health |

Protected endpoints expect a bearer token in the `Authorization` header:

```http
Authorization: Bearer <your-jwt-token>
```

The feed endpoint supports query parameters for time range (`since`, `until`), pagination (`limit`, `offset`), sorting (`sort`), tag filtering (`tags`), and text search (`search`).

## Example Requests

### Register a user

```bash
curl -X POST http://localhost:8080/v1/authentication/user \
  -H "Content-Type: application/json" \
  -d '{
    "username": "gopher",
    "email": "gopher@example.com",
    "password": "secure-password"
  }'
```

### Get an access token

```bash
curl -X POST http://localhost:8080/v1/authentication/token \
  -H "Content-Type: application/json" \
  -d '{
    "email": "gopher@example.com",
    "password": "secure-password"
  }'
```

The registration response includes an **activation token** (also sent by email). Activate the account using the link sent to the configured frontend URL, or call the activation endpoint with that token. Then request a JWT using the login endpoint above. Use the **JWT returned by the login endpoint**, not the activation token, in the `Authorization` header for protected requests.

### Create a post

```bash
curl -X POST http://localhost:8080/v1/posts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-jwt-token>" \
  -d '{
    "title": "Hello, Gophers!",
    "content": "Sharing my first post with the community.",
    "tags": ["go", "backend", "community"]
  }'
```

## API Documentation

Swagger/OpenAPI specifications are generated from the API annotations and are stored in the `docs/` directory.

Install the Swag CLI if needed, then regenerate the documentation after changing API annotations:

```bash
go install github.com/swaggo/swag/cmd/swag@latest
swag init -g ./cmd/api/main.go -d cmd,internal && swag fmt
```

The checked-in Makefile also defines `make gen-docs`; verify its entry-point path in your copy before relying on that target.

When the API is running, the Swagger UI is available at:

```text
http://localhost:8080/swagger/index.html
```

## Development Commands

| Command | Description |
|---|---|
| `make test` | Run all Go tests |
| `make migrate-up` | Apply database migrations |
| `migrate -path=./cmd/migrate/migrations -database="$DB_ADDR" down 1` | Roll back one migration |
| `make seed` | Seed the database |
| `make gen-docs` | Regenerate Swagger documentation |

## Configuration Reference

| Variable | Description |
|---|---|
| `ADDR` | HTTP server address; defaults to `:8080` |
| `EXTERNAL_URL` | Public/base URL used by the application |
| `FRONTEND_URL` | Frontend URL used in account-related links |
| `DB_ADDR` | PostgreSQL connection string |
| `DB_MAX_OPEN_CONNS` | Maximum open database connections |
| `DB_MAX_IDLE_CONNS` | Maximum idle database connections |
| `DB_MAX_IDLE_TIME` | Maximum idle connection lifetime |
| `REDIS_ADDR` | Redis address |
| `REDIS_PW` | Redis password, if configured |
| `REDIS_DB` | Redis database number |
| `REDIS_ENABLED` | Enable or disable Redis caching |
| `AUTH_TOKEN_SECRET` | Secret used to sign JWTs |
| `AUTH_BASIC_USER` / `AUTH_BASIC_PASS` | Credentials used by the Basic Auth middleware |
| `RATE_LIMITER_REQUESTS_COUNT` | Maximum requests allowed in the configured time window |
| `RATE_LIMITER_ENABLED` | Enable or disable rate limiting |
| `FROM_EMAIL` | Sender address for account emails |
| `MAILTRAP_API_KEY` | Mailtrap API key |
| `SENDGRID_API_KEY` | SendGrid API key |

## Tests

Run the Go test suite with:

```bash
make test
```

Or run it directly:

```bash
go test ./...
```

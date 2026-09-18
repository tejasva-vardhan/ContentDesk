# ContentDesk

Student project: write an article once in a browser editor and publish it to **Medium** and **Dev.to** without blocking the HTTP API on the site publish.

This is coursework, not a company product. LinkedIn publishing is not implemented.

## How a publish works

1. The React app (Tiptap editor) saves a draft to PostgreSQL through the Go API.
2. `POST /api/publish/{medium|devto}` builds a job payload (title, HTML, tags, cover image) and puts it on **RabbitMQ**.
3. The API returns `queued` immediately.
4. `cmd/worker` consumes the job:
   - **Medium:** write API first (`internal/browser/medium_api.go`). If that fails, Go-Rod drives a headless Chrome session with stored cookies.
   - **Dev.to:** Go-Rod only (cookie session; no write API path).
5. Circuit breakers in `internal/breaker` stop hammering a platform after repeated failures. Redis is used for cache and rate limits, not as the job queue.

Session cookies / Medium uid-sid-xsrf come from the user’s own accounts (settings UI or `.env`). Do not commit `.env`.

## Stack

- API: Go, Echo (`cmd/api`)
- Worker: Go, RabbitMQ consumer (`cmd/worker`)
- Store: PostgreSQL (drafts, users, credentials)
- Redis: rate limit / cache
- Queue: RabbitMQ
- UI: React, Vite, Tailwind, Tiptap
- Browser fallback: Go-Rod + Chromium
- Optional: Docker Compose (Postgres, Redis, RabbitMQ, MinIO), Chrome/Firefox extension under `extension/`

## Layout

```
cmd/api            HTTP: auth, drafts, settings, enqueue publish
cmd/worker         Consume publish jobs, call site API or browser
internal/browser   Medium API + Rod automation, Dev.to Rod
internal/service   Draft, auth, publish, activity
internal/storage   Postgres, Redis, S3 helpers
frontend           Editor, settings, activity
extension          Browser extension used to capture site session cookies
```

## Run locally

Needs Go 1.24+, Node 20+, Docker (or local Postgres, Redis, and RabbitMQ).

```bash
git clone https://github.com/tejasva-vardhan/ContentDesk.git
cd ContentDesk
cp .env.example .env
docker compose up -d db redis rabbitmq
go run cmd/api/main.go
go run cmd/worker/main.go
cd frontend && npm install && npm run dev
```

- UI: http://localhost:5173
- API: http://localhost:8080

## Author

Tejasva Vardhan Sharma

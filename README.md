# ContentDesk

Write an article once, publish it to **Medium** and **Dev.to**.

The editor is a React + Tiptap app. Drafts live in PostgreSQL. Publish is a background job: the API enqueues on RabbitMQ and returns immediately; a worker talks to the site (Medium REST API first, headless Chrome if that fails; Dev.to via Chrome session cookies). Redis is used for rate limits and cache, not as the queue.

LinkedIn publish was removed from the worker. Do not treat `LI_AT` in `.env.example` as a live feature.

## Why the split

Site publish is slow and flaky (cookies, file pickers, rate limits). Keeping it on the HTTP request would hang the editor. `cmd/api` only authenticates, stores drafts, and enqueues. `cmd/worker` owns retries, circuit breakers, and the browser.

## Features

- Landing, dashboard, profile, settings, full-page editor
- Draft save / load (`PUT/GET /api/drafts/:id`)
- Cover image upload (`POST /api/upload`) — S3-compatible (MinIO / Supabase storage)
- Connect Medium and Dev.to (browser extension under `extension/` harvests session cookies)
- Publish: `POST /api/publish/medium` or `/api/publish/devto` → `{ "status": "queued" }`
- Activity pulls from Medium / Dev.to (`GET /api/medium/activity`, `/api/devto/activity`)
- Prometheus metrics at `/metrics`

## Publish path

```
Editor  --HTTP-->  cmd/api  --RabbitMQ-->  cmd/worker
                                              |
                         Medium: REST API, then Go-Rod fallback
                         Dev.to: Go-Rod + session cookie
```

Circuit breakers (`internal/breaker`) open after repeated platform failures. Credentials come from the settings API or environment (`MEDIUM_UID` / `MEDIUM_SID` / `MEDIUM_XSRF`, `DEVTO_SESSION_TOKEN`). Never commit `.env`.

## Stack

| Layer | Tech |
|---|---|
| API | Go, Echo |
| Worker | Go, RabbitMQ consumer, Go-Rod |
| Database | PostgreSQL |
| Cache / rate limit | Redis |
| Queue | RabbitMQ |
| Object storage | S3-compatible (optional) |
| Frontend | React, Vite, Tailwind, Tiptap |
| Extension | Chrome / Firefox (cookie capture) |

## HTTP API (`:8080`)

| Method | Path | Purpose |
|---|---|---|
| GET | `/health` | liveness |
| POST | `/api/connect/:platform` | connect account |
| POST/GET/DELETE | `/api/settings/credentials` | store / inspect / drop cookies |
| GET/PUT | `/api/profile` | profile |
| GET/PUT | `/api/drafts/:id` | drafts |
| POST | `/api/upload` | cover image |
| POST | `/api/publish/:platform` | enqueue publish (`medium` \| `devto`) |
| GET | `/api/dashboard/activity` | dashboard |
| GET | `/api/medium/activity`, `/api/devto/activity` | live platform activity |

## Layout

```
cmd/api/            Echo server (producer only)
cmd/worker/         job consumer
internal/browser/   Medium API + Rod, Dev.to Rod
internal/service/   drafts, auth, publish, activity
internal/storage/   Postgres, Redis, S3
internal/rabbitmq/  producer / consumer
internal/breaker/   per-platform circuit breaker
frontend/           pages: Landing, Dashboard, Editor, Settings, Profile
extension/          browser extension zips + source
docker-compose.yml  db, redis, rabbitmq, minio, api, worker
```

## Run locally

Go 1.24+, Node 20+, Docker (or local Postgres + Redis + RabbitMQ).

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

Fill `.env` with `DATABASE_URL`, `REDIS_URL`, `RABBITMQ_URL`, and platform cookies from **your** accounts.

## License

MIT.

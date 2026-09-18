# ContentDesk

Student project: write a post once and publish it to Medium and Dev.to.

The Go API stores the draft and enqueues a job. A worker publishes in the background so HTTP requests do not wait on the site. PostgreSQL holds posts. Redis is used for cache and rate limits. RabbitMQ is the job queue. Medium uses the write API first and a headless browser if that fails. Dev.to uses the headless browser.

This is coursework, not a company product.

## Stack

- Backend: Go (Echo), PostgreSQL, Redis, RabbitMQ
- Frontend: React, Vite, Tailwind CSS, Tiptap editor
- Optional: Docker Compose, headless Chrome via Go-Rod

## Layout

- `cmd/api` — HTTP API, auth, enqueue jobs
- `cmd/worker` — publish jobs and browser fallback
- `frontend` — editor

## Run locally

Needs Go 1.24+, Node 20+, Docker (for Postgres, Redis, and RabbitMQ) or local installs of those.

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

Fill `.env` with database, Redis, RabbitMQ, and (if you use the browser fallback) session cookies from your own accounts. Do not commit `.env`.

## Author

Tejasva Vardhan Sharma

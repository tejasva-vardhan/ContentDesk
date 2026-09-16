# ContentDesk

Student project: write a post once and publish it to more than one site (Dev.to, Medium, LinkedIn).

The Go API stores the draft, enqueues jobs, and records status. A React editor is the UI. PostgreSQL holds posts. Redis and Asynq run the workers. If a site has a write API, that is used first. If it does not, a headless browser is the fallback. Canonical URLs are set so the same article is not treated as duplicate SEO content. Duplicate URL checks and retries are built in.

This is coursework/personal software, not a company product.

## Stack

- Backend: Go (Echo), PostgreSQL, Redis, Asynq
- Frontend: React, Vite, Tailwind CSS, Tiptap editor
- Optional: Docker Compose, headless Chrome via Go-Rod

## Layout

- `cmd/api` — HTTP API, auth, enqueue jobs
- `cmd/worker` — job consumers (publish, retries, browser fallback)
- `frontend` — editor and status UI

## Run locally

Needs Go 1.24+, Node 20+, Docker (for Postgres and Redis) or local installs of both.

```bash
git clone https://github.com/tejasva-vardhan/ContentDesk.git
cd ContentDesk
cp .env.example .env
docker compose up --build
```

- UI: http://localhost:5173
- API: http://localhost:8080

Without full Compose:

```bash
docker compose up -d db redis
go run cmd/api/main.go
go run cmd/worker/main.go
cd frontend && npm install && npm run dev
```

Fill `.env` with database, Redis, and (if you use the browser fallback) site session cookies from your own accounts. Do not commit `.env`.

## Author

Tejasva Vardhan Sharma

# ContentDesk frontend

React + Vite + Tailwind UI for ContentDesk. Talks to the Go API (`VITE_API_BASE_URL`, default proxy to `http://localhost:8080`).

## Pages

| Route | File | Role |
|---|---|---|
| `/` | `pages/Landing.jsx` | marketing / entry |
| `/dashboard` | `pages/Dashboard.jsx` | activity after connect |
| `/editor` | `pages/EditorPage.jsx` | Tiptap editor, tags, cover, publish |
| `/settings` | `pages/Settings.jsx` | Medium / Dev.to credentials |
| `/profile` | `pages/Profile.jsx` | display name / bio |

Publish calls `POST /api/publish/{medium|devto}`. Drafts use `/api/drafts/:id`. Cookie connect uses the extension plus `/api/connect/:platform`.

```bash
npm install
npm run dev      # :5173
npm run build
```

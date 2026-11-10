# CyberWatch – Cybersecurity News Portal

Three systems exchanging data over REST:

```
Frontend (React + Bootstrap)  --REST-->  Backend (Node/Express)  --REST-->  GNews
                                              |
                                              +--REST--> Supabase (favorites; optional)
```

## Features
- Search news by keyword
- Category filter: Cybersecurity, AI, Technology, Malware, Data Privacy
- Advanced search filters for topic, event country, and publisher
- Event-country filter searches for a country name in articles (it does not verify where an event happened)
- Cards with headline, source, published date, image
- Details popup + "Open Full Article" link
- Favorites (CREATE / READ / DELETE) stored in Supabase, or a local JSON file if Supabase is not configured
- Search history (stored in the browser)
- Pagination
- Light and dark themes with a saved preference

## Folder structure
```
news-aggregator/
  backend/   server.js, routes/, services/, package.json, .env.example
  frontend/  src/, index.html, package.json
  supabase.sql
```

## Run locally
Requirements: Node.js 18+, an API key from https://gnews.io, and a Supabase project for accounts and favorites.

```bash
# Terminal 1 - backend
cd backend
cp .env.example .env        # then put your GNEWS_API_KEY inside
npm install
npm start                   # http://localhost:5000

# Terminal 2 - frontend
cd frontend
cp .env.example .env       # set the Supabase URL and publishable/anon key
npm install
npm start                   # http://localhost:3000
```

The frontend dev server proxies `/api/*` to the backend, so no extra config is needed.

## REST endpoints (backend)
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/news?q=&category=&page=` | Search/filter news (proxied to GNews) |
| GET | `/api/favorites` | List favorites |
| POST | `/api/favorites` | Save an article `{title, url, source, image, published_at}` |
| DELETE | `/api/favorites/:id` | Remove a favorite |
| GET | `/api/health` | Status check |

## Supabase accounts and favorites
1. Create a project at https://supabase.com
2. Run `supabase.sql` in the SQL Editor. It enables row-level security so each account can only access its own favorites.
3. In Supabase Authentication settings, keep the Email provider enabled, but turn off **Confirm email**. The app provides username/password sign-in by mapping each username to a non-deliverable internal address; no user email address is collected. Set the password policy to require at least 12 characters, uppercase and lowercase letters, numbers, and symbols if those options are available.
4. Put the project URL and publishable/anon key in both `frontend/.env` and `backend/.env`:

   - Frontend: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`
   - Backend: `SUPABASE_URL`, `SUPABASE_ANON_KEY`

   The frontend key is public by design; row-level security protects the data. Never use a Supabase service-role key in the frontend or as `SUPABASE_ANON_KEY`.

Users can sign up or log in with a username from the app's navigation bar. Usernames are 3–32 letters, numbers, underscores, or hyphens and are case-insensitive. Signup requires at least 12 characters, an uppercase letter, a lowercase letter, a number, and a symbol; login continues to accept existing passwords. Because no email is collected, users cannot receive email verification or password-reset messages; provide a separate account recovery process before using this for production. Once logged in, favorites are saved to their account and available across devices. The old local JSON favorites are not account-protected, so this version requires Supabase for favorites.

## Deploy (Vercel)
- Frontend: import `frontend/`, set `VITE_API_URL` to the deployed backend URL.
- Backend: deploy `backend/` as a separate Node project (Render/Railway work well for a plain Express server), set the env vars, and set `CORS_ORIGIN` to the frontend URL.
- Set `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` as frontend build environment variables too.

## Notes
- GNews plan limits apply to request quotas, result counts, and article freshness; check the current limits on your account.
- Responses are cached 5 minutes in the backend to save quota.
- API key lives only in the backend; the browser never sees it.
# SightSec-News

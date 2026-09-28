# PocketPulse Teen Expense Tracker

A full-stack expense and income tracker built with React, Express, and Supabase Postgres. It provides a dashboard, charts, transaction management, category budgets, and supportive spending suggestions based on the account's own records.

## Project layout

- `backend/` — Express REST API using MVC, bcrypt password hashing, JWT authentication, and the Supabase data layer
- `frontend/` — React/Vite app, charts, and browser tests
- `backend/database/schema.sql` — Supabase schema, indexes, triggers, RLS policies, and starter categories

Frontend and backend have separate `package.json` files and can be installed and run independently.

## Requirements

- Node.js 18 or newer and npm
- A Supabase project for persistent storage (optional for a temporary local-only run)

## Local setup

1. In a terminal, go to this project folder.
2. Copy `backend/.env.example` to `backend/.env` and `frontend/.env.example` to `frontend/.env`.
3. Set `JWT_SECRET` in `backend/.env` to a unique random value of at least 32 characters. For example, generate one locally with:

   ```sh
   node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
   ```

   If no `JWT_SECRET` is set during development, the API generates an ephemeral secret when it starts, so existing login sessions will expire when the server restarts. Production refuses to start with a missing or short secret.
4. To use Supabase, follow the database setup below and set `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY` in `backend/.env`. Keep the service-role key private and only in the backend environment.
5. Install and run each app in a separate terminal:

   ```sh
   cd backend
   npm install
   npm run dev
   ```

   ```sh
   cd frontend
   npm install
   npm run dev
   ```

6. Open the Vite URL shown in the frontend terminal (normally `http://localhost:5173`) and create an account. The API health check is `http://localhost:5000/api/health`.

Without Supabase credentials, the API uses an in-memory store for local development and tests. That data is temporary and is cleared when the backend restarts; it is not a substitute for Supabase persistence.

## Supabase setup

1. Create a Supabase project.
2. In the Supabase SQL Editor, run `backend/database/schema.sql`. The script is safe to re-run and does not drop or truncate existing tables.
3. Copy the project URL and service-role key from the Supabase project settings into `backend/.env`.
4. Restart the backend.

This app uses its own bcrypt/JWT authentication and accesses Supabase from the backend with a service-role key. The key is never sent to the browser. Because Supabase service-role access bypasses RLS, backend models scope all user-owned reads and writes by the authenticated `user_id`; the API must remain the only client of this key. The SQL also enables RLS and defines `auth.uid()` policies for use with Supabase Auth, but those policies do not authenticate this app's custom JWTs. Do not expose the service-role key or use it in frontend code.

## Environment variables

### `backend/.env`

| Variable | Purpose |
| --- | --- |
| `PORT` | API port (default `5000`) |
| `NODE_ENV` | Set to `development` locally or `production` for deployment |
| `JWT_SECRET` | Unique signing secret; at least 32 characters in production |
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Private server-side Supabase service-role key |
| `CLIENT_URL` | Frontend origin allowed by CORS (default `http://localhost:5173`) |
| `DEMO_PASSWORD` | Optional secret used only by the development seed command; at least 8 characters |

### `frontend/.env`

| Variable | Purpose |
| --- | --- |
| `VITE_API_URL` | API base URL (default `http://localhost:5000/api`) |
| `VITE_CURRENCY` | Default display currency code, such as `INR`, `USD`, or `EUR`; can also be changed in Account & Settings |

Never commit `.env` files or put backend secrets in `VITE_*` variables.

## Demo data (optional)

Set a private `DEMO_PASSWORD` in `backend/.env`, then run `npm run seed` from `backend/`. The script hashes the supplied password with bcrypt and seeds one demo user and example transactions. It does not print the password and will not duplicate demo data if the demo account already exists. Use that locally supplied password to sign in as `demo@teenfinance.com`.

## Available commands

From `backend/`:

- `npm run dev` — start the API with nodemon
- `npm start` — start the API
- `npm test` — run backend tests
- `npm run seed` — add optional demo data (requires `DEMO_PASSWORD` for first seed)

From `frontend/`:

- `npm run dev` — start the Vite development server
- `npm run build` — create a production build in `frontend/dist/`
- `npm run preview` — preview the production build
- `npm test` — run frontend tests

## API overview

Protected endpoints require `Authorization: Bearer <token>`. Responses use a JSON envelope with `success`, `message`, and `data`; list endpoints may also include `pagination`.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/auth/register` | Create an account; passwords are bcrypt-hashed |
| `POST` | `/api/auth/login` | Authenticate and return a JWT |
| `POST` | `/api/auth/logout` | End the client session |
| `GET` | `/api/auth/me` | Get the authenticated profile |
| `GET`, `POST` | `/api/expenses` | List/create expenses |
| `GET`, `PUT`, `DELETE` | `/api/expenses/:id` | Read/update/delete an owned expense |
| `GET`, `POST` | `/api/income` | List/create income |
| `GET`, `PUT`, `DELETE` | `/api/income/:id` | Read/update/delete owned income |
| `GET` | `/api/dashboard` | Dashboard summary and recent transactions |
| `GET` | `/api/dashboard/summary` | Summary for a selected timeframe |
| `GET` | `/api/analytics/category`, `/api/analytics/monthly`, `/api/analytics/daily` | Chart data |
| `GET` | `/api/suggestions` | Data-based spending suggestions |
| `GET` | `/api/categories` | Expense categories |
| `GET` | `/api/health` | API health check |

Transactions are stored in separate `expenses` and `income` tables, with amount, category/source, date, and optional description. Analytics and suggestions are calculated from records belonging to the authenticated user; the app does not connect to bank accounts or provide professional financial advice.

## Testing

Run `npm test` separately in both `backend/` and `frontend/`. Run `npm run build` in `frontend/` to validate the production bundle.

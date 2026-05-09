# User Service

FastAPI service for user accounts, JWT authentication, XP/level tracking, leaderboards,
and internal `answer-created` events from the survey workflow.

## Port

- HTTP: `8080`

## Environment

| Variable | Default | Purpose |
| --- | --- | --- |
| `APP_NAME` | `User Service` | FastAPI application name |
| `DATABASE_URL` | `sqlite:///./data/user-service.db` | SQLite database URL or path |
| `JWT_SECRET` | `change-me-in-production` | Secret used to sign JWT access tokens |
| `JWT_EXPIRATION_MINUTES` | `60` | Access token TTL |
| `INTERNAL_API_KEY` | `change-me-in-production` | Token for internal service calls |

## HTTP API

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/health` | Healthcheck |
| `POST` | `/register` | Register a user |
| `POST` | `/login` | Issue a JWT |
| `GET` | `/users/{user_id}` | Read user profile using JWT auth |
| `GET` | `/users/{user_id}/stats` | Read XP and level |
| `GET` | `/leaderboard` | Read leaderboard |
| `POST` | `/internal/events/answer-created` | Award XP from an internal answer event |

Internal endpoints require `X-Internal-Token: <INTERNAL_API_KEY>`.

## Local Run

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements-dev.txt
.\.venv\Scripts\python -m uvicorn app.main:app --app-dir src --host 0.0.0.0 --port 8080
```

## Docker

```powershell
docker build -t user-service .
docker run --rm -p 8080:8080 `
  -e DATABASE_URL=sqlite:///./data/user-service.db `
  -e JWT_SECRET=change-me-local-jwt-secret `
  -e INTERNAL_API_KEY=change-me-local-internal-key `
  user-service
```

## Tests

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements-dev.txt
.\.venv\Scripts\python -m unittest discover -s tests -v
```

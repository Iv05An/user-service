# User Service

## 1. Название и назначение сервиса

`user-service` — микросервис пользователей в системе PIUS. Он отвечает за регистрацию, аутентификацию, выдачу JWT, хранение профилей, начисление XP и таблицу лидеров.

Основные функции:

- регистрация пользователя;
- вход в систему и выдача JWT;
- получение профиля пользователя;
- начисление XP по внутренним событиям;
- расчет уровня пользователя;
- таблица лидеров;
- защита внутренних событий от повторной обработки.

## 2. Архитектура и зависимости

Технологии:

- Python 3.11;
- FastAPI и Uvicorn;
- Pydantic;
- SQLite;
- Alembic;
- SQLAlchemy URL parser;
- unittest и FastAPI TestClient;
- JWT HS256 и PBKDF2-SHA256 для паролей.

Взаимодействие с микросервисами:

- принимает от `survey-service` событие `POST /internal/events/answer-created`;
- начисляет пользователю XP за сохраненный ответ;
- внутренний эндпоинт защищен `INTERNAL_API_KEY`;
- повторные события обрабатываются идемпотентно через `Idempotency-Key` и бизнес-ключ ответа.

Внешние сервисы не используются. Redis, Kafka, S3 и внешняя PostgreSQL в текущей версии не требуются.

## 3. Способы запуска сервиса

### Через Docker

```powershell
docker build -t user-service .
docker run --rm -p 8080:8080 `
  -e DATABASE_URL=sqlite:///./data/user-service.db `
  -e JWT_SECRET=change-me-local-jwt-secret `
  -e INTERNAL_API_KEY=change-me-local-internal-key `
  user-service
```

### Без Docker

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements-dev.txt
.\.venv\Scripts\python -m uvicorn app.main:app --app-dir src --host 0.0.0.0 --port 8080
```

### Переменные окружения

| Переменная | По умолчанию | Назначение |
| --- | --- | --- |
| `APP_NAME` | `User Service` | имя FastAPI-приложения |
| `DATABASE_URL` | `sqlite:///./data/user-service.db` | SQLite база данных |
| `JWT_SECRET` | `change-me-in-production` | секрет для JWT |
| `JWT_EXPIRATION_MINUTES` | `60` | время жизни JWT |
| `INTERNAL_API_KEY` | `change-me-in-production` | ключ внутренних API-вызовов |

Для запуска всей системы используется общий репозиторий `bozvan/PIUS` и команда `docker compose up --build -d`.

## 4. API документация

После запуска Swagger доступен по адресу:

- `http://localhost:8080/docs`
- `http://localhost:8080/openapi.json`

Основные эндпоинты:

| Метод | Путь | Описание |
| --- | --- | --- |
| `GET` | `/health` | проверка работоспособности |
| `POST` | `/register` | регистрация пользователя |
| `POST` | `/login` | получение JWT |
| `GET` | `/users/{user_id}` | профиль пользователя по JWT |
| `GET` | `/users/{user_id}/stats` | XP и уровень |
| `GET` | `/leaderboard` | таблица лидеров |
| `POST` | `/internal/events/answer-created` | начисление XP по внутреннему событию |

Внутренний эндпоинт требует заголовок `X-Internal-Token: <INTERNAL_API_KEY>`.

## 5. Как тестировать

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements-dev.txt
.\.venv\Scripts\python -m unittest discover -s tests -v
```

## 6. Контакты и поддержка

Автор сервиса: Андреев И.

Поддержка:

- GitHub Issues: https://github.com/Iv05An/user-service/issues
- GitHub: https://github.com/Iv05An

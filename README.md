# AFIN — моделирование и анализ бизнес-процессов с ML и LLM

Платформа для моделирования бизнес-процессов в BPMN-редакторе, симуляции их выполнения, прогнозирования задержек моделью LightGBM и получения текстовых объяснений от LLM.

🏆 Финалист акселератора МТУСИ.

**FastAPI** • **SQLAlchemy** • **Pydantic** • **JWT** • **LightGBM** • **Hugging Face Transformers** • **React + TypeScript** • **Docker Compose**

## Архитектура

```
Браузер ─► frontend (React, nginx :3000) ─► /api ─► backend (FastAPI :8000) ─► SQLite
                                                        │
                                                        └─ httpx ─► ai-service (FastAPI)
                                                                     LightGBM + Qwen2 1.5B (LoRA)
```

Три контейнера в одном `docker-compose.yml`:

| Сервис | Каталог | Что делает |
| --- | --- | --- |
| `backend` | `afin-backend/` | API для фронтенда. Один процесс FastAPI (gateway), в который подключены доменные модули: `auth`, `models`, `simulation`, `analytics` |
| `ai-service` | `ai-service/` | Отдельный FastAPI-сервис: прогноз задержки (LightGBM) и генерация объяснений (Qwen2 1.5B с LoRA-адаптером через Hugging Face Transformers) |
| `frontend` | `afin-frontend/` | SPA на React + TypeScript, BPMN-редактор на React Flow, графики, AI-чат |

### Backend (`afin-backend/`)

```
shared/                 настройки (pydantic-settings), engine и сессия SQLAlchemy
services/
  gateway/main.py       точка входа, подключение роутеров, CORS, /api/dashboard
  auth/                 регистрация, логин, JWT, профиль, смена пароля (bcrypt)
  models/               CRUD моделей процессов, копирование, импорт BPMN/JSON, экспорт BPMN
  simulation/           движок симуляции по графу процесса с учётом сотрудников и отделов
  analytics/            прогноз задержек (LightGBM), обработка CSV/JSON, клиент LLM-сервиса
```

- **Симуляция.** Движок проходит граф BPMN-модели, распределяет задачи по сотрудникам и отделам из контекста компании, считает время, стоимость, загрузку отделов, тепловую карту рисков и аномалии. Для оценки риска на шагах обращается к ML-прогнозу.
- **Аналитика.** Прогноз задержки для одной задачи и пакетно из загруженного CSV/JSON-файла.
- **Интеграция LLM.** `services/analytics/llm_client.py` — асинхронный клиент на `httpx` к `ai-service` с повторными попытками и экспоненциальной задержкой (сервис с моделью долго стартует). Через него работают эндпоинты `/api/analytics/llm/explain-file` (объяснение результатов прогноза по файлу) и `/api/analytics/llm/chat` (вопросы пользователя с контекстом прогноза).

### Основные эндпоинты

| Группа | Эндпоинты |
| --- | --- |
| Auth | `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`, `PUT /api/users/profile`, `PUT /api/users/password` |
| Модели процессов | `GET/POST /api/process-models`, `GET/PUT/DELETE /api/process-models/{id}`, `POST /api/process-models/{id}/copy`, `POST /api/process-models/import/bpmn`, `POST /api/process-models/import/json`, `GET /api/process-models/{id}/export/bpmn` |
| Симуляции | `GET/POST /api/simulations`, `GET /api/simulations/{id}` |
| Аналитика | `GET /api/analytics/summary`, `POST /api/analytics/predict-delay`, `POST /api/analytics/process-file`, `POST /api/analytics/llm/explain-file`, `POST /api/analytics/llm/chat` |
| Дашборд | `GET /api/dashboard` |
| AI-сервис | `POST /predict_delay`, `POST /explain_delay` (доступен только внутри сети compose) |

Полная схема — в Swagger: <http://localhost:8000/docs>.

## Запуск

```bash
docker compose up --build
```

- Фронтенд: <http://localhost:3000>
- API и Swagger: <http://localhost:8000/docs>

Первый запуск `ai-service` долгий: модель скачивается с Hugging Face Hub. Имя модели задаётся переменной `HF_MODEL_NAME` в `docker-compose.yml`.

База SQLite создаётся автоматически в `afin-backend/storage/` (каталог монтируется в контейнер). Тестового пользователя можно создать скриптом:

```bash
docker exec -it afin-backend python create_test_user.py
```

### Без Docker

```bash
cd afin-backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export DATABASE_URL=sqlite:///./storage/afin.db JWT_SECRET=change-me LLM_SERVICE_URL=http://localhost:8001
uvicorn services.gateway.main:app --reload --port 8000
```

AI-сервис (`ai-service/`) запускается аналогично: `pip install -r requirements.txt` и `uvicorn FastAPI_server:app --port 8001`.

Фронтенд: `cd afin-frontend/frontend && npm install && npm run dev`.

## Конфигурация backend

| Переменная | По умолчанию | Описание |
| --- | --- | --- |
| `DATABASE_URL` | — | Строка подключения SQLAlchemy |
| `JWT_SECRET` | dev-значение | Секрет подписи JWT, в проде обязательно переопределить |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `15` | Время жизни access-токена |
| `LLM_SERVICE_URL` | `http://ai-assistant:8000` | Адрес AI-сервиса |

## Команда

- **Backend, интеграция ML и LLM в платформу, frontend** — Алексей Епифанов ([@Dr0wsyC0der](https://github.com/Dr0wsyC0der))
- **AI-сервис** (обучение LightGBM, дообучение Qwen2 с LoRA) — участники команды проекта

Отчёты по ML-части лежат в `ai-service/docs/`.

## Что можно улучшить

- Перейти с SQLite на PostgreSQL и добавить миграции Alembic
- Добавить refresh-токены и тесты на pytest
- Вынести синхронный вызов ML-прогноза из движка симуляции в асинхронный клиент

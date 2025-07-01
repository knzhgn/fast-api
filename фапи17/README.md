# FastAPI: Мониторинг и Структурированное Логирование

REST API с внедрённой системой структурированного логирования и мониторинга производительности, построенное на **FastAPI**, с использованием **Prometheus** и контейнеризированное с помощью **Docker**. Проект демонстрирует настройку JSON-логов, эндпоинта проверки состояния и сбор метрик для мониторинга.

## Описание

Проект выполнен в рамках задания №17 по внедрению структурированного логирования и мониторинга в FastAPI-приложение. Основной функционал включает:

- Настройку модуля **logging** для вывода структурированных JSON-логов.
- Middleware для логирования HTTP-запросов (метод, путь, статус, время выполнения).
- Эндпоинт `/health` для проверки состояния приложения и базы данных.
- Интеграцию **Prometheus** для сбора метрик через эндпоинт `/metrics`.
- Контейнеризацию приложения, **PostgreSQL** и **Prometheus** с помощью **Docker Compose**.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API.
- **logging (Python)** — стандартный модуль для структурированного логирования.
- **prometheus-fastapi-instrumentator** — библиотека для интеграции Prometheus с FastAPI.
- **Prometheus** — система мониторинга и сбора метрик.
- **PostgreSQL** — реляционная база данных для хранения данных.
- **Uvicorn** — ASGI-сервер для запуска FastAPI-приложения.
- **Docker** / **Docker Compose** — контейнеризация и оркестрация.

## 🚀 Установка и запуск

### 1. Запуск через Docker

Убедитесь, что **Docker** и **Docker Desktop** установлены и запущены.

1. Создайте файл `.env` в корне проекта (пример в разделе "Особенности") и добавьте его в `.gitignore`.
2. Соберите и запустите контейнеры:
   ```bash
   docker compose up --build
   ```

3. После успешного запуска сервисы будут доступны:  
   - **API**: [http://localhost:8000](http://localhost:8000)  
   - **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
   - **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)  
   - **Health Check**: [http://localhost:8000/health](http://localhost:8000/health)  
   - **Metrics**: [http://localhost:8000/metrics](http://localhost:8000/metrics)  
   - **Prometheus UI**: [http://localhost:9090](http://localhost:9090)

4. Проверьте логи контейнеров (`docker-compose up`) для подтверждения корректного запуска.

### 2. Локальный запуск (без Docker)

1. Убедитесь, что **PostgreSQL** установлен и запущен локально.
2. Создайте файл `.env` в корне проекта.
3. Активируйте виртуальное окружение и установите зависимости:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Для Linux/macOS
   .venv\Scripts\activate     # Для Windows
   pip install -r requirements.txt
   ```

4. Запустите сервер:
   ```bash
   uvicorn main:app --reload
   ```

## 🧪 Тестирование

### 1. Настройка логирования

1. Настройте структурированное логирование в `main.py`:
   ```python
   import logging
   import json
   from fastapi import FastAPI, Request
   from logging.handlers import RotatingFileHandler

   app = FastAPI()

   logger = logging.getLogger("app")
   logger.setLevel(logging.INFO)
   handler = RotatingFileHandler("app.log", maxBytes=10*1024*1024, backupCount=5)
   formatter = logging.Formatter(
       '{"time": "%(asctime)s", "level": "%(levelname)s", "message": "%(message)s"}'
   )
   handler.setFormatter(formatter)
   logger.addHandler(handler)

   @app.middleware("http")
   async def log_requests(request: Request, call_next):
       start_time = time.time()
       response = await call_next(request)
       duration = time.time() - start_time
       logger.info(
           f"Request: {request.method} {request.url.path} - Status: {response.status_code} - Duration: {duration:.3f}s"
       )
       return response
   ```

2. Добавьте логи в ключевые точки, например, при регистрации:
   ```python
   @app.post("/register")
   async def register_user(user: UserCreate):
       logger.info(f"Registering user: {user.username}")
       # Логика регистрации
       return {"id": 1, "username": user.username}
   ```

### 2. Тестирование логирования

1. Запустите приложение через Docker:
   ```bash
   docker compose up
   ```

2. Выполните запрос через **Postman**:
   ```
   GET http://localhost:8000/notes
   Authorization: Bearer <JWT-токен>
   ```

3. Проверьте логи контейнера `web`:
   ```bash
   docker logs <web_container_name>
   ```
   Ожидаемый формат логов:
   ```json
   {"time": "2025-06-12T01:33:00+05:00", "level": "INFO", "message": "Request: GET /notes - Status: 200 - Duration: 0.123s"}
   ```

### 3. Тестирование Health Check

1. Отправьте запрос к эндпоинту `/health`:
   ```
   GET http://localhost:8000/health
   ```
   Ожидаемый ответ (при успехе):
   ```json
   {"status": "healthy", "database": "connected"}
   ```
   При ошибке подключения к БД:
   ```json
   {"status": "unhealthy", "database": "disconnected"}
   ```

### 4. Тестирование Prometheus

1. Отправьте несколько запросов к API (например, `/notes`, `/health`).
2. Проверьте эндпоинт `/metrics`:
   ```
   GET http://localhost:8000/metrics
   ```
   Ожидаемый вывод (пример):
   ```
   # HELP http_requests_total Total number of HTTP requests
   # TYPE http_requests_total counter
   http_requests_total{method="GET",endpoint="/notes",status="200"} 5.0
   ```

3. Откройте веб-интерфейс Prometheus:
   ```
   http://localhost:9090
   ```
   Введите запрос, например, `http_requests_total`, и убедитесь, что метрики отображаются.

### Возможные ошибки и их решение

- **Логи не выводятся в JSON**: Проверьте форматтер в конфигурации `logging` и наличие `logging.handlers.RotatingFileHandler`.
- **Health check возвращает 503**: Убедитесь, что `DATABASE_URL` в `.env` корректен и PostgreSQL доступен.
- **Prometheus не собирает метрики**: Проверьте `prometheus.yml` и убедитесь, что `targets` указывает на `web:8000`.
- **Prometheus UI недоступен**: Убедитесь, что порт 9090 не занят, и сервис `prometheus` запущен в `docker-compose.yml`.

## 🧫 Примеры использования (через Postman)

### 📒 Получение заметок `GET /notes`
- **Метод**: GET
- **URL**: `/notes?skip=0&limit=10`
- **Заголовки**:
  ```
  Authorization: Bearer <JWT-токен>
  ```
- **Ответ** (JSON):
  ```json
  [
    {
      "id": 1,
      "text": "Тестовая заметка",
      "owner_id": 1,
      "created_at": "2025-06-12T01:33:00+05:00"
    }
  ]
  ```

### 📌 Проверка состояния `GET /health`
- **Метод**: GET
- **URL**: `/health`
- **Ответ** (JSON):
  ```json
  {"status": "healthy", "database": "connected"}
  ```

## 🧾 Особенности

- **Структурированные логи**: Логи выводятся в JSON-формате, удобном для парсинга (например, с ELK Stack).
- **Middleware логирования**: Автоматически фиксирует метод, путь, статус и время выполнения запросов.
- **Health Check**: Эндпоинт `/health` проверяет подключение к базе данных.
- **Prometheus**: Собирает метрики (количество запросов, задержки) через `/metrics`.
- **Контейнеризация**: Приложение, PostgreSQL и Prometheus упакованы в Docker-контейнеры.
- **Рекомендация**: Для визуализации метрик подключите **Grafana** к Prometheus (порт 3000).

## 📦 Контейнеры

Конфигурация **Docker Compose** включает три сервиса:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    env_file:
      - .env
    environment:
      - DATABASE_URL=postgresql+asyncpg://username:password@db:5432/notesdb
    depends_on:
      - db

  db:
    image: postgres
    environment:
      POST
# FastAPI: Ограничение Частоты Запросов с Redis

REST API с реализованным механизмом ограничения частоты запросов (rate limiting), построенное на **FastAPI** с использованием **Redis** для хранения счетчиков и контейнеризированное с помощью **Docker**. Проект демонстрирует настройку middleware для ограничения запросов на основе фиксированного окна и тестирование функционала.

## Описание

Проект выполнен в рамках задания №18 по внедрению механизма ограничения частоты запросов в FastAPI-приложение. Основной функционал включает:

- Реализацию **RateLimiterMiddleware** для ограничения количества запросов с использованием стратегии "Fixed Window Counter".
- Использование **Redis** для хранения счетчиков запросов с TTL.
- Идентификацию клиентов по IP-адресу или user_id из JWT-токена.
- Настройку лимитов через конфигурацию приложения (Pydantic Settings).
- Тестирование ограничений через **Postman** с проверкой ошибок 429 Too Many Requests.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API.
- **Redis** — высокоскоростное хранилище для счетчиков запросов.
- **aioredis** — асинхронная библиотека для работы с Redis.
- **Uvicorn** — ASGI-сервер для запуска FastAPI-приложения.
- **Docker** / **Docker Compose** — контейнеризация и оркестрация.
- **Pydantic Settings** — для управления конфигурацией лимитов.
- **Postman** — для тестирования API и проверки ограничений.

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

4. Проверьте логи контейнеров (`docker-compose up`) для подтверждения корректного запуска.

### 2. Локальный запуск (без Docker)

1. Убедитесь, что **Redis** установлен и запущен локально.
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

### 1. Настройка Rate Limiter

1. Настройте middleware в `main.py`:
   ```python
   from fastapi import FastAPI, Request, HTTPException
   from aioredis import from_url
   from config import settings

   app = FastAPI()

   async def get_redis():
       redis = await from_url(settings.redis_url)
       try:
           yield redis
       finally:
           await redis.close()

   @app.middleware("http")
   async def rate_limiter_middleware(request: Request, call_next):
       redis = await anext(get_redis())
       client_id = request.client.host  # Или user_id из JWT
       key = f"ratelimit:{client_id}"
       count = await redis.incr(key)
       if count == 1:
           await redis.expire(key, settings.rate_limit_window)
       if count > settings.rate_limit_max_requests:
           raise HTTPException(status_code=429, detail="Too Many Requests")
       return await call_next(request)
   ```

2. Настройте лимиты в `config.py`:
   ```python
   from pydantic_settings import BaseSettings

   class Settings(BaseSettings):
       database_url: str
       redis_url: str
       secret_key: str
       rate_limit_max_requests: int = 100
       rate_limit_window: int = 60  # В секундах

       class Config:
           env_file = ".env"

   settings = Settings()
   ```

### 2. Тестирование ограничений

1. Запустите приложение через Docker:
   ```bash
   docker compose up
   ```

2. Используйте **Postman** для тестирования:
   - Создайте коллекцию с запросом `GET /notes`:
     ```
     GET http://localhost:8000/notes
     Authorization: Bearer <JWT-токен>
     ```
   - Настройте "Collection Runner" для отправки 101 запроса подряд.
   - Первые 100 запросов должны вернуть статус 200 OK.
   - 101-й запрос должен вернуть статус 429 Too Many Requests:
     ```json
     {"detail": "Too Many Requests"}
     ```

3. Подождите 60 секунд (или значение `rate_limit_window`) и повторите запрос:
   - Запрос должен снова вернуть 200 OK, так как окно ограничения истекло.

4. Проверьте счетчики в Redis:
   ```bash
   docker exec -it <redis_container_name> redis-cli
   KEYS ratelimit:*
   GET ratelimit:<client_ip>
   TTL ratelimit:<client_ip>
   ```

### Пример скрипта для Postman

В Postman откройте консоль JavaScript (Tests) и добавьте:
```javascript
for (let i = 0; i < 101; i++) {
    pm.sendRequest({
        url: 'http://localhost:8000/notes',
        method: 'GET',
        header: {
            'Authorization': 'Bearer <your_jwt_token>'
        }
    }, (err, res) => {
        console.log(`Request ${i + 1}: Status ${res.status}`);
    });
}
```

### Возможные ошибки и их решение

- **Redis connection refused**: Убедитесь, что Redis запущен и `REDIS_URL` в `.env` корректен.
- **429 не возвращается**: Проверьте, что `rate_limit_max_requests` и `rate_limit_window` заданы в `config.py`.
- **Middleware не применяется**: Убедитесь, что middleware добавлен в `app` через `@app.middleware("http")`.
- **Высокая нагрузка на Redis**: Используйте Redis Cluster для масштабирования при большом количестве клиентов.

## 🧫 Примеры использования (через Postman)

### 📒 Получение заметок `GET /notes`
- **Метод**: GET
- **URL**: `/notes?skip=0&limit=10`
- **Заголовки**:
  ```
  Authorization: Bearer <JWT-токен>
  ```
- **Ответ** (JSON, до превышения лимита):
  ```json
  [
    {
      "id": 1,
      "text": "Тестовая заметка",
      "owner_id": 1,
      "created_at": "2025-06-12T01:42:00+05:00"
    }
  ]
  ```
- **Ответ** (JSON, при превышении лимита):
  ```json
  {"detail": "Too Many Requests"}
  ```

### 📌 Создание заметки `POST /notes`
- **Метод**: POST
- **URL**: `/notes`
- **Заголовки**:
  ```
  Authorization: Bearer <JWT-токен>
  Content-Type: application/json
  ```
- **Тело запроса** (JSON):
  ```json
  {
    "text": "Новая заметка"
  }
  ```
- **Ответ** (JSON):
  ```json
  {
    "id": 1,
    "text": "Новая заметка",
    "owner_id": 1,
    "created_at": "2025-06-12T01:42:00+05:00"
  }
  ```

## 🧾 Особенности

- **Rate Limiting**: Использует стратегию "Fixed Window Counter" с лимитом 100 запросов в минуту.
- **Redis**: Хранит счетчики запросов с TTL, равным окну ограничения.
- **Middleware**: Применяется ко всем эндпоинтам, идентифицирует клиентов по IP или user_id.
- **Конфигурация**: Лимиты задаются через **Pydantic Settings** в `config.py`.
- **Контейнеризация**: Приложение и Redis упакованы в Docker-контейнеры.
- **Оптимизация**: Для высокой нагрузки рекомендуется использовать Redis Cluster или увеличить лимиты.

## 📦 Контейнеры

Конфигурация **Docker Compose** включает два сервиса:

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
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis

  redis:
    image: redis:latest
    ports:
      - "6379:6379"
```

## 🗂️ Документация

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

## Контакты

Если вам нужна помощь с проектом или есть вопросы, пишите в **Microsoft Teams**.
# FastAPI: Кеширование Данных с Redis

REST API с реализацией кеширования данных для оптимизации производительности часто запрашиваемых эндпоинтов, построенное на **FastAPI** с использованием **Redis** и контейнеризированное с помощью **Docker**. Проект демонстрирует внедрение кеширования для эндпоинта получения заметок и инвалидацию кеша при изменении данных.

## Описание

Проект выполнен в рамках задания №13 по внедрению механизма кеширования с использованием **Redis** для оптимизации работы REST API. Основной функционал включает:

- Настройку подключения к **Redis** с использованием **aioredis** для асинхронной работы.
- Кеширование данных эндпоинта `GET /notes` с учетом параметров запроса.
- Инвалидацию кеша при создании, обновлении или удалении заметок.
- Тестирование производительности и корректности кеширования через **Postman**.
- Контейнеризацию приложения и **Redis** для упрощённого развёртывания.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API.
- **Redis** — высокоскоростное хранилище данных в памяти для кеширования.
- **aioredis** — асинхронная библиотека для работы с Redis.
- **Uvicorn** — ASGI-сервер для запуска FastAPI-приложения.
- **Docker** / **Docker Compose** — контейнеризация и оркестрация.
- **PostgreSQL** — реляционная база данных для хранения заметок.
- **Postman** — для тестирования API и проверки кеширования.

## 🚀 Установка и запуск

### 1. Запуск через Docker

Убедитесь, что **Docker** и **Docker Desktop** установлены и запущены.

1. Соберите и запустите контейнеры:
   ```bash
   docker compose up --build
   ```

2. После успешного запуска сервисы будут доступны:  
   - **API**: [http://localhost:8000](http://localhost:8000)  
   - **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
   - **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)  
   - **Redis**: localhost:6379 (доступен внутри контейнеров)

3. Проверьте логи контейнеров (`docker-compose up`), чтобы убедиться, что приложение, Redis и база данных запустились без ошибок.

### 2. Локальный запуск (без Docker)

1. Убедитесь, что **Redis** и **PostgreSQL** установлены и запущены локально.
2. Активируйте виртуальное окружение и установите зависимости:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Для Linux/macOS
   .venv\Scripts\activate     # Для Windows
   pip install -r requirements.txt
   ```

3. Настройте подключение к PostgreSQL и Redis в файле `config.py`:
   ```python
   DATABASE_URL = "postgresql+asyncpg://username:password@localhost:5432/notesdb"
   REDIS_URL = "redis://localhost:6379/0"
   ```

4. Запустите сервер:
   ```bash
   uvicorn main:app --reload
   ```

## 🧪 Тестирование

### 1. Установка зависимостей

Если зависимости еще не установлены:
```bash
pip install -r requirements.txt
```

### 2. Тестирование кеширования

1. **Очистка кеша**:
   Подключитесь к Redis и выполните:
   ```bash
   redis-cli FLUSHALL
   ```

2. **Первый GET-запрос**:
   - Используйте **Postman** для выполнения запроса:
     ```
     GET http://localhost:8000/notes?skip=0&limit=10
     Authorization: Bearer <JWT-токен>
     ```
   - Проверьте логи сервера. Ожидаемый вывод:
     ```
     INFO: Fetching notes from PostgreSQL for user_id=1, skip=0, limit=10
     ```
   - Запрос должен занять больше времени, так как данные берутся из БД.

3. **Повторный GET-запрос**:
   - Выполните тот же запрос повторно.
   - Проверьте логи. Ожидаемый вывод:
     ```
     INFO: Fetching notes from Redis for key=notes:1:0:10:
     ```
   - Запрос должен выполниться быстрее, так как данные взяты из кеша.

4. **Изменение данных**:
   - Выполните POST-запрос для создания заметки:
     ```
     POST http://localhost:8000/notes
     Authorization: Bearer <JWT-токен>
     Content-Type: application/json
     ```
     ```json
     {
       "text": "Новая заметка"
     }
     ```
   - Проверьте логи. Ожидаемый вывод:
     ```
     INFO: Invalidating cache for user_id=1
     ```

5. **Проверка инвалидации**:
   - Снова выполните GET-запрос `GET /notes?skip=0&limit=10`.
   - Логи должны показать обращение к БД:
     ```
     INFO: Fetching notes from PostgreSQL for user_id=1, skip=0, limit=10
     ```

### Возможные ошибки и их решение
- **Redis connection refused**: Убедитесь, что Redis запущен (`redis-server` для локального запуска или контейнер `redis` работает).
- **Кеш не инвалидируется**: Проверьте, что ключ формируется одинаково в GET и POST/DELETE эндпоинтах.
- **Медленный ответ**: Убедитесь, что TTL кеша (например, 600 секунд) не истёк, и данные всё ещё в Redis.

## 🧫 Примеры использования (через Postman)

### 📒 Получение заметок `GET /notes`
- **Метод**: GET
- **URL**: `/notes?skip=0&limit=10`
- **Заголовки**:
  ```
  Authorization: Bearer <JWT-токен>
  ```
- **Ответ** (JSON, из кеша или БД):
  ```json
  [
    {
      "id": 1,
      "text": "Новая заметка",
      "owner_id": 1,
      "created_at": "2025-06-11T22:30:00+05:00"
    }
  ]
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
    "created_at": "2025-06-11T22:30:00+05:00"
  }
  ```

## 🧾 Особенности

- **Кеширование**: Данные эндпоинта `GET /notes` кешируются в **Redis** с TTL 10 минут.
- **Инвалидация кеша**: Автоматическое удаление кеша при изменении данных (POST, PUT, DELETE).
- **Асинхронность**: Используется **aioredis** для асинхронного взаимодействия с Redis.
- **Безопасность**: Эндпоинты защищены **JWT-токенами**, кеш привязан к `user_id`.
- **Контейнеризация**: Приложение, **Redis** и **PostgreSQL** упакованы в контейнеры.
- **Логирование**: Добавлены логи для отслеживания обращения к кешу или БД.

## 📦 Контейнеры

Конфигурация **Docker Compose** включает три сервиса:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://username:password@db:5432/notesdb
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  db:
    image: postgres
    environment:
      POSTGRES_DB: notesdb
      POSTGRES_USER: username
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:latest
    ports:
      - "6379:6379

volumes:
  postgres_data:

```

## 🗂️ Документация

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

## Контакты

Если вам нужна помощь с проектом или есть вопросы, пишите в **Microsoft Teams**.
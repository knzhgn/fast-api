# FastAPI: Управление Конфигурацией с Pydantic Settings

REST API с реорганизованным управлением настройками, использующее **Pydantic Settings** для считывания и валидации переменных окружения, построенное на **FastAPI** и контейнеризированное с помощью **Docker**. Проект демонстрирует вынос конфигураций в .env файлы и их использование в приложении и Docker Compose.

## Описание

Проект выполнен в рамках задания №14 по реорганизации управления конфигурацией FastAPI-приложения. Основной функционал включает:

- Интеграцию **Pydantic Settings** для валидации и загрузки настроек из переменных окружения.
- Создание и использование .env файла для локальной разработки.
- Рефакторинг кода для использования настроек из Pydantic вместо жестко закодированных значений.
- Настройку **Docker Compose** для передачи конфигураций через переменные окружения.
- Тестирование корректности загрузки настроек локально и в контейнерах.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API.
- **Pydantic Settings** — библиотека для управления и валидации конфигураций.
- **Uvicorn** — ASGI-сервер для запуска FastAPI-приложения.
- **Docker** / **Docker Compose** — контейнеризация и оркестрация.
- **PostgreSQL** — реляционная база данных для хранения данных.
- **python-dotenv** — для загрузки .env файлов в локальной среде.

## 🚀 Установка и запуск

### 1. Запуск через Docker

Убедитесь, что **Docker** и **Docker Desktop** установлены и запущены.

1. Создайте файл `.env` в корне проекта (пример ниже) и добавьте его в `.gitignore`.
2. Соберите и запустите контейнеры:
   ```bash
   docker compose up --build
   ```

3. После успешного запуска сервисы будут доступны:  
   - **API**: [http://localhost:8000](http://localhost:8000)  
   - **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
   - **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

4. Проверьте логи контейнеров (`docker-compose up`), чтобы убедиться, что приложение и база данных запустились без ошибок.

### 2. Локальный запуск (без Docker)

1. Убедитесь, что **PostgreSQL** установлен и запущен локально.
2. Создайте файл `.env` в корне проекта (пример ниже).
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

### Пример .env файла

Создайте файл `.env` в корне проекта и добавьте следующие переменные:
```bash
# .env
DATABASE_URL=postgresql+asyncpg://username:password@localhost:5432/notesdb
SECRET_KEY=your_secret_key_here
REDIS_URL=redis://localhost:6379/0
```

**Важно**: Добавьте `.env` в `.gitignore`, чтобы избежать утечки конфиденциальных данных:
```gitignore
.env
```

## 🧪 Тестирование

### 1. Установка зависимостей

Если зависимости еще не установлены:
```bash
pip install -r requirements.txt
```

### 2. Тестирование локального запуска

1. Убедитесь, что `.env` файл содержит корректные настройки.
2. Запустите приложение локально:
   ```bash
   uvicorn main:app --reload
   ```
3. Выполните тестовый запрос через **Postman**:
   ```
   GET http://localhost:8000/notes
   Authorization: Bearer <JWT-токен>
   ```
4. Проверьте логи приложения, чтобы убедиться, что настройки (например, `DATABASE_URL`) загружены корректно:
   ```
   INFO: Connecting to database: postgresql+asyncpg://username:password@localhost:5432/notesdb
   ```

### 3. Тестирование через Docker

1. Убедитесь, что `docker-compose.yml` настроен для использования переменных окружения (см. раздел "Контейнеры").
2. Запустите контейнеры:
   ```bash
   docker compose up --build
   ```
3. Выполните тот же тестовый запрос через **Postman**.
4. Проверьте логи контейнера `web`:
   ```
   docker logs <container_name>
   ```
   Ожидаемый вывод:
   ```
   INFO: Connecting to database: postgresql+asyncpg://username:password@db:5432/notesdb
   ```

### Возможные ошибки и их решение

- **Pydantic validation error**: Убедитесь, что все обязательные переменные (например, `DATABASE_URL`, `SECRET_KEY`) заданы в `.env` или `docker-compose.yml`.
- **Connection to database failed**: Проверьте, что PostgreSQL запущен и `DATABASE_URL` указывает на правильный хост/порт (в Docker используйте имя сервиса `db`).
- **Environment variable not found**: Убедитесь, что `.env` файл загружается (для локального запуска нужен `python-dotenv`).
- **Docker container fails to start**: Проверьте синтаксис `docker-compose.yml` и наличие всех переменных в `environment` или `env_file`.

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
      "created_at": "2025-06-12T12:39:00+05:00"
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
    "created_at": "2025-06-12T12:39:00+05:00"
  }
  ```

## 🧾 Особенности

- **Pydantic Settings**: Все настройки приложения валидируются и загружаются из переменных окружения.
- **.env файл**: Используется для локальной разработки, игнорируется в Git для безопасности.
- **Docker Compose**: Переменные окружения передаются через `environment` или `env_file`.
- **Рефакторинг**: Жестко закодированные настройки заменены на обращения к `settings.<variable>`.
- **Безопасность**: Конфиденциальные данные (пароли, ключи) хранятся в `.env` и не попадают в репозиторий.
- **Логирование**: Добавлены логи для подтверждения загрузки настроек.

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
      - DATABASE_URL=postgresql+asyncpg://username:password@db:5432/notesdb
    depends_on:
      - db

  db:
    image: postgres
    environment:
      POSTGRES_DB: notesdb
      POSTGRES_USER: username
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## 🗂️ Документация

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

## Контакты

Если вам нужна помощь с проектом или есть вопросы, пишите в **Microsoft Teams**.
# FastAPI: Управление Миграциями Базы Данных с Alembic

REST API с интеграцией **Alembic** для управления миграциями схемы базы данных **PostgreSQL**, построенное на **FastAPI** и контейнеризированное с помощью **Docker**. Проект демонстрирует настройку Alembic для асинхронного SQLAlchemy, создание и применение миграций, а также тестирование изменений схемы.

## Описание

Проект выполнен в рамках задания №16 по интеграции **Alembic** для управления изменениями схемы базы данных. Основной функционал включает:

- Настройку **Alembic** для работы с асинхронным **SQLAlchemy** и моделями приложения.
- Создание и применение миграций для начальной схемы и последующих изменений.
- Модификацию модели (добавление нового поля) и генерацию соответствующей миграции.
- Тестирование применения и отката миграций, а также проверку работы приложения.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API.
- **Alembic** — инструмент для управления миграциями базы данных.
- **SQLAlchemy** — ORM для работы с базой данных (асинхронный режим).
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

4. Примените миграции (внутри контейнера или локально):
   ```bash
   docker exec -it <web_container_name> alembic upgrade head
   ```

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

4. Инициализируйте Alembic (если еще не сделано):
   ```bash
   alembic init alembic
   ```

5. Настройте `alembic.ini` и `alembic/env.py` (см. раздел "Настройка Alembic").
6. Примените миграции:
   ```bash
   alembic upgrade head
   ```

7. Запустите сервер:
   ```bash
   uvicorn main:app --reload
   ```

## 🧪 Тестирование

### 1. Настройка Alembic

1. Установите Alembic:
   ```bash
   pip install alembic
   ```

2. Инициализируйте Alembic:
   ```bash
   alembic init alembic
   ```

3. Настройте `alembic.ini`:
   ```ini
   [alembic]
   sqlalchemy.url = postgresql+asyncpg://username:password@localhost:5432/notesdb
   ```

4. Настройте `alembic/env.py` для асинхронного SQLAlchemy:
   ```python
   from sqlalchemy.ext.asyncio import create_async_engine
   from models import Base  # Импортируйте ваши модели
   # ...
   config = context.config
   connectable = create_async_engine(config.get_main_option("sqlalchemy.url"))
   # ...
   target_metadata = Base.metadata
   ```

### 2. Создание и применение миграций

1. Создайте начальную миграцию:
   ```bash
   alembic revision --autogenerate -m "Initial schema"
   ```
   Проверьте сгенерированный файл в `alembic/versions/`.

2. Примените миграцию:
   ```bash
   alembic upgrade head
   ```

3. Проверьте базу данных (через psql):
   ```sql
   \c notesdb
   \dt
   SELECT * FROM alembic_version;
   ```

4. Модифицируйте модель (например, добавьте `is_completed: bool` в модель `Note`):
   ```python
   class Note(SQLModel, table=True):
       id: Optional[int] = Field(default=None, primary_key=True)
       text: str
       owner_id: int = Field(foreign_key="user.id")
       is_completed: bool = Field(default=False)  # Новое поле
   ```

5. Создайте новую миграцию:
   ```bash
   alembic revision --autogenerate -m "Add completed flag to Note"
   ```

6. Примените новую миграцию:
   ```bash
   alembic upgrade head
   ```

7. Проверьте, что поле появилось:
   ```sql
   \d notes
   ```

8. (Опционально) Откатите миграцию:
   ```bash
   alembic downgrade -1
   ```
   Проверьте, что поле исчезло:
   ```sql
   \d notes
   ```

9. Снова примените миграцию:
   ```bash
   alembic upgrade head
   ```

### 3. Тестирование приложения

1. Выполните запрос через **Postman** для создания заметки:
   ```
   POST http://localhost:8000/notes
   Authorization: Bearer <JWT-токен>
   Content-Type: application/json
   ```
   ```json
   {
     "text": "Тестовая заметка",
     "is_completed": true
   }
   ```

2. Проверьте, что заметка создана с новым полем:
   ```
   GET http://localhost:8000/notes
   Authorization: Bearer <JWT-токен>
   ```
   Ожидаемый ответ:
   ```json
   [
     {
       "id": 1,
       "text": "Тестовая заметка",
       "owner_id": 1,
       "is_completed": true
     }
   ]
   ```

### Возможные ошибки и их решение

- **Alembic cannot detect table**: Убедитесь, что `target_metadata` в `env.py` указывает на `Base.metadata` ваших моделей.
- **Async engine error**: Проверьте, что используете `create_async_engine` и корректный `sqlalchemy.url` в `alembic.ini`.
- **Migration fails to apply**: Проверьте логи Alembic и убедитесь, что база данных доступна и пользователь имеет права на изменение схемы.
- **Model changes not detected**: Убедитесь, что модели импортированы в `env.py` и синхронизированы с базой данных.

## 🧫 Примеры использования (через Postman)

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
    "text": "Новая заметка",
    "is_completed": true
  }
  ```
- **Ответ** (JSON):
  ```json
  {
    "id": 1,
    "text": "Новая заметка",
    "owner_id": 1,
    "is_completed": true
  }
  ```

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
      "text": "Новая заметка",
      "owner_id": 1,
      "is_completed": true
    }
  ]
  ```

## 🧾 Особенности

- **Alembic**: Управляет миграциями базы данных с поддержкой асинхронного SQLAlchemy.
- **Автоматическая генерация**: Миграции создаются с помощью `--autogenerate` на основе изменений моделей.
- **Откат миграций**: Поддерживается откат изменений для тестирования и отладки.
- **Контейнеризация**: Приложение и PostgreSQL упакованы в Docker-контейнеры.
- **Рекомендация для CI/CD**: Добавьте шаг `alembic upgrade head` в GitHub Actions для автоматического применения миграций.

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
    command: sh -c "alembic upgrade head && uvicorn main:app --host 0.0.0.0 --port 8000"

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
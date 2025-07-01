# FastAPI: Углубленное Документирование API

REST API с улучшенной документацией OpenAPI, построенное на **FastAPI** с использованием **Pydantic** для детализации схем и контейнеризированное с помощью **Docker**. Проект демонстрирует настройку тегов, описаний, примеров и ответов для создания информативной и понятной документации API.

## Описание

Проект выполнен в рамках задания №19 по углубленному документированию API в FastAPI-приложении. Основной функционал включает:

- Группировку эндпоинтов с помощью тегов (например, "Authentication", "Notes").
- Добавление подробных описаний и кратких резюме для каждого эндпоинта.
- Детализацию возможных ответов с указанием статус-кодов и структур ошибок.
- Настройку Pydantic-схем с примерами и описаниями полей.
- Указание общей информации об API (название, версия, контактные данные).
- Тестирование отображения документации в **Swagger UI** и **ReDoc**.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API с поддержкой OpenAPI.
- **Pydantic** — библиотека для валидации данных и улучшения документации схем.
- **Uvicorn** — ASGI-сервер для запуска FastAPI-приложения.
- **Docker** / **Docker Compose** — контейнеризация и оркестрация.
- **Swagger UI** / **ReDoc** — инструменты для визуализации документации API.

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

1. Активируйте виртуальное окружение и установите зависимости:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Для Linux/macOS
   .venv\Scripts\activate     # Для Windows
   pip install -r requirements.txt
   ```

2. Запустите сервер:
   ```bash
   uvicorn main:app --reload
   ```

## 🧪 Тестирование

### 1. Настройка документации

1. Настройте приложение в `main.py`:
   ```python
   from fastapi import FastAPI

   app = FastAPI(
       title="Notes API",
       description="A REST API for managing user notes with JWT authentication and advanced documentation.",
       version="1.0.0",
       contact={
           "name": "API Support Team",
           "email": "support@notesapi.com",
           "url": "https://notesapi.com/support"
       }
   )
   ```

2. Настройте Pydantic-схемы в `models.py`:
   ```python
   from pydantic import BaseModel, Field

   class NoteCreate(BaseModel):
       text: str = Field(
           description="The content of the note.",
           example="This is a sample note."
       )
       is_completed: bool = Field(
           default=False,
           description="Indicates whether the note is completed.",
           example=False
       )

   class ErrorResponse(BaseModel):
       detail: str = Field(
           description="Error message describing the issue.",
           example="Invalid credentials provided."
       )
   ```

3. Настройте эндпоинты с тегами, описаниями и responses в `routes.py`:
   ```python
   from fastapi import APIRouter, HTTPException
   from models import NoteCreate, ErrorResponse

   router = APIRouter(prefix="/notes", tags=["Notes"])

   @router.post(
       "/",
       summary="Create a new note",
       description="Creates a new note for the authenticated user. The note is tied to the user's ID extracted from the JWT token.",
       responses={
           201: {"description": "Note created successfully", "model": NoteCreate},
           401: {"description": "Unauthorized", "model": ErrorResponse},
           429: {"description": "Too Many Requests", "model": ErrorResponse}
       }
   )
   async def create_note(note: NoteCreate):
       return {"text": note.text, "is_completed": note.is_completed}
   ```

### 2. Тестирование документации

1. Запустите приложение:
   ```bash
   docker compose up
   ```

2. Откройте **Swagger UI**:
   ```
   http://localhost:8000/docs
   ```
   - Проверьте, что эндпоинты сгруппированы по тегам ("Authentication", "Notes").
   - Убедитесь, что для каждого эндпоинта отображаются `summary` и `description`.
   - Проверьте, что для `/notes` указаны примеры запросов и ответов, а также возможные статус-коды (201, 401, 429).
   - Проверьте, что поля Pydantic-схем содержат описания и примеры.

3. Откройте **ReDoc**:
   ```
   http://localhost:8000/redoc
   ```
   - Убедитесь, что структура документации соответствует Swagger UI.
   - Проверьте наличие общей информации об API (title, description, version, contact).

4. Выполните тестовый запрос через **Swagger UI**:
   - Отправьте `POST /notes` с телом:
     ```json
     {
       "text": "Sample note",
       "is_completed": false
     }
     ```
   - Убедитесь, что ответ соответствует описанной схеме.

### Возможные ошибки и их решение

- **Теги не отображаются**: Убедитесь, что `tags` указаны в `APIRouter` или декораторах эндпоинтов.
- **Примеры не видны**: Проверьте, что `Field(example=...)` или `examples` добавлены в Pydantic-схемы или декораторы.
- **Responses не отображаются**: Убедитесь, что параметр `responses` в декораторе содержит корректные модели и описания.
- **Swagger UI/ReDoc не загружаются**: Проверьте, что приложение запущено и порты 8000 не заняты.

## 🧫 Примеры использования (через Swagger UI/Postman)

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
    "is_completed": false
  }
  ```
- **Ответ** (JSON, статус 201):
  ```json
  {
    "text": "Новая заметка",
    "is_completed": false
  }
  ```
- **Ошибка** (JSON, статус 401):
  ```json
  {
    "detail": "Unauthorized"
  }
  ```

### 📒 Получение заметок `GET /notes`
- **Метод**: GET
- **URL**: `/notes?skip=0&limit=10`
- **Заголовки**:
  ```
  Authorization: Bearer <JWT-токен>
  ```
- **Ответ** (JSON, статус 200):
  ```json
  [
    {
      "id": 1,
      "text": "Тестовая заметка",
      "owner_id": 1,
      "is_completed": true,
      "created_at": "2025-06-12T03:18:00+05:00"
    }
  ]
  ```
- **Ошибка** (JSON, статус 429):
  ```json
  {
    "detail": "Too Many Requests"
  }
  ```

## 🧾 Особенности

- **Теги**: Эндпоинты сгруппированы по категориям ("Authentication", "Notes") для удобной навигации.
- **Описания**: Каждый эндпоинт имеет `summary` и `description` для пояснения его функционала.
- **Responses**: Указаны все возможные статус-коды (200, 201, 400, 401, 429) с моделями ответов.
- **Pydantic-схемы**: Поля содержат описания и примеры через `Field(description=..., example=...)`.
- **Общая информация**: API имеет название, описание, версию и контактные данные.
- **Рекомендация**: Регулярно обновляйте документацию при добавлении новых эндпоинтов.

## 📦 Контейнеры

Конфигурация **Docker Compose** включает один сервис:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    env_file:
      - .env
```

## 🗂️ Документация

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

## Контакты

Если вам нужна помощь с проектом или есть вопросы, пишите в **Microsoft Teams**.
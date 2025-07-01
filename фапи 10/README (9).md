# FastAPI: Контейнеризация, JWT и CRUD API с Автотестами

REST API для управления заметками с аутентификацией на основе **JWT-токенов**, реализованное с использованием **FastAPI**, **SQLModel**, **PostgreSQL** и **Docker**. Проект включает автоматическое тестирование с помощью **pytest** и **httpx**, а также контейнеризацию для упрощённого развёртывания.

## Описание

Проект выполнен в рамках задания по контейнеризации и развёртыванию FastAPI-приложения с базой данных **PostgreSQL** с использованием **Docker** и **Docker Compose**. Основной функционал включает:

- Регистрацию и аутентификацию пользователей с выдачей **JWT-токенов**.
- CRUD-операции для заметок с ограничением доступа (пользователь видит только свои заметки).
- Асинхронное взаимодействие с базой данных.
- Автоматизированное тестирование API с помощью **pytest**, **httpx** и **pytest-asyncio**.
- Контейнеризацию приложения и базы данных для удобного развёртывания.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API
- **SQLModel** — ORM для работы с базой данных
- **PostgreSQL** — реляционная база данных
- **Docker** / **Docker Compose** — контейнеризация и оркестрация
- **PyJWT** / **Passlib** — генерация JWT-токенов и хеширование паролей
- **pytest**, **httpx**, **pytest-asyncio** — инструменты для тестирования API

## 🚀 Установка и запуск

### 1. Запуск через Docker

Убедитесь, что **Docker** и **Docker Desktop** установлены и запущены.

1. Соберите и запустите контейнеры:
   ```bash
   docker compose up --build
   ```

2. После успешного запуска API будет доступен:  
   - **API**: [http://localhost:8000](http://localhost:8000)  
   - **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
   - **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

3. Проверьте логи контейнеров (`docker-compose up`), чтобы убедиться, что приложение и база данных запустились без ошибок.

### 2. Локальный запуск (без Docker)

1. Активируйте виртуальное окружение и установите зависимости:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Для Linux/macOS
   .venv\Scripts\activate     # Для Windows
   pip install -r requirements.txt
   ```

2. Создайте базу данных вручную, если она еще не существует:
   ```sql
   CREATE DATABASE notesdb;
   ```

3. Настройте подключение к базе данных в файле `database.py`, указав свои параметры:
   ```python
   DATABASE_URL = "postgresql+asyncpg://username:password@localhost:5432/notesdb"
   ```

4. Запустите сервер:
   ```bash
   uvicorn main:app --reload
   ```

## 🧪 Тестирование

### 1. Установка зависимостей для тестов

Если зависимости еще не установлены:
```bash
pip install -r requirements.txt
```

### 2. Запуск тестов

Запустите тесты с помощью команды:
```bash
pytest test_main.py
```

Тесты охватывают:  
- Регистрацию и вход пользователей  
- CRUD-операции с заметками (создание, получение, удаление)  
- Защиту данных (доступ только к своим заметкам)  
- Обработку ошибок аутентификации (например, отсутствие токена)

## 🧫 Примеры использования (через Postman)

### 📌 Регистрация `/register`
- **Метод**: POST
- **URL**: `/register`
- **Content-Type**: `application/json`
- **Тело запроса** (JSON):
  ```json
  {
    "username": "alice",
    "password": "secret"
  }
  ```
- **Ответ** (JSON):
  ```json
  {
    "id": 1,
    "username": "alice"
  }
  ```

### 🔐 Логин `/login`
- **Метод**: POST
- **URL**: `/login`
- **Content-Type**: `application/json`
- **Тело запроса** (JSON):
  ```json
  {
    "username": "alice",
    "password": "secret"
  }
  ```
- **Ответ** (JSON):
  ```json
  {
    "access_token": "jwt.token.here",
    "token_type": "bearer"
  }
  ```

### 📒 CRUD заметок `/notes`

#### Создание заметки
- **Метод**: POST
- **URL**: `/notes`
- **Заголовки**:
  ```
  Authorization: Bearer <токен>
  Content-Type: application/json
  ```
- **Тело запроса** (JSON):
  ```json
  {
    "text": "первая заметка"
  }
  ```
- **Ответ** (JSON):
  ```json
  {
    "id": 1,
    "text": "первая заметка",
    "owner_id": 1
  }
  ```

#### Получение всех заметок
- **Метод**: GET
- **URL**: `/notes`
- **Заголовки**:
  ```
  Authorization: Bearer <токен>
  ```
- **Ответ** (JSON):
  ```json
  [
    {
      "id": 1,
      "text": "первая заметка",
      "owner_id": 1
    }
  ]
  ```

#### Удаление заметки
- **Метод**: DELETE
- **URL**: `/notes/{id}`
- **Заголовки**:
  ```
  Authorization: Bearer <токен>
  ```
- **Ответ** (JSON):
  ```json
  {
    "message": "Note deleted successfully"
  }
  ```

## 🧾 Особенности

- **Асинхронность**: Работа с базой данных реализована асинхронно (`asyncpg`, `sqlalchemy.ext.asyncio`).
- **Аутентификация**: Все эндпоинты защищены с помощью **JWT-токенов**.
- **Безопасность**: Пароли хешируются с использованием **bcrypt**.
- **Ограничение доступа**: Каждый пользователь имеет доступ только к своим заметкам.
- **Контейнеризация**: Приложение и база данных упакованы в контейнеры с помощью **Docker Compose**.
- **Переменные окружения**: Настройки, такие как `DATABASE_URL` и `SECRET_KEY`, передаются через переменные окружения в `docker-compose.yml`.

## 📦 Контейнеры

Конфигурация **Docker Compose** включает два сервиса:

```yaml
# docker-compose.yml
services:
  db:
    image: postgres
    environment:
      POSTGRES_DB: notesdb
      POSTGRES_USER: username
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://username:password@db/notesdb
      - SECRET_KEY=your_secret_key_here
    depends_on:
      - db
volumes:
  postgres_data:
networks:
  default:
    driver: bridge
```

## 🗂️ Документация

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

## Контакты

Если вам нужна помощь с проектом или есть вопросы, пишите мне в **Microsoft Teams**.
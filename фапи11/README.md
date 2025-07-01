# FastAPI: Фоновые Задачи с Celery и Redis

REST API с реализацией фоновых задач, использующее **FastAPI**, **Celery**, **Redis** и **Docker**. Проект демонстрирует асинхронное выполнение длительных операций в фоновом режиме с использованием Celery в качестве брокера задач и Redis в качестве брокера сообщений.

## Описание

Проект выполнен в рамках задания по внедрению системы для асинхронного выполнения длительных задач в фоновом режиме. Основной функционал включает:

- Настройку инфраструктуры с использованием Celery и Redis
- Создание и выполнение фоновых задач
- Интеграцию фоновых задач с FastAPI эндпоинтами
- Контейнеризацию всех компонентов системы

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API
- **Celery** — распределенная очередь задач
- **Redis** — брокер сообщений для Celery
- **Docker** / **Docker Compose** — контейнеризация и оркестрация
- **JWT** — аутентификация и авторизация
- **Uvicorn** — ASGI сервер

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

3. Проверьте логи контейнеров (`docker-compose up`), чтобы убедиться, что все сервисы запустились без ошибок:
   - FastAPI приложение
   - Redis
   - Celery Worker

### 2. Локальный запуск (без Docker)

1. Активируйте виртуальное окружение и установите зависимости:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Для Linux/macOS
   .venv\Scripts\activate     # Для Windows
   pip install -r requirements.txt
   ```

2. Запустите Redis (убедитесь, что Redis установлен и запущен)

3. Запустите Celery Worker:
   ```bash
   celery -A app.celery_app worker --loglevel=info
   ```

4. В отдельном терминале запустите FastAPI сервер:
   ```bash
   uvicorn main:app --reload
   ```

## 🧫 Примеры использования (через Postman)

### 🔄 Запуск фоновой задачи `/trigger-task`
- **Метод**: POST
- **URL**: `/trigger-task`
- **Заголовки**:
  ```
  Authorization: Bearer <jwt_token>
  Content-Type: application/json
  ```
- **Тело запроса** (JSON):
  ```json
  {
    "email": "user@example.com"
  }
  ```
- **Ответ** (JSON):
  ```json
  {
    "message": "Task started"
  }
  ```

## 🧾 Особенности

- **Асинхронное выполнение**: Длительные задачи выполняются в фоновом режиме
- **Масштабируемость**: Возможность горизонтального масштабирования через добавление Celery workers
- **Надёжность**: Использование Redis в качестве брокера сообщений обеспечивает надёжную доставку задач
- **Контейнеризация**: Все компоненты системы упакованы в Docker контейнеры
- **Безопасность**: Эндпоинты защищены с помощью JWT аутентификации

## 📦 Контейнеры

Конфигурация **Docker Compose** включает три сервиса:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - redis
    environment:
      - REDIS_URL=redis://redis:6379/0

  redis:
    image: redis:latest
    ports:
      - "6379:6379"

  celery_worker:
    build: .
    command: celery -A app.celery_app worker --loglevel=info
    depends_on:
      - redis
    environment:
      - REDIS_URL=redis://redis:6379/0
```

## 🗂️ Документация

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

## Контакты

Если вам нужна помощь с проектом или есть вопросы, пишите мне в **Microsoft Teams**.
# Проект FastAPI с Регистрацией и Входом Пользователей с Хешированием Паролей

API на основе **FastAPI** для регистрации и аутентификации пользователей с безопасным хранением паролей в базе данных с использованием библиотеки **Passlib** и схемы **bcrypt**.

## Описание

Данный проект представляет собой учебный пример разработки REST API с использованием **FastAPI**, обеспечивающего регистрацию и вход пользователей. Пароли хранятся в зашифрованном виде для повышения безопасности.

## Технологии

- **Python 3.10+**
- **FastAPI**
- **Uvicorn** (ASGI-сервер)
- **PostgreSQL**
- **SQLModel** / **SQLAlchemy** (ORM)
- **Passlib** (схема **bcrypt** для хеширования)
- **asyncpg** (асинхронный драйвер для PostgreSQL)

## Установка и запуск

1. **Клонирование или создание проекта**:  
   Клонируйте репозиторий или настройте новый проект.

2. **Создание и активация виртуального окружения**:  
   - Создайте окружение:
     ```bash
     python -m venv .venv
     ```
   - Активируйте окружение:
     - Для Linux/macOS:
       ```bash
       source .venv/bin/activate
       ```
     - Для Windows:
       ```bash
       .venv\Scripts\activate
       ```

3. **Установка зависимостей**:  
   Установите необходимые пакеты:
   ```bash
   pip install fastapi uvicorn sqlmodel asyncpg psycopg2-binary passlib[bcrypt]
   ```

4. **Настройка подключения к базе данных**:  
   В файле `main.py` обновите строку подключения, указав свои данные:
   ```python
   DATABASE_URL = "postgresql+asyncpg://username:password@localhost:5432/notesdb"
   ```
   Замените `username`, `password` и имя базы данных (`notesdb`) на свои.

5. **Запуск приложения**:  
   Запустите сервер:
   ```bash
   uvicorn main:app --reload
   ```

6. **Доступ к документации**:  
   - **Swagger UI**: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
   - **ReDoc**: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

## API

### POST /register
- **Описание**: Регистрация нового пользователя.
- **Входные данные** (JSON):
  ```json
  {
    "username": "string",
    "password": "string"
  }
  ```
- **Особенности**: Пароль автоматически хешируется перед сохранением в базу данных.
- **Возвращает** (JSON):
  ```json
  {
    "id": 1,
    "username": "string"
  }
  ```

### POST /login
- **Описание**: Аутентификация пользователя.
- **Входные данные** (JSON):
  ```json
  {
    "username": "string",
    "password": "string"
  }
  ```
- **Особенности**: Пароль проверяется с использованием хеширования **bcrypt**.
- **Возвращает** (JSON) при успехе:
  ```json
  {
    "message": "Login successful",
    "username": "string"
  }
  ```
- **Ошибки**: `401 Unauthorized` — при неверных данных пользователя или пароле.

## Безопасность

- **Хеширование**: Используется **bcrypt** из библиотеки **Passlib** для безопасного хранения паролей.
- **Рекомендация**: Никогда не храните пароли в открытом виде в реальных проектах.

## Миграции и база данных

- **Автоматическое создание таблиц**: Таблицы создаются автоматически при запуске приложения с помощью **SQLModel**.
- **Обновление паролей**: Если пароли ранее хранились в открытом виде, реализуйте отдельную функцию для их хеширования.

## Тестирование

- **Рекомендации**: Тестируйте API через **Swagger UI**, **ReDoc** или **Postman**.
- **Проверяйте**:
  - Регистрацию нового пользователя.
  - Вход с корректным и некорректным паролем.
  - Хранение пароля в базе данных в виде хеша.

## Контакты

Если у вас возникнут вопросы или потребуется помощь, обращайтесь ко мне в Microsoft Teams.
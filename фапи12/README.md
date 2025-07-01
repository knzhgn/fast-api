# FastAPI: Внедрение WebSockets для Обмена Сообщениями в Реальном Времени

REST API с реализацией обмена сообщениями в реальном времени через **WebSockets**, построенное на **FastAPI** и контейнеризированное с использованием **Docker**. Проект демонстрирует настройку WebSocket-эндпоинта для передачи сообщений между клиентами в реальном времени, включая управление активными соединениями и тестирование функционала.

## Описание

Проект выполнен в рамках задания №12 по внедрению механизма обмена сообщениями в реальном времени с использованием **WebSockets** в **FastAPI**. Основной функционал включает:

- Реализацию WebSocket-эндпоинта `/ws` для обработки входящих соединений.
- Создание менеджера соединений для отслеживания активных клиентов.
- Передачу сообщений от одного клиента всем остальным в реальном времени.
- Тестирование функционала через **Postman** или простую HTML-страницу с JavaScript.
- Контейнеризацию приложения для упрощённого развёртывания.

## ⚙️ Используемые технологии

- **FastAPI** — высокопроизводительный фреймворк для создания API и работы с WebSockets.
- **Uvicorn** — ASGI-сервер для запуска FastAPI-приложения.
- **Docker** / **Docker Compose** — контейнеризация и оркестрация.
- **Python-WebSocket** — библиотека для работы с WebSocket-клиентами.
- **HTML/JavaScript** — для создания тестового клиента (опционально).
- **Postman** — для тестирования WebSocket-соединений.

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
   - **WebSocket**: [ws://localhost:8000/ws](ws://localhost:8000/ws)

3. Проверьте логи контейнеров (`docker-compose up`), чтобы убедиться, что приложение запустилось без ошибок.

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

### 1. Установка зависимостей

Если зависимости еще не установлены:
```bash
pip install -r requirements.txt
```

### 2. Тестирование WebSocket

#### Через Postman
1. Откройте **Postman** и создайте новый WebSocket-запрос.
2. Подключитесь к эндпоинту: `ws://localhost:8000/ws`.
3. Отправьте текстовое сообщение (например, `"Hello, WebSocket!"`).
4. Откройте несколько вкладок Postman, подключите их к тому же эндпоинту и убедитесь, что сообщение от одного клиента отображается у всех остальных.

#### Через HTML-страницу
1. Создайте файл `index.html` с JavaScript для подключения к WebSocket:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>WebSocket Client</title>
   </head>
   <body>
       <h1>WebSocket Test Client</h1>
       <input type="text" id="messageInput" placeholder="Type a message...">
       <button onclick="sendMessage()">Send</button>
       <ul id="messages"></ul>
       <script>
           const ws = new WebSocket("ws://localhost:8000/ws");
           ws.onmessage = function(event) {
               const messages = document.getElementById("messages");
               const li = document.createElement("li");
               li.textContent = event.data;
               messages.appendChild(li);
           };
           function sendMessage() {
               const input = document.getElementById("messageInput");
               ws.send(input.value);
               input.value = "";
           }
       </script>
   </body>
   </html>
   ```
2. Откройте `index.html` в нескольких вкладках браузера.
3. Отправьте сообщение из одной вкладки и убедитесь, что оно отображается во всех остальных в реальном времени.

## 🧫 Примеры использования

### 🔗 Подключение к WebSocket `/ws`
- **Протокол**: WebSocket
- **URL**: `ws://localhost:8000/ws`
- **Действие**: Подключение к эндпоинту и отправка текстового сообщения.
- **Пример сообщения** (отправляется клиентом):
  ```json
  "Hello, WebSocket!"
  ```
- **Ответ**: Сообщение рассылается всем подключённым клиентам:
  ```json
  "Hello, WebSocket!"
  ```

## 🧾 Особенности

- **WebSocket-эндпоинт**: Реализован эндпоинт `/ws` для обмена сообщениями в реальном времени.
- **Менеджер соединений**: Управляет активными WebSocket-соединениями, поддерживая подключение/отключение клиентов и рассылку сообщений.
- **Асинхронность**: Используется асинхронная обработка соединений через **FastAPI** и **WebSocket**.
- **Контейнеризация**: Приложение упаковано в Docker-контейнер для удобного развёртывания.
- **Тестирование**: Поддерживается тестирование через **Postman** или HTML-клиент.

## 📦 Контейнеры

Конфигурация **Docker Compose** включает один сервис:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - PYTHONUNBUFFERED=1
```

## 🗂️ Документация

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)  
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)

## Контакты

Если вам нужна помощь с проектом или есть вопросы, пишите в **Microsoft Teams**.
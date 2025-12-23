# Практическое задание 9

## ЭФМО-02-25 Мишкин Артём Дмитриевич 23.12.2025

---

## Информация о проекте

REST API на Go для регистрации и входа пользователей с безопасным хранением паролей через **bcrypt**. Используется **PostgreSQL** (через GORM) для хранения пользователей и **chi** для маршрутизации HTTP-запросов. Реализованы эндпоинты:

- `POST /auth/register` — регистрация пользователя (email + password)
- `POST /auth/login` — вход пользователя по email и паролю

Проект подготавливает основу для JWT-аутентификации в следующей практике (ПЗ №10).

## Цели занятия

- Научиться безопасно хранить пароли с помощью **bcrypt**
- Реализовать эндпоинты `POST /auth/register` и `POST /auth/login`
- Закрепить работу с PostgreSQL и GORM, а также валидацией ввода
- Понять базовые подходы к обработке ошибок при регистрации и логине

## Файловая структура проекта

```
pz9-auth/
├── cmd/api/main.go
├── internal/
│   ├── core/
│   │   └── user.go
│   ├── http/
│   │   └── handlers/
│   │       └── auth.go
│   ├── platform/
│   │   └── config/
│   │       └── config.go
│   └── repo/
│       ├── postgres.go
│       └── user_repo.go
├── docker-compose.yml
├── go.mod
└── README.md
```

## Запуск проекта

### 1. Запуск PostgreSQL (Docker)

```powershell
cd pz9-auth
docker compose up -d
docker ps
```

### 2. Установка переменных окружения

```powershell
$env:DB_DSN="postgres://pz9user:pz9pass@localhost:5432/pz9?sslmode=disable"
$env:BCRYPT_COST="12"
$env:APP_ADDR=":8080"
```

### 3. Инициализация зависимостей

```powershell
go mod tidy
```

### 4. Запуск сервера

```powershell
go run ./cmd/api/main.go
```

**Ожидаемый результат:**
```
listening on :8080
```

---

## Тестирование API

### Подготовка переменных PowerShell

```powershell
$apiUrl = "http://localhost:8080"
$headers = @{ "Content-Type" = "application/json" }
```

---

## Тест 1: Успешная регистрация (201 Created)

**Запрос:**
```powershell
$registerBody = @{
    email    = "user@example.com"
    password = "Secret123!"
} | ConvertTo-Json

Invoke-WebRequest -Uri "$apiUrl/auth/register" `
    -Method POST `
    -Headers $headers `
    -Body $registerBody
```


<img width="974" height="772" alt="image" src="https://github.com/user-attachments/assets/55ef016d-00d2-40d5-b8cd-b083f1929e04" />


---

## Тест 2: Повторная регистрация (409 Conflict)

**Запрос:**
```powershell
Invoke-WebRequest -Uri "$apiUrl/auth/register" `
    -Method POST `
    -Headers $headers `
    -Body $registerBody
```

<img width="974" height="106" alt="image" src="https://github.com/user-attachments/assets/346a4ca3-dd24-4941-a3d1-f30e0f7b4b02" />


---

## Тест 3: Успешный вход (200 OK)

**Запрос:**
```powershell
$loginBody = @{
    email    = "user@example.com"
    password = "Secret123!"
} | ConvertTo-Json

Invoke-WebRequest -Uri "$apiUrl/auth/login" `
    -Method POST `
    -Headers $headers `
    -Body $loginBody
```

<img width="974" height="221" alt="image" src="https://github.com/user-attachments/assets/c43cc320-6bdb-4b93-a37c-a81e4849d5c2" />


---

## Тест 4: Неверный пароль (401 Unauthorized)

**Запрос:**
```powershell
$wrongBody = @{
    email    = "user@example.com"
    password = "wrongpass"
} | ConvertTo-Json

Invoke-WebRequest -Uri "$apiUrl/auth/login" `
    -Method POST `
    -Headers $headers `
    -Body $wrongBody `
    -ErrorAction SilentlyContinue
```


<img width="974" height="264" alt="image" src="https://github.com/user-attachments/assets/419ec978-e566-48d1-a8c0-3a2f9c3f5f66" />


---

## Тест 5: Пустой email (400 Bad Request)

**Запрос:**
```powershell
$invalidBody = @{
    email    = ""
    password = "Secret123!"
} | ConvertTo-Json

Invoke-WebRequest -Uri "$apiUrl/auth/register" `
    -Method POST `
    -Headers $headers `
    -Body $invalidBody `
    -ErrorAction SilentlyContinue
```

<img width="974" height="286" alt="image" src="https://github.com/user-attachments/assets/83803568-91d0-46f7-b524-648575e8d5c6" />


---

## Тест 6: Короткий пароль (400 Bad Request)

**Запрос:**
```powershell
$shortPassBody = @{
    email    = "user2@example.com"
    password = "123"
} | ConvertTo-Json

Invoke-WebRequest -Uri "$apiUrl/auth/register" `
    -Method POST `
    -Headers $headers `
    -Body $shortPassBody `
    -ErrorAction SilentlyContinue
```

<img width="974" height="420" alt="image" src="https://github.com/user-attachments/assets/aab2e1cb-9108-46da-9ebc-ea7ed5a554e0" />

---


## Docker конфигурация

**Файл: `docker-compose.yml`**

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:16
    container_name: pz9-postgres
    environment:
      POSTGRES_USER: pz9user
      POSTGRES_PASSWORD: pz9pass
      POSTGRES_DB: pz9
    ports:
      - "5432:5432"
    volumes:
      - pz9_pg_data:/var/lib/postgresql/data

volumes:
  pz9_pg_data:
```

---

## Go зависимости

**Файл: `go.mod`**

```go
module example.com/pz9-auth

go 1.22

require (
    github.com/go-chi/chi/v5 v5.0.12
    golang.org/x/crypto v0.28.0
    gorm.io/driver/postgres v1.5.11
    gorm.io/gorm v1.25.10
)
```

**Установка:**
```powershell
go mod tidy
```

---

## Переменные окружения

| Переменная | Значение | Описание |
|-----------|----------|----------|
| `DB_DSN` | `postgres://pz9user:pz9pass@localhost:5432/pz9?sslmode=disable` | Подключение к PostgreSQL |
| `BCRYPT_COST` | `12` | Сложность хэширования (2^12=4096 итераций) |
| `APP_ADDR` | `:8080` | Адрес слушания сервера |

**Установка (Windows PowerShell):**
```powershell
$env:DB_DSN="postgres://pz9user:pz9pass@localhost:5432/pz9?sslmode=disable"
$env:BCRYPT_COST="12"
$env:APP_ADDR=":8080"
```

---

## Контрольные вопросы и ответы

### 1. Разница между хранением пароля и хэша? Зачем соль? Почему bcrypt, а не SHA-256?

**Хранение пароля в открытом виде:**
- При утечке БД злоумышленник получает все пароли в чистом виде
- Полный доступ ко всем аккаунтам
- Каждая система становится точкой отказа для других сервисов (если пользователь переиспользует пароли)

**Хранение хэша:**
- Одностороннее преобразование
- Невозможно восстановить пароль из хэша математически
- При утечке БД хэши остаются бесполезны для хакера

**Соль (salt):**
- Случайная строка, добавляемая к паролю перед хэшированием
- Одинаковые пароли дают разные хэши
- Защищает от rainbow tables (предрассчитанных таблиц хэшей)
- **bcrypt включает соль автоматически** в результирующий хэш

**SHA-256 vs bcrypt:**
- SHA-256: очень быстрый (~миллиарды хэшей/сек на GPU), подходит для целостности данных
- bcrypt: специально разработан для паролей, медленный, имеет параметр `cost` для регулировки
- bcrypt делает brute force атаки медленнее и дороже

---

### 2. Что происходит при изменении `cost` в bcrypt? Как подобрать значение?

**Меньше cost (8-10):**
- Регистрация и логин быстрее (50-100мс)
- Brute force атака выполняется быстрее
- Подходит для большого трафика, но менее безопасно

**Больше cost (14-16):**
- Регистрация и логин медленнее (500мс - 1сек)
- Brute force практически невозможен на обычном железе
- Может перегружать сервер при пиках нагрузки

**Рекомендация:**
- Измерить время выполнения на целевой машине
- Выбрать cost так, чтобы хэширование занимало 100-300мс
- **В ПЗ9 используем cost=12** (хороший компромисс для dev)

**Формула:** cost=N означает 2^N итераций (cost=12 → 4096 итераций)

---

### 3. Какие статусы и ответы должны возвращать `/auth/register` и `/auth/login`?

**POST /auth/register:**

| Статус | Сценарий | Тело ответа |
|--------|----------|-----------|
| 201 Created | Успешная регистрация | `{"status":"ok","user":{"id":..., "email":...}}` |
| 400 Bad Request | Пустой email / пароль < 8 символов | `{"error":"email_required_and_password_min_8"}` |
| 409 Conflict | Email уже существует | `{"error":"email_taken"}` |
| 500 Internal Server Error | Ошибка БД / bcrypt | `{"error":"db_error"}` или `{"error":"hash_failed"}` |

**POST /auth/login:**

| Статус | Сценарий | Тело ответа |
|--------|----------|-----------|
| 200 OK | Успешный вход | `{"status":"ok","user":{"id":..., "email":...}}` |
| 400 Bad Request | Пустые поля | `{"error":"email_and_password_required"}` |
| 401 Unauthorized | Неверный email или пароль | `{"error":"invalid_credentials"}` |
| 500 Internal Server Error | Ошибка БД | `{"error":"db_error"}` |

---

### 4. Какие риски несут подробные сообщения об ошибках при логине?

**Проблемные сообщения:**
```
 "Email не найден в системе"
   → Атакующий узнаёт существующие email'ы, может собрать базу email'ов

 "Пароль неверный"
   → Атакующий знает, что email существует, может перебирать пароли

"Пользователь заблокирован"
   → Раскрывает бизнес-логику, позволяет сужать область атаки
```

**Правильный подход:**
```
 "Неверные учетные данные" (invalid_credentials)
   → Не раскрывает, что именно неверно (email или пароль)
   → Замедляет перебор (атакующий не может оптимизировать)
   → Невозможно узнать существующие email'ы
```

**Реализация в коде:**
```go
if err != nil {
    // Не говорим "email не найден"
    writeErr(w, http.StatusUnauthorized, "invalid_credentials")
    return
}

if bcrypt.CompareHashAndPassword(...) != nil {
    // Не говорим "пароль неверный"
    writeErr(w, http.StatusUnauthorized, "invalid_credentials")
    return
}
```

---

### 5. Почему в этом ПЗ не выдаём токен, и что изменится в ПЗ10 (JWT)?

**ПЗ9 - текущая практика:**
- Цель: безопасно хранить пароли (bcrypt)
- Цель: реализовать регистрацию и проверку пароля
- Нет сессий, нет токенов
- После успешного логина возвращаем только `{"status":"ok"}`

**ПЗ10 - следующая практика:**
- Добавляем **JWT (JSON Web Token)**
- При успешном логине: `{"status":"ok","token":"eyJhbGc..."}`
- **JWT содержит:** id пользователя, email, время выдачи, время истечения
- **Подпись:** гарантирует, что токен не подделан (secret key на сервере)
- **Middleware:** проверяет токен в заголовке `Authorization: Bearer <token>`
- **Refresh token:** для обновления JWT без повторного логина

**Отличие:**
```
ПЗ9:  POST /auth/login → проверка пароля → ответ "ok" → без доступа к защищённым ресурсам
ПЗ10: POST /auth/login → проверка пароля → выдача JWT → доступ к /api/users (с проверкой токена)
```

---


# Kommo API - Официальная документация

## Обзор

Kommo (бывший AmoCRM) - это CRM-система с REST API для управления контактами, сделками и задачами. Данная документация основана на официальной технической документации Kommo API.

## Основные возможности API

### Поддерживаемые сущности
- **Leads (Сделки)** - управление сделками, воронками и этапами
- **Contacts (Контакты)** - управление контактами и их данными
- **Companies (Компании)** - управление компаниями
- **Tasks (Задачи)** - создание и управление задачами
- **Notes (Примечания)** - добавление примечаний к сущностям
- **Tags (Теги)** - работа с тегами
- **Custom Fields (Пользовательские поля)** - управление дополнительными полями
- **Users and Roles (Пользователи и роли)** - управление пользователями
- **Webhooks (Вебхуки)** - настройка уведомлений о событиях
- **Pipelines and Stages (Воронки и этапы)** - управление воронками продаж

### Основные особенности
- REST API с поддержкой JSON
- OAuth 2.0 авторизация
- Поддержка HTTPS (TLS 1.2)
- Пагинация результатов
- Система webhooks для уведомлений
- Chats API для интеграции с мессенджерами
- VoIP API для интеграции с телефонией
- Kommo AI для автоматизации

## Авторизация

### OAuth 2.0
Kommo использует OAuth 2.0 для авторизации:

1. **Authorization Code** - временный код (действует 20 минут)
2. **Access Token** - токен доступа (действует 24 часа)
3. **Refresh Token** - токен обновления (действует 3 месяца)

### Параметры авторизации
```
client_id - ID интеграции
client_secret - Секретный ключ интеграции
redirect_uri - URI для возврата после авторизации
```

### Получение токена
```
POST https://subdomain.kommo.com/oauth2/access_token
Content-Type: application/json

{
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "grant_type": "authorization_code",
  "code": "authorization_code",
  "redirect_uri": "your_redirect_uri"
}
```

### Обновление токена
```
POST https://subdomain.kommo.com/oauth2/access_token
Content-Type: application/json

{
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "grant_type": "refresh_token",
  "refresh_token": "your_refresh_token",
  "redirect_uri": "your_redirect_uri"
}
```

## Базовые настройки

### Домен API
Все запросы должны выполняться на конкретный домен аккаунта:
```
https://subdomain.kommo.com/api/v4/
```

### Заголовки
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Поддерживаемые протоколы
- **TLS 1.2** (рекомендуется)
- **TLS 1.1**

## Лимиты API

### Лимиты запросов
- **Максимум 7 запросов в секунду**
- При превышении возвращается HTTP 429
- При злоупотреблении IP блокируется (HTTP 403)

### Лимиты данных
- **Максимум 250 сущностей** в одном запросе (рекомендуется не более 50)
- **Максимум 100 источников** на интеграцию
- **Максимум 40 значений пользовательских полей** при комплексном добавлении сделки
- **Максимум 50 воронок** на аккаунт
- **Максимум 100 этапов** в одной воронке
- **Максимум 100 вебхуков** на аккаунт
- **Максимум 10 списков** на аккаунт
- **10 ГБ** дисковое пространство для пробного аккаунта

## Основные методы API

### Аккаунт
```
GET /api/v4/account - Получение информации об аккаунте
```

### Сделки (Leads)
```
GET /api/v4/leads - Получение списка сделок
GET /api/v4/leads/{id} - Получение сделки по ID
POST /api/v4/leads - Создание сделок
PATCH /api/v4/leads/{id} - Обновление сделки
PATCH /api/v4/leads - Массовое обновление сделок
```

### Контакты (Contacts)
```
GET /api/v4/contacts - Получение списка контактов
GET /api/v4/contacts/{id} - Получение контакта по ID
POST /api/v4/contacts - Создание контактов
PATCH /api/v4/contacts/{id} - Обновление контакта
PATCH /api/v4/contacts - Массовое обновление контактов
```

### Компании (Companies)
```
GET /api/v4/companies - Получение списка компаний
GET /api/v4/companies/{id} - Получение компании по ID
POST /api/v4/companies - Создание компаний
PATCH /api/v4/companies/{id} - Обновление компании
PATCH /api/v4/companies - Массовое обновление компаний
```

### Задачи (Tasks)
```
GET /api/v4/tasks - Получение списка задач
GET /api/v4/tasks/{id} - Получение задачи по ID
POST /api/v4/tasks - Создание задач
PATCH /api/v4/tasks/{id} - Обновление задачи
PATCH /api/v4/tasks - Массовое обновление задач
```

### Примечания (Notes)
```
GET /api/v4/{entity_type}/{entity_id}/notes - Получение примечаний сущности
GET /api/v4/{entity_type}/{entity_id}/notes/{id} - Получение примечания по ID
POST /api/v4/{entity_type}/{entity_id}/notes - Создание примечаний
PATCH /api/v4/{entity_type}/{entity_id}/notes/{id} - Обновление примечания
```

### Вебхуки (Webhooks)
```
GET /api/v4/webhooks - Получение списка вебхуков
POST /api/v4/webhooks - Создание вебхука
DELETE /api/v4/webhooks/{id} - Удаление вебхука
```

### Воронки и этапы (Pipelines & Stages)
```
GET /api/v4/leads/pipelines - Получение воронок
GET /api/v4/leads/pipelines/{id} - Получение воронки по ID
POST /api/v4/leads/pipelines - Создание воронок
PATCH /api/v4/leads/pipelines/{id} - Обновление воронки
DELETE /api/v4/leads/pipelines/{id} - Удаление воронки
```

### Пользователи (Users)
```
GET /api/v4/users - Получение списка пользователей
GET /api/v4/users/{id} - Получение пользователя по ID
POST /api/v4/users - Создание пользователей
```

### Теги (Tags)
```
GET /api/v4/{entity_type}/tags - Получение тегов сущности
POST /api/v4/{entity_type}/tags - Создание тегов
PATCH /api/v4/{entity_type}/tags - Обновление тегов
```

### Пользовательские поля (Custom Fields)
```
GET /api/v4/{entity_type}/custom_fields - Получение пользовательских полей
GET /api/v4/{entity_type}/custom_fields/{id} - Получение поля по ID
POST /api/v4/{entity_type}/custom_fields - Создание полей
```

## Параметры запросов

### Общие параметры
- **page** - Номер страницы (по умолчанию 1)
- **limit** - Количество записей на странице (максимум 250)
- **with** - Включение связанных данных
- **query** - Поисковый запрос
- **filter** - Фильтрация данных

### Параметр "with"
Для контактов:
- `leads` - Включить связанные сделки
- `catalog_elements` - Включить связанные элементы списков

Для сделок:
- `contacts` - Включить связанные контакты
- `companies` - Включить связанные компании

### Фильтрация
```
GET /api/v4/leads?filter[created_at][from]=1609459200&filter[created_at][to]=1609545600
```

## Структура ответов

### Успешный ответ
```json
{
  "_page": 1,
  "_links": {
    "self": {
      "href": "https://subdomain.kommo.com/api/v4/leads?page=1"
    },
    "next": {
      "href": "https://subdomain.kommo.com/api/v4/leads?page=2"
    }
  },
  "_embedded": {
    "leads": [
      {
        "id": 123,
        "name": "Название сделки",
        "price": 50000,
        "status_id": 456,
        "pipeline_id": 789,
        "created_at": 1609459200,
        "updated_at": 1609459200
      }
    ]
  }
}
```

### Ошибка
```json
{
  "type": "https://httpstatus.es/400",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation error description"
}
```

## HTTP статус коды

- **200 OK** - Успешный запрос
- **201 Created** - Успешное создание
- **204 No Content** - Успешное удаление
- **400 Bad Request** - Ошибка валидации
- **401 Unauthorized** - Неавторизованный доступ
- **403 Forbidden** - IP заблокирован
- **404 Not Found** - Ресурс не найден
- **422 Unprocessable Entity** - Ошибка обработки данных
- **429 Too Many Requests** - Превышен лимит запросов
- **500 Internal Server Error** - Внутренняя ошибка сервера
- **504 Gateway Timeout** - Таймаут (уменьшите количество записей)

## Webhooks

### Поддерживаемые события
- **leads** - события сделок (add, update, delete, status, responsible)
- **contacts** - события контактов (add, update, delete, responsible)
- **companies** - события компаний (add, update, delete, responsible)
- **tasks** - события задач (add, update, delete, complete)
- **notes** - события примечаний (add, update, delete)

### Создание webhook
```json
POST /api/v4/webhooks
{
  "destination": "https://example.com/webhook",
  "settings": [
    "add_lead",
    "update_lead",
    "delete_lead"
  ]
}
```

### Структура webhook
```json
{
  "account_id": 12345,
  "timestamp": 1609459200,
  "leads": {
    "add": [
      {
        "id": 123,
        "name": "Новая сделка",
        "status_id": 456,
        "old_status_id": 0,
        "price": 50000,
        "responsible_user_id": 789,
        "last_modified": 1609459200,
        "modified_user_id": 789,
        "created_user_id": 789,
        "date_create": 1609459200,
        "pipeline_id": 321,
        "account_id": 12345
      }
    ]
  }
}
```

## Дополнительные возможности

### Chats API
Для интеграции с мессенджерами:
- Отправка и получение сообщений
- Управление чатами
- Статусы доставки
- Реакции на сообщения

### VoIP API
Для интеграции с телефонией:
- Управление звонками
- Запись разговоров
- Push-уведомления

### Kommo AI
Искусственный интеллект для:
- Автоматических ответов
- Анализа данных
- Рекомендаций

### Digital Pipeline
Автоматизация процессов:
- Автоматические действия
- Условная логика
- Интеграция с внешними системами

## Безопасность

### Рекомендации
1. Используйте HTTPS для всех запросов
2. Храните токены в безопасном месте
3. Регулярно обновляйте refresh token
4. Используйте валидацию webhook через HMAC

### Валидация webhook
```php
$signature = hash_hmac('sha256', $clientId . '|' . $accountId, $clientSecret);
if (!hash_equals($signature, $hookSignature)) {
    throw new Exception('Invalid webhook signature');
}
```

## Примеры использования

### Создание сделки
```json
POST /api/v4/leads
{
  "name": "Новая сделка",
  "price": 50000,
  "contacts": [
    {
      "id": 123
    }
  ],
  "custom_fields_values": [
    {
      "field_id": 456,
      "values": [
        {
          "value": "Дополнительная информация"
        }
      ]
    }
  ]
}
```

### Создание контакта
```json
POST /api/v4/contacts
{
  "name": "Иван Петров",
  "first_name": "Иван",
  "last_name": "Петров",
  "custom_fields_values": [
    {
      "field_id": 123,
      "values": [
        {
          "value": "+7 123 456 78 90"
        }
      ]
    }
  ]
}
```

### Создание задачи
```json
POST /api/v4/tasks
{
  "text": "Связаться с клиентом",
  "complete_till": 1609459200,
  "task_type_id": 1,
  "entity_id": 123,
  "entity_type": "leads"
}
```

## Полезные ссылки

- [Официальная документация Kommo API](https://developers.kommo.com/)
- [API Reference](https://developers.kommo.com/reference)
- [OAuth 2.0 документация](https://developers.kommo.com/docs/oauth-20)
- [Webhooks документация](https://developers.kommo.com/docs/webhooks-general)
- [Chats API](https://developers.kommo.com/reference/send-message-guide)
- [Discord сообщество разработчиков](https://discord.gg/CjstJTrBHu)
- [GitHub репозиторий](https://github.com/kommo-crm)

## Часто задаваемые вопросы

**Q: Как получить client_id и client_secret?**
A: Создайте интеграцию в настройках Kommo (Settings → Integrations → Create Integration)

**Q: Что делать при ошибке 429?**
A: Уменьшите частоту запросов, максимум 7 в секунду

**Q: Как работать с пагинацией?**
A: Используйте параметры page и limit, максимум 250 записей за запрос

**Q: Как получить ID пользовательских полей?**
A: Используйте GET /api/v4/{entity_type}/custom_fields

**Q: Как связать сделку с контактом?**
A: Укажите массив contacts с ID контактов при создании сделки

**Q: Срок действия токенов?**
A: Access token - 24 часа, Refresh token - 3 месяца

**Q: Как получить webhook события?**
A: Настройте webhook endpoint и укажите нужные события в settings

## Типы задач

### Стандартные типы задач
- **1** - Follow-up (Перезвонить)
- **2** - Meeting (Встреча)

### Типы примечаний
Kommo поддерживает 10 типов примечаний, которые можно редактировать:
- **common** - Обычное примечание
- **call_in** - Входящий звонок
- **call_out** - Исходящий звонок
- **service_message** - Системное сообщение
- **mail_message** - Email сообщение
- **sms_in** - Входящая SMS
- **sms_out** - Исходящая SMS
- **attachment** - Вложение
- **extended_service_message** - Расширенное системное сообщение
- **targeted_message** - Целевое сообщение

### Цвета этапов воронки
Kommo предоставляет предопределенные цвета для этапов воронки, которые можно использовать при создании и обновлении этапов.
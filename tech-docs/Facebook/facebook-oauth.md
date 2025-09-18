# Facebook OAuth Авторизация для Бизнес Приложений

## Обзор

Документация по интеграции Facebook OAuth для получения данных через Graph API и Marketing API для бизнес приложений.

## Основные этапы OAuth Flow

### 1. Проверка статуса входа

Приложения должны проверять, вошел ли пользователь, с помощью встроенных функций SDK или собственного способа сохранения сведений о входе пользователя.

### 2. Вызов диалога входа

Перенаправление на конечную точку для открытия диалога входа:

```
https://www.facebook.com/v23.0/dialog/oauth?
  client_id={app-id}
  &redirect_uri={redirect-uri}
  &state={state-param}
  &scope={разрешения}
  &response_type={code|token}
```

**Обязательные параметры:**
- `client_id` - ID приложения из панели приложений
- `redirect_uri` - URL для перенаправления после авторизации
- `state` - строка для защиты от CSRF атак

**Дополнительные параметры:**
- `response_type`:
  - `code` - для серверной обработки (по умолчанию)
  - `token` - для клиентской обработки
  - `code%20token` - возвращает и код, и токен
- `scope` - разделенный запятыми список разрешений
- `granted_scopes` - возвращает список предоставленных разрешений

### 3. Обработка ответа

**При успешной авторизации:**
- Для `response_type=code`: возвращается код авторизации в параметрах URL
- Для `response_type=token`: возвращается access_token во фрагменте URL

**При отмене:**
```
YOUR_REDIRECT_URI?
error_reason=user_denied
&error=access_denied
&error_description=Permissions+error
```

### 4. Обмен кода на маркер доступа

```
GET https://graph.facebook.com/v23.0/oauth/access_token?
   client_id={app-id}
   &redirect_uri={redirect-uri}
   &client_secret={app-secret}
   &code={code-parameter}
```

**Ответ:**
```json
{
  "access_token": "{access-token}", 
  "token_type": "{type}",
  "expires_in": "{seconds-til-expiration}"
}
```

## Типы маркеров доступа

### 1. Маркеры доступа пользователя

#### Краткосрочные маркеры
- Действуют 1-2 часа
- Создаются при веб-авторизации

#### Долгосрочные маркеры  
- Действуют ~60 дней
- Создаются в мобильных приложениях
- Можно получить из краткосрочных через API

#### Пример получения (JavaScript):
```javascript
FB.getLoginStatus(function(response) {
  if (response.status === 'connected') {
    var accessToken = response.authResponse.accessToken;
  } 
});
```

### 2. Маркеры доступа приложения

Для вызовов от имени приложения:

```bash
curl -X GET "https://graph.facebook.com/oauth/access_token
  ?client_id={your-app-id}
  &client_secret={your-app-secret}
  &grant_type=client_credentials"
```

Альтернативно:
```
access_token={app-id}|{app-secret}
```

### 3. Маркеры доступа к Страницам

Для управления Facebook страницами:

```bash
curl -i -X GET "https://graph.facebook.com/{user-id}/accounts?access_token={user-access-token}"
```

### 4. Маркеры системного пользователя

- Создаются в Meta Business Manager
- Не имеют срока действия
- Используются для межсерверного взаимодействия
- Подходят для долгосрочных скриптов

## Разрешения для бизнес приложений

### Основные разрешения:

1. **ads_read** - чтение рекламных данных
2. **ads_management** - управление рекламой
3. **business_management** - управление бизнес-ресурсами
4. **email** - доступ к email пользователя
5. **public_profile** - публичная информация профиля

### Уровни доступа:

1. **Стандартный доступ**
   - Ограниченный функционал
   - Не требует проверки приложения

2. **Расширенный доступ**
   - Полный функционал API
   - Требует проверку приложения
   - Требует бизнес-верификацию

### Пример запроса разрешений:

```
https://www.facebook.com/v23.0/dialog/oauth?
client_id=<YOUR_APP_ID>
&redirect_uri=<YOUR_URL>
&scope=ads_management,ads_read,business_management
```

## Marketing API

### Получение маркера через Graph API Explorer:

1. Выберите приложение в поле "Приложение Meta"
2. Выберите "Маркер пользователя" 
3. Добавьте разрешения (ads_read, ads_management)
4. Нажмите "Создать маркер доступа"

### Продление маркера:

1. Используйте отладчик маркеров доступа
2. Нажмите "Продлить срок действия маркера доступа"
3. Получите долгосрочный маркер

### Требования для Marketing API:

- Активный рекламный аккаунт
- Зарегистрированный аккаунт разработчика Meta
- Приложение Facebook
- Соответствующие разрешения

## Проверка маркеров

### Отладка маркера:

```
GET graph.facebook.com/debug_token?
     input_token={token-to-inspect}
     &access_token={app-token-or-admin-token}
```

**Ответ:**
```json
{
    "data": {
        "app_id": 138483919580948, 
        "type": "USER",
        "application": "Social Cafe", 
        "expires_at": 1352419328, 
        "is_valid": true, 
        "issued_at": 1347235328, 
        "scopes": [
            "email", 
            "ads_management"
        ], 
        "user_id": "1207059"
    }
}
```

### Проверка разрешений:

```
GET https://graph.facebook.com/me/permissions?access_token={user-access-token}
```

## Рекомендации по безопасности

1. **Используйте HTTPS** для всех передач маркеров
2. **Храните маркеры безопасно** в зашифрованных базах данных
3. **Ограничивайте область действия** - запрашивайте только нужные разрешения
4. **Регулярно обновляйте маркеры** и обрабатывайте истечение срока действия
5. **Никогда не встраивайте секрет приложения** в клиентский код
6. **Используйте параметр state** для защиты от CSRF

## Обработка ошибок

### Основные коды ошибок:

- `user_denied` - пользователь отклонил разрешения
- `access_denied` - доступ запрещен
- `invalid_request` - неверные параметры запроса
- `invalid_token` - недействительный маркер

### Повторный запрос отклоненных разрешений:

```
https://www.facebook.com/v23.0/dialog/oauth?
    client_id={app-id}
    &redirect_uri={redirect-uri}
    &auth_type=rerequest
    &scope=email
```

## Практические примеры

### Создание приложения:

1. Перейти в [Facebook Developers](https://developers.facebook.com/)
2. Создать новое приложение
3. Выбрать тип "Бизнес"
4. Настроить OAuth redirect URIs
5. Получить App ID и App Secret

### Базовый флоу авторизации:

```javascript
// 1. Перенаправление на Facebook
const authUrl = `https://www.facebook.com/v23.0/dialog/oauth?client_id=${APP_ID}&redirect_uri=${REDIRECT_URI}&scope=ads_read,ads_management&response_type=code&state=${STATE}`;
window.location.href = authUrl;

// 2. Обработка callback
const urlParams = new URLSearchParams(window.location.search);
const code = urlParams.get('code');
const state = urlParams.get('state');

// 3. Обмен кода на токен (серверная сторона)
const tokenResponse = await fetch(`https://graph.facebook.com/v23.0/oauth/access_token?client_id=${APP_ID}&redirect_uri=${REDIRECT_URI}&client_secret=${APP_SECRET}&code=${code}`);
const tokenData = await tokenResponse.json();
```

### Работа с Marketing API:

```javascript
// Получение рекламных аккаунтов
const adAccounts = await fetch(`https://graph.facebook.com/v23.0/me/adaccounts?access_token=${ACCESS_TOKEN}`);

// Получение кампаний
const campaigns = await fetch(`https://graph.facebook.com/v23.0/act_${AD_ACCOUNT_ID}/campaigns?access_token=${ACCESS_TOKEN}`);

// Получение статистики
const insights = await fetch(`https://graph.facebook.com/v23.0/act_${AD_ACCOUNT_ID}/insights?access_token=${ACCESS_TOKEN}`);
```

## Полезные ссылки

- [Facebook Developers Console](https://developers.facebook.com/)
- [Graph API Explorer](https://developers.facebook.com/tools/explorer)
- [Access Token Debugger](https://developers.facebook.com/tools/debug/accesstoken)
- [Business Manager](https://business.facebook.com/)
- [Meta for Developers Documentation](https://developers.facebook.com/docs/)

## Webhooks

### Настройка Webhooks

Webhooks позволяют получать уведомления HTTP в реальном времени об изменениях объектов в социальном графе Meta.

#### Требования:
- HTTPS сервер с действительным TLS/SSL сертификатом
- Самозаверенные сертификаты не поддерживаются
- Обработка HTTP POST запросов с JSON payload

#### Пример уведомления:
```json
{
  "entry": [
    {
      "time": 1520383571,
      "changes": [
        {
          "field": "photos",
          "value": {
            "verb": "update",
            "object_id": "10211885744794461"
          }
        }
      ],
      "id": "10210299214172187",
      "uid": "10210299214172187"
    }
  ],
  "object": "user"
}
```

#### Настройка:
1. Добавить продукт Webhooks в панели приложения
2. Настроить callback URL (HTTPS)
3. Выбрать объекты и поля для подписки
4. Проверить webhook endpoint

#### Объекты для подписки:
- `user` - пользователи
- `page` - страницы Facebook
- `application` - приложения
- `permissions` - изменения разрешений
- `payments` - платежи

## Системные пользователи

### Что такое системные пользователи

Системные пользователи позволяют автоматизировать действия с рекламными объектами и страницами программным способом через серверы или ПО.

#### Преимущества:
- **Нет срока действия** - маркеры не истекают
- **Межсерверное взаимодействие** - подходят для автоматизации
- **Стабильность** - меньше вероятность аннулирования
- **Разделение ответственности** - отдельно от личных аккаунтов

#### Создание системного пользователя:
1. Открыть Business Manager
2. Бизнес-настройки → Пользователи → Системные пользователи
3. Добавить новый системный пользователь
4. Назначить разрешения для рекламных аккаунтов
5. Сгенерировать маркер доступа

#### Пример генерации маркера:
```bash
curl -X POST \
"https://graph.facebook.com/v23.0/{business-id}/system_users" \
-d "name=MySystemUser" \
-d "role=ADMIN" \
-d "access_token={business-manager-token}"
```

## Бизнес-верификация и App Review

### Бизнес-верификация

Обязательна для получения расширенного доступа к разрешениям с 1 февраля 2023 года.

#### Процесс верификации:
1. **Подключение к компании**
   - Панель приложений → Настройки → Основные → Подтверждение
   - Связать с существующей компанией или создать новую

2. **Подтверждение компании**
   - Проходится в Business Manager
   - Требуются документы компании
   - Проверка юридического лица

#### Документы для верификации:
- Документы о регистрации компании
- Налоговые документы
- Подтверждение адреса
- Удостоверение личности представителя

### App Review (Проверка приложения)

Необходима для получения доступа к большинству разрешений для внешних пользователей.

#### Когда требуется:
- Приложение используется пользователями без роли в приложении
- Запрос расширенного доступа к разрешениям
- Доступ к конфиденциальным данным пользователей

#### Процесс подачи:
1. Подготовить демонстрацию функционала
2. Описать сценарий использования разрешений
3. Предоставить тестовые данные
4. Пройти техническую проверку
5. Соответствие политикам Meta

## Ограничения Rate Limiting

### Ограничения платформы

#### Для пользовательских маркеров:
- Не публикуется точное количество (конфиденциальность)
- Распределяется между всеми приложениями пользователя

#### Для маркеров приложения:
```
Calls per hour = 200 * Number of Users
```

### Ограничения бизнес-приложений

#### Marketing API (Ads Management):
**Стандартный доступ:**
```
Calls per hour = 300 + 40 * Number of Active Ads
```

**Расширенный доступ:**
```
Calls per hour = 100,000 + 40 * Number of Active Ads
```

#### Ads Insights:
**Стандартный доступ:**
```
Calls per hour = 600 + 400 * Number of Active Ads - 0.001 * User Errors
```

**Расширенный доступ:**
```
Calls per hour = 190,000 + 400 * Number of Active Ads - 0.001 * User Errors
```

#### Custom Audiences:
**Стандартный доступ:**
```
Calls per hour = 5,000 + 40 * Number of Active Custom Audiences
```

**Расширенный доступ:**
```
Calls per hour = 190,000 + 40 * Number of Active Custom Audiences
```

### Мониторинг ограничений

#### HTTP Headers для отслеживания:
```json
// Платформенные ограничения
x-app-usage: {
  "call_count": 28,
  "total_time": 25,
  "total_cputime": 25
}

// Бизнес ограничения
x-business-use-case-usage: {
  "business-object-id": [{
    "type": "ads_management",
    "call_count": 95,
    "total_cputime": 20,
    "total_time": 20,
    "estimated_time_to_regain_access": 19,
    "ads_api_access_tier": "standard_access"
  }]
}
```

### Рекомендации по оптимизации:

1. **Распределение запросов** - равномерно во времени
2. **Фильтрация данных** - используйте поля и фильтры для уменьшения ответов
3. **Мониторинг заголовков** - отслеживайте X-App-Usage и X-Business-Use-Case-Usage
4. **Обработка ошибок** - корректно обрабатывайте коды 4, 17, 32, 613
5. **Batch запросы** - объединяйте множественные запросы в один

## Дополнительные API

### WhatsApp Business API

Ограничения:
- 200 вызовов/час по умолчанию на аккаунт
- 5,000 вызовов/час для активных аккаунтов
- Используйте Webhooks для мониторинга статусов

### Pages API

Для маркеров страниц:
```
Calls per 24 hours = 4800 * Number of Engaged Users
```

## Продвинутые сценарии

### Многотенантные приложения

1. **Используйте системных пользователей** для каждого клиента
2. **Разделяйте Business Manager** для изоляции данных
3. **Мониторьте rate limits** отдельно по каждому tenant
4. **Настройте webhooks** для каждого клиента отдельно

### Масштабирование

1. **App Clustering** - используйте несколько App ID для увеличения лимитов
2. **Data Sharding** - разделите данные по регионам/клиентам
3. **Caching Strategy** - кешируйте неизменяемые данные
4. **Queue Management** - очереди для обработки массовых операций

### Мониторинг и алертинг

```javascript
// Пример мониторинга rate limits
function checkRateLimit(response) {
  const usage = response.headers['x-app-usage'];
  if (usage) {
    const usageData = JSON.parse(usage);
    if (usageData.call_count > 80) {
      // Предупреждение о приближении к лимиту
      console.warn('Approaching rate limit:', usageData.call_count + '%');
    }
  }
}

// Пример обработки ошибок rate limiting
async function handleRateLimit(error) {
  if (error.code === 4 || error.code === 17 || error.code === 32) {
    const retryAfter = error.headers['retry-after'] || 3600; // 1 час по умолчанию
    console.log(`Rate limited. Retry after ${retryAfter} seconds`);
    await sleep(retryAfter * 1000);
    return true; // можно повторить запрос
  }
  return false;
}
```

## Troubleshooting

### Частые ошибки

**OAuth ошибки:**
- `invalid_client` - неверный App ID или Secret
- `invalid_grant` - истекший или неверный authorization code
- `invalid_scope` - запрошенное разрешение недоступно

**Rate Limiting ошибки:**
- Code 4: App level rate limiting
- Code 17: User level rate limiting  
- Code 32: Page level rate limiting
- Code 613: Business level rate limiting

**Permissions ошибки:**
- `insufficient_scope` - недостаточные разрешения
- `user_access_token_required` - нужен пользовательский токен
- `business_verification_required` - требуется бизнес-верификация

### Отладка

1. **Access Token Debugger** - проверка валидности и разрешений токенов
2. **Graph API Explorer** - тестирование запросов
3. **App Dashboard** - мониторинг метрик и ошибок
4. **Webhook Debugger** - тестирование webhook endpoints

## Соответствие требованиям

### GDPR и конфиденциальность

1. **Минимальные разрешения** - запрашивайте только необходимые
2. **Удаление данных** - реализуйте обработку запросов на удаление
3. **Прозрачность** - четко объясняйте использование данных
4. **Согласие пользователей** - получайте явное согласие

### Политики платформы

1. **Не спамьте** - соблюдайте частоту публикаций
2. **Качественный контент** - публикуйте релевантную информацию
3. **Безопасность** - защищайте маркеры и секреты
4. **Актуальность** - регулярно обновляйте разрешения

## Заключение

Facebook OAuth предоставляет мощные возможности для интеграции с бизнес-данными через Graph API и Marketing API. Правильная реализация авторизации, работы с маркерами доступа, соблюдение rate limits и требований безопасности критически важны для стабильной работы приложения.

Ключевые моменты для успешной интеграции:
- Используйте системных пользователей для автоматизации
- Проходите бизнес-верификацию для расширенного доступа
- Мониторьте rate limits и оптимизируйте запросы
- Настройте webhooks для real-time уведомлений
- Следуйте best practices по безопасности и privacy

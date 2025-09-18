# 📱 WhatsApp Subscriber Data Structure V2

> **📎 Связанные документы:**
> - [Webhook Payload Specification](../incoming-webhooks/webhook-payload-specification.md) - структура входящих webhook
> - Этот документ описывает финальную структуру данных подписчика в БД

---

## ✅ БАЗОВЫЕ ДАННЫЕ (из первого webhook)

Эти поля заполняются при создании записи подписчика из первого входящего сообщения.

### 🆔 Идентификация

**`id`** - UUID подписчика в нашей системе
```json
"550e8400-e29b-41d4-a716-446655440000"
```

**`tenantId`** - ID арендатора (мультитенантность)
```json
"tenant_12345"
```

**`waPhone`** - WhatsApp номер в формате E.164
```json
"+15551234567"  // США
"+380671234567" // Украина  
"+447911123456" // Великобритания
```

**`waId`** - Оригинальный ID от провайдера (Cloud API/Gupshup)
```json
"15551234567"     // Cloud API format
"15551234567@s.whatsapp.net"  // Gupshup format
```

**`phoneHash`** - SHA256 хэш номера для Facebook CAPI и безопасного поиска
```json
"a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3"
```

> 💡 **Маппинг из webhook:** Детали извлечения полей из различных провайдеров см. в [webhook-payload-specification.md](../incoming-webhooks/webhook-payload-specification.md#field-mapping)

---

### 👤 Профиль WhatsApp

**`profileName`**  
Имя профиля пользователя в WhatsApp (может отсутствовать из-за privacy)  
```
"John Doe"
"Maria 🌸"  
"+1 555 123-4567"  // Иногда вместо имени номер
null              // Если скрыто настройками
```

---

### 💬 Сессия и разговор

**`conversationId`**  
Уникальный ID разговора для группировки сообщений в 24-часовой сессии  
```
"gBEGkYiEB1VXAglK1ZEqA1YKPrU"
```
⚡ Используется для тарификации WhatsApp Business

**`conversationExpiresAt`**  
Время окончания бесплатного 24-часового окна (Unix timestamp в мс)  
```
1736539200000  // = 10 Jan 2025 12:00:00 UTC
null           // Если окно активно
```
💡 После истечения нужны template messages

---

### 📊 Статусы и активность

**`lastMessageStatus`**  
Последний статус сообщения  
```
"received"   // Входящее от пользователя (по умолчанию для первого)
"sent"       // Исходящее отправлено
"delivered"  // Доставлено на устройство
"read"       // Прочитано пользователем  
"failed"     // Ошибка доставки
```

**`lastMessageStatusAt`**  
Время последнего события (Unix timestamp в мс)  
```
1736535600000  // = 10 Jan 2025 11:00:00 UTC
```

**`firstSeenAt`**  
Первое появление пользователя в системе  
```
1736535600000
```

**`lastInboundAt`**  
Последнее входящее сообщение от пользователя  
```
1736535600000
```

**`inboundCount`**  
Счётчик входящих сообщений (начинается с 1)  
```
1
```

---

### ⚙️ Технические поля

**`status`**  
Статус подписчика в нашей системе  
```
"active"     // По умолчанию при создании
"inactive"   // Долго не писал
"blocked"    // Заблокирован нами
"opted_out"  // Отписался через STOP
```

**`rawLastEventType`**  
Тип последнего webhook события от провайдера  
```
"message"        // Обычное сообщение
"message-event"  // Статус доставки  
"user-event"     // Opt-in/opt-out
"sandbox-start"  // Начало тестового режима
```

**`rawVersion`**  
Версия API провайдера  
```
"v3"  // Текущая версия
"v2"  // Legacy
```

---

## 🎯 ЕСЛИ ПРИШЁЛ ЧЕРЕЗ РЕКЛАМУ (referral блок)

Эти поля заполняются только если пользователь пришёл по рекламной ссылке Facebook/Instagram.

> 💡 **Полная спецификация referral блока** в webhook см. [webhook-payload-specification.md](../incoming-webhooks/webhook-payload-specification.md#referral)

### Атрибуция Click-to-WhatsApp

**`ctwaClid`**  
Facebook Click ID для трекинга конверсий  
```
"fb.1.1736535600.IwAR123abcDEF456ghiJKL789mno"
```
🎯 Критически важно для Facebook CAPI и атрибуции

**`referral.sourceUrl`**  
Полный URL источника перехода  
```
"https://m.me/yourbusiness?ref=ad_campaign"
"https://wa.me/15551234567?text=hello"  
"https://business.facebook.com/latest/inbox/all?asset_id=123456"
```

**`referral.sourceType`**  
Тип источника трафика  
```
"ad"       // Рекламное объявление
"post"     // Органический пост
"page"     // Страница бизнеса
"website"  // Внешний сайт
```

**`referral.sourceId`**  
Facebook ID источника (объявление/пост/страница)  
```
"120210000000000000"  // ID рекламы
"100044123456789"     // ID поста  
"108123456789012345"  // ID страницы
```

**`referral.headline`**  
Заголовок рекламного объявления  
```
"Начни чат с нами"
"Скидка 50% только сегодня"
"Бесплатная консультация эксперта"
```

**`referral.body`**  
Текст описания в объявлении  
```
"Получи скидку 20% на первую покупку"
"Напиши нам в WhatsApp и узнай подробности"
"Ответим на все вопросы за 5 минут"
```

**`referral.mediaType`**  
Тип медиа в рекламе  
```
"image"     // Статичная картинка
"video"     // Видео
"carousel"  // Карусель изображений
```

**`referral.imageUrl`**  
URL превью изображения (если тип = image)  
```
"https://scontent.xx.fbcdn.net/v/t45.1600-4/123456_789012345_n.jpg"
```

---

## 📋 МИНИМАЛЬНЫЙ JSON (первое сообщение)

### Обычный пользователь (органика)
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "tenantId": "tenant_12345",
  "waPhone": "+15551234567",
  "phoneHash": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3",
  "profileName": "John Doe",
  "conversationId": "gBEGkYiEB1VXAglK1ZEqA1YKPrU",
  "conversationExpiresAt": 1736539200000,
  "lastMessageStatus": "received",
  "lastMessageStatusAt": 1736535600000,
  "firstSeenAt": 1736535600000,
  "lastInboundAt": 1736535600000,
  "inboundCount": 1,
  "status": "active",
  "rawLastEventType": "message",
  "rawVersion": "v3"
}
```

### Пользователь из рекламы (с referral)
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "tenantId": "tenant_12345",
  "waPhone": "+15551234567",
  "phoneHash": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3",
  "profileName": "Maria Garcia",
  "conversationId": "gBEGkYiEB1VXAglK1ZEqA1YKPrU",
  "conversationExpiresAt": 1736539200000,
  "lastMessageStatus": "received",
  "lastMessageStatusAt": 1736535600000,
  "firstSeenAt": 1736535600000,
  "lastInboundAt": 1736535600000,
  "inboundCount": 1,
  "status": "active",
  "rawLastEventType": "message",
  "rawVersion": "v3",
  "ctwaClid": "fb.1.1736535600.IwAR123abcDEF456ghiJKL789mno",
  "referral": {
    "sourceUrl": "https://m.me/yourbusiness?ref=summer_sale",
    "sourceType": "ad",
    "sourceId": "120210000000000000",
    "headline": "Скидка 50% только сегодня",
    "body": "Напиши нам в WhatsApp и получи промокод",
    "mediaType": "video",
    "imageUrl": null
  }
}
```

---

## ⏱️ ПОЯВИТСЯ ПОЗЖЕ (требует времени или событий)

### После отправки нами сообщений
- **`lastOutboundMessageId`** — ID последнего исходящего (`gsId` или `wamid`)
- **`lastOutboundAt`** — время последней отправки
- **`outboundCount`** — счётчик отправленных сообщений

### При статусных событиях
- **`lastDeliveredAt`** — когда доставлено (status=delivered)
- **`lastReadAt`** — когда прочитано (status=read)
- **`lastFailedAt`** — время последней ошибки
- **`lastFailedCode`** — код ошибки (470, 1002, etc)
- **`lastFailedReason`** — описание ошибки

### User events
- **`optInStatus`** — true/false (после opted-in/opted-out события)
- **`optInAt`** — время получения согласия
- **`optOutAt`** — время отписки

### Lifecycle (бизнес-логика)
- **`lifecycleState`** — lead → prospect → customer → repeat_customer
- **`lifecycleChangedAt`** — время изменения стадии
- **`lifecycleScore`** — оценка качества лида (0-100)

---

## 🔄 ОБОГАЩЕНИЕ (внешние источники)

### 🌍 Геолокация (IP lookup / phone parsing)
- **`city`** — "New York"
- **`state`** — "NY"
- **`country`** — "US"
- **`timezone`** — "America/New_York"
- **`postalCode`** — "10001"

### 🎯 UTM метки (парсинг referral.sourceUrl)
- **`utmSource`** — "facebook"
- **`utmMedium`** — "cpc"
- **`utmCampaign`** — "summer_sale_2025"
- **`utmContent`** — "video_testimonial"
- **`utmTerm`** — "discount_whatsapp"

### 📊 Facebook Ads (Marketing API по ctwaClid)
- **`campaignId`** — "23850123456789012"
- **`campaignName`** — "Summer Sale 2025"
- **`adsetId`** — "23850123456789013"
- **`adsetName`** — "Lookalike 1% Purchasers"
- **`adId`** — "23850123456789014"
- **`adName`** — "Video - 50% Off"
- **`creativeId`** — "23850123456789015"
- **`creativeName`** — "Summer Promo Video v3"
- **`placement`** — "instagram_stories"
- **`publisherPlatform`** — "instagram"

### 💰 Commerce (Stripe API/webhooks)
- **`stripeCustomerId`** — "cus_P1234567890"
- **`totalRevenue`** — 1250.00
- **`purchaseCount`** — 3
- **`averageOrderValue`** — 416.67
- **`lifetimeValue`** — 1250.00
- **`firstPurchaseAt`** — timestamp
- **`lastPurchaseAt`** — timestamp
- **`subscriptions[]`** — массив подписок

### 🤝 CRM (Kommo/HubSpot/Salesforce)
- **`crmContactId`** — внешний ID в CRM
- **`crmLeadId`** — ID сделки/лида
- **`leadScore`** — 85 (scoring модель)
- **`crmStage`** — "qualified"
- **`responsibleManager`** — "user_123"
- **`customFields`** — произвольные поля CRM

### 📈 Поведенческие метрики (внутренняя аналитика)
- **`totalSessions`** — количество сессий
- **`totalSessionDuration`** — общее время в секундах
- **`averageEngagementScore`** — средний engagement (0-10)
- **`lastSessionId`** — ID последней сессии
- **`lastSessionAt`** — время последней сессии
- **`daysSinceLastActivity`** — дней с последней активности
- **`conversionProbability`** — вероятность конверсии (0-100%)

### 👥 Facebook данные (Graph API)
- **`facebookUserId`** — глобальный Facebook ID
- **`facebookPageScopedUserId`** — ID в контексте страницы
- **`facebookAudiences[]`** — список Custom Audiences
- **`lookalikesAudiences[]`** — список Lookalike аудиторий

### 📝 Профильные данные (формы/импорт)
- **`firstName`** — "John"
- **`lastName`** — "Doe"
- **`fullName`** — "John Doe"
- **`email`** — "john.doe@example.com"
- **`emailHash`** — SHA256 для CAPI
- **`language`** — "en"
- **`profilePictureUrl`** — URL аватара

### 🏷️ Дополнительные атрибуты
- **`contactUuid`** — главный UUID для cross-system аналитики
- **`externalId`** — внешний идентификатор
- **`tags[]`** — массив тегов ["vip", "high_value"]
- **`customFields`** — произвольные поля
- **`metadata`** — служебные данные
- **`preferredChannel`** — предпочитаемый канал связи
- **`channels[]`** — все каналы [{type: "whatsapp", verified: true}]

### 📊 История и конверсии
- **`conversions[]`** — массив всех конверсий с детальной атрибуцией
- **`lifecycleHistory[]`** — история изменений состояний
- **`interactions[]`** — история взаимодействий

### ✅ Флаги обогащения
- **`gupshupEnriched`** — данные из Gupshup получены
- **`stripeEnriched`** — данные Stripe синхронизированы
- **`kommoEnriched`** — CRM данные загружены
- **`facebookSynced`** — Facebook данные актуальны
- **`isOptedIn`** — согласие на рассылки
- **`isBot`** — флаг бота (false для людей)

---

## 🚀 АВТОМАТИЧЕСКИЕ ПРОЦЕССЫ

После сохранения первого сообщения автоматически запускаются:

| Процесс           | Когда                | Что делает                                      | Приоритет |
|-------------------|----------------------|-------------------------------------------------|-----------|
| **CAPI Event**    | Сразу                | Отправляет "Contact" событие в Facebook         | P0        |
| **Audience Add**  | 1-2 сек              | Добавляет в Custom Audience "All WhatsApp Users" | P0        |
| **CRM Create**    | 2-5 сек              | Создаёт контакт в CRM если не существует        | P1        |
| **Enrichment**    | 5-10 сек             | Запускает обогащение профиля                   | P1        |
| **Analytics**     | 10-30 сек            | Обновляет метрики и дашборды                   | P2        |
| **Attribution**   | Если есть ctwaClid   | Загружает данные кампании из Facebook          | P0        |
| **Stripe Check**  | 30-60 сек            | Проверяет существующие платежи                 | P2        |

---

## 📚 СПРАВОЧНЫЕ ССЫЛКИ

- **Gupshup Webhooks:** https://docs.gupshup.io/docs/inbound-messages-events
- **Facebook CTWA:** https://developers.facebook.com/docs/whatsapp/conversation-types/
- **Facebook CAPI:** https://developers.facebook.com/docs/marketing-api/conversions-api/
- **Facebook Audiences:** https://developers.facebook.com/docs/marketing-api/audiences/
- **Stripe Webhooks:** https://stripe.com/docs/webhooks
- **Kommo API:** https://developers.kommo.com/docs/kommo-for-developers
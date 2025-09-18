# Facebook Conversions API (CAPI) - Полная документация

## Обзор

**Conversions API** — это инструмент Meta Business, который позволяет создавать связь между маркетинговыми данными рекламодателя (такими как события на сайте, в приложении, события переписки с компанией и офлайн-конверсии) с сервера рекламодателя, платформы сайта, из мобильного приложения или CRM с системами Meta. Это помогает оптимизировать таргетинг рекламы, снизить цену за результат и измерить результаты.

### Основные возможности:
- Отправка событий с сервера напрямую в Meta
- Улучшение атрибуции и качества данных
- Оптимизация рекламных кампаний
- Создание пользовательских аудиторий
- Измерение эффективности рекламы

## Начало работы

### Рекомендуемые действия:

1. **Начало работы**: выберите оптимальный способ интеграции, ознакомьтесь с предварительными требованиями для использования API
2. **Реализация API и отправка запросов**: узнайте, как выполнять запросы POST и что такое отброшенные события, пакетные запросы и время транзакции
3. **Подтверждение конфигурации**: убедитесь, что получаете соответствующие события, правильно выполняете их дедупликацию и сопоставление

## Основные параметры

### Параметры основного тела запроса

- **data** - массив событий для отправки
- **test_event_code** - код для тестирования событий

### Параметры информации о клиентах (User Data)

**Обязательные к хэшированию:**
- **em** - электронный адрес
- **ph** - номер телефона  
- **fn** - имя
- **ln** - фамилия
- **ge** - пол
- **db** - дата рождения
- **ct** - город
- **st** - штат, провинция или область
- **zp** - индекс
- **country** - страна
- **external_id** - внешний ID (рекомендуется хэширование)

**Не хэшируемые:**
- **client_ip_address** - IP-адрес клиента
- **client_user_agent** - пользовательский агент клиента
- **fbc** - ID клика Facebook
- **fbp** - ID браузера Facebook
- **subscription_id** - ID подписки
- **fb_login_id** - ID входа через Facebook
- **lead_id** - ID лида
- **anon_id** - ID установки (только для событий в приложении)
- **madid** - ID рекламодателя на мобильном устройстве (только для событий в приложении)

### Параметры для Business Messaging (WhatsApp)

- **ctwa_clid** - ID клика с переходом в WhatsApp
- **whatsapp_business_account_id** - ID бизнес-аккаунта WhatsApp

### Параметры серверных событий

- **event_name** - название события
- **event_time** - время события (Unix timestamp)
- **user_data** - данные пользователя
- **custom_data** - дополнительные данные события
- **event_source_url** - URL источника события
- **opt_out** - отказ от отслеживания
- **event_id** - уникальный ID события
- **action_source** - источник действия
- **data_processing_options** - параметры обработки данных
- **referrer_url** - URL источника перехода
- **customer_segmentation** - сегментация клиентов

## Conversions API для WhatsApp Business Messaging

### Интеграция с WhatsApp Business

#### Предварительные требования:

1. **Создание приложения Facebook для разработчиков**
2. **Интеграция с WhatsApp Business Platform**:
   - Cloud API (рекомендуется)
   - On-Premises API (версия 2.45.1+)
3. **Получение расширенного доступа к разрешениям**:
   - `whatsapp_business_management`
   - `whatsapp_business_manage_events`
   - "Ads Management Standard Access"

#### Шаги интеграции:

1. **Получение Access Token**
   - Используйте токен с необходимыми разрешениями
   - Можно использовать токен из Embedded Signup

2. **Получение WhatsApp Business Account ID**
   - Можно получить после завершения Embedded Signup

3. **Настройка Dataset API**
   ```bash
   POST https://graph.facebook.com/v16.0/{WHATSAPP_BUSINESS_ACCOUNT_ID}/dataset?access_token={TOKEN}
   ```

4. **Получение Click-to-WhatsApp Click ID (ctwa_clid)**
   - Получается из объекта referral в webhooks сообщений
   - Уникальный идентификатор для каждого клика

#### Пример webhook с ctwa_clid:
```json
{
  "data": [
    {
      "contacts": [
        {
          "profile": {
            "name": "Kerry Fisher"
          },
          "wa_id": "16315551234"
        }
      ],
      "messages": [
        {
          "from": "12345678",
          "id": "ABGGFlA5FpafAgo6tHcNmNjXmuSf",
          "referral": {
            "body": "This is a great product",
            "ctwa_clid": "ARAkLkA8rmlFeiCktEJQ-QTwRiyYHAFDLMNDBH0CD3qpjd0HR4irJ6LEkR7JwFF4XvnO2E4Nx0-eM-GABDLOPaOdRMv-_zfUQ2a",
            "headline": "Our new product",
            "source_id": "1234567890",
            "source_type": "ad",
            "source_url": "https://fb.me/AAAAA"
          },
          "text": {
            "body": "Can I learn more about your business?"
          },
          "timestamp": "1678189586",
          "type": "text"
        }
      ]
    }
  ]
}
```

#### Отправка событий через Conversions API:

```bash
POST https://graph.facebook.com/v16.0/{DATASET_ID}/events?access_token={TOKEN}
```

**Пример события покупки для WhatsApp:**
```json
{
  "data": [
    {
      "event_name": "Purchase",
      "event_time": 1675999999,
      "action_source": "business_messaging",
      "messaging_channel": "whatsapp",
      "user_data": {
        "whatsapp_business_account_id": "WHATSAPP_BUSINESS_ACCOUNT_ID",
        "ctwa_clid": "ARAkLkA8rmlFeiCktEJQ-QTwRiyYHAFDLMNDBH0CD3qpjd0HR4irJ6LEkR7JwFF4XvnO2E4Nx0-eM-GABDLOPaOdRMv-_zfUQ2a"
      },
      "custom_data": {
        "currency": "USD",
        "value": 123
      }
    }
  ],
  "partner_agent": "PARTNER_NAME"
}
```

## Поддерживаемые события для WhatsApp Business Messaging

Conversions API для WhatsApp Business Messaging поддерживает следующие типы событий:

- **Purchase** - покупка
- **LeadSubmitted** - отправка лида
- **InitiateCheckout** - начало оформления заказа
- **AddToCart** - добавление в корзину
- **ViewContent** - просмотр контента
- **OrderCreated** - создание заказа
- **OrderShipped** - отправка заказа
- **OrderDelivered** - доставка заказа
- **OrderCanceled** - отмена заказа
- **OrderReturned** - возврат заказа
- **CartAbandoned** - брошенная корзина
- **QualifiedLead** - квалифицированный лид
- **RatingProvided** - предоставление рейтинга
- **ReviewProvided** - предоставление отзыва

## Стандартные события пикселя Meta

### Основные события:

1. **Purchase** - покупка товаров или услуг
2. **ViewContent** - просмотр содержимого страницы
3. **AddToCart** - добавление товара в корзину
4. **InitiateCheckout** - начало процесса оформления заказа
5. **AddPaymentInfo** - добавление платежной информации
6. **Lead** - отправка информации с намерением рассмотрения товара или услуги
7. **CompleteRegistration** - завершение регистрации
8. **Contact** - телефонный звонок, отправка SMS, email или другой контакт
9. **CustomizeProduct** - настройка товара
10. **Donate** - пожертвование на благотворительность
11. **FindLocation** - поиск локации
12. **Schedule** - планирование встречи
13. **Search** - поиск на сайте
14. **StartTrial** - начало пробного периода
15. **SubmitApplication** - отправка заявления
16. **Subscribe** - подписка на платные товары или услуги
17. **AdImpression** - показ рекламы
18. **AdClick** - клик по рекламе

### Отслеживание стандартных событий:

```javascript
// Базовое событие
fbq('track', 'Purchase', {
  currency: "USD", 
  value: 30.00
});

// Событие с дополнительными параметрами
fbq('track', 'Purchase', {
  value: 115.00,
  currency: 'USD',
  contents: [
    {
      id: '301',
      quantity: 1
    },
    {
      id: '401', 
      quantity: 2
    }
  ],
  content_type: 'product'
});
```

## Специально настроенные события

Если стандартных событий недостаточно, можно создавать специально настроенные события:

```javascript
fbq('trackCustom', 'ShareDiscount', {
  promotion: 'share_discount_10%'
});
```

**Ограничения:**
- Имя события до 50 символов
- Используются для создания пользовательских аудиторий
- Поддерживают параметры как стандартные события

## Параметры событий

### Свойства объектов (стандартные параметры):

- **value** - значение/стоимость
- **currency** - валюта
- **content_type** - тип контента
- **content_ids** - массив ID контента
- **contents** - массив объектов контента
- **content_name** - название контента
- **content_category** - категория контента
- **num_items** - количество элементов
- **search_string** - строка поиска
- **status** - статус
- **delivery_category** - категория доставки

### Пользовательские свойства:

Можно добавлять собственные свойства для более детального анализа:

```javascript
fbq('track', 'Purchase', {
  value: 115.00,
  currency: 'USD',
  contents: [...],
  content_type: 'product',
  compared_product: 'recommended-banner-shoes', // пользовательское свойство
  delivery_category: 'in_store'
});
```

## WhatsApp Business API - Webhooks

### Общий формат webhook:

```json
{
  "object": "whatsapp_business_account",
  "entry": [{
    "id": "WHATSAPP_BUSINESS_ACCOUNT_ID",
    "changes": [{
      "value": {
        "messaging_product": "whatsapp",
        "metadata": {
          "display_phone_number": "PHONE_NUMBER",
          "phone_number_id": "PHONE_NUMBER_ID"
        }
        // специфичные данные webhook
      },
      "field": "messages"
    }]
  }]
}
```

### Типы уведомлений:

1. **Полученные сообщения (входящие)**:
   - Уведомления о новых сообщениях от пользователей
   - Содержат информацию о ctwa_clid для отслеживания

2. **Уведомления о статусе (исходящие)**:
   - Статусы доставки сообщений
   - Подтверждения прочтения
   - Ошибки доставки

### Настройка Webhooks:

#### Необходимые разрешения:
- `whatsapp_business_messaging` - для webhooks сообщений
- `whatsapp_business_management` - для остальных webhooks

#### Требования к endpoint:
- Поддержка HTTPS с валидным SSL сертификатом
- Обработка двух типов запросов:
  - Verification Requests (верификация)
  - Event Notifications (уведомления о событиях)

### Интеграция с Conversions API:

1. **Получение ctwa_clid из webhook**
2. **Сохранение ctwa_clid для конкретного разговора**
3. **Отправка события через Conversions API при конверсии**

## Лучшие практики

### Для всех типов событий:
1. **Использовать event_id** для дедупликации событий
2. **Отправлять события как можно быстрее** после их совершения
3. **Включать максимум доступных параметров** для улучшения атрибуции
4. **Хэшировать персональные данные** согласно требованиям

### Для Business Messaging (WhatsApp):
1. **Сохранять ctwa_clid** для каждого разговора
2. **Отправлять события только для действий в рамках WhatsApp**
3. **Использовать правильный messaging_channel** (whatsapp)
4. **Выполнять дедупликацию событий** перед отправкой

### Для качества данных:
1. **Тестировать события** с помощью test_event_code
2. **Мониторить качество** через Events Manager
3. **Проверять атрибуцию** и корректность данных
4. **Использовать правильные action_source** значения

## Устранение неполадок

### Общие проблемы:

1. **События не отображаются в Events Manager**:
   - Проверить правильность dataset_id
   - Убедиться в корректности access_token
   - Проверить формат данных

2. **Низкое качество событий**:
   - Добавить больше параметров user_data
   - Улучшить качество хэширования
   - Проверить актуальность данных

3. **Проблемы с webhook**:
   - Убедиться в корректности endpoint
   - Проверить SSL сертификат
   - Тестировать через App Dashboard

### Коды ошибок:

- **400 Bad Request** - неверный формат запроса
- **401 Unauthorized** - проблемы с авторизацией
- **403 Forbidden** - недостаточно прав доступа
- **429 Too Many Requests** - превышение лимитов API

## Полезные ссылки

- [Официальная документация Conversions API](https://developers.facebook.com/docs/marketing-api/conversions-api/)
- [Руководство по Business Messaging](https://developers.facebook.com/docs/marketing-api/conversions-api/business-messaging/)
- [WhatsApp Business API](https://developers.facebook.com/docs/whatsapp/cloud-api/)
- [Events Manager](https://www.facebook.com/events_manager2)
- [Справочный центр для бизнеса](https://www.facebook.com/business/help/)

---

*Документация обновлена: 6 июля 2025 г.*
*Версия API: v16.0+*
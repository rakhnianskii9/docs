# 📡 WhatsApp Message Types (Единый источник лимитов)

> NOTE: Этот файл является ЕДИНСТВЕННЫМ источником правды по лимитам WhatsApp (символы, размеры, структура). Все отдельные файлы *whatsapp-api-media-limits*.md и *wa-types-message.discrepancies.md* консолидированы сюда и будут удалены (deprecated). Обновление: 2025-09-16.

---

## ✅ ВСЕ ТИПЫ СООБЩЕНИЙ (общий список)

### 📝 Базовые текстовые

**`text`**  
Простое текстовое сообщение

**`reaction`**  
Эмодзи-реакция на предыдущее сообщение

---

### 🖼️ Медиа-сообщения

**`image`**  
Изображение с опциональной подписью

**`video`**  
Видео с опциональной подписью

**`audio`**  
Аудио/голосовое сообщение

**`document`**  
Документ любого типа

**`sticker`**  
Статичный WebP стикер

---

### 📍 Геоданные и контакты

**`location`**  
Геолокация на карте

**`contacts`**  
Визитная карточка (vCard)

---

### 🎯 Интерактивные (interactive)

**`button`** (interactive.type)  
До 3 кнопок быстрого ответа

**`list`** (interactive.type)  
Выпадающий список с секциями

**`product`** (interactive.type)  
Карточка одного товара из каталога

**`product_list`** (interactive.type)  
Витрина нескольких товаров

**`flow`** (interactive.type)  
Встроенная форма WhatsApp Flow

---

### 📋 Шаблонные сообщения (template)

**`text template`**  
Текстовый шаблон с переменными

**`media template`**  
Шаблон с изображением/видео в заголовке

**`button template`**  
Шаблон с кнопками действий

**`carousel template`**  
Карусель карточек (новое)

**`otp/authentication template`**  
Шаблон с одноразовым кодом

**`location template`**  
Шаблон с картой в заголовке

---

### 🛠️ Специальные и служебные (расширения BSP)

**`address`** (Gupshup расширение)  
Структурированный адрес

**`flow message`** (BSP расширение)  
Запуск flow вне interactive

---

### ⚙️ Опции / НЕ типы сообщений

Следующие элементы НЕ являются самостоятельными типами сообщения, а лишь опциями или условиями:
- `ephemeral` — флаг исчезающего сообщения (политика чата)
- `Вне 24h окно` — условие: для отправки используется только template message
- `currency` / `date_time` — типы ПАРАМЕТРОВ шаблона (components.parameters), не отдельные payload
- `copy_code`, `one_tap`, `url`, `phone_number` — подтипы кнопок template, а не типы сообщений

---

## 🧭 Сводные ключевые лимиты

| Тип | Размер / Ограничение | Примечание |
|-----|-----------------------|-----------|
| Text body (session) | ≤ 4096 символов | 24h окно |
| Image | ≤ 5MB | caption ≤3000 |
| Video | ≤ 16MB | caption ≤3000 |
| Audio | ≤ 16MB | без caption |
| Document | ≤ 100MB | filename ≤240; caption ≤3000* |
| Sticker | ≤ 100KB | static WebP 512x512 |
| Template Header | ≤ 60 символов | text/media/location |
| Template Body | ≤ 1024 символов | переменные {{n}} |
| Template Footer | ≤ 60 символов | без placeholders |
| Template button text | ≤ 20 символов | quick reply / cta |
| Template buttons total | ≤ 10 | аггрегировано |
| Quick Reply buttons | ≤ 3 | template/interactive |
| CTA (URL/Call) | ≤ 2 | разные типы |
| Button reply id | ≤ 256 символов | |
| List rows total | ≤ 10 | суммарно |
| List sections | ≤ 10 | |
| List row title | ≤ 24 символа | |
| List row desc | ≤ 72 символа | optional |
| List row id | ≤ 200 символов | уникально |
| List row postback | ≤ 200 символов | payload |
| Product list items | ≤ 30 | commerce |
| Product list sections | ≤ 10 | |
| Flow header text | ≤ 60 символов | |

*caption для документа зависит от BSP

## 📊 ДЕТАЛЬНЫЕ СТРУКТУРЫ PAYLOAD

### TEXT - Простое текстовое

**Описание:**
```
"Привет! Как дела?"
"Ваш заказ #12345 готов к выдаче"
```
📍 Максимум: 4096 символов  
💡 Самый частый тип, работает в 24h окне

**Минимальный payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "text",
  "text": {
    "body": "Привет! Это тестовое сообщение"
  }
}
```

**Полный payload с preview:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual", 
  "to": "+79991234567",
  "type": "text",
  "text": {
    "preview_url": true,
    "body": "Посмотрите наш сайт https://example.com"
  }
}
```

**Поля text объекта:**
- `body` (required, string) - текст сообщения, до 4096 символов
- `preview_url` (optional, boolean) - показывать превью ссылок

**Когда использовать:**
- Простые ответы в диалоге
- Уведомления без медиа
- Текст с эмодзи
- Сообщения со ссылками

---

### REACTION - Эмодзи реакция

**Описание:**
```
"👍" на сообщение wamid.XXX
"❤️" на фото
```
📍 Требует: message_id существующего сообщения  
⚠️ Не все клиенты поддерживают отображение

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "reaction",
  "reaction": {
    "message_id": "wamid.HBgNNzk5OTE2MzQwNjcVAgARGBI5RjQyNUE3RDYxMzRGNjE3RTYA",
    "emoji": "👍"
  }
}
```

**Удаление реакции:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "reaction",
  "reaction": {
    "message_id": "wamid.HBgNNzk5OTE2MzQwNjcVAgARGBI5RjQyNUE3RDYxMzRGNjE3RTYA",
    "emoji": ""
  }
}
```

**Поля reaction объекта:**
- `message_id` (required, string) - ID сообщения на которое реагируем
- `emoji` (required, string) - эмодзи (пустая строка = удалить)

**Поддерживаемые эмодзи:**
Все стандартные Unicode эмодзи

---

### IMAGE - Изображение

**Описание:**
```
Фото товара + "Артикул 12345, цена $99"
Скриншот + "Вот тут ошибка"
```
📍 Форматы: JPEG, PNG  
📍 Максимум: 5MB (зависит от BSP)  
💡 Можно передать как link или media_id

**С прямой ссылкой:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "image",
  "image": {
    "link": "https://cdn.example.com/photo.jpg",
    "caption": "Новое поступление! Артикул: 12345"
  }
}
```

**С media_id (предзагруженное):**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "image", 
  "image": {
    "id": "3095844730685123",
    "caption": "Ваш QR-код для входа"
  }
}
```

**Поля image объекта:**
- `link` (string) - HTTPS URL изображения, ИЛИ
- `id` (string) - ID предзагруженного медиа
- `caption` (optional, string) - подпись до 3000 символов

**Требования к изображениям:**
- Форматы: JPEG, PNG
- Размер: до 5MB (рекомендуется до 1MB для скорости)
- Разрешение: рекомендуется 800x800 для квадратных
- URL должен быть публично доступен
- HTTPS обязателен

---

### VIDEO - Видео контент

**Описание:**
```
Видео-инструкция + "Смотрите как настроить"
Демо продукта + описание
```
📍 Форматы: MP4, 3GPP  
📍 Максимум: 16MB  
⚠️ Только H.264 video codec, AAC audio

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "video",
  "video": {
    "link": "https://cdn.example.com/instruction.mp4",
    "caption": "Видео-инструкция по настройке"
  }
}
```

**Поля video объекта:**
- `link` (string) - HTTPS URL видео, ИЛИ
- `id` (string) - ID предзагруженного видео
- `caption` (optional, string) - описание до 3000 символов

**Требования:**
- Форматы: MP4, 3GPP
- Видео кодек: H.264
- Аудио кодек: AAC
- Размер: до 16MB
- Рекомендуется: 480p или 720p

---

### AUDIO - Голосовые и аудио

**Описание:**
```
Голосовая заметка от менеджера
Подкаст-фрагмент
```
📍 Форматы: AAC, MP4, AMRNB, AMRWB, OGG (opus)  
📍 Максимум: 16MB  
💡 WhatsApp автоматически показывает как voice note

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "audio",
  "audio": {
    "link": "https://cdn.example.com/voice-message.ogg"
  }
}
```

**Поля audio объекта:**
- `link` (string) - HTTPS URL аудио, ИЛИ
- `id` (string) - ID предзагруженного аудио

**Форматы и кодеки:**
- AAC (.m4a)
- MP4 (.mp4)
- AMR (.amr)
- OGG с Opus кодеком (.ogg) - рекомендуется
- Размер: до 16MB
- Автоматически отображается как voice note

---

### DOCUMENT - Файлы и документы

**Описание:**
```
PDF счёт-фактура
Excel прайс-лист
ZIP архив с файлами
```
📍 Любой MIME type  
📍 Максимум: 100MB  
💡 Показывает имя файла и размер

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "document",
  "document": {
    "link": "https://cdn.example.com/invoice.pdf",
    "filename": "Счёт-фактура-12345.pdf",
    "caption": "Счёт на оплату за январь 2025"
  }
}
```

**Поля document объекта:**
- `link` (string) - HTTPS URL документа, ИЛИ
- `id` (string) - ID предзагруженного документа
- `filename` (optional, string) - имя файла для отображения (до 240 символов)
- `caption` (optional, string) - описание

**Поддерживаемые форматы:**
- PDF (.pdf) - самый популярный
- Microsoft: Word (.docx), Excel (.xlsx), PowerPoint (.pptx)
- Архивы: ZIP, RAR
- Текст: TXT, CSV
- Размер: до 100MB

---

### STICKER - Стикеры

**Описание:**
```
WebP стикер 512x512
Анимированный стикер
```
📍 Только WebP формат  
📍 Размер: точно 512x512 px  
⚠️ Требует предварительной загрузки через Media API

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "sticker",
  "sticker": {
    "id": "23095844730685123"
  }
}
```

**Поля sticker объекта:**
- `id` (required, string) - ID предзагруженного стикера
- `link` (string) - URL стикера (редко используется)

**Требования к стикерам:**
- Формат: только WebP
- Размер: точно 512x512 пикселей
- Фон: прозрачный рекомендуется
- Максимальный размер файла: 100KB
- Анимированные (animated) не поддерживаются в Gupshup (только статичные)

---

### LOCATION - Геолокация

**Описание:**
```
Координаты офиса: 55.7558, 37.6173
Место встречи с названием и адресом
```
📍 Обязательно: latitude, longitude  
📍 Опционально: name, address  
💡 Кликабельно - открывает карты

**Полный payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "location",
  "location": {
    "latitude": 55.7558,
    "longitude": 37.6173,
    "name": "Офис компании",
    "address": "Красная площадь, 1, Москва"
  }
}
```

**Минимальный payload:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "location",
  "location": {
    "latitude": 55.7558,
    "longitude": 37.6173
  }
}
```

**Поля location объекта:**
- `latitude` (required, number) - широта
- `longitude` (required, number) - долгота
- `name` (optional, string) - название места
- `address` (optional, string) - адрес

---

### CONTACTS - Визитные карточки

**Описание:**
```
Контакт менеджера с телефоном и email
Массив контактов отдела продаж
```
📍 Формат: vCard 3.0  
📍 Можно несколько контактов в одном сообщении  
💡 Добавляется в контакты одним кликом

**Payload с полным контактом:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "contacts",
  "contacts": [{
    "addresses": [{
      "street": "Невский проспект 1",
      "city": "Санкт-Петербург",
      "state": "СПб",
      "zip": "190000",
      "country": "Россия",
      "country_code": "RU",
      "type": "WORK"
    }],
    "birthday": "1990-01-15",
    "emails": [{
      "email": "ivan@example.com",
      "type": "WORK"
    }],
    "name": {
      "formatted_name": "Иван Петров",
      "first_name": "Иван",
      "last_name": "Петров",
      "middle_name": "Сергеевич",
      "suffix": "",
      "prefix": ""
    },
    "org": {
      "company": "ООО Ромашка",
      "department": "Продажи",
      "title": "Менеджер"
    },
    "phones": [{
      "phone": "+79991234567",
      "type": "WORK",
      "wa_id": "79991234567"
    }],
    "urls": [{
      "url": "https://example.com",
      "type": "WORK"
    }]
  }]
}
```

**Минимальный контакт:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "contacts",
  "contacts": [{
    "name": {
      "formatted_name": "Служба поддержки"
    },
    "phones": [{
      "phone": "+78001234567"
    }]
  }]
}
```

---

## 🎮 INTERACTIVE MESSAGES

### BUTTON - Кнопки быстрого ответа

**Описание:**
```
"Да" | "Нет" | "Может быть"
"Заказать" | "Отменить"
```
📍 Максимум: 3 кнопки  
📍 Текст кнопки: до 20 символов  
💡 Идеально для простого выбора

**Полная структура с header и footer:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "button",
    "header": {
      "type": "text",
      "text": "Подтвердите заказ"
    },
    "body": {
      "text": "Заказ №12345 на сумму 5000₽. Доставка завтра с 10:00 до 14:00"
    },
    "footer": {
      "text": "Отмена доступна до 20:00"
    },
    "action": {
      "buttons": [
        {
          "type": "reply",
          "reply": {
            "id": "confirm_yes",
            "title": "Подтвердить"
          }
        },
        {
          "type": "reply",
          "reply": {
            "id": "confirm_no",
            "title": "Отменить"
          }
        },
        {
          "type": "reply",
          "reply": {
            "id": "confirm_change",
            "title": "Изменить"
          }
        }
      ]
    }
  }
}
```

**Минимальная структура:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "button",
    "body": {
      "text": "Вам удобно время доставки?"
    },
    "action": {
      "buttons": [
        {
          "type": "reply",
          "reply": {
            "id": "yes",
            "title": "Да"
          }
        },
        {
          "type": "reply",
          "reply": {
            "id": "no",
            "title": "Нет"
          }
        }
      ]
    }
  }
}
```

**Header варианты:**
```json
// Текстовый заголовок
"header": {
  "type": "text",
  "text": "Важное уведомление"
}

// Картинка в заголовке
"header": {
  "type": "image",
  "image": {
    "link": "https://cdn.example.com/header.jpg"
  }
}

// Видео в заголовке
"header": {
  "type": "video",
  "video": {
    "link": "https://cdn.example.com/intro.mp4"
  }
}

// Документ в заголовке
"header": {
  "type": "document",
  "document": {
    "link": "https://cdn.example.com/terms.pdf",
    "filename": "Условия.pdf"
  }
}
```

**Ограничения:**
- Максимум 3 кнопки
- ID кнопки: до 256 символов
- Текст кнопки: до 20 символов
- Body: обязателен, до 1024 символов
- Header: опционален, до 60 символов (текст)
- Footer: опционален, до 60 символов

---

### LIST - Выпадающий список

**Описание:**
```
Меню ресторана:
  Супы → [Борщ, Солянка, Том Ям]
  Горячее → [Стейк, Рыба, Паста]
```
📍 До 10 секций  
📍 До 10 строк суммарно  
💡 Компактно для большого выбора

**Полная структура меню ресторана:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "list",
    "header": {
      "type": "text",
      "text": "Меню ресторана"
    },
    "body": {
      "text": "Выберите блюдо из нашего меню. Все цены указаны в рублях."
    },
    "footer": {
      "text": "Доставка от 2000₽ бесплатно"
    },
    "action": {
      "button": "Посмотреть меню",
      "sections": [
        {
          "title": "🍜 Первые блюда",
          "rows": [
            {
              "id": "soup_1",
              "title": "Борщ",
              "description": "Традиционный со сметаной - 350₽"
            },
            {
              "id": "soup_2",
              "title": "Солянка",
              "description": "Мясная сборная - 400₽"
            },
            {
              "id": "soup_3",
              "title": "Том Ям",
              "description": "Острый тайский суп - 550₽"
            }
          ]
        },
        {
          "title": "🥩 Горячие блюда",
          "rows": [
            {
              "id": "main_1",
              "title": "Стейк Рибай",
              "description": "Мраморная говядина 300г - 2500₽"
            },
            {
              "id": "main_2",
              "title": "Лосось",
              "description": "На гриле с овощами - 1200₽"
            },
            {
              "id": "main_3",
              "title": "Паста Карбонара",
              "description": "С беконом и пармезаном - 650₽"
            }
          ]
        },
        {
          "title": "🥗 Салаты",
          "rows": [
            {
              "id": "salad_1",
              "title": "Цезарь",
              "description": "С курицей - 450₽"
            },
            {
              "id": "salad_2",
              "title": "Греческий",
              "description": "С фетой - 400₽"
            }
          ]
        }
      ]
    }
  }
}
```

**Минимальная структура выбора города:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "list",
    "body": {
      "text": "В каком городе вы находитесь?"
    },
    "action": {
      "button": "Выбрать город",
      "sections": [
        {
          "rows": [
            {
              "id": "msk",
              "title": "Москва"
            },
            {
              "id": "spb",
              "title": "Санкт-Петербург"
            },
            {
              "id": "ekb",
              "title": "Екатеринбург"
            }
          ]
        }
      ]
    }
  }
}
```

**Ограничения:**
- Максимум 10 секций
- Максимум 10 строк суммарно во всех секциях
- Button текст: обязателен, до 20 символов
- Section title: опционален, до 24 символов
- Row ID: до 200 символов
- Row title: обязателен, до 24 символов
- Row description: опционален, до 72 символов
- Row postback/payload: до 200 символов
- Body: обязателен, до 1024 символов
- Header: только текст, до 60 символов
- Footer: до 60 символов

---

### PRODUCT - Карточка товара

**Описание:**
```
Товар SKU_12345 из каталога CATALOG_001
С фото, ценой, описанием из каталога
```
📍 Требует: настроенный WhatsApp каталог  
📍 Автоматически подтягивает данные товара  
💡 Для e-commerce

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "product",
    "body": {
      "text": "Рекомендуем этот товар специально для вас!"
    },
    "footer": {
      "text": "Бесплатная доставка от 3000₽"
    },
    "action": {
      "catalog_id": "362892059384230",
      "product_retailer_id": "SKU12345"
    }
  }
}
```

**Что происходит:**
1. WhatsApp загружает данные товара из каталога
2. Показывает карточку с фото, названием, ценой, описанием
3. Пользователь может добавить в корзину прямо из чата

**Требования:**
- Каталог должен быть создан в WhatsApp Business Manager
- Товар должен быть активен в каталоге
- catalog_id: ID вашего каталога
- product_retailer_id: SKU товара в каталоге

---

### PRODUCT_LIST - Витрина товаров

**Описание:**
```
"Рекомендуемые": [Товар1, Товар2, Товар3]
"Акции": [Товар4, Товар5]
```
📍 До 30 товаров (обычно)  
📍 Группировка по секциям  
💡 Мини-магазин в чате

**Структура с секциями:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "product_list",
    "header": {
      "type": "text",
      "text": "Новая коллекция"
    },
    "body": {
      "text": "Специально отобранные товары для вас"
    },
    "footer": {
      "text": "Нажмите для просмотра каталога"
    },
    "action": {
      "catalog_id": "362892059384230",
      "sections": [
        {
          "title": "🔥 Хиты продаж",
          "product_items": [
            {"product_retailer_id": "SKU001"},
            {"product_retailer_id": "SKU002"},
            {"product_retailer_id": "SKU003"}
          ]
        },
        {
          "title": "🆕 Новинки",
          "product_items": [
            {"product_retailer_id": "SKU101"},
            {"product_retailer_id": "SKU102"}
          ]
        },
        {
          "title": "💰 Скидки",
          "product_items": [
            {"product_retailer_id": "SKU201"},
            {"product_retailer_id": "SKU202"},
            {"product_retailer_id": "SKU203"},
            {"product_retailer_id": "SKU204"}
          ]
        }
      ]
    }
  }
}
```

**Минимальная структура:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "product_list",
    "body": {
      "text": "Посмотрите наши товары"
    },
    "action": {
      "catalog_id": "362892059384230",
      "sections": [
        {
          "product_items": [
            {"product_retailer_id": "SKU001"},
            {"product_retailer_id": "SKU002"}
          ]
        }
      ]
    }
  }
}
```

**Ограничения:**
- Максимум 30 товаров суммарно
- Максимум 10 секций
- Минимум 1 товар в секции
- Section title: до 24 символов

---

### FLOW - Встроенные формы

**Описание:**
```
Форма регистрации с полями
Анкета с валидацией
Multi-step опросник
```
📍 Требует: созданный Flow в WhatsApp Manager  
📍 Поддержка: не все регионы  
⚠️ Beta функция, сложная настройка

**Запуск формы регистрации:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "flow",
    "header": {
      "type": "text",
      "text": "Регистрация"
    },
    "body": {
      "text": "Заполните форму для регистрации в программе лояльности"
    },
    "footer": {
      "text": "Займёт 2 минуты"
    },
    "action": {
      "flow_id": "1000128587500897",
      "flow_message_version": "3",
      "flow_token": "unused",
      "flow_cta": "Заполнить форму",
      "flow_action": "navigate",
      "flow_action_payload": {
        "screen": "WELCOME",
        "data": {
          "prefill_name": "Иван",
          "prefill_phone": "+79991234567"
        }
      }
    }
  }
}
```

**Минимальный запуск:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "interactive",
  "interactive": {
    "type": "flow",
    "body": {
      "text": "Пройдите опрос"
    },
    "action": {
      "flow_id": "1000128587500897",
      "flow_cta": "Начать",
      "flow_action": "navigate"
    }
  }
}
```

**Параметры action:**
- `flow_id` (required) - ID созданного Flow
- `flow_cta` (required) - текст кнопки запуска
- `flow_action` (required) - "navigate" или "data_exchange"
- `flow_token` (required for data_exchange) - токен для обмена данными
- `flow_message_version` - версия Flow (обычно "3")
- `flow_action_payload` - начальные данные для формы

---

## 📋 TEMPLATE MESSAGES

### Central Template Limits (норматив)
```
Header (text)          ≤ 60 символов
Body                   ≤ 1024 символов
Footer                 ≤ 60 символов
Button text            ≤ 20 символов
Quick Reply buttons    ≤ 3
CTA (URL/Call)         ≤ 2 (разные типы)
Total buttons (agg.)   ≤ 10
Reply ID (button)      ≤ 256 символов
Variables {{n}}        Ограничены длиной body
```


### TEXT TEMPLATE - Текстовый шаблон

**Описание:**
```
"Здравствуйте {{1}}! Ваш заказ {{2}} готов"
→ "Здравствуйте Иван! Ваш заказ #12345 готов"
```
📍 Требует: предварительное утверждение Meta  
📍 Переменные: {{1}}, {{2}}, {{3}}...  
💡 Единственный способ писать вне 24h

**Простой шаблон приветствия:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "welcome_message",
    "language": {
      "code": "ru"
    }
  }
}
```

**Шаблон с переменными:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "order_confirmation",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "Иван"
          },
          {
            "type": "text",
            "text": "ORD-12345"
          },
          {
            "type": "text",
            "text": "15 января"
          }
        ]
      }
    ]
  }
}
```

**Текст шаблона в WhatsApp Manager:**
```
Здравствуйте, {{1}}! 
Ваш заказ {{2}} подтверждён.
Ожидаемая дата доставки: {{3}}.
Спасибо за покупку!
```

**Результат для пользователя:**
```
Здравствуйте, Иван!
Ваш заказ ORD-12345 подтверждён.
Ожидаемая дата доставки: 15 января.
Спасибо за покупку!
```

---

### MEDIA TEMPLATE - Шаблон с медиа

**Описание:**
```
[Изображение товара]
"{{1}}, специально для вас скидка {{2}}%"
```
📍 Header: image, video, document  
📍 Body: текст с переменными  
💡 Выше CTR чем text

**С изображением в заголовке:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "product_announcement",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "header",
        "parameters": [
          {
            "type": "image",
            "image": {
              "link": "https://cdn.example.com/product.jpg"
            }
          }
        ]
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "iPhone 15 Pro"
          },
          {
            "type": "text",
            "text": "20%"
          }
        ]
      }
    ]
  }
}
```

**С видео в заголовке:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "video_tutorial",
    "language": {
      "code": "en_US"
    },
    "components": [
      {
        "type": "header",
        "parameters": [
          {
            "type": "video",
            "video": {
              "link": "https://cdn.example.com/tutorial.mp4"
            }
          }
        ]
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "5 минут"
          }
        ]
      }
    ]
  }
}
```

**С документом в заголовке:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "invoice_notification",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "header",
        "parameters": [
          {
            "type": "document",
            "document": {
              "link": "https://cdn.example.com/invoice.pdf",
              "filename": "Счёт_12345.pdf"
            }
          }
        ]
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "50000"
          },
          {
            "type": "currency",
            "currency": {
              "fallback_value": "50000 RUB",
              "code": "RUB",
              "amount_1000": 50000000
            }
          }
        ]
      }
    ]
  }
}
```

---

### BUTTON TEMPLATE - Шаблон с кнопками

**Описание:**
```
Текст шаблона
[Позвонить] [Перейти на сайт] [Отписаться]
```
📍 Quick Reply: до 3 кнопок  
📍 URL Button: до 2 + динамические параметры  
📍 Phone Button: 1 кнопка  
⚠️ Комбинации ограничены

**Quick Reply кнопки:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "appointment_reminder",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "завтра в 15:00"
          }
        ]
      },
      {
        "type": "button",
        "sub_type": "quick_reply",
        "index": "0",
        "parameters": [
          {
            "type": "payload",
            "payload": "confirm_yes"
          }
        ]
      },
      {
        "type": "button",
        "sub_type": "quick_reply",
        "index": "1",
        "parameters": [
          {
            "type": "payload",
            "payload": "confirm_no"
          }
        ]
      }
    ]
  }
}
```

**URL кнопки с параметрами:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "tracking_update",
    "language": {
      "code": "en_US"
    },
    "components": [
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "TRACK123456"
          }
        ]
      },
      {
        "type": "button",
        "sub_type": "url",
        "index": "0",
        "parameters": [
          {
            "type": "text",
            "text": "TRACK123456"
          }
        ]
      }
    ]
  }
}
```

**Шаблон в Manager с URL:**
```
Body: Ваш заказ {{1}} отправлен!
Button: [Отследить] -> https://example.com/track?id={{1}}
```

**Call кнопка:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "support_contact",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "button",
        "sub_type": "phone_number",
        "index": "0",
        "parameters": []
      }
    ]
  }
}
```

---

### CAROUSEL TEMPLATE - Карусель карточек

**Описание:**
```
[Карточка 1: фото + текст + кнопка]
[Карточка 2: фото + текст + кнопка]
Свайп для просмотра →
```
📍 До 10 карточек  
📍 Каждая: header + body + buttons  
⚠️ Не все BSP поддерживают

**Структура карусели товаров:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "summer_sale_carousel",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "carousel",
        "cards": [
          {
            "card_index": 0,
            "components": [
              {
                "type": "header",
                "parameters": [
                  {
                    "type": "image",
                    "image": {
                      "link": "https://cdn.example.com/product1.jpg"
                    }
                  }
                ]
              },
              {
                "type": "body",
                "parameters": [
                  {
                    "type": "text",
                    "text": "Футболка"
                  },
                  {
                    "type": "text",
                    "text": "1990"
                  }
                ]
              },
              {
                "type": "button",
                "sub_type": "quick_reply",
                "index": "0",
                "parameters": [
                  {
                    "type": "payload",
                    "payload": "buy_product_1"
                  }
                ]
              }
            ]
          },
          {
            "card_index": 1,
            "components": [
              {
                "type": "header",
                "parameters": [
                  {
                    "type": "image",
                    "image": {
                      "link": "https://cdn.example.com/product2.jpg"
                    }
                  }
                ]
              },
              {
                "type": "body",
                "parameters": [
                  {
                    "type": "text",
                    "text": "Джинсы"
                  },
                  {
                    "type": "text",
                    "text": "3990"
                  }
                ]
              },
              {
                "type": "button",
                "sub_type": "quick_reply",
                "index": "0",
                "parameters": [
                  {
                    "type": "payload",
                    "payload": "buy_product_2"
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

---

### OTP/AUTHENTICATION TEMPLATE

**Описание:**
```
"Ваш код: 123456"
[Копировать код] - автозаполнение
```
📍 Специальный тип для 2FA  
📍 Кнопка copy_code или one_tap  
💡 Повышенная доставляемость

**One-tap authentication:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "otp_login",
    "language": {
      "code": "en_US"
    },
    "components": [
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "784523"
          }
        ]
      },
      {
        "type": "button",
        "sub_type": "url",
        "index": "0",
        "parameters": [
          {
            "type": "text",
            "text": "784523"
          }
        ]
      }
    ]
  }
}
```

**Copy code button:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "verification_code",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "456789"
          }
        ]
      },
      {
        "type": "button",
        "sub_type": "copy_code",
        "index": "0",
        "parameters": [
          {
            "type": "copy_code",
            "code": "456789"
          }
        ]
      }
    ]
  }
}
```

---

### LOCATION TEMPLATE - Шаблон с картой

**Описание:**
```
[Карта с пином]
"Ждём вас по адресу {{1}}"
```
📍 Header type: location  
⚠️ Редко используется

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "+79991234567",
  "type": "template",
  "template": {
    "name": "store_location",
    "language": {
      "code": "ru"
    },
    "components": [
      {
        "type": "header",
        "parameters": [
          {
            "type": "location",
            "location": {
              "latitude": 55.7558,
              "longitude": 37.6173,
              "name": "Наш магазин",
              "address": "Красная площадь, 1"
            }
          }
        ]
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "с 10:00 до 22:00"
          }
        ]
      }
    ]
  }
}
```

---

## 🛠️ СПЕЦИАЛЬНЫЕ ТИПЫ

### ADDRESS - Структурированный адрес (Gupshup)

**Описание:**
```
{
  line1: "Невский пр. 1",
  city: "Санкт-Петербург",
  postal_code: "190000"
}
```
📍 НЕ стандарт Cloud API  
📍 Только некоторые BSP  
⚠️ Редко нужно

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "address",
  "address": {
    "line1": "Невский проспект 1",
    "line2": "офис 100",
    "city": "Санкт-Петербург",
    "state": "Ленинградская область",
    "postal_code": "190000",
    "country": "Россия",
    "country_code": "RU"
  }
}
```

---

### FLOW MESSAGE - BSP расширение

**Описание:**
```
Триггер кастомного flow_123
С предзаполненными данными
```
📍 Отличается от interactive.flow  
⚠️ Проприетарное у BSP

**Payload:**
```json
{
  "messaging_product": "whatsapp",
  "to": "+79991234567",
  "type": "flow",
  "flow": {
    "id": "flow_custom_123",
    "entry_point": "registration",
    "data": {
      "user_name": "Иван",
      "user_phone": "+79991234567"
    }
  }
}
```

---

### EPHEMERAL - Исчезающие сообщения

**Описание:**
```
Сообщение удалится через 24 часа
Временный пароль с автоудалением
```
📍 Флаг/политика, не отдельный тип  
⚠️ Зависит от настроек чата

**Использование:**
Добавляется как мета-параметр к любому типу сообщения через контекст чата или настройки аккаунта. Не имеет отдельного payload.

---

## 🔧 СПЕЦИАЛЬНЫЕ ТИПЫ ПАРАМЕТРОВ

### CURRENCY - Валюта в шаблонах

```json
{
  "type": "currency",
  "currency": {
    "fallback_value": "$100.99",
    "code": "USD",
    "amount_1000": 100990
  }
}
```

**Поля:**
- `fallback_value` - текстовое представление для старых клиентов
- `code` - ISO 4217 код валюты
- `amount_1000` - сумма × 1000 (для точности)

---

### DATE_TIME - Дата и время

```json
{
  "type": "date_time",
  "date_time": {
    "fallback_value": "15 января 2025, 10:00",
    "day_of_week": 3,
    "day_of_month": 15,
    "year": 2025,
    "month": 1,
    "hour": 10,
    "minute": 0
  }
}
```

---

## 📊 СРАВНИТЕЛЬНАЯ ТАБЛИЦА ВОЗМОЖНОСТЕЙ

| Функция      | Session         | Template           | Interactive        | Ограничения      |
|--------------|-----------------|--------------------|--------------------|------------------|
| Текст        | ✅ text         | ✅ с переменными   | ✅ в body           | 2000 символов    | 
| Изображения  | ✅ image        | ✅ в header        | ✅ в header         | 5MB              |
| Видео        | ✅ video        | ✅ в header        | ✅ в header         | 16MB             |
| Аудио        | ✅ audio        | ❌                 | ❌                  | 16MB             |
| Документы    | ✅ document     | ✅ в header        | ✅ в header         | 100MB            |
| Стикеры      | ✅ sticker      | ❌                 | ❌                  | 512x512 WebP     |
| Локация      | ✅ location     | ✅ в header        | ❌                  | -                |
| Контакты     | ✅ contacts     | ❌                 | ❌                  | vCard 3.0        |
| Кнопки       | ❌              | ✅ до 3            | ✅ до 3             | 20 символов      |
| Списки       | ❌              | ❌                 | ✅ list             | 10 rows          |
| Товары       | ❌              | ✅ carousel        | ✅ product/list     | Нужен каталог    |
| Формы        | ❌              | ✅ с flow          | ✅ flow             | Beta             |

### 🛒 Каталог / Commerce (детализация)

| Вид (catalog)            | Тип / канал                | Требует каталог | Ограничения                       | Примечание |
|--------------------------|----------------------------|-----------------|----------------------------------|------------|
| Один товар               | interactive.product        | Да              | 1 product_retailer_id            | Автоданные из каталога |
| Список товаров           | interactive.product_list   | Да              | ≤30 товаров, ≤10 секций          | ≥1 товар в секции |
| Карусель карточек        | template.carousel          | Часто           | ≤10 карточек                     | Ограниченный rollout |
| Товар через media header | template (media header)    | Да              | Media header + body vars         | Не выбор списка |
| Flow форма (commerce)    | interactive.flow           | Опционально     | Beta, региональные ограничения   | Data capture |

---

## 🚫 ЧАСТЫЕ ОШИБКИ И РЕШЕНИЯ

### Ошибка 470 - Invalid phone number
**Причина:** Неверный формат номера  
**Решение:** Используйте E.164 формат: +79991234567

### Ошибка 131051 - Message type not supported
**Причина:** Комбинация полей не поддерживается  
**Решение:** Проверьте совместимость header типа с основным типом

### Ошибка 131053 - Missing required field
**Причина:** Не указано обязательное поле  
**Решение:** Для interactive всегда нужен body

### Ошибка 131047 - Re-engagement message
**Причина:** 24h окно закрыто  
**Решение:** Используйте template message

### Ошибка 131052 - Media download failed
**Причина:** URL недоступен или не HTTPS  
**Решение:** Проверьте доступность URL, используйте HTTPS

### Ошибка 133000 - Template not found
**Причина:** Шаблон не существует или не утверждён  
**Решение:** Проверьте название и статус в Business Manager

### Ошибка 131049 - Template parameter mismatch
**Причина:** Количество параметров не совпадает  
**Решение:** Передайте все переменные {{1}}, {{2}}...

---

## 🔗 ПОЛЕЗНЫЕ ССЫЛКИ

**Официальная документация:**
- WhatsApp Cloud API: https://developers.facebook.com/docs/whatsapp/cloud-api/
- Interactive Messages: https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages#interactive-messages
- Template Messages: https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-message-templates
- Media Handling: https://developers.facebook.com/docs/whatsapp/cloud-api/reference/media

**Gupshup специфика:**
- Session Messages: docs/tech-docs/Gupshup/Messaging/Send Session Message V3 Examples.md
- Template Messages: docs/tech-docs/Gupshup/Messaging/Send Template Message v3 Examples.md

**Внутренние документы:**
- Структура подписчика: docs/my-docs/exact-subscriber-data-structure-v2.md
- Arriwa логика: docs/my-docs/arriwa-logic.md
# 📦 Ограничения Медиа и Символьных Лимитов WhatsApp Business API (RU)

**Назначение:** Свод проверенных лимитов (Meta Cloud API + Gupshup) для медиа, символов и структур (Template / Interactive). Документ служит операционным референсом для валидации и генерации сообщений. Все значения синхронизированы на дату: 2025‑09‑16.

---

## 🎯 Кратко (TL;DR)
| Тип | Размер | Форматы | Ключевые символьные лимиты |
|-----|--------|---------|----------------------------|
| Изображение | 5 MB | JPEG, PNG | Caption ≤ 3000 символов |
| Видео | 16 MB | MP4, 3GP | Caption ≤ 3000 символов |
| Аудио | 16 MB | AAC, MP4, AMR, OGG | (нет caption) |
| Документ | 100 MB | PDF, DOC(X), PPT(X), XLS(X), TXT, CSV и др. | filename ≤ 240; caption ≤ 3000 (если поддерживается) |
| Стикер | 100 KB | WebP | 512×512, статичный (анимированные Gupshup не поддерживает) |
| Free‑form текст | — | — | Body ≤ 4096 символов |
| Template Header | — | text/media/location | Текст ≤ 60 |
| Template Body | — | text | ≤ 1024 символов |
| Template Footer | — | text | ≤ 60 символов |
| Template Button | — | quick_reply / cta | Текст ≤ 20, всего кнопок ≤ 10 (типовые сублимиты) |
| Interactive List | — | list | Body ≤ 1024; Header ≤ 60; Footer ≤ 60; Button ≤ 20 |
| List Row | — | rows | title ≤ 24; description ≤ 72; id ≤ 200; postback ≤ 200 |
| Interactive Buttons | — | button | ≤ 3 кнопок, title ≤ 20, reply id ≤ 256 |
| Product List | — | product_list | ≤ 30 товаров, ≤ 10 секций, section title ≤ 24 |

---

## 🗂 Медиа: Размеры и Форматы
### 🖼 Изображения
- Макс: 5 MB (JPEG, PNG)
- Рекомендуемые пропорции: 1:1 или 4:3
- Авто ресайз сервером WhatsApp
- Caption (подпись) в свободной (free-form) сессии: ≤ 3000 символов

### 🎬 Видео
- Макс: 16 MB (MP4, 3GP)
- Кодеки: H.264 (видео), AAC (аудио)
- Ориентировочная длительность: ~3 мин (зависит от битрейта)
- Caption ≤ 3000 символов

### 🎧 Аудио
- Макс: 16 MB (AAC, MP4, AMR, OGG Opus)
- Ориентировочная длительность: ~16 мин @ средний битрейт
- Caption / текстовая подпись не используется

### 📄 Документы
- Макс: 100 MB
- Форматы: PDF, DOC, DOCX, PPT(X), XLS(X), TXT, CSV и др.
- filename ≤ 240 символов (иначе усечение)
- (Если поле caption используется через некоторые BSP) — до 3000 символов

### 🧩 Стикеры
- Формат: WebP только
- Размер: 512×512 px, прозрачный фон, 16 px отступ (padding)
- Файл ≤ 100 KB
- В Gupshup (согласно их публичным страницам) анимированные не поддержаны — локальная дока с обратным утверждением требует правки

---

## ✉️ Текстовые и Шаблонные Лимиты
| Элемент | Лимит | Примечание |
|---------|-------|------------|
| Free‑form текст (session) | 4096 символов | В локальном файле ранее ошибочно указано 2000 |
| Template Header (text) | 60 | 1 placeholder допустим |
| Template Body | 1024 | Форматирование: **bold**, _italic_, ~strike~, `code` |
| Template Footer | 60 | Без placeholders |
| Template Button Text | 20 | Один тип кнопок на шаблон (в конкретной версии) |
| Совокупно Template Buttons | ≤ 10 | Ограничение платформы (каждый тип имеет сублимит) |
| Quick Reply (template) | ≤ 3 | Входит в общее число |
| Call-To-Action (template) | ≤ 2 | Разные действия: call + url |
| Переменные {{n}} | Ограничены body length | Используются как {{1}}, {{2}}, ... |
| Currency parameter | amount_1000 | Масштаб: 1000 = 1.000 единица |
| Date/Time parameter | поля даты/времени | fallback_value обязателен |

---

## 🧭 Interactive (Интерактивные) Структуры
### 🔘 Кнопки (Interactive type="button")
| Поле | Лимит |
|------|-------|
| Кнопок | ≤ 3 |
| Текст кнопки | ≤ 20 символов |
| reply.id | ≤ 256 символов |
| Body | ≤ 1024 |
| Header (text) | ≤ 60 |
| Footer | ≤ 60 |

### 📜 Списки (Interactive type="list")
| Элемент | Лимит | Примечание |
|---------|-------|-----------|
| Секций | ≤ 10 | |
| Строк суммарно | ≤ 10 | По всем секциям |
| Header | ≤ 60 | Опционально |
| Body | ≤ 1024 | Обязательно |
| Footer | ≤ 60 | Опционально |
| Основная кнопка (button) | ≤ 20 | Обязательно |
| Section title | ≤ 24 | Опционально |
| Row id | ≤ 200 | Уникально |
| Row title | ≤ 24 | Обязательно |
| Row description | ≤ 72 | Опционально |
| Row postback/payload | ≤ 200 | Независимо от title |

### 🛍 Product / Product List
| Параметр | Лимит |
|----------|------|
| Секций (product_list) | ≤ 10 |
| Товаров (product_list) | ≤ 30 суммарно |
| Section title | ≤ 24 |
| product_retailer_id | Завязан на каталоге |

### 🧾 Flow (interactive type="flow")
- Header text ≤ 60
- CTA (flow_cta) — краткий, рекомендуется ≤ 20
- Доп. payload структуры не меняют символьных глобальных лимитов

### 🗺 Location
- Имя и адрес: не документировано строго → практический безопасный предел ≤ 100 символов каждое (консервативно)

---

## 🔗 Caption & Filename Сводка
| Поле | Лимит |
|------|-------|
| Image caption | ≤ 3000 |
| Video caption | ≤ 3000 |
| Document filename | ≤ 240 |
| Document caption* | ≤ 3000 (если поддержано BSP) |
| Sticker caption | — (отсутствует) |

*Некоторые BSP не передают caption для документа — использовать fallback текст в отдельном сообщении.

---

## ⚠️ Расхождения, Требующие Коррекции (в локальном `wa-types-message.md`)
| Что сейчас | Как должно быть | Действие |
|------------|-----------------|----------|
| Free‑form text: 2000 | 4096 | Исправить значение |
| Animated sticker: поддерживается | Не поддерживается (Gupshup) | Удалить / пометить как unsupported |
| Image/Video caption: 1024 | 3000 | Обновить лимит |
| Не указаны: list row postback 200, filename 240 | Должны быть | Добавить строки |
| Нет явного упоминания caption 3000 для документов | Добавить | Дополнить описание |

---

## 🔍 Источники Проверки
- Gupshup Docs: Outbound (free form), Outbound (templates), Create Template, Message Types Outgoing (дата выгрузки 2025‑09‑16)
- Meta Cloud API (Media, Template, Interactive) — актуальные endpoints

---

## 🛠 Интеграция в Код
Файл валидатора: `src/integrations/whatsapp/validators/whatsapp-limits.validator.ts` (должен использовать значения из этого документа). При изменении лимитов:
1. Обновить константы / маппинг.
2. Добавить тесты граничных значений (максимум+1 символ/байт).
3. Обновить документацию (этот файл + `wa-types-message.md`).

---

## ✅ Контроль Качества Перед Релизом
- Сценарии отправки для каждого media type (smoke)
- Проверка отклонения (reject) при превышении лимита
- Логгирование причПричины отказа (error code + human readable)
- Синхронизация с шаблонами (template placeholders vs body length)

---

## 🧾 Глоссарий
| Термин | Описание |
|--------|----------|
| Free‑form | Сообщение внутри 24h окна без шаблона |
| Template | Предварительно утверждённый шаблон (вне окна / нотификация) |
| Interactive | Структурированные UI-сообщения (button, list, product, flow) |
| Caption | Текстовая подпись к media (image/video/document) |
| Payload | JSON структура конкретного сообщения |

---

## 📌 Примечания
- Если Meta обновит лимиты — приоритет всегда у официального обновления: фиксировать дату, ссылку, дифф.
- Для новых типов (carousel, order, payment) добавлять раздел после первичной валидации.
- Строго избегать локальных «пониженных» лимитов без документированной причины (ADR).

---

## 🔗 References (EN)
- https://developers.facebook.com/docs/whatsapp/api/media
- https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages
- https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-message-templates
- Gupshup public docs (captured snapshots 2025‑09‑16)

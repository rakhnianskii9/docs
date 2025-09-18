# 🚀 Arriwa - Техническая документация платформы

> **📎 Связанные документы:**
> - [Webhook Payload Specification](../incoming-webhooks/webhook-payload-specification.md) - RAW формат входящих webhook
> - [Subscriber Data Structure](../subscriber/exact-subscriber-data-structure-v2.md) - модель данных подписчика в БД
> - [WhatsApp Message Types](../wa-types-message.md) - полный список типов сообщений
> - [Project Description](../project-description/description.md) - концептуальный обзор ОБЩИЙ ПУТЬ

---

## ✅ АРХИТЕКТУРА И КОМПОНЕНТЫ СИСТЕМЫ

### 📱 Точки входа

**`CTWA`** (Click to WhatsApp Ads)  
Реклама Facebook/Instagram с переходом в WhatsApp

**`Direct Message`**  
Прямое сообщение без рекламного источника

**`QR Code`**  
Сканирование QR-кода с переходом в чат

**`Link Share`**  
Переход по wa.me ссылке

### 🎯 Этапы воронки

**`Qualification Steps`** (Q1...Qn)  
Пошаговая квалификация лида

**`Content Funnel`**  
Обучающий контент после квалификации

**`Product Showcase`**  
Демонстрация товаров/услуг

**`Payment Flow`**  
Процесс оплаты через Stripe

### 🤖 Классификация ответов

**`valid`**  
Валидный ответ на вопрос воронки (confidence ≥ 0.65)

**`noise`**  
Шум/бред/нерелевантный ответ (confidence < 0.3)

**`ambiguous`**  
Неоднозначный ответ требующий уточнения (0.3 ≤ confidence < 0.65)

### 🔄 Потоки данных

**`GOOD`**  
Основной поток валидных ответов

**`NOISE`**  
Поток невалидных/мусорных ответов

**`ESCALATION`**  
Эскалация на оператора

### 🛑 Код-слова управления

**`stop`** / **`стоп`** / **`unsubscribe`**  
Полная отписка от рассылок

**`pause`** / **`пауза`**  
Временная приостановка

**`start`** / **`старт`**  
Возобновление после паузы

**`operator`** / **`оператор`**  
Запрос живого оператора

### 📊 Интеграции

**`CAPI`** (Facebook Conversions API)  
События конверсий для оптимизации

**`Pixels`**  
для регистрации событий (GOOD и NOISE потоки на разные pixels/datasets)

**`Ads Manager`**  
для выгрузки статистики рекламных кампаний

**`Custom Audience`**  
для сохранения и расширения, LAL

**`E-commerce и WhatsApp catalog`**  
для отправки карточек товаров напрямую в чат WhatsApp

**`Ads Audience Transporter`**  
для экспорта аудиторий в Google, TikTok, LinkedIn

**`Enrichment`**  
для двухстороннего обогащения всех компонентов данными

**`Kommo CRM`**  
для управления статусами сделок

**`Stripe`**  
для проведения оплат

**`Vuexy`**  
frontend для работы с некоторыми модулями, отчетами и пр

---

## 🏢 Онбординг (из описания проекта)

Шаги (строго по описанию):
1. https://flows.rakhnianskii.com — первичная авторизация Facebook (public_profile, business_management, email)
2. /wa-onboarding — только этот раздел доступен
3. Прохождение онбординга (создание приложения, получение/привязка номера)
4. /credentials — разблокировка раздела для привязки CAPI, Pixels, аудитории, статистики (ads_read, ads_management)
5. Опционально авторизация для e-commerce каталога (commerce_account_read_orders, commerce_account_read_reports, commerce_account_read_settings, catalog_management)
6. Редирект на /chatflows

---

## 🔧 FLOWISE ОРКЕСТРАЦИЯ И ИСТОЧНИКИ ИСТИНЫ

### Agentflow → Chatflow архитектура

**Двухуровневая система:**
- **Agentflow** - управляющий слой, маршрутизация сообщений, агент/оркестратор (код-слова, старт/эскалации, вызов чат-сценария)
- **Chatflow** - исполнительный слой, LLM обработка и валидация, детерминированные шаги квалификации (валидатор/парсинг/ветвления/эмиссии)

**Формат sessionId (универсальный):**
```
{source}_{details}_{subscriber_id}_{timestamp}
```

Примеры:
- **CTWA**: `ctwa_fb_{subscriber_id}_{timestamp}`
- **QR**: `qr_{product_id}_{subscriber_id}_{timestamp}`  
- **Direct**: `direct_{subscriber_id}_{timestamp}`
- **Link**: `link_{utm_source}_{subscriber_id}_{timestamp}`
- **WhatsApp практика**: `<tenant>:<phone>:<flow>` — чтобы не смешивать проекты/воронки

### Согласование терминов: «сессия» (sessionId)

- **Определение:** ярлык, под которым Flowise/Arriwa хранит и читает историю *данного* разговора и состояние шага
- **Кто задаёт:** sessionId задаём **мы**; все Memory-ноды и кастом-тулы читают/пишут по нему
- **Где живёт:** в Arriwa — `Redis` (живая история окна/контекст), `PostgreSQL` (архив/состояние шагов), долговременная память через `pgvector`. Встроенная Buffer-память Flowise пишет в `chat_message`, но **не** считается источником истины
- **Передача в рантайме:** Prediction API → `overrideConfig.sessionId`; в узлах/тулах доступно как `$flow.sessionId`
- **Отличие от chatId:** chatId — внутренний ID беседы Flowise; sessionId — наш внешний/логический ID. Источник истины для склейки — `sessionId`

### Инварианты (зафиксировано)

- **Порог валидности:** `0.65` (единый для валидации ответов и логики агента)
- **Хранилища:** `Redis` (оперативная память/контекст + кэш), `PostgreSQL` (источник истины), `pgvector` (семантическая память/RAG)

### Двухуровневая проверка код-слов

**Уровень 1: Бэкенд (primary source of truth)**
- Проверяется на КАЖДОМ входящем сообщении до любой другой логики
- Нормализация в lowercase, поиск синонимов
- Немедленное исполнение действий (optOut, snooze, escalation)

**Уровень 2: Flowise Agentflow (страховочная сетка)**
- Дублирующая проверка для edge cases
- Срабатывает при ручном запуске или сбое webhook
- Синхронизация с бэкендом через API

### Источники истины и согласованность

**Код-слова**: бэкенд (единственный источник истины)
- Flowise: страховочная проверка для edge cases (ручной запуск, сбой webhook)

**Состояние шага**: Flowise Variables + БД (для восстановления)
- Variables: живое состояние в рамках сессии
- БД: персистентное состояние для восстановления после рестарта

**История чата**: Redis/Valkey (живая) + Postgres (архив)

**Счетчики репромптов**: Flowise Variables (в рамках шага)

**Счетчики нуджей**: бэкенд nudge-service (между сообщениями)

**CTWA данные**: БД (неизменяемые после создания)

### Идемпотентность через event_id

```javascript
// Генерация уникального event_id
event_id = `{sessionId}-q{n}-{hash(answer_raw)}`
// normalize: trim/lower/collapse spaces/remove urls/phones/emojis

// Проверка в outbox перед эмиссией
const exists = await db.events_outbox.findUnique({
  where: { event_id }
})

if (exists) {
  return exists.result // Возвращаем предыдущий результат
}
```

---

## 📋 ТОЧКИ ВХОДА И ИНИЦИАЛИЗАЦИЯ

### CTWA-клик и первое сообщение

**Последовательность действий:**
1. Пользователь кликает на рекламу Facebook/Instagram
2. Открывается WhatsApp с предзаполненным номером бизнеса
3. Пользователь отправляет первое сообщение (любой текст или шаблонное приветствие)

**Webhook Gupshup принимает сообщение:**
- **Извлекаются CTWA-параметры**: campaign_id, adset_id, ad_id, creative_id, utm_*, placement, publisher_platform, click_id
- **Создается уникальный sessionId**: `ctwa_fb_{subscriber_id}_{timestamp}`
- **Стартовый триггер**: любое первое входящее сообщение
- **Нормализация данных**: phone, name, language, country, timezone

### QR Code вход (альтернативная точка)

**QR содержит**: `wa.me/79991234567?text=QR_START_PROMO&product_id=SKU001&location_id=MSK`

**Извлекаются параметры**: product_id, location_id, promo_code

**Создается сессия**: `qr_{product_id}_{subscriber_id}_{timestamp}`

**Приоритет**: показ конкретного товара/услуги из product_id

### Постоянный мониторинг код-слов

**Проверяется на КАЖДОМ входящем сообщении до любой другой логики:**

| Триггер  | Варианты                              | Условие        | Действие                      | Ответ                  | Последствия    |
| -------- | ------------------------------------- | -------------- | ----------------------------- | ---------------------- | -------------- |
| Оператор | оператор/человек/support/agent/помощь | без «не» рядом | `hold=true`, HITL             | «Подключаю оператора…» | SLA, нуджи off |
| Стоп     | стоп/stop/unsubscribe/отписка         | однозначно     | `optOut=true`                 | «Остановлено…»         | Блок исходящих |
| Пауза    | пауза/позже/через X                   | —              | `snoozeUntil=ts`, `hold=true` | «Пауза до…»            | Ретраи off     |
| Старт    | старт/продолжим/продолжай             | из hold        | `hold=false`                  | «Продолжаем с qN»      | Таймеры on     |

### Атомарная инициализация (параллельно)

**CAPI Facebook**: событие Start/OptIn с хешированным номером телефона
**Custom Audiences**: добавление в аудиторию A1_STARTED_NOT_FINISHED
**Kommo CRM**: создается контакт и сделка в стадии "Новый"
**База данных**: записывается подписчик с CTWA-параметрами и raw referral блоком
**Менеджер**: назначается по правилам (round-robin/нагрузка/гео)
**Flowise**: запускается Agentflow с контекстом сессии

---

## 🎯 N-ШАГОВЫЙ КВАЛИФИКАТОР + «ЖИВОЙ» АГЕНТ + ДВА ПОТОКА (GOOD/NOISE)

### Назначение и рамки

- N шагов квалификации (не фикс.)
- На **каждом валидном ответе** — немедленные эмиссии: **CAPI / Аудитории / CRM / БД**
- **Шум/ambiguous** — не в основные каналы; либо в отдельные **noise-каналы/аудитории**/**негативный пиксель/датасет** (для исключений и аналитики)
- Параллельно всегда активен «живой» агент: анти-чушь, ответы на вопросы (БЗ), код-слова, нуджи, HITL
- **Persist-all:** сохраняем **все** ответы/шумы в БД (для анализа и **lookalike-исключений**)
- Идемпотентность, outbox, ретраи — **обязательно**

### Состояния и флаги (минимум)

`stage∈{Q1..Qn,CONTENT,END}`, `askedId`, `expectedType∈{free_text,choice}`, `answers{}`, `validity∈{valid,noise,ambiguous}`, `quality_score∈[0..1]`, `retries[qN]`, `hold`, `optOut`, `snoozeUntil`, доставка (`lastOutboundId,lastStatus,lastStatusAt,nudgeCount[qN]`), `parkingLot[]`

### Справочники/словари

`willCoverLater(topic)→step`, синонимы/негации код-слов, синонимы опций, каталоги `niche/budget_bucket/channel`, маркеры важности вопросов (`high/low`)

### Классификация входящих

1. Код-слова (приоритет): `operator`→HITL/hold, `stop`→optOut, `pause`→snooze+hold, `start`→resume
2. Если не код-слово → `intent∈{funnel_answer,question,smalltalk,noise}`
3. Для `funnel_answer` → определить `validity, quality_score`
4. Для `question` → `relevance to_funnel/off_funnel`, `willCoverLater`, `importance`

### Матрица решений по шагам

**Стандартные вопросы:**
- Q1: "Какая у вас основная боль/задача?" / "Какая ниша?"
- Q2: "Какой у вас месячный бюджет на маркетинг?"
- Q3: "Какой основной канал продаж сейчас?"
- Q4: "Готовы ли увеличить бюджет при росте продаж?"
- Q5: "Какой ключевой показатель (KPI) станет для вас мерилом успешного масштабирования бизнеса?"

### Анти-чушь и парсинг по шагам

- **Q1 (free_text: niche,pain):** pain ≥4 значимых слов; niche ∈ каталоге; **близость ≥0.7**
- **Q2 (free_text: budget):** извлечь число/диапазон → `bucket∈{<1k,1–5k,5–20k,20k+}`; **близость ≥0.7**
- **Q3 (choice):** сопоставление с опциями/синонимами **≥0.8**
- **Q4 (choice):** да/нет **≥0.8**
- Ретраи анти-чуши: 1 — репромпт; 2 — подсказки/кнопки; 3 — оператор

### Логика валидации ответов

**При валидном ответе (confidence ≥ 0.65):**
- CAPI → событие QUALIFY_Q{n}
- Custom Audiences → добавление в CA_QUAL_Q{n} и сегментные (бюджет/канал/ниша)
- CRM → обновление custom fields в контакте (cf_niche, cf_budget_month)
- БД → сохранение ответа и confidence score
- Переход к следующему шагу

**При шуме/неопределенности (confidence < 0.65):**
- Репромпт → до 3 попыток уточнения в рамках шага
- NOISE pixel → отдельный dataset для исключений
- Custom Audiences → CA_EXCLUDE_NOISE
- CRM → тег trash/ambiguous
- При превышении лимита → эскалация на оператора

### Эмиссии на **каждом валидном** ответе

**CAPI (пример):**

```json
{
  "event_name": "QUALIFY_Q{N}",
  "event_time": 1736123456,
  "event_id": "sess-123-q2-9a1c...",
  "action_source": "chat",
  "user_data": {
    "external_id": "<sha256(tenantId:subscriberId)>",
    "ph": "<sha256(msisdn)>"
  },
  "custom_data": {
    "step": "q2",
    "answer_raw": "около 2500$",
    "answer_parsed": {"budget_month":2500,"bucket":"1–5k"},
    "quality_score": 0.86,
    "validity": "valid",
    "session_id": "sess-123",
    "tenant_id": "tenant-1",
    "project_id": "project-1",
    "mapping_version": "1.0.0"
  }
}
```

**Аудитории:**
- `valid` → `CA_QUAL_{stepKey}` + ветвевые (`CA_NICHE_*`, `CA_BUDGET_*`, `CA_CHANNEL_*`, `CA_UPLIFT_{yes|no}`)
- `noise/ambiguous` → `CA_EXCLUDE_NOISE` **и/или** «теневой» пиксель/датасет **NEG**

**CRM:**
Upsert лида; обновление полей шага; комментарий `QUALIFY_{stepKey}` (raw/parsed/qs)

**Эмиссии по шагам — дайджест:**
- **Q1:** CAPI `QUALIFY_Q1{niche,pain}`; Aud `CA_QUAL_Q1 + CA_NICHE_*`; CRM `cf_niche/cf_pain`; DB+outbox
- **Q2:** CAPI `QUALIFY_Q2{budget_month,bucket}`; Aud `CA_QUAL_Q2 + CA_BUDGET_*`; CRM `cf_budget_*`; DB
- **Q3:** CAPI `QUALIFY_Q3{main_traffic_channel}`; Aud `CA_QUAL_Q3 + CA_CHANNEL_*`; CRM `cf_main_channel`; DB
- **Q4:** CAPI `QUALIFY_Q4{uplift}`; Aud `CA_QUAL_Q4 + CA_UPLIFT_{yes|no}`; CRM `cf_budget_uplift_ready`; DB

### Политики для noise/ambiguous

- **S1 (жёсткая):** БД + `CA_EXCLUDE_NOISE`, без CAPI/CRM
- **S2 (теневая):** **NEG** CAPI в отдельный пиксель/датасет `NEG_QUALIFY_{stepKey}`; CRM — нет
- **S3 (смешанная):** CRM-заметка (без стадии) + `CA_EXCLUDE_NOISE`; CAPI — нет
**Persist-all:** сохраняем **всё** (для аналитики и **lookalike-исключений**)

### Идемпотентность/обновления/hold

- Первый `valid` по шагу → штатный пакет эмиссий
- Повторный `valid` → `QUALIFY_{stepKey}_UPDATE` (или тот же `event_id`)
- `ambiguous/noise` **никогда** не апдейтит `valid`-ивент
- `hold=true` (оператор/пауза): вопросы не шлём и нуджи off; **валидные входящие ответы сохраняем и эмитим**
- `optOut=true`: все исходящие off; валидные входящие — **только БД** (без внешних эмиссий)

### StepValidator (Custom Function в Chatflow)

```javascript
async function validateStep(input, options) {
  const { step, answer, session } = input
  
  // Вызов LLM для классификации
  const validation = await classifyAnswer({
    step,
    question: QUESTIONS[step],
    answer,
    context: session.variables
  })
  
  // Определение validity
  let validity = 'ambiguous'
  if (validation.confidence >= 0.65) {
    validity = 'valid'
  } else if (validation.confidence < 0.3) {
    validity = 'noise'
  }
  
  // Сохранение в БД с идемпотентностью
  const eventId = generateEventId(session.id, step, answer)
  await db.funnel_answers.upsert({
    where: { event_id: eventId },
    create: {
      tenant_id: session.tenant_id,
      subscriber_id: session.subscriber_id,
      session_id: session.id,
      step,
      answer_raw: answer,
      answer_parsed: validation.parsed,
      validity,
      quality_score: validation.confidence,
      event_id: eventId,
      version: 1
    }
  })
  
  // Эмиссия событий для valid
  if (validity === 'valid') {
    await emitToAllChannels({
      event_type: `QUALIFY_${step}`,
      event_id: eventId,
      session,
      answer: validation.parsed
    })
  }
  
  return {
    validity,
    confidence: validation.confidence,
    parsed: validation.parsed,
    nextAction: determineNextAction(validity, session)
  }
}
```

---

## ⏱ НУДЖИ И НАПОМИНАНИЯ - STATE MACHINE

### Автоматические триггеры

- `delivered` (не прочитано): `T+15м` → до `24ч`, **лимит 2**
- `read` (прочитано, нет ответа): `T+2м` → до `20м`, **лимит 2**
- `failed`: сразу; затем `snooze 24ч` + алерт оператору

**Правила:** мин. интервал ≥2 мин; при `hold/optOut/snooze` — OFF; учитывать рабочие окна

### State Machine для сообщений

```
PENDING → SENT → DELIVERED → READ → REPLIED
         ↓       ↓           ↓       ↓
       FAILED  NOT_DELIVERED IGNORED PROCESSED
         ↓                    ↓
      RETRY/ESCALATE      NUDGE_SENT
```

### Конфигурация нуджей

```json
{
  "nudge_rules": [
    {
      "trigger": "message_sent_not_read",
      "delay": "15m",
      "max_attempts": 2,
      "window": "24h",
      "action": {
        "type": "reminder",
        "template": "nudge_not_read",
        "text": "👋 Видимо вы заняты. Это сообщение будет ждать вашего ответа"
      }
    },
    {
      "trigger": "message_read_no_reply", 
      "delay": "2m",
      "max_attempts": 2,
      "window": "20m",
      "action": {
        "type": "follow_up",
        "template": "nudge_no_reply",
        "text": "Вы здесь? Нужна помощь с ответом?"
      }
    },
    {
      "trigger": "qualification_abandoned",
      "delay": "1h",
      "max_attempts": 1,
      "window": "24h",
      "action": {
        "type": "re_engage",
        "template": "come_back_template",
        "switch_to_template": true
      }
    }
  ]
}
```

### Различие: Репромпты vs Нуджи

**Репромпты** (в рамках шага):
- Счетчик в Flowise Variables
- До 3 попыток сразу в диалоге
- При неправильном ответе на конкретный вопрос

**Нуджи** (между сообщениями):
- Счетчик в бэкенд nudge-service
- Отложенные напоминания по таймингу
- При отсутствии реакции на сообщение

---

## 🙋 ВОПРОСЫ И ЭСКАЛАЦИЯ

### Вопросы (в любой точке)

- `to_funnel & будет` → назвать шаг, вернуться к `askedId`
- `to_funnel & не будет` → `importance`: low→parking; high→KB (≥0.7) или оператор
- `off_funnel` — аналогично важности; после — «Возвращаемся к {askedId}»

### Мульти-интент

Если в одном сообщении ответ+вопрос: 
1. сохранить/эмитить valid
2. обработать вопрос (§10)
3. вернуться к текущему шагу

### Паркинг/доработка после квалификации

В `CONTENT/END`: сначала high (KB≥0.7/оператор), затем low батч-ответом

### Эскалация (HITL)

**Триггеры:** `retries≥3`, `KB.conf<0.7` при high, токсичность/жалоба/риски, явный «оператор»

**Пакет оператору:** `stage/askedId`, последние N сообщений, `importance/relevance/confidence`, извлечённые поля, обещания

**После:** «Продолжаем с qN»

---

## 💾 ПАМЯТЬ: КАК И ЧТО ИСПОЛЬЗУЕМ

### Слой памяти

**Короткая (runtime, 24–72ч) → Redis**
- **Что кладём:** история для LLM, последние N сообщений, быстрые флаги (`retries/hold/optOut`), контекст шага
- **Зачем:** дёшево/быстро, низкая латентность, TTL, ключи по `sessionId`
- **Где в Flowise:**
  - **Redis Cache** — кэш ответов LLM
  - **Redis-Backed Chat Memory** — диалог для агента

**Долгая (аудит/аналитика/поиск) → Postgres (+pgvector по необходимости)**
- **Что кладём:** `messages_raw/meta/links`, `funnel_answers`, `events_outbox`, CTWA-метки, заказы/оплаты, «вечная» память фактов
- **Зачем:** надёжность, SQL, отчёты, восстановление, RAG (через `pgvector`)

**(Опц.) Векторный слой → pgvector** — семантический поиск фактов/предпочтений

### Конкретные узлы/параметры в Chatflow

**Redis Cache**
- Credential: ваш Redis/Valkey
- `TTL: 600–3600s`
- `Key prefix: ws:{{$vars.workspaceId}}:cache:`
- Подключать к ChatOpenAI/модели как Cache

**Redis-Backed Chat Memory**
- Credential: тот же Redis
- `Session Id: {{$vars.sessionId}}`
- `Prefix: ws:{{$vars.workspaceId}}:chat:`
- **Window: 8–20 сообщений** *(это **пример**, не согласованная константа)*
- Подключать в Tool/Conversational Agent → Memory

**Переменные (Set/Get Variable)**
- Инициализируем: `workspaceId, tenantId, sessionId, stepKey, retries`

**Долгая память** — делает **бэкенд** по вебхукам (Gupshup/Stripe/CRM) в Postgres (+ индекс в pgvector). В Chatflow отдельный узел не нужен

### Политики и изоляция

**Ключи:**
- `ws:{workspaceId}:chat:{sessionId}` — история
- `ws:{workspaceId}:cache:{hash(prompt)}` — кэш LLM
- **Redis key prefix:** `org:{tenantId}:prj:{projectId}:sess:{sessionId}`
- **pgvector namespace:** фильтр `WHERE tenant_id=? AND project_id=?` (+индексы)

**TTL:** чат-история 24–72ч; кэш LLM 10–60 мин (диалоги) / дольше на статике

**Очистка:** при `optOut/stop` — сразу чистим `chat:{sessionId}`

**Суммаризация (опц.):** воркер сворачивает длинные чаты в Postgres; в Redis держим `summary+tail`

---

## 📡 МАРШРУТИЗАЦИЯ И ОНБОРДИНГ

### Вход WhatsApp ↔ Gupshup ↔ Backend ↔ Agentflow ↔ Chatflow

**«Главный» флоу и старт:**
- Вход **всегда** в бэкенд (webhook Gupshup)
- Старт **всегда** в **Agentflow**

### Реестр маршрутизации (БД `routing`)

**Поля:** `tenant_id, bm_id, wa_app_id(или wa_phone), entry_type(default|ctwa|keyword), entry_value(ad_id|adset_id|campaign_id|keyword), agentflow_id, chatflow_id_default(опц.), status`

**Уникальность:** `(wa_app_id, entry_type, entry_value)` — `UNIQUE`

**Логика:** по метаданным входящего (номер/wa_app_id + CTWA-теги) бек находит строку и получает `agentflow_id` → вызывает Agentflow `/predict`

### Как бек понимает «это WhatsApp»

Из webhook: `provider="gupshup"`, `wa_app_id/phone`, тип события (`message`, `template`, `status: delivered|read|failed`). Эти поля идут в `messages_meta` и в роутинг

### Несколько WA-номеров в одном BM + онбординг

- Мастер-онбординг (в админке) и нода «Onboarding» для записи в БД
- Онбординг делает: список WA-аккаунтов/номеров; выбор «привязать существующий»/«завести новый»; выбор главного `Agentflow` (и опц. дефолтного `Chatflow`); запись в `routing`
- Ограничение: `wa_app_id` **UNIQUE** в активных связках (повторная привязка запрещена)

### Кто «кому звонит»

- Бек **никому внутри канвы не звонит** — только `/predict` у **Agentflow**
- Очерёдность/step/пороги — **в Chatflow** (Variables + Memory)
- Бек передаёт контекст: `tenantId, orgId, workspaceId, sessionId, wa_app_id/phone, ctwa_*`
- Дальше Agentflow → Execute Flow(Chatflow); внутри Chatflow тулами выполняются WA-отправки/эмиссии

### «Страница Sequences» и «нода WhatsApp»

- Делаем внешнюю страницу **Sequences** (конструктор сообщений/кнопок/листов с версионированием)
- В Flowise — **одна** нода **WA Send (Gupshup)** (Custom Tool), на вход подаётся `sequence_id`/`message_id`
- Нода сама тянет из БД и отправляет пакет
- Порядок сообщений в пакете решает бек (по `sequence_id`), на канве порядок управляет поток

### Где живёт «нода WhatsApp»

- Отправка (text/template/buttons/list/…) — **в Chatflow** (через Tool Agent → WA Send)
- Короткие ack на код-слова до входа в сценарий — можно и из **Agentflow** той же нодой
- «Слушатель» входящих — **только бек** (webhook)

### «Нода постоянного прослушивания» и классификация

- Слушатель — бек
- Классификация — внутри **Chatflow** (Tool Agent + промпт/правила) или отдельная Intent/Moderation-нода
- Статусы доставки (delivered/read/failed) ловит бек и триггерит нуджи (через свой планировщик)
- В Flowise — только команда `/nudge/schedule`

### Идеология

- Start — только в **Agentflow** (код-слова/глобальные условия) → Execute Flow(Chatflow)
- В **Chatflow** — бизнес-логика шагов, валидатор, инструменты (WA Send, CAPI, Audiences, CRM, Stripe, Nudge, Operator)
- Память и шаги — в Chatflow (Redis-Backed Chat Memory + Variables)
- Мультиплатформа (WA/TG/IG/FBM): одинаковая схема, меняется только транспортный тул

### Множественные сценарии/каналы

- Можно держать несколько Chatflow
- В Agentflow через Condition/ветвления выбирать, какой Chatflow дернуть (по номеру/CTWA/keyword)
- Оператор (HITL) — в Agentflow (ветка operator) + нода «Создать тикет» + Human Input

### Что хранить и где (минимум)

**БД:** `routing`, `messages_raw/meta`, `funnel_answers`, `events_outbox`, `sequences`, `wa_accounts`, `projects`

**Redis:** `org:{orgId}:ws:{wsId}:session:{sessionId}` — история/состояние

**Variables (Workspace):** `qual_manifest`, `policy` (пороги/ретраи/нуджи), `ids` (пиксель/аудитории/CRM-маппинг), `texts_common`

---

## 📲 НОДА «WA SEND (GUPSHUP)» — «НАЧИНКА»

### Внутренности ноды (6 групп полей)

**1. Адресация**
- `to` (E.164, +7999… или `contact_id` из CRM)
- `wa_app_id` (подключённый WA-аккаунт Gupshup)
- `project_id / tenant_id / session_id` (маршрутизация/логи)
- `idempotency_key` *(по умолчанию: `sessionId-msg-<hash(message_id+vars)>`)*

**2. Что отправить (контент)**
- `mode: message | sequence`
- `message_id | sequence_id` (ссылки на библиотеку сообщений)
- `vars {}` (подстановки, напр. `{name:"Иван", budget:"5k"}`)

**(Опц.) Inline для разовых случаев:**
- `type:` `text | template | interactive_buttons | interactive_list | media_image | media_video | media_audio | document | location | contact`
- `payload` по типу:
  - `text: { text }`
  - `interactive_buttons: { text, buttons:[{id,title}] }`
  - `interactive_list: { header?, body, sections:[{title, rows:[{id,title,description?}]}] }`
  - `media_*: { url | file_id, caption? }`
  - `template: { template_name, language, components:[…], variables:{…} }`
  - `location: { lat, lng, name?, address? }`
  - `contact: { vcard | fields… }`

**3. Режим доставки (24 часа)**
- `delivery_mode:` `auto | session_only | template_only`
- `fallback_template?` (имя шаблона за пределами 24ч)
- `typing_simulation? on|off, delay_ms?`

**4. Политики и валидация**
- `validate: on|off` (лимиты WA: длины/кнопки/типы)
- `block_if_optout: on|off`
- `respect_quiet_hours: on|off, window:{start,end,tz}`
- `rate_limit_tag?` (троттлинг/BSP лимиты)

**5. Логирование и дебаг**
- `trace_id` *(авто: `sessionId:stepKey:ts`)*
- `dry_run: on|off` — не шлёт, валидирует и отдаёт сгенерированный payload
- `return_raw: on|off` — вернуть сырой ответ Gupshup

**6. Выходы ноды (outputs)**
- `status: queued | sent | failed`
- `outbound_id` (внутренний id)
- `provider_message_id` (id у Gupshup/WA)
- `idempotency_key` (фактически применённый)
- `error: code, message, retry_after?`
- `meta: delivery_mode_used, template_used?, media_info?`

**Мини-пример вызова (ссылкой на библиотеку):**
```
to=+79991234567
wa_app_id=gs_app_01
mode=message
message_id=msg_q2@v7
vars={ "name":"Иван", "budget":"5k" }
delivery_mode=auto
idempotency_key=session123-msg-7b1c…
```

---

## 🏗️ ПРОЕКТЫ И РОУТИНГ

### Минимальная схема + A/B (позже)

**Модель данных (минимум):**
- `projects(id, tenant_id, name, slug, status, default_agentflow_id, default_chatflow_id, qual_manifest_id, policy_id, created_at)`
- `wa_accounts(id, bm_id, wa_app_id, phone_e164, status, created_at)`
- `routing(id, tenant_id, project_id, wa_app_id, entry_type(default|ctwa|keyword), entry_value, agentflow_id, chatflow_id, status, created_at)`
  — `UNIQUE(wa_app_id, entry_type, entry_value)` (исключает двойную привязку)

### Роутинг входящих

1. Gupshup webhook → извлечь `wa_app_id/phone, ctwa_*, message/text`
2. Найти в `routing` по `(wa_app_id, entry_type, entry_value)` (приоритет: `ctwa:ad|adset|campaign` → `keyword` → `default`)
3. Получить `project_id, agentflow_id, chatflow_id`; вызвать Agentflow `/predict` (`question`, Vars: `projectId, tenantId, wa_app_id, sessionId, ctwa_*`)
4. Agentflow → Execute Flow(Chatflow)

### Онбординг номеров

UI: выбрать Проект → WA номер (`wa_accounts`) → создать `routing(default, '*')`. Если `wa_app_id` уже привязан как `default` в активном проекте — запретить

### События/эмиссии: указывать `project_id`

CAPI/custom_data, Audiences payload, CRM note/fields, DB (`funnel_answers`, `events_outbox`) — всегда добавлять `project_id`

### A/B (позже, без ломки)

- `routing_variants{project_id, variant_code, weight, agentflow_id, chatflow_id}`
- Sticky-бакет: хеш телефона → `variant_code`
- Вызов разных Chatflow **или** одного Chatflow с разными `qual_manifest_id/policy_id`

### Flowise: папки/теги/переменные (нейминг) + изоляция

**Теги/нейминг:**
- **Agentflow:** `[PRJ:slug] Agent START`
- **Chatflow:** `[PRJ:slug] Chat QUALIFIER`

**Workspace/Prediction Variables:** `projectId, tenantId, wa_app_id, sessionId, ctwa, qual_manifest_id, policy_id`

---

## 🔗 ИНТЕГРАЦИИ (продолжение)

### CAPI - Facebook Conversions API

**События квалификации:**
```json
{
  "data": [
    {
      "event_name": "QUALIFY_Q2",
      "event_time": 1736123456,
      "event_id": "sess-123-q2-9a1c...",
      "event_source_url": "https://wa.me/79991234567",
      "action_source": "chat",
      "user_data": {
        "external_id": "<sha256(tenantId:subscriberId)>",
        "ph": "<sha256(msisdn)>"
      },
      "custom_data": {
        "step": "q2",
        "answer_raw": "около 2500$",
        "answer_parsed": {"budget_month":2500,"bucket":"1–5k"},
        "quality_score": 0.86,
        "validity": "valid",
        "session_id": "sess-123",
        "tenant_id": "tenant-1",
        "project_id": "project-1",
        "mapping_version": "1.0.0"
      }
    }
  ]
}
```

### Custom Audiences - Рекламные аудитории

**Базовые аудитории:**
- `A1_STARTED_NOT_FINISHED` - Начали но не завершили воронку
- `A2_QUALIFIED` - Прошли квалификацию  
- `A3_REJECTED` - Отказники и шум

**Сегментные аудитории:**
- `CA_QUAL_Q1`, `CA_QUAL_Q2` - По шагам квалификации
- `CA_BUDGET_10K_50K`, `CA_BUDGET_50K_PLUS` - По бюджету
- `CA_CHANNEL_INSTAGRAM` - По каналу
- `CA_PURCHASED` - Совершили покупку
- `CA_EXCLUDE_NOISE` - Исключения шума

### CRM - Управление клиентами

**Маппинг полей (Kommo):**
```json
{
  "contact": {
    "name": "{{profileName || waPhone}}",
    "phone": "{{waPhone}}",
    "custom_fields": {
      "641289": "{{ctwa.campaign_id}}",
      "641290": "{{ctwa.adset_id}}",
      "641291": "{{ctwa.ad_id}}",
      "641292": "{{answers.Q1.niche}}",
      "641293": "{{answers.Q2.budget}}",
      "641294": "{{answers.Q3.channel}}"
    }
  }
}
```

**Триггеры синхронизации:**
| Событие | Действие в CRM |
|---------|---------------|
| CTWA старт | Создать контакт и сделку |
| Валидный Qn | Обновить поля + коммент |
| Квалификация пройдена | Сменить стадию |
| Оплата получена | Сменить на "Оплачено" |

### Stripe - Платежная система

**Создание платежной ссылки:**
```javascript
const paymentLink = await stripe.paymentLinks.create({
  line_items: [{
    price: 'price_ABC123',
    quantity: 1
  }],
  metadata: {
    session_id: 'session_123',
    subscriber_id: 'sub_789',
    product_id: 'SKU_001',
    source: 'whatsapp_funnel'
  }
})
```

**Обработка webhook событий:**
- `checkout.session.completed` → Purchase событие в CAPI
- Обновление аудиторий → перенос в CA_PURCHASED
- Обновление CRM → смена стадии на "Оплачено"
- WhatsApp подтверждение → template с чеком

---

## 📐 СТРУКТУРА БАЗЫ ДАННЫХ

### Основные таблицы

**`subscribers`**
```sql
CREATE TABLE subscribers (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR NOT NULL,
  wa_phone VARCHAR UNIQUE NOT NULL,
  phone_hash VARCHAR NOT NULL,
  profile_name VARCHAR,
  first_seen_at TIMESTAMP,
  last_inbound_at TIMESTAMP,
  status VARCHAR DEFAULT 'active',
  optout BOOLEAN DEFAULT false,
  snoozed_until TIMESTAMP,
  conversation_id VARCHAR,
  conversation_expires_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**`funnel_answers`**
```sql
CREATE TABLE funnel_answers (
  id UUID PRIMARY KEY,
  tenant_id BIGINT NOT NULL,
  subscriber_id BIGINT NOT NULL,
  session_id TEXT NOT NULL,
  step_key TEXT NOT NULL,
  validity TEXT NOT NULL CHECK (validity IN ('valid','noise','ambiguous')),
  quality_score NUMERIC(3,2) NOT NULL,
  answer_raw TEXT NOT NULL,
  answer_parsed JSONB NOT NULL DEFAULT '{}'::jsonb,
  version INT NOT NULL DEFAULT 1,
  event_id TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ,
  PRIMARY KEY (tenant_id, subscriber_id, session_id, step_key)
);
```

**`events_outbox`**
```sql
CREATE TABLE events_outbox (
  id BIGSERIAL PRIMARY KEY,
  event_id TEXT NOT NULL UNIQUE,
  channel TEXT NOT NULL CHECK (channel IN ('CAPI','AUDIENCE','CRM')),
  payload JSONB NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending','sent','failed','on_hold')),
  retries INT NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  last_error TEXT
);
```

**`messages_raw`**
```sql
CREATE TABLE messages_raw (
  id UUID PRIMARY KEY,
  message_id VARCHAR UNIQUE NOT NULL,
  session_id VARCHAR,
  direction VARCHAR NOT NULL, -- inbound/outbound
  type VARCHAR NOT NULL, -- text/image/button/list/template
  payload JSONB NOT NULL,
  status VARCHAR, -- sent/delivered/read/failed
  parent_id UUID, -- ссылка на предыдущее
  parent_type VARCHAR, -- step/message/none
  created_at TIMESTAMP DEFAULT NOW(),
  delivered_at TIMESTAMP,
  read_at TIMESTAMP,
  replied_at TIMESTAMP
);
```

**Транзакция:** `funnel_answers` + пачка `events_outbox` → коммит. Worker: идемпотентные отправки, backoff `1→2→5→15` мин

---

## 📊 МЕТРИКИ И АЛЕРТЫ

По шагам: доля `valid/noise/ambiguous`, ср. `quality_score`, % обновлений

Коннекторы: latency/ошибки/повторы

Качество трафика: доля `CA_EXCLUDE_NOISE`

HITL: SLA, время возврата

Нуджи: CTR/ответы/эскалации

Реабилитация: % `NOISE→GOOD`

---

## 🔢 ПОРОГОВЫЕ ЗНАЧЕНИЯ

- Free-text ≥0.7
- Choice ≥0.8
- KB-ответ ≥0.7
- Анти-чушь ретраи `1→2→3(оператор)`
- Нуджи `read:2→20м(2)`, `delivered:15м→24ч(2)`

---

## 🔐 НАДЁЖНОСТЬ И ПРИВАТНОСТЬ

- Идемпотентность (`event_id` + уникальный индекс в outbox)
- Backoff-ретраи
- Hash-PII (sha256) для `external_id/ph`
- Аудит решений (valid/noise/ambiguous, нуджи, код-слова, эскалации)

---

## 🔄 МИНИ-FSM (упрощённо)

```
Qn.wait → (код-слово) → hold/optOut/snooze
Qn.wait → (valid) → emit(Qn) → Q{n+1}.wait|CONTENT
Qn.wait → (ambiguous/noise) → retry/assist → Qn.wait|operator
any → (question) → route(§10) → return(Qn.wait)
CONTENT → END
```

---

## 🎮 AGENTFLOW КОНФИГУРАЦИЯ

### WhatsApp Send Node - Универсальная нода

**Конфигурация ноды:**
```javascript
{
  name: "WhatsAppSend_Agentflow",
  type: "WhatsAppSend",
  category: "Agent Flows",
  version: "1.0.0",
  description: "Универсальная отправка в WhatsApp",
  inputs: [
    {
      label: "Message Type",
      name: "messageType",
      type: "options",
      options: [
        { value: "text", label: "Text" },
        { value: "image", label: "Image" },
        { value: "button", label: "Interactive Buttons" },
        { value: "list", label: "Interactive List" },
        { value: "template", label: "Template" }
      ],
      default: "text"
    },
    {
      label: "To",
      name: "to",
      type: "string",
      placeholder: "+79991234567",
      acceptVariable: true
    },
    {
      label: "Session ID", 
      name: "sessionId",
      type: "string",
      acceptVariable: true
    },
    {
      label: "Idempotency Key",
      name: "idempotencyKey",
      type: "string",
      acceptVariable: true,
      optional: true
    }
  ]
}
```

**Автоматическое переключение session/template:**
- В пределах 24h окна → session message
- После истечения 24h → template message
- При failed доставке session → retry как template

---

## 🏁 КРАЕВЫЕ СЛУЧАИ

- Длинные полотна → суммаризация/допрос
- Голос/фото → ASR/OCR→те же правила
- Дубликаты → идемпотентность `lastOutboundId/event_id`
- `hold=true` — валидные ответы **не теряем** и **эмитим**
- `optOut` — входящие **только БД**, наружу **не** шлём

---

## ⚙️ ГОТОВНОСТЬ К ВНЕДРЕНИЮ

- Карта `willCoverLater`
- Словари/негации
- Пороги/окна/лимиты
- БД/воркеры/outbox
- Аудитории `CA_QUAL_*` / `CA_EXCLUDE_NOISE`
- Мониторинг/алерты
- Тесты: идемпотентность/ретраи/noise/обновления/код-слова/нуджи

---

## 🚫 ТИПОВЫЕ ПРОБЛЕМЫ И РЕШЕНИЯ

### Потеря контекста при перезапуске Flowise
**Симптом:** Variables (stepKey, answers, retries) сбрасываются в null/undefined
**Причина:** Flowise не персистит Variables между рестартами
**Решение:**
- Сохранение критичных переменных в БД через Custom Function после каждого шага
- Восстановление состояния из БД при инициализации сессии
- Fallback: начало с Q1 при потере состояния

### Дублирование событий CAPI
**Симптом:** Одно событие QUALIFY_Q{n} отправляется несколько раз
**Причина:** Retry логика без идемпотентности  
**Решение:**
- Уникальный event_id = `{sessionId}-q{n}-{hash(answer_raw)}`
- Проверка в outbox перед эмиссией: `SELECT 1 FROM events_outbox WHERE event_id = ?`
- Constraint UNIQUE(event_id) в БД

### Истечение 24h окна WhatsApp
**Симптом:** Ошибка 403/400 при отправке session message через 24 часа
**Причина:** WhatsApp Policy - session messages только в течение 24h после последнего входящего
**Решение:**
- Автопереключение на template message в WA Send node
- Проверка `conversation_expires_at` перед отправкой
- Fallback templates для каждого типа сообщения

### Низкий confidence score в LLM
**Симптом:** Много ответов попадает в категорию ambiguous (0.3 ≤ confidence < 0.65)
**Причина:** Неоднозначные вопросы, короткие ответы, отсутствие контекста
**Решение:**
- Улучшение промптов с примерами валидных ответов
- Использование контекста предыдущих ответов для классификации
- Clarification prompts: "Уточните, пожалуйста..."
- Снижение порога confidence для определенных шагов

### Рассинхронизация CRM
**Симптом:** Поля в CRM не обновляются или обновляются с задержкой
**Причина:** Webhook пропущен, timeout, неправильный mapping
**Решение:**
- Retry механизм для CRM webhooks с exponential backoff
- Периодическая синхронизация (каждые 15 минут)
- Логирование всех CRM операций для отладки
- Fallback на прямые API вызовы при неудаче webhook

### Перегрузка при высоком трафике
**Симптом:** Медленная обработка сообщений, таймауты LLM
**Причина:** Недостаточно ресурсов, блокирующие операции
**Решение:**
- Горизонтальное масштабирование Flowise через Docker replicas
- Асинхронная обработка через Queue (Bull/Agenda)
- Кэширование LLM ответов для повторяющихся паттернов
- Circuit breaker для внешних API (OpenAI, Gupshup)

### Некорректное извлечение CTWA данных
**Симптом:** campaign_id/ad_id приходят пустыми или с мусорными значениями
**Причина:** Различия в форматах webhook от разных провайдеров
**Решение:**
- Нормализация в parser слое: строгая валидация по регексам
- Fallback на utm_* из referral.source_url при отсутствии прямых полей
- Логирование raw webhook для анализа новых форматов
- Мониторинг процента успешного извлечения CTWA

---

## 🚦 БИЗНЕС-ПРАВИЛА И ИНВАРИАНТЫ

### Жесткие инварианты

**Правило 1: Немедленная эмиссия**
```
ЕСЛИ ответ валидный (confidence ≥ 0.65)
ТО СРАЗУ отправить в CAPI + Audiences + CRM + БД
БЕЗ ОЖИДАНИЯ завершения воронки
```

**Правило 2: Разделение потоков**
```
GOOD поток → основной pixel/dataset
NOISE поток → отдельный pixel/dataset или негативный пиксель/датасет NEG
НИКОГДА не смешивать потоки
```

**Правило 3: Приоритет код-слов**
```
Код-слова обрабатываются ДО любой другой логики
stop/pause/operator (синонимы) имеют абсолютный приоритет
```

**Правило 4: Идемпотентность**
```
Каждое событие имеет уникальный event_id
Повторная обработка возвращает тот же результат
```

**Правило 5: CRM стадии**
```
НЕ двигаем стадии автоматически
КРОМЕ явных правил (квалификация пройдена)
```

### Ключевые принципы

1. **Порог уверенности**: 0.65 (единый для валидации и ответов агента)
2. **Репромпты**: до 3 попыток сразу в диалоге (счетчик per step)
3. **Нуджи**: до 2 напоминаний с отложенным таймингом (счетчик per case)
4. **Стадии CRM**: движение только по явным правилам (не автоматически)
5. **Эмиссии**: всегда через бэкенд с outbox-паттерном
6. **Потоки данных**: GOOD и NOISE строго разделены (разные pixels)
7. **24h окно**: автопереключение на template messages при истечении
8. **Идемпотентность**: уникальный event_id для каждого действия
9. **Persist-all**: сохраняем **все** ответы/шумы в БД для анализа

---

## 🔄 ПРОЦЕССЫ И АВТОМАТИЗАЦИЯ

### Qualification Pipeline

```mermaid
graph TD
    A[CTWA Click] --> B[Create Session]
    B --> C[Send Q1]
    C --> D{Validate Answer}
    D -->|Valid| E[Emit CAPI/CRM/CA]
    D -->|Noise| F[Retry/Escalate]
    D -->|Ambiguous| G[Clarify]
    E --> H[<!-- filepath: /home/projects/new-flowise/docs/my-docs/about-arriwa-project/arriwa-technical-v1-updated.md -->
# 🚀 Arriwa - Техническая документация платформы

> **📎 Связанные документы:**
> - [Webhook Payload Specification](../incoming-webhooks/webhook-payload-specification.md) - RAW формат входящих webhook
> - [Subscriber Data Structure](../subscriber/exact-subscriber-data-structure-v2.md) - модель данных подписчика в БД
> - [WhatsApp Message Types](../wa-types-message.md) - полный список типов сообщений
> - [Project Description](../project-description/description.md) - концептуальный обзор ОБЩИЙ ПУТЬ

---

## ✅ АРХИТЕКТУРА И КОМПОНЕНТЫ СИСТЕМЫ

### 📱 Точки входа

**`CTWA`** (Click to WhatsApp Ads)  
Реклама Facebook/Instagram с переходом в WhatsApp

**`Direct Message`**  
Прямое сообщение без рекламного источника

**`QR Code`**  
Сканирование QR-кода с переходом в чат

**`Link Share`**  
Переход по wa.me ссылке

### 🎯 Этапы воронки

**`Qualification Steps`** (Q1...Qn)  
Пошаговая квалификация лида

**`Content Funnel`**  
Обучающий контент после квалификации

**`Product Showcase`**  
Демонстрация товаров/услуг

**`Payment Flow`**  
Процесс оплаты через Stripe

### 🤖 Классификация ответов

**`valid`**  
Валидный ответ на вопрос воронки (confidence ≥ 0.65)

**`noise`**  
Шум/бред/нерелевантный ответ (confidence < 0.3)

**`ambiguous`**  
Неоднозначный ответ требующий уточнения (0.3 ≤ confidence < 0.65)

### 🔄 Потоки данных

**`GOOD`**  
Основной поток валидных ответов

**`NOISE`**  
Поток невалидных/мусорных ответов

**`ESCALATION`**  
Эскалация на оператора

### 🛑 Код-слова управления

**`stop`** / **`стоп`** / **`unsubscribe`**  
Полная отписка от рассылок

**`pause`** / **`пауза`**  
Временная приостановка

**`start`** / **`старт`**  
Возобновление после паузы

**`operator`** / **`оператор`**  
Запрос живого оператора

### 📊 Интеграции

**`CAPI`** (Facebook Conversions API)  
События конверсий для оптимизации

**`Pixels`**  
для регистрации событий (GOOD и NOISE потоки на разные pixels/datasets)

**`Ads Manager`**  
для выгрузки статистики рекламных кампаний

**`Custom Audience`**  
для сохранения и расширения, LAL

**`E-commerce и WhatsApp catalog`**  
для отправки карточек товаров напрямую в чат WhatsApp

**`Ads Audience Transporter`**  
для экспорта аудиторий в Google, TikTok, LinkedIn

**`Enrichment`**  
для двухстороннего обогащения всех компонентов данными

**`Kommo CRM`**  
для управления статусами сделок

**`Stripe`**  
для проведения оплат

**`Vuexy`**  
frontend для работы с некоторыми модулями, отчетами и пр

---

## 🏢 Онбординг (из описания проекта)

Шаги (строго по описанию):
1. https://flows.rakhnianskii.com — первичная авторизация Facebook (public_profile, business_management, email)
2. /wa-onboarding — только этот раздел доступен
3. Прохождение онбординга (создание приложения, получение/привязка номера)
4. /credentials — разблокировка раздела для привязки CAPI, Pixels, аудитории, статистики (ads_read, ads_management)
5. Опционально авторизация для e-commerce каталога (commerce_account_read_orders, commerce_account_read_reports, commerce_account_read_settings, catalog_management)
6. Редирект на /chatflows

---

## 🔧 FLOWISE ОРКЕСТРАЦИЯ И ИСТОЧНИКИ ИСТИНЫ

### Agentflow → Chatflow архитектура

**Двухуровневая система:**
- **Agentflow** - управляющий слой, маршрутизация сообщений, агент/оркестратор (код-слова, старт/эскалации, вызов чат-сценария)
- **Chatflow** - исполнительный слой, LLM обработка и валидация, детерминированные шаги квалификации (валидатор/парсинг/ветвления/эмиссии)

**Формат sessionId (универсальный):**
```
{source}_{details}_{subscriber_id}_{timestamp}
```

Примеры:
- **CTWA**: `ctwa_fb_{subscriber_id}_{timestamp}`
- **QR**: `qr_{product_id}_{subscriber_id}_{timestamp}`  
- **Direct**: `direct_{subscriber_id}_{timestamp}`
- **Link**: `link_{utm_source}_{subscriber_id}_{timestamp}`
- **WhatsApp практика**: `<tenant>:<phone>:<flow>` — чтобы не смешивать проекты/воронки

### Согласование терминов: «сессия» (sessionId)

- **Определение:** ярлык, под которым Flowise/Arriwa хранит и читает историю *данного* разговора и состояние шага
- **Кто задаёт:** sessionId задаём **мы**; все Memory-ноды и кастом-тулы читают/пишут по нему
- **Где живёт:** в Arriwa — `Redis` (живая история окна/контекст), `PostgreSQL` (архив/состояние шагов), долговременная память через `pgvector`. Встроенная Buffer-память Flowise пишет в `chat_message`, но **не** считается источником истины
- **Передача в рантайме:** Prediction API → `overrideConfig.sessionId`; в узлах/тулах доступно как `$flow.sessionId`
- **Отличие от chatId:** chatId — внутренний ID беседы Flowise; sessionId — наш внешний/логический ID. Источник истины для склейки — `sessionId`

### Инварианты (зафиксировано)

- **Порог валидности:** `0.65` (единый для валидации ответов и логики агента)
- **Хранилища:** `Redis` (оперативная память/контекст + кэш), `PostgreSQL` (источник истины), `pgvector` (семантическая память/RAG)

### Двухуровневая проверка код-слов

**Уровень 1: Бэкенд (primary source of truth)**
- Проверяется на КАЖДОМ входящем сообщении до любой другой логики
- Нормализация в lowercase, поиск синонимов
- Немедленное исполнение действий (optOut, snooze, escalation)

**Уровень 2: Flowise Agentflow (страховочная сетка)**
- Дублирующая проверка для edge cases
- Срабатывает при ручном запуске или сбое webhook
- Синхронизация с бэкендом через API

### Источники истины и согласованность

**Код-слова**: бэкенд (единственный источник истины)
- Flowise: страховочная проверка для edge cases (ручной запуск, сбой webhook)

**Состояние шага**: Flowise Variables + БД (для восстановления)
- Variables: живое состояние в рамках сессии
- БД: персистентное состояние для восстановления после рестарта

**История чата**: Redis/Valkey (живая) + Postgres (архив)

**Счетчики репромптов**: Flowise Variables (в рамках шага)

**Счетчики нуджей**: бэкенд nudge-service (между сообщениями)

**CTWA данные**: БД (неизменяемые после создания)

### Идемпотентность через event_id

```javascript
// Генерация уникального event_id
event_id = `{sessionId}-q{n}-{hash(answer_raw)}`
// normalize: trim/lower/collapse spaces/remove urls/phones/emojis

// Проверка в outbox перед эмиссией
const exists = await db.events_outbox.findUnique({
  where: { event_id }
})

if (exists) {
  return exists.result // Возвращаем предыдущий результат
}
```

---

## 📋 ТОЧКИ ВХОДА И ИНИЦИАЛИЗАЦИЯ

### CTWA-клик и первое сообщение

**Последовательность действий:**
1. Пользователь кликает на рекламу Facebook/Instagram
2. Открывается WhatsApp с предзаполненным номером бизнеса
3. Пользователь отправляет первое сообщение (любой текст или шаблонное приветствие)

**Webhook Gupshup принимает сообщение:**
- **Извлекаются CTWA-параметры**: campaign_id, adset_id, ad_id, creative_id, utm_*, placement, publisher_platform, click_id
- **Создается уникальный sessionId**: `ctwa_fb_{subscriber_id}_{timestamp}`
- **Стартовый триггер**: любое первое входящее сообщение
- **Нормализация данных**: phone, name, language, country, timezone

### QR Code вход (альтернативная точка)

**QR содержит**: `wa.me/79991234567?text=QR_START_PROMO&product_id=SKU001&location_id=MSK`

**Извлекаются параметры**: product_id, location_id, promo_code

**Создается сессия**: `qr_{product_id}_{subscriber_id}_{timestamp}`

**Приоритет**: показ конкретного товара/услуги из product_id

### Постоянный мониторинг код-слов

**Проверяется на КАЖДОМ входящем сообщении до любой другой логики:**

| Триггер  | Варианты                              | Условие        | Действие                      | Ответ                  | Последствия    |
| -------- | ------------------------------------- | -------------- | ----------------------------- | ---------------------- | -------------- |
| Оператор | оператор/человек/support/agent/помощь | без «не» рядом | `hold=true`, HITL             | «Подключаю оператора…» | SLA, нуджи off |
| Стоп     | стоп/stop/unsubscribe/отписка         | однозначно     | `optOut=true`                 | «Остановлено…»         | Блок исходящих |
| Пауза    | пауза/позже/через X                   | —              | `snoozeUntil=ts`, `hold=true` | «Пауза до…»            | Ретраи off     |
| Старт    | старт/продолжим/продолжай             | из hold        | `hold=false`                  | «Продолжаем с qN»      | Таймеры on     |

### Атомарная инициализация (параллельно)

**CAPI Facebook**: событие Start/OptIn с хешированным номером телефона
**Custom Audiences**: добавление в аудиторию A1_STARTED_NOT_FINISHED
**Kommo CRM**: создается контакт и сделка в стадии "Новый"
**База данных**: записывается подписчик с CTWA-параметрами и raw referral блоком
**Менеджер**: назначается по правилам (round-robin/нагрузка/гео)
**Flowise**: запускается Agentflow с контекстом сессии

---

## 🎯 N-ШАГОВЫЙ КВАЛИФИКАТОР + «ЖИВОЙ» АГЕНТ + ДВА ПОТОКА (GOOD/NOISE)

### Назначение и рамки

- N шагов квалификации (не фикс.)
- На **каждом валидном ответе** — немедленные эмиссии: **CAPI / Аудитории / CRM / БД**
- **Шум/ambiguous** — не в основные каналы; либо в отдельные **noise-каналы/аудитории**/**негативный пиксель/датасет** (для исключений и аналитики)
- Параллельно всегда активен «живой» агент: анти-чушь, ответы на вопросы (БЗ), код-слова, нуджи, HITL
- **Persist-all:** сохраняем **все** ответы/шумы в БД (для анализа и **lookalike-исключений**)
- Идемпотентность, outbox, ретраи — **обязательно**

### Состояния и флаги (минимум)

`stage∈{Q1..Qn,CONTENT,END}`, `askedId`, `expectedType∈{free_text,choice}`, `answers{}`, `validity∈{valid,noise,ambiguous}`, `quality_score∈[0..1]`, `retries[qN]`, `hold`, `optOut`, `snoozeUntil`, доставка (`lastOutboundId,lastStatus,lastStatusAt,nudgeCount[qN]`), `parkingLot[]`

### Справочники/словари

`willCoverLater(topic)→step`, синонимы/негации код-слов, синонимы опций, каталоги `niche/budget_bucket/channel`, маркеры важности вопросов (`high/low`)

### Классификация входящих

1. Код-слова (приоритет): `operator`→HITL/hold, `stop`→optOut, `pause`→snooze+hold, `start`→resume
2. Если не код-слово → `intent∈{funnel_answer,question,smalltalk,noise}`
3. Для `funnel_answer` → определить `validity, quality_score`
4. Для `question` → `relevance to_funnel/off_funnel`, `willCoverLater`, `importance`

### Матрица решений по шагам

**Стандартные вопросы:**
- Q1: "Какая у вас основная боль/задача?" / "Какая ниша?"
- Q2: "Какой у вас месячный бюджет на маркетинг?"
- Q3: "Какой основной канал продаж сейчас?"
- Q4: "Готовы ли увеличить бюджет при росте продаж?"
- Q5: "Какой ключевой показатель (KPI) станет для вас мерилом успешного масштабирования бизнеса?"

### Анти-чушь и парсинг по шагам

- **Q1 (free_text: niche,pain):** pain ≥4 значимых слов; niche ∈ каталоге; **близость ≥0.7**
- **Q2 (free_text: budget):** извлечь число/диапазон → `bucket∈{<1k,1–5k,5–20k,20k+}`; **близость ≥0.7**
- **Q3 (choice):** сопоставление с опциями/синонимами **≥0.8**
- **Q4 (choice):** да/нет **≥0.8**
- Ретраи анти-чуши: 1 — репромпт; 2 — подсказки/кнопки; 3 — оператор

### Логика валидации ответов

**При валидном ответе (confidence ≥ 0.65):**
- CAPI → событие QUALIFY_Q{n}
- Custom Audiences → добавление в CA_QUAL_Q{n} и сегментные (бюджет/канал/ниша)
- CRM → обновление custom fields в контакте (cf_niche, cf_budget_month)
- БД → сохранение ответа и confidence score
- Переход к следующему шагу

**При шуме/неопределенности (confidence < 0.65):**
- Репромпт → до 3 попыток уточнения в рамках шага
- NOISE pixel → отдельный dataset для исключений
- Custom Audiences → CA_EXCLUDE_NOISE
- CRM → тег trash/ambiguous
- При превышении лимита → эскалация на оператора

### Эмиссии на **каждом валидном** ответе

**CAPI (пример):**

```json
{
  "event_name": "QUALIFY_Q{N}",
  "event_time": 1736123456,
  "event_id": "sess-123-q2-9a1c...",
  "action_source": "chat",
  "user_data": {
    "external_id": "<sha256(tenantId:subscriberId)>",
    "ph": "<sha256(msisdn)>"
  },
  "custom_data": {
    "step": "q2",
    "answer_raw": "около 2500$",
    "answer_parsed": {"budget_month":2500,"bucket":"1–5k"},
    "quality_score": 0.86,
    "validity": "valid",
    "session_id": "sess-123",
    "tenant_id": "tenant-1",
    "project_id": "project-1",
    "mapping_version": "1.0.0"
  }
}
```

**Аудитории:**
- `valid` → `CA_QUAL_{stepKey}` + ветвевые (`CA_NICHE_*`, `CA_BUDGET_*`, `CA_CHANNEL_*`, `CA_UPLIFT_{yes|no}`)
- `noise/ambiguous` → `CA_EXCLUDE_NOISE` **и/или** «теневой» пиксель/датасет **NEG**

**CRM:**
Upsert лида; обновление полей шага; комментарий `QUALIFY_{stepKey}` (raw/parsed/qs)

**Эмиссии по шагам — дайджест:**
- **Q1:** CAPI `QUALIFY_Q1{niche,pain}`; Aud `CA_QUAL_Q1 + CA_NICHE_*`; CRM `cf_niche/cf_pain`; DB+outbox
- **Q2:** CAPI `QUALIFY_Q2{budget_month,bucket}`; Aud `CA_QUAL_Q2 + CA_BUDGET_*`; CRM `cf_budget_*`; DB
- **Q3:** CAPI `QUALIFY_Q3{main_traffic_channel}`; Aud `CA_QUAL_Q3 + CA_CHANNEL_*`; CRM `cf_main_channel`; DB
- **Q4:** CAPI `QUALIFY_Q4{uplift}`; Aud `CA_QUAL_Q4 + CA_UPLIFT_{yes|no}`; CRM `cf_budget_uplift_ready`; DB

### Политики для noise/ambiguous

- **S1 (жёсткая):** БД + `CA_EXCLUDE_NOISE`, без CAPI/CRM
- **S2 (теневая):** **NEG** CAPI в отдельный пиксель/датасет `NEG_QUALIFY_{stepKey}`; CRM — нет
- **S3 (смешанная):** CRM-заметка (без стадии) + `CA_EXCLUDE_NOISE`; CAPI — нет
**Persist-all:** сохраняем **всё** (для аналитики и **lookalike-исключений**)

### Идемпотентность/обновления/hold

- Первый `valid` по шагу → штатный пакет эмиссий
- Повторный `valid` → `QUALIFY_{stepKey}_UPDATE` (или тот же `event_id`)
- `ambiguous/noise` **никогда** не апдейтит `valid`-ивент
- `hold=true` (оператор/пауза): вопросы не шлём и нуджи off; **валидные входящие ответы сохраняем и эмитим**
- `optOut=true`: все исходящие off; валидные входящие — **только БД** (без внешних эмиссий)

### StepValidator (Custom Function в Chatflow)

```javascript
async function validateStep(input, options) {
  const { step, answer, session } = input
  
  // Вызов LLM для классификации
  const validation = await classifyAnswer({
    step,
    question: QUESTIONS[step],
    answer,
    context: session.variables
  })
  
  // Определение validity
  let validity = 'ambiguous'
  if (validation.confidence >= 0.65) {
    validity = 'valid'
  } else if (validation.confidence < 0.3) {
    validity = 'noise'
  }
  
  // Сохранение в БД с идемпотентностью
  const eventId = generateEventId(session.id, step, answer)
  await db.funnel_answers.upsert({
    where: { event_id: eventId },
    create: {
      tenant_id: session.tenant_id,
      subscriber_id: session.subscriber_id,
      session_id: session.id,
      step,
      answer_raw: answer,
      answer_parsed: validation.parsed,
      validity,
      quality_score: validation.confidence,
      event_id: eventId,
      version: 1
    }
  })
  
  // Эмиссия событий для valid
  if (validity === 'valid') {
    await emitToAllChannels({
      event_type: `QUALIFY_${step}`,
      event_id: eventId,
      session,
      answer: validation.parsed
    })
  }
  
  return {
    validity,
    confidence: validation.confidence,
    parsed: validation.parsed,
    nextAction: determineNextAction(validity, session)
  }
}
```

---

## ⏱ НУДЖИ И НАПОМИНАНИЯ - STATE MACHINE

### Автоматические триггеры

- `delivered` (не прочитано): `T+15м` → до `24ч`, **лимит 2**
- `read` (прочитано, нет ответа): `T+2м` → до `20м`, **лимит 2**
- `failed`: сразу; затем `snooze 24ч` + алерт оператору

**Правила:** мин. интервал ≥2 мин; при `hold/optOut/snooze` — OFF; учитывать рабочие окна

### State Machine для сообщений

```
PENDING → SENT → DELIVERED → READ → REPLIED
         ↓       ↓           ↓       ↓
       FAILED  NOT_DELIVERED IGNORED PROCESSED
         ↓                    ↓
      RETRY/ESCALATE      NUDGE_SENT
```

### Конфигурация нуджей

```json
{
  "nudge_rules": [
    {
      "trigger": "message_sent_not_read",
      "delay": "15m",
      "max_attempts": 2,
      "window": "24h",
      "action": {
        "type": "reminder",
        "template": "nudge_not_read",
        "text": "👋 Видимо вы заняты. Это сообщение будет ждать вашего ответа"
      }
    },
    {
      "trigger": "message_read_no_reply", 
      "delay": "2m",
      "max_attempts": 2,
      "window": "20m",
      "action": {
        "type": "follow_up",
        "template": "nudge_no_reply",
        "text": "Вы здесь? Нужна помощь с ответом?"
      }
    },
    {
      "trigger": "qualification_abandoned",
      "delay": "1h",
      "max_attempts": 1,
      "window": "24h",
      "action": {
        "type": "re_engage",
        "template": "come_back_template",
        "switch_to_template": true
      }
    }
  ]
}
```

### Различие: Репромпты vs Нуджи

**Репромпты** (в рамках шага):
- Счетчик в Flowise Variables
- До 3 попыток сразу в диалоге
- При неправильном ответе на конкретный вопрос

**Нуджи** (между сообщениями):
- Счетчик в бэкенд nudge-service
- Отложенные напоминания по таймингу
- При отсутствии реакции на сообщение

---

## 🙋 ВОПРОСЫ И ЭСКАЛАЦИЯ

### Вопросы (в любой точке)

- `to_funnel & будет` → назвать шаг, вернуться к `askedId`
- `to_funnel & не будет` → `importance`: low→parking; high→KB (≥0.7) или оператор
- `off_funnel` — аналогично важности; после — «Возвращаемся к {askedId}»

### Мульти-интент

Если в одном сообщении ответ+вопрос: 
1. сохранить/эмитить valid
2. обработать вопрос (§10)
3. вернуться к текущему шагу

### Паркинг/доработка после квалификации

В `CONTENT/END`: сначала high (KB≥0.7/оператор), затем low батч-ответом

### Эскалация (HITL)

**Триггеры:** `retries≥3`, `KB.conf<0.7` при high, токсичность/жалоба/риски, явный «оператор»

**Пакет оператору:** `stage/askedId`, последние N сообщений, `importance/relevance/confidence`, извлечённые поля, обещания

**После:** «Продолжаем с qN»

---

## 💾 ПАМЯТЬ: КАК И ЧТО ИСПОЛЬЗУЕМ

### Слой памяти

**Короткая (runtime, 24–72ч) → Redis**
- **Что кладём:** история для LLM, последние N сообщений, быстрые флаги (`retries/hold/optOut`), контекст шага
- **Зачем:** дёшево/быстро, низкая латентность, TTL, ключи по `sessionId`
- **Где в Flowise:**
  - **Redis Cache** — кэш ответов LLM
  - **Redis-Backed Chat Memory** — диалог для агента

**Долгая (аудит/аналитика/поиск) → Postgres (+pgvector по необходимости)**
- **Что кладём:** `messages_raw/meta/links`, `funnel_answers`, `events_outbox`, CTWA-метки, заказы/оплаты, «вечная» память фактов
- **Зачем:** надёжность, SQL, отчёты, восстановление, RAG (через `pgvector`)

**(Опц.) Векторный слой → pgvector** — семантический поиск фактов/предпочтений

### Конкретные узлы/параметры в Chatflow

**Redis Cache**
- Credential: ваш Redis/Valkey
- `TTL: 600–3600s`
- `Key prefix: ws:{{$vars.workspaceId}}:cache:`
- Подключать к ChatOpenAI/модели как Cache

**Redis-Backed Chat Memory**
- Credential: тот же Redis
- `Session Id: {{$vars.sessionId}}`
- `Prefix: ws:{{$vars.workspaceId}}:chat:`
- **Window: 8–20 сообщений** *(это **пример**, не согласованная константа)*
- Подключать в Tool/Conversational Agent → Memory

**Переменные (Set/Get Variable)**
- Инициализируем: `workspaceId, tenantId, sessionId, stepKey, retries`

**Долгая память** — делает **бэкенд** по вебхукам (Gupshup/Stripe/CRM) в Postgres (+ индекс в pgvector). В Chatflow отдельный узел не нужен

### Политики и изоляция

**Ключи:**
- `ws:{workspaceId}:chat:{sessionId}` — история
- `ws:{workspaceId}:cache:{hash(prompt)}` — кэш LLM
- **Redis key prefix:** `org:{tenantId}:prj:{projectId}:sess:{sessionId}`
- **pgvector namespace:** фильтр `WHERE tenant_id=? AND project_id=?` (+индексы)

**TTL:** чат-история 24–72ч; кэш LLM 10–60 мин (диалоги) / дольше на статике

**Очистка:** при `optOut/stop` — сразу чистим `chat:{sessionId}`

**Суммаризация (опц.):** воркер сворачивает длинные чаты в Postgres; в Redis держим `summary+tail`

---

## 📡 МАРШРУТИЗАЦИЯ И ОНБОРДИНГ

### Вход WhatsApp ↔ Gupshup ↔ Backend ↔ Agentflow ↔ Chatflow

**«Главный» флоу и старт:**
- Вход **всегда** в бэкенд (webhook Gupshup)
- Старт **всегда** в **Agentflow**

### Реестр маршрутизации (БД `routing`)

**Поля:** `tenant_id, bm_id, wa_app_id(или wa_phone), entry_type(default|ctwa|keyword), entry_value(ad_id|adset_id|campaign_id|keyword), agentflow_id, chatflow_id_default(опц.), status`

**Уникальность:** `(wa_app_id, entry_type, entry_value)` — `UNIQUE`

**Логика:** по метаданным входящего (номер/wa_app_id + CTWA-теги) бек находит строку и получает `agentflow_id` → вызывает Agentflow `/predict`

### Как бек понимает «это WhatsApp»

Из webhook: `provider="gupshup"`, `wa_app_id/phone`, тип события (`message`, `template`, `status: delivered|read|failed`). Эти поля идут в `messages_meta` и в роутинг

### Несколько WA-номеров в одном BM + онбординг

- Мастер-онбординг (в админке) и нода «Onboarding» для записи в БД
- Онбординг делает: список WA-аккаунтов/номеров; выбор «привязать существующий»/«завести новый»; выбор главного `Agentflow` (и опц. дефолтного `Chatflow`); запись в `routing`
- Ограничение: `wa_app_id` **UNIQUE** в активных связках (повторная привязка запрещена)

### Кто «кому звонит»

- Бек **никому внутри канвы не звонит** — только `/predict` у **Agentflow**
- Очерёдность/step/пороги — **в Chatflow** (Variables + Memory)
- Бек передаёт контекст: `tenantId, orgId, workspaceId, sessionId, wa_app_id/phone, ctwa_*`
- Дальше Agentflow → Execute Flow(Chatflow); внутри Chatflow тулами выполняются WA-отправки/эмиссии

### «Страница Sequences» и «нода WhatsApp»

- Делаем внешнюю страницу **Sequences** (конструктор сообщений/кнопок/листов с версионированием)
- В Flowise — **одна** нода **WA Send (Gupshup)** (Custom Tool), на вход подаётся `sequence_id`/`message_id`
- Нода сама тянет из БД и отправляет пакет
- Порядок сообщений в пакете решает бек (по `sequence_id`), на канве порядок управляет поток

### Где живёт «нода WhatsApp»

- Отправка (text/template/buttons/list/…) — **в Chatflow** (через Tool Agent → WA Send)
- Короткие ack на код-слова до входа в сценарий — можно и из **Agentflow** той же нодой
- «Слушатель» входящих — **только бек** (webhook)

### «Нода постоянного прослушивания» и классификация

- Слушатель — бек
- Классификация — внутри **Chatflow** (Tool Agent + промпт/правила) или отдельная Intent/Moderation-нода
- Статусы доставки (delivered/read/failed) ловит бек и триггерит нуджи (через свой планировщик)
- В Flowise — только команда `/nudge/schedule`

### Идеология

- Start — только в **Agentflow** (код-слова/глобальные условия) → Execute Flow(Chatflow)
- В **Chatflow** — бизнес-логика шагов, валидатор, инструменты (WA Send, CAPI, Audiences, CRM, Stripe, Nudge, Operator)
- Память и шаги — в Chatflow (Redis-Backed Chat Memory + Variables)
- Мультиплатформа (WA/TG/IG/FBM): одинаковая схема, меняется только транспортный тул

### Множественные сценарии/каналы

- Можно держать несколько Chatflow
- В Agentflow через Condition/ветвления выбирать, какой Chatflow дернуть (по номеру/CTWA/keyword)
- Оператор (HITL) — в Agentflow (ветка operator) + нода «Создать тикет» + Human Input

### Что хранить и где (минимум)

**БД:** `routing`, `messages_raw/meta`, `funnel_answers`, `events_outbox`, `sequences`, `wa_accounts`, `projects`

**Redis:** `org:{orgId}:ws:{wsId}:session:{sessionId}` — история/состояние

**Variables (Workspace):** `qual_manifest`, `policy` (пороги/ретраи/нуджи), `ids` (пиксель/аудитории/CRM-маппинг), `texts_common`

---

## 📲 НОДА «WA SEND (GUPSHUP)» — «НАЧИНКА»

### Внутренности ноды (6 групп полей)

**1. Адресация**
- `to` (E.164, +7999… или `contact_id` из CRM)
- `wa_app_id` (подключённый WA-аккаунт Gupshup)
- `project_id / tenant_id / session_id` (маршрутизация/логи)
- `idempotency_key` *(по умолчанию: `sessionId-msg-<hash(message_id+vars)>`)*

**2. Что отправить (контент)**
- `mode: message | sequence`
- `message_id | sequence_id` (ссылки на библиотеку сообщений)
- `vars {}` (подстановки, напр. `{name:"Иван", budget:"5k"}`)

**(Опц.) Inline для разовых случаев:**
- `type:` `text | template | interactive_buttons | interactive_list | media_image | media_video | media_audio | document | location | contact`
- `payload` по типу:
  - `text: { text }`
  - `interactive_buttons: { text, buttons:[{id,title}] }`
  - `interactive_list: { header?, body, sections:[{title, rows:[{id,title,description?}]}] }`
  - `media_*: { url | file_id, caption? }`
  - `template: { template_name, language, components:[…], variables:{…} }`
  - `location: { lat, lng, name?, address? }`
  - `contact: { vcard | fields… }`

**3. Режим доставки (24 часа)**
- `delivery_mode:` `auto | session_only | template_only`
- `fallback_template?` (имя шаблона за пределами 24ч)
- `typing_simulation? on|off, delay_ms?`

**4. Политики и валидация**
- `validate: on|off` (лимиты WA: длины/кнопки/типы)
- `block_if_optout: on|off`
- `respect_quiet_hours: on|off, window:{start,end,tz}`
- `rate_limit_tag?` (троттлинг/BSP лимиты)

**5. Логирование и дебаг**
- `trace_id` *(авто: `sessionId:stepKey:ts`)*
- `dry_run: on|off` — не шлёт, валидирует и отдаёт сгенерированный payload
- `return_raw: on|off` — вернуть сырой ответ Gupshup

**6. Выходы ноды (outputs)**
- `status: queued | sent | failed`
- `outbound_id` (внутренний id)
- `provider_message_id` (id у Gupshup/WA)
- `idempotency_key` (фактически применённый)
- `error: code, message, retry_after?`
- `meta: delivery_mode_used, template_used?, media_info?`

**Мини-пример вызова (ссылкой на библиотеку):**
```
to=+79991234567
wa_app_id=gs_app_01
mode=message
message_id=msg_q2@v7
vars={ "name":"Иван", "budget":"5k" }
delivery_mode=auto
idempotency_key=session123-msg-7b1c…
```

---

## 🏗️ ПРОЕКТЫ И РОУТИНГ

### Минимальная схема + A/B (позже)

**Модель данных (минимум):**
- `projects(id, tenant_id, name, slug, status, default_agentflow_id, default_chatflow_id, qual_manifest_id, policy_id, created_at)`
- `wa_accounts(id, bm_id, wa_app_id, phone_e164, status, created_at)`
- `routing(id, tenant_id, project_id, wa_app_id, entry_type(default|ctwa|keyword), entry_value, agentflow_id, chatflow_id, status, created_at)`
  — `UNIQUE(wa_app_id, entry_type, entry_value)` (исключает двойную привязку)

### Роутинг входящих

1. Gupshup webhook → извлечь `wa_app_id/phone, ctwa_*, message/text`
2. Найти в `routing` по `(wa_app_id, entry_type, entry_value)` (приоритет: `ctwa:ad|adset|campaign` → `keyword` → `default`)
3. Получить `project_id, agentflow_id, chatflow_id`; вызвать Agentflow `/predict` (`question`, Vars: `projectId, tenantId, wa_app_id, sessionId, ctwa_*`)
4. Agentflow → Execute Flow(Chatflow)

### Онбординг номеров

UI: выбрать Проект → WA номер (`wa_accounts`) → создать `routing(default, '*')`. Если `wa_app_id` уже привязан как `default` в активном проекте — запретить

### События/эмиссии: указывать `project_id`

CAPI/custom_data, Audiences payload, CRM note/fields, DB (`funnel_answers`, `events_outbox`) — всегда добавлять `project_id`

### A/B (позже, без ломки)

- `routing_variants{project_id, variant_code, weight, agentflow_id, chatflow_id}`
- Sticky-бакет: хеш телефона → `variant_code`
- Вызов разных Chatflow **или** одного Chatflow с разными `qual_manifest_id/policy_id`

### Flowise: папки/теги/переменные (нейминг) + изоляция

**Теги/нейминг:**
- **Agentflow:** `[PRJ:slug] Agent START`
- **Chatflow:** `[PRJ:slug] Chat QUALIFIER`

**Workspace/Prediction Variables:** `projectId, tenantId, wa_app_id, sessionId, ctwa, qual_manifest_id, policy_id`

---

## 🔗 ИНТЕГРАЦИИ (продолжение)

### CAPI - Facebook Conversions API

**События квалификации:**
```json
{
  "data": [
    {
      "event_name": "QUALIFY_Q2",
      "event_time": 1736123456,
      "event_id": "sess-123-q2-9a1c...",
      "event_source_url": "https://wa.me/79991234567",
      "action_source": "chat",
      "user_data": {
        "external_id": "<sha256(tenantId:subscriberId)>",
        "ph": "<sha256(msisdn)>"
      },
      "custom_data": {
        "step": "q2",
        "answer_raw": "около 2500$",
        "answer_parsed": {"budget_month":2500,"bucket":"1–5k"},
        "quality_score": 0.86,
        "validity": "valid",
        "session_id": "sess-123",
        "tenant_id": "tenant-1",
        "project_id": "project-1",
        "mapping_version": "1.0.0"
      }
    }
  ]
}
```

### Custom Audiences - Рекламные аудитории

**Базовые аудитории:**
- `A1_STARTED_NOT_FINISHED` - Начали но не завершили воронку
- `A2_QUALIFIED` - Прошли квалификацию  
- `A3_REJECTED` - Отказники и шум

**Сегментные аудитории:**
- `CA_QUAL_Q1`, `CA_QUAL_Q2` - По шагам квалификации
- `CA_BUDGET_10K_50K`, `CA_BUDGET_50K_PLUS` - По бюджету
- `CA_CHANNEL_INSTAGRAM` - По каналу
- `CA_PURCHASED` - Совершили покупку
- `CA_EXCLUDE_NOISE` - Исключения шума

### CRM - Управление клиентами

**Маппинг полей (Kommo):**
```json
{
  "contact": {
    "name": "{{profileName || waPhone}}",
    "phone": "{{waPhone}}",
    "custom_fields": {
      "641289": "{{ctwa.campaign_id}}",
      "641290": "{{ctwa.adset_id}}",
      "641291": "{{ctwa.ad_id}}",
      "641292": "{{answers.Q1.niche}}",
      "641293": "{{answers.Q2.budget}}",
      "641294": "{{answers.Q3.channel}}"
    }
  }
}
```

**Триггеры синхронизации:**
| Событие | Действие в CRM |
|---------|---------------|
| CTWA старт | Создать контакт и сделку |
| Валидный Qn | Обновить поля + коммент |
| Квалификация пройдена | Сменить стадию |
| Оплата получена | Сменить на "Оплачено" |

### Stripe - Платежная система

**Создание платежной ссылки:**
```javascript
const paymentLink = await stripe.paymentLinks.create({
  line_items: [{
    price: 'price_ABC123',
    quantity: 1
  }],
  metadata: {
    session_id: 'session_123',
    subscriber_id: 'sub_789',
    product_id: 'SKU_001',
    source: 'whatsapp_funnel'
  }
})
```

**Обработка webhook событий:**
- `checkout.session.completed` → Purchase событие в CAPI
- Обновление аудиторий → перенос в CA_PURCHASED
- Обновление CRM → смена стадии на "Оплачено"
- WhatsApp подтверждение → template с чеком

---

## 📐 СТРУКТУРА БАЗЫ ДАННЫХ

### Основные таблицы

**`subscribers`**
```sql
CREATE TABLE subscribers (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR NOT NULL,
  wa_phone VARCHAR UNIQUE NOT NULL,
  phone_hash VARCHAR NOT NULL,
  profile_name VARCHAR,
  first_seen_at TIMESTAMP,
  last_inbound_at TIMESTAMP,
  status VARCHAR DEFAULT 'active',
  optout BOOLEAN DEFAULT false,
  snoozed_until TIMESTAMP,
  conversation_id VARCHAR,
  conversation_expires_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**`funnel_answers`**
```sql
CREATE TABLE funnel_answers (
  id UUID PRIMARY KEY,
  tenant_id BIGINT NOT NULL,
  subscriber_id BIGINT NOT NULL,
  session_id TEXT NOT NULL,
  step_key TEXT NOT NULL,
  validity TEXT NOT NULL CHECK (validity IN ('valid','noise','ambiguous')),
  quality_score NUMERIC(3,2) NOT NULL,
  answer_raw TEXT NOT NULL,
  answer_parsed JSONB NOT NULL DEFAULT '{}'::jsonb,
  version INT NOT NULL DEFAULT 1,
  event_id TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ,
  PRIMARY KEY (tenant_id, subscriber_id, session_id, step_key)
);
```

**`events_outbox`**
```sql
CREATE TABLE events_outbox (
  id BIGSERIAL PRIMARY KEY,
  event_id TEXT NOT NULL UNIQUE,
  channel TEXT NOT NULL CHECK (channel IN ('CAPI','AUDIENCE','CRM')),
  payload JSONB NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending','sent','failed','on_hold')),
  retries INT NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  last_error TEXT
);
```

**`messages_raw`**
```sql
CREATE TABLE messages_raw (
  id UUID PRIMARY KEY,
  message_id VARCHAR UNIQUE NOT NULL,
  session_id VARCHAR,
  direction VARCHAR NOT NULL, -- inbound/outbound
  type VARCHAR NOT NULL, -- text/image/button/list/template
  payload JSONB NOT NULL,
  status VARCHAR, -- sent/delivered/read/failed
  parent_id UUID, -- ссылка на предыдущее
  parent_type VARCHAR, -- step/message/none
  created_at TIMESTAMP DEFAULT NOW(),
  delivered_at TIMESTAMP,
  read_at TIMESTAMP,
  replied_at TIMESTAMP
);
```

**Транзакция:** `funnel_answers` + пачка `events_outbox` → коммит. Worker: идемпотентные отправки, backoff `1→2→5→15` мин

---

## 📊 МЕТРИКИ И АЛЕРТЫ

По шагам: доля `valid/noise/ambiguous`, ср. `quality_score`, % обновлений

Коннекторы: latency/ошибки/повторы

Качество трафика: доля `CA_EXCLUDE_NOISE`

HITL: SLA, время возврата

Нуджи: CTR/ответы/эскалации

Реабилитация: % `NOISE→GOOD`

---

## 🔢 ПОРОГОВЫЕ ЗНАЧЕНИЯ

- Free-text ≥0.7
- Choice ≥0.8
- KB-ответ ≥0.7
- Анти-чушь ретраи `1→2→3(оператор)`
- Нуджи `read:2→20м(2)`, `delivered:15м→24ч(2)`

---

## 🔐 НАДЁЖНОСТЬ И ПРИВАТНОСТЬ

- Идемпотентность (`event_id` + уникальный индекс в outbox)
- Backoff-ретраи
- Hash-PII (sha256) для `external_id/ph`
- Аудит решений (valid/noise/ambiguous, нуджи, код-слова, эскалации)

---

## 🔄 МИНИ-FSM (упрощённо)

```
Qn.wait → (код-слово) → hold/optOut/snooze
Qn.wait → (valid) → emit(Qn) → Q{n+1}.wait|CONTENT
Qn.wait → (ambiguous/noise) → retry/assist → Qn.wait|operator
any → (question) → route(§10) → return(Qn.wait)
CONTENT → END
```

---

## 🎮 AGENTFLOW КОНФИГУРАЦИЯ

### WhatsApp Send Node - Универсальная нода

**Конфигурация ноды:**
```javascript
{
  name: "WhatsAppSend_Agentflow",
  type: "WhatsAppSend",
  category: "Agent Flows",
  version: "1.0.0",
  description: "Универсальная отправка в WhatsApp",
  inputs: [
    {
      label: "Message Type",
      name: "messageType",
      type: "options",
      options: [
        { value: "text", label: "Text" },
        { value: "image", label: "Image" },
        { value: "button", label: "Interactive Buttons" },
        { value: "list", label: "Interactive List" },
        { value: "template", label: "Template" }
      ],
      default: "text"
    },
    {
      label: "To",
      name: "to",
      type: "string",
      placeholder: "+79991234567",
      acceptVariable: true
    },
    {
      label: "Session ID", 
      name: "sessionId",
      type: "string",
      acceptVariable: true
    },
    {
      label: "Idempotency Key",
      name: "idempotencyKey",
      type: "string",
      acceptVariable: true,
      optional: true
    }
  ]
}
```

**Автоматическое переключение session/template:**
- В пределах 24h окна → session message
- После истечения 24h → template message
- При failed доставке session → retry как template

---

## 🏁 КРАЕВЫЕ СЛУЧАИ

- Длинные полотна → суммаризация/допрос
- Голос/фото → ASR/OCR→те же правила
- Дубликаты → идемпотентность `lastOutboundId/event_id`
- `hold=true` — валидные ответы **не теряем** и **эмитим**
- `optOut` — входящие **только БД**, наружу **не** шлём

---

## ⚙️ ГОТОВНОСТЬ К ВНЕДРЕНИЮ

- Карта `willCoverLater`
- Словари/негации
- Пороги/окна/лимиты
- БД/воркеры/outbox
- Аудитории `CA_QUAL_*` / `CA_EXCLUDE_NOISE`
- Мониторинг/алерты
- Тесты: идемпотентность/ретраи/noise/обновления/код-слова/нуджи

---

## 🚫 ТИПОВЫЕ ПРОБЛЕМЫ И РЕШЕНИЯ

### Потеря контекста при перезапуске Flowise
**Симптом:** Variables (stepKey, answers, retries) сбрасываются в null/undefined
**Причина:** Flowise не персистит Variables между рестартами
**Решение:**
- Сохранение критичных переменных в БД через Custom Function после каждого шага
- Восстановление состояния из БД при инициализации сессии
- Fallback: начало с Q1 при потере состояния

### Дублирование событий CAPI
**Симптом:** Одно событие QUALIFY_Q{n} отправляется несколько раз
**Причина:** Retry логика без идемпотентности  
**Решение:**
- Уникальный event_id = `{sessionId}-q{n}-{hash(answer_raw)}`
- Проверка в outbox перед эмиссией: `SELECT 1 FROM events_outbox WHERE event_id = ?`
- Constraint UNIQUE(event_id) в БД

### Истечение 24h окна WhatsApp
**Симптом:** Ошибка 403/400 при отправке session message через 24 часа
**Причина:** WhatsApp Policy - session messages только в течение 24h после последнего входящего
**Решение:**
- Автопереключение на template message в WA Send node
- Проверка `conversation_expires_at` перед отправкой
- Fallback templates для каждого типа сообщения

### Низкий confidence score в LLM
**Симптом:** Много ответов попадает в категорию ambiguous (0.3 ≤ confidence < 0.65)
**Причина:** Неоднозначные вопросы, короткие ответы, отсутствие контекста
**Решение:**
- Улучшение промптов с примерами валидных ответов
- Использование контекста предыдущих ответов для классификации
- Clarification prompts: "Уточните, пожалуйста..."
- Снижение порога confidence для определенных шагов

### Рассинхронизация CRM
**Симптом:** Поля в CRM не обновляются или обновляются с задержкой
**Причина:** Webhook пропущен, timeout, неправильный mapping
**Решение:**
- Retry механизм для CRM webhooks с exponential backoff
- Периодическая синхронизация (каждые 15 минут)
- Логирование всех CRM операций для отладки
- Fallback на прямые API вызовы при неудаче webhook

### Перегрузка при высоком трафике
**Симптом:** Медленная обработка сообщений, таймауты LLM
**Причина:** Недостаточно ресурсов, блокирующие операции
**Решение:**
- Горизонтальное масштабирование Flowise через Docker replicas
- Асинхронная обработка через Queue (Bull/Agenda)
- Кэширование LLM ответов для повторяющихся паттернов
- Circuit breaker для внешних API (OpenAI, Gupshup)

### Некорректное извлечение CTWA данных
**Симптом:** campaign_id/ad_id приходят пустыми или с мусорными значениями
**Причина:** Различия в форматах webhook от разных провайдеров
**Решение:**
- Нормализация в parser слое: строгая валидация по регексам
- Fallback на utm_* из referral.source_url при отсутствии прямых полей
- Логирование raw webhook для анализа новых форматов
- Мониторинг процента успешного извлечения CTWA

---

## 🚦 БИЗНЕС-ПРАВИЛА И ИНВАРИАНТЫ

### Жесткие инварианты

**Правило 1: Немедленная эмиссия**
```
ЕСЛИ ответ валидный (confidence ≥ 0.65)
ТО СРАЗУ отправить в CAPI + Audiences + CRM + БД
БЕЗ ОЖИДАНИЯ завершения воронки
```

**Правило 2: Разделение потоков**
```
GOOD поток → основной pixel/dataset
NOISE поток → отдельный pixel/dataset или негативный пиксель/датасет NEG
НИКОГДА не смешивать потоки
```

**Правило 3: Приоритет код-слов**
```
Код-слова обрабатываются ДО любой другой логики
stop/pause/operator (синонимы) имеют абсолютный приоритет
```

**Правило 4: Идемпотентность**
```
Каждое событие имеет уникальный event_id
Повторная обработка возвращает тот же результат
```

**Правило 5: CRM стадии**
```
НЕ двигаем стадии автоматически
КРОМЕ явных правил (квалификация пройдена)
```

### Ключевые принципы

1. **Порог уверенности**: 0.65 (единый для валидации и ответов агента)
2. **Репромпты**: до 3 попыток сразу в диалоге (счетчик per step)
3. **Нуджи**: до 2 напоминаний с отложенным таймингом (счетчик per case)
4. **Стадии CRM**: движение только по явным правилам (не автоматически)
5. **Эмиссии**: всегда через бэкенд с outbox-паттерном
6. **Потоки данных**: GOOD и NOISE строго разделены (разные pixels)
7. **24h окно**: автопереключение на template messages при истечении
8. **Идемпотентность**: уникальный event_id для каждого действия
9. **Persist-all**: сохраняем **все** ответы/шумы в БД для анализа

---

## 🔄 ПРОЦЕССЫ И АВТОМАТИЗАЦИЯ

### Qualification Pipeline

```mermaid
graph TD
    A[CTWA Click] --> B[Create Session]
    B --> C[Send Q1]
    C --> D{Validate Answer}
    D -->|Valid ≥0.65| E[Emit CAPI/CRM/CA]
    D -->|Noise <0.3| F[Retry/Escalate]
    D -->|Ambiguous 0.3-0.65| G[Clarify]
    E --> H[Send Q2]
    F --> I{Retry < 3?}
    I -->|Yes| C
    I -->|No| J[Escalate to Operator]
    G --> K[Wait for Clarification]
    K --> D
    H --> L[Continue Steps...]


    Детализация валидации по шагам (из v2):
  const STEP_VALIDATION_RULES = {
  Q1: {
    type: 'free_text',
    fields: ['niche', 'pain'],
    rules: {
      pain: 'минимум 4 значимых слова',
      niche: 'должна быть в каталоге',
      similarity_threshold: 0.7
    }
  },
  Q2: {
    type: 'free_text', 
    fields: ['budget'],
    parser: 'extractBudgetBucket',
    buckets: ['<1k', '1-5k', '5-20k', '20k+'],
    similarity_threshold: 0.7
  },
  Q3: {
    type: 'choice',
    options: ['instagram', 'facebook', 'google', 'tiktok'],
    allow_synonyms: true,
    similarity_threshold: 0.8
  },
  Q4: {
    type: 'choice',
    options: ['yes', 'no'],
    similarity_threshold: 0.8
  }
}


Nudge State Machine (расширено из v2)
stateDiagram-v2
    [*] --> Sent
    Sent --> Delivered: WhatsApp delivered
    Sent --> Failed: Delivery failed
    Delivered --> Read: WhatsApp read
    Delivered --> NotRead: 15min timeout
    Read --> Replied: User replied
    Read --> NoReply: 2min timeout
    NotRead --> Nudge1: Send reminder (limit 2)
    NoReply --> Nudge2: Send follow-up (limit 2)
    Nudge1 --> Delivered
    Nudge2 --> Read
    Failed --> Retry: Immediate
    Failed --> Snooze24h: After retry
    Retry --> Sent: Retry attempt
    Retry --> Escalate: Max retries
    Snooze24h --> AlertOperator


    Конфигурация нуджей с лимитами (из v2):

delivered (не прочитано): T+15м → до 24ч, лимит 2
read (прочитано, нет ответа): T+2м → до 20м, лимит 2
failed: сразу retry; затем snooze 24ч + алерт оператору
Минимальный интервал между нуджами: ≥2 мин
При hold/optOut/snooze — нуджи OFF
📊 ПАМЯТЬ И ХРАНИЛИЩА (из v2 #input3)
Архитектура памяти
Короткая память (Redis) - runtime, 24-72ч:
const REDIS_MEMORY_CONFIG = {
  chat_history: {
    key_pattern: 'ws:{workspaceId}:chat:{sessionId}',
    ttl: 86400 * 3, // 72 часа
    window_size: 20, // последние 20 сообщений
    content: ['messages', 'context', 'flags']
  },
  llm_cache: {
    key_pattern: 'ws:{workspaceId}:cache:{hash(prompt)}',
    ttl: 3600, // 1 час для диалогов
    static_ttl: 86400 // 24 часа для статики
  },
  session_state: {
    key_pattern: 'org:{tenantId}:prj:{projectId}:sess:{sessionId}',
    ttl: 86400,
    content: ['retries', 'hold', 'optOut', 'stepContext']
  }
}


Долгая память (PostgreSQL + pgvector):
-- Расширенная схема из v2

-- Память фактов с векторным поиском
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE subscriber_facts (
  id UUID PRIMARY KEY,
  tenant_id BIGINT NOT NULL,
  subscriber_id BIGINT NOT NULL,
  fact_type VARCHAR NOT NULL, -- preference/behavior/purchase
  fact_text TEXT NOT NULL,
  fact_embedding vector(1536), -- для семантического поиска
  confidence DECIMAL(3,2),
  extracted_at TIMESTAMP,
  session_id VARCHAR,
  created_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_vector_search ON subscriber_facts 
    USING ivfflat (fact_embedding vector_cosine_ops)
    WITH (lists = 100)
);


Конфигурация Flowise Memory Nodes
Redis-Backed Chat Memory:
{
  credential: 'Redis/Valkey Connection',
  sessionId: '{{$vars.sessionId}}',
  prefix: 'ws:{{$vars.workspaceId}}:chat:',
  windowSize: 20, // НЕ согласованная константа, пример
  ttl: 259200 // 72 часа
}


Variables для инициализации:
const WORKSPACE_VARIABLES = {
  workspaceId: context.workspace.id,
  tenantId: context.tenant.id,
  projectId: context.project.id,
  sessionId: generateSessionId(),
  stepKey: 'Q1',
  retries: {},
  qual_manifest_id: context.manifest.id,
  policy_id: context.policy.id
}


🚦 МАРШРУТИЗАЦИЯ И МНОЖЕСТВЕННЫЕ НОМЕРА (из v2 #input4)
Архитектура маршрутизации

  agentflow_id VARCHAR NOT NULL,
  chatflow_id VARCHAR,
  status VARCHAR DEFAULT 'active',
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(wa_app_id, entry_type, entry_value)
);

CREATE TABLE wa_accounts (
  id BIGSERIAL PRIMARY KEY,
  bm_id VARCHAR NOT NULL,
  wa_app_id VARCHAR NOT NULL UNIQUE,
  phone_e164 VARCHAR NOT NULL,
  status VARCHAR DEFAULT 'active',
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE projects (
  id BIGSERIAL PRIMARY KEY,
  tenant_id BIGINT NOT NULL,
  name VARCHAR NOT NULL,
  slug VARCHAR NOT NULL,
  status VARCHAR DEFAULT 'active',
  default_agentflow_id VARCHAR,
  default_chatflow_id VARCHAR,
  qual_manifest_id VARCHAR,
  policy_id VARCHAR,
  created_at TIMESTAMP DEFAULT NOW()
);


Pipeline входящих сообщений
async function routeIncomingMessage(webhook: GupshupWebhook) {
  // 1. Извлечение данных
  const waAppId = webhook.app.id
  const ctwaParams = extractCTWA(webhook.referral)
  
  // 2. Поиск маршрута (приоритет)
  const route = await findRoute(waAppId, ctwaParams) || 
                 await findRoute(waAppId, 'keyword', webhook.text) ||
                 await findRoute(waAppId, 'default', '*')
  
  if (!route) {
    throw new Error(`No route found for ${waAppId}`)
  }
  
  // 3. Создание/восстановление сессии
  const sessionId = generateSessionId({
    source: ctwaParams ? 'ctwa' : 'direct',
    details: ctwaParams?.ad_id || '',
    subscriberId: webhook.sender,
    timestamp: Date.now()
  })
  
  // 4. Вызов Agentflow (всегда стартовая точка)
  const response = await callAgentflow({
    agentflowId: route.agentflow_id,
    sessionId,
    message: webhook.text,
    variables: {
      projectId: route.project_id,
      tenantId: route.tenant_id,
      waAppId,
      ctwa: ctwaParams,
      qual_manifest_id: route.qual_manifest_id,
      policy_id: route.policy_id
    }
  })
  
  return response
}


📤 WHATSAPP SEND NODE - ПОЛНАЯ СПЕЦИФИКАЦИЯ (из v2 #input5)
Внутренности ноды WA Send (Gupshup)
class WhatsAppSendTool extends Tool {
  constructor() {
    super()
    this.name = 'WA Send (Gupshup)'
    this.description = 'Универсальная отправка WhatsApp сообщений'
    this.inputs = {
      // 1. Адресация
      to: { type: 'string', required: true }, // E.164 или contact_id
      wa_app_id: { type: 'string', required: true },
      project_id: { type: 'string' },
      tenant_id: { type: 'string' },
      session_id: { type: 'string' },
      idempotency_key: { type: 'string' }, // auto: sessionId-msg-<hash>
      
      // 2. Контент
      mode: { type: 'enum', values: ['message', 'sequence'] },
      message_id: { type: 'string' },
      sequence_id: { type: 'string' },
      vars: { type: 'object' }, // подстановки {name: "Иван"}
      
      // Inline payload (опционально)
      type: { 
        type: 'enum', 
        values: ['text', 'template', 'interactive_buttons', 
                 'interactive_list', 'media_image', 'media_video', 
                 'media_audio', 'document', 'location', 'contact']
      },
      payload: { type: 'object' },
      
      // 3. Режим доставки
      delivery_mode: { 
        type: 'enum', 
        values: ['auto', 'session_only', 'template_only'],
        default: 'auto'
      },
      fallback_template: { type: 'string' },
      typing_simulation: { type: 'boolean', default: false },
      delay_ms: { type: 'number' },
      
      // 4. Политики
      validate: { type: 'boolean', default: true },
      block_if_optout: { type: 'boolean', default: true },
      respect_quiet_hours: { type: 'boolean', default: true },
      quiet_window: { type: 'object' }, // {start, end, tz}
      rate_limit_tag: { type: 'string' },
      
      // 5. Debug
      trace_id: { type: 'string' }, // auto: sessionId:stepKey:ts
      dry_run: { type: 'boolean', default: false },
      return_raw: { type: 'boolean', default: false }
    }
  }
  
  async execute(inputs) {
    // Генерация idempotency_key если не задан
    if (!inputs.idempotency_key) {
      inputs.idempotency_key = generateIdempotencyKey(
        inputs.session_id,
        inputs.message_id || inputs.sequence_id,
        inputs.vars
      )
    }
    
    // Проверка optout статуса
    if (inputs.block_if_optout) {
      const subscriber = await getSubscriber(inputs.to)
      if (subscriber.optout) {
        return {
          status: 'blocked',
          reason: 'subscriber_opted_out'
        }
      }
    }
    
    // Автоматическое переключение session/template
    if (inputs.delivery_mode === 'auto') {
      const canUseSession = await checkSessionWindow(inputs.to)
      inputs.delivery_mode = canUseSession ? 'session' : 'template'
      
      if (!canUseSession && inputs.fallback_template) {
        inputs.type = 'template'
        inputs.payload = { template_name: inputs.fallback_template }
      }
    }
    
    // Отправка через Gupshup API
    const result = await gupshupClient.send(inputs)
    
    // Сохранение в outbox для гарантированной доставки
    await saveToOutbox({
      event_id: inputs.idempotency_key,
      channel: 'WHATSAPP',
      payload: inputs,
      status: result.success ? 'sent' : 'failed',
      provider_message_id: result.messageId
    })
    
    return {
      status: result.success ? 'sent' : 'failed',
      outbound_id: result.outboundId,
      provider_message_id: result.messageId,
      idempotency_key: inputs.idempotency_key,
      error: result.error,
      meta: {
        delivery_mode_used: inputs.delivery_mode,
        template_used: inputs.payload?.template_name
      }
    }
  }
}


Примеры вызова ноды
// Отправка по ссылке на библиотеку
{
  to: '+79991234567',
  wa_app_id: 'gs_app_01',
  mode: 'message',
  message_id: 'msg_q2@v7',
  vars: { name: 'Иван', budget: '5k' },
  delivery_mode: 'auto',
  session_id: 'session123'
}

// Inline отправка с кнопками
{
  to: '+79991234567',
  wa_app_id: 'gs_app_01',
  type: 'interactive_buttons',
  payload: {
    text: 'Выберите основной канал продаж:',
    buttons: [
      { id: 'instagram', title: 'Instagram' },
      { id: 'facebook', title: 'Facebook' },
      { id: 'google', title: 'Google Ads' }
    ]
  },
  delivery_mode: 'auto'
}
````

---

## 🎯 A/B ТЕСТИРОВАНИЕ И ВАРИАНТЫ

### Подготовка для A/B без ломки текущей логики

````sql
CREATE TABLE routing_variants (
  id BIGSERIAL PRIMARY KEY,
  project_id BIGINT NOT NULL,
  variant_code VARCHAR NOT NULL,
  weight DECIMAL(3,2) NOT NULL, -- вес варианта 0.00-1.00
  agentflow_id VARCHAR NOT NULL,
  chatflow_id VARCHAR,
  qual_manifest_id VARCHAR,
  policy_id VARCHAR,
  status VARCHAR DEFAULT 'active',
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(project_id, variant_code)
);

-- Sticky assignment для консистентности
CREATE TABLE subscriber_variants (
  subscriber_id BIGINT PRIMARY KEY,
  project_id BIGINT NOT NULL,
  variant_code VARCHAR NOT NULL,
  assigned_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_project_variant (project_id, variant_code)
);
````

---

## 📈 МЕТРИКИ И МОНИТОРИНГ

### Ключевые метрики по шагам

````javascript
const QUALIFICATION_METRICS = {
  step_performance: {
    valid_rate: 'COUNT(validity="valid") / COUNT(*)',
    noise_rate: 'COUNT(validity="noise") / COUNT(*)', 
    ambiguous_rate: 'COUNT(validity="ambiguous") / COUNT(*)',
    avg_quality_score: 'AVG(quality_score) WHERE validity="valid"',
    update_rate: 'COUNT(version > 1) / COUNT(*)'
  },
  
  connectors: {
    capi_latency: 'p95(response_time) WHERE channel="CAPI"',
    capi_error_rate: 'COUNT(status="failed") / COUNT(*)',
    retry_rate: 'COUNT(retries > 0) / COUNT(*)'
  },
  
  traffic_quality: {
    noise_audience_size: 'COUNT(DISTINCT subscriber_id) IN CA_EXCLUDE_NOISE',
    noise_to_good_recovery: 'COUNT(transitioned from noise to valid)'
  },
  
  hitl_sla: {
    escalation_rate: 'COUNT(escalated) / COUNT(*)',
    resolution_time: 'AVG(resolved_at - escalated_at)',
    return_to_bot_rate: 'COUNT(returned_to_bot) / COUNT(escalated)'
  },
  
  nudge_effectiveness: {
    delivered_ctr: 'COUNT(replied after nudge) / COUNT(nudged)',
    read_response_rate: 'COUNT(replied) / COUNT(read)',
    escalation_after_nudge: 'COUNT(escalated) / COUNT(nudged)'
  }
}
````

### Алерты и пороги

````yaml
alerts:
  qualification:
    - name: low_valid_rate
      condition: valid_rate < 0.5
      severity: warning
      notification: slack
    
    - name: high_noise_rate  
      condition: noise_rate > 0.3
      severity: critical
      notification: [slack, email]
    
    - name: capi_failures
      condition: error_rate > 0.05
      window: 5m
      severity: critical
      notification: pagerduty
  
  system:
    - name: hitl_queue_overflow
      condition: unassigned_tickets > 10
      severity: warning
      
    - name: nudge_limit_exceeded
      condition: nudge_count > configured_limit
      severity: info
````





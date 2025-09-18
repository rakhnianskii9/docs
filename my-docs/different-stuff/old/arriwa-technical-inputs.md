Дополнения к /home/projects/new-flowise/docs/my-docs/about-arriwa-project/arriwa-technical-v1.md

---

\#input1

**Инварианты (зафиксировано)**

* **Порог валидности:** `0.65` (единый для валидации ответов и логики агента).
* **Хранилища:** `Redis` (оперативная память/контекст + кэш), `PostgreSQL` (источник истины), `pgvector` (семантическая память/RAG).
* **Роли Flowise:** `Agentflow` — агент/оркестратор (код-слова, старт/эскалации, вызов чат-сценария); `Chatflow` — детерминированные шаги квалификации (валидатор/парсинг/ветвления/эмиссии).

**Согласование терминов: «сессия» (sessionId)**

* **Определение:** ярлык, под которым Flowise/Arriwa хранит и читает историю *данного* разговора и состояние шага.
* **Кто задаёт:** sessionId задаём **мы**; все Memory-ноды и кастом-тулы читают/пишут по нему.
* **Где живёт:** в Arriwa — `Redis` (живая история окна/контекст), `PostgreSQL` (архив/состояние шагов), долговременная память через `pgvector`. Встроенная Buffer-память Flowise пишет в `chat_message`, но **не** считается источником истины.
* **Передача в рантайме:** Prediction API → `overrideConfig.sessionId`; в узлах/тулах доступно как `$flow.sessionId`.
* **Отличие от chatId:** chatId — внутренний ID беседы Flowise; sessionId — наш внешний/логический ID. Источник истины для склейки — `sessionId`.
* **Практика (WhatsApp):** `sessionId = <tenant>:<phone>:<flow>` — чтобы не смешивать проекты/воронки.

---

\#input2

**N-ШАГОВЫЙ КВАЛИФИКАТОР + «ЖИВОЙ» АГЕНТ + ДВА ПОТОКА (GOOD/NOISE)**

**0) Назначение и рамки**

* N шагов квалификации (не фикс.).
* На **каждом валидном ответе** — немедленные эмиссии: **CAPI / Аудитории / CRM / БД**.
* **Шум/ambiguous** — не в основные каналы; либо в отдельные **noise-каналы/аудитории**/**негативный пиксель/датасет** (для исключений и аналитики).
* Параллельно всегда активен «живой» агент: анти-чушь, ответы на вопросы (БЗ), код-слова, нуджи, HITL.
* **Persist-all:** сохраняем **все** ответы/шумы в БД (для анализа и **lookalike-исключений**).
* Идемпотентность, outbox, ретраи — **обязательно**.

**1) Состояния и флаги (минимум)**
`stage∈{Q1..Qn,CONTENT,END}`, `askedId`, `expectedType∈{free_text,choice}`, `answers{}`, `validity∈{valid,noise,ambiguous}`, `quality_score∈[0..1]`, `retries[qN]`, `hold`, `optOut`, `snoozeUntil`, доставка (`lastOutboundId,lastStatus,lastStatusAt,nudgeCount[qN]`), `parkingLot[]`.

**2) Справочники/словари**
`willCoverLater(topic)→step`, синонимы/негации код-слов, синонимы опций, каталоги `niche/budget_bucket/channel`, маркеры важности вопросов (`high/low`).

**3) Классификация входящих**

1. Код-слова (приоритет): `operator`→HITL/hold, `stop`→optOut, `pause`→snooze+hold, `start`→resume.
2. Если не код-слово → `intent∈{funnel_answer,question,smalltalk,noise}`.
3. Для `funnel_answer` → определить `validity, quality_score`.
4. Для `question` → `relevance to_funnel/off_funnel`, `willCoverLater`, `importance`.

**4) Анти-чушь и парсинг по шагам**

* **Q1 (free\_text: niche,pain):** pain ≥4 значимых слов; niche ∈ каталоге; **близость ≥0.7**.
* **Q2 (free\_text: budget):** извлечь число/диапазон → `bucket∈{<1k,1–5k,5–20k,20k+}`; **близость ≥0.7**.
* **Q3 (choice):** сопоставление с опциями/синонимами **≥0.8**.
* **Q4 (choice):** да/нет **≥0.8**.
* Ретраи анти-чуши: 1 — репромпт; 2 — подсказки/кнопки; 3 — оператор.

**5) Эмиссии на **каждом валидном** ответе**
`event_id = sessionId-{stepKey}-{sha256(normalize(answer_raw))}`; `normalize`: trim/lower/collapse spaces/remove urls/phones/emojis.

**5.1 CAPI (пример)**

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
    "mapping_version": "1.0.0"
  }
}
```

**5.2 Аудитории**

* `valid` → `CA_QUAL_{stepKey}` + ветвевые (`CA_NICHE_*`, `CA_BUDGET_*`, `CA_CHANNEL_*`, `CA_UPLIFT_{yes|no}`).
* `noise/ambiguous` → `CA_EXCLUDE_NOISE` **и/или** «теневой» пиксель/датасет **NEG** (см. политики).

**5.3 CRM**
Upsert лида; обновление полей шага; комментарий `QUALIFY_{stepKey}` (raw/parsed/qs).

**5.4 БД (нормализация + outbox)**

```sql
create table funnel_answers(
  tenant_id bigint not null,
  subscriber_id bigint not null,
  session_id text not null,
  step_key text not null,
  validity text not null check (validity in ('valid','noise','ambiguous')),
  quality_score numeric(3,2) not null,
  answer_raw text not null,
  answer_parsed jsonb not null default '{}'::jsonb,
  version int not null default 1,
  event_id text not null,
  created_at timestamptz not null default now(),
  updated_at timestamptz,
  primary key (tenant_id, subscriber_id, session_id, step_key)
);

create table events_outbox(
  id bigserial primary key,
  event_id text not null unique,
  channel text not null check (channel in ('CAPI','AUDIENCE','CRM')),
  payload jsonb not null,
  status text not null default 'pending' check (status in ('pending','sent','failed','on_hold')),
  retries int not null default 0,
  created_at timestamptz not null default now(),
  last_error text
);
```

Транзакция: `funnel_answers` + пачка `events_outbox` → коммит. Worker: идемпотентные отправки, backoff `1→2→5→15` мин.

**6) Политики для noise/ambiguous**

* **S1 (жёсткая):** БД + `CA_EXCLUDE_NOISE`, без CAPI/CRM.
* **S2 (теневая):** **NEG** CAPI в отдельный пиксель/датасет `NEG_QUALIFY_{stepKey}`; CRM — нет.
* **S3 (смешанная):** CRM-заметка (без стадии) + `CA_EXCLUDE_NOISE`; CAPI — нет.
  **Persist-all:** сохраняем **всё** (для аналитики и **lookalike-исключений**).

**7) Идемпотентность/обновления/hold**

* Первый `valid` по шагу → штатный пакет эмиссий.
* Повторный `valid` → `QUALIFY_{stepKey}_UPDATE` (или тот же `event_id`).
* `ambiguous/noise` **никогда** не апдейтит `valid`-ивент.
* `hold=true` (оператор/пауза): вопросы не шлём и нуджи off; **валидные входящие ответы сохраняем и эмитим**.
* `optOut=true`: все исходящие off; валидные входящие — **только БД** (без внешних эмиссий).

**8) Код-слова (детерминированно)**

| Триггер  | Варианты                              | Условие        | Действие                      | Ответ                  | Последствия    |
| -------- | ------------------------------------- | -------------- | ----------------------------- | ---------------------- | -------------- |
| Оператор | оператор/человек/support/agent/помощь | без «не» рядом | `hold=true`, HITL             | «Подключаю оператора…» | SLA, нуджи off |
| Стоп     | стоп/stop/unsubscribe/отписка         | однозначно     | `optOut=true`                 | «Остановлено…»         | Блок исходящих |
| Пауза    | пауза/позже/через X                   | —              | `snoozeUntil=ts`, `hold=true` | «Пауза до…»            | Ретраи off     |
| Старт    | старт/продолжим/продолжай             | из hold        | `hold=false`                  | «Продолжаем с qN»      | Таймеры on     |

**9) Нуджи (read/delivered/failed)**

* `delivered` (не прочитано): `T+15м` → до `24ч`, **лимит 2**.
* `read` (прочитано, нет ответа): `T+2м` → до `20м`, **лимит 2**.
* `failed`: сразу; затем `snooze 24ч` + алерт оператору.
  Правила: мин. интервал ≥2 мин; при `hold/optOut/snooze` — OFF; учитывать рабочие окна.

**10) Вопросы (в любой точке)**
`to_funnel & будет` → назвать шаг, вернуться к `askedId`.
`to_funnel & не будет` → `importance`: low→parking; high→KB (≥0.7) или оператор.
`off_funnel` — аналогично важности; после — «Возвращаемся к {askedId}».

**11) Мульти-интент**
Если в одном сообщении ответ+вопрос: (1) сохранить/эмитить valid; (2) обработать вопрос (§10); (3) вернуться к текущему шагу.

**12) Паркинг/доработка после квалификации**
В `CONTENT/END`: сначала high (KB≥0.7/оператор), затем low батч-ответом.

**13) Эскалация (HITL)**
Триггеры: `retries≥3`, `KB.conf<0.7` при high, токсичность/жалоба/риски, явный «оператор». Пакет оператору: `stage/askedId`, последние N сообщений, `importance/relevance/confidence`, извлечённые поля, обещания. После — «Продолжаем с qN».

**14) Метрики/алерты**
По шагам: доля `valid/noise/ambiguous`, ср. `quality_score`, % обновлений. Коннекторы: latency/ошибки/повторы. Качество трафика: доля `CA_EXCLUDE_NOISE`. HITL: SLA, время возврата. Нуджи: CTR/ответы/эскалации. Реабилитация: % `NOISE→GOOD`.

**15) Пороговые значения**
Free-text ≥0.7; Choice ≥0.8; KB-ответ ≥0.7; анти-чушь ретраи `1→2→3(оператор)`; нуджи `read:2→20м(2)`, `delivered:15м→24ч(2)`.

**16) Эмиссии по шагам — дайджест**

* **Q1:** CAPI `QUALIFY_Q1{niche,pain}`; Aud `CA_QUAL_Q1 + CA_NICHE_*`; CRM `cf_niche/cf_pain`; DB+outbox.
* **Q2:** CAPI `QUALIFY_Q2{budget_month,bucket}`; Aud `CA_QUAL_Q2 + CA_BUDGET_*`; CRM `cf_budget_*`; DB.
* **Q3:** CAPI `QUALIFY_Q3{main_traffic_channel}`; Aud `CA_QUAL_Q3 + CA_CHANNEL_*`; CRM `cf_main_channel`; DB.
* **Q4:** CAPI `QUALIFY_Q4{uplift}`; Aud `CA_QUAL_Q4 + CA_UPLIFT_{yes|no}`; CRM `cf_budget_uplift_ready`; DB.

**17) Надёжность/приватность**
Идемпотентность (`event_id` + уникальный индекс в outbox); backoff-ретраи; Hash-PII (sha256) для `external_id/ph`; аудит решений (valid/noise/ambiguous, нуджи, код-слова, эскалации).

**18) Мини-FSM (упрощённо)**
`Qn.wait → (код-слово) → hold/optOut/snooze`
`Qn.wait → (valid) → emit(Qn) → Q{n+1}.wait|CONTENT`
`Qn.wait → (ambiguous/noise) → retry/assist → Qn.wait|operator`
`any → (question) → route(§10) → return(Qn.wait)`
`CONTENT → END`.

**19) Краевые случаи**
Длинные полотна → суммаризация/допрос; голос/фото → ASR/OCR→те же правила; дубликаты → идемпотентность `lastOutboundId/event_id`; `hold=true` — валидные ответы **не теряем** и **эмитим**; `optOut` — входящие **только БД**, наружу **не** шлём.

**20) Готовность к внедрению**
Карта `willCoverLater`; словари/негации; пороги/окна/лимиты; БД/воркеры/outbox; аудитории `CA_QUAL_*` / `CA_EXCLUDE_NOISE`; мониторинг/алерты; тесты: идемпотентность/ретраи/noise/обновления/код-слова/нуджи.

---

\#input3

**Память: как и что используем**

**1) Слой памяти**

* **Короткая (runtime, 24–72ч) → Redis**
  Что кладём: история для LLM, последние N сообщений, быстрые флаги (`retries/hold/optOut`), контекст шага.
  Зачем: дёшево/быстро, низкая латентность, TTL, ключи по `sessionId`.
  Где в Flowise:
  — **Redis Cache** — кэш ответов LLM.
  — **Redis-Backed Chat Memory** — диалог для агента.
* **Долгая (аудит/аналитика/поиск) → Postgres (+pgvector по необходимости)**
  Что кладём: `messages_raw/meta/links`, `funnel_answers`, `events_outbox`, CTWA-метки, заказы/оплаты, «вечная» память фактов.
  Зачем: надёжность, SQL, отчёты, восстановление, RAG (через `pgvector`).
* **(Опц.) Векторный слой → pgvector** — семантический поиск фактов/предпочтений.

**2) Конкретные узлы/параметры в Chatflow**

* **Redis Cache**
  Credential: ваш Redis/Valkey; `TTL: 600–3600s`; `Key prefix: ws:{{$vars.workspaceId}}:cache:`; подключать к ChatOpenAI/модели как Cache.
* **Redis-Backed Chat Memory**
  Credential: тот же Redis; `Session Id: {{$vars.sessionId}}`; `Prefix: ws:{{$vars.workspaceId}}:chat:`; **Window: 8–20 сообщений** *(это **пример**, не согласованная константа)*; подключать в Tool/Conversational Agent → Memory.
* **Переменные (Set/Get Variable)**
  Инициализируем: `workspaceId, tenantId, sessionId, stepKey, retries`.
* **Долгая память** — делает **бэкенд** по вебхукам (Gupshup/Stripe/CRM) в Postgres (+ индекс в pgvector). В Chatflow отдельный узел не нужен.

**3) Политики и изоляция**
Ключи:
`ws:{workspaceId}:chat:{sessionId}` — история;
`ws:{workspaceId}:cache:{hash(prompt)}` — кэш LLM.
TTL: чат-история 24–72ч; кэш LLM 10–60 мин (диалоги) / дольше на статике.
Очистка: при `optOut/stop` — сразу чистим `chat:{sessionId}`.
Суммаризация (опц.): воркер сворачивает длинные чаты в Postgres; в Redis держим `summary+tail`.

**4) Коротко по выбору**
Короткая — Redis; долгая — Postgres (+pgvector). Низкая задержка онлайн + полный аудит/аналитика в БД.

---

\#input4

**Вход WhatsApp ↔ Gupshup ↔ Backend ↔ Agentflow ↔ Chatflow: маршрутизация, онбординг, несколько номеров, зоны ответственности**

**1) «Главный» флоу и старт**
Вход **всегда** в бэкенд (webhook Gupshup), старт **всегда** в **Agentflow**.

**Реестр маршрутизации (БД `routing`)**
Поля: `tenant_id, bm_id, wa_app_id(или wa_phone), entry_type(default|ctwa|keyword), entry_value(ad_id|adset_id|campaign_id|keyword), agentflow_id, chatflow_id_default(опц.), status`.
Уникальность: `(wa_app_id, entry_type, entry_value)` — `UNIQUE`.
Логика: по метаданным входящего (номер/wa\_app\_id + CTWA-теги) бек находит строку и получает `agentflow_id` → вызывает Agentflow `/predict`.

**2) Как бек понимает «это WhatsApp»**
Из webhook: `provider="gupshup"`, `wa_app_id/phone`, тип события (`message`, `template`, `status: delivered|read|failed`). Эти поля идут в `messages_meta` и в роутинг.

**3) Несколько WA-номеров в одном BM + онбординг**
Мастер-онбординг (в админке) и нода «Onboarding» для записи в БД.
Онбординг делает: список WA-аккаунтов/номеров; выбор «привязать существующий»/«завести новый»; выбор главного `Agentflow` (и опц. дефолтного `Chatflow`); запись в `routing`.
Ограничение: `wa_app_id` **UNIQUE** в активных связках (повторная привязка запрещена).

**4) Кто «кому звонит»**
Бек **никому внутри канвы не звонит** — только `/predict` у **Agentflow**. Очерёдность/step/пороги — **в Chatflow** (Variables + Memory). Бек передаёт контекст: `tenantId, orgId, workspaceId, sessionId, wa_app_id/phone, ctwa_*`. Дальше Agentflow → Execute Flow(Chatflow); внутри Chatflow тулами выполняются WA-отправки/эмиссии.

**5) «Страница Sequences» и «нода WhatsApp»**
Делаем внешнюю страницу **Sequences** (конструктор сообщений/кнопок/листов с версионированием). В Flowise — **одна** нода **WA Send (Gupshup)** (Custom Tool), на вход подаётся `sequence_id`/`message_id`; нода сама тянет из БД и отправляет пакет. Порядок сообщений в пакете решает бек (по `sequence_id`), на канве порядок управляет поток.

**6) Где живёт «нода WhatsApp»**
Отправка (text/template/buttons/list/…) — **в Chatflow** (через Tool Agent → WA Send). Короткие ack на код-слова до входа в сценарий — можно и из **Agentflow** той же нодой. «Слушатель» входящих — **только бек** (webhook).

**7) «Нода постоянного прослушивания» и классификация**
Слушатель — бек. Классификация — внутри **Chatflow** (Tool Agent + промпт/правила) или отдельная Intent/Moderation-нода. Статусы доставки (delivered/read/failed) ловит бек и триггерит нуджи (через свой планировщик); в Flowise — только команда `/nudge/schedule`.

**8) Идеология**
Start — только в **Agentflow** (код-слова/глобальные условия) → Execute Flow(Chatflow). В **Chatflow** — бизнес-логика шагов, валидатор, инструменты (WA Send, CAPI, Audiences, CRM, Stripe, Nudge, Operator). Память и шаги — в Chatflow (Redis-Backed Chat Memory + Variables). Мультиплатформа (WA/TG/IG/FBM): одинаковая схема, меняется только транспортный тул.

**9) Множественные сценарии/каналы**
Можно держать несколько Chatflow; в Agentflow через Condition/ветвления выбирать, какой Chatflow дернуть (по номеру/CTWA/keyword). Оператор (HITL) — в Agentflow (ветка operator) + нода «Создать тикет» + Human Input.

**10) Что хранить и где (минимум)**
**БД:** `routing`, `messages_raw/meta`, `funnel_answers`, `events_outbox`, `sequences`, `wa_accounts`.
**Redis:** `org:{orgId}:ws:{wsId}:session:{sessionId}` — история/состояние.
**Variables (Workspace):** `qual_manifest`, `policy` (пороги/ретраи/нуджи), `ids` (пиксель/аудитории/CRM-маппинг), `texts_common`.

---

\#input5

**ДОПОЛНЕНИЯ к #input4 (из ваших фрагментов, без сокращений):**

**A) Внутренности ноды «WA Send (Gupshup)» — «начинка» (6 групп полей)**

1. **Адресация**

* `to` (E.164, +7999… или `contact_id` из CRM)
* `wa_app_id` (подключённый WA-аккаунт Gupshup)
* `project_id / tenant_id / session_id` (маршрутизация/логи)
* `idempotency_key` *(по умолчанию: `sessionId-msg-<hash(message_id+vars)>`)*

2. **Что отправить (контент)**

* `mode: message | sequence`
* `message_id | sequence_id` (ссылки на библиотеку сообщений)
* `vars {}` (подстановки, напр. `{name:"Иван", budget:"5k"}`)
  **(Опц.) Inline для разовых случаев**
* `type:` `text | template | interactive_buttons | interactive_list | media_image | media_video | media_audio | document | location | contact`
* `payload` по типу:

  * `text: { text }`
  * `interactive_buttons: { text, buttons:[{id,title}] }`
  * `interactive_list: { header?, body, sections:[{title, rows:[{id,title,description?}]}] }`
  * `media_*: { url | file_id, caption? }`
  * `template: { template_name, language, components:[…], variables:{…} }`
  * `location: { lat, lng, name?, address? }`
  * `contact: { vcard | fields… }`

3. **Режим доставки (24 часа)**

* `delivery_mode:` `auto | session_only | template_only`
* `fallback_template?` (имя шаблона за пределами 24ч)
* `typing_simulation? on|off, delay_ms?`

4. **Политики и валидация**

* `validate: on|off` (лимиты WA: длины/кнопки/типы)
* `block_if_optout: on|off`
* `respect_quiet_hours: on|off, window:{start,end,tz}`
* `rate_limit_tag?` (троттлинг/BSP лимиты)

5. **Логирование и дебаг**

* `trace_id` *(авто: `sessionId:stepKey:ts`)*
* `dry_run: on|off` — не шлёт, валидирует и отдаёт сгенерированный payload
* `return_raw: on|off` — вернуть сырой ответ Gupshup

6. **Выходы ноды (outputs)**

* `status: queued | sent | failed`
* `outbound_id` (внутренний id)
* `provider_message_id` (id у Gupshup/WA)
* `idempotency_key` (фактически применённый)
* `error: code, message, retry_after?`
* `meta: delivery_mode_used, template_used?, media_info?`

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

**B) Flowise: папки/теги/переменные (нейминг) + изоляция**

* Теги/нейминг:

  * **Agentflow:** `[PRJ:slug] Agent START`
  * **Chatflow:** `[PRJ:slug] Chat QUALIFIER`
* Workspace/Prediction Variables: `projectId, tenantId, wa_app_id, sessionId, ctwa, qual_manifest_id, policy_id`.
* Изоляция состояния/памяти:

  * **Redis key prefix:** `org:{tenantId}:prj:{projectId}:sess:{sessionId}`
  * **pgvector namespace:** фильтр `WHERE tenant_id=? AND project_id=?` (+индексы)

**C) Проекты и роутинг: минимальная схема + A/B (позже)**
**Модель данных (минимум)**

* `projects(id, tenant_id, name, slug, status, default_agentflow_id, default_chatflow_id, qual_manifest_id, policy_id, created_at)`
* `wa_accounts(id, bm_id, wa_app_id, phone_e164, status, created_at)`
* `routing(id, tenant_id, project_id, wa_app_id, entry_type(default|ctwa|keyword), entry_value, agentflow_id, chatflow_id, status, created_at)`
  — `UNIQUE(wa_app_id, entry_type, entry_value)` (исключает двойную привязку).
  **Роутинг входящих**

1. Gupshup webhook → извлечь `wa_app_id/phone, ctwa_*, message/text`.
2. Найти в `routing` по `(wa_app_id, entry_type, entry_value)` (приоритет: `ctwa:ad|adset|campaign` → `keyword` → `default`).
3. Получить `project_id, agentflow_id, chatflow_id`; вызвать Agentflow `/predict` (`question`, Vars: `projectId, tenantId, wa_app_id, sessionId, ctwa_*`).
4. Agentflow → Execute Flow(Chatflow).
   **Онбординг номеров**
   UI: выбрать Проект → WA номер (`wa_accounts`) → создать `routing(default, '*')`. Если `wa_app_id` уже привязан как `default` в активном проекте — запретить.
   **События/эмиссии: указывать `project_id`**
   CAPI/custom\_data, Audiences payload, CRM note/fields, DB (`funnel_answers`, `events_outbox`) — всегда добавлять `project_id`.
   **A/B (позже, без ломки):**

* `routing_variants{project_id, variant_code, weight, agentflow_id, chatflow_id}`
* Sticky-бакет: хеш телефона → `variant_code`
* Вызов разных Chatflow **или** одного Chatflow с разными `qual_manifest_id/policy_id`.

---

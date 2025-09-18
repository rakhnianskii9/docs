## Gupshup Partner: Онбординг WABA — кратко

Назначение: Onboarding APIs позволяют тех‑провайдерам (ISV/TP) онбордить WABA у себя в продукте без захода в интерфейс Gupshup.

### Пререквизиты и токены
- Аккаунт в Gupshup Partner Portal с одобренным Solution ID.
- Один Gupshup Wallet для всех WABA (нельзя создавать WABA в разных кошельках).
- Токены:
	- PARTNER_TOKEN — JWT после логина партнёра (partner‑level API).
	- PARTNER_APP_TOKEN — токен конкретного приложения (app‑level API, подписки и т.п.).

### Ключевые эндпоинты (Onboarding)
- Create App — создать приложение (WABA онбординг), привязано к вашему Partner ID.
- Set subscription (v3) — подписаться на события/сообщения (для Go‑Live нужен режим ACCOUNT).
- Generate Embed Signed Link — сгенерировать embed‑ссылку для прохождения онбординга владельцем WABA.
- Update App — обновить настройки (templateMessaging, storageRegion и пр.).
- Get App / Get App List — получить состояние приложения / список приложений.
- Mark App for Migration — пометить миграцию с другого BSP (синк шаблонов и действия миграции).
- Set Contact Details — задать контакты для приложения.
- Resend Verification Link — переотправить письмо/ссылку для верификации.

См. полный список в API Reference: Partner Portal Onboarding API’s (ссылка ниже).

### Рекомендуемый флоу (end‑to‑end)
1) Создать App.
2) (Если миграция) Mark App for Migration.
3) Поставить подписку (Set subscription v3) на ваш webhook URL:
	 - modes: как минимум ACCOUNT (для Go‑Live), дополнительно TEMPLATE/MESSAGE/ALL/READ/SENT/DELIVERED/ENQUEUED по потребности.
	 - version: 3; tag: уникальный; url: ваш callback; meta: опциональные кастомные заголовки.
	 - Authorization: PARTNER_APP_TOKEN.
4) Сгенерировать Embed Signed Link и выдать клиенту.
5) Клиент проходит онбординг по ссылке.
6) Ловить события (ниже) и убедиться, что пришёл Go‑Live.
7) Проверить состояние через Get App (waId/namespace/live и т.д.).
8) Финальные настройки: Set Contact Details, Update App.
9) При сбоях верификации — Resend Verification Link; следить за 401/429/500.

### Подписки и события
- Set subscription v3: режимы TEMPLATE, ACCOUNT, MESSAGE/ALL/READ/SENT/DELIVERED/ENQUEUED, PAYMENTS, FLOWS_MESSAGE, OTHERS, BILLING, FAILED (см. справку по режимам в Reference).
- Meta (поле meta) позволяет добавить заголовки, которые платформа будет присылать вашему webhook — удобно для аутентификации.
- Go‑Live (System Events): придёт type="onboarding-event" с payload.type="docker-status-event" и status="live", плюс wabaId и namespace.
- Прочие account‑events: review‑event (APPROVED/REJECTED), status‑event (ACCOUNT_VIOLATION/DISABLE/RESTRICTED/VERIFIED), pndn‑event (статусы номера/имени), tier‑event (лимиты), capability‑event (возможности/лимиты).
- Template‑events: status‑update, category‑update, quality‑update; авто‑миграция категорий Meta сопровождается двумя вебхуками (alert + фактическое изменение в следующем месяце).
- V3 message events: ENQUEUED, SENT, DELIVERED, READ, FAILED — при соответствующих подписках.

### Полезные ссылки (официальная документация)
- Onboarding APIs (гид): https://partner-docs.gupshup.io/docs/onboarding-apis
- API Reference — Partner Portal Onboarding API’s: https://partner-docs.gupshup.io/reference/app-onboarding-apis
- Set subscription (v3): https://partner-docs.gupshup.io/reference/setsubscription-api-v3
- System Events (включая Go‑Live): https://partner-docs.gupshup.io/docs/system-events
- Incoming Events (V3 message statuses): https://partner-docs.gupshup.io/docs/v3-events
- API Reference (каталог): https://partner-docs.gupshup.io/reference
- Introduction (обзор Partner Portal): https://partner-docs.gupshup.io/docs/introduction

### Заметки
- Подписки можно ставить в Sandbox; при Go‑Live сохраняются.
- Возвращайте 2xx быстро в webhook, делайте идемпотентность и логируйте события.
- Правило одного Wallet для WABA обязательно.

## Партнерский токен: получение и управление (реализация в Flowise)

Ниже — точное соответствие текущему коду сервера Flowise (ветка dev) по получению/обновлению PARTNER_TOKEN и его использованию во всех Gupshup Partner API вызовах.

### Где хранится токен
- Таблица: `gupshup_partner_token` (PostgreSQL), миграция: `packages/server/src/enterprise/database/migrations/postgres/1750200000002-AddGupshupPartnerToken.ts`.
- Схема полей:
	- `id` VARCHAR(50) PK, используется фиксированное значение `partner_jwt`.
	- `token_value` TEXT — актуальный PARTNER_TOKEN (JWT).
	- `expires_at` TIMESTAMP — срок действия токена (из поля exp JWT; если exp отсутствует — задаётся дефолт ~23ч).
	- `refresh_count` INTEGER — счётчик обновлений.
	- `last_error` TEXT — последняя ошибка обновления (на будущее, сейчас заполняется только при расширенном сервисе).
	- `created_at`, `updated_at` TIMESTAMP — служебные.
- При миграции сидируется пустая запись с `id='partner_jwt'` (если отсутствует).

### Сервис управления токеном
- Файл: `packages/server/src/services/gupshup/partnerToken.service.ts`.
- Ключевые методы:
	- `refreshPartnerToken()` — логинится в Partner API и записывает новый токен в БД (upsert по `id='partner_jwt'`).
	- `getValidPartnerToken()` — выдаёт токен из БД; если истёк — обновляет; если < 1ч до истечения — запускает фоновое обновление.
	- `startAutoRefresh()` — при старте сервера проактивно обновляет токен и затем планирует автообновление каждые 23 часа.
	- `stopAutoRefresh()` — останавливает таймер автообновления.
- Алгоритм определения срока жизни:
	- Токен парсится как JWT (`token.split('.')[1]` ⇒ JSON.parse(payload)). Если есть `exp` — берётся как `expires_at`; иначе выставляется `now + 23h`.
	- Важно: в текущем коде парсинг JWT не обёрнут в try/catch, поэтому если токен не‑JWT — операция упадёт на JSON.parse; это ожидается, так как Partner API возвращает JWT.
- Источник DataSource: через `getRunningExpressApp().AppDataSource`.

### Как получается токен у Gupshup
- Endpoint: `POST {GUPSHUP_PARTNER_BASE_URL || https://partner.gupshup.io/partner}/account/login`
- Тело: `Content-Type: application/x-www-form-urlencoded`
	- `email`: `GUPSHUP_PARTNER_USER_ID`
	- `password`: `GUPSHUP_CLIENT_SECRET` или `GUPSHUP_PARTNER_PASSWORD`
- Ответ ожидается вида `{ token: <JWT> }`.

### Клиент для Partner API
- Файл: `packages/server/src/services/gupshup/partner.ts`.
- Создаёт `axios` с `baseURL` и `timeout=20000`.
- Заголовок авторизации: `token: <PARTNER_TOKEN>` (не Bearer; так требует Gupshup Partner API).
- Токен-менеджмент в клиенте:
	- Перед каждым вызовом — `ensureToken()` берёт токен из БД через `gupshupPartnerTokenService.getValidPartnerToken()`; при отсутствии — обновляет по кредам.
	- Перехватчик `401` повторяет запрос: сначала пытается взять валидный токен из БД, иначе делает принудительное обновление.
	- Примечание: прямое обновление в клиенте не записывает токен обратно в БД (основной стор — сервис). Это ок, т.к. нормальный путь — через сервис; прямой путь — fallback.
- Реализованные вызовы:
	- `createApp` POST `/app`
	- `updateApp` PUT `/app/{appId}`
	- `getAppDetails` GET `/app/{appId}/details`
	- `listApps` GET `/app/list`
	- `setContactDetails` PUT `/app/{appId}/onboarding/contact`
	- `resendEmailVerification` POST `/app/{appId}/onboarding/contact/email/resend`
	- `generateEmbedLink` GET `/app/{appId}/onboarding/embed/link`
	- `markAppForMigration` POST `/app/{appId}/onboarding/phoneMigration`

### Серверные маршруты Flowise (внешний API)
- Файл: `packages/server/src/enterprise/routes/gupshup.route.ts`, смонтирован под `/api/v1/gupshup/*` в `packages/server/src/routes/index.ts`.
- Аутентификация: whitelist разрешает путь, но внутри требуется один из вариантов:
	- Bearer HS256: `Authorization: Bearer <HS256(jwt)>`, подпись `SHARED_SSO_SECRET`/`SSO_SHARED_SECRET`; payload должен содержать `email`.
	- Либо заголовок `x-user-email`.
- Маршруты и контракт:
	- `POST /onboarding/start` — body: `{ name: string, contact?: object }` ⇒ `{ appId, embed? }`. В БД создаётся/обновляется запись в служебной таблице `ee_gs_app` (временная, создаётся в рантайме, не миграция) с привязкой к email.
	- `PUT /onboarding/contact` — body: `{ appId: string, contact: object }` ⇒ `{ ok: true, result }`.
	- `GET /onboarding/embed-link?appId=...` ⇒ `{ appId, link }`.
	- `GET /onboarding/status?appId=...` ⇒ `{ appId, live, details }`. Если `live=true` и нет привязки к пользователю/организации/воркспейсу — выполняется автопровижининг через `UserService/OrganizationUserService/WorkspaceUserService` и сохраняется связь в `ee_gs_app`.
	- `POST /onboarding/resend-email` — body: `{ appId }` ⇒ `{ ok: true, result }`.
	- `POST /onboarding/migrate` — body: `{ appId, countryCode, phone, previousBsp? }` ⇒ `{ ok: true, result }`.
	- `POST /webhook` — заглушка: логирует тело и, если найден `appId`, пишет снапшот в `ee_gs_app`.

### Переменные окружения (Flowise)
- Авторизация партнёра:
	- `GUPSHUP_PARTNER_USER_ID` — email партнёрского аккаунта.
	- `GUPSHUP_CLIENT_SECRET` — client secret (может быть `GUPSHUP_PARTNER_PASSWORD`).
	- `GUPSHUP_PARTNER_BASE_URL` — база для Partner API (по умолчанию `https://partner.gupshup.io/partner`).
	- (опц.) `GUPSHUP_PARTNER_TOKEN` — токен из вне, если хотим стартовать без логина.
- Безопасность SSO между Vuexy → Flowise:
	- `SHARED_SSO_SECRET` или `SSO_SHARED_SECRET` — общий секрет для HS256.

### Наблюдаемость и планировщик
- При старте сервера (`packages/server/src/index.ts`) выполняется:
	- Инициализация БД и миграций.
	- Неблокирующий вызов `gupshupPartnerTokenService.refreshPartnerToken()`.
	- Запуск `gupshupPartnerTokenService.startAutoRefresh()` (интервал ~23ч).
- При оставшемся сроке < 1ч `getValidPartnerToken()` инициирует фоновое обновление.

### Несоответствия и TODO (по коду)
- Парсинг JWT в `partnerToken.service.ts` не защищён try/catch. Если ответ не‑JWT — `JSON.parse` выбросит исключение. В реальности Partner API возвращает JWT — но стоит добавить защиту.
- Прямое обновление токена внутри клиента `partner.ts` не записывает токен в БД (считается аварийным путём). Рекомендуется централизовать запись через сервис или добавить upsert при клиентском refresh.
- `ee_gs_app` — временная служебная таблица, создаётся в рантайме, вне миграций. Для прод‑надёжности лучше оформить миграцией и нормализовать связи.
- Подписка Set Subscription v3 ещё не обёрнута в клиент/роуты (в этом репозитории); в техдоке шаг есть — потребуется добавить метод и маршрут.
- Нейминг ответа embed‑ссылки: сервер отдаёт `{ link }` (в `GET /onboarding/embed-link`) и `{ embed }` (в `POST /onboarding/start`), в Vuexy‑доке упоминается `{ embedUrl }`. На стороне Vuexy уже есть нормализация, но в Flowise‑API стоит унифицировать поле.

### Проверочный чек‑лист (быстрое само‑аудит)
- [ ] В .env заданы `GUPSHUP_PARTNER_USER_ID` и `GUPSHUP_CLIENT_SECRET`.
- [ ] Миграции применены, таблица `gupshup_partner_token` создана, запись `partner_jwt` присутствует.
- [ ] При старте в логах видно попытку refresh и запуск автообновления.
- [ ] Вызовы Partner API содержат заголовок `token: <PARTNER_TOKEN>`.
- [ ] SSO‑секрет (`SHARED_SSO_SECRET`/`SSO_SHARED_SECRET`) совпадает с Vuexy, а Bearer‑payload содержит `email`.
- [ ] Маршруты `/api/v1/gupshup/*` доступны с Bearer HS256 или `x-user-email`.

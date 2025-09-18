# Facebook Insights API - Полный список показателей

## Обзор

Facebook Insights API предоставляет единый интерфейс для получения статистики рекламы. API может возвращать приблизительные метрики, метрики в разработке или комбинированные показатели.

## Базовые показатели

### Основные метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `account_currency` | string | Валюта рекламного аккаунта |
| `account_id` | numeric string | ID рекламного аккаунта |
| `account_name` | string | Название рекламного аккаунта |
| `ad_id` | numeric string | Уникальный ID рекламного объявления |
| `ad_name` | string | Название рекламного объявления |
| `adset_id` | numeric string | Уникальный ID группы объявлений |
| `adset_name` | string | Название группы объявлений |
| `campaign_id` | numeric string | Уникальный ID кампании |
| `campaign_name` | string | Название кампании |
| `date_start` | string | Дата начала периода данных |
| `date_stop` | string | Дата окончания периода данных |
| `impressions` | numeric string | Количество показов рекламы |
| `reach` | numeric string | Количество людей, увидевших рекламу хотя бы один раз (приблизительно) |
| `frequency` | numeric string | Среднее количество показов рекламы для каждого человека (приблизительно) |
| `clicks` | numeric string | Количество кликов по рекламе |
| `spend` | numeric string | Потраченная сумма на рекламу (приблизительно) |

### Дополнительные основные метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `total_actions` | numeric string | Общее количество действий |
| `unique_clicks` | numeric string | Количество уникальных кликов (приблизительно) |
| `unique_ctr` | numeric string | CTR по уникальным кликам (приблизительно) |
| `unique_link_clicks_ctr` | numeric string | CTR по уникальным кликам ссылок (приблизительно) |
| `unique_actions` | list<AdsActionStats> | Уникальные действия (приблизительно) |
| `unique_conversions` | list<AdsActionStats> | Уникальные конверсии (приблизительно) |
| `unique_outbound_clicks` | list<AdsActionStats> | Уникальные аутбаунд клики (приблизительно) |
| `unique_outbound_clicks_ctr` | list<AdsActionStats> | CTR по уникальным аутбаунд кликам (приблизительно) |
| `unique_inline_link_clicks` | numeric string | Уникальные клики по встроенным ссылкам (приблизительно) |
| `unique_inline_link_click_ctr` | numeric string | CTR по уникальным встроенным ссылкам (приблизительно) |

### Финансовые показатели

| Поле | Тип | Описание |
|------|-----|----------|
| `cpc` | numeric string | Средняя стоимость за клик |
| `cpm` | numeric string | Средняя стоимость за 1000 показов |
| `cpp` | numeric string | Средняя стоимость охвата 1000 человек (приблизительно) |
| `ctr` | numeric string | CTR - процент кликов от показов |

### Показатели кликов

| Поле | Тип | Описание |
|------|-----|----------|
| `inline_link_clicks` | numeric string | Количество кликов по ссылкам в рекламе |
| `inline_link_click_ctr` | numeric string | CTR по встроенным ссылкам |
| `cost_per_inline_link_click` | numeric string | Средняя стоимость клика по встроенной ссылке |
| `cost_per_unique_click` | numeric string | Средняя стоимость уникального клика (приблизительно) |
| `cost_per_unique_inline_link_click` | numeric string | Средняя стоимость уникального клика по встроенной ссылке (приблизительно) |

### Показатели взаимодействий

| Поле | Тип | Описание |
|------|-----|----------|
| `inline_post_engagement` | numeric string | Общее количество взаимодействий с рекламой |
| `cost_per_inline_post_engagement` | numeric string | Средняя стоимость взаимодействия с публикацией |

## Видео метрики

### Основные видео показатели

| Поле | Тип | Описание |
|------|-----|----------|
| `video_30_sec_watched_actions` | list<AdsActionStats> | Количество просмотров видео в течение 30+ секунд |
| `video_p25_watched_actions` | list<AdsActionStats> | Количество просмотров 25% видео |
| `video_p50_watched_actions` | list<AdsActionStats> | Количество просмотров 50% видео |
| `video_p75_watched_actions` | list<AdsActionStats> | Количество просмотров 75% видео |
| `video_p95_watched_actions` | list<AdsActionStats> | Количество просмотров 95% видео |
| `video_p100_watched_actions` | list<AdsActionStats> | Количество полных просмотров видео |
| `video_play_actions` | list<AdsActionStats> | Количество запусков видео |
| `video_avg_time_watched_actions` | list<AdsActionStats> | Среднее время просмотра видео |
| `video_continuous_2_sec_watched_actions` | list<AdsActionStats> | Количество непрерывных 2-секундных просмотров |
| `video_time_watched_actions` | list<AdsActionStats> | Общее время просмотра видео |

### Расширенные видео метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `video_play_curve_actions` | list<AdsHistogramStats> | График кривой воспроизведения видео |
| `video_play_retention_0_to_15s_actions` | list<AdsHistogramStats> | Удержание аудитории 0-15 секунд |
| `video_play_retention_20_to_60s_actions` | list<AdsHistogramStats> | Удержание аудитории 20-60 секунд |
| `video_play_retention_graph_actions` | list<AdsHistogramStats> | График удержания аудитории |

### Стоимостные видео метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `cost_per_15_sec_video_view` | list<AdsActionStats> | Стоимость 15-секундного просмотра видео |
| `cost_per_2_sec_continuous_video_view` | list<AdsActionStats> | Стоимость 2-секундного непрерывного просмотра |
| `cost_per_thruplay` | list<AdsActionStats> | Стоимость ThruPlay (в разработке) |

## Конверсионные метрики

### Общие конверсии

| Поле | Тип | Описание |
|------|-----|----------|
| `actions` | list<AdsActionStats> | Общее количество действий, приписываемых рекламе |
| `action_values` | list<AdsActionStats> | Общая стоимость всех конверсий |
| `conversions` | list<AdsActionStats> | Количество конверсий |
| `conversion_values` | list<AdsActionStats> | Стоимость конверсий |
| `cost_per_action_type` | list<AdsActionStats> | Средняя стоимость релевантного действия |
| `cost_per_conversion` | list<AdsActionStats> | Средняя стоимость конверсии |
| `cost_per_unique_action_type` | list<AdsActionStats> | Средняя стоимость уникального действия (приблизительно) |
| `cost_per_unique_conversion` | list<AdsActionStats> | Средняя стоимость уникальной конверсии |

### Аутбаунд метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `outbound_clicks` | list<AdsActionStats> | Клики, уводящие с платформ Facebook |
| `outbound_clicks_ctr` | list<AdsActionStats> | CTR по аутбаунд кликам |
| `cost_per_outbound_click` | list<AdsActionStats> | Средняя стоимость аутбаунд клика |
| `cost_per_unique_outbound_click` | list<AdsActionStats> | Средняя стоимость уникального аутбаунд клика (приблизительно) |

### Веб-сайт метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `website_ctr` | list<AdsActionStats> | CTR по кликам на веб-сайт |
| `website_purchase_roas` | list<AdsActionStats> | ROAS от покупок на веб-сайте |

## Мобильные приложения

### App метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `mobile_app_purchase_roas` | list<AdsActionStats> | ROAS от покупок в мобильном приложении |
| `app_store_clicks` | numeric string | Клики в магазин приложений (устарело) |

### Catalog метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `catalog_segment_actions` | list<AdsActionStats> | Действия по сегменту каталога |
| `catalog_segment_value` | list<AdsActionStats> | Стоимость конверсий сегмента каталога |
| `catalog_segment_value_mobile_purchase_roas` | list<AdsActionStats> | ROAS мобильных покупок сегмента каталога |
| `catalog_segment_value_omni_purchase_roas` | list<AdsActionStats> | Общий ROAS покупок сегмента каталога |
| `catalog_segment_value_website_purchase_roas` | list<AdsActionStats> | ROAS веб-покупок сегмента каталога |

### Устаревшие метрики (недоступны при разбивках)

| Поле | Тип | Описание |
|------|-----|----------|
| `app_store_clicks` | numeric string | Клики в магазин приложений |
| `newsfeed_avg_position` | numeric string | Средняя позиция в ленте новостей |
| `newsfeed_clicks` | numeric string | Клики в ленте новостей |
| `newsfeed_impressions` | numeric string | Показы в ленте новостей |
| `relevance_score` | object | Оценка релевантности рекламы |

## Продуктовые метрики

### Конвертированные продукты

| Поле | Тип | Описание |
|------|-----|----------|
| `converted_product_quantity` | list<AdsActionStats> | Количество купленных продуктов |
| `converted_product_value` | list<AdsActionStats> | Стоимость купленных продуктов |
| `converted_product_app_custom_event_fb_mobile_purchase` | list<AdsActionStats> | Покупки продуктов через мобильное приложение |
| `converted_product_app_custom_event_fb_mobile_purchase_value` | list<AdsActionStats> | Стоимость покупок продуктов через мобильное приложение |
| `converted_product_offline_purchase` | list<AdsActionStats> | Офлайн покупки продуктов |
| `converted_product_offline_purchase_value` | list<AdsActionStats> | Стоимость офлайн покупок продуктов |
| `converted_product_omni_purchase` | list<AdsActionStats> | Омни-канальные покупки продуктов |
| `converted_product_omni_purchase_values` | list<AdsActionStats> | Стоимость омни-канальных покупок |
| `converted_product_website_pixel_purchase` | list<AdsActionStats> | Покупки продуктов через веб-сайт (пиксель) |
| `converted_product_website_pixel_purchase_value` | list<AdsActionStats> | Стоимость покупок продуктов через веб-сайт |

### Продвигаемые продукты

| Поле | Тип | Описание |
|------|-----|----------|
| `converted_promoted_product_quantity` | list<AdsActionStats> | Количество купленных продвигаемых продуктов |
| `converted_promoted_product_value` | list<AdsActionStats> | Стоимость купленных продвигаемых продуктов |
| `converted_promoted_product_app_custom_event_fb_mobile_purchase` | list<AdsActionStats> | Покупки продвигаемых продуктов через приложение |
| `converted_promoted_product_app_custom_event_fb_mobile_purchase_value` | list<AdsActionStats> | Стоимость покупок продвигаемых продуктов через приложение |
| `converted_promoted_product_offline_purchase` | list<AdsActionStats> | Офлайн покупки продвигаемых продуктов |
| `converted_promoted_product_offline_purchase_value` | list<AdsActionStats> | Стоимость офлайн покупок продвигаемых продуктов |
| `converted_promoted_product_omni_purchase` | list<AdsActionStats> | Омни-канальные покупки продвигаемых продуктов |
| `converted_promoted_product_omni_purchase_values` | list<AdsActionStats> | Стоимость омни-канальных покупок продвигаемых продуктов |
| `converted_promoted_product_website_pixel_purchase` | list<AdsActionStats> | Покупки продвигаемых продуктов через веб-сайт |
| `converted_promoted_product_website_pixel_purchase_value` | list<AdsActionStats> | Стоимость покупок продвигаемых продуктов через веб-сайт |

## Результативность и цели

### Результаты кампании

| Поле | Тип | Описание |
|------|-----|----------|
| `objective` | string | Цель кампании |
| `optimization_goal` | string | Цель оптимизации |
| `buying_type` | string | Тип закупки рекламы |
| `objective_results` | list<AdsInsightsResult> | Количество результатов по цели кампании |
| `objective_result_rate` | list<AdsInsightsResult> | Коэффициент результатов по цели |
| `cost_per_objective_result` | list<AdsInsightsResult> | Стоимость результата по цели |
| `results` | list<AdsInsightsResult> | Количество результатов |
| `result_rate` | list<AdsInsightsResult> | Коэффициент результатов |
| `cost_per_result` | list<AdsInsightsResult> | Стоимость результата |

### ROAS метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `purchase_roas` | list<AdsActionStats> | Общий ROAS от покупок |

## Интерактивные и специальные метрики

### Canvas/Instant Experience

| Поле | Тип | Описание |
|------|-----|----------|
| `canvas_avg_view_percent` | numeric string | Средний процент просмотра Instant Experience |
| `canvas_avg_view_time` | numeric string | Среднее время просмотра Instant Experience в секундах |
| `instant_experience_clicks_to_open` | numeric string | Клики для открытия Instant Experience |
| `instant_experience_clicks_to_start` | numeric string | Клики для запуска Instant Experience |
| `instant_experience_outbound_clicks` | list<AdsActionStats> | Аутбаунд клики из Instant Experience |

### Интерактивные компоненты

| Поле | Тип | Описание |
|------|-----|----------|
| `interactive_component_tap` | list<AdsActionStats> | Нажатия на интерактивные компоненты |

### Представления страниц

| Поле | Тип | Описание |
|------|-----|----------|
| `full_view_impressions` | numeric string | Количество полных просмотров публикаций страницы |
| `full_view_reach` | numeric string | Количество людей, совершивших полный просмотр |

### Landing Page метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `landing_page_view_per_link_click` | numeric string | Просмотры посадочной страницы на клик по ссылке |
| `purchase_per_landing_page_view` | numeric string | Покупки на просмотр посадочной страницы |

## Социальные и вовлеченность метрики

### Социальная активность

| Поле | Тип | Описание |
|------|-----|----------|
| `social_spend` | numeric string | Потрачено на рекламу с социальной информацией |

### Instagram метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `instagram_upcoming_event_reminders_set` | numeric string | Установленные напоминания о предстоящих событиях в Instagram |

### Качественные лиды

| Поле | Тип | Описание |
|------|-----|----------|
| `qualifying_question_qualify_answer_rate` | numeric string | Коэффициент качественных ответов на квалифицирующие вопросы |

### Маркетинговые сообщения

| Поле | Тип | Описание |
|------|-----|----------|
| `marketing_messages_delivery_rate` | numeric string | Коэффициент доставки маркетинговых сообщений |

## Продвинутые метрики атрибуции

### DDA метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `dda_countby_convs` | numeric string | Количество конверсий по модели DDA |
| `cost_per_dda_countby_convs` | numeric string | Стоимость конверсии по модели DDA |
| `dda_results` | list<AdsInsightsDdaResult> | Результаты модели DDA |

## Креативные и ассеты метрики

### Ассеты рекламы

| Поле | Тип | Описание |
|------|-----|----------|
| `body_asset` | AdAssetBody | Текст рекламы |
| `description_asset` | AdAssetDescription | Описание рекламы |
| `image_asset` | AdAssetImage | Изображение рекламы |
| `media_asset` | AdAssetMedia | Медиа-ассет рекламы |
| `title_asset` | AdAssetTitle | Заголовок рекламы |
| `video_asset` | AdAssetVideo | Видео-ассет рекламы |
| `rule_asset` | AdAssetRule | Правило для ассета |

### Метрики креативов

| Поле | Тип | Описание |
|------|-----|----------|
| `ad_format_asset` | string | Формат рекламного ассета |
| `creative_relaxation_asset_type` | string | Тип ассета креативной релаксации |
| `flexible_format_asset_type` | string | Тип ассета гибкого формата |
| `gen_ai_asset_type` | string | Тип ассета генеративного ИИ |

### Продуктовые просмотры и карточки

| Поле | Тип | Описание |
|------|-----|----------|
| `product_views` | string | Просмотры продуктов |
| `total_card_view` | string | Общие просмотры карточек |

### Магазины

| Поле | Тип | Описание |
|------|-----|----------|
| `shops_assisted_purchases` | string | Покупки с помощью магазинов |

## Аукционные и конкурентные метрики

### Аукционные данные

| Поле | Тип | Описание |
|------|-----|----------|
| `auction_bid` | numeric string | Ставка на аукционе |
| `auction_competitiveness` | numeric string | Конкурентоспособность на аукционе |
| `auction_max_competitor_bid` | numeric string | Максимальная ставка конкурента |
| `wish_bid` | numeric string | Желаемая ставка |

## Атрибуция и настройки

### Атрибуционные настройки

| Поле | Тип | Описание |
|------|-----|----------|
| `attribution_setting` | string | Настройка атрибуции по умолчанию |

## Клики и взаимодействия с рекламой

### Расширенные клики

| Поле | Тип | Описание |
|------|-----|----------|
| `ad_click_actions` | list<AdsActionStats> | Действия кликов по рекламе |
| `ad_impression_actions` | list<AdsActionStats> | Действия показов рекламы |
| `cost_per_ad_click` | list<AdsActionStats> | Стоимость клика по рекламе |
| `cost_per_one_thousand_ad_impression` | list<AdsActionStats> | Стоимость 1000 показов рекламы |

## Сравнение и производительность

### Сравнительные метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `comparison_node` | AdsInsightsComparison | Узел для сравнения временных диапазонов |
| `result_values_performance_indicator` | string | Индикатор производительности значений результатов |

## Метаданные и временные метки

### Временные метки

| Поле | Тип | Описание |
|------|-----|----------|
| `created_time` | string | Время создания |
| `updated_time` | string | Время обновления |

## Типы действий (Action Types)

### Основные типы действий

Поле `action_type` может содержать следующие значения:

#### Мобильные события приложений (app_custom_event)
- `app_custom_event.fb_mobile_achievement_unlocked` - Разблокировка достижений в мобильном приложении
- `app_custom_event.fb_mobile_activate_app` - Запуски мобильного приложения
- `app_custom_event.fb_mobile_add_payment_info` - Добавление платежных данных в мобильном приложении
- `app_custom_event.fb_mobile_add_to_cart` - Добавления в корзину в мобильном приложении
- `app_custom_event.fb_mobile_add_to_wishlist` - Добавления в список желаний в мобильном приложении
- `app_custom_event.fb_mobile_complete_registration` - Регистрации в мобильном приложении
- `app_custom_event.fb_mobile_content_view` - Просмотры контента в мобильном приложении
- `app_custom_event.fb_mobile_initiated_checkout` - Оформления заказа в мобильном приложении
- `app_custom_event.fb_mobile_level_achieved` - Достижения уровней в мобильном приложении
- `app_custom_event.fb_mobile_purchase` - Покупки в мобильном приложении

#### Основные действия (On-Facebook)
- `link_click` - Клики по ссылкам в рекламе
- `page_engagement` - Взаимодействие со страницей Facebook
- `post_engagement` - Взаимодействие с публикацией (включает лайки, комментарии, реакции)
- `mobile_app_install` - Установки мобильного приложения
- `comment` - Комментарии к публикациям или рекламе
- `like` - Лайки публикаций или страниц
- `post_reaction` - Реакции на публикацию (Like, Love, Haha, Wow, Sad, Angry)
- `video_view` - Просмотры видео
- `page_like` - Лайки страницы Facebook
- `checkin` - Отметки в местах
- `rsvp` - Ответы на события
- `share` - Репосты публикаций

#### Стандартные события пикселя (offsite_conversion.fb_pixel_*)
События, отслеживаемые Facebook Pixel на веб-сайтах:
- `offsite_conversion.fb_pixel_add_payment_info` - Добавление платежной информации
- `offsite_conversion.fb_pixel_add_to_cart` - Добавление в корзину
- `offsite_conversion.fb_pixel_add_to_wishlist` - Добавление в список желаний
- `offsite_conversion.fb_pixel_complete_registration` - Завершение регистрации
- `offsite_conversion.fb_pixel_contact` - Обращение/контакт
- `offsite_conversion.fb_pixel_customize_product` - Настройка продукта
- `offsite_conversion.fb_pixel_donate` - Пожертвование
- `offsite_conversion.fb_pixel_find_location` - Поиск местоположения
- `offsite_conversion.fb_pixel_initiate_checkout` - Начало оформления заказа
- `offsite_conversion.fb_pixel_lead` - Генерация лида
- `offsite_conversion.fb_pixel_purchase` - Покупка
- `offsite_conversion.fb_pixel_schedule` - Планирование/запись
- `offsite_conversion.fb_pixel_search` - Поиск на сайте
- `offsite_conversion.fb_pixel_start_trial` - Начало пробного периода
- `offsite_conversion.fb_pixel_submit_application` - Отправка заявки
- `offsite_conversion.fb_pixel_subscribe` - Подписка
- `offsite_conversion.fb_pixel_view_content` - Просмотр контента

#### Пользовательские конверсии (offsite_conversion.custom.*)
- `offsite_conversion.custom.{ID}` - Пользовательские конверсии с уникальными ID

#### События лидов и форм
- `lead` - Генерация лида через Facebook Lead Ads
- `landing_page_view` - Просмотр посадочной страницы
- `onsite_conversion.fb_pixel_initiate_checkout` - Начало оформления заказа (on-site)
- `onsite_conversion.fb_pixel_purchase` - Покупка (on-site)

#### Дополнительные on-Facebook действия
- `app_engagement` - Взаимодействие с приложением
- `app_story` - История приложения
- `call_confirmed` - Подтвержденный звонок
- `checkin` - Отметки в местах
- `credit_spent` - Потраченные кредиты
- `game_plays` - Игровые сессии
- `rsvp` - Ответы на события
- `share` - Репосты публикаций
- `message` - Сообщения
- `follow` - Подписки
- `directions` - Запросы направлений к бизнесу

#### Messenger и бизнес-сообщения
- `messaging_conversation_started_7d` - Начатые разговоры в Messenger (7 дней)
- `messaging_first_reply` - Первые ответы в Messenger
- `messaging_blocked` - Заблокированные разговоры в Messenger

#### Instagram Shopping и коммерция
- `shop_view` - Просмотры магазина Instagram
- `shop_engagement` - Взаимодействие с магазином Instagram

### Поля разбивок действий

#### Базовые поля AdsActionStats
- `action_type` - Тип действия
- `action_device` - Устройство, на котором произошло действие
- `action_destination` - Место назначения после клика
- `action_target_id` - ID места назначения
- `action_carousel_card_id` - ID карточки карусели
- `action_carousel_card_name` - Название карточки карусели
- `action_canvas_component_name` - Название компонента Canvas
- `action_reaction` - Тип реакции
- `action_video_sound` - Статус звука при воспроизведении видео
- `action_video_type` - Тип видео метрики

#### Окна атрибуции
- `1d_click` - 1 день после клика
- `1d_view` - 1 день после просмотра
- `1d_ev` - 1 день после вовлеченного просмотра
- `7d_click` - 7 дней после клика
- `7d_view` - 7 дней после просмотра
- `28d_click` - 28 дней после клика
- `28d_view` - 28 дней после просмотра
- `dda` - Модель, основанная на данных
- `incrementality` - Инкрементальная атрибуция
- `inline` - Встроенная атрибуция
- `value` - Значение окна атрибуции по умолчанию

#### Типы конверсий
- `*_all_conversions` - Все конверсии (считает каждую конверсию)
- `*_first_conversion` - Первые конверсии (считает только первую конверсию)

## Метрики товарных разбивок

### Товарные поля
- `product_id` - ID продукта
- `product_name` - Название продукта
- `product_brand` - Бренд продукта
- `product_category` - Категория продукта
- `product_content_id` - ID контента продукта
- `product_retailer_id` - ID продукта у ритейлера
- `product_group_content_id` - ID группы продуктов
- `product_group_retailer_id` - ID группы продуктов у ритейлера
- `product_custom_label_0` до `product_custom_label_4` - Пользовательские метки продуктов

## Общие разбивки (Breakdowns)

### Список всех доступных разбивок

| Разбивка | Описание |
|----------|----------|
| `action_canvas_component_name` | Название компонента Canvas |
| `action_carousel_card_id` | ID карточки карусели |
| `action_carousel_card_name` | Название карточки карусели |
| `action_destination` | Место назначения действия |
| `action_device` | Устройство действия |
| `action_reaction` | Тип реакции |
| `action_target_id` | ID цели действия |
| `action_type` | Тип действия |
| `action_video_sound` | Звук видео при действии |
| `action_video_type` | Тип видео действия |
| `ad_format_asset` | Формат рекламного ассета |
| `age` | Возраст аудитории |
| `app_id` | ID приложения |
| `body_asset` | Текст рекламы |
| `coarse_conversion_value` | Грубое значение конверсии |
| `country` | Страна |
| `creative_relaxation_asset_type` | Тип ассета креативной релаксации |
| `description_asset` | Описание рекламы |
| `device_platform` | Платформа устройства |
| `dma` | Designated Market Area |
| `fidelity_type` | Тип точности SKAdNetwork |
| `flexible_format_asset_type` | Тип ассета гибкого формата |
| `frequency_value` | Значение частоты |
| `gender` | Пол аудитории |
| `gen_ai_asset_type` | Тип ассета генеративного ИИ |
| `hourly_stats_aggregated_by_advertiser_time_zone` | Почасовая статистика по часовому поясу рекламодателя |
| `hourly_stats_aggregated_by_audience_time_zone` | Почасовая статистика по часовому поясу аудитории |
| `hsid` | Hierarchical Source Identifier |
| `image_asset` | Изображение рекламы |
| `impression_device` | Устройство показа |
| `is_auto_advance` | Автопродвижение |
| `media_asset` | Медиа-ассет |
| `platform_position` | Позиция размещения |
| `postback_sequence_index` | Порядок постбеков SKAdNetwork |
| `product_brand` | Бренд продукта |
| `product_category` | Категория продукта |
| `product_content_id` | ID контента продукта |
| `product_custom_label_0` | Пользовательская метка продукта 0 |
| `product_custom_label_1` | Пользовательская метка продукта 1 |
| `product_custom_label_2` | Пользовательская метка продукта 2 |
| `product_custom_label_3` | Пользовательская метка продукта 3 |
| `product_custom_label_4` | Пользовательская метка продукта 4 |
| `product_group_content_id` | ID группы продуктов |
| `product_group_retailer_id` | ID группы продуктов у ритейлера |
| `product_id` | ID продукта |
| `product_name` | Название продукта |
| `product_retailer_id` | ID продукта у ритейлера |
| `publisher_platform` | Платформа публикации |
| `redownload` | Флаг повторной загрузки |
| `region` | Регион |
| `rule_asset` | Правило для ассета |
| `skan_conversion_id` | ID конверсии SKAdNetwork |
| `skan_version` | Версия SKAdNetwork |
| `title_asset` | Заголовок рекламы |
| `video_asset` | Видео-ассет |

### Разбивки действий (Action Breakdowns)

| Разбивка | Описание |
|----------|----------|
| `action_canvas_component_name` | Название компонента Canvas |
| `action_carousel_card_id` | ID карточки карусели |
| `action_carousel_card_name` | Название карточки карусели |
| `action_destination` | Место назначения действия |
| `action_device` | Устройство, на котором произошло действие |
| `action_reaction` | Тип реакции (Like, Love, Haha, Wow, Sad, Angry) |
| `action_target_id` | ID цели действия |
| `action_type` | Тип действия |
| `action_video_sound` | Статус звука при воспроизведении видео |
| `action_video_type` | Тип видео метрики |
| `conversion_destination` | Место назначения конверсии |
| `matched_persona_id` | ID совпадающей персоны |
| `matched_persona_name` | Название совпадающей персоны |
| `signal_source_bucket` | Группа источников сигналов |
| `standard_event_content_type` | Тип контента стандартного события |

## SKAdNetwork метрики

### SKAdNetwork 4.0+ поля
- `coarse_conversion_value` - Грубое значение конверсии (low, medium, high)
- `fidelity_type` - Тип точности (0 для view-through, 1 для кликовых)
- `postback_sequence_index` - Порядок постбеков (0, 1, 2)
- `redownload` - Флаг повторной загрузки приложения
- `skan_version` - Версия SKAdNetwork
- `hsid` - Hierarchical Source Identifier
- `skan_conversion_id` - ID конверсии SKAdNetwork

## Дополнительные специфические метрики

### Метрики для сегментации пользователей

| Поле | Тип | Описание |
|------|-----|----------|
| `user_segment_key` | string | Ключ сегмента пользователя |
| `matched_persona_id` | string | ID совпадающей персоны |
| `matched_persona_name` | string | Название совпадающей персоны |
| `activity_recency` | string | Недавность активности пользователя |

### Источники сигналов и конверсии

| Поле | Тип | Описание |
|------|-----|----------|
| `signal_source_bucket` | string | Группа источников сигналов |
| `standard_event_content_type` | string | Тип контента стандартного события |
| `conversion_destination` | string | Место назначения конверсии |

### Дополнительные географические метрики

| Поле | Тип | Описание |
|------|-----|----------|
| `comscore_market` | string | Рынок ComScore для измерения аудитории |

### Автоматизация и настройки

| Поле | Тип | Описание |
|------|-----|----------|
| `is_auto_advance` | string | Флаг автоматического продвижения |
| `frequency_value` | string | Значение частоты (используется только с reach) |

### Полное название всех стандартных событий пикселя

#### События электронной коммерции
- `offsite_conversion.fb_pixel_purchase` - Покупка
- `offsite_conversion.fb_pixel_add_to_cart` - Добавление в корзину
- `offsite_conversion.fb_pixel_add_to_wishlist` - Добавление в список желаний
- `offsite_conversion.fb_pixel_initiate_checkout` - Начало оформления заказа
- `offsite_conversion.fb_pixel_add_payment_info` - Добавление платежной информации

#### События контента и поиска
- `offsite_conversion.fb_pixel_view_content` - Просмотр контента
- `offsite_conversion.fb_pixel_search` - Поиск на сайте

#### События регистрации и лидов
- `offsite_conversion.fb_pixel_complete_registration` - Завершение регистрации
- `offsite_conversion.fb_pixel_lead` - Генерация лида
- `offsite_conversion.fb_pixel_submit_application` - Отправка заявки

#### События подписок и услуг
- `offsite_conversion.fb_pixel_subscribe` - Подписка
- `offsite_conversion.fb_pixel_start_trial` - Начало пробного периода
- `offsite_conversion.fb_pixel_schedule` - Планирование/запись на прием

#### Другие события
- `offsite_conversion.fb_pixel_contact` - Обращение/контакт
- `offsite_conversion.fb_pixel_customize_product` - Настройка продукта
- `offsite_conversion.fb_pixel_donate` - Пожертвование
- `offsite_conversion.fb_pixel_find_location` - Поиск местоположения

### Полный список мобильных событий приложений

#### События активации и регистрации
- `app_custom_event.fb_mobile_activate_app` - Запуски мобильного приложения
- `app_custom_event.fb_mobile_complete_registration` - Регистрации в мобильном приложении

#### События покупок и платежей
- `app_custom_event.fb_mobile_purchase` - Покупки в мобильном приложении
- `app_custom_event.fb_mobile_add_payment_info` - Добавление платежных данных
- `app_custom_event.fb_mobile_initiated_checkout` - Оформления заказа

#### События каталога и контента
- `app_custom_event.fb_mobile_add_to_cart` - Добавления в корзину
- `app_custom_event.fb_mobile_add_to_wishlist` - Добавления в список желаний
- `app_custom_event.fb_mobile_content_view` - Просмотры контента

#### События достижений и игр
- `app_custom_event.fb_mobile_achievement_unlocked` - Разблокировка достижений
- `app_custom_event.fb_mobile_level_achieved` - Достижения уровней

#### Другие события приложений
- `app_custom_event.other` - Другие пользовательские события
- `app_custom_event.{custom_name}` - Пользовательские события с произвольными названиями

## Ограничения и особенности

### Недоступные поля при разбивках
- `app_store_clicks`
- `newsfeed_avg_position`
- `newsfeed_clicks`
- `relevance_score`
- `newsfeed_impressions`

### Ограничения для метрик действий вне Meta
Некоторые разбивки не поддерживаются для действий вне Facebook (например, установки приложений, покупки на сайтах).

### Приблизительные метрики
Многие метрики помечены как приблизительные или в разработке. Используйте их для общего направления анализа.

### Окна атрибуции
- По умолчанию используется `7d_click`
- Можно указать несколько окон атрибуции для сравнения
- Уникальные метрики занимают больше времени для вычисления

### Важные ограничения API

#### Ограничения по reach с разбивками
С 10 июня 2025 года параметр `reach` больше не возвращается для запросов с:
- Разбивками (breakdowns)
- `start_date` более 13 месяцев назад

Для получения reach с разбивками старше 13 месяцев используйте асинхронные задания (до 20 запросов в день на аккаунт).

#### Ограничения скорости вызовов
- Контролируется заголовком `x-fb-ads-insights-throttle`
- Применяется на уровне приложения и рекламного аккаунта
- При превышении: `error_code = 4, CodedException`

#### Ограничения объема данных
- По количеству строк в ответе
- По количеству точек данных для расчета
- При превышении: `error_code = 100, CodeException (error subcode: 1487534)`

#### Уровни доступа
- `development_access` - ограниченный доступ
- `standard_access` - расширенные лимиты
- Влияет на частоту запросов и доступные функции

### Рекомендации по использованию

#### Для избежания ограничений
- Используйте `date_preset` вместо custom диапазонов
- Ограничивайте количество запрашиваемых метрик
- Применяйте фильтрацию `filtering` для объектов с данными
- Используйте пакетные запросы для множественных вызовов
- Распределяйте запросы во времени

#### Для больших объемов данных
- Используйте асинхронные задания (POST запросы)
- Разбивайте запросы на более мелкие диапазоны дат
- Избегайте детальных разбивок на уровне аккаунта
- Используйте параметр `level` для получения нужной детализации

#### Обновление данных
- Статистика обновляется каждые 15 минут
- Данные не изменяются после 28 дней публикации
- Учитывайте часовой пояс рекламного аккаунта для ежедневных запросов

## Примеры использования

### Базовый запрос статистики
```bash
curl -G \
  -d "fields=impressions,clicks,spend,cpm,cpc,ctr" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ID/insights"
```

### Запрос с разбивкой по действиям
```bash
curl -G \
  -d "fields=actions,action_values" \
  -d "action_breakdowns=action_type" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ID/insights"
```

### Запрос видео метрик
```bash
curl -G \
  -d "fields=video_p25_watched_actions,video_p50_watched_actions,video_p75_watched_actions,video_p100_watched_actions" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ID/insights"
```

### Запрос с разбивкой по возрасту и полу
```bash
curl -G \
  -d "fields=impressions,clicks,spend" \
  -d "breakdowns=age,gender" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ID/insights"
```

### Запрос продуктовых метрик
```bash
curl -G \
  -d "fields=converted_product_quantity,converted_product_value" \
  -d "breakdowns=product_id" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ID/insights"
```

### Запрос с окнами атрибуции
```bash
curl -G \
  -d "fields=actions" \
  -d "action_attribution_windows=['1d_click','7d_click','1d_view']" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ID/insights"
```

### Асинхронный запрос для больших данных
```bash
curl -X POST \
  -d "fields=impressions,clicks,spend,actions" \
  -d "level=ad" \
  -d "date_preset=last_30_days" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ACCOUNT_ID/insights"
```

### Проверка статуса асинхронного задания
```bash
curl -G \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/REPORT_RUN_ID"
```

### Получение результатов асинхронного задания
```bash
curl -G \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/REPORT_RUN_ID/insights"
```

### Пакетный запрос
```bash
curl \
  -F 'access_token=ACCESS_TOKEN' \
  -F 'batch=[
    {
      "method": "GET",
      "relative_url": "v23.0/CAMPAIGN_ID_1/insights?fields=impressions,spend"
    },
    {
      "method": "GET", 
      "relative_url": "v23.0/CAMPAIGN_ID_2/insights?fields=impressions,spend"
    }
  ]' \
  "https://graph.facebook.com"
```

### Запрос с фильтрацией
```bash
curl -G \
  -d "fields=impressions,clicks,spend" \
  -d "level=ad" \
  -d "filtering=[{\"field\":\"ad.impressions\",\"operator\":\"GREATER_THAN\",\"value\":0}]" \
  -d "access_token=ACCESS_TOKEN" \
  "https://graph.facebook.com/v23.0/AD_ACCOUNT_ID/insights"
```

## Статусы асинхронных заданий

| Статус | Описание |
|--------|----------|
| `Job Started` | Задание начато |
| `Job Running` | Задание выполняется |
| `Job Completed` | Задание завершено |
| `Job Failed` | Задание завершилось с ошибкой |
| `Job Skipped` | Задание пропущено |

## Экспорт отчетов

Facebook предоставляет возможность экспорта отчетов в различных форматах:

```bash
curl -G \
  -d "report_run_id=REPORT_RUN_ID" \
  -d "name=myreport" \
  -d "format=xls" \
  "https://www.facebook.com/ads/ads_insights/export_report/"
```

### Форматы экспорта
- `xls` - Excel файл
- `csv` - CSV файл

---

*Документация основана на Facebook Marketing API v23.0. Некоторые метрики могут быть приблизительными или находиться в разработке.*

## Статус документа

**Последняя проверка:** Январь 2025  
**Версия API:** v23.0  
**Статус полноты:** ✅ Максимально полный справочник  

### Охват документа:
- ✅ Все основные метрики (базовые, уникальные, видео, конверсионные)
- ✅ Продуктовые и креативные метрики  
- ✅ SKAdNetwork и аукционные метрики
- ✅ Полный перечень action types (включая Messenger, Instagram Shopping)
- ✅ Все разбивки (breakdowns) и action_breakdowns
- ✅ Ограничения API, рекомендации, best practices
- ✅ Практические примеры и статусы асинхронных заданий
- ✅ Экспорт отчетов и форматы данных
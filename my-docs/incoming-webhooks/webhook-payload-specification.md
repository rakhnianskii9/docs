# 📥 WhatsApp Webhook Payload Specification

> **📎 Связанные документы:**
> - [Структура данных подписчика](../subscriber/exact-subscriber-data-structure-v2.md) - как данные сохраняются в БД
> - Этот документ описывает RAW формат входящих webhook от провайдеров

## � Навигация
- **Этот документ**: RAW webhook payloads от Cloud API/Gupshup
- **[Subscriber Structure](../subscriber/exact-subscriber-data-structure-v2.md)**: Как данные сохраняются в БД
- **Поток данных**: Webhook → Parser → Subscriber DB

---

## 🔄 Единая модель первого входящего webhook

Контракт данных, который мы получаем из первого входящего сообщения от любого провайдера:

{
  "id": "uuid",
  "tenantId": "string",

  "waPhone": "+E.164",                    // нормализованный для витрины/поиска
  "waId": "string",                       // сырой идентификатор провайдера (Cloud API/Gupshup)
  "profileName": "string|null",

  "rawLastEventType": "message",          // тип входящего события провайдера
  "rawVersion": "v3",
  "receivedAt": 1736535600000,            // UNIX ms

  "message": {
    "type": "text|image|video|audio|document|sticker|location|contacts|interactive|reaction|order|system|unsupported",

    "text": "string|null",
    "mediaId": "string|null",             // Cloud API media.id (для скачивания через /media)
    "mimeType": "string|null",
    "caption": "string|null",

    "contacts": [
      {
        "phones": [{"wa_id": "string", "type": "cell|home|work|null"}],
        "name": {"formatted_name": "string|null", "first_name": "string|null", "last_name": "string|null"},
        "emails": [{"email": "string", "type": "work|home|null"}]
      }
    ],

    "location": { "lat": 0, "lng": 0, "name": "string|null", "address": "string|null" },

    "interactive": {
      "type": "button_reply|list_reply|product|product_list|nfm_reply|null",
      "id": "string|null",
      "title": "string|null",
      "payload": "string|null",
      "button_reply": { "id": "string|null", "title": "string|null" },
      "list_reply": { "id": "string|null", "title": "string|null", "description": "string|null" },
      "nfm_reply": { "response_json": "string|null" }   // WhatsApp Flows
    },

    "reaction": { "messageId": "WAMID|null", "emoji": "string|null" },

    "forwarded": false,
    "frequentlyForwarded": false,
    "replyToId": "WAMID|null"
  },

  "referral": {
    "ctwaClid": "string|null",
    "sourceUrl": "string|null",
    "sourceType": "ad|post|page|website|null",
    "sourceId": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "mediaType": "image|video|carousel|null",
    "imageUrl": "string|null",
    "videoUrl": "string|null",
    "thumbnailUrl": "string|null"
  },

  "utm": {
    "source": "string|null",
    "medium": "string|null",
    "campaign": "string|null",
    "content": "string|null",
    "term": "string|null"
  }
}

Замечания (без сокращений):
— Полный перечень message.type включает contacts, interactive, reaction, order, system, unsupported; интерактивы: button_reply/list_reply/product/product_list/nfm_reply.
— forwarded/frequentlyForwarded, replyToId, mediaId, location — доступны в первом входящем.
— mediaId скачивается через Media API по токену; постоянный URL не присылается в вебхуке.
— referral.* и ctwaClid приходят в первом сообщении, если диалог начат из объявления (CTWA). utm.* парсим из referral.sourceUrl.

⸻

2) Что приходит позже (не «моментально») ⏳

{
  "conversation": {
    "id": "string",                        // statuses[].conversation.id
    "expirationTimestamp": 1736539200000,  // когда истечёт окно
    "pricing": {
      "category": "utility|authentication|marketing|service|marketing_lite",
      "billable": true,
      "pricingModel": "CBP|per_template|PMP"
    }
  }
}

Эти поля приходят в статусных вебхуках (delivery/read/pricing), а не в первом inbound.

⸻

3) Эталоны пэйлоэдов провайдеров 🧩

3.1 Meta Cloud API — входящее текстовое (фрагмент)

{
  "entry": [{
    "changes": [{
      "value": {
        "contacts": [{"wa_id": "15551234567", "profile": {"name": "John Doe"}}],
        "messages": [{
          "from": "15551234567",
          "id": "wamid.HBgMNTU1NTEyMzQ1Njc...",
          "timestamp": "1736535600",
          "type": "text",
          "text": {"body": "Hello"},
          "context": {"id": "wamid.HBgM..."}          // если reply
        }],
        "metadata": {"phone_number_id": "1234567890"}
      },
      "field": "messages"
    }]
  }]
}

Маппинг: contacts[0].wa_id → waId, contacts[0].profile.name → profileName, messages[0].type → message.type, messages[0].text.body → message.text, messages[0].id → replyToId/корреляция, context.id → replyToId.

3.2 Meta Cloud API — входящее с referral/CTWA (фрагмент)

{
  "entry": [{
    "changes": [{
      "value": {
        "messages": [{
          "type": "text",
          "text": {"body": "Hi"},
          "referral": {
            "ctwa_clid": "fb.1.1736535600.IwAR123...",
            "source_url": "https://m.facebook.com/...utm_source=facebook&utm_medium=cpc",
            "source_type": "ad",
            "source_id": "120210000000000000",
            "headline": "Скидка 50%",
            "body": "Напиши нам в WhatsApp",
            "media_type": "image",
            "image_url": "https://scontent.xx.fbcdn.net/..."
          }
        }]
      }
    }]
  }]
}

Маппинг: referral.* → referral, referral.source_url → utm.* (парсинг).

3.3 Meta Cloud API — статусное событие (conversation/pricing)

{
  "entry": [{
    "changes": [{
      "value": {
        "statuses": [{
          "id": "wamid.HBgM...",
          "status": "delivered",
          "timestamp": "1736535800",
          "conversation": {
            "id": "gBEGkYiEB1VXAglK1ZEqA1YKPrU",
            "expiration_timestamp": "1736539200"
          },
          "pricing": {
            "billable": true,
            "category": "utility",
            "pricing_model": "CBP"
          }
        }]
      }
    }]
  }]
}

Маппинг: statuses[0].conversation.* → conversation, statuses[0].pricing.* → conversation.pricing.*.

3.4 Gupshup — входящее сообщение (эталон)

{
  "app": "your-app",
  "version": "v3",
  "type": "message",
  "timestamp": 1736535600000,
  "payload": {
    "id": "wamid.HBgM...",
    "source": "15551234567",               // E.164 (может быть без '+')
    "sender": {"phone": "15551234567", "name": "John Doe", "country_code": "1"},
    "type": "text",
    "payload": { "text": "Hello" },
    "context": {"id": "wamid.HBgM..."}     // если это reply
  }
}

Маппинг: payload.sender.phone → waPhone, payload.id → WAMID, payload.type → message.type, payload.payload.text → message.text, app/version/timestamp → метаданные.

Медиа: Cloud API присылает messages[0].image.id|video.id|audio.id|document.id → сохраняем как message.mediaId и скачиваем через Media API.

⸻

4) Маппинг (короткая таблица)

Unified	Cloud API	Gupshup
waId	contacts[0].wa_id	payload.id (WAMID), sender.phone для номера
waPhone	нормализация из contacts[0].wa_id	payload.sender.phone
profileName	contacts[0].profile.name	payload.sender.name
message.type	messages[0].type	payload.type
message.text	messages[0].text.body	payload.payload.text
message.mediaId	`messages[0].image	video
replyToId	messages[0].context.id	payload.context.id
referral.*	messages[0].referral.*	—
utm.*	parse(referral.source_url)	—
conversation.*	statuses[0].conversation.*	—


⸻

5) Полные разметки по ВСЕМ типам messages[] (Cloud API)

В каждом типе ниже — встроенные и полностью раскрытые блоки referral и context.

5.1 text

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "text",
  "text": { "body": "string" },

  "referral": {
    "ctwa_clid": "string|null",
    "source_url": "string|null",
    "source_type": "ad|post|page|website|null",
    "source_id": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "media_type": "image|video|carousel|null",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null"
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.2 image

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "image",
  "image": {
    "id": "MEDIA_ID",
    "mime_type": "image/jpeg",
    "sha256": "hex",
    "caption": "string|null"
  },

  "referral": {
    "ctwa_clid": "string|null",
    "source_url": "string|null",
    "source_type": "ad|post|page|website|null",
    "source_id": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "media_type": "image|video|carousel|null",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null"
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.3 video

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "video",
  "video": {
    "id": "MEDIA_ID",
    "mime_type": "video/mp4",
    "sha256": "hex",
    "caption": "string|null"
  },

  "referral": {
    "ctwa_clid": "string|null",
    "source_url": "string|null",
    "source_type": "ad|post|page|website|null",
    "source_id": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "media_type": "image|video|carousel|null",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null"
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.4 audio (включая voice note)

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "audio",
  "audio": {
    "id": "MEDIA_ID",
    "mime_type": "audio/ogg",
    "sha256": "hex",
    "voice": true
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.5 document

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "document",
  "document": {
    "id": "MEDIA_ID",
    "mime_type": "application/pdf",
    "sha256": "hex",
    "filename": "string",
    "caption": "string|null"
  },

  "referral": {
    "ctwa_clid": "string|null",
    "source_url": "string|null",
    "source_type": "ad|post|page|website|null",
    "source_id": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "media_type": "image|video|carousel|null",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null"
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.6 sticker (РАЗВЁРНУТО, как просил)

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "sticker",
  "sticker": {
    "id": "MEDIA_ID",
    "mime_type": "image/webp",
    "sha256": "hex",
    "animated": true
  },

  "referral": {
    "ctwa_clid": "string|null",
    "source_url": "string|null",
    "source_type": "ad|post|page|website|null",
    "source_id": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "media_type": "image|video|carousel|null",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null"
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.7 location

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "location",
  "location": {
    "latitude": 0.0,
    "longitude": 0.0,
    "name": "string|null",
    "address": "string|null"
  },

  "referral": {
    "ctwa_clid": "string|null",
    "source_url": "string|null",
    "source_type": "ad|post|page|website|null",
    "source_id": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "media_type": "image|video|carousel|null",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null"
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.8 contacts (vCard — полный состав)

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "contacts",
  "contacts": [{
    "name": {
      "formatted_name": "string",
      "first_name": "string|null",
      "last_name": "string|null",
      "middle_name": "string|null",
      "suffix": "string|null",
      "prefix": "string|null"
    },
    "org": { "company": "string|null", "department": "string|null", "title": "string|null" },
    "phones": [{ "phone": "string", "type": "CELL|HOME|WORK|null", "wa_id": "string|null" }],
    "emails": [{ "email": "string", "type": "WORK|HOME|null" }],
    "addresses": [{
      "street": "string|null",
      "city": "string|null",
      "state": "string|null",
      "zip": "string|null",
      "country": "string|null",
      "country_code": "string|null",
      "type": "WORK|HOME|null"
    }],
    "urls": [{ "url": "string", "type": "WORK|HOME|null" }],
    "birthday": "YYYY-MM-DD|null"
  }],

  "referral": {
    "ctwa_clid": "string|null",
    "source_url": "string|null",
    "source_type": "ad|post|page|website|null",
    "source_id": "string|null",
    "headline": "string|null",
    "body": "string|null",
    "media_type": "image|video|carousel|null",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null"
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  },

  "forwarded": false,
  "frequently_forwarded": false
}

5.9 interactive (все варианты)

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "interactive",
  "interactive": {
    "type": "button_reply|list_reply|nfm_reply",
    "button_reply": { "id": "string|null", "title": "string|null" },
    "list_reply": { "id": "string|null", "title": "string|null", "description": "string|null" },
    "nfm_reply": { "response_json": "string|null" }
  },

  "context": {
    "id": "wamid.Оригинала|null",
    "from": "WA_ID_автора_оригинала|null",
    "referred_product": {
      "catalog_id": "string",
      "product_retailer_id": "string"
    }
  }
}

5.10 reaction

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "reaction",
  "reaction": { "message_id": "wamid...", "emoji": "👍" }
}

5.11 order (после product / product_list)

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "order",
  "order": {
    "catalog_id": "string",
    "text": "string|null",
    "product_items": [{
      "product_retailer_id": "string",
      "quantity": 1,
      "item_price": "12.34",
      "currency": "USD"
    }]
  }
}

5.12 system

{
  "from": "string",
  "id": "wamid...",
  "timestamp": "string",
  "type": "system",
  "system": {
    "body": "string",
    "type": "user_changed_number|security_notice|...",
    "wa_id": "string|null",
    "new_wa_id": "string|null"
  }
}

5.13 unsupported

{ "from": "string", "id": "wamid...", "timestamp": "string", "type": "unsupported" }


⸻

6) CTWA/Referral — полный разрез (в первом inbound)

{
  "ctwa_clid": "string",
  "source_url": "https://...utm_source=...",
  "source_type": "ad|post|page|website",
  "source_id": "string",
  "headline": "string",
  "body": "string",
  "media_type": "image|video|carousel",
  "image_url": "string|null",
  "video_url": "string|null",
  "thumbnail_url": "string|null"
}

UTM: парсить сразу из source_url (utm_source, utm_medium, utm_campaign, utm_content, utm_term).
Динамические параметры в целевом URL объявления: {{ad.id}}, {{adset.id}}, {{campaign.id}}, {{placement}}, {{site_source_name}} — чтобы сразу получить ad_id/adset_id/campaign_id в source_url.
CAPI: отправлять событие с ctwa_clid в custom_data (склейка клика с WA-сессией).

⸻

7) Сводная матрица наличия полей по типам

Тип	Контент-узел	Reply (context.id)	Referral	Media ID
text	text.body	✅	✅	—
image	image.{id,mime_type,sha256,caption}	✅	✅	✅
video	video.{id,mime_type,sha256,caption}	✅	✅	✅
audio/voice	audio.{id,mime_type,sha256,voice}	✅	—	✅
document	document.{id,mime_type,sha256,filename,caption}	✅	✅	✅
sticker	sticker.{id,mime_type,sha256,animated}	✅	✅	✅
location	location.{latitude,longitude,name,address}	✅	✅	—
contacts	contacts[] {name,org,phones,emails,addresses,urls}	—	✅	—
interactive	`interactive.{button_reply	list_reply	nfm_reply}`	✅
reaction	reaction.{message_id,emoji}	—	—	—
order	order.{catalog_id,product_items[],text}	—	—	—
system	system.{body,type,(new_)wa_id}	—	—	—
unsupported	—	—	—	—


⸻

8) Gupshup (мы через него работаем) — полное сопоставление

8.1 Конверт/метаданные

Наша витрина	Gupshup
rawVersion	version (v2/v3)
rawLastEventType	type = message/message-event/user-event/billing-event
receivedAt	timestamp (UNIX ms)
waPhone	payload.source и/или payload.sender.phone
message.context.id	payload.context.id (WAMID оригинала)
message.context.gsId	payload.context.gsId (если есть)

8.2 Типы входящих у Gupshup (payload.type)

text | image | file | audio | video | contact | location | button_reply | list_reply | sticker | reaction | order | voice

	•	Интерактив у Gupshup разнесён: button_reply, list_reply.
	•	file = наш document.
	•	reaction, sticker, order поддерживаются.

8.3 Контентные узлы (mapping)

Cloud API	Gupshup
text.body	payload.payload.text
image.* / video.* / audio.* / document.*	payload.payload.* (+ их Media)
sticker.*	payload.payload.* (type=sticker)
contacts[]	type=contact, payload.contacts[]
interactive.button_reply	type=button_reply
interactive.list_reply	type=list_reply
reaction	type=reaction
order	type=order

Отличия/ограничения Gupshup:
	•	Флаги forwarded/frequently_forwarded в публичной inbound-доке не указаны.
	•	По location в примерах — гарантированы координаты; name/address могут отсутствовать.

8.4 Referral/CTWA у Gupshup (полный)

{
  "referral": {
    "source_url": "string",
    "source_type": "ad|post|page|website",
    "source_id": "string",
    "headline": "string",
    "body": "string",
    "media_type": "image|video|carousel",
    "image_url": "string|null",
    "video_url": "string|null",
    "thumbnail_url": "string|null",
    "ctwa_clid": "string"
  }
}

UTM-метки берём парсингом из referral.source_url.

8.5 Статусы/биллинг/окна у Gupshup
	•	message-event: enqueued|failed|sent|delivered|read; может включать conversation{ id, expiresAt, type } и pricing{ policy: CBP|PMP, category }.
	•	billing-event: для PMP/marketing_lite и т.д.; после перехода на PMP отдельный conversation_id может отсутствовать.
	•	«24-часовое окно» отражается статусами/биллингом.

8.6 WA Flows через Gupshup
	•	Ответы Flows возвращаются как «nfm reply» на ваш вебхук (динамические флоу поддерживаются).
	•	В нашей витрине это — interactive.nfm_reply.response_json (JSON-строка).

⸻

9) Минимальные JSON-примеры

9.1 Органика

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "tenantId": "tenant_12345",
  "waPhone": "+15551234567",
  "waId": "15551234567",
  "profileName": "John Doe",
  "rawLastEventType": "message",
  "rawVersion": "v3",
  "receivedAt": 1736535600000,

  "message": {
    "type": "text",
    "text": "Hello",
    "mediaId": null,
    "mimeType": null,
    "caption": null,
    "contacts": null,
    "location": null,
    "interactive": null,
    "reaction": null,
    "forwarded": false,
    "frequentlyForwarded": false,
    "replyToId": null
  },

  "referral": {
    "ctwaClid": null, "sourceUrl": null, "sourceType": null, "sourceId": null,
    "headline": null, "body": null, "mediaType": null, "imageUrl": null,
    "videoUrl": null, "thumbnailUrl": null
  },

  "utm": { "source": null, "medium": null, "campaign": null, "content": null, "term": null }
}

9.2 Пришёл через рекламу (CTWA)

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "tenantId": "tenant_12345",
  "waPhone": "+15551234567",
  "waId": "15551234567",
  "profileName": "Maria Garcia",
  "rawLastEventType": "message",
  "rawVersion": "v3",
  "receivedAt": 1736535600000,

  "message": {
    "type": "text",
    "text": "Hi",
    "mediaId": null,
    "mimeType": null,
    "caption": null,
    "contacts": null,
    "location": null,
    "interactive": null,
    "reaction": null,
    "forwarded": false,
    "frequentlyForwarded": false,
    "replyToId": null
  },

  "referral": {
    "ctwaClid": "fb.1.1736535600.IwAR123abcDEF456ghiJKL789mno",
    "sourceUrl": "https://m.facebook.com/...utm_source=facebook&utm_medium=cpc",
    "sourceType": "ad",
    "sourceId": "120210000000000000",
    "headline": "Скидка 50% только сегодня",
    "body": "Напиши нам в WhatsApp",
    "mediaType": "image",
    "imageUrl": "https://scontent.xx.fbcdn.net/...",
    "videoUrl": null,
    "thumbnailUrl": null
  },

  "utm": {
    "source": "facebook",
    "medium": "cpc",
    "campaign": "summer_sale_2025",
    "content": "video_testimonial",
    "term": "discount_whatsapp"
  }
}


⸻

## 🔗 Маппинг полей из webhook в структуру подписчика {#field-mapping}

### Основные поля

| Поле подписчика | Cloud API | Gupshup | Примечание |
|-----------------|-----------|---------|------------|
| `waPhone` | `contacts[0].wa_id` | `payload.source` | Нормализация в E.164 |
| `waId` | `contacts[0].wa_id` | `payload.source` | Сырой формат провайдера |
| `profileName` | `contacts[0].profile.name` | `payload.sender.name` | Может быть null |
| `conversationId` | `conversation.id` | `payload.conversation.id` | Для биллинга |
| `conversationExpiresAt` | `conversation.expiration_timestamp` | `payload.conversation.expiration_timestamp` | Unix ms |
| `rawLastEventType` | `"message"` | `payload.type` | Тип webhook |
| `rawVersion` | `"v3"` | `payload.version` | Версия API |

### Referral поля {#referral}

| Поле подписчика | Cloud API | Gupshup | Примечание |
|-----------------|-----------|---------|------------|
| `ctwaClid` | `referral.ctwa_clid` | `payload.referral.ctwa_clid` | Facebook Click ID |
| `referral.sourceUrl` | `referral.source_url` | `payload.referral.source_url` | Полный URL |
| `referral.sourceType` | `referral.source_type` | `payload.referral.source_type` | ad/post/page |
| `referral.sourceId` | `referral.source_id` | `payload.referral.source_id` | Facebook ID |
| `referral.headline` | `referral.headline` | `payload.referral.headline` | Заголовок |
| `referral.body` | `referral.body` | `payload.referral.body` | Описание |
| `referral.mediaType` | `referral.media_type` | `payload.referral.media_type` | image/video |
| `referral.imageUrl` | `referral.image_url` | `payload.referral.image_url` | URL превью |

---

## 🚀 Жёсткие правила обработки
	•	Хранить оба: waPhone (+E.164) и waId (сырой).
	•	UTM парсить немедленно из referral.source_url.
	•	В объявлении проставлять {{campaign.id}}, {{adset.id}}, {{ad.id}}, {{placement}}, {{site_source_name}}.
	•	ctwa_clid писать в лог и слать в CAPI.
	•	Медиа забирать только по MEDIA_ID через Media API (а не по ссылке из referral).
	•	Flows: парсить interactive.nfm_reply.response_json.
	•	Reaction: учесть лимит (реакции на старые сообщения могут не приходить вебхуком).
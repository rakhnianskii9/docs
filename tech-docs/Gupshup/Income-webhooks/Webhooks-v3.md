# Gupshup Webhooks Documentation

## Overview

This document provides comprehensive information about Gupshup webhooks, covering all event types that can be received on your callback URL.

## Event Types

### 1. Message Events

Message events provide status updates for messages sent via the Gupshup API to WhatsApp users.

#### Common Message Event Structure

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "id": "59f8db90-c37e-4408-90ab-cc54ef8246ad",
        "gsId": "ee4a68a0-1203-4c85-8dc3-49d0b3226a35",
        "type": "enqueued|failed|sent|delivered|read|deleted",
        "destination": "91XX985XX10X",
        "payload": "## Varies by event type"
    }
}
```

#### Message Event Types

##### 1.1 Enqueued

Received when a message is successfully sent to the WhatsApp Business API client.

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "id": "59f8db90-c37e-4408-90ab-cc54ef8246ad",
        "type": "enqueued",
        "destination": "91XX985XX10X",
        "payload": {
            "whatsappMessageId": "gBEGkYaYVSEEAgkD7bRi9syGnBk",
            "type": "session"
        }
    }
}
```

##### 1.2 Failed (Sync)

Received when message sending fails immediately.

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "id": "ee4a68a0-1203-4c85-8dc3-49d0b3226a35",
        "type": "failed",
        "destination": "91XX985XX10X",
        "payload": {
            "code": 1002,
            "reason": "Number Does Not Exist On WhatsApp"
        }
    }
}
```

##### 1.3 Failed (Async)

Received when messages fail after being processed by WhatsApp.

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "gsId": "72f61f22-5aa4-4615-a970-943edf6da01c",
        "type": "failed",
        "destination": "91XX985XX10X",
        "payload": {
            "code": 1002,
            "reason": "Number Does Not Exists On WhatsApp"
        }
    }
}
```

##### 1.4 Sent

Received when the message is successfully sent to the end-user.

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "id": "wamid.HBgMOTE5MTYzODA1ODczFQIAERgSMDQ4Rj...",
        "gsId": "7daeb742-f2dc-4c09-90e0-d879d20f7b98",
        "type": "sent",
        "destination": "91XX985XX10X",
        "ts": 1710930461,
        "payload": {
            "conversation": {
                "id": "3dabdf65abaf1cbbf85217d232cce91f",
                "expiresAt": 1710936120,
                "type": "Marketing",
                "category": "marketing_lite"
            },
            "pricing": {
                "policy": "CBP",
                "category": "Marketing"
            }
        }
    }
}
```

##### 1.5 Delivered

Received when the message is delivered to the user's device.

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "id": "wamid.HBgMOTE5MTYzODA1ODczFQIAERgSMDQ4Rj...",
        "gsId": "7daeb742-f2dc-4c09-90e0-d879d20f7b98",
        "type": "delivered",
        "destination": "91XX985XX10X",
        "ts": 1710930462,
        "payload": {
            "conversation": {
                "id": "3dabdf65abaf1cbbf85217d232cce91f"
            }
        }
    }
}
```

##### 1.6 Read

Received when the message is read by the user (only for users with read receipts enabled).

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "id": "wamid.HBgMOTE5MTYzODA1ODczFQIAERgSMUUzNz...",
        "gsId": "7b8f0910-db98-4cce-be5f-e6e1e847b4bd",
        "type": "read",
        "destination": "91XX985XX10X",
        "ts": 1720779817
    }
}
```

##### 1.7 Mismatch

Triggered when the destination number doesn't match the WhatsApp ID.

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "type": "mismatch",
        "destination": "553587654321",
        "payload": {
            "wa_id": "5593587654321",
            "phone": "553587654321"
        }
    }
}
```

##### 1.8 Referral (CTX)

Received when a user contacts you through an ad or post.

```json
{
    "app": "DemoAPI",
    "timestamp": 1580546677791,
    "version": 2,
    "type": "message-event",
    "payload": {
        "type": "referral",
        "destination": "91XX985XX10X",
        "payload": {
            "source_url": "https://www.meta.com/ad/12345",
            "source_type": "ad",
            "source_id": "1234567890",
            "headline": "Get 20% off on your first order!",
            "body": "Shop now and save on your favorite items.",
            "media_type": "image",
            "image_url": "https://www.meta.com/ad/12345/image.jpg",
            "ctwa_clid": "abc123xyz456"
        }
    }
}
```

### 2. User Events

User events are triggered by user actions like opt-in/opt-out.

#### User Event Structure

```json
{
    "app": "DemoApp",
    "timestamp": 1580142086287,
    "version": 2,
    "type": "user-event",
    "payload": {
        "phone": "918x98xx21x4",
        "type": "sandbox-start|opted-in|opted-out"
    }
}
```

#### User Event Types

##### 2.1 Sandbox Start

Received when the app is in Sandbox mode and callback URL is set.

```json
{
    "app": "DemoApp",
    "timestamp": 1580142086287,
    "version": 2,
    "type": "user-event",
    "payload": {
        "phone": "callbackSetPhone",
        "type": "sandbox-start"
    }
}
```

##### 2.2 Opted In

Received when a user opts in to receive notifications.

```json
{
    "app": "DemoApp",
    "timestamp": 1580142086287,
    "version": 2,
    "type": "user-event",
    "payload": {
        "phone": "918x98xx21x4",
        "type": "opted-in"
    }
}
```

##### 2.3 Opted Out

Received when a user opts out from receiving notifications.

```json
{
    "app": "DemoApp",
    "timestamp": 1580142086287,
    "version": 2,
    "type": "user-event",
    "payload": {
        "phone": "918x98xx21x4",
        "type": "opted-out"
    }
}
```

### 3. System Events

System events are generated when platform-level events occur.

#### 3.1 Template Events

Received when template status changes.

##### Template Status Update

```json
{
    "app": "jeet20",
    "timestamp": 1636986446609,
    "version": 2,
    "type": "template-event",
    "payload": {
        "type": "status-update",
        "id": "4dacef15-6c04-12db-b393-6190ac567eff",
        "status": "rejected|approved|paused|deleted|disabled|in_appeal",
        "elementName": "abcd",
        "languageCode": "en_US",
        "rejectedReason": "INVALID_FORMAT",
        "description": "Your WhatsApp message template has been paused for 3 hours until Apr 25 at 8:07 AM UTC because it had issues."
    }
}
```

##### Template Category Update (Alert)

```json
{
    "app": "jeet20",
    "timestamp": 1717249875000,
    "version": 2,
    "type": "template-event",
    "payload": {
        "type": "category-update",
        "id": "4dacef15-6c04-12db-b393-6190ac567eff",
        "elementName": "abcd",
        "languageCode": "en_US",
        "category": {
            "current": "MARKETING",
            "correct": "UTILITY"
        }
    }
}
```

##### Template Category Update (Actual)

```json
{
    "app": "jeet20",
    "timestamp": 1719815415000,
    "version": 2,
    "type": "template-event",
    "payload": {
        "type": "category-update",
        "id": "4dacef15-6c04-12db-b393-6190ac567eff",
        "elementName": "abcd",
        "languageCode": "en_US",
        "category": {
            "old": "MARKETING",
            "new": "UTILITY"
        }
    }
}
```

#### 3.2 Account Events

Received when account-level events occur.

##### Review Event

```json
{
    "app": "jeet20",
    "timestamp": 1636986446609,
    "version": 2,
    "type": "account-event",
    "payload": {
        "type": "review-event",
        "payload": {
            "status": "approved|rejected",
            "actionDate": "January 31,2021"
        }
    }
}
```

##### Account Status Event

```json
{
    "app": "jeet20",
    "timestamp": 1636986446609,
    "version": 2,
    "type": "account-event",
    "payload": {
        "type": "status-event",
        "payload": {
            "status": "ACCOUNT_VIOLATION|ACCOUNT_DISABLE|ACCOUNT_VERIFIED|ACCOUNT_RESTRICTED",
            "restrictions": [
                {
                    "type": "RESTRICTION_ADD_PHONE_NUMBER_ACTION",
                    "expiry": "2024-12-31T23:59:59Z"
                }
            ]
        }
    }
}
```

##### Go-Live Event

```json
{
    "app": "APP_NAME",
    "appId": "APP_ID",
    "phone": "PHONE_NUMBER",
    "timestamp": 1234567890,
    "version": 2,
    "type": "onboarding-event",
    "payload": {
        "type": "docker-status-event",
        "payload": {
            "status": "live",
            "waId": "WABA_ID",
            "namespace": "META-TEMPLATE-NAMESPACE"
        }
    }
}
```

##### Tier Event

```json
{
    "app": "jeet20",
    "timestamp": 1636986446609,
    "version": 2,
    "type": "account-event",
    "payload": {
        "type": "tier-event",
        "payload": {
            "event": "ONBOARDING|UPGRADE|DOWNGRADE|UNFLAGGED|FLAGGED",
            "currentTier": "TIER_250|TIER_1K|TIER_10K|TIER_100K|TIER_UNLIMITED",
            "newTier": "TIER_1K"
        }
    }
}
```

##### Capability Event

```json
{
    "app": "jeet20",
    "timestamp": 1636986446609,
    "version": 2,
    "type": "account-event",
    "payload": {
        "type": "capability-event",
        "payload": {
            "maxDailyConversationPerPhone": 1000,
            "maxPhoneNumbersPerBusiness": 5
        }
    }
}
```

### 4. V3 Events (WhatsApp Business Account Format)

V3 events follow the WhatsApp Business Account webhook format.

#### V3 Message Status Events

##### Enqueued - V3

```json
{
    "entry": [
        {
            "changes": [
                {
                    "field": "messages",
                    "value": {
                        "messaging_product": "whatsapp",
                        "statuses": [
                            {
                                "gs_id": "2baff204-c39a-4121-90b5-f53f26d522b3",
                                "id": "wamid.HBgMOTE5MTYzODA1ODczFQIAERgSNkZCMThGMEEwQzJDOUFGQjFBAA==",
                                "recipient_id": "9191****873",
                                "status": "enqueued",
                                "timestamp": 1710941393420
                            }
                        ]
                    }
                }
            ]
        }
    ],
    "gs_app_id": "82ed52f4-30c0-4f12-81b4-e7ad07bd41de",
    "object": "whatsapp_business_account"
}
```

##### Sent - V3

```json
{
    "entry": [
        {
            "changes": [
                {
                    "field": "messages",
                    "value": {
                        "messaging_product": "whatsapp",
                        "metadata": {
                            "display_phone_number": "9****8",
                            "phone_number_id": "158072934066266"
                        },
                        "statuses": [
                            {
                                "conversation": {
                                    "expiration_timestamp": "1710936120",
                                    "id": "3dabdf65abaf1cbbf85217d232cce91f"
                                },
                                "gs_id": "7daeb742-f2dc-4c09-90e0-d879d20f7b98",
                                "id": "wamid.HBgMOTE5MTYzODA1ODczFQIAERgSMDQ4Rj...",
                                "recipient_id": "91****73",
                                "status": "sent",
                                "timestamp": "1710930461"
                            }
                        ]
                    }
                }
            ],
            "id": "112535025189792"
        }
    ],
    "gs_app_id": "82ed52f4-30c0-4f12-81b4-e7ad07bd41de",
    "object": "whatsapp_business_account"
}
```

##### Delivered - V3

```json
{
    "entry": [
        {
            "changes": [
                {
                    "field": "messages",
                    "value": {
                        "messaging_product": "whatsapp",
                        "metadata": {
                            "display_phone_number": "9*****58",
                            "phone_number_id": "158072934066266"
                        },
                        "statuses": [
                            {
                                "conversation": {
                                    "id": "3dabdf65abaf1cbbf85217d232cce91f"
                                },
                                "gs_id": "7daeb742-f2dc-4c09-90e0-d879d20f7b98",
                                "id": "wamid.HBgMOTE5MTYzODA1ODczFQIAERgSMDQ4Rj...",
                                "recipient_id": "91****873",
                                "status": "delivered",
                                "timestamp": "1710930462"
                            }
                        ]
                    }
                }
            ],
            "id": "112535025189792"
        }
    ],
    "gs_app_id": "82ed52f4-30c0-4f12-81b4-e7ad07bd41de",
    "object": "whatsapp_business_account"
}
```

##### Read - V3

```json
{
    "entry": [
        {
            "changes": [
                {
                    "field": "messages",
                    "value": {
                        "messaging_product": "whatsapp",
                        "metadata": {
                            "display_phone_number": "9*******8",
                            "phone_number_id": "158072934066266"
                        },
                        "statuses": [
                            {
                                "gs_id": "7b8f0910-db98-4cce-be5f-e6e1e847b4bd",
                                "id": "wamid.HBgMOTE5MTYzODA1ODczFQIAERgSMUUzNz...",
                                "recipient_id": "91*******73",
                                "status": "read",
                                "timestamp": "1720779817"
                            }
                        ]
                    }
                }
            ],
            "id": "112535025189792"
        }
    ],
    "gs_app_id": "82ed52f4-30c0-4f12-81b4-e7ad07bd41de",
    "object": "whatsapp_business_account"
}
```

##### Failed - V3

```json
{
    "entry": [
        {
            "changes": [
                {
                    "field": "messages",
                    "value": {
                        "messaging_product": "whatsapp",
                        "metadata": {
                            "display_phone_number": "91*******8",
                            "phone_number_id": "158072934066266"
                        },
                        "statuses": [
                            {
                                "errors": [
                                    {
                                        "code": 131047,
                                        "href": "https://developers.facebook.com/docs/whatsapp/cloud-api/support/error-codes/",
                                        "title": "Message failed to send..."
                                    }
                                ],
                                "gs_id": "91f58e90-f5ba-4cfc-9ca7-e4ab965b9449",
                                "id": "wamid.HBgMOTE3OTgwODkwOTc4FQIAERgSNERF...",
                                "recipient_id": "91*******78",
                                "status": "failed",
                                "timestamp": "1710941507"
                            }
                        ]
                    }
                }
            ],
            "id": "112535025189792"
        }
    ],
    "gs_app_id": "82ed52f4-30c0-4f12-81b4-e7ad07bd41de",
    "object": "whatsapp_business_account"
}
```

## Field Descriptions

### Common Fields

-   `app`: App name
-   `timestamp`: Event timestamp in milliseconds
-   `version`: Webhook version (usually 2)
-   `type`: Event type (message-event, user-event, template-event, account-event, etc.)
-   `payload`: Event-specific payload

### Message Event Fields

-   `id`: Message ID (Gupshup ID for enqueued/failed, WhatsApp ID for DLR events)
-   `gsId`: Gupshup Message ID (present in DLR events)
-   `destination`: Recipient phone number
-   `ts`: Meta timestamp (for sent/delivered/read events)
-   `whatsappMessageId`: WhatsApp message ID

### Conversation Fields

-   `id`: Unique conversation ID
-   `expiresAt`: Conversation expiration timestamp
-   `type`: Conversation type (FEP, Marketing, Authentication, Utility, Service)
-   `category`: Conversation category

### Pricing Fields

-   `policy`: Pricing policy (CBP - Conversation Based Pricing, NBP - Notification Based Pricing)
-   `category`: Pricing category

### Template Event Fields

-   `elementName`: Template name
-   `languageCode`: Template language (e.g., en_US)
-   `status`: Template status (approved, rejected, paused, deleted, disabled, in_appeal)
-   `rejectedReason`: Reason for rejection
-   `category`: Template category information

### Account Event Fields

-   `status`: Account status
-   `waId`: WhatsApp Business Account ID
-   `namespace`: Meta template namespace

## Event Processing Notes

1. **Event Order**: The order of notifications may not reflect actual timing. Use timestamps to determine sequence.
2. **Read Receipts**: Read events are only sent for users with read receipts enabled.
3. **GSID Availability**: Events received after a week won't have GSID in the payload.
4. **Template Categories**: Can be Authentication, Marketing, or Utility.
5. **Account Restrictions**: Multiple restriction types can be applied simultaneously.

## Webhook Configuration

To receive these events, configure your webhook URL in the Gupshup dashboard and subscribe to the appropriate event types:

-   MESSAGE mode for message events
-   USER mode for user events
-   ACCOUNT mode for account events
-   TEMPLATE mode for template events

## Error Handling

Always implement proper error handling for:

-   Network failures
-   Invalid JSON
-   Missing required fields
-   Unexpected event types
-   Rate limiting

## Security

-   Validate webhook signatures if provided
-   Use HTTPS for webhook endpoints
-   Implement proper authentication
-   Log all webhook events for debugging

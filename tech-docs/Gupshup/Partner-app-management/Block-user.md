# Gupshup User Blocking API Documentation

This document covers the user blocking and unblocking APIs for Gupshup Partner applications. These APIs allow you to manage blocked users for your WhatsApp Business applications.

> **📚 Reference:** [Meta Documentation](https://developers.facebook.com/docs/whatsapp/cloud-api/block-users/)

## Overview

When you block a WhatsApp user, the following happens:

• The user cannot contact your business or see that you are online.
• Your business cannot message the user. If you do, you will encounter an error.
• You cannot use this API to block another WhatsApp Business.

Errors on the API occur per-number since blocks might be successful on some numbers and not others.

The Block Users API is synchronous.

---

## Block Users

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/user/block`

**Description:** Use this API to block one or more users for the specified app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The Id should be a valid app Id of Gupshup. The App must be associated with the account that owns the PARTNER_APP_TOKEN being used. |
| messaging_product | whatsapp | Messaging service used for the request | String | Required | Must be "whatsapp". Cloud API only. |
| block_users | - | List of user(s) to block | Object | Required | Each element contains a user field. |
| user | - | The phone number or WhatsApp ID to be blocked | String | Required | - |

### Sample Request

**cURL:**
```bash
curl --location 'https://partner.gupshup.io/partner/app/{{APP_ID}}/user/block' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/json' \
--data '{
    "messaging_product": "whatsapp",
    "block_users": [
        {
            "user": "<PHONE_NUMBER>"
        }
    ]
}'
```

**Node.js:**
```javascript
const axios = require('axios');

const blockUsers = async (appId, users, partnerAppToken) => {
  try {
    const response = await axios.post(`https://partner.gupshup.io/partner/app/${appId}/user/block`, {
      messaging_product: "whatsapp",
      block_users: users.map(user => ({ user }))
    }, {
      headers: {
        'Authorization': partnerAppToken,
        'Content-Type': 'application/json'
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Error blocking users:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const blockResult = await blockUsers('your-app-id', ['919163805873'], 'your-partner-app-token');
```

### Sample Response

```json
{
    "block_users": {
        "added_users": [
            {
                "input": "919163805873",
                "wa_id": "919163805873"
            }
        ]
    },
    "messaging_product": "whatsapp",
    "status": "success"
}
```

### Response Fields Description

- `block_users`: Object containing block operation results
  - `added_users`: Array of successfully blocked users
    - `input`: The input phone number or WhatsApp ID
    - `wa_id`: The WhatsApp ID of the blocked user
- `messaging_product`: The messaging service used ("whatsapp")
- `status`: Status of the operation ("success")

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "block_users": {"added_users": [{"input": "919163805873", "wa_id": "919163805873"}]}, "messaging_product": "whatsapp", "status": "success" }` | - |
| Error | 400 | Partial failure response with `added_users`, `failed_users`, and `errors` arrays | When some users fail to block |

### Error Response Format

```json
{
    "messaging_product": "whatsapp",
    "block_users": {
        "added_users": [
            {
                "input": "<PHONE_NUMBER> or <WA_ID>",
                "wa_id": "<WA_ID>"
            }
        ],
        "failed_users": [
            {
                "input": "<PHONE_NUMBER> or <WA_ID>",
                "wa_id": "<WA_ID>",
                "errors": [
                    {
                        "message": "<MESSAGE>",
                        "code": "<CODE>",
                        "error_data": {
                            "details": "<DETAILS>"
                        }
                    }
                ]
            }
        ]
    },
    "error": {
        "message": "(#139100) Failed to block/unblock users",
        "type": "OAuthException",
        "code": 139100,
        "error_data": {
            "details": "Failed to block some users, see the block_users response list for details"
        },
        "fbtrace_id": "<FBTRACE_ID>"
    }
}
```

---

## Get Blocked Users List

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/user/blocklist`

**Description:** Use this API to get a list of users that have been blocked for the specified app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The Id should be a valid app Id of Gupshup. The App must be associated with the account that owns the PARTNER_APP_TOKEN being used. |
| limit | - | Limits the number of blocked users returned in the response | Integer | Optional | Default value is 100 if not specified. Maximum value is 100 |
| after | - | Specifies the starting point for the next set of results (Next page) | String | Optional | The value of this field is present in the response of the get request |

### Sample Request

**cURL:**
```bash
curl --location 'https://partner.gupshup.io/partner/app/{{APP_ID}}/user/blocklist?limit={{LIMIT}}&after={{AFTER}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

**Node.js:**
```javascript
const axios = require('axios');

const getBlockedUsers = async (appId, partnerAppToken, limit = 100, after = null) => {
  try {
    const params = { limit };
    if (after) params.after = after;
    
    const response = await axios.get(`https://partner.gupshup.io/partner/app/${appId}/user/blocklist`, {
      headers: {
        'Authorization': partnerAppToken
      },
      params: params
    });
    
    return response.data;
  } catch (error) {
    console.error('Error fetching blocked users:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const blockedUsers = await getBlockedUsers('your-app-id', 'your-partner-app-token');
```

### Sample Response

```json
{
    "data": [
        {
            "messaging_product": "whatsapp",
            "wa_id": "919163805873"
        },
        {
            "messaging_product": "whatsapp",
            "wa_id": "918910864371"
        }
    ],
    "paging": {
        "cursors": {
            "after": "eyJvZAmZAzZAXQiOjEsInZAlcnNpb25JZACI6IjE3NDEyNTc3OTI1NjY1MDAifQZDZD",
            "before": "eyJvZAmZAzZAXQiOjAsInZAlcnNpb25JZACI6IjE3NDEyNTc3OTI1NjY1MDAifQZDZD"
        }
    },
    "status": "success"
}
```

### Response Fields Description

- `data`: Array of blocked users
  - `messaging_product`: The messaging service ("whatsapp")
  - `wa_id`: The WhatsApp ID of the blocked user
- `paging`: Pagination information
  - `cursors`: Cursor-based pagination
    - `after`: Cursor for the next page
    - `before`: Cursor for the previous page
- `status`: Status of the operation ("success")

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "data": [...], "paging": {...}, "status": "success" }` | - |

---

## Unblock Users

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/user/unblock`

**Description:** Use this API to unblock one or more users for the specified app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The Id should be a valid app Id of Gupshup. The App must be associated with the account that owns the PARTNER_APP_TOKEN being used. |
| messaging_product | whatsapp | Messaging service used for the request | String | Required | Must be "whatsapp". Cloud API only. |
| block_users | - | List of user(s) to unblock | Object | Required | Each element contains a user field. |
| user | - | The phone number or WhatsApp ID to be unblocked | String | Required | - |

### Sample Request

```bash
curl --location 'https://partner.gupshup.io/partner/app/{{APP_ID}}/user/unblock' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/json' \
--data '{
    "messaging_product": "whatsapp",
    "block_users": [
        {
            "user": "<PHONE_NUMBER>"
        }
    ]
}'
```

### Sample Response

```json
{
    "block_users": {
        "removed_users": [
            {
                "input": "919163805873",
                "wa_id": "919163805873"
            }
        ]
    },
    "messaging_product": "whatsapp",
    "status": "success"
}
```

### Response Fields Description

- `block_users`: Object containing unblock operation results
  - `removed_users`: Array of successfully unblocked users
    - `input`: The input phone number or WhatsApp ID
    - `wa_id`: The WhatsApp ID of the unblocked user
- `messaging_product`: The messaging service used ("whatsapp")
- `status`: Status of the operation ("success")

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "block_users": {"removed_users": [{"input": "919163805873", "wa_id": "919163805873"}]}, "messaging_product": "whatsapp", "status": "success" }` | - |
| Error | 400 | Partial failure response with `removed_users`, `failed_users`, and `errors` arrays | When some users fail to unblock |

### Error Response Format

The error response format for unblock is similar to the block API, but with `removed_users` instead of `added_users`:

```json
{
    "messaging_product": "whatsapp",
    "block_users": {
        "removed_users": [
            {
                "input": "<PHONE_NUMBER> or <WA_ID>",
                "wa_id": "<WA_ID>"
            }
        ],
        "failed_users": [
            {
                "input": "<PHONE_NUMBER> or <WA_ID>",
                "wa_id": "<WA_ID>",
                "errors": [
                    {
                        "message": "<MESSAGE>",
                        "code": "<CODE>",
                        "error_data": {
                            "details": "<DETAILS>"
                        }
                    }
                ]
            }
        ]
    },
    "error": {
        "message": "(#139100) Failed to block/unblock users",
        "type": "OAuthException",
        "code": 139100,
        "error_data": {
            "details": "Failed to unblock some users, see the block_users response list for details"
        },
        "fbtrace_id": "<FBTRACE_ID>"
    }
}
```

---

## Important Notes

### General

- All APIs require Partner App Access Token for authentication
- The `messaging_product` must always be "whatsapp"
- You can block/unblock multiple users in a single request
- Operations are synchronous and return immediate results

### Blocking Behavior

- Blocked users cannot contact your business or see that you are online
- Your business cannot message blocked users
- Attempting to message a blocked user will result in an error
- You cannot block another WhatsApp Business account

### Error Handling

- Errors occur per-number since operations might be successful on some numbers and not others
- Partial failures are returned with both successful and failed operations
- Check the `failed_users` array for details on why specific users failed to block/unblock

### Pagination

- The blocklist API supports pagination with `limit` and `after` parameters
- Maximum limit is 100 users per request
- Use the `after` cursor from the response to get the next page of results

### Best Practices

1. Always check for partial failures in the response
2. Handle errors gracefully for individual users
3. Use pagination when retrieving large blocklists
4. Validate phone numbers before attempting to block/unblock
5. Keep track of blocked users in your application for better user experience

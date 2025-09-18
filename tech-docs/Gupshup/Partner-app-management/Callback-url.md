# Gupshup Callback URL Management API Documentation

⚠️ **DEPRECATION NOTICE**: Both APIs below are going to be deprecated soon. It's recommended to start using the [subscription API](https://partner-docs.gupshup.io/reference/setsubscription-api-v3#/) as it provides granular control for event management on your callback URL.

---

## Set Callback URL

**Endpoint:** `PUT https://partner.gupshup.io/partner/app/{appId}/callbackUrl`

**Description:** Use this API to set callback URL for your Gupshup App. Once set, you will get DLR events on that callback URL.

### Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App |
| callbackUrl | {{Callback_URL}} | Learn how you can successfully set a callback URL for an app. |

### Sample Request

```bash
curl --location --request PUT 'https://partner.gupshup.io/partner/app/{{APP_ID}}/callbackUrl' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'callbackUrl={{CALLBACK_URL}}'
```

### Sample Response

```json
{
  "status": "success"
}
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success" }` | - |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Error updating callback URL" }` | For any Internal Error |

### Notes

- This API requires the Partner App Access Token for authentication
- The request uses `application/x-www-form-urlencoded` content type
- Rate limit: 10 requests per minute
- Successfully setting a callback URL will enable DLR events delivery

---

## Update Inbound Events on App's Callback

**Endpoint:** `PUT https://partner.gupshup.io/partner/app/{appId}/callback/mode`

**Description:** Use this API to update the type of events you receive on your callback URL for your Gupshup app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | appId for the app whose business email need to verify. | String | Required | The ID should be a valid app Id of Gupshup It should belong to the same account as the apikey |
| modes | {{MODES}} | Update inbound events that you want to receive on your App's callback URL. You may provide all the values for which you want to receive events. If no values are provided, all events will be deselected. Possible values - DLR events: DELIVERED, READ, SENT, DELETED, and OTHERS. System events: TEMPLATE and ACCOUNT. | String | Required | - |

### Sample Request

```bash
curl --location --request PUT 'https://partner.gupshup.io/partner/app/{{APP_ID}}/callback/mode' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'modes={{MODES}}'
```

### Sample Response

```json
{
    "callback": {
        "custom": false,
        "modes": [
            "SENT",
            "DELIVERED",
            "READ",
            "DELETED"
        ]
    },
    "status": "success"
}
```

### Response Fields Description

- `callback`: Object containing callback configuration
  - `custom`: Boolean indicating if custom callback is enabled
  - `modes`: Array of event types that will be sent to callback URL
- `status`: Status of the request ("success")

### Available Event Modes

**DLR Events:**
- `SENT`: Message sent events
- `DELIVERED`: Message delivered events
- `READ`: Message read events
- `DELETED`: Message deleted events
- `OTHERS`: Other DLR events

**System Events:**
- `TEMPLATE`: Template-related events
- `ACCOUNT`: Account-related events

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "callback": { "custom": false, "modes": [ "SENT", "DELIVERED", "READ", "DELETED" ] }, "status": "success" }` | - |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Internal server error. Please try again later and If Issue still persist than contact Gupshup Dev Support" }` | For any Internal Error |

### Notes

- This API requires the Partner App Access Token for authentication
- The request uses `application/x-www-form-urlencoded` content type
- Rate limit: 10 requests per minute
- If no modes are provided, all events will be deselected
- You can specify multiple event types by providing them in the modes parameter
- The API provides granular control over which events are sent to your callback URL

---

## Node.js Code Examples

### Set Callback URL - Node.js

```javascript
const axios = require('axios');

async function setCallbackUrl(appId, callbackUrl, partnerAppToken) {
  try {
    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/callbackUrl`,
      new URLSearchParams({
        callbackUrl: callbackUrl
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Callback URL set successfully:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error setting callback URL:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
setCallbackUrl('your-app-id', 'https://your-domain.com/callback', 'your-partner-app-token');
```

### Update Callback Mode - Node.js

```javascript
const axios = require('axios');

async function updateCallbackMode(appId, modes, partnerAppToken) {
  try {
    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/callback/mode`,
      new URLSearchParams({
        modes: modes
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Callback mode updated successfully:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating callback mode:', error.response?.data || error.message);
    throw error;
  }
}

// Usage examples
updateCallbackMode('your-app-id', 'SENT,DELIVERED,READ', 'your-partner-app-token');
updateCallbackMode('your-app-id', 'TEMPLATE,ACCOUNT', 'your-partner-app-token');
```

### Complete Example with Error Handling

```javascript
const axios = require('axios');

class GupshupCallbackManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseUrl = 'https://partner.gupshup.io/partner/app';
  }

  async setCallbackUrl(appId, callbackUrl) {
    try {
      const response = await axios.put(
        `${this.baseUrl}/${appId}/callbackUrl`,
        new URLSearchParams({ callbackUrl }),
        {
          headers: {
            'Authorization': this.partnerAppToken,
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }

  async updateCallbackMode(appId, modes) {
    try {
      const response = await axios.put(
        `${this.baseUrl}/${appId}/callback/mode`,
        new URLSearchParams({ modes }),
        {
          headers: {
            'Authorization': this.partnerAppToken,
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }
}

// Usage
const callbackManager = new GupshupCallbackManager('your-partner-app-token');

async function main() {
  // Set callback URL
  const result1 = await callbackManager.setCallbackUrl('your-app-id', 'https://your-domain.com/callback');
  console.log('Set callback URL result:', result1);

  // Update callback modes
  const result2 = await callbackManager.updateCallbackMode('your-app-id', 'SENT,DELIVERED,READ,DELETED');
  console.log('Update callback mode result:', result2);
}

main();
```

---

## Important Notes

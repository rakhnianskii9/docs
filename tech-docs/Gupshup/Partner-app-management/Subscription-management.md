# Gupshup Subscription Management API Documentation

This document covers all the subscription management APIs for Gupshup Partner applications. Subscriptions provide granular control for event management on your callback URLs.

---

## Get All Subscriptions

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/subscription`

**Description:** Use this API to get all the subscriptions of an app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The Id should be a valid app Id of Gupshup |

### Sample Request

```bash
curl --location 'https://partner.gupshup.io/partner/app/{{APPId}}/subscription' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
	"status": "success",
	"subscriptions": [
		{
			"active": true,
			"appId": "57d9179b-7412-4621-bf86-57ee1962fd12",
			"createdOn": 1735818105594,
			"id": "29040",
			"mode": 16383,
			"modes": [
				"SENT",
				"DELIVERED",
				"READ",
				"DELETED",
				"OTHERS",
				"FAILED",
				"MESSAGE",
				"TEMPLATE",
				"ACCOUNT",
				"BILLING",
				"ENQUEUED",
				"FLOWS_MESSAGE",
				"PAYMENTS",
				"ALL"
			],
			"modifiedOn": 1735818105594,
			"showOnUI": false,
			"tag": "mm",
			"url": "https://webhook.site/e1d58c71-2c14-4e82-90ce-4fd4c0caaf96",
			"version": 3
		}
	]
}
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 204 | `{ "status": "success", "subscriptions": [...] }` | - |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | - |
| Error | 500 | `{ "status": "error", "message": "Internal server error. Please try again later and If issue still persist, then contact Gupshup Dev Support" }` | - |

---

## Get Specific App Subscription

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/subscription/{subscriptionId}`

**Description:** Use this API to get a specific subscription of an app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The Id should be a valid app Id of Gupshup. The App must be associated with the account that owns the PARTNER_APP_TOKEN being used |
| SUBSCRIPTION_ID | 2xxxx | The identifier for the subscription to retrieve | String | Required | - |

### Sample Request

```bash
curl --location 'https://partner.gupshup.io/partner/app/{{APP_ID}}/subscription/{{SUBSCRIPTION_ID}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
  "status": "success",
  "subscription": {
    "active": true,
    "appId": "657c0203-0b4d-4ba1-bbf5-a679cfa35a16",
    "createdOn": 1739862125507,
    "id": "32595",
    "latencyBucket": "lt_1_s",
    "mode": 1025,
    "modes": ["SENT", "ENQUEUED"],
    "modifiedOn": 1739870558358,
    "showOnUI": false,
    "tag": "V33i4",
    "url": "https://01hqjda5pbywgv7xw5e9ckd5e800-51132d4cc32709e9078d.requestinspector.com",
    "version": 2
  }
}
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "subscription": {...} }` | - |
| Error | 400 | `{ "status": "error", "message": "API key is not associated with partner" }` | When API key not found on partner DB |
| Error | 400 | `{ "status": "error", "message": "Subscription Does Not Exist" }` | When a subscription does not exist with the provided subscriptionId |
| Error | 401 | `{ "status": "error", "message": "Authentication Failed" }` | When authentication fails |
| Error | 403 | `{ "status": "error", "message": "Not Subscription Owner" }` | When the requested subscription is not associated with the provided appId |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | - |

---

## Set Subscription for an App

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/subscription`

**Description:** Use this API to create a new subscription for an app.

> **Note:** Subscriptions can now be set for sandbox apps as well. Once the app goes live, the current subscription will be retained.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| PARTNER_APP_TOKEN | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| modes | {{modes}} | Stages in subscription processing | String | Required | Should be one of the following: NONE, READ, DELIVERED, SENT, DELETED, FLOWS_MESSAGE, PAYMENTS, ALL, OTHERS. For version 3: TEMPLATE, ACCOUNT, COPY are not supported |
| tag | {{tagName}} | Name tag for URL | String | Required | Must be unique and mandatory for each app |
| url | {{url}} | Callback URL | String | Required | Should be a valid URL |
| version | {{versionNumber}} | Version | String | Required | Should be one of the following: 0, 1, 2, 3 |
| showOnUI | {{true/false}} | Used internally | Boolean | Optional | Default value is false |
| meta | {{meta}} | Meta json string (meta json key and value will be passed to the subscription URL as headers, users can set custom headers for the URL which can be used for authentication) | String | Optional | - |

### Sample Request

```bash
curl --location 'https://partner.gupshup.io/partner/app/:appId/subscription' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--data-urlencode 'modes={{modes}}' \
--data-urlencode 'tag={{tagName}}' \
--data-urlencode 'showOnUI={{true/false}}' \
--data-urlencode 'version={{versionNumber}}' \
--data-urlencode 'url={{url}}' \
--data-urlencode 'meta={{meta}}'
```

### Sample Response

```json
{
	"status": "success",
	"subscription": {
		"active": true,
		"appId": "bf9ee64c-xxxxxxxx-xxxx-xxxx577007c4",
		"createdOn": 1705574838954,
		"id": "8166",
		"mode": 2047,
		"meta": "{\"headers\": {\"Authorisation\":\"Bearer eyJhbGciOiJIxxxxxxxxxxxxxxxxxxxxxxxxxx6biZ7Sbhl0N0u_aI\"}}",
		"modifiedOn": 1705574838954,
		"showOnUI": false,
		"tag": "V3 Subscription",
		"url": "https://webhook.site/0b092322-b55b-419e-9c74-eb92f2b38553",
		"version": 3
	}
}
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "subscription": {...} }` | - |
| Error | 400 | `{ "status": "error", "message": "Invalid URL Passed" }` | The URL is not valid |
| Error | 401 | `{ "status": "error", "message": "Authentication Failed" }` | When authentication fails |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | When the rate limit is exceeded |
| Error | 500 | `{ "status": "error", "message": "Unable To Create Subscription" }` | When error occurred while creating the subscription |

### Available Modes

- **TEMPLATE**: The template events get forwarded to subscriptions if TEMPLATE mode is subscribed.
- **ACCOUNT**: The account events get forwarded to subscriptions if ACCOUNT mode is subscribed.
- **PAYMENTS**: Incoming WhatsApp pay events get forwarded to subscriptions if PAYMENTS mode is subscribed. [Only for v3].
- **FLOWS_MESSAGE**: Incoming flow messages get forwarded to subscriptions if FLOWS_MESSAGE mode is subscribed. [Only for v3].
- **MESSAGE**: All incoming messages (not events) except flow messages get forwarded to subscriptions if MESSAGE mode is subscribed.
- **OTHERS**: All incoming new events whose exclusive mode is not present (eg read, delivered, sent, payments) get forwarded to subscriptions if OTHERS mode is subscribed [Only for v3].
- **ALL**: All incoming messages (not events) get forwarded to subscriptions if ALL mode is subscribed [Only for v3].
- **BILLING**: Billing events get forwarded to subscriptions if billing mode is subscribed.
- **FAILED**: The failed events get forwarded to subscriptions if failed mode is subscribed.
- **SENT**: Sent events get forwarded to subscriptions if sent mode is subscribed.
- **DELIVERED**: Delivered events get forwarded to subscriptions if delivered mode is subscribed.
- **READ**: Read events get forwarded to subscriptions if read mode is subscribed.
- **ENQUEUED**: Enqueued events get forwarded to subscriptions if enqueued mode is subscribed.

---

## Update App Subscription

**Endpoint:** `PUT https://partner.gupshup.io/partner/app/{appId}/subscription/{subscriptionId}`

**Description:** Use this API to update a specific subscription of an app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The Id should be a valid app Id of Gupshup. The App must be associated with the account that owns the PARTNER_APP_TOKEN being used |
| SUBSCRIPTION_ID | 2xxxx | The identifier for the subscription to update | String | Required | - |
| modes | <MODES> | Stages in subscription processing | String | Optional | Should be one of the following: NONE, READ, DELIVERED, SENT, DELETED, OTHERS. For version 3 TEMPLATE, ACCOUNT, COPY are not supported |
| tag | <name_tag_for_url> | Name tag for URL | String | Optional | Tag must be unique for each app |
| url | <callback_url> | Callback URL | String | Optional | Should be valid URL |
| version | <version> | Version | Integer | Optional | Should be one of the following: 0, 1, 2, 3 |
| doCheck | <for_bypassing_url_check> | For bypassing URL check | Boolean | Optional | Should be a boolean value: true/false |
| active | <true/false> | Active status of subscription | Boolean | Optional | Should be a boolean value: true/false |

### Sample Request

```bash
curl --location --request PUT 'https://partner.gupshup.io/partner/app/{{APP_ID}}/subscription/{{SUBSCRIPTION_ID}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'url=<callback_url>' \
--data-urlencode 'tag=<name_tag_for_url>' \
--data-urlencode 'version=<Version>' \
--data-urlencode 'modes=<MODES>' \
--data-urlencode 'doCheck=<for_bypassing_url_check>' \
--data-urlencode 'active=<true/false>'
```

### Sample Response

```json
{
  "status": "success",
  "subscription": {
    "active": true,
    "appId": "657c0203-0b4d-4ba1-bbf5-a679cfa35a16",
    "createdOn": 1739862125507,
    "id": "32595",
    "latencyBucket": "lt_1_s",
    "mode": 1025,
    "modes": ["SENT", "ENQUEUED"],
    "modifiedOn": 1739870558358,
    "showOnUI": false,
    "tag": "V33i4",
    "url": "https://01hqjda5pbywgv7xw5e9ckd5e800-51132d4cc32709e9078d.requestinspector.com",
    "version": 2
  }
}
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "subscription": {...} }` | - |
| Error | 400 | `{ "status": "error", "message": "API key is not associated with partner" }` | When API key not found on partner DB |
| Error | 400 | `{ "status": "error", "message": "Subscription Does Not Exist" }` | When a subscription does not exist with the provided subscriptionId |
| Error | 401 | `{ "status": "error", "message": "Authentication Failed" }` | When authentication fails |
| Error | 403 | `{ "status": "error", "message": "Not Subscription Owner" }` | When the requested subscription is not associated with the provided appId |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | - |

---

## Delete Specific Subscription

**Endpoint:** `DELETE https://partner.gupshup.io/partner/app/{appId}/subscription/{subscriptionId}`

**Description:** Use this API to delete specific subscription for an app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Partner app access token issued post partner login | String | Required | Should be a valid Partner app access token belonging to the passed appId |
| appId | {{App_ID}} | App ID for the app | String | Required | Valid app ID |
| SUBSCRIPTION_ID | 2xxxx | Subscription ID | String | Required | Id of the subscription to be deleted |

### Sample Request

```bash
curl --location --request DELETE 'https://partner.gupshup.io/partner/app/{{APP_ID}}/subscription/{{SUBSCRIPTION_ID}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
No content returned (204 status code)
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 204 | - | Success when subscription is deleted. No content. |
| Error | 400 | `{ "status": "error", "message": "API key is not associated with partner" }` | Error with respect to API Key |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |

---

## Delete All Subscriptions

**Endpoint:** `DELETE https://partner.gupshup.io/partner/app/{appId}/subscription`

**Description:** Use this API to delete all subscriptions for an app.

### Request Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {PARTNER_APP_TOKEN} | Partner app access token issued post partner login | String | Required | Should be a valid Partner app access token belonging to the passed appId |
| appId | {App_ID} | App ID for the app | String | Required | Valid app ID |

### Sample Request

```bash
curl --location --request DELETE 'https://partner.gupshup.io/partner/app/:appId/subscription' \
--header 'Authorization: {PARTNER_APP_TOKEN}'
```

### Sample Response

```json
No content returned (204 status code)
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 204 | - | Success when all subscriptions are deleted. No content. |
| Error | 400 | `{ "status": "error", "message": "<Specific to API>" }` | Error with respect to API |
| Error | 401 | `{ "status": "error", "message": "Unauthorized Access" }` | When authentication fails |
| Error | 403 | `{ "status": "error", "message": "Unauthorized Access" }` | Access is not allowed |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 5 calls per minute |
| Error | 500 | `{ "status": "error", "message": "Internal Server Error" }` | For any Internal Error |

---

## Subscription Object Fields

Common fields in subscription responses:

- `active`: Boolean indicating if the subscription is active
- `appId`: The app ID this subscription belongs to
- `createdOn`: Timestamp of when the subscription was created
- `id`: Unique identifier for the subscription
- `latencyBucket`: Latency bucket classification (e.g., "lt_1_s")
- `mode`: Numeric representation of the modes
- `modes`: Array of event modes subscribed to
- `modifiedOn`: Timestamp of when the subscription was last modified
- `showOnUI`: Boolean indicating if the subscription should be shown in UI
- `tag`: Unique tag for the subscription
- `url`: Callback URL for events
- `version`: Version of the subscription API (0, 1, 2, or 3)
- `meta`: Optional metadata with custom headers for authentication

---

## Node.js Code Examples

### Create Subscription - Node.js

```javascript
const axios = require('axios');

async function createSubscription(appId, subscriptionData, partnerAppToken) {
  try {
    const response = await axios.post(
      `https://partner.gupshup.io/partner/app/${appId}/subscription`,
      new URLSearchParams({
        phoneNumber: subscriptionData.phoneNumber,
        webhook: subscriptionData.webhook,
        channel: subscriptionData.channel,
        contactName: subscriptionData.contactName,
        isActive: subscriptionData.isActive
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Subscription created successfully:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error creating subscription:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
createSubscription('your-app-id', {
  phoneNumber: '+1234567890',
  webhook: 'https://your-domain.com/webhook',
  channel: 'WhatsApp',
  contactName: 'John Doe',
  isActive: true
}, 'your-partner-app-token');
```

### Get Subscription - Node.js

```javascript
const axios = require('axios');

async function getSubscription(appId, subscriptionId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/subscription/${subscriptionId}`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Subscription details:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting subscription:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getSubscription('your-app-id', 'subscription-id', 'your-partner-app-token');
```

### Update Subscription - Node.js

```javascript
const axios = require('axios');

async function updateSubscription(appId, subscriptionId, updateData, partnerAppToken) {
  try {
    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/subscription/${subscriptionId}`,
      new URLSearchParams({
        webhook: updateData.webhook,
        contactName: updateData.contactName,
        isActive: updateData.isActive
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Subscription updated successfully:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating subscription:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
updateSubscription('your-app-id', 'subscription-id', {
  webhook: 'https://your-domain.com/new-webhook',
  contactName: 'Jane Doe',
  isActive: false
}, 'your-partner-app-token');
```

### Delete Subscription - Node.js

```javascript
const axios = require('axios');

async function deleteSubscription(appId, subscriptionId, partnerAppToken) {
  try {
    const response = await axios.delete(
      `https://partner.gupshup.io/partner/app/${appId}/subscription/${subscriptionId}`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Subscription deleted successfully:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error deleting subscription:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
deleteSubscription('your-app-id', 'subscription-id', 'your-partner-app-token');
```

### List Subscriptions - Node.js

```javascript
const axios = require('axios');

async function listSubscriptions(appId, partnerAppToken, options = {}) {
  try {
    const params = new URLSearchParams();
    
    if (options.page) params.append('page', options.page);
    if (options.limit) params.append('limit', options.limit);
    if (options.channel) params.append('channel', options.channel);
    if (options.isActive !== undefined) params.append('isActive', options.isActive);
    
    const queryString = params.toString();
    const url = `https://partner.gupshup.io/partner/app/${appId}/subscription${queryString ? '?' + queryString : ''}`;
    
    const response = await axios.get(url, {
      headers: {
        'Authorization': partnerAppToken
      }
    });
    
    console.log('Subscriptions list:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error listing subscriptions:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
listSubscriptions('your-app-id', 'your-partner-app-token', {
  page: 1,
  limit: 10,
  channel: 'WhatsApp',
  isActive: true
});
```

### Complete Subscription Manager Class - Node.js

```javascript
const axios = require('axios');

class GupshupSubscriptionManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseUrl = 'https://partner.gupshup.io/partner/app';
  }

  async createSubscription(appId, subscriptionData) {
    try {
      const response = await axios.post(
        `${this.baseUrl}/${appId}/subscription`,
        new URLSearchParams(subscriptionData),
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

  async getSubscription(appId, subscriptionId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/subscription/${subscriptionId}`,
        {
          headers: {
            'Authorization': this.partnerAppToken
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

  async updateSubscription(appId, subscriptionId, updateData) {
    try {
      const response = await axios.put(
        `${this.baseUrl}/${appId}/subscription/${subscriptionId}`,
        new URLSearchParams(updateData),
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

  async deleteSubscription(appId, subscriptionId) {
    try {
      const response = await axios.delete(
        `${this.baseUrl}/${appId}/subscription/${subscriptionId}`,
        {
          headers: {
            'Authorization': this.partnerAppToken
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

  async listSubscriptions(appId, options = {}) {
    try {
      const params = new URLSearchParams();
      Object.entries(options).forEach(([key, value]) => {
        if (value !== undefined) params.append(key, value);
      });
      
      const queryString = params.toString();
      const url = `${this.baseUrl}/${appId}/subscription${queryString ? '?' + queryString : ''}`;
      
      const response = await axios.get(url, {
        headers: {
          'Authorization': this.partnerAppToken
        }
      });
      
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
const subscriptionManager = new GupshupSubscriptionManager('your-partner-app-token');

async function main() {
  const appId = 'your-app-id';
  
  // Create subscription
  const createResult = await subscriptionManager.createSubscription(appId, {
    phoneNumber: '+1234567890',
    webhook: 'https://your-domain.com/webhook',
    channel: 'WhatsApp',
    contactName: 'John Doe',
    isActive: true
  });
  console.log('Create subscription result:', createResult);

  // Get subscription
  if (createResult.success) {
    const subscriptionId = createResult.data.id;
    const getResult = await subscriptionManager.getSubscription(appId, subscriptionId);
    console.log('Get subscription result:', getResult);
  }
  
  // List subscriptions
  const listResult = await subscriptionManager.listSubscriptions(appId, {
    page: 1,
    limit: 10,
    channel: 'WhatsApp'
  });
  console.log('List subscriptions result:', listResult);
}

main();
```

---

## Important Notes

- All APIs require Partner App Access Token for authentication
- Subscription tags must be unique for each app
- Version 3 provides the most comprehensive event support
- Rate limits apply to all endpoints
- The `meta` field allows setting custom headers for webhook authentication
- Sandbox apps can have subscriptions that persist when going live

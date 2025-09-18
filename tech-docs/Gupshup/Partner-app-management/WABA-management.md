# Gupshup WABA Management API Documentation

This document covers the WABA (WhatsApp Business Account) Management APIs for Gupshup Partner applications. These APIs allow you to monitor app health, check wallet balance, get quality ratings, and retrieve comprehensive WABA information.

---

## Check App Health

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/health`

**Description:** Use this API to check the health of the Gupshup App.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/health' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "status": "success",
    "healthy": "true"
}
```

### Response Fields Description

- `status`: Status of the request ("success")
- `healthy`: Health status of the app ("true" or "false")

### Node.js Example

```javascript
const axios = require('axios');

async function checkAppHealth(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/health`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('App health status:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error checking app health:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
checkAppHealth('your-app-id', 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "healthy": "true" }` | App is healthy |
| Error | 400 | `{ "message": "App is not live.", "status": "error" }` | If app is not live |
| Error | 400 | `{ "message": "Waba id not found for the given App", "status": "error" }` | If the WABA ID is not present in the DB |
| Error | 400 | `{ "message": "WABA id is invalid or given phone is not associated with WABA", "status": "error" }` | If the WABA ID present in DB is not correct |
| Error | 400 | `{ "message": "Error while getting waba info for given app", "status": "error" }` | General error |
| Error | 400 | `{ "message": "Authentication Failed", "status": "error" }` | Authentication failed |

---

## Get Wallet Balance

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/wallet/balance`

**Description:** Use this API to fetch your wallet balance.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/wallet/balance' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "walletResponse": {
        "currency": "USD",
        "currentBalance": 150.5000,
        "overDraftLimit": 0
    }
}
```

### Response Fields Description

- `walletResponse`: Object containing wallet information
  - `currency`: Currency of the wallet (e.g., "USD")
  - `currentBalance`: Current balance amount
  - `overDraftLimit`: Overdraft limit (usually 0)

### Node.js Example

```javascript
const axios = require('axios');

async function getWalletBalance(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/wallet/balance`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Wallet balance:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting wallet balance:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getWalletBalance('your-app-id', 'your-partner-app-token');
```

---

## Get Quality Rating

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/ratings`

**Description:** Use this API to check the Quality Rating and Messaging Limits of your Gupshup app.

> **📘 Note:** For an App, API requests are limited to once every 24 hours. To know more about Messaging Limits, visit [Facebook Documentation](https://developers.facebook.com/docs/whatsapp/api/rate-limits). To learn about Quality Rating (Green, Yellow, Red), visit [Facebook Business Help](https://www.facebook.com/business/help/896873687365001).

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/ratings' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

**When event data is available:**
```json
{
    "oldLimit": "TIER_10K",
    "currentLimit": "TIER_10K",
    "event": "ONBOARDING",
    "eventTime": 1234567890,
    "phoneQuality": "GREEN"
}
```

**When no event data is available:**
```json
{
    "message": "no event update available",
    "status": "success"
}
```

### Response Fields Description

**With Event Data:**
- `oldLimit`: Previous messaging limit tier
- `currentLimit`: Current messaging limit tier
- `event`: Type of event (e.g., "ONBOARDING")
- `eventTime`: Unix timestamp of the event
- `phoneQuality`: Quality rating ("GREEN", "YELLOW", "RED")

**Without Event Data:**
- `message`: Informational message
- `status`: Status of the request

### Node.js Example

```javascript
const axios = require('axios');

async function getQualityRating(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/ratings`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Quality rating:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting quality rating:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getQualityRating('your-app-id', 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "oldLimit": "TIER_10K", "currentLimit": "TIER_10K", "event": "ONBOARDING", "eventTime": 1234567890, "phoneQuality": "GREEN" }` | Event data available |
| Success | 200 | `{ "message": "no event update available", "status": "success" }` | No event data available |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Internal server error. Please try again later and If Issue still persist than contact Gupshup Dev Support" }` | Internal server error |

---

## Get WABA Information

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/waba/info/`

**Description:** This API provides an overview of WABA health and detailed information.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| token | {{PARTNER_APP_TOKEN}} | App Access Token issued post Partner login | String | Required |
| appId | {{APP_ID}} | App ID for the app that is linked to the Partner Account | String | Required |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/waba/info' \
--header 'token: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "status": "success",
    "wabaInfo": {
        "accountStatus": "ACTIVE",
        "dockerStatus": "CONNECTED",
        "messagingLimit": "TIER_1K",
        "mmLiteStatus": "ELIGIBLE",
        "ownershipType": "CLIENT_OWNED",
        "phone": "917835039205",
        "phoneQuality": "GREEN",
        "throughput": "STANDARD",
        "verifiedName": "Your Business Name",
        "wabaId": "116676371511027",
        "canSendMessage": "AVAILABLE",
        "errors": [],
        "additionalInfo": []
    }
}
```

### Response Fields Description

- `status`: Status of the request ("success")
- `wabaInfo`: Object containing comprehensive WABA information
  - `accountStatus`: Account status ("ACTIVE", "BANNED")
  - `dockerStatus`: Docker connection status ("CONNECTED", "DISCONNECTED", "FLAGGED", "PENDING", "RESTRICTED", "UNKNOWN")
  - `messagingLimit`: Current messaging tier ("TIER_50", "TIER_250", "TIER_1K", "TIER_10K", "TIER_100K", "TIER_NOT_SET", "TIER_UNLIMITED")
  - `mmLiteStatus`: MM Lite eligibility ("INELIGIBLE", "ELIGIBLE", "ONBOARDED")
  - `ownershipType`: Ownership type ("CLIENT_OWNED", "ON_BEHALF_OF", "SELF")
  - `phone`: Phone number associated with WABA
  - `phoneQuality`: Phone quality rating ("GREEN", "YELLOW", "RED", "UNKNOWN")
  - `throughput`: Throughput level ("HIGH", "STANDARD", "NOT_APPLICABLE")
  - `verifiedName`: Verified display name
  - `wabaId`: WhatsApp Business Account ID
  - `canSendMessage`: Message sending capability ("AVAILABLE", "LIMITED", "BLOCKED")
  - `errors`: Array of error objects (if any)
  - `additionalInfo`: Array of additional information messages

### Error Object Structure

```json
{
    "error_code": "141014",
    "error_description": "Error description",
    "possible_solution": "Possible solution"
}
```

### Node.js Example

```javascript
const axios = require('axios');

async function getWabaInfo(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/waba/info`,
      {
        headers: {
          'token': partnerAppToken
        }
      }
    );
    
    console.log('WABA information:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting WABA info:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getWabaInfo('your-app-id', 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | WABA info with `canSendMessage: "AVAILABLE"` | Full messaging capability |
| Success | 200 | WABA info with `canSendMessage: "LIMITED"` | Limited messaging capability |
| Success | 200 | WABA info with `canSendMessage: "BLOCKED"` | Blocked messaging capability |
| Error | 400 | `{ "message": "App is not live.", "status": "error" }` | If app is not live |
| Error | 400 | `{ "message": "Waba id not found for the given App", "status": "error" }` | If WABA ID is not present in DB |
| Error | 400 | `{ "message": "WABA id is invalid or given phone is not associated with WABA", "status": "error" }` | If WABA ID is incorrect |
| Error | 400 | `{ "message": "Error while getting waba info for given app", "status": "error" }` | General error |
| Error | 401 | `{ "message": "Authentication Failed", "status": "error" }` | Authentication failed |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Internal Server Error" }` | Internal server error |

---

## Complete WABA Management Class - Node.js

```javascript
const axios = require('axios');

class GupshupWABAManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseUrl = 'https://partner.gupshup.io/partner/app';
  }

  async checkAppHealth(appId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/health`,
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

  async getWalletBalance(appId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/wallet/balance`,
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

  async getQualityRating(appId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/ratings`,
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

  async getWabaInfo(appId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/waba/info`,
        {
          headers: {
            'token': this.partnerAppToken
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

  // Helper method to get comprehensive app status
  async getComprehensiveAppStatus(appId) {
    try {
      const [healthResult, balanceResult, ratingResult, wabaResult] = await Promise.allSettled([
        this.checkAppHealth(appId),
        this.getWalletBalance(appId),
        this.getQualityRating(appId),
        this.getWabaInfo(appId)
      ]);

      return {
        health: healthResult.status === 'fulfilled' ? healthResult.value : { success: false, error: healthResult.reason },
        balance: balanceResult.status === 'fulfilled' ? balanceResult.value : { success: false, error: balanceResult.reason },
        rating: ratingResult.status === 'fulfilled' ? ratingResult.value : { success: false, error: ratingResult.reason },
        waba: wabaResult.status === 'fulfilled' ? wabaResult.value : { success: false, error: wabaResult.reason }
      };
    } catch (error) {
      return { 
        success: false, 
        error: error.message 
      };
    }
  }

  // Helper method to interpret phone quality
  interpretPhoneQuality(quality) {
    const qualityMap = {
      'GREEN': 'Excellent - No restrictions',
      'YELLOW': 'Warning - Some restrictions may apply',
      'RED': 'Poor - Severe restrictions apply',
      'UNKNOWN': 'Quality status unknown'
    };
    return qualityMap[quality] || 'Unknown quality status';
  }

  // Helper method to interpret messaging limits
  interpretMessagingLimit(limit) {
    const limitMap = {
      'TIER_50': '50 messages per day',
      'TIER_250': '250 messages per day',
      'TIER_1K': '1,000 messages per day',
      'TIER_10K': '10,000 messages per day',
      'TIER_100K': '100,000 messages per day',
      'TIER_NOT_SET': 'Limit not set',
      'TIER_UNLIMITED': 'Unlimited messages'
    };
    return limitMap[limit] || 'Unknown limit';
  }

  // Helper method to check if app can send messages
  canSendMessages(wabaInfo) {
    return wabaInfo?.canSendMessage === 'AVAILABLE';
  }

  // Helper method to get app issues
  getAppIssues(wabaInfo) {
    const issues = [];
    
    if (wabaInfo?.canSendMessage === 'BLOCKED') {
      issues.push('Messaging is blocked');
    }
    
    if (wabaInfo?.canSendMessage === 'LIMITED') {
      issues.push('Messaging is limited');
    }
    
    if (wabaInfo?.phoneQuality === 'RED') {
      issues.push('Poor phone quality rating');
    }
    
    if (wabaInfo?.phoneQuality === 'YELLOW') {
      issues.push('Phone quality needs attention');
    }
    
    if (wabaInfo?.dockerStatus !== 'CONNECTED') {
      issues.push(`Docker status: ${wabaInfo.dockerStatus}`);
    }
    
    if (wabaInfo?.accountStatus !== 'ACTIVE') {
      issues.push(`Account status: ${wabaInfo.accountStatus}`);
    }
    
    return issues;
  }
}

// Usage
const wabaManager = new GupshupWABAManager('your-partner-app-token');

async function main() {
  const appId = 'your-app-id';
  
  // Get comprehensive status
  const status = await wabaManager.getComprehensiveAppStatus(appId);
  console.log('Comprehensive app status:', status);
  
  // Check individual components
  const health = await wabaManager.checkAppHealth(appId);
  console.log('App health:', health);
  
  const balance = await wabaManager.getWalletBalance(appId);
  console.log('Wallet balance:', balance);
  
  const rating = await wabaManager.getQualityRating(appId);
  console.log('Quality rating:', rating);
  
  const wabaInfo = await wabaManager.getWabaInfo(appId);
  console.log('WABA info:', wabaInfo);
  
  // Analyze WABA status
  if (wabaInfo.success) {
    const canSend = wabaManager.canSendMessages(wabaInfo.data.wabaInfo);
    const issues = wabaManager.getAppIssues(wabaInfo.data.wabaInfo);
    const qualityInterpretation = wabaManager.interpretPhoneQuality(wabaInfo.data.wabaInfo.phoneQuality);
    const limitInterpretation = wabaManager.interpretMessagingLimit(wabaInfo.data.wabaInfo.messagingLimit);
    
    console.log('Can send messages:', canSend);
    console.log('Quality:', qualityInterpretation);
    console.log('Messaging limit:', limitInterpretation);
    console.log('Issues:', issues);
  }
}

main();
```

---

## Important Notes

### General

- All APIs require Partner App Access Token for authentication
- Note that some endpoints use `Authorization` header while others use `token` header
- Rate limiting applies to most endpoints (typically 10 requests per minute)

### Health Monitoring

- **App Health**: Provides basic health status of the application
- **WABA Info**: Provides comprehensive health and status information
- **Quality Rating**: Limited to once every 24 hours per app

### Wallet Management

- **Balance Checking**: Monitor your account balance to ensure uninterrupted service
- **Currency**: Balance is typically shown in USD
- **Overdraft**: Most accounts have 0 overdraft limit

### Quality and Limits

- **Phone Quality**: GREEN (good), YELLOW (warning), RED (poor)
- **Messaging Limits**: Range from 50 messages/day to unlimited
- **Event-Based**: Quality ratings are updated based on WhatsApp events

### Message Sending Capability

- **AVAILABLE**: Full messaging capability
- **LIMITED**: Some restrictions apply
- **BLOCKED**: Messaging is disabled

### Best Practices

1. **Regular Monitoring**: Check app health and WABA status regularly
2. **Balance Alerts**: Monitor wallet balance to avoid service interruption
3. **Quality Maintenance**: Maintain good phone quality ratings
4. **Error Handling**: Implement proper error handling for all status checks
5. **Rate Limiting**: Respect API rate limits, especially for quality rating checks
6. **Issue Resolution**: Address any issues indicated in WABA info promptly

### Error Handling

- Handle authentication failures appropriately
- Check for app live status before making requests
- Monitor for WABA configuration issues
- Implement retry logic for temporary failures
- Log errors for troubleshooting and monitoring

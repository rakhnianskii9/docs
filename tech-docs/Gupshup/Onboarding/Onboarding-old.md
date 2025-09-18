# Gupshup Partner API - Onboarding Documentation

## Overview
The Gupshup Partner API provides comprehensive onboarding endpoints for managing WhatsApp Business Account (WABA) applications. These APIs allow partners to create applications programmatically and generate embed links for client onboarding. The actual onboarding process happens through Gupshup's embedded UI popup, where clients complete Facebook Business Manager integration and phone number setup independently.

## Base URL
**Base URL:** `https://partner.gupshup.io/partner`

## Authentication
All endpoints require a **Partner JWT Token** in the request header:
```
token: YOUR_PARTNER_JWT_TOKEN
```

## Rate Limiting
- **Standard Rate Limit:** 10 requests per minute
- **Some endpoints:** 10 requests per second
- **429 Status Code:** Returned when rate limit is exceeded

## Complete Onboarding Process

The Gupshup onboarding process consists of the following steps:

1. **Partner creates WABA application** via API (`POST /partner/app`)
2. **Partner optionally sets contact details** (`PUT /partner/app/{appId}/onboarding/contact`)
3. **Partner generates embed link** (`GET /partner/app/{appId}/onboarding/embed/link`)
4. **Client opens embed link** - Gupshup popup window appears
5. **Client completes onboarding in popup:**
   - Connects to Facebook Business Manager
   - Selects or creates WhatsApp Business Account
   - Chooses phone number
   - Completes verification process
6. **Gupshup handles all Facebook integration** automatically
7. **Application becomes live** and ready for messaging

> **Key Point:** Partners only need to create the app and provide the embed link. All user interaction happens within Gupshup's embedded interface - no custom forms or UI components are required in your application.

## Onboarding API Endpoints Overview

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/partner/app` | POST | Create a new WABA application |
| `/partner/app/{appId}` | PUT | Update application settings |
| `/partner/app/{appId}/details` | GET | Get application details |
| `/partner/app/list` | GET | List all applications with filtering |
| `/partner/app/{appId}/onboarding/contact` | PUT | Set business contact details |
| `/partner/app/{appId}/onboarding/contact/email/resend` | POST | Resend email verification |
| `/partner/app/{appId}/onboarding/embed/link` | GET | Generate embed signed link |
| `/partner/app/{appId}/onboarding/phoneMigration` | POST | Mark app for migration |

---

## 1. Create Application

### POST `/partner/app`

Create a new WABA application outside of the Gupshup UI.

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| name | App name | String | Required | 6-150 characters, no special characters |
| templateMessaging | Enable template messaging | Boolean | Optional | Default: false |
| disableOptinPrefUrl | Disable optin preferences | Boolean | Optional | Default: false |

#### cURL Example

```bash
curl -X POST "https://partner.gupshup.io/partner/app" \
  -H "token: YOUR_PARTNER_JWT_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "name=MyWhatsAppApp" \
  -d "templateMessaging=true" \
  -d "disableOptinPrefUrl=false"
```

#### Node.js Example

```javascript
const axios = require('axios');

class GupshupPartnerAPI {
    constructor(partnerToken) {
        this.partnerToken = partnerToken;
        this.baseURL = 'https://partner.gupshup.io/partner';
    }

    async createApp(name, templateMessaging = false, disableOptinPrefUrl = false) {
        try {
            const params = new URLSearchParams();
            params.append('name', name);
            params.append('templateMessaging', templateMessaging.toString());
            params.append('disableOptinPrefUrl', disableOptinPrefUrl.toString());

            const response = await axios.post(
                `${this.baseURL}/app`,
                params,
                {
                    headers: {
                        'token': this.partnerToken,
                        'Content-Type': 'application/x-www-form-urlencoded'
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            console.error('Error creating app:', error.response?.data || error.message);
            throw error;
        }
    }
}

// Usage
const partnerAPI = new GupshupPartnerAPI('your-partner-token');
partnerAPI.createApp('MyWhatsAppApp', true, false)
    .then(result => console.log('App created:', result))
    .catch(error => console.error('Error:', error));
```

#### Response Format

```json
{
    "appId": "bf9ee64c-3d4d-4ac4-xxxx-732e577007c4"
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | `{ "appId": "<app_id>" }` | Success |
| 400 | `{ "status": "error", "message": "Invalid characters used in app name" }` | Special characters in name |
| 400 | `{ "status": "error", "message": "App name should be between 6 to 150 characters in length" }` | Invalid name length |
| 409 | `{ "status": "error", "message": "Bot Already Exists" }` | App name already exists |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |
| 500 | `{ "status": "error", "message": "Unable to create App" }` | Internal server error |

---

## 2. Update Application

### PUT `/partner/app/{appId}`

Update application settings and configuration.

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| templateMessaging | Enable template messaging | Boolean | Optional | Default: false |
| storageRegion | Data storage region | String | Optional | IN, DE, BR, CH, GB, BH, ZA, AE, US, CA, AU, ID, JP, SG, KR |
| disableOptinPrefUrl | Disable optin preferences | Boolean | Optional | Default: false |

#### cURL Example

```bash
curl -X PUT "https://partner.gupshup.io/partner/app/YOUR_APP_ID" \
  -H "token: YOUR_PARTNER_JWT_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "templateMessaging=true" \
  -d "storageRegion=US" \
  -d "disableOptinPrefUrl=false"
```

#### Node.js Example

```javascript
async function updateApp(appId, options = {}) {
    try {
        const params = new URLSearchParams();
        
        if (options.templateMessaging !== undefined) {
            params.append('templateMessaging', options.templateMessaging.toString());
        }
        if (options.storageRegion) {
            params.append('storageRegion', options.storageRegion);
        }
        if (options.disableOptinPrefUrl !== undefined) {
            params.append('disableOptinPrefUrl', options.disableOptinPrefUrl.toString());
        }

        const response = await axios.put(
            `${this.baseURL}/app/${appId}`,
            params,
            {
                headers: {
                    'token': this.partnerToken,
                    'Content-Type': 'application/x-www-form-urlencoded'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error updating app:', error.response?.data || error.message);
        throw error;
    }
}

// Add to GupshupPartnerAPI class
GupshupPartnerAPI.prototype.updateApp = updateApp;
```

#### Response Format

```json
{
    "app": {
        "id": "bf9ee64c-3d4d-4ac4-xxxx-732e577007c4",
        "customerId": 12345,
        "customerType": "PARTNER",
        "billingType": "POSTPAID",
        "createdOn": 1640995200000,
        "phone": "919876543210",
        "callbackUrl": "https://example.com/callback",
        "disableOptinPrefUrl": false,
        "federated": false,
        "live": true,
        "modifiedOn": 1640995200000,
        "name": "MyWhatsAppApp",
        "stopped": false,
        "templateMessaging": true,
        "type": "apicallbackpost",
        "storageRegion": "US",
        "version": 2
    },
    "status": "success"
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | App details object | Success |
| 400 | `{ "status": "error", "message": "Cannot Change Region After Go live" }` | Cannot change region after going live |
| 400 | `{ "status": "error", "message": "Invalid Region Passed" }` | Invalid region code |
| 400 | `{ "status": "error", "message": "Fields to be updated not provided." }` | No fields provided |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |
| 500 | `{ "status": "error", "message": "Unable to update details. Please try again or contact support." }` | Internal server error |

---

## 3. Get Application Details

### GET `/partner/app/{appId}/details`

Fetch detailed information about a specific application.

> **Note:** `callbackURL` and `billingType` in response will be deprecated from February 19, 2025.

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| appId | Application ID | String | Required | Valid app ID from your account |

#### cURL Example

```bash
curl -X GET "https://partner.gupshup.io/partner/app/YOUR_APP_ID/details" \
  -H "token: YOUR_PARTNER_JWT_TOKEN"
```

#### Node.js Example

```javascript
async function getAppDetails(appId) {
    try {
        const response = await axios.get(
            `${this.baseURL}/app/${appId}/details`,
            {
                headers: {
                    'token': this.partnerToken
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error getting app details:', error.response?.data || error.message);
        throw error;
    }
}

// Add to GupshupPartnerAPI class
GupshupPartnerAPI.prototype.getAppDetails = getAppDetails;
```

#### Response Format

```json
{
    "status": "success",
    "appDetails": {
        "billingType": "POSTPAID",
        "createdOn": 1640995200000,
        "callbackUrl": "https://example.com/callback",
        "disableOptinPrefUrl": false,
        "embedAllowed": true,
        "federated": false,
        "id": "bf9ee64c-3d4d-4ac4-xxxx-732e577007c4",
        "languageCode": "en",
        "live": true,
        "liveTs": 1640995200000,
        "modifiedOn": 1640995200000,
        "name": "MyWhatsAppApp",
        "phone": "919876543210",
        "stopped": false,
        "storageRegion": "US",
        "templateMessaging": true,
        "type": "apicallbackpost",
        "version": 2
    },
    "app": "true"
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | App details object | Success |
| 400 | `{ "status":"error", "message/data":"<Specific to API>" }` | Invalid request |
| 401 | `{ "status":"error", "message":"Unauthorized Access" }` | Invalid token |
| 403 | `{ "status":"error", "message":"Unauthorized Access" }` | App doesn't belong to partner |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |

---

## 4. List Applications

### GET `/partner/app/list`

Get a filtered list of all applications for the partner account.

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| name | App name filter | String | Optional | Contains search |
| phone | Phone number filter | String | Optional | Contains search |
| pageNo | Page number | Number | Optional | Default: 1 |
| pageSize | Records per page | Number | Optional | Default: 10 |

#### cURL Example

```bash
curl -X GET "https://partner.gupshup.io/partner/app/list?name=MyApp&phone=9876543210&pageNo=1&pageSize=10" \
  -H "token: YOUR_PARTNER_JWT_TOKEN"
```

#### Node.js Example

```javascript
async function listApps(filters = {}) {
    try {
        const params = new URLSearchParams();
        
        if (filters.name) params.append('name', filters.name);
        if (filters.phone) params.append('phone', filters.phone);
        if (filters.pageNo) params.append('pageNo', filters.pageNo.toString());
        if (filters.pageSize) params.append('pageSize', filters.pageSize.toString());

        const response = await axios.get(
            `${this.baseURL}/app/list?${params.toString()}`,
            {
                headers: {
                    'token': this.partnerToken
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error listing apps:', error.response?.data || error.message);
        throw error;
    }
}

// Add to GupshupPartnerAPI class
GupshupPartnerAPI.prototype.listApps = listApps;
```

#### Response Format

```json
{
    "status": "success",
    "data": [
        {
            "id": "bf9ee64c-3d4d-4ac4-xxxx-732e577007c4",
            "name": "MyWhatsAppApp",
            "type": "apicallbackpost",
            "live": true,
            "phone": "919876543210",
            "stopped": false,
            "storageRegion": "US",
            "uiFormStage": "COMPLETED",
            "pipelineStage": "LIVE",
            "creationStage": "COMPLETED",
            "whatsappVerificationStatus": "VERIFIED",
            "version": 2,
            "products": ["ANALYTICS", "CUSTOMER_SUPPORT"],
            "createdOn": 1640995200000,
            "modifiedOn": 1640995200000
        }
    ]
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | Array of app objects | Success |
| 400 | `{ "status":"error", "message/data":"<Specific to API>" }` | Invalid request |
| 401 | `{ "status":"error", "message":"Unauthorized Access" }` | Invalid token |
| 403 | `{ "status":"error", "message":"Unauthorized Access" }` | Unauthorized access |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |

---

## 5. Set Contact Details

### PUT `/partner/app/{appId}/onboarding/contact`

Set business contact details for the application.

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| appId | Application ID | String | Required | Valid app ID |
| contactEmail | Business contact email | String | Required | Valid email address |
| contactName | Business contact name | String | Required | Valid name |
| contactNumber | Business contact phone | String | Required | Valid phone number |

#### cURL Example

```bash
curl -X PUT "https://partner.gupshup.io/partner/app/YOUR_APP_ID/onboarding/contact" \
  -H "token: YOUR_PARTNER_JWT_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "contactEmail=business@example.com" \
  -d "contactName=John Doe" \
  -d "contactNumber=+1234567890"
```

#### Node.js Example

```javascript
async function setContactDetails(appId, contactDetails) {
    try {
        const params = new URLSearchParams();
        params.append('contactEmail', contactDetails.email);
        params.append('contactName', contactDetails.name);
        params.append('contactNumber', contactDetails.phone);

        const response = await axios.put(
            `${this.baseURL}/app/${appId}/onboarding/contact`,
            params,
            {
                headers: {
                    'token': this.partnerToken,
                    'Content-Type': 'application/x-www-form-urlencoded'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error setting contact details:', error.response?.data || error.message);
        throw error;
    }
}

// Add to GupshupPartnerAPI class
GupshupPartnerAPI.prototype.setContactDetails = setContactDetails;
```

#### Response Format

```json
{
    "status": "success",
    "message": "contact details updated successfully"
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | `{ "status": "success", "message": "contact details updated successfully" }` | Success |
| 400 | `{ "status": "error", "message": "Invalid app id provided" }` | Invalid app ID |
| 400 | `{ "status": "error", "message": "Please provide valid details" }` | Invalid contact details |
| 401 | `{ "status": "error", "message": "Authentication Failed" }` | Invalid token or app ID |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |

---

## 6. Resend Email Verification

### POST `/partner/app/{appId}/onboarding/contact/email/resend`

Resend email verification link to the business contact email.

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| appId | Application ID | String | Required | Valid app ID |

#### cURL Example

```bash
curl -X POST "https://partner.gupshup.io/partner/app/YOUR_APP_ID/onboarding/contact/email/resend" \
  -H "token: YOUR_PARTNER_JWT_TOKEN"
```

#### Node.js Example

```javascript
async function resendEmailVerification(appId) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${appId}/onboarding/contact/email/resend`,
            {},
            {
                headers: {
                    'token': this.partnerToken
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error resending email verification:', error.response?.data || error.message);
        throw error;
    }
}

// Add to GupshupPartnerAPI class
GupshupPartnerAPI.prototype.resendEmailVerification = resendEmailVerification;
```

#### Response Format

```json
{
    "status": "success",
    "message": "mail sent successfully"
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | `{ "status": "success", "message": "mail sent successfully" }` | Success |
| 400 | `{ "status": "error", "message": "Email is already Verified For This Container" }` | Email already verified |
| 400 | `{ "status": "error", "message": "Please set business email first" }` | Business email not set |
| 401 | `{ "status": "error", "message": "Authentication Failed" }` | Invalid token or app ID |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |

---

## 7. Generate Embed Link

### GET `/partner/app/{appId}/onboarding/embed/link`

Generate a signed embed link for the application onboarding process. This link opens Gupshup's embedded onboarding UI where clients complete the entire setup process including Facebook Business Manager connection and WhatsApp number configuration.

> **Important:** 
> - The signed link is valid for 5 days only
> - All onboarding happens in Gupshup's popup UI - no forms needed in your application
> - Clients handle Facebook Business Manager integration directly through the popup
> - Phone number selection and WABA setup is completed by the client in the embedded interface

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| appId | Application ID | String | Required | Valid app ID |
| regenerate | Regenerate link | Boolean | Optional | Default: false |
| user | Username | String | Required | Valid username |
| lang | Language | String | Required | Supported language code |

#### cURL Example

```bash
curl -X GET "https://partner.gupshup.io/partner/app/YOUR_APP_ID/onboarding/embed/link?regenerate=true&user=john_doe&lang=en" \
  -H "token: YOUR_PARTNER_JWT_TOKEN"
```

#### Node.js Example

```javascript
async function generateEmbedLink(appId, user, lang, regenerate = false) {
    try {
        const params = new URLSearchParams();
        params.append('regenerate', regenerate.toString());
        params.append('user', user);
        params.append('lang', lang);

        const response = await axios.get(
            `${this.baseURL}/app/${appId}/onboarding/embed/link?${params.toString()}`,
            {
                headers: {
                    'token': this.partnerToken
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error generating embed link:', error.response?.data || error.message);
        throw error;
    }
}

// Add to GupshupPartnerAPI class
GupshupPartnerAPI.prototype.generateEmbedLink = generateEmbedLink;
```

#### Response Format

```json
{
    "status": "success",
    "link": "https://partner.gupshup.io/embed/onboarding?token=eyJhbGciOiJIUzI1NiJ9..."
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | `{ "status": "success", "link": "<embed_link>" }` | Success |
| 400 | `{ "status": "error", "message": "Query Params cannot be empty" }` | Missing required parameters |
| 400 | `{ "status": "error", "message": "Max Links Already sent, please contact support" }` | Too many links generated |
| 400 | `{ "status": "error", "message": "Link has expired, please regenerate link" }` | Link expired |
| 401 | `{ "status": "error", "message": "Authentication Failed" }` | Invalid token or app ID |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |
| 500 | `{ "status": "error", "message": "Unable to create link" }` | Internal server error |

---

## 8. Mark App for Migration

### POST `/partner/app/{appId}/onboarding/phoneMigration`

Mark an application as migrated from another BSP to Gupshup.

#### Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| appId | Application ID | String | Required | Valid app ID |
| migrationStatus | Migration status | String | Required | META_EMBED_MIGRATION or MIGRATED_IN |

#### cURL Example

```bash
curl -X POST "https://partner.gupshup.io/partner/app/YOUR_APP_ID/onboarding/phoneMigration" \
  -H "token: YOUR_PARTNER_JWT_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "migrationStatus=META_EMBED_MIGRATION"
```

#### Node.js Example

```javascript
async function markAppForMigration(appId, migrationStatus) {
    try {
        const params = new URLSearchParams();
        params.append('migrationStatus', migrationStatus);

        const response = await axios.post(
            `${this.baseURL}/app/${appId}/onboarding/phoneMigration`,
            params,
            {
                headers: {
                    'token': this.partnerToken,
                    'Content-Type': 'application/x-www-form-urlencoded'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error marking app for migration:', error.response?.data || error.message);
        throw error;
    }
}

// Add to GupshupPartnerAPI class
GupshupPartnerAPI.prototype.markAppForMigration = markAppForMigration;
```

#### Response Format

```json
{
    "status": "success",
    "whatsapp": {
        "businessId": "1234567890",
        "countryCode": 91,
        "createdOn": 1713430478907,
        "creationStage": "DOCKER_PROVISIONING",
        "dialCode": "91",
        "displayName": "MyWhatsAppApp",
        "embedLiveCheckTimestamp": 0,
        "embedStage": "EMBED_PHONE_SELECTION",
        "fbBusinessId": "1234567890123456",
        "fbVerificationBypassed": false,
        "formattedPhoneNumber": "+91 98765 43210",
        "goLiveProcessRequestEmailSent": false,
        "id": "bf9ee64c-3d4d-4ac4-xxxx-732e577007c4",
        "lastPNDNCheckTimestamp": 0,
        "lastWABACheckTimestamp": 0,
        "migrationStatus": "META_EMBED_MIGRATION",
        "migrationTime": 1719297738982,
        "modifiedOn": 1719297738982,
        "normalizedPhoneNumber": "919876543210",
        "pipeLineStage": "GET_CERTIFICATE",
        "pndnApprovalTimestamp": 0,
        "preferredCurrency": "INR",
        "preferredLocale": "en_US",
        "preferredTimezone": "71",
        "providedAccessToGupshup": false,
        "regionCode": "IN",
        "stageRetries": 0,
        "uiFormStage": "GO_LIVE_FLOW_SELECTED",
        "wabaApprovalTimestamp": 0,
        "whatsappVerificationStatus": "WABA_APPROVED"
    }
}
```

#### Status Codes

| Code | Response | Description |
|------|----------|-------------|
| 200 | WhatsApp details object | Success |
| 400 | `{ "status": "error", "message": "Application not found" }` | Invalid app ID |
| 400 | `{ "status": "error", "message": "Please provide valid details" }` | Invalid migration status |
| 401 | `{ "status": "error", "message": "Authentication Failed" }` | Invalid token or app ID |
| 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |

---

## Complete Node.js SDK Class

```javascript
const axios = require('axios');

class GupshupPartnerAPI {
    constructor(partnerToken) {
        this.partnerToken = partnerToken;
        this.baseURL = 'https://partner.gupshup.io/partner';
    }

    // Create a new application
    async createApp(name, templateMessaging = false, disableOptinPrefUrl = false) {
        try {
            const params = new URLSearchParams();
            params.append('name', name);
            params.append('templateMessaging', templateMessaging.toString());
            params.append('disableOptinPrefUrl', disableOptinPrefUrl.toString());

            const response = await axios.post(
                `${this.baseURL}/app`,
                params,
                {
                    headers: {
                        'token': this.partnerToken,
                        'Content-Type': 'application/x-www-form-urlencoded'
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // Update application settings
    async updateApp(appId, options = {}) {
        try {
            const params = new URLSearchParams();
            
            if (options.templateMessaging !== undefined) {
                params.append('templateMessaging', options.templateMessaging.toString());
            }
            if (options.storageRegion) {
                params.append('storageRegion', options.storageRegion);
            }
            if (options.disableOptinPrefUrl !== undefined) {
                params.append('disableOptinPrefUrl', options.disableOptinPrefUrl.toString());
            }

            const response = await axios.put(
                `${this.baseURL}/app/${appId}`,
                params,
                {
                    headers: {
                        'token': this.partnerToken,
                        'Content-Type': 'application/x-www-form-urlencoded'
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // Get application details
    async getAppDetails(appId) {
        try {
            const response = await axios.get(
                `${this.baseURL}/app/${appId}/details`,
                {
                    headers: {
                        'token': this.partnerToken
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // List applications with filtering
    async listApps(filters = {}) {
        try {
            const params = new URLSearchParams();
            
            if (filters.name) params.append('name', filters.name);
            if (filters.phone) params.append('phone', filters.phone);
            if (filters.pageNo) params.append('pageNo', filters.pageNo.toString());
            if (filters.pageSize) params.append('pageSize', filters.pageSize.toString());

            const response = await axios.get(
                `${this.baseURL}/app/list?${params.toString()}`,
                {
                    headers: {
                        'token': this.partnerToken
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // Set business contact details
    async setContactDetails(appId, contactDetails) {
        try {
            const params = new URLSearchParams();
            params.append('contactEmail', contactDetails.email);
            params.append('contactName', contactDetails.name);
            params.append('contactNumber', contactDetails.phone);

            const response = await axios.put(
                `${this.baseURL}/app/${appId}/onboarding/contact`,
                params,
                {
                    headers: {
                        'token': this.partnerToken,
                        'Content-Type': 'application/x-www-form-urlencoded'
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // Resend email verification
    async resendEmailVerification(appId) {
        try {
            const response = await axios.post(
                `${this.baseURL}/app/${appId}/onboarding/contact/email/resend`,
                {},
                {
                    headers: {
                        'token': this.partnerToken
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // Generate embed link
    async generateEmbedLink(appId, user, lang, regenerate = false) {
        try {
            const params = new URLSearchParams();
            params.append('regenerate', regenerate.toString());
            params.append('user', user);
            params.append('lang', lang);

            const response = await axios.get(
                `${this.baseURL}/app/${appId}/onboarding/embed/link?${params.toString()}`,
                {
                    headers: {
                        'token': this.partnerToken
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // Mark app for migration
    async markAppForMigration(appId, migrationStatus) {
        try {
            const params = new URLSearchParams();
            params.append('migrationStatus', migrationStatus);

            const response = await axios.post(
                `${this.baseURL}/app/${appId}/onboarding/phoneMigration`,
                params,
                {
                    headers: {
                        'token': this.partnerToken,
                        'Content-Type': 'application/x-www-form-urlencoded'
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    // Error handler
    handleError(error) {
        if (error.response) {
            const { status, data } = error.response;
            const errorMessage = data.message || data.data || 'Unknown error';
            
            switch (status) {
                case 400:
                    return new Error(`Bad Request: ${errorMessage}`);
                case 401:
                    return new Error(`Unauthorized: ${errorMessage}`);
                case 403:
                    return new Error(`Forbidden: ${errorMessage}`);
                case 409:
                    return new Error(`Conflict: ${errorMessage}`);
                case 429:
                    return new Error(`Rate Limit Exceeded: ${errorMessage}`);
                case 500:
                    return new Error(`Internal Server Error: ${errorMessage}`);
                default:
                    return new Error(`HTTP ${status}: ${errorMessage}`);
            }
        } else if (error.request) {
            return new Error('Network error - No response received');
        } else {
            return new Error(`Request error: ${error.message}`);
        }
    }
}

// Usage Examples
const partnerAPI = new GupshupPartnerAPI('your-partner-token');

async function onboardingExample() {
    try {
        // Create a new app
        const newApp = await partnerAPI.createApp('MyBusinessApp', true, false);
        console.log('App created:', newApp);

        const appId = newApp.appId;

        // Update app settings
        await partnerAPI.updateApp(appId, {
            templateMessaging: true,
            storageRegion: 'US'
        });

        // Set contact details
        await partnerAPI.setContactDetails(appId, {
            email: 'business@example.com',
            name: 'John Doe',
            phone: '+1234567890'
        });

        // Generate embed link
        const embedLink = await partnerAPI.generateEmbedLink(appId, 'admin', 'en', false);
        console.log('Embed link:', embedLink);

        // Get app details
        const appDetails = await partnerAPI.getAppDetails(appId);
        console.log('App details:', appDetails);

        // List all apps
        const appsList = await partnerAPI.listApps({ pageSize: 20 });
        console.log('Apps list:', appsList);

        console.log('Onboarding process completed successfully!');
    } catch (error) {
        console.error('Error in onboarding:', error.message);
    }
}

// Export the class
module.exports = GupshupPartnerAPI;
```

## Storage Regions

| Code | Region |
|------|--------|
| IN | India |
| DE | Germany |
| BR | Brazil |
| CH | Switzerland |
| GB | Great Britain |
| BH | Bahrain |
| ZA | South Africa |
| AE | United Arab Emirates |
| US | United States |
| CA | Canada |
| AU | Australia |
| ID | Indonesia |
| JP | Japan |
| SG | Singapore |
| KR | South Korea |

## Migration Status Values

| Status | Description |
|--------|-------------|
| META_EMBED_MIGRATION | App is yet to be made live |
| MIGRATED_IN | App is already live |

## Best Practices

### 1. Error Handling
- Always implement comprehensive error handling
- Use exponential backoff for rate-limited requests
- Log all API interactions for debugging

### 2. Security
- Never expose partner tokens in client-side code
- Validate all input parameters before sending requests
- Use HTTPS for all API communications

### 3. Rate Limiting
- Implement proper rate limiting in your applications
- Use queuing systems for bulk operations
- Monitor API usage to avoid hitting limits

### 4. Data Management
- Store app IDs securely for future reference
- Implement proper data validation
- Use appropriate storage regions for compliance

### 5. Monitoring
- Set up monitoring for API health and performance
- Track onboarding completion rates
- Monitor error patterns and resolution times

## Common Error Patterns

### Authentication Errors
- Always verify token validity before making requests
- Handle token expiration gracefully
- Implement token refresh mechanisms

### Validation Errors
- Validate all input parameters client-side
- Provide clear error messages to users
- Handle edge cases appropriately

### Rate Limit Handling
- Implement exponential backoff
- Queue requests during high-traffic periods
- Monitor and alert on rate limit approaches

This comprehensive documentation provides everything needed to integrate with the Gupshup Partner API onboarding system, including complete code examples, error handling, and best practices for production use.

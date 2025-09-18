# Gupshup Token Management API Documentation

## Get Partner Token

**Endpoint:** `POST https://partner.gupshup.io/partner/account/login`

**Description:** Use this API to get the partner token for your Gupshup App

The partner token will be used as authorization in other Partner APIs.
Currently, the expiry for the token is 24 hours.

### Request Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Email | {{EMAIL}} | Email address through partner has signed up on partner portal |
| Password | {{PASSWORD}} | Password set by the partner on the partner portal |

### Sample Request

**cURL:**
```bash
curl --location --request POST 'https://partner.gupshup.io/partner/account/login' \
--header 'Accept: application/json' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'email={{EMAIL}}' \
--data-urlencode 'password={{PASSWORD}}'
```

**Node.js:**
```javascript
const axios = require('axios');

const getPartnerToken = async (email, password) => {
  try {
    const response = await axios.post('https://partner.gupshup.io/partner/account/login', 
      new URLSearchParams({
        email: email,
        password: password
      }), 
      {
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    return response.data;
  } catch (error) {
    console.error('Error getting partner token:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const tokenData = await getPartnerToken('your-email@example.com', 'your-password');
```

### Sample Response

```json
{
  "token": "{{PARTNER_TOKEN}}",
  "id": 1**3,
  "name": "gupshup",
  "terms_read": true,
  "admin": true,
  "email": "peter.parker@zylker.com",
  "activationRead": false,
  "billingType": "PREPAID",
  "contactName": "peter",
  "phoneNumber": "898",
  "enableCustomer": false,
  "enableWallet": true,
  "enableInrWallet": false,
  "enableLoaderWallet": false,
  "enableAppOnboarding": true,
  "isTpp": false,
  "onboardEnabled": false
}
```

### Response Fields Description

- `token`: JWT token to be used in other Partner APIs
- `id`: Partner ID (partially masked)
- `name`: Partner name
- `terms_read`: Boolean indicating if terms have been read
- `admin`: Boolean indicating admin privileges
- `email`: Partner email address
- `activationRead`: Boolean indicating if activation has been read
- `billingType`: Type of billing (e.g., "PREPAID")
- `contactName`: Contact person name
- `phoneNumber`: Phone number
- `enableCustomer`: Boolean for customer enablement
- `enableWallet`: Boolean for wallet enablement
- `enableInrWallet`: Boolean for INR wallet enablement
- `enableLoaderWallet`: Boolean for loader wallet enablement
- `enableAppOnboarding`: Boolean for app onboarding enablement
- `isTpp`: Boolean indicating if it's a third-party provider
- `onboardEnabled`: Boolean for onboard enablement

---

## Get Access Token for an App

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/token`

**Description:** Use this API to get the access token for your Gupshup App

You can use this token to get App's templates, submit templates, send messages etc. You will need below details to start using this API:

1. Partner token

### Request Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Authorization | {{PARTNER_TOKEN}} | JWT Token issued post Partner login |
| App ID | {{APP_ID}} | Unique Identifier for Gupshup App. The ID should be a valid App ID of Gupshup The App must be associated with the account that owns the PARTNER_APP_TOKEN being utilized. |

### Sample Request

**cURL:**
```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/token/' \
--header 'Authorization: {{PARTNER_TOKEN}}'
```

**Node.js:**
```javascript
const axios = require('axios');

const getAppAccessToken = async (appId, partnerToken) => {
  try {
    const response = await axios.get(`https://partner.gupshup.io/partner/app/${appId}/token/`, {
      headers: {
        'Authorization': partnerToken
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Error getting app access token:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const tokenData = await getAppAccessToken('your-app-id', 'your-partner-token');
```

### Sample Response

```json
{
    "status": "success",
    "token": {
        "token": "sk_20c901e*********2a592484********1",
        "authoriserId": "aac***e8-d**c-****-a93f-**d6d5a***e8",
        "requestorId": "4**2",
        "createdOn": 1705905891100,
        "modifiedOn": 1705905891100,
        "expiresOn": 0,
        "active": true
    }
}
```

### Response Fields Description

- `status`: Status of the request ("success")
- `token`: Token object containing:
  - `token`: The access token (partially masked for security)
  - `authoriserId`: ID of the authorizer (partially masked)
  - `requestorId`: ID of the requestor (partially masked)
  - `createdOn`: Timestamp of when the token was created
  - `modifiedOn`: Timestamp of when the token was last modified
  - `expiresOn`: Expiration timestamp (0 means no expiry)
  - `active`: Boolean indicating if the token is active

### Notes

- The Partner token is required for authentication
- The App ID must be a valid Gupshup App ID
- The App must be associated with the account that owns the Partner token
- The access token can be used for various app operations like template management and messaging
- Token expiry is controlled by the `expiresOn` field (0 means no expiry)
# Gupshup Partner Apps API Documentation

## Get Partner Apps

**Endpoint:** `GET https://partner.gupshup.io/partner/account/api/partnerApps`

**Description:** Use this API to fetch the list of Apps which are linked to your partner account

### Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Authorization | {{PARTNER_TOKEN}} | JWT Token issued post Partner login. |

### Sample Request

**cURL:**
```bash
curl --location --request GET 'https://partner.gupshup.io/partner/account/api/partnerApps' \
--header 'Authorization: {{PARTNER_TOKEN}}'
```

**Node.js:**
```javascript
const axios = require('axios');

const getPartnerApps = async (partnerToken) => {
  try {
    const response = await axios.get('https://partner.gupshup.io/partner/account/api/partnerApps', {
      headers: {
        'Authorization': partnerToken
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Error fetching partner apps:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const apps = await getPartnerApps('your-partner-token');
```

### Sample Response

```json
{
    "status": "success",
    "partnerAppsList": [
        {
            "id": "00065097-93ff-4c22-bfa4-845f01b7de3b",
            "name": "BotTch20",
            "customerId": "4000001311",
            "live": false,
            "partnerId": 6,
            "createdOn": 1624339843605,
            "modifiedOn": 1651130376253,
            "partnerCreated": false,
            "cxpEnabled": false,
            "partnerUsage": true, 
            "stopped": false,
            "healthy": true,
            "cap": 0.0
        },
        {
            "id": "018358b5-5ac1-44af-bbfe-36ac4e9f01cc",
            "name": "automation030542",
            "customerId": "NA",
            "live": false,
            "partnerId": 6,
            "createdOn": 1703857396150,
            "modifiedOn": 1703857396150,
            "partnerCreated": false,
            "cxpEnabled": false,
            "partnerUsage": true,
            "stopped": false,
            "healthy": true,
            "cap": 0.0
        },
        {
            "id": "02f9e530-f7dd-4a04-ad33-4406d29c9e56",
            "name": "TestAPI70",
            "customerId": "4000001585",
            "live": false,
            "partnerId": 6,
            "createdOn": 1651738868991,
            "modifiedOn": 1651738868991,
            "partnerCreated": false,
            "cxpEnabled": false,
            "partnerUsage": true, 
            "stopped": false,
            "healthy": true,
            "cap": 0.0
        },
        {
            "id": "04e6c66b-bb95-4bfc-b75e-4e7391ddc680",
            "name": "GupShupQAHSM",
            "phone": "919324927406",
            "customerId": "4000001343",
            "live": false,
            "partnerId": 6,
            "createdOn": 1690874640468,
            "modifiedOn": 1698409662019,
            "partnerCreated": false,
            "cxpEnabled": false,
            "partnerUsage": true, 
            "stopped": false,
            "healthy": true,
            "cap": 0.0
        },
        {
            "id": "0adef57e-b1fe-41eb-94e7-c47e29fbc50f",
            "name": "automation9525531",
            "customerId": "NA",
            "live": false,
            "partnerId": 6,
            "createdOn": 1703856050646,
            "modifiedOn": 1703856050646,
            "partnerCreated": false,
            "cxpEnabled": false,
            "partnerUsage": true, 
            "stopped": false,
            "healthy": true,
            "cap": 0.0
        }
    ]
}
```

### Response Fields Description

Each partner app object in the `partnerAppsList` array contains:

- `id`: Unique identifier for the app
- `name`: Name of the app
- `customerId`: Customer ID associated with the app (can be "NA" if not assigned)
- `live`: Boolean indicating if the app is live
- `partnerId`: Partner identifier
- `createdOn`: Timestamp of when the app was created
- `modifiedOn`: Timestamp of when the app was last modified
- `partnerCreated`: Boolean indicating if the app was created by partner
- `cxpEnabled`: Boolean indicating if CXP is enabled
- `partnerUsage`: Boolean indicating if partner usage is enabled
- `stopped`: Boolean indicating if the app is stopped
- `healthy`: Boolean indicating if the app is healthy
- `cap`: Capacity limit (numeric value)
- `phone`: Phone number (optional field, present for some apps)

### Notes

- Authentication is required using a Partner JWT token
- The response includes all apps linked to the partner account
- Apps can have different statuses (live/stopped, healthy/unhealthy)
- Some apps may have additional fields like phone numbers

---

## Link App with Partner

**Endpoint:** `POST https://partner.gupshup.io/partner/account/api/appLink`

**Description:** Use this API to map your Gupshup App with a Partner

### Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Authorization | {{PARTNER_TOKEN}} | JWT Token issued post Partner login |
| apiKey | {{API_KEY}} | API key for the app |
| appName | {{APP_NAME}} | Unique Identifier for Gupshup App |

### Sample Request

**cURL:**
```bash
curl --location --request POST 'https://partner.gupshup.io/partner/account/api/appLink' \
--header 'Authorization: {{PARTNER_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'apiKey={{API_KEY}}' \
--data-urlencode 'appName={{APP_NAME}}'
```

**Node.js:**
```javascript
const axios = require('axios');

const linkAppWithPartner = async (apiKey, appName, partnerToken) => {
  try {
    const response = await axios.post('https://partner.gupshup.io/partner/account/api/appLink',
      new URLSearchParams({
        apiKey: apiKey,
        appName: appName
      }),
      {
        headers: {
          'Authorization': partnerToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    return response.data;
  } catch (error) {
    console.error('Error linking app with partner:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const linkResult = await linkAppWithPartner('your-api-key', 'your-app-name', 'your-partner-token');
```

### Sample Response

```json
{
    "partnerApps": {
        "createdOn": 1609829592910,
        "healthy": false,
        "id": "fa*b***e-d2**-****-be38-0****d*9a**b",
        "live": false,
        "modifiedOn": 1614162854108,
        "name": "assistant0092",
        "partnerId": 1,
        "phone": "91**********",
        "stopped": false,
        "walletId": "1**"
    }
}
```

### Response Fields Description

The `partnerApps` object contains:

- `createdOn`: Timestamp of when the app was created
- `healthy`: Boolean indicating if the app is healthy
- `id`: Unique identifier for the app (partially masked in response)
- `live`: Boolean indicating if the app is live
- `modifiedOn`: Timestamp of when the app was last modified
- `name`: Name of the app
- `partnerId`: Partner identifier
- `phone`: Phone number associated with the app (partially masked)
- `stopped`: Boolean indicating if the app is stopped
- `walletId`: Wallet identifier (partially masked)

### Notes

- This API requires both the Partner JWT token and the app's API key
- The request uses `application/x-www-form-urlencoded` content type
- The response includes masked sensitive information for security
- Successfully linking an app will return the app details with partner association
# Gupshup Analytics API Documentation

This document covers the analytics and usage reporting APIs for Gupshup Partner applications. These APIs allow you to monitor app usage, billing, and discount information.

---

## Get App's Daily Usage

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/usage`

**Description:** Use this API to get the daily usage breakdown for a particular Gupshup app ranging between two dates.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |
| from | {{FROM_DATE}} | Date in YYYY-MM-DD format | String | Required |
| to | {{TO_DATE}} | Date in YYYY-MM-DD format | String | Required |

### Sample Request

**cURL:**
```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/usage?from={{FROM_DATE}}&to={{TO_DATE}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

**Node.js:**
```javascript
const axios = require('axios');

const getAppUsage = async (appId, fromDate, toDate, token) => {
  try {
    const response = await axios.get(`https://partner.gupshup.io/partner/app/${appId}/usage`, {
      headers: {
        'Authorization': token
      },
      params: {
        from: fromDate,
        to: toDate
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Error fetching app usage:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const usage = await getAppUsage('your-app-id', '2024-01-01', '2024-01-31', 'your-partner-app-token');
```

### Sample Response

```json
{
    "status": "success",
    "partnerAppUsageList": [
        {
            "appId": "bf9ee64c-3d4d-4ac4-8668-732e577007c4",
            "appName": "Jan10pass",
            "authentication": 0,
            "cumulativeBill": 0.007,
            "currency": "USD",
            "date": "2024-01-10",
            "discount": 0.0,
            "fep": 0,
            "ftc": 1,
            "gsCap": 75.0,
            "gsFees": 0.007,
            "incomingMsg": 2,
            "outgoingMsg": 4,
            "outgoingMediaMsg": 0,
            "marketing": 0,
            "service": 0,
            "templateMsg": 0,
            "templateMediaMsg": 1,
            "totalFees": 0.007,
            "totalMsg": 7,
            "utility": 0,
            "waFees": 0.0,
            "internationalAuthentication": 1
        },
        {
            "appId": "bf9ee64c-3d4d-4ac4-8668-732e577007c4",
            "appName": "Jan10pass",
            "authentication": 0,
            "cumulativeBill": 0.234,
            "currency": "USD",
            "date": "2024-01-18",
            "discount": 0.0,
            "fep": 0,
            "ftc": 1,
            "gsCap": 75.0,
            "gsFees": 0.091,
            "incomingMsg": 9,
            "outgoingMsg": 82,
            "outgoingMediaMsg": 0,
            "marketing": 1,
            "service": 0,
            "templateMsg": 0,
            "templateMediaMsg": 0,
            "totalFees": 0.101,
            "totalMsg": 91,
            "utility": 0,
            "waFees": 0.01,
            "internationalAuthentication": 1
        }
    ]
}
```

### Response Fields Description

Each usage record in the `partnerAppUsageList` array contains:

#### App Information
- `appId`: Unique identifier for the app
- `appName`: Name of the app
- `date`: Date of the usage record (YYYY-MM-DD format)
- `currency`: Currency used for billing (e.g., "USD")

#### Message Counts
- `incomingMsg`: Number of incoming messages
- `outgoingMsg`: Number of outgoing messages
- `outgoingMediaMsg`: Number of outgoing media messages
- `totalMsg`: Total number of messages
- `templateMsg`: Number of template messages
- `templateMediaMsg`: Number of template media messages

#### Message Categories
- `authentication`: Number of authentication messages
- `marketing`: Number of marketing messages
- `service`: Number of service messages
- `utility`: Number of utility messages
- `internationalAuthentication`: Number of international authentication messages

#### Billing Information
- `cumulativeBill`: Cumulative bill amount
- `totalFees`: Total fees for the day
- `gsFees`: Gupshup fees
- `waFees`: WhatsApp fees
- `discount`: Discount amount
- `gsCap`: Gupshup capacity limit

#### Other Fields
- `fep`: Free Entry Point messages
- `ftc`: Free Tier Conversation messages

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "partnerAppUsageList": [...] }` | - |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Internal server error. Please try again later and If Issue still persist than contact Gupshup Dev Support" }` | For any Internal Error |

---

## Get App's Daily Discount

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/discount`

**Description:** Use this API to get the daily discount, daily bill, and the cumulative bill for a Gupshup app for a specific month.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |
| year | {{YEAR}} | Year in YYYY format | String | Required |
| month | {{MONTH}} | Month in MM format | String | Required |

### Sample Request

**cURL:**
```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/discount?year={{YEAR}}&month={{MONTH}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

**Node.js:**
```javascript
const axios = require('axios');

const getAppDiscount = async (appId, year, month, token) => {
  try {
    const response = await axios.get(`https://partner.gupshup.io/partner/app/${appId}/discount`, {
      headers: {
        'Authorization': token
      },
      params: {
        year: year,
        month: month
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Error fetching app discount:', error.response?.data || error.message);
    throw error;
  }
};

// Usage example
const discount = await getAppDiscount('your-app-id', '2022', '04', 'your-partner-app-token');
```

### Sample Response

```json
{
    "dailyAppDiscountList": [
        {
            "appId": "8*****j7-a**3-4**d-8***-c2******7bf**3",
            "cumulativeBill": 0,
            "dailyBill": 0,
            "day": 2,
            "discount": 0,
            "gsCap": 75,
            "gsFees": 0,
            "month": 4,
            "partnerId": 15,
            "year": 2022
        },
        {
            "appId": "8*****j7-a**3-4**d-8***-c2******7bf**3",
            "cumulativeBill": 0.014,
            "dailyBill": 0.014,
            "day": 4,
            "discount": 0,
            "gsCap": 75,
            "gsFees": 0.014,
            "month": 4,
            "partnerId": 15,
            "year": 2022
        }
    ]
}
```

### Response Fields Description

Each discount record in the `dailyAppDiscountList` array contains:

#### App Information
- `appId`: Unique identifier for the app (partially masked for security)
- `partnerId`: Partner identifier

#### Date Information
- `year`: Year of the record
- `month`: Month of the record
- `day`: Day of the record

#### Billing Information
- `dailyBill`: Daily bill amount
- `cumulativeBill`: Cumulative bill amount up to this day
- `discount`: Discount amount applied
- `gsFees`: Gupshup fees
- `gsCap`: Gupshup capacity limit

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "dailyAppDiscountList": [...] }` | - |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |
| Error | 500 | `{ "status": "error", "message": "Internal server error" }` | For any Internal Error |

---

## Usage Examples

### Get Usage for a Date Range

**cURL:**
```bash
# Get usage for January 2024
curl --location 'https://partner.gupshup.io/partner/app/your-app-id/usage?from=2024-01-01&to=2024-01-31' \
--header 'Authorization: Bearer your-partner-app-token'
```

**Node.js:**
```javascript
// Get usage for January 2024
const januaryUsage = await getAppUsage('your-app-id', '2024-01-01', '2024-01-31', 'your-partner-app-token');
console.log('January Usage:', januaryUsage);
```

### Get Discount Information for a Month

**cURL:**
```bash
# Get discount information for April 2022
curl --location 'https://partner.gupshup.io/partner/app/your-app-id/discount?year=2022&month=04' \
--header 'Authorization: Bearer your-partner-app-token'
```

**Node.js:**
```javascript
// Get discount information for April 2022
const aprilDiscount = await getAppDiscount('your-app-id', '2022', '04', 'your-partner-app-token');
console.log('April Discount:', aprilDiscount);
```

---

## Important Notes

### General

- All APIs require Partner App Access Token for authentication
- Both APIs are read-only and provide historical data
- Rate limiting applies to both endpoints (10 requests per minute for usage API)

### Date Formats

- **Usage API**: Use YYYY-MM-DD format for `from` and `to` parameters
- **Discount API**: Use YYYY format for `year` and MM format for `month` parameters

### Data Availability

- Usage data is typically available with a delay of a few hours
- Discount data is calculated daily and available for completed days
- Historical data retention may vary based on your subscription

### Billing Information

- All monetary values are in the currency specified in the response
- Cumulative bills represent running totals for the specified period
- Discounts are applied automatically and reflected in the totals

### Message Categories

- **Authentication**: Messages used for user verification
- **Marketing**: Promotional messages sent to users
- **Service**: Customer service related messages
- **Utility**: Transactional messages (receipts, updates, etc.)
- **Template messages**: Pre-approved message templates
- **Media messages**: Messages containing images, videos, or documents

### Best Practices

1. **Monitor Usage Regularly**: Check usage patterns to optimize costs
2. **Set Up Alerts**: Monitor cumulative bills to avoid unexpected charges
3. **Analyze Message Types**: Understand which message categories drive costs
4. **Use Date Ranges Wisely**: Request only the data you need to minimize API calls
5. **Cache Results**: Store frequently accessed data to reduce API calls
6. **Handle Rate Limits**: Implement proper retry logic for rate-limited requests

# Gupshup Template Analytics API Documentation

This document covers the Template Analytics APIs for Gupshup Partner applications. These APIs allow you to enable template analytics, retrieve analytics data, and manage button click analytics for your templates.

> **📘 Note:** All analytics results are returned in IST (Indian Standard Time).

---

## Get Template Analytics

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/template/analytics`

**Description:** Use this API to get template analytics for a template.

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | Should be a valid appId |
| start | {{START}} | Start time of the query | Long | Optional | Epoch timestamp. If both start and end are not provided, default time corresponding to the last 30 days is populated |
| end | {{END}} | End time of the query | Long | Optional | Epoch timestamp. If both start and end are not provided, default time corresponding to the last 30 days is populated |
| granularity | {{GRANULARITY}} | Granularity of template analytics | String | Optional | DAILY or AGGREGATED. If not provided, AGGREGATED is taken |
| metric_types | {{METRIC_TYPES}} | Comma-separated string for metrics to fetch | String | Optional | SENT, DELIVERED, READ, CLICKED (all or some of the mentioned values). If not provided, all metrics are considered |
| template_ids | {{TEMPLATE_IDS}} | Comma-separated string for GS template IDs | String | Required | Currently supports only 1 template at a time |
| limit | {{LIMIT}} | Number of permitted entries in response per page | Integer | Optional | Default is 30. Maximum is 30. Impacts response only if granularity is DAILY |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/template/analytics?start=1711935412&end=1712021812&granularity=DAILY&metric_types=SENT,DELIVERED,READ,CLICKED&template_ids={{TEMPLATE_ID}}&limit=30' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "status": "success",
    "template_analytics": [
        {
            "clicked": [
                {
                    "button_content": "imgurl1",
                    "count": 8,
                    "type": "url_button"
                },
                {
                    "button_content": "imgurl2",
                    "count": 6,
                    "type": "url_button"
                }
            ],
            "delivered": 2,
            "end": 1706572800,
            "read": 1,
            "sent": 2,
            "start": 1706400000,
            "template_id": "2c679517-6450-4556-abff-a64d1355ee7e"
        }
    ]
}
```

### Response Fields Description

- `status`: Status of the request ("success")
- `template_analytics`: Array of analytics data for templates
  - `clicked`: Array of button click analytics
    - `button_content`: Content of the button that was clicked
    - `count`: Number of times the button was clicked
    - `type`: Type of button (e.g., "url_button")
  - `delivered`: Number of messages delivered
  - `read`: Number of messages read
  - `sent`: Number of messages sent
  - `start`: Start timestamp for the analytics period
  - `end`: End timestamp for the analytics period
  - `template_id`: Unique identifier of the template

### Node.js Example

```javascript
const axios = require('axios');

async function getTemplateAnalytics(appId, options, partnerAppToken) {
  try {
    const params = new URLSearchParams();
    
    if (options.start) params.append('start', options.start);
    if (options.end) params.append('end', options.end);
    if (options.granularity) params.append('granularity', options.granularity);
    if (options.metric_types) params.append('metric_types', options.metric_types);
    if (options.template_ids) params.append('template_ids', options.template_ids);
    if (options.limit) params.append('limit', options.limit);
    
    const queryString = params.toString();
    const url = `https://partner.gupshup.io/partner/app/${appId}/template/analytics${queryString ? '?' + queryString : ''}`;
    
    const response = await axios.get(url, {
      headers: {
        'Authorization': partnerAppToken
      }
    });
    
    console.log('Template analytics:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting template analytics:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getTemplateAnalytics('your-app-id', {
  start: 1711935412,
  end: 1712021812,
  granularity: 'DAILY',
  metric_types: 'SENT,DELIVERED,READ,CLICKED',
  template_ids: 'your-template-id',
  limit: 30
}, 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "template_analytics": [...] }` | Successful response |
| Success | 200 | `{ "status": "success", "template_analytics": [] }` | When Meta's API returns no data points (occurs when template has no buttons, or template is authentication type with button clicks disabled) |
| Error | 400 | `{ "message": "Start time can't be older than 90 days", "status": "error" }` | Start time validation error |
| Error | 401 | `{ "status": "error", "message": "Unauthorised access to the resource. Please review request parameters and headers and retry" }` | Unauthorized access |
| Error | 404 | `{ "message": "Template Analytics settings for app [...] does not exist", "status": "error" }` | Template analytics not enabled for the app |

---

## Enable Template Analytics

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/template/analytics`

**Description:** Use this API to enable template analytics on Meta.

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The ID should be a valid app ID of Gupshup |
| enable | True/False | Flag to enable template analytics setting | Boolean | Required | - |
| enableOnUi | True/False | Flag to enable template analytics setting on UI | Boolean | Optional | - |

### Sample Request

```bash
curl --location --request POST 'https://partner.gupshup.io/partner/app/{{APP_ID}}/template/analytics' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'enable=true' \
--data-urlencode 'enableOnUi=true'
```

### Sample Response

```json
{
    "status": "success"
}
```

### Node.js Example

```javascript
const axios = require('axios');

async function enableTemplateAnalytics(appId, enable, enableOnUi, partnerAppToken) {
  try {
    const response = await axios.post(
      `https://partner.gupshup.io/partner/app/${appId}/template/analytics`,
      new URLSearchParams({
        enable: enable,
        enableOnUi: enableOnUi
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Template analytics enabled:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error enabling template analytics:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
enableTemplateAnalytics('your-app-id', true, true, 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success" }` | Successful response |
| Error | 400 | `{ "status": "error", "message": "Bad Request" }` | Bad request |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Internal server error. Please try again later and If Issue still persist than contact Gupshup Dev Support" }` | For any Internal Error |

---

## Disable Button Click Analytics

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/template/analytics/buttonclick`

**Description:** Use this API to disable button click analytics for a template.

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The ID should be a valid app ID of Gupshup |
| templateId | {{TEMPLATE_ID}} | GS-template ID of template | String | Required | - |
| disable | True/False | Flag to disable or enable button click analytics | Boolean | Required | - |

### Sample Request

```bash
curl --location --request POST 'https://partner.gupshup.io/partner/app/{{APP_ID}}/template/analytics/buttonclick' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'templateId={{TEMPLATE_ID}}' \
--data-urlencode 'disable=false'
```

### Sample Response

```json
{
    "status": "success"
}
```

### Node.js Example

```javascript
const axios = require('axios');

async function toggleButtonClickAnalytics(appId, templateId, disable, partnerAppToken) {
  try {
    const response = await axios.post(
      `https://partner.gupshup.io/partner/app/${appId}/template/analytics/buttonclick`,
      new URLSearchParams({
        templateId: templateId,
        disable: disable
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Button click analytics toggled:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error toggling button click analytics:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
toggleButtonClickAnalytics('your-app-id', 'your-template-id', false, 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success" }` | Successful response |
| Error | 401 | `{ "status": "error", "message": "Unauthorised access to the resource. Please review request parameters and headers and retry" }` | Unauthorized access |

---

## Complete Template Analytics Manager Class - Node.js

```javascript
const axios = require('axios');

class GupshupTemplateAnalyticsManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseUrl = 'https://partner.gupshup.io/partner/app';
  }

  async getTemplateAnalytics(appId, options = {}) {
    try {
      const params = new URLSearchParams();
      
      Object.entries(options).forEach(([key, value]) => {
        if (value !== undefined) params.append(key, value);
      });
      
      const queryString = params.toString();
      const url = `${this.baseUrl}/${appId}/template/analytics${queryString ? '?' + queryString : ''}`;
      
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

  async enableTemplateAnalytics(appId, enable, enableOnUi = true) {
    try {
      const response = await axios.post(
        `${this.baseUrl}/${appId}/template/analytics`,
        new URLSearchParams({
          enable: enable,
          enableOnUi: enableOnUi
        }),
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

  async toggleButtonClickAnalytics(appId, templateId, disable) {
    try {
      const response = await axios.post(
        `${this.baseUrl}/${appId}/template/analytics/buttonclick`,
        new URLSearchParams({
          templateId: templateId,
          disable: disable
        }),
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

  // Helper method to get analytics for the last 30 days
  async getRecentAnalytics(appId, templateId, granularity = 'AGGREGATED') {
    const now = Math.floor(Date.now() / 1000);
    const thirtyDaysAgo = now - (30 * 24 * 60 * 60);
    
    return this.getTemplateAnalytics(appId, {
      start: thirtyDaysAgo,
      end: now,
      granularity: granularity,
      template_ids: templateId,
      metric_types: 'SENT,DELIVERED,READ,CLICKED'
    });
  }

  // Helper method to get daily analytics for a specific date range
  async getDailyAnalytics(appId, templateId, startDate, endDate) {
    const start = Math.floor(new Date(startDate).getTime() / 1000);
    const end = Math.floor(new Date(endDate).getTime() / 1000);
    
    return this.getTemplateAnalytics(appId, {
      start: start,
      end: end,
      granularity: 'DAILY',
      template_ids: templateId,
      metric_types: 'SENT,DELIVERED,READ,CLICKED',
      limit: 30
    });
  }
}

// Usage
const analyticsManager = new GupshupTemplateAnalyticsManager('your-partner-app-token');

async function main() {
  const appId = 'your-app-id';
  const templateId = 'your-template-id';
  
  // Enable template analytics
  const enableResult = await analyticsManager.enableTemplateAnalytics(appId, true, true);
  console.log('Enable analytics result:', enableResult);
  
  // Get recent analytics (last 30 days)
  const recentAnalytics = await analyticsManager.getRecentAnalytics(appId, templateId);
  console.log('Recent analytics:', recentAnalytics);
  
  // Get daily analytics for a specific date range
  const dailyAnalytics = await analyticsManager.getDailyAnalytics(
    appId, 
    templateId, 
    '2024-01-01', 
    '2024-01-31'
  );
  console.log('Daily analytics:', dailyAnalytics);
  
  // Enable button click analytics for a template
  const buttonClickResult = await analyticsManager.toggleButtonClickAnalytics(appId, templateId, false);
  console.log('Button click analytics result:', buttonClickResult);
}

main();
```

---

## Important Notes

### General

- All APIs require Partner App Access Token for authentication
- Analytics results are returned in IST (Indian Standard Time)
- Template analytics must be enabled before you can retrieve analytics data

### Time Constraints

- Start time cannot be older than 90 days
- If start and end times are not provided, the API defaults to the last 30 days
- Use epoch timestamps for start and end parameters

### Template Limitations

- Currently supports only 1 template at a time for analytics retrieval
- Button click analytics are only available for templates with buttons
- Authentication type templates may have button clicks disabled

### Metrics and Granularity

- Available metrics: SENT, DELIVERED, READ, CLICKED
- Granularity options: DAILY or AGGREGATED
- Daily granularity supports pagination with limit parameter (max 30)

### Best Practices

1. **Enable Analytics First**: Always enable template analytics before attempting to retrieve data
2. **Time Range Management**: Use appropriate time ranges to avoid hitting the 90-day limit
3. **Pagination**: Use the limit parameter for daily analytics to manage response size
4. **Error Handling**: Implement proper error handling for all API calls
5. **Button Analytics**: Enable button click analytics for templates with interactive buttons
6. **Regular Monitoring**: Set up regular analytics retrieval to monitor template performance

### Analytics Data Interpretation

- **Sent**: Number of messages successfully sent
- **Delivered**: Number of messages delivered to recipients
- **Read**: Number of messages read by recipients
- **Clicked**: Detailed button click information including button content and type
- **Button Types**: Common types include "url_button", "phone_button", "quick_reply"

### Error Handling

- Handle 404 errors when template analytics are not enabled
- Check for empty analytics arrays when templates have no relevant data
- Implement retry logic for rate-limited requests (429 errors)
- Validate template IDs before making API calls

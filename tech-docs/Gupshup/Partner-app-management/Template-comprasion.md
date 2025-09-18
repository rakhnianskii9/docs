# Gupshup Template Comparison and Media API Documentation

This document covers the Template Comparison and Media Management APIs for Gupshup Partner applications. These APIs allow you to compare template performance and manage media files for your WhatsApp Business applications.

---

## Template Comparison API

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/template/analytics/{templateId}/compare`

**Description:** Use this API to compare templates performance metrics.

> **📘 Important Notes:**
> 1. The API needs sufficient messages (1000) in the given time frame to work.
> 2. The difference between the start and end timestamps should be exactly 604800 (7 days), 2592000 (30 days), 5184000 (60 days), or 7776000 (90 days).

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID of the app for which the embed link needs to be generated | String | Required | Should be a valid appId associated with the account |
| templateId | {{TEMPLATE_ID}} | ID of the WhatsApp Message Template to target | String | Required | Should be a valid template ID |
| template_ids | {{TEMPLATE_IDS}} | Array of template IDs to compare | String | Required | The array should contain exactly one valid template ID |
| start_timestamp | {{START}} | Start time | Long | Required | Start time in seconds (epoch format) |
| end_timestamp | {{END}} | End time | Long | Required | End time in seconds (epoch format) |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/template/analytics/{{TEMPLATE_ID}}/compare?templateList={{COMPARE_TEMPLATE_ID}}&end={{END_TIMESTAMP}}&start={{START_TIMESTAMP}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "data": [
        {
            "metric": "BLOCK_RATE",
            "order_by_relative_metric": ["template-id-1", "template-id-2"],
            "type": "RELATIVE"
        },
        {
            "metric": "MESSAGE_SENDS",
            "number_values": [
                {
                    "key": "template-id-1",
                    "value": 1500
                },
                {
                    "key": "template-id-2",
                    "value": 1200
                }
            ],
            "type": "NUMBER_VALUES"
        },
        {
            "metric": "TOP_BLOCK_REASON",
            "string_values": [
                {
                    "key": "template-id-1",
                    "value": "Spam"
                },
                {
                    "key": "template-id-2",
                    "value": "Policy violation"
                }
            ],
            "type": "STRING_VALUES"
        }
    ],
    "status": "success"
}
```

### Response Fields Description

- `status`: Status of the request ("success")
- `data`: Array of comparison metrics
  - `metric`: Type of metric (BLOCK_RATE, MESSAGE_SENDS, TOP_BLOCK_REASON)
  - `type`: Data type (RELATIVE, NUMBER_VALUES, STRING_VALUES)
  - `order_by_relative_metric`: Array of template IDs ordered by relative metric
  - `number_values`: Array of numerical values for each template
  - `string_values`: Array of string values for each template

### Node.js Example

```javascript
const axios = require('axios');

async function compareTemplates(appId, templateId, compareTemplateId, startTimestamp, endTimestamp, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/template/analytics/${templateId}/compare`,
      {
        params: {
          templateList: compareTemplateId,
          start: startTimestamp,
          end: endTimestamp
        },
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Template comparison result:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error comparing templates:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
const startTime = Math.floor(Date.now() / 1000) - (30 * 24 * 60 * 60); // 30 days ago
const endTime = Math.floor(Date.now() / 1000);
compareTemplates('your-app-id', 'template-id-1', 'template-id-2', startTime, endTime, 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "data": [...], "status": "success" }` | Successful response |
| Error | 400 | `{ "status": "error", "message": "Authentication Failed" }` | Authentication failed |
| Error | 400 | `{ "status": "error", "message": "Invalid parameter - Templates not eligible for comparison, Templates must have been sent at least 1,000 times in the specified time frame." }` | Insufficient message volume |
| Error | 400 | `{ "status": "error", "message": "(#100) The parameter template_ids is required. - null, null" }` | Missing required parameter |
| Error | 400 | `{ "message": "The difference between start and end times should be the number of seconds in 604800(7 days), 2592000(30 days), 5184000(60 days), 7776000(90 days).", "status": "error" }` | Invalid time range |
| Error | 400 | `{ "message": "One or more templates in the list is not approved.", "status": "error" }` | Template not approved |
| Error | 400 | `{ "message": "The parameter start_timestamp should be less than end_timestamp.", "status": "error" }` | Invalid timestamp order |
| Error | 400 | `{ "message": "Not Template Owner", "status": "error" }` | Not authorized to access template |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | Rate limit exceeded |

---

## Generate Media ID Using File Upload

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/media`

**Description:** Use this API to generate the media ID using file upload.

> **⚠️ Important:** All media files sent through this endpoint are encrypted and persist for 30 days, unless they are deleted earlier.

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The ID should be a valid appId of Gupshup |
| file_type | {{FILE_TYPE}} | File type to generate media ID | String | Required | Supported types: audio/aac, audio/mp4, audio/mpeg, audio/amr, audio/ogg, audio/opus, application/vnd.ms-powerpoint, application/msword, application/vnd.openxmlformats-officedocument.wordprocessingml.document, application/vnd.openxmlformats-officedocument.presentationml.presentation, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/pdf, text/plain, application/vnd.ms-excel, image/jpeg, image/png, image/webp, video/mp4, video/3gpp |
| file | {{FILE_PATH}} | File path to upload file | File | Required | Should be valid file path. File size should not be more than 100 MB |

### Sample Request

```bash
curl --location --request POST 'https://partner.gupshup.io/partner/app/{{APP_ID}}/media' \
--header 'token: {{PARTNER_APP_TOKEN}}' \
--form 'file_type="{{FILE_TYPE}}"' \
--form 'file=@"/path/to/file"'
```

### Sample Response

```json
{
    "mediaId": "1852559851913765",
    "status": "success"
}
```

### Node.js Example

```javascript
const axios = require('axios');
const FormData = require('form-data');
const fs = require('fs');

async function uploadMedia(appId, filePath, fileType, partnerAppToken) {
  try {
    const formData = new FormData();
    formData.append('file', fs.createReadStream(filePath));
    formData.append('file_type', fileType);

    const response = await axios.post(
      `https://partner.gupshup.io/partner/app/${appId}/media`,
      formData,
      {
        headers: {
          'token': partnerAppToken,
          ...formData.getHeaders()
        }
      }
    );
    
    console.log('Media uploaded successfully:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error uploading media:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
uploadMedia('your-app-id', '/path/to/image.jpg', 'image/jpeg', 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "mediaId": "1852559851913765", "status": "success" }` | Successful upload |
| Error | 400 | `{ "message": "Only CAPI apps allowed to generate media ID", "status": "error" }` | App is not CAPI enabled |
| Error | 413 | `{ "message": "File size exceeds the maximum limit!", "status": "error" }` | File exceeds 100 MB limit |
| Error | 500 | `{ "message": "Unable to upload requested Media", "status": "error" }` | Internal server error |

---

## Generate Media ID Using URL

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/media`

**Description:** Use this API to generate the media ID using URL.

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The ID should be a valid app ID of Gupshup |
| file_type | {{FILE_TYPE}} | File type to generate media ID | String | Required | Same supported types as file upload |
| file | {{FILE_URL}} | File URL to generate media ID | String | Required | Should be a valid file URL. File size should not be more than 100 MB |

### Sample Request

```bash
curl --request POST \
     --url https://partner.gupshup.io/partner/app/{{APP_ID}}/media \
     --header 'accept: application/json' \
     --header 'content-type: multipart/form-data' \
     --header 'token: {{PARTNER_APP_TOKEN}}' \
     --form file={{FILE_URL}} \
     --form file_type={{FILE_TYPE}}
```

### Sample Response

```json
{
  "mediaId": "1852559851913765",
  "status": "success"
}
```

### Node.js Example

```javascript
const axios = require('axios');

async function generateMediaIdFromUrl(appId, fileUrl, fileType, partnerAppToken) {
  try {
    const formData = new FormData();
    formData.append('file', fileUrl);
    formData.append('file_type', fileType);

    const response = await axios.post(
      `https://partner.gupshup.io/partner/app/${appId}/media`,
      formData,
      {
        headers: {
          'token': partnerAppToken,
          'accept': 'application/json',
          'content-type': 'multipart/form-data'
        }
      }
    );
    
    console.log('Media ID generated from URL:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error generating media ID from URL:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
generateMediaIdFromUrl('your-app-id', 'https://example.com/image.jpg', 'image/jpeg', 'your-partner-app-token');
```

### Status Codes

Same as file upload method.

---

## Download Media

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/media/{mediaId}/`

**Description:** Use this API to download the media associated with the app based on media ID at an app level.

> **ℹ️ Rate Limit:** 5 requests per hour

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required | Should be a valid Partner App Access Token |
| appId | {{APP_ID}} | App ID to fetch the access token | String | Required | The ID should be a valid app ID of Gupshup. The App must be associated with the account that owns the PARTNER_APP_TOKEN being used |
| mediaId | {{MEDIA_ID}} | Media ID of the media which is being downloaded | String | Required | Valid media ID associated with the given app ID |
| fileName | {{FILE_NAME}} | Name of the file where the media is being downloaded | String | Optional | Valid file name |

### Sample Request

```bash
curl --location 'https://partner.gupshup.io/partner/app/{{APP_ID}}/media/{{MEDIA_ID}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--output {{FILE_NAME}}
```

### Sample Response

```
Downloaded media stream
```

### Node.js Example

```javascript
const axios = require('axios');
const fs = require('fs');
const path = require('path');

async function downloadMedia(appId, mediaId, fileName, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/media/${mediaId}`,
      {
        headers: {
          'Authorization': partnerAppToken
        },
        responseType: 'stream'
      }
    );
    
    const filePath = path.join(__dirname, fileName);
    const writer = fs.createWriteStream(filePath);
    
    response.data.pipe(writer);
    
    return new Promise((resolve, reject) => {
      writer.on('finish', () => {
        console.log(`Media downloaded successfully: ${filePath}`);
        resolve(filePath);
      });
      writer.on('error', reject);
    });
  } catch (error) {
    console.error('Error downloading media:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
downloadMedia('your-app-id', 'your-media-id', 'downloaded-file.jpg', 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | Downloaded media stream | Successful download |
| Error | 400 | `{ "status":"error","message/data":"Specific to API"}` | Error with respect to API |
| Error | 401 | `{ "status":"error","message":"Unauthorized Access"}` | When authentication fails |
| Error | 500 | `{ "status": "error","message": "Internal Server Error"}` | For any Internal Error |

---

## Complete Template Comparison and Media Manager Class - Node.js

```javascript
const axios = require('axios');
const FormData = require('form-data');
const fs = require('fs');
const path = require('path');

class GupshupTemplateComparisonMediaManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseUrl = 'https://partner.gupshup.io/partner/app';
  }

  // Template Comparison Methods
  async compareTemplates(appId, templateId, compareTemplateId, startTimestamp, endTimestamp) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/template/analytics/${templateId}/compare`,
        {
          params: {
            templateList: compareTemplateId,
            start: startTimestamp,
            end: endTimestamp
          },
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

  // Helper method to get valid time ranges
  getValidTimeRanges() {
    const now = Math.floor(Date.now() / 1000);
    return {
      '7_days': {
        start: now - (7 * 24 * 60 * 60),
        end: now,
        duration: 604800
      },
      '30_days': {
        start: now - (30 * 24 * 60 * 60),
        end: now,
        duration: 2592000
      },
      '60_days': {
        start: now - (60 * 24 * 60 * 60),
        end: now,
        duration: 5184000
      },
      '90_days': {
        start: now - (90 * 24 * 60 * 60),
        end: now,
        duration: 7776000
      }
    };
  }

  // Media Management Methods
  async uploadMedia(appId, filePath, fileType) {
    try {
      const formData = new FormData();
      formData.append('file', fs.createReadStream(filePath));
      formData.append('file_type', fileType);

      const response = await axios.post(
        `${this.baseUrl}/${appId}/media`,
        formData,
        {
          headers: {
            'token': this.partnerAppToken,
            ...formData.getHeaders()
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

  async generateMediaIdFromUrl(appId, fileUrl, fileType) {
    try {
      const formData = new FormData();
      formData.append('file', fileUrl);
      formData.append('file_type', fileType);

      const response = await axios.post(
        `${this.baseUrl}/${appId}/media`,
        formData,
        {
          headers: {
            'token': this.partnerAppToken,
            'accept': 'application/json',
            'content-type': 'multipart/form-data'
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

  async downloadMedia(appId, mediaId, fileName) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/media/${mediaId}`,
        {
          headers: {
            'Authorization': this.partnerAppToken
          },
          responseType: 'stream'
        }
      );
      
      const filePath = path.join(__dirname, fileName);
      const writer = fs.createWriteStream(filePath);
      
      response.data.pipe(writer);
      
      return new Promise((resolve, reject) => {
        writer.on('finish', () => {
          resolve({ success: true, filePath: filePath });
        });
        writer.on('error', (error) => {
          reject({ success: false, error: error.message });
        });
      });
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }

  // Helper method to get supported file types
  getSupportedFileTypes() {
    return {
      audio: [
        'audio/aac',
        'audio/mp4',
        'audio/mpeg',
        'audio/amr',
        'audio/ogg',
        'audio/opus'
      ],
      document: [
        'application/vnd.ms-powerpoint',
        'application/msword',
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        'application/vnd.openxmlformats-officedocument.presentationml.presentation',
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
        'application/pdf',
        'text/plain',
        'application/vnd.ms-excel'
      ],
      image: [
        'image/jpeg',
        'image/png',
        'image/webp'
      ],
      video: [
        'video/mp4',
        'video/3gpp'
      ]
    };
  }
}

// Usage
const manager = new GupshupTemplateComparisonMediaManager('your-partner-app-token');

async function main() {
  const appId = 'your-app-id';
  
  // Compare templates for the last 30 days
  const timeRanges = manager.getValidTimeRanges();
  const comparisonResult = await manager.compareTemplates(
    appId, 
    'template-id-1', 
    'template-id-2',
    timeRanges['30_days'].start,
    timeRanges['30_days'].end
  );
  console.log('Template comparison result:', comparisonResult);
  
  // Upload media file
  const uploadResult = await manager.uploadMedia(appId, '/path/to/image.jpg', 'image/jpeg');
  console.log('Media upload result:', uploadResult);
  
  // Generate media ID from URL
  const urlResult = await manager.generateMediaIdFromUrl(appId, 'https://example.com/image.jpg', 'image/jpeg');
  console.log('Media ID from URL result:', urlResult);
  
  // Download media
  if (uploadResult.success) {
    const downloadResult = await manager.downloadMedia(appId, uploadResult.data.mediaId, 'downloaded-image.jpg');
    console.log('Media download result:', downloadResult);
  }
}

main();
```

---

## Important Notes

### Template Comparison

- **Message Volume Requirement**: Templates must have been sent at least 1,000 times in the specified time frame
- **Time Range Validation**: The difference between start and end timestamps must be exactly 7, 30, 60, or 90 days
- **Template Ownership**: You can only compare templates that you own
- **Template Status**: Templates must be approved to be eligible for comparison

### Media Management

- **File Size Limit**: Maximum file size is 100 MB
- **File Persistence**: Media files are encrypted and persist for 30 days
- **CAPI Requirement**: Only CAPI apps are allowed to generate media IDs
- **Rate Limiting**: Download API has a rate limit of 5 requests per hour

### Supported File Types

- **Audio**: aac, mp4, mpeg, amr, ogg, opus
- **Documents**: PowerPoint, Word, Excel, PDF, plain text
- **Images**: JPEG, PNG, WebP
- **Video**: MP4, 3GPP

### Best Practices

1. **Template Comparison**: Ensure templates have sufficient message volume before comparison
2. **Time Range Planning**: Use the helper methods to get valid time ranges
3. **File Validation**: Validate file types and sizes before uploading
4. **Error Handling**: Implement proper error handling for all API calls
5. **Rate Limiting**: Respect the download API rate limits
6. **File Management**: Clean up downloaded files regularly to save storage space

### Error Handling

- Handle authentication errors properly
- Check for template eligibility before comparison
- Validate file types and sizes before media operations
- Implement retry logic for rate-limited requests
- Monitor file upload progress for large files

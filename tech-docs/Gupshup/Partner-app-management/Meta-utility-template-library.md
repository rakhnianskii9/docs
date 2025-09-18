# Gupshup Meta Utility Template Library API Documentation

This document covers the Meta Utility Template Library APIs for Gupshup Partner applications. These APIs allow you to browse and create templates from Meta's pre-approved template library, which provides ready-to-use templates for common business use cases.

---

## Get Templates from the Library

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/template/metalibrary`

**Description:** Use this API to fetch pre-approved Meta Library Templates. You can filter templates by various criteria to find the most suitable ones for your business needs.

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | App Access Token issued post partner login | String | Required | Should be a valid partner app access token belonging to the passed appId |
| appId | {{APP_ID}} | App ID for the app that is linked to the Partner Account | String | Required | - |
| elementName | {{ELEMENT_NAME}} | Name of element | String | Optional | Meta library template name |
| industry | {{INDUSTRY}} | Name of industry | String | Optional | Possible values: E_COMMERCE, FINANCIAL_SERVICES |
| languageCode | {{LANGUAGE_CODE}} | Language code of the meta library template | String | Optional | Supported Languages from WhatsApp Business Management API |
| topic | {{TOPIC}} | Topic of the meta library template | String | Optional | Possible values: ACCOUNT_UPDATES, CUSTOMER_FEEDBACK, ORDER_MANAGEMENT, PAYMENTS |
| usecase | {{USECASE}} | Usecase of the meta library template | String | Optional | See detailed list below |

### Available Use Cases

The `usecase` parameter supports the following values:
- `ACCOUNT_CREATION_CONFIRMATION`
- `AUTO_PAY_REMINDER`
- `DELIVERY_CONFIRMATION`
- `DELIVERY_FAILED`
- `DELIVERY_UPDATE`
- `FEEDBACK_SURVEY`
- `FIXED_TEMPLATE_PRICE_TEST`
- `FRAUD_ALERT`
- `LOW_BALANCE_WARNING`
- `ORDER_ACTION_NEEDED`
- `ORDER_CONFIRMATION`
- `ORDER_DELAY`
- `ORDER_OR_TRANSACTION_CANCEL`
- `ORDER_PICK_UP`
- `PAYMENT_ACTION_REQUIRED`
- `PAYMENT_CONFIRMATION`
- `PAYMENT_DUE_REMINDER`
- `PAYMENT_NOTICE`
- `PAYMENT_OVERDUE`
- `PAYMENT_REJECT_FAIL`
- `PAYMENT_SCHEDULED`
- `RECEIPT_ATTACHMENT`
- `RETURN_CONFIRMATION`
- `SHIPMENT_CONFIRMATION`
- `STATEMENT_ATTACHMENT`
- `STATEMENT_AVAILABLE`
- `TRANSACTION_ALERT`

### Sample Request

```bash
curl --location 'https://partner.gupshup.io/partner/app/{{APP_ID}}/template/metalibrary?elementName={{ELEMENT_NAME}}&industry={{INDUSTRY}}&languageCode={{LANGUAGE_CODE}}&topic={{TOPIC}}&usecase={{USECASE}}' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "status": "success",
    "templates": [
        {
            "category": "UTILITY",
            "containerMeta": "{\"data\":\"Hei, {{1}}\\n\\nDen nye kontoen din er opprettet.\\n\\nBekreft {{2}} for å fullføre profilen.\",\"buttons\":[{\"type\":\"URL\",\"text\":\"Bekreft konto\",\"url\":\"https://www.example.com\"}],\"header\":\"Fullfør konfigurering av konto\",\"sampleText\":\"Hei, [John]\\n\\nDen nye kontoen din er opprettet.\\n\\nBekreft [e-postadressen din] for å fullføre profilen.\",\"sampleHeader\":\"Fullfør konfigurering av konto\",\"enableSample\":false,\"editTemplate\":false,\"allowTemplateCategoryChange\":false,\"addSecurityRecommendation\":false}",
            "data": "Hei, {{1}}\n\nDen nye kontoen din er opprettet.\n\nBekreft {{2}} for å fullføre profilen.",
            "elementName": "account_creation_confirmation_3",
            "industry": "E_COMMERCE,FINANCIAL_SERVICES",
            "languageCode": "nb",
            "topic": "ACCOUNT_UPDATES",
            "usecase": "ACCOUNT_CREATION_CONFIRMATION"
        },
        {
            "category": "UTILITY",
            "containerMeta": "{\"data\":\"{{1}}, bestillingen din er levert! \\n\\nDu kan spore pakken din og administrere bestillingen nedenfor.\",\"buttons\":[{\"type\":\"URL\",\"text\":\"Administrer bestilling\",\"url\":\"https://www.example.com\"}],\"sampleText\":\"[John], bestillingen din er levert! \\n\\nDu kan spore pakken din og administrere bestillingen nedenfor.\",\"enableSample\":false,\"editTemplate\":false,\"allowTemplateCategoryChange\":false,\"addSecurityRecommendation\":false}",
            "data": "{{1}}, bestillingen din er levert! \n\nDu kan spore pakken din og administrere bestillingen nedenfor.",
            "elementName": "delivery_confirmation_1",
            "industry": "E_COMMERCE",
            "languageCode": "nb",
            "topic": "ORDER_MANAGEMENT",
            "usecase": "DELIVERY_CONFIRMATION"
        }
    ]
}
```

### Response Fields Description

- `status`: Status of the request ("success")
- `templates`: Array of template objects
  - `category`: Template category (currently "UTILITY")
  - `containerMeta`: JSON string containing template metadata including buttons, headers, and sample text
  - `data`: Template message content with placeholders
  - `elementName`: Unique identifier for the template
  - `industry`: Industries this template is suitable for
  - `languageCode`: Language code of the template
  - `topic`: Topic category of the template
  - `usecase`: Specific use case for the template

### Node.js Example

```javascript
const axios = require('axios');

async function getTemplatesFromLibrary(appId, filters, partnerAppToken) {
  try {
    const params = new URLSearchParams();
    
    // Add optional filters
    if (filters.elementName) params.append('elementName', filters.elementName);
    if (filters.industry) params.append('industry', filters.industry);
    if (filters.languageCode) params.append('languageCode', filters.languageCode);
    if (filters.topic) params.append('topic', filters.topic);
    if (filters.usecase) params.append('usecase', filters.usecase);
    
    const queryString = params.toString();
    const url = `https://partner.gupshup.io/partner/app/${appId}/template/metalibrary${queryString ? '?' + queryString : ''}`;
    
    const response = await axios.get(url, {
      headers: {
        'Authorization': partnerAppToken
      }
    });
    
    console.log('Templates from library:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting templates from library:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getTemplatesFromLibrary('your-app-id', {
  industry: 'E_COMMERCE',
  topic: 'ORDER_MANAGEMENT',
  usecase: 'DELIVERY_CONFIRMATION',
  languageCode: 'en'
}, 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "templates": [...] }` | Templates found |
| Success | 200 | `{ "status": "success", "templates": [] }` | No templates found for given filters |
| Error | 401 | `{ "message": "Authentication Failed", "status": "error" }` | Authentication failed |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Internal Server Error" }` | Internal server error |

---

## Create Templates from the Template Library

**Endpoint:** `POST https://partner.gupshup.io/partner/app/{appId}/template/metalibrary`

**Description:** Use this API to create a template from a pre-approved Meta Library Template. This allows you to customize the library templates for your specific business needs.

### Parameters

| Parameter | Value | Description | Type | Required | Notes |
|-----------|-------|-------------|------|----------|-------|
| Authorization | {{PARTNER_APP_TOKEN}} | App Access Token issued post partner login | String | Required | Should be a valid partner app access token belonging to the passed appId |
| appId | {{APP_ID}} | App ID for the app that is linked to the Partner Account | String | Required | - |
| elementName | {{NEW_ELEMENT_NAME}} | Name for the new template | String | Required | Unique name for your new template |
| category | UTILITY | Category name | String | Required | Currently only UTILITY is supported |
| languageCode | {{LANGUAGE_CODE}} | Language code of the meta library template | String | Required | Must match the library template's language |
| libraryTemplateName | {{LIBRARY_TEMPLATE_NAME}} | Name of meta library template used as base | String | Required | Element name of the library template |
| buttons | {{BUTTONS}} | Buttons list for new template | String | Required | Buttons in the same format as the library template |

### Sample Request

```bash
curl --location --request POST 'https://partner.gupshup.io/partner/app/{{APP_ID}}/template/metalibrary' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'elementName={{NEW_ELEMENT_NAME}}' \
--data-urlencode 'category=UTILITY' \
--data-urlencode 'languageCode={{LANGUAGE_CODE}}' \
--data-urlencode 'libraryTemplateName={{LIBRARY_TEMPLATE_NAME}}' \
--data-urlencode 'buttons={{BUTTONS}}'
```

### Sample Response

```json
{
    "status": "success",
    "templates": {
        "buttonSupported": "URL",
        "category": "UTILITY",
        "containerMeta": "{\"data\":\"Hei, {{1}}\\n\\nDen nye kontoen din er opprettet.\\n\\nBekreft {{2}} for å fullføre profilen.\",\"buttons\":[{\"type\":\"URL\",\"text\":\"Bekreft konto\",\"url\":\"https://www.example.com/\"}],\"header\":\"Fullfør konfigurering av konto\",\"sampleText\":\"Hei, John\\n\\nDen nye kontoen din er opprettet.\\n\\nBekreft e-postadressen din for å fullføre profilen.\",\"sampleHeader\":\"Fullfør konfigurering av konto\",\"enableSample\":true,\"editTemplate\":false,\"allowTemplateCategoryChange\":false,\"addSecurityRecommendation\":false}",
        "createdOn": 1721897898300,
        "data": "Fullfør konfigurering av konto\nHei, {{1}}\n\nDen nye kontoen din er opprettet.\n\nBekreft {{2}} for å fullføre profilen. | [Bekreft konto,https://www.example.com/]",
        "elementName": "jul23libtest1_8",
        "externalId": "399568216474711",
        "id": "0cc281c9-dc45-4ea3-bf50-411e9b8ff9b3",
        "languageCode": "nb",
        "languagePolicy": "deterministic",
        "meta": "{\"example\":\"Hei, John\\n\\nDen nye kontoen din er opprettet.\\n\\nBekreft e-postadressen din for å fullføre profilen.\"}",
        "modifiedOn": 1721897898300,
        "namespace": "18cfa544_9c62_4dcd_b8f3_b3785d8c917c",
        "quality": "UNKNOWN",
        "status": "APPROVED",
        "templateType": "TEXT",
        "wabaId": "216141188246170"
    }
}
```

### Response Fields Description

- `status`: Status of the request ("success")
- `templates`: Object containing the created template information
  - `buttonSupported`: Type of buttons supported
  - `category`: Template category
  - `containerMeta`: JSON string with template metadata
  - `createdOn`: Timestamp when template was created
  - `data`: Template content with formatting
  - `elementName`: Name of the created template
  - `externalId`: External ID from WhatsApp
  - `id`: Internal Gupshup template ID
  - `languageCode`: Language code of the template
  - `languagePolicy`: Language policy setting
  - `meta`: JSON string with example data
  - `modifiedOn`: Timestamp when template was last modified
  - `namespace`: WhatsApp namespace
  - `quality`: Template quality rating
  - `status`: Template approval status
  - `templateType`: Type of template
  - `wabaId`: WhatsApp Business Account ID

### Node.js Example

```javascript
const axios = require('axios');

async function createTemplateFromLibrary(appId, templateData, partnerAppToken) {
  try {
    const response = await axios.post(
      `https://partner.gupshup.io/partner/app/${appId}/template/metalibrary`,
      new URLSearchParams({
        elementName: templateData.elementName,
        category: templateData.category || 'UTILITY',
        languageCode: templateData.languageCode,
        libraryTemplateName: templateData.libraryTemplateName,
        buttons: templateData.buttons
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Template created from library:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error creating template from library:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
createTemplateFromLibrary('your-app-id', {
  elementName: 'my_delivery_confirmation',
  category: 'UTILITY',
  languageCode: 'en',
  libraryTemplateName: 'delivery_confirmation_1',
  buttons: '[{"type":"URL","text":"Track Order","url":"https://mystore.com/track"}]'
}, 'your-partner-app-token');
```

### Status Codes

| Type | Code | Response | Description |
|------|------|----------|-------------|
| Success | 200 | `{ "status": "success", "templates": {...} }` | Template created successfully |
| Error | 400 | `{ "status": "error", "message": "Template Name is missing" }` | elementName not provided |
| Error | 400 | `{ "status": "error", "message": "Category is missing" }` | category not provided |
| Error | 400 | `{ "status": "error", "message": "Language is missing" }` | languageCode not provided |
| Error | 400 | `{ "status": "error", "message": "Library Template Name is missing" }` | libraryTemplateName not provided |
| Error | 400 | `{ "status": "error", "message": "(#100) The parameter library_template_button_inputs[0]['type'] is required. - null, null" }` | Button type not provided |
| Error | 400 | `{ "status": "error", "message": "Template Already exists with same namespace and elementName and languageCode" }` | Template name already exists |
| Error | 401 | `{ "message": "Authentication Failed", "status": "error" }` | Authentication failed |
| Error | 429 | `{ "status": "error", "message": "Too Many Requests" }` | 10 Requests per Minute |
| Error | 500 | `{ "status": "error", "message": "Internal Server Error" }` | Internal server error |

---

## Complete Meta Template Library Manager Class - Node.js

```javascript
const axios = require('axios');

class GupshupMetaTemplateLibraryManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseUrl = 'https://partner.gupshup.io/partner/app';
  }

  async getTemplatesFromLibrary(appId, filters = {}) {
    try {
      const params = new URLSearchParams();
      
      // Add optional filters
      Object.entries(filters).forEach(([key, value]) => {
        if (value !== undefined && value !== null) {
          params.append(key, value);
        }
      });
      
      const queryString = params.toString();
      const url = `${this.baseUrl}/${appId}/template/metalibrary${queryString ? '?' + queryString : ''}`;
      
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

  async createTemplateFromLibrary(appId, templateData) {
    try {
      const response = await axios.post(
        `${this.baseUrl}/${appId}/template/metalibrary`,
        new URLSearchParams({
          elementName: templateData.elementName,
          category: templateData.category || 'UTILITY',
          languageCode: templateData.languageCode,
          libraryTemplateName: templateData.libraryTemplateName,
          buttons: templateData.buttons
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

  // Helper method to get templates by industry
  async getTemplatesByIndustry(appId, industry, languageCode = 'en') {
    return this.getTemplatesFromLibrary(appId, {
      industry: industry,
      languageCode: languageCode
    });
  }

  // Helper method to get templates by use case
  async getTemplatesByUseCase(appId, usecase, languageCode = 'en') {
    return this.getTemplatesFromLibrary(appId, {
      usecase: usecase,
      languageCode: languageCode
    });
  }

  // Helper method to get templates by topic
  async getTemplatesByTopic(appId, topic, languageCode = 'en') {
    return this.getTemplatesFromLibrary(appId, {
      topic: topic,
      languageCode: languageCode
    });
  }

  // Helper method to search templates
  async searchTemplates(appId, searchCriteria) {
    return this.getTemplatesFromLibrary(appId, searchCriteria);
  }

  // Helper method to create template with custom buttons
  async createCustomTemplate(appId, libraryTemplate, customization) {
    const templateData = {
      elementName: customization.elementName,
      category: 'UTILITY',
      languageCode: libraryTemplate.languageCode,
      libraryTemplateName: libraryTemplate.elementName,
      buttons: JSON.stringify(customization.buttons)
    };

    return this.createTemplateFromLibrary(appId, templateData);
  }

  // Helper method to get available industries
  getAvailableIndustries() {
    return ['E_COMMERCE', 'FINANCIAL_SERVICES'];
  }

  // Helper method to get available topics
  getAvailableTopics() {
    return [
      'ACCOUNT_UPDATES',
      'CUSTOMER_FEEDBACK',
      'ORDER_MANAGEMENT',
      'PAYMENTS'
    ];
  }

  // Helper method to get available use cases
  getAvailableUseCases() {
    return [
      'ACCOUNT_CREATION_CONFIRMATION',
      'AUTO_PAY_REMINDER',
      'DELIVERY_CONFIRMATION',
      'DELIVERY_FAILED',
      'DELIVERY_UPDATE',
      'FEEDBACK_SURVEY',
      'FIXED_TEMPLATE_PRICE_TEST',
      'FRAUD_ALERT',
      'LOW_BALANCE_WARNING',
      'ORDER_ACTION_NEEDED',
      'ORDER_CONFIRMATION',
      'ORDER_DELAY',
      'ORDER_OR_TRANSACTION_CANCEL',
      'ORDER_PICK_UP',
      'PAYMENT_ACTION_REQUIRED',
      'PAYMENT_CONFIRMATION',
      'PAYMENT_DUE_REMINDER',
      'PAYMENT_NOTICE',
      'PAYMENT_OVERDUE',
      'PAYMENT_REJECT_FAIL',
      'PAYMENT_SCHEDULED',
      'RECEIPT_ATTACHMENT',
      'RETURN_CONFIRMATION',
      'SHIPMENT_CONFIRMATION',
      'STATEMENT_ATTACHMENT',
      'STATEMENT_AVAILABLE',
      'TRANSACTION_ALERT'
    ];
  }

  // Helper method to parse container meta
  parseContainerMeta(containerMeta) {
    try {
      return JSON.parse(containerMeta);
    } catch (error) {
      console.error('Error parsing container meta:', error);
      return null;
    }
  }

  // Helper method to extract button information
  extractButtons(template) {
    const containerMeta = this.parseContainerMeta(template.containerMeta);
    return containerMeta?.buttons || [];
  }

  // Helper method to get template preview
  getTemplatePreview(template) {
    const containerMeta = this.parseContainerMeta(template.containerMeta);
    return {
      header: containerMeta?.header,
      body: template.data,
      buttons: containerMeta?.buttons,
      sampleText: containerMeta?.sampleText
    };
  }
}

// Usage
const templateManager = new GupshupMetaTemplateLibraryManager('your-partner-app-token');

async function main() {
  const appId = 'your-app-id';
  
  // Get all available templates
  const allTemplates = await templateManager.getTemplatesFromLibrary(appId);
  console.log('All templates:', allTemplates);
  
  // Get e-commerce templates
  const ecommerceTemplates = await templateManager.getTemplatesByIndustry(appId, 'E_COMMERCE');
  console.log('E-commerce templates:', ecommerceTemplates);
  
  // Get delivery confirmation templates
  const deliveryTemplates = await templateManager.getTemplatesByUseCase(appId, 'DELIVERY_CONFIRMATION');
  console.log('Delivery confirmation templates:', deliveryTemplates);
  
  // Search for specific templates
  const searchResults = await templateManager.searchTemplates(appId, {
    industry: 'E_COMMERCE',
    topic: 'ORDER_MANAGEMENT',
    languageCode: 'en'
  });
  console.log('Search results:', searchResults);
  
  // Create a custom template from library
  if (deliveryTemplates.success && deliveryTemplates.data.templates.length > 0) {
    const libraryTemplate = deliveryTemplates.data.templates[0];
    
    const customTemplate = await templateManager.createCustomTemplate(appId, libraryTemplate, {
      elementName: 'my_custom_delivery_notification',
      buttons: [
        {
          type: 'URL',
          text: 'Track Your Order',
          url: 'https://mystore.com/track'
        }
      ]
    });
    console.log('Custom template created:', customTemplate);
  }
  
  // Get template preview
  if (allTemplates.success && allTemplates.data.templates.length > 0) {
    const preview = templateManager.getTemplatePreview(allTemplates.data.templates[0]);
    console.log('Template preview:', preview);
  }
  
  // Get available options
  console.log('Available industries:', templateManager.getAvailableIndustries());
  console.log('Available topics:', templateManager.getAvailableTopics());
  console.log('Available use cases:', templateManager.getAvailableUseCases());
}

main();
```

---

## Important Notes

### General

- All APIs require Partner App Access Token for authentication
- Templates from the library are pre-approved by Meta, ensuring faster approval process
- Currently only UTILITY category templates are supported

### Template Library Benefits

- **Pre-approved**: Templates are already approved by Meta
- **Industry-specific**: Templates tailored for different industries
- **Use case focused**: Templates designed for specific business scenarios
- **Multi-language**: Support for various languages
- **Customizable**: Buttons and URLs can be customized for your business

### Available Filters

- **Industry**: E_COMMERCE, FINANCIAL_SERVICES
- **Topic**: ACCOUNT_UPDATES, CUSTOMER_FEEDBACK, ORDER_MANAGEMENT, PAYMENTS
- **Use Case**: 26+ specific use cases available
- **Language**: Multiple language codes supported
- **Element Name**: Search for specific template names

### Template Creation Process

1. **Browse Library**: Use the GET endpoint to find suitable templates
2. **Select Template**: Choose a template that matches your use case
3. **Customize**: Modify buttons, URLs, and element name as needed
4. **Create**: Use the POST endpoint to create your custom template
5. **Deploy**: The template is automatically approved and ready to use

### Best Practices

1. **Search Efficiently**: Use filters to narrow down template selection
2. **Naming Convention**: Use descriptive and unique element names
3. **Button Customization**: Ensure URLs and button text match your business
4. **Language Consistency**: Match language codes between library template and your version
5. **Industry Alignment**: Choose templates that align with your business industry
6. **Use Case Matching**: Select templates that match your specific communication needs

### Button Format

Buttons should be provided as a JSON string with the following structure:
```json
[
  {
    "type": "URL",
    "text": "Button Text",
    "url": "https://your-domain.com/action"
  }
]
```

### Error Handling

- Handle template name conflicts by using unique element names
- Validate button format before creating templates
- Check for required parameters before making API calls
- Implement retry logic for rate-limited requests
- Monitor template creation status and handle approval workflows

### Rate Limiting

- Both endpoints have a rate limit of 10 requests per minute
- Implement proper throttling in your applications
- Use bulk operations when possible to minimize API calls

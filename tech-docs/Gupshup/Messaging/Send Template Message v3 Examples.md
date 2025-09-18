# Gupshup Partner API - Send Template Message V3 Examples

## Overview
The Send Template Message V3 API allows you to send WhatsApp template messages to users. Template messages are pre-approved message formats that can be used for marketing, notifications, and customer service communications. This API supports various template types including text, media, interactive, location, authentication, and specialized templates.

## Base Endpoint
**URL:** `https://partner.gupshup.io/partner/app/{appId}/v3/message`  
**Method:** `POST`  
**Content-Type:** `application/json`

## Common Request Parameters

| Parameter | Description | Type | Required | Notes |
|-----------|-------------|------|----------|-------|
| Authorization | Partner App Access Token | String | Required | Should be a valid Partner App Access Token |
| appId | Application ID | String | Required | The ID should be a valid Gupshup app ID |
| messaging_product | Messaging product | String | Required | Must be "whatsapp" |
| recipient_type | Recipient type | String | Required | Must be "individual" |
| to | Destination phone number | String | Required | Must be a valid phone number |
| type | Message type | String | Required | Must be "template" |
| template | Template object | Object | Required | Contains template details |

## Common Request Headers

```javascript
{
    "Authorization": "YOUR_PARTNER_APP_TOKEN",
    "Content-Type": "application/json"
}
```

## Template Message Types Overview

| Template Type | Description |
|---------------|-------------|
| **Text-Based Template** | Simple text messages with dynamic parameters |
| **Media-Based Template** | Templates with image, video, or document headers |
| **Interactive Template** | Templates with buttons (call-to-action, quick reply) |
| **Authentication Template** | Templates with OTP and authentication buttons |
| **Location Template** | Templates with location parameters |
| **Multi-Product Template** | Templates showcasing multiple products |
| **Product Card Carousel** | Horizontal scrollable product cards |
| **Flow Template** | Templates with WhatsApp Flow integration |

---

## 1. Text-Based Template Message

### Basic Text Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "TEMPLATE_NAME",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "John Doe"
            },
            {
              "type": "text",
              "text": "ORDER12345"
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
const axios = require('axios');

class WhatsAppTemplateManager {
    constructor(appId, token) {
        this.appId = appId;
        this.token = token;
        this.baseURL = 'https://partner.gupshup.io/partner';
    }

    async sendTextTemplate(to, templateName, languageCode, parameters) {
        try {
            const response = await axios.post(
                `${this.baseURL}/app/${this.appId}/v3/message`,
                {
                    messaging_product: "whatsapp",
                    recipient_type: "individual",
                    to: to,
                    type: "template",
                    template: {
                        name: templateName,
                        language: {
                            code: languageCode
                        },
                        components: [
                            {
                                type: "body",
                                parameters: parameters.map(param => ({
                                    type: "text",
                                    text: param
                                }))
                            }
                        ]
                    }
                },
                {
                    headers: {
                        'Authorization': this.token,
                        'Content-Type': 'application/json'
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            console.error('Error sending text template:', error.response?.data || error.message);
            throw error;
        }
    }
}

// Usage
const templateManager = new WhatsAppTemplateManager('your-app-id', 'your-token');
templateManager.sendTextTemplate(
    '1234567890', 
    'order_confirmation', 
    'en_US', 
    ['John Doe', 'ORDER12345']
)
.then(result => console.log('Template sent:', result))
.catch(error => console.error('Error:', error));
```

---

## 2. Media-Based Template Message

### Image Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "TEMPLATE_NAME",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "header",
          "parameters": [
            {
              "type": "image",
              "image": {
                "link": "https://example.com/image.jpg"
              }
            }
          ]
        },
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "Product Name"
            },
            {
              "type": "currency",
              "currency": {
                "fallback_value": "$19.99",
                "code": "USD",
                "amount_1000": 19990
              }
            }
          ]
        }
      ]
    }
  }'
```

### Video Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "TEMPLATE_NAME",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "header",
          "parameters": [
            {
              "type": "video",
              "video": {
                "link": "https://example.com/video.mp4"
              }
            }
          ]
        },
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "Video Title"
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
async function sendMediaTemplate(to, templateName, languageCode, mediaType, mediaUrl, bodyParameters) {
    try {
        const headerComponent = {
            type: "header",
            parameters: [
                {
                    type: mediaType,
                    [mediaType]: {
                        link: mediaUrl
                    }
                }
            ]
        };

        const bodyComponent = {
            type: "body",
            parameters: bodyParameters.map(param => {
                if (typeof param === 'string') {
                    return { type: "text", text: param };
                } else if (param.type === 'currency') {
                    return {
                        type: "currency",
                        currency: {
                            fallback_value: param.fallback_value,
                            code: param.code,
                            amount_1000: param.amount_1000
                        }
                    };
                } else if (param.type === 'date_time') {
                    return {
                        type: "date_time",
                        date_time: {
                            fallback_value: param.fallback_value
                        }
                    };
                }
                return param;
            })
        };

        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "template",
                template: {
                    name: templateName,
                    language: {
                        code: languageCode
                    },
                    components: [headerComponent, bodyComponent]
                }
            },
            {
                headers: {
                    'Authorization': this.token,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending media template:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppTemplateManager class
WhatsAppTemplateManager.prototype.sendMediaTemplate = sendMediaTemplate;
```

---

## 3. Interactive Template Message

### Button Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "TEMPLATE_NAME",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "header",
          "parameters": [
            {
              "type": "image",
              "image": {
                "link": "https://example.com/image.jpg"
              }
            }
          ]
        },
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "John Doe"
            },
            {
              "type": "currency",
              "currency": {
                "fallback_value": "$19.99",
                "code": "USD",
                "amount_1000": 19990
              }
            }
          ]
        },
        {
          "type": "button",
          "sub_type": "quick_reply",
          "index": "0",
          "parameters": [
            {
              "type": "payload",
              "payload": "CONFIRM_ORDER"
            }
          ]
        },
        {
          "type": "button",
          "sub_type": "quick_reply",
          "index": "1",
          "parameters": [
            {
              "type": "payload",
              "payload": "CANCEL_ORDER"
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
async function sendInteractiveTemplate(to, templateName, languageCode, headerParams, bodyParams, buttonParams) {
    try {
        const components = [];

        // Header component
        if (headerParams) {
            components.push({
                type: "header",
                parameters: headerParams
            });
        }

        // Body component
        if (bodyParams && bodyParams.length > 0) {
            components.push({
                type: "body",
                parameters: bodyParams
            });
        }

        // Button components
        if (buttonParams && buttonParams.length > 0) {
            buttonParams.forEach((button, index) => {
                components.push({
                    type: "button",
                    sub_type: button.sub_type || "quick_reply",
                    index: index.toString(),
                    parameters: [
                        {
                            type: "payload",
                            payload: button.payload
                        }
                    ]
                });
            });
        }

        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "template",
                template: {
                    name: templateName,
                    language: {
                        code: languageCode
                    },
                    components: components
                }
            },
            {
                headers: {
                    'Authorization': this.token,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending interactive template:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppTemplateManager class
WhatsAppTemplateManager.prototype.sendInteractiveTemplate = sendInteractiveTemplate;
```

---

## 4. Authentication Template Message

### OTP Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "TEMPLATE_NAME",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "123456"
            }
          ]
        },
        {
          "type": "button",
          "sub_type": "url",
          "index": "0",
          "parameters": [
            {
              "type": "text",
              "text": "123456"
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
async function sendAuthTemplate(to, templateName, languageCode, otpCode) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "template",
                template: {
                    name: templateName,
                    language: {
                        code: languageCode
                    },
                    components: [
                        {
                            type: "body",
                            parameters: [
                                {
                                    type: "text",
                                    text: otpCode
                                }
                            ]
                        },
                        {
                            type: "button",
                            sub_type: "url",
                            index: "0",
                            parameters: [
                                {
                                    type: "text",
                                    text: otpCode
                                }
                            ]
                        }
                    ]
                }
            },
            {
                headers: {
                    'Authorization': this.token,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending auth template:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppTemplateManager class
WhatsAppTemplateManager.prototype.sendAuthTemplate = sendAuthTemplate;
```

---

## 5. Location Template Message

### Location Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "order_delivery_update",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "header",
          "parameters": [
            {
              "type": "location",
              "location": {
                "latitude": "37.483307",
                "longitude": "-122.148981",
                "name": "Delivery Location",
                "address": "123 Main St, San Francisco, CA 94102"
              }
            }
          ]
        },
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "John Doe"
            },
            {
              "type": "text",
              "text": "ORDER12345"
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
async function sendLocationTemplate(to, templateName, languageCode, location, bodyParams) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "template",
                template: {
                    name: templateName,
                    language: {
                        code: languageCode
                    },
                    components: [
                        {
                            type: "header",
                            parameters: [
                                {
                                    type: "location",
                                    location: {
                                        latitude: location.latitude,
                                        longitude: location.longitude,
                                        name: location.name,
                                        address: location.address
                                    }
                                }
                            ]
                        },
                        {
                            type: "body",
                            parameters: bodyParams.map(param => ({
                                type: "text",
                                text: param
                            }))
                        }
                    ]
                }
            },
            {
                headers: {
                    'Authorization': this.token,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending location template:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppTemplateManager class
WhatsAppTemplateManager.prototype.sendLocationTemplate = sendLocationTemplate;
```

---

## 6. Multi-Product Template Message

### Multi-Product Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "abandoned_cart",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "header",
          "parameters": [
            {
              "type": "text",
              "text": "John Doe"
            }
          ]
        },
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "10OFF"
            }
          ]
        },
        {
          "type": "button",
          "sub_type": "mpm",
          "index": 0,
          "parameters": [
            {
              "type": "action",
              "action": {
                "thumbnail_product_retailer_id": "product123",
                "sections": [
                  {
                    "title": "Popular Items",
                    "product_items": [
                      {
                        "product_retailer_id": "product123"
                      },
                      {
                        "product_retailer_id": "product456"
                      }
                    ]
                  }
                ]
              }
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
async function sendMultiProductTemplate(to, templateName, languageCode, headerParams, bodyParams, productAction) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "template",
                template: {
                    name: templateName,
                    language: {
                        code: languageCode
                    },
                    components: [
                        {
                            type: "header",
                            parameters: headerParams.map(param => ({
                                type: "text",
                                text: param
                            }))
                        },
                        {
                            type: "body",
                            parameters: bodyParams.map(param => ({
                                type: "text",
                                text: param
                            }))
                        },
                        {
                            type: "button",
                            sub_type: "mpm",
                            index: 0,
                            parameters: [
                                {
                                    type: "action",
                                    action: productAction
                                }
                            ]
                        }
                    ]
                }
            },
            {
                headers: {
                    'Authorization': this.token,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending multi-product template:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppTemplateManager class
WhatsAppTemplateManager.prototype.sendMultiProductTemplate = sendMultiProductTemplate;
```

---

## 7. Product Card Carousel Template

### Product Carousel Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "TEMPLATE_NAME",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "Featured Products"
            }
          ]
        },
        {
          "type": "carousel",
          "cards": [
            {
              "card_index": 0,
              "components": [
                {
                  "type": "header",
                  "parameters": [
                    {
                      "type": "product",
                      "product": {
                        "product_retailer_id": "product123",
                        "catalog_id": "catalog456"
                      }
                    }
                  ]
                }
              ]
            },
            {
              "card_index": 1,
              "components": [
                {
                  "type": "header",
                  "parameters": [
                    {
                      "type": "product",
                      "product": {
                        "product_retailer_id": "product789",
                        "catalog_id": "catalog456"
                      }
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
async function sendProductCarouselTemplate(to, templateName, languageCode, bodyParams, products, catalogId) {
    try {
        const cards = products.map((productId, index) => ({
            card_index: index,
            components: [
                {
                    type: "header",
                    parameters: [
                        {
                            type: "product",
                            product: {
                                product_retailer_id: productId,
                                catalog_id: catalogId
                            }
                        }
                    ]
                }
            ]
        }));

        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "template",
                template: {
                    name: templateName,
                    language: {
                        code: languageCode
                    },
                    components: [
                        {
                            type: "body",
                            parameters: bodyParams.map(param => ({
                                type: "text",
                                text: param
                            }))
                        },
                        {
                            type: "carousel",
                            cards: cards
                        }
                    ]
                }
            },
            {
                headers: {
                    'Authorization': this.token,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending product carousel template:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppTemplateManager class
WhatsAppTemplateManager.prototype.sendProductCarouselTemplate = sendProductCarouselTemplate;
```

---

## 8. Flow Template Message

### Flow Template

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "template",
    "template": {
      "name": "flow_template_name",
      "language": {
        "code": "en_US"
      },
      "components": [
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "Complete Registration"
            }
          ]
        },
        {
          "type": "button",
          "sub_type": "flow",
          "index": "0",
          "parameters": [
            {
              "type": "action",
              "action": {
                "flow_token": "FLOW_TOKEN",
                "flow_action_data": {
                  "screen": "WELCOME_SCREEN",
                  "data": {}
                }
              }
            }
          ]
        }
      ]
    }
  }'
```

### Node.js Example

```javascript
async function sendFlowTemplate(to, templateName, languageCode, bodyParams, flowToken, flowActionData) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "template",
                template: {
                    name: templateName,
                    language: {
                        code: languageCode
                    },
                    components: [
                        {
                            type: "body",
                            parameters: bodyParams.map(param => ({
                                type: "text",
                                text: param
                            }))
                        },
                        {
                            type: "button",
                            sub_type: "flow",
                            index: "0",
                            parameters: [
                                {
                                    type: "action",
                                    action: {
                                        flow_token: flowToken,
                                        flow_action_data: flowActionData
                                    }
                                }
                            ]
                        }
                    ]
                }
            },
            {
                headers: {
                    'Authorization': this.token,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending flow template:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppTemplateManager class
WhatsAppTemplateManager.prototype.sendFlowTemplate = sendFlowTemplate;
```

---

## Complete Node.js Template Manager Class

```javascript
const axios = require('axios');

class WhatsAppTemplateManager {
    constructor(appId, token) {
        this.appId = appId;
        this.token = token;
        this.baseURL = 'https://partner.gupshup.io/partner';
    }

    // Base method for sending template messages
    async sendTemplate(to, templateConfig) {
        try {
            const response = await axios.post(
                `${this.baseURL}/app/${this.appId}/v3/message`,
                {
                    messaging_product: "whatsapp",
                    recipient_type: "individual",
                    to: to,
                    type: "template",
                    template: templateConfig
                },
                {
                    headers: {
                        'Authorization': this.token,
                        'Content-Type': 'application/json'
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            console.error('Error sending template:', error.response?.data || error.message);
            throw error;
        }
    }

    // Text template
    async sendTextTemplate(to, templateName, languageCode, parameters) {
        const templateConfig = {
            name: templateName,
            language: { code: languageCode },
            components: [{
                type: "body",
                parameters: parameters.map(param => ({
                    type: "text",
                    text: param
                }))
            }]
        };

        return this.sendTemplate(to, templateConfig);
    }

    // Media template
    async sendMediaTemplate(to, templateName, languageCode, mediaType, mediaUrl, bodyParameters) {
        const templateConfig = {
            name: templateName,
            language: { code: languageCode },
            components: [
                {
                    type: "header",
                    parameters: [{
                        type: mediaType,
                        [mediaType]: { link: mediaUrl }
                    }]
                },
                {
                    type: "body",
                    parameters: bodyParameters.map(param => {
                        if (typeof param === 'string') {
                            return { type: "text", text: param };
                        }
                        return param;
                    })
                }
            ]
        };

        return this.sendTemplate(to, templateConfig);
    }

    // Authentication template
    async sendAuthTemplate(to, templateName, languageCode, otpCode) {
        const templateConfig = {
            name: templateName,
            language: { code: languageCode },
            components: [
                {
                    type: "body",
                    parameters: [{ type: "text", text: otpCode }]
                },
                {
                    type: "button",
                    sub_type: "url",
                    index: "0",
                    parameters: [{ type: "text", text: otpCode }]
                }
            ]
        };

        return this.sendTemplate(to, templateConfig);
    }

    // Location template
    async sendLocationTemplate(to, templateName, languageCode, location, bodyParams) {
        const templateConfig = {
            name: templateName,
            language: { code: languageCode },
            components: [
                {
                    type: "header",
                    parameters: [{
                        type: "location",
                        location: location
                    }]
                },
                {
                    type: "body",
                    parameters: bodyParams.map(param => ({
                        type: "text",
                        text: param
                    }))
                }
            ]
        };

        return this.sendTemplate(to, templateConfig);
    }

    // Error handler
    handleError(error) {
        if (error.response) {
            const { status, data } = error.response;
            switch (status) {
                case 400:
                    return new Error(`Bad Request: ${data.message || 'Invalid parameters'}`);
                case 401:
                    return new Error('Authentication Failed: Invalid token');
                case 403:
                    return new Error('Forbidden: Insufficient permissions');
                case 429:
                    return new Error('Rate limit exceeded');
                case 500:
                    return new Error('Internal server error');
                default:
                    return new Error(`HTTP ${status}: ${data.message || 'Unknown error'}`);
            }
        } else if (error.request) {
            return new Error('Network error - No response received');
        } else {
            return new Error(`Request error: ${error.message}`);
        }
    }
}

// Usage Examples
const templateManager = new WhatsAppTemplateManager('your-app-id', 'your-token');

async function sendTemplateExamples() {
    const recipient = '1234567890';
    
    try {
        // Text template
        await templateManager.sendTextTemplate(
            recipient, 
            'order_confirmation', 
            'en_US', 
            ['John Doe', 'ORDER12345']
        );

        // Media template
        await templateManager.sendMediaTemplate(
            recipient,
            'product_promo',
            'en_US',
            'image',
            'https://example.com/product.jpg',
            ['New Product', { 
                type: 'currency', 
                fallback_value: '$29.99', 
                code: 'USD', 
                amount_1000: 29990 
            }]
        );

        // Authentication template
        await templateManager.sendAuthTemplate(
            recipient,
            'verify_otp',
            'en_US',
            '123456'
        );

        // Location template
        await templateManager.sendLocationTemplate(
            recipient,
            'delivery_update',
            'en_US',
            {
                latitude: '37.4838',
                longitude: '-122.1490',
                name: 'Delivery Location',
                address: '123 Main St, San Francisco, CA'
            },
            ['John Doe', 'ORDER12345']
        );

        console.log('All templates sent successfully!');
    } catch (error) {
        console.error('Error sending templates:', error.message);
    }
}

// Export the class
module.exports = WhatsAppTemplateManager;
```

## Best Practices

### 1. Template Design
- Keep templates concise and focused on the main message
- Use clear, actionable language
- Ensure templates are approved before use

### 2. Parameter Handling
- Validate all parameters before sending
- Use appropriate parameter types (text, currency, date_time)
- Handle missing or invalid parameters gracefully

### 3. Error Handling
- Implement comprehensive error handling
- Log template delivery status
- Use retry logic for transient failures

### 4. Performance
- Batch template messages when possible
- Implement rate limiting to avoid API limits
- Cache template configurations for reuse

### 5. Security
- Never expose authentication tokens in logs
- Validate all input parameters
- Use HTTPS for all media URLs

## Common Response Format

```json
{
    "messages": [
        {
            "id": "GUPSHUP_MESSAGE_ID"
        }
    ],
    "messaging_product": "whatsapp",
    "contacts": [
        {
            "input": "DESTINATION_PHONE_NO",
            "wa_id": "DESTINATION_PHONE_NO"
        }
    ]
}
```

## Status Codes

| Code | Description |
|------|-------------|
| 200 | Template sent successfully |
| 400 | Bad request - Invalid parameters or callback billing not enabled |
| 401 | Unauthorized - Invalid token |
| 403 | Forbidden - Insufficient permissions |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

## Parameter Types Reference

### Text Parameters
```javascript
{
    "type": "text",
    "text": "Your text value"
}
```

### Currency Parameters
```javascript
{
    "type": "currency",
    "currency": {
        "fallback_value": "$19.99",
        "code": "USD",
        "amount_1000": 19990
    }
}
```

### Date/Time Parameters
```javascript
{
    "type": "date_time",
    "date_time": {
        "fallback_value": "January 1, 2024"
    }
}
```

### Location Parameters
```javascript
{
    "type": "location",
    "location": {
        "latitude": "37.4838",
        "longitude": "-122.1490",
        "name": "Location Name",
        "address": "Full Address"
    }
}
```

This comprehensive documentation provides all the necessary information and examples to implement WhatsApp template messaging functionality using the Gupshup Partner API V3.

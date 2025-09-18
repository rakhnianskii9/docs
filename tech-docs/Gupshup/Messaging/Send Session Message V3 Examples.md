# Gupshup Partner API - Send Session Message V3 Examples

## Overview
The Send Session Message V3 API allows you to send various types of messages to WhatsApp users within an active session. This API supports multiple message formats including text, media, interactive, and specialized message types.

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
| type | Message type | String | Required | Type of message (text, image, video, etc.) |

## Common Request Headers

```javascript
{
    "Authorization": "YOUR_PARTNER_APP_TOKEN",
    "Content-Type": "application/json"
}
```

## Message Types Overview

| Message Type | Description |
|--------------|-------------|
| **Address Message** | Address messages give users a simpler way to share the shipping address with the business on WhatsApp |
| **Audio Message** | Displays an audio icon, and plays when the WhatsApp user taps it |
| **Contact Message** | Allows users to send rich contact information directly to WhatsApp users |
| **Flow Message** | User can send a flow message after creating a WhatsApp flow |
| **Image Message** | Send an image message to a WhatsApp user |
| **Interactive Message** | Send interactive messages with buttons, lists, or quick replies |
| **Reaction Message** | Send a reaction message to a WhatsApp user |
| **Sticker Message** | Send sticker messages |
| **Text Message** | Send a text message to a WhatsApp user |
| **Video Message** | Send a video message to a WhatsApp user |

---

## 1. Text Message

### Basic Text Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "text",
    "text": {
      "body": "Hello! This is a text message."
    }
  }'
```

### Node.js Examples

#### Basic Text Message

```javascript
const axios = require('axios');

class WhatsAppMessageSender {
    constructor(appId, token) {
        this.appId = appId;
        this.token = token;
        this.baseURL = 'https://partner.gupshup.io/partner';
    }

    async sendTextMessage(to, text) {
        try {
            const response = await axios.post(
                `${this.baseURL}/app/${this.appId}/v3/message`,
                {
                    messaging_product: "whatsapp",
                    recipient_type: "individual",
                    to: to,
                    type: "text",
                    text: {
                        body: text
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
            console.error('Error sending text message:', error.response?.data || error.message);
            throw error;
        }
    }
}

// Usage
const messageSender = new WhatsAppMessageSender('your-app-id', 'your-token');
messageSender.sendTextMessage('1234567890', 'Hello from WhatsApp!')
    .then(result => console.log('Message sent:', result))
    .catch(error => console.error('Error:', error));
```

---

## 2. Image Message

### Basic Image Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "image",
    "image": {
      "id": "MEDIA_ID",
      "link": "https://example.com/image.jpg",
      "caption": "Check out this image!"
    }
  }'
```

### Node.js Example

```javascript
async function sendImageMessage(to, imageUrl, caption = '', mediaId = null) {
    try {
        const imagePayload = mediaId 
            ? { id: mediaId, caption: caption }
            : { link: imageUrl, caption: caption };
            
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "image",
                image: imagePayload
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
        console.error('Error sending image message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendImageMessage = sendImageMessage;
```

---

## 3. Video Message

### Basic Video Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "video",
    "video": {
      "id": "MEDIA_ID",
      "link": "https://example.com/video.mp4",
      "caption": "Watch this amazing video!"
    }
  }'
```

### Node.js Example

```javascript
async function sendVideoMessage(to, videoUrl, caption = '', mediaId = null) {
    try {
        const videoPayload = mediaId 
            ? { id: mediaId, caption: caption }
            : { link: videoUrl, caption: caption };
            
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "video",
                video: videoPayload
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
        console.error('Error sending video message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendVideoMessage = sendVideoMessage;
```

---

## 4. Audio Message

### Basic Audio Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "audio",
    "audio": {
      "id": "MEDIA_ID"
    }
  }'
```

### Node.js Example

```javascript
async function sendAudioMessage(to, audioUrl, mediaId = null) {
    try {
        const audioPayload = mediaId 
            ? { id: mediaId }
            : { link: audioUrl };
            
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "audio",
                audio: audioPayload
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
        console.error('Error sending audio message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendAudioMessage = sendAudioMessage;
```

---

## 5. Interactive Message

### Button Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "interactive",
    "interactive": {
      "type": "button",
      "header": {
        "type": "text",
        "text": "Header Text"
      },
      "body": {
        "text": "Please choose an option:"
      },
      "footer": {
        "text": "Footer Text"
      },
      "action": {
        "buttons": [
          {
            "type": "reply",
            "reply": {
              "id": "option_1",
              "title": "Option 1"
            }
          },
          {
            "type": "reply",
            "reply": {
              "id": "option_2",
              "title": "Option 2"
            }
          }
        ]
      }
    }
  }'
```

### List Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "interactive",
    "interactive": {
      "type": "list",
      "header": {
        "type": "text",
        "text": "Choose from menu"
      },
      "body": {
        "text": "Select an option from the list below:"
      },
      "footer": {
        "text": "Powered by Our Service"
      },
      "action": {
        "button": "View Options",
        "sections": [
          {
            "title": "Main Options",
            "rows": [
              {
                "id": "option_1",
                "title": "Option 1",
                "description": "Description for option 1"
              },
              {
                "id": "option_2",
                "title": "Option 2",
                "description": "Description for option 2"
              }
            ]
          }
        ]
      }
    }
  }'
```

### Node.js Examples

```javascript
// Button Message
async function sendButtonMessage(to, bodyText, buttons, headerText = '', footerText = '') {
    try {
        const interactive = {
            type: "button",
            body: {
                text: bodyText
            },
            action: {
                buttons: buttons.map((button, index) => ({
                    type: "reply",
                    reply: {
                        id: button.id || `button_${index}`,
                        title: button.title
                    }
                }))
            }
        };

        if (headerText) {
            interactive.header = {
                type: "text",
                text: headerText
            };
        }

        if (footerText) {
            interactive.footer = {
                text: footerText
            };
        }

        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "interactive",
                interactive: interactive
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
        console.error('Error sending button message:', error.response?.data || error.message);
        throw error;
    }
}

// List Message
async function sendListMessage(to, headerText, bodyText, footerText, buttonText, sections) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "interactive",
                interactive: {
                    type: "list",
                    header: {
                        type: "text",
                        text: headerText
                    },
                    body: {
                        text: bodyText
                    },
                    footer: {
                        text: footerText
                    },
                    action: {
                        button: buttonText,
                        sections: sections
                    }
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
        console.error('Error sending list message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendButtonMessage = sendButtonMessage;
WhatsAppMessageSender.prototype.sendListMessage = sendListMessage;

// Usage examples
const buttons = [
    { id: 'yes', title: 'Yes' },
    { id: 'no', title: 'No' }
];

const sections = [
    {
        title: "Menu Items",
        rows: [
            { id: "pizza", title: "Pizza", description: "Delicious pizza" },
            { id: "burger", title: "Burger", description: "Juicy burger" }
        ]
    }
];

messageSender.sendButtonMessage('1234567890', 'Do you want to continue?', buttons);
messageSender.sendListMessage('1234567890', 'Our Menu', 'Choose your favorite:', 'Thank you!', 'View Menu', sections);
```

---

## 6. Contact Message

### Basic Contact Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "to": "PHONE_NUMBER",
    "type": "contacts",
    "contacts": [
      {
        "addresses": [
          {
            "street": "123 Main St",
            "city": "New York",
            "state": "NY",
            "zip": "10001",
            "country": "USA",
            "country_code": "US",
            "type": "WORK"
          }
        ],
        "birthday": "1990-01-01",
        "emails": [
          {
            "email": "john@example.com",
            "type": "WORK"
          }
        ],
        "name": {
          "formatted_name": "John Doe",
          "first_name": "John",
          "last_name": "Doe"
        },
        "org": {
          "company": "Example Corp",
          "department": "Engineering",
          "title": "Software Engineer"
        },
        "phones": [
          {
            "phone": "+1234567890",
            "type": "WORK"
          }
        ],
        "urls": [
          {
            "url": "https://example.com",
            "type": "WORK"
          }
        ]
      }
    ]
  }'
```

### Node.js Example

```javascript
async function sendContactMessage(to, contactInfo) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                to: to,
                type: "contacts",
                contacts: [contactInfo]
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
        console.error('Error sending contact message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendContactMessage = sendContactMessage;

// Usage
const contactInfo = {
    addresses: [
        {
            street: "123 Main St",
            city: "New York",
            state: "NY",
            zip: "10001",
            country: "USA",
            country_code: "US",
            type: "WORK"
        }
    ],
    emails: [
        {
            email: "john@example.com",
            type: "WORK"
        }
    ],
    name: {
        formatted_name: "John Doe",
        first_name: "John",
        last_name: "Doe"
    },
    phones: [
        {
            phone: "+1234567890",
            type: "WORK"
        }
    ]
};

messageSender.sendContactMessage('1234567890', contactInfo);
```

---

## 7. Reaction Message

### Basic Reaction Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "reaction",
    "reaction": {
      "message_id": "MESSAGE_ID_TO_REACT_TO",
      "emoji": "👍"
    }
  }'
```

### Node.js Example

```javascript
async function sendReactionMessage(to, messageId, emoji) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "reaction",
                reaction: {
                    message_id: messageId,
                    emoji: emoji
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
        console.error('Error sending reaction message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendReactionMessage = sendReactionMessage;

// Usage
messageSender.sendReactionMessage('1234567890', 'msg_id_123', '❤️');
```

---

## 8. Sticker Message

### Basic Sticker Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "sticker",
    "sticker": {
      "id": "MEDIA_ID"
    }
  }'
```

### Node.js Example

```javascript
async function sendStickerMessage(to, mediaId, stickerUrl = null) {
    try {
        const stickerPayload = mediaId 
            ? { id: mediaId }
            : { link: stickerUrl };
            
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "sticker",
                sticker: stickerPayload
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
        console.error('Error sending sticker message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendStickerMessage = sendStickerMessage;

// Usage
messageSender.sendStickerMessage('1234567890', 'https://example.com/sticker.webp');
```

---

## 9. Address Message

### Basic Address Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/v3/message" \
  -H "Authorization: YOUR_PARTNER_APP_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "interactive",
    "interactive": {
      "type": "address_message",
      "body": {
        "text": "Please share your shipping address"
      },
      "action": {
        "name": "address_message",
        "parameters": {
          "country": "IN",
          "values": {
            "name": "CUSTOMER_NAME",
            "phone_number": "+91xxxxxxxxxx"
          }
        }
      }
    }
  }'
```

### Node.js Example

```javascript
async function sendAddressMessage(to, bodyText, country = 'IN', values = {}) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/v3/message`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "interactive",
                interactive: {
                    type: "address_message",
                    body: {
                        text: bodyText
                    },
                    action: {
                        name: "address_message",
                        parameters: {
                            country: country,
                            values: values
                        }
                    }
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
        console.error('Error sending address message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendAddressMessage = sendAddressMessage;

// Usage
messageSender.sendAddressMessage('1234567890', 'Please provide your shipping address', 'IN', {
    name: "Customer Name",
    phone_number: "+91xxxxxxxxxx"
});
```

---

## 10. Flow Message

### Basic Flow Message

```bash
curl -X POST "https://partner.gupshup.io/partner/app/{appId}/msg" \
  -H "Authorization: Bearer YOUR_PARTNER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "recipient_type": "individual",
    "to": "PHONE_NUMBER",
    "type": "interactive",
    "interactive": {
      "type": "flow",
      "header": {
        "type": "text",
        "text": "Flow Header"
      },
      "body": {
        "text": "Please complete the flow"
      },
      "footer": {
        "text": "Flow Footer"
      },
      "action": {
        "name": "flow",
        "parameters": {
          "flow_message_version": "3",
          "flow_token": "FLOW_TOKEN",
          "flow_id": "FLOW_ID",
          "flow_cta": "Start Flow",
          "flow_action": "navigate",
          "flow_action_payload": {
            "screen": "WELCOME_SCREEN"
          }
        }
      }
    }
  }'
```

### Node.js Example

```javascript
async function sendFlowMessage(to, flowConfig) {
    try {
        const response = await axios.post(
            `${this.baseURL}/app/${this.appId}/msg`,
            {
                messaging_product: "whatsapp",
                recipient_type: "individual",
                to: to,
                type: "interactive",
                interactive: {
                    type: "flow",
                    header: {
                        type: "text",
                        text: flowConfig.headerText
                    },
                    body: {
                        text: flowConfig.bodyText
                    },
                    footer: {
                        text: flowConfig.footerText
                    },
                    action: {
                        name: "flow",
                        parameters: {
                            flow_message_version: "3",
                            flow_token: flowConfig.flowToken,
                            flow_id: flowConfig.flowId,
                            flow_cta: flowConfig.flowCta,
                            flow_action: flowConfig.flowAction || "navigate",
                            flow_action_payload: flowConfig.flowActionPayload || {}
                        }
                    }
                }
            },
            {
                headers: {
                    'Authorization': `Bearer ${this.token}`,
                    'Content-Type': 'application/json'
                }
            }
        );
        
        return response.data;
    } catch (error) {
        console.error('Error sending flow message:', error.response?.data || error.message);
        throw error;
    }
}

// Add to WhatsAppMessageSender class
WhatsAppMessageSender.prototype.sendFlowMessage = sendFlowMessage;

// Usage
const flowConfig = {
    headerText: "Welcome to our service",
    bodyText: "Please complete the registration flow",
    footerText: "This will only take a minute",
    flowToken: "your-flow-token",
    flowId: "your-flow-id",
    flowCta: "Start Registration",
    flowAction: "navigate",
    flowActionPayload: {
        screen: "REGISTRATION_SCREEN"
    }
};

messageSender.sendFlowMessage('1234567890', flowConfig);
```

---

## Complete Node.js Manager Class

```javascript
const axios = require('axios');

class WhatsAppMessageManager {
    constructor(appId, token) {
        this.appId = appId;
        this.token = token;
        this.baseURL = 'https://partner.gupshup.io/partner';
        this.axiosInstance = axios.create({
            baseURL: this.baseURL,
            timeout: 10000,
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json'
            }
        });
    }

    async sendMessage(to, messageData) {
        try {
            const response = await this.axiosInstance.post(
                `/app/${this.appId}/msg`,
                {
                    messaging_product: "whatsapp",
                    recipient_type: "individual",
                    to: to,
                    ...messageData
                }
            );
            
            return {
                success: true,
                data: response.data,
                timestamp: new Date().toISOString()
            };
        } catch (error) {
            console.error('Error sending message:', error.response?.data || error.message);
            throw this.handleError(error);
        }
    }

    // Text Message
    async sendTextMessage(to, text) {
        return this.sendMessage(to, {
            type: "text",
            text: { body: text }
        });
    }

    // Image Message
    async sendImageMessage(to, imageUrl, caption = '') {
        return this.sendMessage(to, {
            type: "image",
            image: {
                link: imageUrl,
                caption: caption
            }
        });
    }

    // Video Message
    async sendVideoMessage(to, videoUrl, caption = '') {
        return this.sendMessage(to, {
            type: "video",
            video: {
                link: videoUrl,
                caption: caption
            }
        });
    }

    // Audio Message
    async sendAudioMessage(to, audioUrl) {
        return this.sendMessage(to, {
            type: "audio",
            audio: { link: audioUrl }
        });
    }

    // Button Message
    async sendButtonMessage(to, bodyText, buttons) {
        return this.sendMessage(to, {
            type: "interactive",
            interactive: {
                type: "button",
                body: { text: bodyText },
                action: {
                    buttons: buttons.map((button, index) => ({
                        type: "reply",
                        reply: {
                            id: button.id || `button_${index}`,
                            title: button.title
                        }
                    }))
                }
            }
        });
    }

    // List Message
    async sendListMessage(to, headerText, bodyText, footerText, buttonText, sections) {
        return this.sendMessage(to, {
            type: "interactive",
            interactive: {
                type: "list",
                header: { type: "text", text: headerText },
                body: { text: bodyText },
                footer: { text: footerText },
                action: {
                    button: buttonText,
                    sections: sections
                }
            }
        });
    }

    // Contact Message
    async sendContactMessage(to, contactInfo) {
        return this.sendMessage(to, {
            type: "contacts",
            contacts: [contactInfo]
        });
    }

    // Reaction Message
    async sendReactionMessage(to, messageId, emoji) {
        return this.sendMessage(to, {
            type: "reaction",
            reaction: {
                message_id: messageId,
                emoji: emoji
            }
        });
    }

    // Sticker Message
    async sendStickerMessage(to, stickerUrl) {
        return this.sendMessage(to, {
            type: "sticker",
            sticker: { link: stickerUrl }
        });
    }

    // Address Message
    async sendAddressMessage(to, bodyText, country = 'US', values = {}) {
        return this.sendMessage(to, {
            type: "interactive",
            interactive: {
                type: "address_message",
                body: { text: bodyText },
                action: {
                    name: "address_message",
                    parameters: {
                        country: country,
                        values: values
                    }
                }
            }
        });
    }

    // Flow Message
    async sendFlowMessage(to, flowConfig) {
        return this.sendMessage(to, {
            type: "interactive",
            interactive: {
                type: "flow",
                header: { type: "text", text: flowConfig.headerText },
                body: { text: flowConfig.bodyText },
                footer: { text: flowConfig.footerText },
                action: {
                    name: "flow",
                    parameters: {
                        flow_message_version: "3",
                        flow_token: flowConfig.flowToken,
                        flow_id: flowConfig.flowId,
                        flow_cta: flowConfig.flowCta,
                        flow_action: flowConfig.flowAction || "navigate",
                        flow_action_payload: flowConfig.flowActionPayload || {}
                    }
                }
            }
        });
    }

    // Bulk send messages
    async sendBulkMessages(recipients, messageData) {
        const results = [];
        
        for (const recipient of recipients) {
            try {
                const result = await this.sendMessage(recipient, messageData);
                results.push({
                    recipient,
                    success: true,
                    result: result.data
                });
            } catch (error) {
                results.push({
                    recipient,
                    success: false,
                    error: error.message
                });
            }
            
            // Add delay to avoid rate limiting
            await new Promise(resolve => setTimeout(resolve, 1000));
        }
        
        return results;
    }

    // Error handling
    handleError(error) {
        if (error.response) {
            const { status, data } = error.response;
            
            switch (status) {
                case 400:
                    return new Error(`Bad Request: ${data.message || 'Invalid request parameters'}`);
                case 401:
                    return new Error('Unauthorized - Invalid or expired token');
                case 403:
                    return new Error('Forbidden - Insufficient permissions');
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

    // Message template builder
    buildMessageTemplate(type, data) {
        const templates = {
            text: (text) => ({ type: "text", text: { body: text } }),
            image: (url, caption) => ({ type: "image", image: { link: url, caption: caption || '' } }),
            video: (url, caption) => ({ type: "video", video: { link: url, caption: caption || '' } }),
            audio: (url) => ({ type: "audio", audio: { link: url } }),
            sticker: (url) => ({ type: "sticker", sticker: { link: url } }),
            reaction: (messageId, emoji) => ({ type: "reaction", reaction: { message_id: messageId, emoji } })
        };

        return templates[type] ? templates[type](...data) : null;
    }
}

// Usage Examples
const messageManager = new WhatsAppMessageManager('your-app-id', 'your-token');

// Send different types of messages
async function sendMessageExamples() {
    const recipient = '1234567890';
    
    try {
        // Text message
        await messageManager.sendTextMessage(recipient, 'Hello from WhatsApp!');
        
        // Image message
        await messageManager.sendImageMessage(recipient, 'https://example.com/image.jpg', 'Check this out!');
        
        // Button message
        const buttons = [
            { id: 'yes', title: 'Yes' },
            { id: 'no', title: 'No' }
        ];
        await messageManager.sendButtonMessage(recipient, 'Do you agree?', buttons);
        
        // List message
        const sections = [
            {
                title: "Options",
                rows: [
                    { id: "opt1", title: "Option 1", description: "First option" },
                    { id: "opt2", title: "Option 2", description: "Second option" }
                ]
            }
        ];
        await messageManager.sendListMessage(recipient, 'Menu', 'Choose an option:', 'Thank you!', 'View Options', sections);
        
        console.log('All messages sent successfully!');
    } catch (error) {
        console.error('Error sending messages:', error.message);
    }
}

// Export the class
module.exports = WhatsAppMessageManager;
```

## Best Practices

### 1. Message Formatting
- Keep text messages concise and clear
- Use proper formatting for interactive messages
- Ensure media URLs are accessible and properly formatted

### 2. Error Handling
- Always implement proper error handling
- Use retry logic for transient failures
- Log message delivery status for monitoring

### 3. Rate Limiting
- Implement delays between bulk messages
- Monitor API usage to avoid rate limits
- Use exponential backoff for retries

### 4. Security
- Validate all input parameters
- Never expose authentication tokens in logs
- Use HTTPS for all media URLs

### 5. User Experience
- Provide clear instructions in interactive messages
- Use appropriate message types for different scenarios
- Test messages thoroughly before production use

## Common Response Format

```json
{
    "messaging_product": "whatsapp",
    "contacts": [
        {
            "input": "PHONE_NUMBER",
            "wa_id": "WHATSAPP_ID"
        }
    ],
    "messages": [
        {
            "id": "MESSAGE_ID"
        }
    ]
}
```

## Status Codes

| Code | Description |
|------|-------------|
| 200 | Message sent successfully |
| 400 | Bad request - Invalid parameters |
| 401 | Unauthorized - Invalid token |
| 403 | Forbidden - Insufficient permissions |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

This comprehensive documentation provides all the necessary information and examples to implement WhatsApp messaging functionality using the Gupshup Partner API V3.
# Conversational Component API

The Conversational Component API allows you to configure and manage conversational automation features for your WhatsApp Business applications. This includes setting up welcome messages, commands, and ice breakers (prompts) to enhance user engagement.

## API Overview

| Feature | Description |
|---------|-------------|
| **Base URL** | `https://partner.gupshup.io/partner/app/{appId}/conversational/component` |
| **Authentication** | Bearer Token (Partner App Token) |
| **Content Type** | `application/json` |

## Authentication

All requests require a valid Partner App Token in the Authorization header:

```bash
Authorization: Bearer YOUR_PARTNER_APP_TOKEN
```

## Available Endpoints

### 1. Get Conversational Component

Retrieve the current conversational automation configuration for your app.

**Endpoint:** `GET /partner/app/{appId}/conversational/component`

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `appId` | string | Yes | App ID of the account |

#### Sample Request (cURL)

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/YOUR_APP_ID/conversational/component' \
--header 'Authorization: Bearer YOUR_PARTNER_APP_TOKEN'
```

#### Sample Response

```json
{
  "conversational_automation": {
    "prompts": [
      "Book a flight",
      "plan a vacation"
    ],
    "commands": [
      {
        "command_name": "tickets",
        "command_description": "Book flight tickets"
      },
      {
        "command_name": "hotel",
        "command_description": "Book hotel"
      }
    ],
    "enable_welcome_message": false,
    "id": "{id}"
  },
  "id": "{phone_id}"
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `conversational_automation` | object | Main automation configuration |
| `conversational_automation.prompts` | array | List of ice breakers (up to 4) |
| `conversational_automation.commands` | array | List of commands (up to 30) |
| `conversational_automation.commands[].command_name` | string | Command name (max 32 characters) |
| `conversational_automation.commands[].command_description` | string | Command description (max 256 characters) |
| `conversational_automation.enable_welcome_message` | boolean | Whether welcome message is enabled |
| `conversational_automation.id` | string | Automation configuration ID |
| `id` | string | Phone ID |

### 2. Set Conversational Component

Configure or update the conversational automation settings for your app.

**Endpoint:** `POST /partner/app/{appId}/conversational/component`

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `appId` | string | Yes | App ID of the account |
| `enable_welcome_message` | boolean | Yes | Enable/disable welcome message events |
| `commands` | array | Yes | List of command objects (max 30) |
| `commands[].command_name` | string | Yes | Command name (max 32 characters) |
| `commands[].command_description` | string | Yes | Command description (max 256 characters) |
| `prompts` | array | Yes | List of ice breakers (max 4, max 80 characters each) |

#### Constraints

- **Commands**: Maximum 30 commands
- **Command Name**: Maximum 32 characters
- **Command Description**: Maximum 256 characters
- **Prompts**: Maximum 4 ice breakers
- **Prompt Length**: Maximum 80 characters each
- **Emojis**: Not supported in commands or prompts

#### Sample Request (cURL)

```bash
curl --location --request POST 'https://partner.gupshup.io/partner/app/YOUR_APP_ID/conversational/component' \
--header 'Authorization: Bearer YOUR_PARTNER_APP_TOKEN' \
--header 'Content-Type: application/json' \
--data-raw '{
    "enable_welcome_message": true,
    "commands": [
        {
            "command_name": "tickets",
            "command_description": "Book flight tickets"
        },
        {
            "command_name": "hotel",
            "command_description": "Book hotel"
        }
    ],
    "prompts": [
        "Book a flight",
        "plan a vacation"
    ]
}'
```

#### Sample Response

```json
{
  "status": "success",
  "success": true
}
```

## Status Codes

| Status | Response | Description |
|--------|----------|-------------|
| **Success** | | |
| 200 | `{"status": "success", "success": true}` | Configuration updated successfully |
| 200 | `{"conversational_automation": {...}}` | Configuration retrieved successfully |
| **Error** | | |
| 400 | `{"error": {"code": 105, "message": "The parameter prompts must have at most 4 elements..."}}` | Too many ice breakers (max 4) |
| 400 | `{"error": {"code": 100, "message": "Param commands[x]['command_name'] must be at most 32 characters long."}}` | Command name too long (max 32 chars) |
| 401 | `{"message": "Authentication Failed", "status": "error"}` | Invalid or missing token |

## Node.js Examples

### Basic Usage

```javascript
const axios = require('axios');

class ConversationalComponentManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseURL = 'https://partner.gupshup.io/partner/app';
  }

  // Get conversational component configuration
  async getConversationalComponent(appId) {
    try {
      const response = await axios.get(
        `${this.baseURL}/${appId}/conversational/component`,
        {
          headers: {
            'Authorization': `Bearer ${this.partnerAppToken}`
          }
        }
      );

      return response.data;
    } catch (error) {
      console.error('Error fetching conversational component:', error.response?.data || error.message);
      throw error;
    }
  }

  // Set conversational component configuration
  async setConversationalComponent(appId, config) {
    try {
      const response = await axios.post(
        `${this.baseURL}/${appId}/conversational/component`,
        config,
        {
          headers: {
            'Authorization': `Bearer ${this.partnerAppToken}`,
            'Content-Type': 'application/json'
          }
        }
      );

      return response.data;
    } catch (error) {
      console.error('Error setting conversational component:', error.response?.data || error.message);
      throw error;
    }
  }
}

// Usage example
const componentManager = new ConversationalComponentManager('YOUR_PARTNER_APP_TOKEN');

// Get current configuration
async function getCurrentConfig() {
  try {
    const config = await componentManager.getConversationalComponent('YOUR_APP_ID');
    console.log('Current configuration:', config);
    return config;
  } catch (error) {
    console.error('Failed to get configuration:', error);
  }
}

// Set new configuration
async function setNewConfig() {
  try {
    const newConfig = {
      enable_welcome_message: true,
      commands: [
        {
          command_name: "help",
          command_description: "Get help and support"
        },
        {
          command_name: "products",
          command_description: "Browse our product catalog"
        },
        {
          command_name: "orders",
          command_description: "Track your orders"
        }
      ],
      prompts: [
        "How can I help you?",
        "Browse products",
        "Check order status",
        "Get support"
      ]
    };

    const result = await componentManager.setConversationalComponent('YOUR_APP_ID', newConfig);
    console.log('Configuration updated:', result);
    return result;
  } catch (error) {
    console.error('Failed to set configuration:', error);
  }
}
```

### Advanced Usage with Validation

```javascript
class AdvancedConversationalComponentManager extends ConversationalComponentManager {
  // Validate configuration before setting
  validateConfig(config) {
    const errors = [];

    // Validate enable_welcome_message
    if (typeof config.enable_welcome_message !== 'boolean') {
      errors.push('enable_welcome_message must be a boolean');
    }

    // Validate commands
    if (!Array.isArray(config.commands)) {
      errors.push('commands must be an array');
    } else {
      if (config.commands.length > 30) {
        errors.push('Maximum 30 commands allowed');
      }

      config.commands.forEach((command, index) => {
        if (!command.command_name || typeof command.command_name !== 'string') {
          errors.push(`Command ${index}: command_name is required and must be a string`);
        } else if (command.command_name.length > 32) {
          errors.push(`Command ${index}: command_name must be 32 characters or less`);
        }

        if (!command.command_description || typeof command.command_description !== 'string') {
          errors.push(`Command ${index}: command_description is required and must be a string`);
        } else if (command.command_description.length > 256) {
          errors.push(`Command ${index}: command_description must be 256 characters or less`);
        }

        // Check for emojis (basic check)
        const emojiRegex = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]/gu;
        if (emojiRegex.test(command.command_name) || emojiRegex.test(command.command_description)) {
          errors.push(`Command ${index}: Emojis are not supported`);
        }
      });
    }

    // Validate prompts
    if (!Array.isArray(config.prompts)) {
      errors.push('prompts must be an array');
    } else {
      if (config.prompts.length > 4) {
        errors.push('Maximum 4 prompts allowed');
      }

      config.prompts.forEach((prompt, index) => {
        if (typeof prompt !== 'string') {
          errors.push(`Prompt ${index}: must be a string`);
        } else if (prompt.length > 80) {
          errors.push(`Prompt ${index}: must be 80 characters or less`);
        }

        // Check for emojis
        const emojiRegex = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]/gu;
        if (emojiRegex.test(prompt)) {
          errors.push(`Prompt ${index}: Emojis are not supported`);
        }
      });
    }

    return errors;
  }

  // Set configuration with validation
  async setValidatedConfig(appId, config) {
    try {
      // Validate configuration
      const errors = this.validateConfig(config);
      if (errors.length > 0) {
        throw new Error(`Configuration validation failed: ${errors.join(', ')}`);
      }

      // Set configuration
      const result = await this.setConversationalComponent(appId, config);
      console.log('Configuration set successfully with validation');
      return result;
    } catch (error) {
      console.error('Failed to set validated configuration:', error.message);
      throw error;
    }
  }

  // Update only specific parts of configuration
  async updatePartialConfig(appId, updates) {
    try {
      // Get current configuration
      const currentConfig = await this.getConversationalComponent(appId);
      
      if (!currentConfig.conversational_automation) {
        throw new Error('No existing configuration found');
      }

      // Merge updates with current configuration
      const updatedConfig = {
        enable_welcome_message: updates.enable_welcome_message !== undefined 
          ? updates.enable_welcome_message 
          : currentConfig.conversational_automation.enable_welcome_message,
        commands: updates.commands || currentConfig.conversational_automation.commands,
        prompts: updates.prompts || currentConfig.conversational_automation.prompts
      };

      // Set updated configuration
      return await this.setValidatedConfig(appId, updatedConfig);
    } catch (error) {
      console.error('Failed to update partial configuration:', error);
      throw error;
    }
  }

  // Add new command to existing configuration
  async addCommand(appId, newCommand) {
    try {
      const currentConfig = await this.getConversationalComponent(appId);
      
      if (!currentConfig.conversational_automation) {
        throw new Error('No existing configuration found');
      }

      const commands = [...currentConfig.conversational_automation.commands, newCommand];
      
      if (commands.length > 30) {
        throw new Error('Cannot add command: Maximum 30 commands allowed');
      }

      return await this.updatePartialConfig(appId, { commands });
    } catch (error) {
      console.error('Failed to add command:', error);
      throw error;
    }
  }

  // Remove command by name
  async removeCommand(appId, commandName) {
    try {
      const currentConfig = await this.getConversationalComponent(appId);
      
      if (!currentConfig.conversational_automation) {
        throw new Error('No existing configuration found');
      }

      const commands = currentConfig.conversational_automation.commands.filter(
        cmd => cmd.command_name !== commandName
      );

      return await this.updatePartialConfig(appId, { commands });
    } catch (error) {
      console.error('Failed to remove command:', error);
      throw error;
    }
  }

  // Add new prompt to existing configuration
  async addPrompt(appId, newPrompt) {
    try {
      const currentConfig = await this.getConversationalComponent(appId);
      
      if (!currentConfig.conversational_automation) {
        throw new Error('No existing configuration found');
      }

      const prompts = [...currentConfig.conversational_automation.prompts, newPrompt];
      
      if (prompts.length > 4) {
        throw new Error('Cannot add prompt: Maximum 4 prompts allowed');
      }

      return await this.updatePartialConfig(appId, { prompts });
    } catch (error) {
      console.error('Failed to add prompt:', error);
      throw error;
    }
  }
}

// Usage examples
const advancedManager = new AdvancedConversationalComponentManager('YOUR_PARTNER_APP_TOKEN');

async function demonstrateAdvancedFeatures() {
  try {
    // Set configuration with validation
    const config = {
      enable_welcome_message: true,
      commands: [
        {
          command_name: "support",
          command_description: "Get customer support"
        },
        {
          command_name: "catalog",
          command_description: "Browse product catalog"
        }
      ],
      prompts: [
        "How can I help you today?",
        "Browse products",
        "Get support"
      ]
    };

    await advancedManager.setValidatedConfig('YOUR_APP_ID', config);
    console.log('Configuration set with validation');

    // Add a new command
    await advancedManager.addCommand('YOUR_APP_ID', {
      command_name: "tracking",
      command_description: "Track your order"
    });
    console.log('Command added successfully');

    // Add a new prompt
    await advancedManager.addPrompt('YOUR_APP_ID', "Track order");
    console.log('Prompt added successfully');

    // Remove a command
    await advancedManager.removeCommand('YOUR_APP_ID', "support");
    console.log('Command removed successfully');

    // Update welcome message setting
    await advancedManager.updatePartialConfig('YOUR_APP_ID', {
      enable_welcome_message: false
    });
    console.log('Welcome message disabled');

  } catch (error) {
    console.error('Advanced features demonstration failed:', error);
  }
}
```

### Configuration Templates

```javascript
class ConfigurationTemplates {
  // E-commerce configuration template
  static getEcommerceConfig() {
    return {
      enable_welcome_message: true,
      commands: [
        {
          command_name: "products",
          command_description: "Browse our product catalog"
        },
        {
          command_name: "orders",
          command_description: "Track your orders"
        },
        {
          command_name: "cart",
          command_description: "View your shopping cart"
        },
        {
          command_name: "support",
          command_description: "Get customer support"
        },
        {
          command_name: "returns",
          command_description: "Return or exchange items"
        }
      ],
      prompts: [
        "Shop now",
        "Track my order",
        "Need help?",
        "Browse deals"
      ]
    };
  }

  // Customer service configuration template
  static getCustomerServiceConfig() {
    return {
      enable_welcome_message: true,
      commands: [
        {
          command_name: "help",
          command_description: "Get help and support"
        },
        {
          command_name: "faq",
          command_description: "View frequently asked questions"
        },
        {
          command_name: "contact",
          command_description: "Contact support team"
        },
        {
          command_name: "feedback",
          command_description: "Provide feedback"
        }
      ],
      prompts: [
        "How can I help you?",
        "View FAQ",
        "Contact support",
        "Give feedback"
      ]
    };
  }

  // Restaurant configuration template
  static getRestaurantConfig() {
    return {
      enable_welcome_message: true,
      commands: [
        {
          command_name: "menu",
          command_description: "View our menu"
        },
        {
          command_name: "order",
          command_description: "Place an order"
        },
        {
          command_name: "reservations",
          command_description: "Make a reservation"
        },
        {
          command_name: "delivery",
          command_description: "Check delivery options"
        },
        {
          command_name: "specials",
          command_description: "Today's specials"
        }
      ],
      prompts: [
        "View menu",
        "Order food",
        "Make reservation",
        "Check specials"
      ]
    };
  }

  // Healthcare configuration template
  static getHealthcareConfig() {
    return {
      enable_welcome_message: true,
      commands: [
        {
          command_name: "appointment",
          command_description: "Schedule an appointment"
        },
        {
          command_name: "symptoms",
          command_description: "Check symptoms"
        },
        {
          command_name: "prescriptions",
          command_description: "Manage prescriptions"
        },
        {
          command_name: "emergency",
          command_description: "Emergency contact"
        }
      ],
      prompts: [
        "Schedule appointment",
        "Check symptoms",
        "Refill prescription",
        "Emergency help"
      ]
    };
  }
}

// Usage example with templates
async function setTemplateConfiguration() {
  try {
    const manager = new AdvancedConversationalComponentManager('YOUR_PARTNER_APP_TOKEN');
    
    // Set e-commerce configuration
    const ecommerceConfig = ConfigurationTemplates.getEcommerceConfig();
    await manager.setValidatedConfig('YOUR_APP_ID', ecommerceConfig);
    console.log('E-commerce configuration applied');

    // Or set customer service configuration
    // const serviceConfig = ConfigurationTemplates.getCustomerServiceConfig();
    // await manager.setValidatedConfig('YOUR_APP_ID', serviceConfig);
    // console.log('Customer service configuration applied');

  } catch (error) {
    console.error('Failed to set template configuration:', error);
  }
}
```

### Configuration Backup and Restore

```javascript
class ConfigurationBackupManager {
  constructor(componentManager) {
    this.componentManager = componentManager;
    this.backups = new Map();
  }

  // Backup current configuration
  async backupConfiguration(appId, backupName) {
    try {
      const config = await this.componentManager.getConversationalComponent(appId);
      this.backups.set(backupName, {
        config: config.conversational_automation,
        timestamp: Date.now(),
        appId
      });
      console.log(`Configuration backed up as: ${backupName}`);
      return true;
    } catch (error) {
      console.error('Failed to backup configuration:', error);
      throw error;
    }
  }

  // Restore configuration from backup
  async restoreConfiguration(appId, backupName) {
    try {
      const backup = this.backups.get(backupName);
      if (!backup) {
        throw new Error(`Backup not found: ${backupName}`);
      }

      const restoreConfig = {
        enable_welcome_message: backup.config.enable_welcome_message,
        commands: backup.config.commands,
        prompts: backup.config.prompts
      };

      await this.componentManager.setConversationalComponent(appId, restoreConfig);
      console.log(`Configuration restored from backup: ${backupName}`);
      return true;
    } catch (error) {
      console.error('Failed to restore configuration:', error);
      throw error;
    }
  }

  // List all backups
  listBackups() {
    const backupList = [];
    for (const [name, backup] of this.backups) {
      backupList.push({
        name,
        appId: backup.appId,
        timestamp: new Date(backup.timestamp).toISOString(),
        commandCount: backup.config.commands.length,
        promptCount: backup.config.prompts.length
      });
    }
    return backupList;
  }

  // Delete backup
  deleteBackup(backupName) {
    const deleted = this.backups.delete(backupName);
    if (deleted) {
      console.log(`Backup deleted: ${backupName}`);
    } else {
      console.log(`Backup not found: ${backupName}`);
    }
    return deleted;
  }
}

// Usage example
async function demonstrateBackupRestore() {
  try {
    const manager = new AdvancedConversationalComponentManager('YOUR_PARTNER_APP_TOKEN');
    const backupManager = new ConfigurationBackupManager(manager);

    // Backup current configuration
    await backupManager.backupConfiguration('YOUR_APP_ID', 'before-update');

    // Make changes
    await manager.setValidatedConfig('YOUR_APP_ID', ConfigurationTemplates.getEcommerceConfig());

    // List backups
    const backups = backupManager.listBackups();
    console.log('Available backups:', backups);

    // Restore if needed
    // await backupManager.restoreConfiguration('YOUR_APP_ID', 'before-update');

  } catch (error) {
    console.error('Backup/restore demonstration failed:', error);
  }
}
```

## Best Practices

1. **Validation**: Always validate configuration before setting
2. **Character Limits**: Respect character limits for commands and prompts
3. **No Emojis**: Avoid using emojis in commands or prompts
4. **Clear Commands**: Use clear, descriptive command names
5. **User-Friendly Prompts**: Create engaging and helpful ice breakers
6. **Backup**: Keep backups of working configurations
7. **Testing**: Test configurations with real users
8. **Gradual Updates**: Update configurations gradually to monitor impact
9. **Error Handling**: Implement robust error handling
10. **Documentation**: Document your configuration choices

## Important Notes

- **Welcome Messages**: Enable `enable_welcome_message` to receive `request_welcome` events
- **Command Limits**: Maximum 30 commands per configuration
- **Prompt Limits**: Maximum 4 ice breakers per configuration
- **Character Limits**: Commands (32 chars), descriptions (256 chars), prompts (80 chars)
- **No Emojis**: Emojis are not supported in any text fields
- **Case Sensitivity**: Command names are case-sensitive
- **Unique Commands**: Each command name should be unique
- **User Experience**: Design commands and prompts for optimal user experience
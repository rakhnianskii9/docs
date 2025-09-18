# OBO Embed API

The OBO (On Behalf Of) Embed API provides functionality to manage the transition from OBO (On Behalf Of) to Embed mode for WhatsApp Business Accounts (WABA). This API handles whitelisting WABA IDs and verifying ownership transitions.

## API Overview

| Feature | Description |
|---------|-------------|
| **Base URL** | `https://partner.gupshup.io/partner/app/{appId}/obotoembed` |
| **Authentication** | Bearer Token (Partner App Token) |
| **Rate Limit** | 10 requests per minute |
| **Content Type** | `application/json` |

## Authentication

All requests require a valid Partner App Token in the Authorization header:

```bash
Authorization: Bearer YOUR_PARTNER_APP_TOKEN
```

## Available Endpoints

### 1. Whitelist WABA ID

Whitelist a WhatsApp Business Account (WABA) ID for the embed signup process.

**Endpoint:** `POST /partner/app/{appId}/obotoembed/whitelist`

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `appId` | string | Yes | App ID for which whitelist is to be set |

#### Sample Request (cURL)

```bash
curl --location --request POST 'https://partner.gupshup.io/partner/app/YOUR_APP_ID/obotoembed/whitelist' \
--header 'Authorization: Bearer YOUR_PARTNER_APP_TOKEN'
```

#### Sample Response

```json
{
  "embedSignupUrl": "https://gs.tc.im/kZviNa8Zlx9",
  "id": "1016427996774921",
  "status": "success"
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `embedSignupUrl` | string | URL for the embed signup process |
| `id` | string | WABA ID that has been whitelisted |
| `status` | string | Response status (`success` or `error`) |

### 2. Verify and Attach Credit Line

Verify the ownership type and attach the credit line if the WABA has been migrated from OBO to Embed.

**Endpoint:** `GET /partner/app/{appId}/obotoembed/verify`

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `appId` | string | Yes | App ID for which verification is to be performed |

#### Sample Request (cURL)

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/YOUR_APP_ID/obotoembed/verify' \
--header 'Authorization: Bearer YOUR_PARTNER_APP_TOKEN'
```

#### Sample Response

```json
{
  "message": "Credit line added successfully for WABA id {waba_id}",
  "status": "success"
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `message` | string | Success message with WABA ID |
| `status` | string | Response status (`success` or `error`) |

## Status Codes

| Status | Response | Description |
|--------|----------|-------------|
| **Success** | | |
| 200 | `{"embedSignupUrl": "...", "id": "...", "status": "success"}` | WABA whitelisted successfully |
| 200 | `{"message": "Credit line added successfully...", "status": "success"}` | Credit line attached successfully |
| **Error** | | |
| 400 | `{"message": "Error while whitelisting WABA.", "status": "error"}` | Whitelisting failed |
| 400 | `{"message": "WABA is not migrated to embed yet...", "status": "error"}` | WABA not migrated to embed |
| 400 | `{"status": "error", "message": "Unable to add credit line for WABA id ..."}` | Credit line attachment failed |
| 429 | `{"status": "error", "message": "Too Many Requests"}` | Rate limit exceeded |
| 500 | `{"status": "error", "message": "Internal Server Error"}` | Server error |

## Node.js Examples

### Basic Usage

```javascript
const axios = require('axios');

class OBOEmbedManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseURL = 'https://partner.gupshup.io/partner/app';
  }

  // Whitelist WABA ID
  async whitelistWABA(appId) {
    try {
      const response = await axios.post(
        `${this.baseURL}/${appId}/obotoembed/whitelist`,
        {},
        {
          headers: {
            'Authorization': `Bearer ${this.partnerAppToken}`
          }
        }
      );

      return response.data;
    } catch (error) {
      console.error('Error whitelisting WABA:', error.response?.data || error.message);
      throw error;
    }
  }

  // Verify and attach credit line
  async verifyAndAttachCreditLine(appId) {
    try {
      const response = await axios.get(
        `${this.baseURL}/${appId}/obotoembed/verify`,
        {
          headers: {
            'Authorization': `Bearer ${this.partnerAppToken}`
          }
        }
      );

      return response.data;
    } catch (error) {
      console.error('Error verifying and attaching credit line:', error.response?.data || error.message);
      throw error;
    }
  }
}

// Usage example
const oboEmbedManager = new OBOEmbedManager('YOUR_PARTNER_APP_TOKEN');

// Whitelist WABA
async function whitelistWABA() {
  try {
    const result = await oboEmbedManager.whitelistWABA('YOUR_APP_ID');
    console.log('WABA whitelisted successfully:', result);
    console.log('Embed signup URL:', result.embedSignupUrl);
    console.log('WABA ID:', result.id);
    return result;
  } catch (error) {
    console.error('Failed to whitelist WABA:', error);
  }
}

// Verify and attach credit line
async function verifyAndAttachCreditLine() {
  try {
    const result = await oboEmbedManager.verifyAndAttachCreditLine('YOUR_APP_ID');
    console.log('Credit line attached successfully:', result);
    return result;
  } catch (error) {
    console.error('Failed to verify and attach credit line:', error);
  }
}
```

### Advanced Usage with Error Handling

```javascript
class AdvancedOBOEmbedManager extends OBOEmbedManager {
  // Enhanced whitelist with retry logic
  async whitelistWithRetry(appId, maxRetries = 3, retryDelay = 5000) {
    let lastError;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        console.log(`Attempting to whitelist WABA (attempt ${attempt}/${maxRetries})`);
        const result = await this.whitelistWABA(appId);
        console.log('WABA whitelisted successfully on attempt', attempt);
        return result;
      } catch (error) {
        lastError = error;
        console.error(`Attempt ${attempt} failed:`, error.response?.data || error.message);
        
        if (attempt < maxRetries) {
          console.log(`Retrying in ${retryDelay}ms...`);
          await new Promise(resolve => setTimeout(resolve, retryDelay));
        }
      }
    }
    
    throw new Error(`Failed to whitelist WABA after ${maxRetries} attempts. Last error: ${lastError.message}`);
  }

  // Enhanced verify with detailed error handling
  async verifyWithDetailedErrors(appId) {
    try {
      const result = await this.verifyAndAttachCreditLine(appId);
      return {
        success: true,
        data: result,
        message: 'Credit line attached successfully'
      };
    } catch (error) {
      const errorData = error.response?.data;
      
      if (errorData?.message?.includes('WABA is not migrated to embed yet')) {
        return {
          success: false,
          error: 'WABA_NOT_MIGRATED',
          message: 'WABA has not been migrated to embed yet. Please complete the embed signup process first.',
          data: errorData
        };
      }
      
      if (errorData?.message?.includes('Unable to add credit line')) {
        return {
          success: false,
          error: 'CREDIT_LINE_FAILED',
          message: 'Unable to add credit line for the WABA. Please try again later.',
          data: errorData
        };
      }
      
      if (error.response?.status === 429) {
        return {
          success: false,
          error: 'RATE_LIMIT_EXCEEDED',
          message: 'Rate limit exceeded. Please wait before trying again.',
          data: errorData
        };
      }
      
      return {
        success: false,
        error: 'UNKNOWN_ERROR',
        message: 'An unknown error occurred during verification.',
        data: errorData
      };
    }
  }

  // Complete migration flow
  async completeMigrationFlow(appId) {
    try {
      console.log('Starting OBO to Embed migration flow...');
      
      // Step 1: Whitelist WABA
      console.log('Step 1: Whitelisting WABA...');
      const whitelistResult = await this.whitelistWithRetry(appId);
      
      if (!whitelistResult.embedSignupUrl) {
        throw new Error('No embed signup URL received from whitelist operation');
      }
      
      console.log('WABA whitelisted successfully');
      console.log('Embed signup URL:', whitelistResult.embedSignupUrl);
      
      // Step 2: Wait for user to complete signup (manual step)
      console.log('Step 2: Please complete the embed signup process using the provided URL');
      console.log('Once completed, proceed with verification...');
      
      return {
        success: true,
        phase: 'WHITELIST_COMPLETE',
        embedSignupUrl: whitelistResult.embedSignupUrl,
        wabaId: whitelistResult.id,
        message: 'WABA whitelisted. Please complete embed signup before verification.'
      };
      
    } catch (error) {
      console.error('Migration flow failed:', error);
      return {
        success: false,
        phase: 'WHITELIST_FAILED',
        error: error.message
      };
    }
  }

  // Complete verification after signup
  async completeVerificationFlow(appId) {
    try {
      console.log('Step 3: Verifying and attaching credit line...');
      
      const verifyResult = await this.verifyWithDetailedErrors(appId);
      
      if (verifyResult.success) {
        console.log('Migration completed successfully!');
        return {
          success: true,
          phase: 'MIGRATION_COMPLETE',
          message: 'OBO to Embed migration completed successfully',
          data: verifyResult.data
        };
      } else {
        console.log('Verification failed:', verifyResult.message);
        return {
          success: false,
          phase: 'VERIFICATION_FAILED',
          error: verifyResult.error,
          message: verifyResult.message,
          data: verifyResult.data
        };
      }
      
    } catch (error) {
      console.error('Verification flow failed:', error);
      return {
        success: false,
        phase: 'VERIFICATION_ERROR',
        error: error.message
      };
    }
  }

  // Check migration status
  async checkMigrationStatus(appId) {
    try {
      const verifyResult = await this.verifyWithDetailedErrors(appId);
      
      if (verifyResult.success) {
        return {
          status: 'COMPLETED',
          message: 'Migration is complete and credit line is attached'
        };
      } else if (verifyResult.error === 'WABA_NOT_MIGRATED') {
        return {
          status: 'PENDING',
          message: 'WABA is whitelisted but embed signup is not complete'
        };
      } else {
        return {
          status: 'FAILED',
          message: verifyResult.message,
          error: verifyResult.error
        };
      }
    } catch (error) {
      return {
        status: 'UNKNOWN',
        message: 'Unable to determine migration status',
        error: error.message
      };
    }
  }
}

// Usage examples
const advancedManager = new AdvancedOBOEmbedManager('YOUR_PARTNER_APP_TOKEN');

async function demonstrateAdvancedFeatures() {
  try {
    // Complete migration flow
    const migrationResult = await advancedManager.completeMigrationFlow('YOUR_APP_ID');
    console.log('Migration result:', migrationResult);
    
    if (migrationResult.success) {
      console.log('Please complete the embed signup using this URL:', migrationResult.embedSignupUrl);
      
      // Simulate waiting for user to complete signup
      console.log('Waiting for user to complete signup...');
      
      // After user completes signup, verify
      const verificationResult = await advancedManager.completeVerificationFlow('YOUR_APP_ID');
      console.log('Verification result:', verificationResult);
    }
    
    // Check migration status
    const status = await advancedManager.checkMigrationStatus('YOUR_APP_ID');
    console.log('Migration status:', status);
    
  } catch (error) {
    console.error('Advanced features demonstration failed:', error);
  }
}
```

### Migration Workflow Manager

```javascript
class MigrationWorkflowManager {
  constructor(oboEmbedManager) {
    this.oboEmbedManager = oboEmbedManager;
    this.workflows = new Map();
  }

  // Start migration workflow
  async startMigrationWorkflow(appId, workflowId) {
    try {
      const workflow = {
        id: workflowId,
        appId: appId,
        status: 'STARTED',
        startedAt: Date.now(),
        steps: []
      };
      
      this.workflows.set(workflowId, workflow);
      
      // Step 1: Whitelist WABA
      const whitelistResult = await this.oboEmbedManager.whitelistWithRetry(appId);
      
      workflow.steps.push({
        step: 'WHITELIST',
        status: 'COMPLETED',
        result: whitelistResult,
        completedAt: Date.now()
      });
      
      workflow.status = 'AWAITING_SIGNUP';
      workflow.embedSignupUrl = whitelistResult.embedSignupUrl;
      workflow.wabaId = whitelistResult.id;
      
      console.log(`Migration workflow ${workflowId} started. Awaiting user signup.`);
      return workflow;
      
    } catch (error) {
      const workflow = this.workflows.get(workflowId);
      if (workflow) {
        workflow.status = 'FAILED';
        workflow.error = error.message;
        workflow.steps.push({
          step: 'WHITELIST',
          status: 'FAILED',
          error: error.message,
          completedAt: Date.now()
        });
      }
      throw error;
    }
  }

  // Complete migration workflow
  async completeMigrationWorkflow(workflowId) {
    try {
      const workflow = this.workflows.get(workflowId);
      if (!workflow) {
        throw new Error(`Workflow ${workflowId} not found`);
      }
      
      if (workflow.status !== 'AWAITING_SIGNUP') {
        throw new Error(`Workflow ${workflowId} is not ready for completion. Current status: ${workflow.status}`);
      }
      
      // Step 2: Verify and attach credit line
      const verifyResult = await this.oboEmbedManager.verifyWithDetailedErrors(workflow.appId);
      
      if (verifyResult.success) {
        workflow.steps.push({
          step: 'VERIFY',
          status: 'COMPLETED',
          result: verifyResult.data,
          completedAt: Date.now()
        });
        
        workflow.status = 'COMPLETED';
        workflow.completedAt = Date.now();
        
        console.log(`Migration workflow ${workflowId} completed successfully.`);
        return workflow;
      } else {
        workflow.steps.push({
          step: 'VERIFY',
          status: 'FAILED',
          error: verifyResult.error,
          message: verifyResult.message,
          completedAt: Date.now()
        });
        
        workflow.status = 'VERIFICATION_FAILED';
        workflow.error = verifyResult.message;
        
        throw new Error(`Verification failed: ${verifyResult.message}`);
      }
      
    } catch (error) {
      const workflow = this.workflows.get(workflowId);
      if (workflow) {
        workflow.status = 'FAILED';
        workflow.error = error.message;
      }
      throw error;
    }
  }

  // Get workflow status
  getWorkflowStatus(workflowId) {
    return this.workflows.get(workflowId);
  }

  // List all workflows
  listWorkflows() {
    return Array.from(this.workflows.values());
  }

  // Cancel workflow
  cancelWorkflow(workflowId) {
    const workflow = this.workflows.get(workflowId);
    if (workflow) {
      workflow.status = 'CANCELLED';
      workflow.cancelledAt = Date.now();
      return true;
    }
    return false;
  }

  // Cleanup completed workflows
  cleanupCompletedWorkflows(maxAge = 24 * 60 * 60 * 1000) { // 24 hours
    const now = Date.now();
    const completed = [];
    
    for (const [workflowId, workflow] of this.workflows) {
      if (workflow.status === 'COMPLETED' && (now - workflow.completedAt) > maxAge) {
        this.workflows.delete(workflowId);
        completed.push(workflowId);
      }
    }
    
    return completed;
  }
}

// Usage example
async function demonstrateWorkflowManager() {
  try {
    const oboManager = new AdvancedOBOEmbedManager('YOUR_PARTNER_APP_TOKEN');
    const workflowManager = new MigrationWorkflowManager(oboManager);
    
    // Start migration workflow
    const workflowId = `migration-${Date.now()}`;
    const workflow = await workflowManager.startMigrationWorkflow('YOUR_APP_ID', workflowId);
    
    console.log('Migration workflow started:', workflow);
    console.log('Please complete signup at:', workflow.embedSignupUrl);
    
    // Simulate waiting for user to complete signup
    console.log('Waiting for user to complete signup...');
    
    // Check workflow status
    const status = workflowManager.getWorkflowStatus(workflowId);
    console.log('Workflow status:', status);
    
    // Complete workflow (after user completes signup)
    // const completedWorkflow = await workflowManager.completeMigrationWorkflow(workflowId);
    // console.log('Workflow completed:', completedWorkflow);
    
  } catch (error) {
    console.error('Workflow demonstration failed:', error);
  }
}
```

### Monitoring and Logging

```javascript
class OBOEmbedMonitor {
  constructor(oboEmbedManager) {
    this.oboEmbedManager = oboEmbedManager;
    this.logs = [];
    this.metrics = {
      whitelistAttempts: 0,
      whitelistSuccesses: 0,
      whitelistFailures: 0,
      verificationAttempts: 0,
      verificationSuccesses: 0,
      verificationFailures: 0
    };
  }

  // Log event
  log(level, message, data = {}) {
    const logEntry = {
      timestamp: Date.now(),
      level,
      message,
      data
    };
    
    this.logs.push(logEntry);
    console.log(`[${level}] ${message}`, data);
    
    // Keep only last 1000 logs
    if (this.logs.length > 1000) {
      this.logs = this.logs.slice(-1000);
    }
  }

  // Monitor whitelist operation
  async monitorWhitelist(appId) {
    this.metrics.whitelistAttempts++;
    this.log('INFO', 'Starting WABA whitelist operation', { appId });
    
    try {
      const result = await this.oboEmbedManager.whitelistWABA(appId);
      this.metrics.whitelistSuccesses++;
      this.log('SUCCESS', 'WABA whitelist completed successfully', { appId, result });
      return result;
    } catch (error) {
      this.metrics.whitelistFailures++;
      this.log('ERROR', 'WABA whitelist failed', { appId, error: error.message });
      throw error;
    }
  }

  // Monitor verification operation
  async monitorVerification(appId) {
    this.metrics.verificationAttempts++;
    this.log('INFO', 'Starting verification and credit line attachment', { appId });
    
    try {
      const result = await this.oboEmbedManager.verifyAndAttachCreditLine(appId);
      this.metrics.verificationSuccesses++;
      this.log('SUCCESS', 'Verification and credit line attachment completed', { appId, result });
      return result;
    } catch (error) {
      this.metrics.verificationFailures++;
      this.log('ERROR', 'Verification failed', { appId, error: error.message });
      throw error;
    }
  }

  // Get metrics
  getMetrics() {
    return {
      ...this.metrics,
      whitelistSuccessRate: this.metrics.whitelistAttempts > 0 
        ? (this.metrics.whitelistSuccesses / this.metrics.whitelistAttempts * 100).toFixed(2) + '%'
        : 'N/A',
      verificationSuccessRate: this.metrics.verificationAttempts > 0
        ? (this.metrics.verificationSuccesses / this.metrics.verificationAttempts * 100).toFixed(2) + '%'
        : 'N/A'
    };
  }

  // Get logs
  getLogs(level = null, limit = 100) {
    let filteredLogs = this.logs;
    
    if (level) {
      filteredLogs = this.logs.filter(log => log.level === level);
    }
    
    return filteredLogs.slice(-limit);
  }

  // Clear logs
  clearLogs() {
    this.logs = [];
    this.log('INFO', 'Logs cleared');
  }

  // Reset metrics
  resetMetrics() {
    this.metrics = {
      whitelistAttempts: 0,
      whitelistSuccesses: 0,
      whitelistFailures: 0,
      verificationAttempts: 0,
      verificationSuccesses: 0,
      verificationFailures: 0
    };
    this.log('INFO', 'Metrics reset');
  }
}

// Usage example
async function demonstrateMonitoring() {
  try {
    const oboManager = new AdvancedOBOEmbedManager('YOUR_PARTNER_APP_TOKEN');
    const monitor = new OBOEmbedMonitor(oboManager);
    
    // Monitor whitelist operation
    await monitor.monitorWhitelist('YOUR_APP_ID');
    
    // Monitor verification operation
    await monitor.monitorVerification('YOUR_APP_ID');
    
    // Get metrics
    const metrics = monitor.getMetrics();
    console.log('Metrics:', metrics);
    
    // Get logs
    const logs = monitor.getLogs('ERROR', 10);
    console.log('Error logs:', logs);
    
  } catch (error) {
    console.error('Monitoring demonstration failed:', error);
  }
}
```

## Best Practices

1. **Rate Limiting**: Respect the 10 requests per minute limit
2. **Error Handling**: Implement comprehensive error handling for different scenarios
3. **Retry Logic**: Use retry mechanisms for transient failures
4. **Workflow Management**: Track migration workflows systematically
5. **User Communication**: Clearly communicate the embed signup process to users
6. **Status Monitoring**: Regular monitoring of migration status
7. **Logging**: Maintain detailed logs for troubleshooting
8. **Validation**: Validate app IDs and tokens before making requests
9. **Cleanup**: Clean up completed workflows to manage memory
10. **Documentation**: Document the migration process for users

## Migration Process

1. **Whitelist WABA**: Call the whitelist endpoint to get the embed signup URL
2. **User Signup**: User completes the embed signup process using the provided URL
3. **Verification**: Call the verify endpoint to attach the credit line
4. **Completion**: Migration from OBO to Embed is complete

## Important Notes

- **Sequential Process**: The whitelist step must be completed before verification
- **User Action Required**: Users must complete the embed signup process manually
- **WABA Migration**: The WABA must be successfully migrated to embed before verification
- **Credit Line**: The credit line is automatically attached upon successful verification
- **Rate Limits**: Both endpoints are subject to rate limiting (10 requests per minute)
- **Error Handling**: Different error types require different handling strategies
- **Monitoring**: Monitor the migration process for successful completion
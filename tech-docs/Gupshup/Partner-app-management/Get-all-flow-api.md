# Gupshup Get All Flow API

## Overview
The Get All Flow API allows you to retrieve a list of all flows associated with a specific app ID in Gupshup Partner Portal. This API provides comprehensive information about each flow including its status, categories, and validation errors.

## Base URL
```
https://partner.gupshup.io/partner/app/{appId}/flows
```

## Authentication
- **Method**: Bearer Token
- **Header**: `Authorization: {PARTNER_APP_TOKEN}`

## Endpoint

### Get All Flows
**GET** `/partner/app/{appId}/flows`

Retrieves a list of all flows associated with the specified app ID.

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | String | Yes | Access Token for the application |
| appId | String | Yes | App ID to fetch the flows for |

#### Sample cURL Request
```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/flows' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/json'
```

**Real Example:**
```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/YOUR_APP_ID/flows' \
--header 'Authorization: YOUR_PARTNER_APP_TOKEN' \
--header 'Content-Type: application/json'
```

#### Sample Response
```json
[
    {
        "categories": [
            "APPOINTMENT_BOOKING"
        ],
        "id": "460785803057688",
        "name": "Message templates_delivery_failed_form_1_APPOINTMENT_BOOKING_93375",
        "status": "PUBLISHED",
        "validation_errors": []
    },
    {
        "categories": [
            "SIGN_UP"
        ],
        "id": "881255207342075",
        "name": "Message templates_temp_signup_form_MARKETING_e7589",
        "status": "PUBLISHED",
        "validation_errors": []
    },
    {
        "categories": [
            "SIGN_UP"
        ],
        "id": "356169917364619",
        "name": "flow_sign_up",
        "status": "PUBLISHED",
        "validation_errors": []
    }
]
```

#### Status Codes

| Status | Response | Description |
|--------|----------|-------------|
| 200 | Array of flow objects | Successfully retrieved all flows |
| 401 | `{ "status": "error", "message": "Unauthorised access to the resource. Please review request parameters and headers and retry" }` | For invalid app Id or partner app token |

## Node.js Examples

### Basic Get All Flows
```javascript
const https = require('https');

async function getAllFlows(appId, partnerToken) {
    const options = {
        hostname: 'partner.gupshup.io',
        port: 443,
        path: `/partner/app/${appId}/flows`,
        method: 'GET',
        headers: {
            'Authorization': partnerToken,
            'Content-Type': 'application/json'
        }
    };

    return new Promise((resolve, reject) => {
        const req = https.request(options, (res) => {
            let responseData = '';
            
            res.on('data', (chunk) => {
                responseData += chunk;
            });
            
            res.on('end', () => {
                try {
                    const parsedData = JSON.parse(responseData);
                    resolve(parsedData);
                } catch (error) {
                    reject(new Error('Invalid JSON response'));
                }
            });
        });

        req.on('error', (error) => {
            reject(error);
        });

        req.end();
    });
}

// Usage
getAllFlows('your-app-id', 'your-partner-token')
    .then(flows => {
        console.log('All flows:', flows);
        console.log(`Total flows: ${flows.length}`);
    })
    .catch(error => {
        console.error('Error fetching flows:', error);
    });
```

### Advanced Flow Manager with Fetch
```javascript
const fetch = require('node-fetch');

class FlowManager {
    constructor(baseUrl, partnerToken) {
        this.baseUrl = baseUrl || 'https://partner.gupshup.io';
        this.partnerToken = partnerToken;
    }

    async getAllFlows(appId) {
        const url = `${this.baseUrl}/partner/app/${appId}/flows`;
        
        try {
            const response = await fetch(url, {
                method: 'GET',
                headers: {
                    'Authorization': this.partnerToken,
                    'Content-Type': 'application/json'
                }
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(`HTTP error! status: ${response.status}, message: ${errorData.message}`);
            }

            return await response.json();
        } catch (error) {
            throw new Error(`Failed to fetch flows: ${error.message}`);
        }
    }

    async getFlowsByStatus(appId, status) {
        const allFlows = await this.getAllFlows(appId);
        return allFlows.filter(flow => flow.status === status);
    }

    async getFlowsByCategory(appId, category) {
        const allFlows = await this.getAllFlows(appId);
        return allFlows.filter(flow => flow.categories.includes(category));
    }

    async getFlowsWithValidationErrors(appId) {
        const allFlows = await this.getAllFlows(appId);
        return allFlows.filter(flow => flow.validation_errors && flow.validation_errors.length > 0);
    }

    async getFlowById(appId, flowId) {
        const allFlows = await this.getAllFlows(appId);
        return allFlows.find(flow => flow.id === flowId);
    }

    async getFlowsByName(appId, namePattern) {
        const allFlows = await this.getAllFlows(appId);
        const regex = new RegExp(namePattern, 'i');
        return allFlows.filter(flow => regex.test(flow.name));
    }

    async getFlowStatistics(appId) {
        const allFlows = await this.getAllFlows(appId);
        
        const stats = {
            total: allFlows.length,
            byStatus: {},
            byCategory: {},
            withValidationErrors: 0
        };

        allFlows.forEach(flow => {
            // Count by status
            stats.byStatus[flow.status] = (stats.byStatus[flow.status] || 0) + 1;
            
            // Count by category
            flow.categories.forEach(category => {
                stats.byCategory[category] = (stats.byCategory[category] || 0) + 1;
            });
            
            // Count validation errors
            if (flow.validation_errors && flow.validation_errors.length > 0) {
                stats.withValidationErrors++;
            }
        });

        return stats;
    }

    async searchFlows(appId, searchCriteria) {
        const allFlows = await this.getAllFlows(appId);
        
        return allFlows.filter(flow => {
            let matches = true;
            
            if (searchCriteria.status) {
                matches = matches && flow.status === searchCriteria.status;
            }
            
            if (searchCriteria.category) {
                matches = matches && flow.categories.includes(searchCriteria.category);
            }
            
            if (searchCriteria.name) {
                const regex = new RegExp(searchCriteria.name, 'i');
                matches = matches && regex.test(flow.name);
            }
            
            if (searchCriteria.hasValidationErrors !== undefined) {
                const hasErrors = flow.validation_errors && flow.validation_errors.length > 0;
                matches = matches && (hasErrors === searchCriteria.hasValidationErrors);
            }
            
            return matches;
        });
    }
}

// Usage
const flowManager = new FlowManager('https://partner.gupshup.io', 'your-partner-token');

// Get all flows
flowManager.getAllFlows('your-app-id')
    .then(flows => {
        console.log('All flows:', flows);
    })
    .catch(error => {
        console.error('Error:', error.message);
    });

// Get published flows only
flowManager.getFlowsByStatus('your-app-id', 'PUBLISHED')
    .then(publishedFlows => {
        console.log('Published flows:', publishedFlows);
    })
    .catch(error => {
        console.error('Error:', error.message);
    });

// Get flows by category
flowManager.getFlowsByCategory('your-app-id', 'SIGN_UP')
    .then(signupFlows => {
        console.log('Sign-up flows:', signupFlows);
    })
    .catch(error => {
        console.error('Error:', error.message);
    });

// Get flow statistics
flowManager.getFlowStatistics('your-app-id')
    .then(stats => {
        console.log('Flow statistics:', stats);
    })
    .catch(error => {
        console.error('Error:', error.message);
    });
```

### Comprehensive Flow Analytics Manager
```javascript
const axios = require('axios');

class FlowAnalyticsManager {
    constructor(baseUrl, partnerToken) {
        this.baseUrl = baseUrl || 'https://partner.gupshup.io';
        this.partnerToken = partnerToken;
        this.cache = new Map();
        this.cacheTimeout = 300000; // 5 minutes
    }

    async getAllFlowsWithCache(appId, forceRefresh = false) {
        const cacheKey = `flows_${appId}`;
        const cachedData = this.cache.get(cacheKey);
        
        if (!forceRefresh && cachedData && Date.now() - cachedData.timestamp < this.cacheTimeout) {
            return cachedData.data;
        }

        try {
            const response = await axios.get(`${this.baseUrl}/partner/app/${appId}/flows`, {
                headers: {
                    'Authorization': this.partnerToken,
                    'Content-Type': 'application/json'
                }
            });

            const flows = response.data;
            this.cache.set(cacheKey, {
                data: flows,
                timestamp: Date.now()
            });

            return flows;
        } catch (error) {
            throw new Error(`Failed to fetch flows: ${error.message}`);
        }
    }

    async getFlowAnalytics(appId) {
        const flows = await this.getAllFlowsWithCache(appId);
        
        const analytics = {
            overview: {
                totalFlows: flows.length,
                lastUpdated: new Date().toISOString()
            },
            statusDistribution: this.calculateStatusDistribution(flows),
            categoryDistribution: this.calculateCategoryDistribution(flows),
            validationAnalysis: this.analyzeValidationErrors(flows),
            flowHealth: this.calculateFlowHealth(flows),
            recommendations: this.generateRecommendations(flows)
        };

        return analytics;
    }

    calculateStatusDistribution(flows) {
        const distribution = {};
        
        flows.forEach(flow => {
            distribution[flow.status] = (distribution[flow.status] || 0) + 1;
        });

        const total = flows.length;
        const percentages = {};
        
        Object.keys(distribution).forEach(status => {
            percentages[status] = {
                count: distribution[status],
                percentage: ((distribution[status] / total) * 100).toFixed(2) + '%'
            };
        });

        return percentages;
    }

    calculateCategoryDistribution(flows) {
        const distribution = {};
        
        flows.forEach(flow => {
            flow.categories.forEach(category => {
                distribution[category] = (distribution[category] || 0) + 1;
            });
        });

        const sortedCategories = Object.entries(distribution)
            .sort(([,a], [,b]) => b - a)
            .reduce((acc, [category, count]) => {
                acc[category] = count;
                return acc;
            }, {});

        return sortedCategories;
    }

    analyzeValidationErrors(flows) {
        const flowsWithErrors = flows.filter(flow => 
            flow.validation_errors && flow.validation_errors.length > 0
        );

        const errorTypes = {};
        flowsWithErrors.forEach(flow => {
            flow.validation_errors.forEach(error => {
                const errorType = error.type || 'UNKNOWN';
                errorTypes[errorType] = (errorTypes[errorType] || 0) + 1;
            });
        });

        return {
            totalFlowsWithErrors: flowsWithErrors.length,
            errorRate: ((flowsWithErrors.length / flows.length) * 100).toFixed(2) + '%',
            errorTypes,
            flowsWithErrors: flowsWithErrors.map(flow => ({
                id: flow.id,
                name: flow.name,
                errorCount: flow.validation_errors.length
            }))
        };
    }

    calculateFlowHealth(flows) {
        const healthyFlows = flows.filter(flow => 
            flow.status === 'PUBLISHED' && 
            (!flow.validation_errors || flow.validation_errors.length === 0)
        );

        const draftFlows = flows.filter(flow => flow.status === 'DRAFT');
        const deprecatedFlows = flows.filter(flow => flow.status === 'DEPRECATED');

        return {
            healthy: {
                count: healthyFlows.length,
                percentage: ((healthyFlows.length / flows.length) * 100).toFixed(2) + '%'
            },
            draft: {
                count: draftFlows.length,
                percentage: ((draftFlows.length / flows.length) * 100).toFixed(2) + '%'
            },
            deprecated: {
                count: deprecatedFlows.length,
                percentage: ((deprecatedFlows.length / flows.length) * 100).toFixed(2) + '%'
            },
            overallHealthScore: this.calculateHealthScore(flows)
        };
    }

    calculateHealthScore(flows) {
        if (flows.length === 0) return 0;
        
        let score = 0;
        
        flows.forEach(flow => {
            if (flow.status === 'PUBLISHED') {
                score += 10;
            } else if (flow.status === 'DRAFT') {
                score += 5;
            } else if (flow.status === 'DEPRECATED') {
                score += 1;
            }
            
            if (!flow.validation_errors || flow.validation_errors.length === 0) {
                score += 5;
            }
        });
        
        const maxScore = flows.length * 15;
        return Math.round((score / maxScore) * 100);
    }

    generateRecommendations(flows) {
        const recommendations = [];
        
        const draftFlows = flows.filter(flow => flow.status === 'DRAFT');
        const flowsWithErrors = flows.filter(flow => 
            flow.validation_errors && flow.validation_errors.length > 0
        );
        const deprecatedFlows = flows.filter(flow => flow.status === 'DEPRECATED');

        if (draftFlows.length > flows.length * 0.3) {
            recommendations.push({
                type: 'DRAFT_FLOWS',
                priority: 'HIGH',
                message: `You have ${draftFlows.length} draft flows. Consider publishing or cleaning up unused drafts.`
            });
        }

        if (flowsWithErrors.length > 0) {
            recommendations.push({
                type: 'VALIDATION_ERRORS',
                priority: 'CRITICAL',
                message: `${flowsWithErrors.length} flows have validation errors that need attention.`
            });
        }

        if (deprecatedFlows.length > flows.length * 0.2) {
            recommendations.push({
                type: 'DEPRECATED_FLOWS',
                priority: 'MEDIUM',
                message: `Consider removing ${deprecatedFlows.length} deprecated flows to clean up your flow library.`
            });
        }

        if (flows.length === 0) {
            recommendations.push({
                type: 'NO_FLOWS',
                priority: 'LOW',
                message: 'No flows found. Consider creating your first flow to get started.'
            });
        }

        return recommendations;
    }

    async exportFlowData(appId, format = 'json') {
        const flows = await this.getAllFlowsWithCache(appId);
        const analytics = await this.getFlowAnalytics(appId);
        
        const exportData = {
            exportTimestamp: new Date().toISOString(),
            appId,
            flows,
            analytics,
            summary: {
                totalFlows: flows.length,
                publishedFlows: flows.filter(f => f.status === 'PUBLISHED').length,
                draftFlows: flows.filter(f => f.status === 'DRAFT').length,
                deprecatedFlows: flows.filter(f => f.status === 'DEPRECATED').length
            }
        };

        if (format === 'csv') {
            return this.convertToCSV(flows);
        }

        return exportData;
    }

    convertToCSV(flows) {
        const headers = ['ID', 'Name', 'Status', 'Categories', 'Validation Errors'];
        const csvRows = [headers.join(',')];
        
        flows.forEach(flow => {
            const row = [
                flow.id,
                `"${flow.name.replace(/"/g, '""')}"`,
                flow.status,
                `"${flow.categories.join('; ')}"`,
                flow.validation_errors ? flow.validation_errors.length : 0
            ];
            csvRows.push(row.join(','));
        });

        return csvRows.join('\n');
    }

    async generateFlowReport(appId, includeDetails = false) {
        const flows = await this.getAllFlowsWithCache(appId);
        const analytics = await this.getFlowAnalytics(appId);
        
        const report = {
            reportGenerated: new Date().toISOString(),
            appId,
            summary: analytics.overview,
            health: analytics.flowHealth,
            distribution: {
                status: analytics.statusDistribution,
                categories: analytics.categoryDistribution
            },
            issues: {
                validationErrors: analytics.validationAnalysis,
                recommendations: analytics.recommendations
            }
        };

        if (includeDetails) {
            report.flows = flows;
        }

        return report;
    }

    clearCache() {
        this.cache.clear();
    }
}

// Usage
const analyticsManager = new FlowAnalyticsManager('https://partner.gupshup.io', 'your-partner-token');

// Get comprehensive analytics
analyticsManager.getFlowAnalytics('your-app-id')
    .then(analytics => {
        console.log('Flow Analytics:', JSON.stringify(analytics, null, 2));
    })
    .catch(error => {
        console.error('Error:', error.message);
    });

// Generate report
analyticsManager.generateFlowReport('your-app-id', true)
    .then(report => {
        console.log('Flow Report:', JSON.stringify(report, null, 2));
    })
    .catch(error => {
        console.error('Error:', error.message);
    });

// Export data
analyticsManager.exportFlowData('your-app-id', 'json')
    .then(data => {
        console.log('Export Data:', data);
    })
    .catch(error => {
        console.error('Error:', error.message);
    });
```

### Flow Monitoring and Alerting
```javascript
class FlowMonitoringService {
    constructor(flowAnalyticsManager) {
        this.analyticsManager = flowAnalyticsManager;
        this.alertThresholds = {
            errorRate: 10, // percentage
            draftRatio: 30, // percentage
            deprecatedRatio: 20 // percentage
        };
        this.alerts = [];
    }

    async monitorFlows(appId) {
        try {
            const analytics = await this.analyticsManager.getFlowAnalytics(appId);
            const alerts = this.checkAlerts(analytics);
            
            if (alerts.length > 0) {
                this.alerts.push(...alerts);
                await this.sendAlerts(alerts);
            }

            return {
                monitoringTimestamp: new Date().toISOString(),
                appId,
                analytics,
                alerts,
                status: alerts.length > 0 ? 'ALERTS_DETECTED' : 'HEALTHY'
            };
        } catch (error) {
            const errorAlert = {
                type: 'MONITORING_ERROR',
                severity: 'CRITICAL',
                message: `Failed to monitor flows: ${error.message}`,
                timestamp: new Date().toISOString()
            };
            
            this.alerts.push(errorAlert);
            await this.sendAlerts([errorAlert]);
            
            throw error;
        }
    }

    checkAlerts(analytics) {
        const alerts = [];

        // Check error rate
        const errorRate = parseFloat(analytics.validationAnalysis.errorRate);
        if (errorRate > this.alertThresholds.errorRate) {
            alerts.push({
                type: 'HIGH_ERROR_RATE',
                severity: 'HIGH',
                message: `Error rate ${errorRate}% exceeds threshold ${this.alertThresholds.errorRate}%`,
                timestamp: new Date().toISOString(),
                data: analytics.validationAnalysis
            });
        }

        // Check draft ratio
        const draftRatio = parseFloat(analytics.flowHealth.draft.percentage);
        if (draftRatio > this.alertThresholds.draftRatio) {
            alerts.push({
                type: 'HIGH_DRAFT_RATIO',
                severity: 'MEDIUM',
                message: `Draft flows ${draftRatio}% exceeds threshold ${this.alertThresholds.draftRatio}%`,
                timestamp: new Date().toISOString(),
                data: analytics.flowHealth.draft
            });
        }

        // Check deprecated ratio
        const deprecatedRatio = parseFloat(analytics.flowHealth.deprecated.percentage);
        if (deprecatedRatio > this.alertThresholds.deprecatedRatio) {
            alerts.push({
                type: 'HIGH_DEPRECATED_RATIO',
                severity: 'LOW',
                message: `Deprecated flows ${deprecatedRatio}% exceeds threshold ${this.alertThresholds.deprecatedRatio}%`,
                timestamp: new Date().toISOString(),
                data: analytics.flowHealth.deprecated
            });
        }

        // Check overall health score
        if (analytics.flowHealth.overallHealthScore < 70) {
            alerts.push({
                type: 'LOW_HEALTH_SCORE',
                severity: 'MEDIUM',
                message: `Overall health score ${analytics.flowHealth.overallHealthScore}% is below recommended threshold`,
                timestamp: new Date().toISOString(),
                data: analytics.flowHealth
            });
        }

        return alerts;
    }

    async sendAlerts(alerts) {
        // In production, this would send to monitoring service, email, Slack, etc.
        alerts.forEach(alert => {
            console.log(`🚨 ALERT [${alert.severity}]: ${alert.message}`);
        });
        
        // Example: Send to webhook
        // await this.sendToWebhook(alerts);
    }

    async sendToWebhook(alerts) {
        // Example webhook implementation
        const webhookUrl = process.env.WEBHOOK_URL;
        if (!webhookUrl) return;

        try {
            await axios.post(webhookUrl, {
                alerts,
                timestamp: new Date().toISOString()
            });
        } catch (error) {
            console.error('Failed to send webhook:', error.message);
        }
    }

    getAlertHistory() {
        return this.alerts;
    }

    clearAlerts() {
        this.alerts = [];
    }

    setAlertThresholds(thresholds) {
        this.alertThresholds = { ...this.alertThresholds, ...thresholds };
    }
}

// Usage
const analyticsManager = new FlowAnalyticsManager('https://partner.gupshup.io', 'your-partner-token');
const monitoringService = new FlowMonitoringService(analyticsManager);

// Monitor flows
monitoringService.monitorFlows('your-app-id')
    .then(result => {
        console.log('Monitoring Result:', result);
    })
    .catch(error => {
        console.error('Monitoring Error:', error);
    });

// Set custom thresholds
monitoringService.setAlertThresholds({
    errorRate: 15,
    draftRatio: 25,
    deprecatedRatio: 15
});
```

### Batch Operations Manager
```javascript
class BatchFlowOperationsManager {
    constructor(flowManager) {
        this.flowManager = flowManager;
        this.batchSize = 10;
        this.delay = 1000; // 1 second between batches
    }

    async processMultipleApps(appIds, operation) {
        const results = [];
        
        for (const appId of appIds) {
            try {
                const result = await this.processApp(appId, operation);
                results.push({
                    appId,
                    success: true,
                    data: result
                });
            } catch (error) {
                results.push({
                    appId,
                    success: false,
                    error: error.message
                });
            }
            
            // Add delay between app processing
            await this.sleep(this.delay);
        }

        return results;
    }

    async processApp(appId, operation) {
        switch (operation) {
            case 'getAllFlows':
                return await this.flowManager.getAllFlows(appId);
            case 'getAnalytics':
                return await this.flowManager.getFlowAnalytics(appId);
            case 'getStatistics':
                return await this.flowManager.getFlowStatistics(appId);
            default:
                throw new Error(`Unknown operation: ${operation}`);
        }
    }

    async compareApps(appIds) {
        const results = await this.processMultipleApps(appIds, 'getStatistics');
        const comparison = {
            apps: {},
            summary: {
                totalApps: appIds.length,
                successfulApps: results.filter(r => r.success).length,
                failedApps: results.filter(r => !r.success).length
            }
        };

        results.forEach(result => {
            if (result.success) {
                comparison.apps[result.appId] = result.data;
            } else {
                comparison.apps[result.appId] = { error: result.error };
            }
        });

        return comparison;
    }

    async findFlowsAcrossApps(appIds, searchCriteria) {
        const results = [];
        
        for (const appId of appIds) {
            try {
                const flows = await this.flowManager.searchFlows(appId, searchCriteria);
                if (flows.length > 0) {
                    results.push({
                        appId,
                        flows,
                        count: flows.length
                    });
                }
            } catch (error) {
                console.error(`Error searching flows for app ${appId}:`, error.message);
            }
        }

        return results;
    }

    async generateCrossAppReport(appIds) {
        const results = await this.processMultipleApps(appIds, 'getAnalytics');
        
        const crossAppReport = {
            reportGenerated: new Date().toISOString(),
            totalApps: appIds.length,
            processedApps: results.filter(r => r.success).length,
            failedApps: results.filter(r => !r.success).length,
            aggregatedStats: {
                totalFlows: 0,
                totalPublished: 0,
                totalDraft: 0,
                totalDeprecated: 0,
                totalWithErrors: 0
            },
            appBreakdown: {}
        };

        results.forEach(result => {
            if (result.success) {
                const analytics = result.data;
                crossAppReport.aggregatedStats.totalFlows += analytics.overview.totalFlows;
                
                // Add to breakdown
                crossAppReport.appBreakdown[result.appId] = {
                    totalFlows: analytics.overview.totalFlows,
                    healthScore: analytics.flowHealth.overallHealthScore,
                    errorRate: analytics.validationAnalysis.errorRate,
                    status: analytics.flowHealth
                };
            } else {
                crossAppReport.appBreakdown[result.appId] = {
                    error: result.error
                };
            }
        });

        return crossAppReport;
    }

    sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// Usage
const flowManager = new FlowManager('https://partner.gupshup.io', 'your-partner-token');
const batchManager = new BatchFlowOperationsManager(flowManager);

// Process multiple apps
const appIds = ['app1', 'app2', 'app3'];
batchManager.processMultipleApps(appIds, 'getAllFlows')
    .then(results => {
        console.log('Batch Results:', results);
    })
    .catch(error => {
        console.error('Batch Error:', error);
    });

// Generate cross-app report
batchManager.generateCrossAppReport(appIds)
    .then(report => {
        console.log('Cross-App Report:', JSON.stringify(report, null, 2));
    })
    .catch(error => {
        console.error('Report Error:', error);
    });
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| categories | Array | List of flow categories |
| id | String | Unique flow identifier |
| name | String | Flow name |
| status | String | Flow status (PUBLISHED, DRAFT, DEPRECATED) |
| validation_errors | Array | List of validation errors (if any) |

## Flow Status Types

- **PUBLISHED**: Flow is live and active
- **DRAFT**: Flow is in development/editing state
- **DEPRECATED**: Flow is deprecated and no longer active

## Common Flow Categories

- `APPOINTMENT_BOOKING` - Appointment scheduling flows
- `SIGN_UP` - User registration flows
- `MARKETING` - Marketing campaign flows
- `SUPPORT` - Customer support flows
- `FAQ` - Frequently asked questions flows
- `ORDERS` - Order management flows
- `PAYMENTS` - Payment processing flows
- `NOTIFICATIONS` - Notification flows

## Error Handling

```javascript
function handleFlowFetchError(error) {
    if (error.response) {
        const { status, data } = error.response;
        
        switch (status) {
            case 401:
                console.error('Unauthorized: Invalid partner token or app ID');
                return 'UNAUTHORIZED';
            case 403:
                console.error('Forbidden: Insufficient permissions');
                return 'FORBIDDEN';
            case 404:
                console.error('Not Found: App not found');
                return 'NOT_FOUND';
            case 500:
                console.error('Server Error: Try again later');
                return 'SERVER_ERROR';
            default:
                console.error('Unknown error:', data);
                return 'UNKNOWN_ERROR';
        }
    } else if (error.request) {
        console.error('Network error: No response received');
        return 'NETWORK_ERROR';
    } else {
        console.error('Error:', error.message);
        return 'UNKNOWN_ERROR';
    }
}
```

## Best Practices

1. **Caching**: Implement caching for frequently accessed flow data
2. **Pagination**: Handle large numbers of flows efficiently
3. **Error Handling**: Implement comprehensive error handling
4. **Rate Limiting**: Respect API rate limits when making multiple requests
5. **Filtering**: Use client-side filtering for better performance
6. **Monitoring**: Monitor flow health and status regularly
7. **Analytics**: Track flow performance and usage patterns
8. **Batch Operations**: Process multiple apps efficiently
9. **Data Export**: Provide data export capabilities for analysis
10. **Alerting**: Set up alerts for flow health issues

## Common Use Cases

### 1. Flow Inventory Management
```javascript
const flows = await flowManager.getAllFlows(appId);
const inventory = {
    total: flows.length,
    published: flows.filter(f => f.status === 'PUBLISHED').length,
    draft: flows.filter(f => f.status === 'DRAFT').length
};
```

### 2. Health Monitoring
```javascript
const healthyFlows = await flowManager.searchFlows(appId, {
    status: 'PUBLISHED',
    hasValidationErrors: false
});
```

### 3. Category Analysis
```javascript
const categoryStats = await flowManager.getFlowStatistics(appId);
console.log('Most used categories:', categoryStats.byCategory);
```

### 4. Error Detection
```javascript
const problematicFlows = await flowManager.getFlowsWithValidationErrors(appId);
console.log('Flows needing attention:', problematicFlows);
```

## Important Notes

- **Authentication**: Always use a valid Partner App Token
- **App ID Validation**: Ensure the app ID is valid and accessible
- **Rate Limiting**: Be mindful of API rate limits for frequent requests
- **Data Caching**: Consider caching flow data for better performance
- **Error Handling**: Implement proper error handling for production use
- **Monitoring**: Regular monitoring helps maintain flow health
- **Batch Processing**: Use batch operations for multiple apps efficiently
- **Data Export**: Export capabilities help with analysis and reporting
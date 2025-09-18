# Facebook Graph API Reference - Complete Technical Documentation

## Overview

The Facebook Graph API is the primary way to read and write data to Facebook's social graph. It's an HTTP-based API that provides programmatic access to Facebook data, enabling applications to query data, publish stories, manage ads, upload photos, and perform numerous other tasks. All Facebook SDKs and products interact with the Graph API, making it the foundation of Facebook platform development.

## Core Architecture

### Social Graph Structure

The Graph API is built around Facebook's "social graph" - a representation of information on Facebook consisting of:

#### Nodes
Individual objects with unique IDs representing:
- **Users**: Individual Facebook users
- **Pages**: Facebook Pages and business profiles
- **Groups**: Facebook Groups and communities
- **Posts**: Content posts and stories
- **Photos**: Image content and media
- **Comments**: User interactions and responses
- **Events**: Facebook Events and gatherings

#### Edges
Connections between nodes representing relationships:
- **User → Photos**: Photos a user has posted
- **Page → Posts**: Posts published by a page
- **Post → Comments**: Comments on a specific post
- **User → Friends**: A user's friend connections

#### Fields
Properties and attributes of nodes:
- **User fields**: name, email, birthday, location
- **Page fields**: name, category, description, follower_count
- **Post fields**: message, created_time, reactions, shares

### API Foundation

**HTTP-Based Protocol**:
- Built on HTTP/1.1 protocol
- HTTPS required for all endpoints
- Works with any language having HTTP libraries
- Browser-compatible for direct testing

**RESTful Design**:
- GET for reading data
- POST for creating and updating
- DELETE for removing objects
- Standard HTTP status codes

## Technical Infrastructure

### Host URLs

#### Primary Endpoint
**Standard API calls**: `graph.facebook.com`
```bash
curl -X GET "https://graph.facebook.com/v23.0/me?access_token=ACCESS_TOKEN"
```

#### Video Upload Endpoint
**Video uploads only**: `graph-video.facebook.com`
```bash
curl -X POST "https://graph-video.facebook.com/v23.0/me/videos"
```

#### HTTPS Requirements
- **Mandatory HTTPS**: All API calls must use HTTPS
- **TLS/SSL**: Valid certificates required
- **HSTS Policy**: HTTP Strict Transport Security enabled with includeSubdomains

### Access Tokens

Access tokens serve two primary functions:

#### User Authentication
- **Password-free access**: Apps access user data without passwords
- **Secure authorization**: Users grant specific permissions to apps
- **Scope control**: Tokens define what data can be accessed

#### App Identification
- **App verification**: Identifies the requesting application
- **User identification**: Links requests to specific users
- **Permission scope**: Defines the type of accessible data

#### Token Types
- **User Access Tokens**: Access user-specific data
- **App Access Tokens**: Access app-level data and perform app operations
- **Page Access Tokens**: Manage and access Page data
- **Client Access Tokens**: Limited client-side operations

### API Versioning

#### Version Format
```bash
# Versioned API call
curl -X GET "https://graph.facebook.com/v23.0/USER_ID"

# Unversioned (defaults to oldest available version - not recommended)
curl -X GET "https://graph.facebook.com/USER_ID"
```

#### Current Version
**Latest Version**: v23.0
**Release Schedule**: Quarterly releases
**Version Lifecycle**: Each version supported for specific timeframe

#### Version Management
- **Explicit versioning**: Always specify version in production
- **Default behavior**: Unversioned calls use oldest available version
- **Migration planning**: Monitor changelog for version updates
- **Backward compatibility**: Plan for deprecation timelines

## Core API Operations

### Reading Data (GET)

#### Basic Node Access
```bash
# Get user information
curl -X GET "https://graph.facebook.com/v23.0/USER_ID?access_token=ACCESS_TOKEN"

# Response format
{
  "name": "User Name",
  "id": "USER_ID"
}
```

#### Field Selection
```bash
# Request specific fields
curl -X GET "https://graph.facebook.com/v23.0/USER_ID?fields=id,name,email,picture&access_token=ACCESS_TOKEN"

# Response with requested fields
{
  "id": "USER_ID",
  "name": "User Name",
  "email": "user@example.com",
  "picture": {
    "data": {
      "height": 50,
      "is_silhouette": false,
      "url": "https://profile-picture-url",
      "width": 50
    }
  }
}
```

#### Node Metadata
```bash
# Get all available fields for a node type
curl -X GET "https://graph.facebook.com/v23.0/USER_ID?metadata=1&access_token=ACCESS_TOKEN"

# Metadata response structure
{
  "name": "User Name",
  "metadata": {
    "fields": [
      {
        "name": "id",
        "description": "The app user's App-Scoped User ID",
        "type": "numeric string"
      },
      {
        "name": "age_range",
        "description": "The age segment for this person",
        "type": "agerange"
      }
    ]
  }
}
```

### Edge Connections

#### Accessing Related Objects
```bash
# Get user's photos
curl -X GET "https://graph.facebook.com/v23.0/USER_ID/photos?access_token=ACCESS_TOKEN"

# Response with related objects
{
  "data": [
    {
      "created_time": "2024-01-15T18:04:10+0000",
      "id": "PHOTO_ID_1"
    },
    {
      "created_time": "2024-01-15T18:01:13+0000",
      "id": "PHOTO_ID_2"
    }
  ],
  "paging": {
    "cursors": {
      "before": "BEFORE_CURSOR",
      "after": "AFTER_CURSOR"
    }
  }
}
```

#### Pagination
```bash
# Get next page of results
curl -X GET "https://graph.facebook.com/v23.0/USER_ID/photos?after=AFTER_CURSOR&access_token=ACCESS_TOKEN"
```

### Special Endpoints

#### /me Endpoint
The `/me` endpoint is a special shortcut that resolves to the ID of the user or Page whose access token is being used:

```bash
# Using /me endpoint
curl -X GET "https://graph.facebook.com/v23.0/me?access_token=ACCESS_TOKEN"

# Equivalent to
curl -X GET "https://graph.facebook.com/v23.0/USER_ID?access_token=ACCESS_TOKEN"
```

### Creating Data (POST)

#### Publishing Content
```bash
# Post to Page feed
curl -X POST "https://graph.facebook.com/v23.0/PAGE_ID/feed" \
  -d "message=Hello World" \
  -d "access_token=ACCESS_TOKEN"

# Response
{
  "id": "PAGE_ID_POST_ID"
}
```

#### Read-After-Write
```bash
# Post with read-after-write to get additional fields
curl -X POST "https://graph.facebook.com/v23.0/PAGE_ID/feed" \
  -d "message=Hello World" \
  -d "fields=created_time,from,id,message" \
  -d "access_token=ACCESS_TOKEN"

# Enhanced response
{
  "created_time": "2024-01-15T22:04:21+0000",
  "from": {
    "name": "My Facebook Page",
    "id": "PAGE_ID"
  },
  "id": "POST_ID",
  "message": "Hello World"
}
```

### Updating Data (POST)

#### Modifying Existing Objects
```bash
# Update user email
curl -X POST "https://graph.facebook.com/v23.0/USER_ID" \
  -d "email=newemail@example.com" \
  -d "access_token=ACCESS_TOKEN"
```

### Deleting Data (DELETE)

#### Removing Objects
```bash
# Delete a photo
curl -X DELETE "https://graph.facebook.com/v23.0/PHOTO_ID?access_token=ACCESS_TOKEN"

# Success response
{
  "success": true
}
```

**Deletion Rules**:
- Generally can only delete objects you created
- Some objects may have specific deletion requirements
- Refer to individual node documentation for deletion policies

## Advanced Features

### Complex Parameters

#### List Parameters
```bash
# JSON array format
curl -X GET "https://graph.facebook.com/v23.0/PAGE_ID/insights" \
  -d "metric=[\"page_fans\",\"page_impressions\",\"page_engaged_users\"]" \
  -d "access_token=ACCESS_TOKEN"
```

#### Object Parameters
```bash
# JSON object format
curl -X POST "https://graph.facebook.com/v23.0/AD_ACCOUNT_ID/campaigns" \
  -d "targeting={\"geo_locations\":{\"countries\":[\"US\"]},\"age_min\":21}" \
  -d "access_token=ACCESS_TOKEN"
```

### Field Expansion

#### Nested Requests
```bash
# Get user and their page information in one request
curl -X GET "https://graph.facebook.com/v23.0/USER_ID?fields=name,accounts{name,category,fan_count}&access_token=ACCESS_TOKEN"

# Response with nested data
{
  "name": "User Name",
  "accounts": {
    "data": [
      {
        "name": "Page Name",
        "category": "Business",
        "fan_count": 1500,
        "id": "PAGE_ID"
      }
    ]
  },
  "id": "USER_ID"
}
```

#### Field Filtering
```bash
# Limit fields in nested objects
curl -X GET "https://graph.facebook.com/v23.0/PAGE_ID/posts?fields=message,created_time,comments.limit(5){message,from}&access_token=ACCESS_TOKEN"
```

### Batch Requests

#### Multiple Operations in Single Call
```bash
curl -X POST "https://graph.facebook.com/v23.0" \
  -d "access_token=ACCESS_TOKEN" \
  -d "batch=[
    {\"method\":\"GET\",\"relative_url\":\"me\"},
    {\"method\":\"GET\",\"relative_url\":\"me/friends\"},
    {\"method\":\"POST\",\"relative_url\":\"me/feed\",\"body\":\"message=Test\"}
  ]"

# Batch response
[
  {
    "code": 200,
    "headers": [{"name": "Content-Type", "value": "application/json"}],
    "body": "{\"name\":\"User Name\",\"id\":\"USER_ID\"}"
  },
  {
    "code": 200,
    "headers": [{"name": "Content-Type", "value": "application/json"}],
    "body": "{\"data\":[],\"summary\":{\"total_count\":0}}"
  },
  {
    "code": 200,
    "headers": [{"name": "Content-Type", "value": "application/json"}],
    "body": "{\"id\":\"POST_ID\"}"
  }
]
```

#### Batch Request Features
- **Maximum 50 requests** per batch
- **Dependency handling**: Results from one request can be used in subsequent requests
- **Error isolation**: Failed requests don't affect others in the batch
- **Performance optimization**: Reduces network round trips

## Error Handling

### Error Response Format
```json
{
  "error": {
    "message": "Error message description",
    "type": "ErrorType",
    "code": 12345,
    "error_subcode": 67890,
    "fbtrace_id": "trace_id_for_debugging"
  }
}
```

### Common Error Types

#### Authentication Errors
- **Code 190**: Invalid access token
- **Code 102**: API session limit reached
- **Code 104**: Invalid access token type

#### Permission Errors
- **Code 200**: Permission denied
- **Code 10**: Application not authorized
- **Code 294**: Permission required

#### Rate Limiting
- **Code 4**: Application request limit reached
- **Code 17**: User request limit reached
- **Code 613**: Rate limit exceeded

#### Data Errors
- **Code 100**: Invalid parameter
- **Code 110**: Invalid user ID
- **Code 803**: Object not found

### Error Recovery Strategies

#### Exponential Backoff
```python
import time
import requests

def api_call_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(url)
            if response.status_code == 200:
                return response.json()
            elif response.status_code == 429:  # Rate limited
                wait_time = 2 ** attempt
                time.sleep(wait_time)
            else:
                response.raise_for_status()
        except requests.RequestException as e:
            if attempt == max_retries - 1:
                raise e
            time.sleep(2 ** attempt)
```

## Security and Best Practices

### Secure API Calls

#### HTTPS Requirements
- **Always use HTTPS**: Never make API calls over HTTP
- **Certificate validation**: Verify SSL certificates
- **Secure token storage**: Protect access tokens from exposure

#### Access Token Security
```javascript
// Good: Server-side token usage
const token = process.env.FB_ACCESS_TOKEN;
const response = await fetch(`https://graph.facebook.com/v23.0/me?access_token=${token}`);

// Bad: Client-side token exposure
// Never include tokens in client-side code
```

### Rate Limiting Management

#### Respect Rate Limits
- **Monitor usage**: Track API call frequency
- **Implement backoff**: Use exponential backoff for rate limit responses
- **Distribute load**: Spread API calls over time
- **Cache responses**: Reduce unnecessary API calls

#### Application-Level Limits
```javascript
class APIRateLimiter {
  constructor(callsPerSecond = 10) {
    this.callsPerSecond = callsPerSecond;
    this.calls = [];
  }

  async makeCall(apiFunction) {
    const now = Date.now();
    this.calls = this.calls.filter(time => now - time < 1000);
    
    if (this.calls.length >= this.callsPerSecond) {
      const waitTime = 1000 - (now - this.calls[0]);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.calls.push(now);
    return await apiFunction();
  }
}
```

### Data Privacy and Compliance

#### Minimal Data Collection
- **Request only necessary fields**: Don't request data you don't need
- **Respect user permissions**: Only access data users have explicitly allowed
- **Regular permission audits**: Review and remove unnecessary permissions

#### GDPR Compliance
```javascript
// Implement data deletion endpoint
app.delete('/user-data/:userId', async (req, res) => {
  const userId = req.params.userId;
  
  // Delete all stored user data
  await deleteUserData(userId);
  
  // Confirm deletion to Facebook
  res.json({ success: true, message: 'User data deleted' });
});
```

## Performance Optimization

### Efficient API Usage

#### Field Selection Optimization
```bash
# Inefficient: Getting all default fields
curl -X GET "https://graph.facebook.com/v23.0/PAGE_ID/posts?access_token=ACCESS_TOKEN"

# Efficient: Only request needed fields
curl -X GET "https://graph.facebook.com/v23.0/PAGE_ID/posts?fields=id,message,created_time&access_token=ACCESS_TOKEN"
```

#### Pagination Optimization
```javascript
async function getAllPosts(pageId, accessToken) {
  let allPosts = [];
  let url = `https://graph.facebook.com/v23.0/${pageId}/posts?access_token=${accessToken}`;
  
  while (url) {
    const response = await fetch(url);
    const data = await response.json();
    
    allPosts = allPosts.concat(data.data);
    url = data.paging?.next;
  }
  
  return allPosts;
}
```

### Caching Strategies

#### Response Caching
```javascript
const cache = new Map();

async function cachedApiCall(endpoint, ttl = 300000) { // 5 minutes TTL
  const cacheKey = endpoint;
  const cached = cache.get(cacheKey);
  
  if (cached && Date.now() - cached.timestamp < ttl) {
    return cached.data;
  }
  
  const response = await fetch(`https://graph.facebook.com/v23.0/${endpoint}`);
  const data = await response.json();
  
  cache.set(cacheKey, {
    data: data,
    timestamp: Date.now()
  });
  
  return data;
}
```

## Development Tools and Testing

### Graph API Explorer

The Graph API Explorer is a web-based tool for testing API calls:

#### Features
- **Interactive testing**: Test API calls directly in browser
- **Token management**: Generate and manage access tokens
- **Permission visualization**: See what permissions are required
- **Response inspection**: Examine API responses in detail

#### Usage Example
1. Navigate to [developers.facebook.com/tools/explorer](https://developers.facebook.com/tools/explorer)
2. Select your app and generate appropriate access token
3. Enter API endpoint (e.g., `me?fields=id,name,email`)
4. Click "Submit" to see response

### Debugging Tools

#### Access Token Debugger
```bash
# Debug access token
curl -X GET "https://graph.facebook.com/v23.0/debug_token?input_token=TOKEN_TO_DEBUG&access_token=APP_TOKEN"

# Response shows token details
{
  "data": {
    "app_id": "APP_ID",
    "type": "USER",
    "application": "App Name",
    "expires_at": 1234567890,
    "is_valid": true,
    "scopes": ["public_profile", "email"],
    "user_id": "USER_ID"
  }
}
```

#### Request Debugging
```bash
# Add debug parameter to see additional information
curl -X GET "https://graph.facebook.com/v23.0/me?debug=all&access_token=ACCESS_TOKEN"
```

## Integration Patterns

### Real-Time Data Synchronization

#### Webhook Integration
```javascript
// Express.js webhook handler
app.post('/webhook', (req, res) => {
  const body = req.body;
  
  if (body.object === 'page') {
    body.entry.forEach(entry => {
      entry.changes.forEach(change => {
        if (change.field === 'feed') {
          handlePageFeedUpdate(change.value);
        }
      });
    });
  }
  
  res.status(200).send('EVENT_RECEIVED');
});
```

#### Polling Alternative
```javascript
// Periodic data polling
async function pollForUpdates(pageId, accessToken, lastCheck) {
  const since = Math.floor(lastCheck.getTime() / 1000);
  const response = await fetch(
    `https://graph.facebook.com/v23.0/${pageId}/posts?since=${since}&access_token=${accessToken}`
  );
  
  const data = await response.json();
  return data.data;
}
```

### Business Integration Scenarios

#### CRM Integration
```javascript
// Sync Facebook user data to CRM
async function syncUserToCRM(facebookUserId, accessToken) {
  const userData = await fetch(
    `https://graph.facebook.com/v23.0/${facebookUserId}?fields=id,name,email&access_token=${accessToken}`
  );
  
  const user = await userData.json();
  
  // Update CRM system
  await updateCRMContact({
    facebookId: user.id,
    name: user.name,
    email: user.email
  });
}
```

#### Social Media Management
```javascript
// Multi-platform posting
async function postToFacebook(pageId, message, accessToken) {
  const response = await fetch(`https://graph.facebook.com/v23.0/${pageId}/feed`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: new URLSearchParams({
      message: message,
      access_token: accessToken
    })
  });
  
  return await response.json();
}
```

## Platform Integration

### SDK Integration

#### JavaScript SDK
```html
<script>
  window.fbAsyncInit = function() {
    FB.init({
      appId: 'YOUR_APP_ID',
      cookie: true,
      xfbml: true,
      version: 'v23.0'
    });
  };

  // Graph API call using SDK
  FB.api('/me', {fields: 'name,email'}, function(response) {
    console.log('User data:', response);
  });
</script>
```

#### Mobile SDK Integration
```swift
// iOS Swift example
import FBSDKCoreKit

let request = GraphRequest(graphPath: "me", parameters: ["fields": "id,name,email"])
request.start { connection, result, error in
    if let error = error {
        print("Graph API error: \(error)")
    } else if let result = result as? [String: Any] {
        print("User data: \(result)")
    }
}
```

### Server-Side Integration

#### Node.js Example
```javascript
const axios = require('axios');

class FacebookAPI {
  constructor(accessToken) {
    this.accessToken = accessToken;
    this.baseURL = 'https://graph.facebook.com/v23.0';
  }

  async get(endpoint, params = {}) {
    const url = `${this.baseURL}/${endpoint}`;
    const response = await axios.get(url, {
      params: { ...params, access_token: this.accessToken }
    });
    return response.data;
  }

  async post(endpoint, data = {}) {
    const url = `${this.baseURL}/${endpoint}`;
    const response = await axios.post(url, {
      ...data,
      access_token: this.accessToken
    });
    return response.data;
  }
}

// Usage
const fb = new FacebookAPI(process.env.FB_ACCESS_TOKEN);
const userData = await fb.get('me', { fields: 'id,name,email' });
```

#### Python Example
```python
import requests

class FacebookGraphAPI:
    def __init__(self, access_token):
        self.access_token = access_token
        self.base_url = 'https://graph.facebook.com/v23.0'

    def get(self, endpoint, params=None):
        if params is None:
            params = {}
        
        params['access_token'] = self.access_token
        response = requests.get(f'{self.base_url}/{endpoint}', params=params)
        return response.json()

    def post(self, endpoint, data=None):
        if data is None:
            data = {}
        
        data['access_token'] = self.access_token
        response = requests.post(f'{self.base_url}/{endpoint}', data=data)
        return response.json()

# Usage
fb = FacebookGraphAPI(os.getenv('FB_ACCESS_TOKEN'))
user_data = fb.get('me', {'fields': 'id,name,email'})
```

## Webhooks Integration

### Real-Time Updates

Webhooks provide real-time notifications about changes to subscribed objects:

```javascript
// Webhook endpoint verification
app.get('/webhook', (req, res) => {
  const VERIFY_TOKEN = process.env.WEBHOOK_VERIFY_TOKEN;
  
  const mode = req.query['hub.mode'];
  const token = req.query['hub.verify_token'];
  const challenge = req.query['hub.challenge'];
  
  if (mode && token === VERIFY_TOKEN) {
    console.log('Webhook verified');
    res.status(200).send(challenge);
  } else {
    res.sendStatus(403);
  }
});

// Handle webhook events
app.post('/webhook', (req, res) => {
  const body = req.body;
  
  if (body.object === 'page') {
    body.entry.forEach(entry => {
      const webhookEvent = entry.messaging[0];
      console.log('Webhook event:', webhookEvent);
    });
  }
  
  res.status(200).send('EVENT_RECEIVED');
});
```

## API Version Information

**Current Version**: v23.0

### Version Management
- **Quarterly releases**: New versions released every quarter
- **Version support**: Each version supported for specified duration
- **Migration planning**: Plan for version upgrades and deprecations
- **Backward compatibility**: Test applications with new versions

### Changelog Monitoring
- **Breaking changes**: Monitor for API breaking changes
- **New features**: Stay updated with new API capabilities
- **Deprecation notices**: Plan for deprecated feature replacements
- **Performance improvements**: Leverage performance enhancements

## Related Documentation

- [Facebook Login & OAuth](../facebook-login-oauth.md)
- [Permissions Reference](../permissions-reference.md)
- [Webhooks](../webhooks.md)
- [Business Manager](../business-manager.md)
- [Marketing API](../marketing-api-overview.md)

## Resources and Tools

### Development Resources
- **Graph API Explorer**: Interactive API testing tool
- **Access Token Debugger**: Debug and inspect access tokens
- **Webhook Tester**: Test webhook endpoints and payloads
- **SDK Documentation**: Platform-specific implementation guides

### Official Tools
- **Graph API Reference**: Complete API endpoint documentation
- **Changelog**: Version-specific changes and updates
- **Best Practices**: Recommended implementation patterns
- **Community Forums**: Developer community support and discussions

---

*This documentation covers Facebook Graph API v23.0. For the most current information, always refer to the official Facebook Developer Documentation.*

# Facebook Webhooks - Complete Technical Documentation

## Overview

Facebook Webhooks provide real-time HTTP notifications about changes to specific objects in Meta's social graph. Instead of continuously polling the Graph API for changes, webhooks push updates directly to your server endpoint when subscribed events occur. This dramatically reduces API rate limiting issues and enables real-time responsiveness in applications.

## Core Concepts

### Real-Time Notifications

Webhooks eliminate the need for polling by delivering instant notifications when:
- Users modify their profile information
- Page content is updated
- Comments are posted on your pages
- Ad account changes occur
- Instagram content is modified
- Business account changes happen

### Object-Field Subscription Model

Webhooks operate on an object-field subscription system:

#### Objects
Different types of entities in Meta's social graph:
- **User**: Individual Facebook users
- **Page**: Facebook Pages and their content
- **Ad Account**: Advertising account changes
- **Application**: App-specific updates
- **Instagram**: Instagram account updates
- **WhatsApp Business Account**: WhatsApp business updates
- **Certificate Transparency**: Security certificate updates
- **Permissions**: User permission changes

#### Fields
Specific properties within each object type:
- **User fields**: photos, email, name, location
- **Page fields**: posts, comments, ratings, messaging
- **Ad Account fields**: campaigns, ad sets, ads, insights

#### Sample Notification Structure
```json
{
  "entry": [
    {
      "time": 1520383571,
      "changes": [
        {
          "field": "photos",
          "value": {
            "verb": "update",
            "object_id": "10211885744794461"
          }
        }
      ],
      "id": "10210299214172187",
      "uid": "10210299214172187"
    }
  ],
  "object": "user"
}
```

## Technical Requirements

### HTTPS Server Requirements

**Mandatory HTTPS Support**:
- Valid TLS/SSL certificate required
- Self-signed certificates not supported
- Secure endpoint capable of processing POST requests
- Ability to handle both verification and notification requests

**Server Capabilities**:
- Process HTTP GET requests for verification
- Handle HTTP POST requests for notifications
- Maintain stable endpoint availability
- Support JSON payload parsing

### Endpoint Responsibilities

Your webhook endpoint must handle two distinct types of requests:

#### 1. Verification Requests (GET)
Sent during webhook configuration to verify endpoint ownership.

#### 2. Event Notifications (POST)
Real-time notifications about subscribed object changes.

## Verification Process

### Verification Request Structure

When configuring webhooks, Facebook sends a GET request to verify your endpoint:

```
GET https://your-domain.com/webhooks?
  hub.mode=subscribe&
  hub.challenge=1158201444&
  hub.verify_token=your_verification_token
```

#### Query Parameters

| Parameter | Description |
|-----------|-------------|
| `hub.mode` | Always "subscribe" for verification requests |
| `hub.challenge` | Random string that must be echoed back |
| `hub.verify_token` | Token you specify in App Dashboard for verification |

### Verification Response Requirements

Your endpoint must:

1. **Verify Token Match**: Confirm `hub.verify_token` matches your configured verification token
2. **Echo Challenge**: Return the `hub.challenge` value exactly as received
3. **HTTP 200 Response**: Respond with 200 OK status code

#### Example Verification Handler

```python
def verify_webhook(request):
    verify_token = "your_verification_token"
    
    if request.GET.get('hub.verify_token') == verify_token:
        return HttpResponse(request.GET.get('hub.challenge'))
    else:
        return HttpResponse('Invalid verification token', status=403)
```

### PHP Parameter Handling Note

PHP converts periods (.) to underscores (_) in parameter names:
- `hub.mode` becomes `hub_mode`
- `hub.challenge` becomes `hub_challenge`
- `hub.verify_token` becomes `hub_verify_token`

## Event Notifications

### Notification Structure

Event notifications are sent as HTTP POST requests with JSON payloads:

```http
POST /webhooks HTTP/1.1
Host: your-domain.com
Content-Type: application/json
X-Hub-Signature-256: sha256={signature}
Content-Length: 311

{
  "entry": [
    {
      "time": 1520383571,
      "changes": [
        {
          "field": "photos",
          "value": {
            "verb": "update",
            "object_id": "10211885744794461"
          }
        }
      ],
      "id": "10210299214172187",
      "uid": "10210299214172187"
    }
  ],
  "object": "user"
}
```

### Common Payload Properties

| Property | Type | Description |
|----------|------|-------------|
| `object` | string | Type of object that changed (user, page, ad_account, etc.) |
| `entry` | array | Array of change entries |
| `entry[].id` | string | ID of the object that changed |
| `entry[].uid` | string | User ID (for user objects) |
| `entry[].time` | integer | Unix timestamp of the change |
| `entry[].changes` | array | Array of specific field changes |
| `entry[].changes[].field` | string | Name of the field that changed |
| `entry[].changes[].value` | object | Details about the change |

### Payload Configuration Options

When configuring webhooks, you can specify:

#### Include Values
- **Enabled**: Payloads include both field names and new values
- **Disabled**: Payloads include only field names (for privacy)

#### Example with Values Enabled
```json
{
  "field": "email",
  "value": {
    "email": "new_email@example.com",
    "verified": true
  }
}
```

#### Example with Values Disabled
```json
{
  "field": "email",
  "value": {
    "verb": "update"
  }
}
```

### Payload Security Validation

Facebook signs all payloads with SHA256 for security verification.

#### Signature Verification Process

1. **Extract Signature**: Get signature from `X-Hub-Signature-256` header
2. **Generate Local Signature**: Create SHA256 hash using payload and app secret
3. **Compare Signatures**: Verify signatures match

#### Implementation Example

```python
import hashlib
import hmac

def verify_signature(payload, signature, app_secret):
    expected_signature = hmac.new(
        app_secret.encode('utf-8'),
        payload,
        hashlib.sha256
    ).hexdigest()
    
    # Remove 'sha256=' prefix from received signature
    received_signature = signature.replace('sha256=', '')
    
    return hmac.compare_digest(expected_signature, received_signature)
```

### Response Requirements

#### Success Response
- **Status Code**: 200 OK
- **Response Time**: Respond quickly to avoid timeouts
- **Body**: Response body content is ignored

#### Failure Handling
If your endpoint doesn't respond with 200 OK:
- **Immediate Retry**: Facebook retries immediately
- **Exponential Backoff**: Additional retries with decreasing frequency
- **36-Hour Window**: Retries continue for up to 36 hours
- **Final Drop**: Unacknowledged notifications are dropped after 36 hours

### Batching and Frequency

#### Notification Batching
- **Maximum Batch Size**: Up to 1000 updates per request
- **No Guaranteed Batching**: Single notifications may be sent individually
- **Deduplication Required**: Your server must handle potential duplicates

#### Retry Logic
Implement deduplication using:
- Unique change timestamps
- Object ID + field combinations
- Idempotent processing logic

## Webhook Topics and Objects

### User Object

Subscribe to user profile changes:

#### Available Fields
- `about`: About section updates
- `birthday`: Birthday information changes
- `email`: Email address modifications
- `hometown`: Hometown updates
- `location`: Current location changes
- `name`: Name changes
- `photos`: Photo uploads and modifications
- `picture`: Profile picture changes
- `posts`: Post creation and updates
- `videos`: Video uploads and changes

#### Example User Notification
```json
{
  "object": "user",
  "entry": [
    {
      "id": "user_id",
      "time": 1520383571,
      "changes": [
        {
          "field": "email",
          "value": {
            "email": "new_email@example.com",
            "verified": true
          }
        }
      ]
    }
  ]
}
```

### Page Object

Subscribe to Facebook Page activity:

#### Available Fields
- `posts`: New posts and post updates
- `comments`: Comments on page posts
- `ratings`: Page rating changes
- `conversations`: Messenger conversations
- `messaging_posts`: Messaging-related posts
- `messaging_referrals`: Referral events
- `feed`: Feed updates
- `mention`: Page mentions

#### Permission Requirements
Page webhooks require:
- Page admin with MODERATE privileges
- `pages_manage_metadata` permission granted to your app

#### Example Page Notification
```json
{
  "object": "page",
  "entry": [
    {
      "id": "page_id",
      "time": 1520383571,
      "changes": [
        {
          "field": "posts",
          "value": {
            "verb": "add",
            "post_id": "page_id_post_id",
            "sender_name": "Page Name"
          }
        }
      ]
    }
  ]
}
```

### Ad Account Object

Subscribe to advertising account changes:

#### Available Fields
- `campaigns`: Campaign creation, updates, and deletion
- `adsets`: Ad set modifications
- `ads`: Individual ad changes
- `account`: Account-level updates
- `billing`: Billing and payment updates

#### Example Ad Account Notification
```json
{
  "object": "ad_account",
  "entry": [
    {
      "id": "act_ad_account_id",
      "time": 1520383571,
      "changes": [
        {
          "field": "campaigns",
          "value": {
            "verb": "update",
            "object_id": "campaign_id"
          }
        }
      ]
    }
  ]
}
```

### Instagram Object

Subscribe to Instagram business account changes:

#### Available Fields
- `comments`: Comments on Instagram posts
- `mentions`: Mentions in Instagram content
- `story_insights`: Story performance insights

#### Example Instagram Notification
```json
{
  "object": "instagram",
  "entry": [
    {
      "id": "instagram_account_id",
      "time": 1520383571,
      "changes": [
        {
          "field": "comments",
          "value": {
            "verb": "add",
            "comment_id": "comment_id"
          }
        }
      ]
    }
  ]
}
```

### WhatsApp Business Account Object

Subscribe to WhatsApp Business API changes:

#### Available Fields
- `messages`: Message events
- `message_template_status_update`: Template approval status
- `phone_number_name_update`: Phone number name changes
- `phone_number_quality_update`: Phone number quality ratings

### Application Object

Subscribe to app-specific events:

#### Available Fields
- `app_installed`: App installation events
- `app_removed`: App removal events

### Permissions Object

Subscribe to permission changes:

#### Available Fields
- `permission_granted`: User grants permission
- `permission_revoked`: User revokes permission

### Certificate Transparency Object

Subscribe to certificate transparency events for security monitoring.

## Development and Testing

### Development Mode Limitations

Apps in development mode receive only:
- **Test Notifications**: Triggered manually through App Dashboard
- **Role-Based Events**: Events from users with roles in your app
- **Limited Scope**: Real user events not delivered until app is live

#### Messenger Exception
Messenger webhooks work differently in development mode - refer to Messenger Platform documentation for specific behavior.

### Testing Webhooks

#### Manual Testing
1. Configure webhook in App Dashboard
2. Send test notifications using "Test" button
3. Verify payload reception and processing
4. Check signature validation

#### Automated Testing
```python
def test_webhook_endpoint():
    payload = {
        "object": "user",
        "entry": [
            {
                "id": "test_user",
                "time": int(time.time()),
                "changes": [
                    {
                        "field": "photos",
                        "value": {"verb": "add", "object_id": "test_photo"}
                    }
                ]
            }
        ]
    }
    
    response = requests.post(
        'https://your-domain.com/webhooks',
        json=payload,
        headers={'X-Hub-Signature-256': generate_signature(payload)}
    )
    
    assert response.status_code == 200
```

## App Review and Permissions

### Review Requirements

#### Webhook-Specific Review
- **No Review Required**: Webhooks themselves don't require app review
- **Permission Dependencies**: Must have appropriate permissions for data access
- **Object Access**: Requires permissions corresponding to webhook object types

#### Permission Requirements
To receive webhook notifications, you need:
- **Appropriate Permissions**: Permissions matching the data type
- **User Authorization**: Users must grant your app access to their data
- **Active Permissions**: Permissions must be actively granted and not expired

#### Example Permission Mapping
| Webhook Object | Required Permission |
|----------------|-------------------|
| User photos | `user_photos` |
| User email | `email` |
| Page posts | `pages_read_engagement` |
| Page messaging | `pages_messaging` |

### Production Requirements

Before going live:
1. **Complete App Review**: For all required permissions
2. **Verify Endpoint**: Ensure webhook endpoint is stable and responsive
3. **Test Thoroughly**: Test all subscribed webhook fields
4. **Monitor Performance**: Set up logging and monitoring
5. **Handle Errors**: Implement robust error handling and retry logic

## Advanced Features

### Mutual TLS (mTLS)

For enhanced security, Facebook supports mutual TLS authentication.

#### mTLS Configuration

When enabled, Facebook presents a client certificate for webhook requests:

#### Certificate Details
- **Client Certificate CN**: `client.webhooks.fbclientcerts.com`
- **Intermediate CA**: DigiCert SHA2 High Assurance Server CA
- **Root CA**: DigiCert High Assurance EV Root CA

#### Nginx Configuration Example
```nginx
ssl_verify_client       on;
ssl_client_certificate  /etc/ssl/certs/DigiCert_High_Assurance_EV_Root_CA.pem;
ssl_verify_depth        3;

# Verify common name
if ($ssl_client_s_dn ~ "CN=client.webhooks.fbclientcerts.com") {
    return 200 "$ssl_client_s_dn";
}
```

#### AWS Application Load Balancer Configuration
1. Download intermediate certificate to S3 bucket
2. Configure HTTPS listener with mTLS trust store
3. Extract CN from `X-Amzn-Mtls-Clientcert-Subject` header
4. Verify CN equals `client.webhooks.fbclientcerts.com`

### Subscription Management API

Programmatically manage webhook subscriptions using the Graph API.

#### Create Subscription
```bash
curl -X POST \
  "https://graph.facebook.com/v23.0/{app-id}/subscriptions" \
  -H "Content-Type: application/json" \
  -d '{
    "object": "user",
    "callback_url": "https://your-domain.com/webhooks",
    "fields": "photos,email",
    "verify_token": "your_verify_token",
    "access_token": "app_access_token"
  }'
```

#### List Subscriptions
```bash
curl -X GET \
  "https://graph.facebook.com/v23.0/{app-id}/subscriptions?access_token={app_access_token}"
```

#### Update Subscription
```bash
curl -X POST \
  "https://graph.facebook.com/v23.0/{app-id}/subscriptions" \
  -H "Content-Type: application/json" \
  -d '{
    "object": "user",
    "fields": "photos,email,name",
    "access_token": "app_access_token"
  }'
```

#### Delete Subscription
```bash
curl -X DELETE \
  "https://graph.facebook.com/v23.0/{app-id}/subscriptions?object=user&access_token={app_access_token}"
```

## Error Handling and Best Practices

### Endpoint Reliability

#### Response Time Optimization
- **Quick Response**: Respond with 200 OK immediately
- **Async Processing**: Handle payload processing asynchronously
- **Timeout Prevention**: Avoid long-running operations in webhook handler

#### Error Recovery
```python
def webhook_handler(request):
    try:
        # Immediate acknowledgment
        payload = json.loads(request.body)
        
        # Queue for async processing
        celery_task.delay(payload)
        
        return HttpResponse(status=200)
        
    except Exception as e:
        logger.error(f"Webhook error: {e}")
        return HttpResponse(status=500)
```

### Deduplication Strategies

#### Timestamp-Based Deduplication
```python
def process_webhook(entry):
    change_key = f"{entry['id']}_{entry['time']}"
    
    if redis_client.exists(change_key):
        return  # Already processed
        
    redis_client.setex(change_key, 3600, "processed")
    # Process the change
```

#### Content-Based Deduplication
```python
def process_change(change):
    change_hash = hashlib.md5(
        json.dumps(change, sort_keys=True).encode()
    ).hexdigest()
    
    if Change.objects.filter(hash=change_hash).exists():
        return  # Duplicate change
        
    Change.objects.create(
        hash=change_hash,
        data=change,
        processed_at=timezone.now()
    )
```

### Monitoring and Alerting

#### Key Metrics to Monitor
- **Response Rate**: Percentage of successful 200 responses
- **Response Time**: Average webhook processing time
- **Error Rate**: Rate of failed webhook processing
- **Signature Validation**: Rate of signature verification failures

#### Logging Best Practices
```python
import logging

webhook_logger = logging.getLogger('webhook')

def log_webhook_event(object_type, entries, success=True):
    webhook_logger.info(
        f"Webhook processed: object={object_type}, "
        f"entries={len(entries)}, success={success}"
    )
```

### Security Best Practices

#### Signature Validation
- **Always validate**: Never skip signature verification in production
- **Secure comparison**: Use `hmac.compare_digest()` to prevent timing attacks
- **Log failures**: Monitor signature validation failures for security threats

#### Endpoint Security
- **HTTPS Only**: Never expose webhook endpoints over HTTP
- **Rate Limiting**: Implement rate limiting to prevent abuse
- **Input Validation**: Validate all incoming payload data
- **Error Handling**: Don't expose internal errors in responses

## Integration Patterns

### Real-Time Data Synchronization

#### User Profile Sync
```python
def sync_user_profile(user_id, changes):
    user = User.objects.get(facebook_id=user_id)
    
    for change in changes:
        if change['field'] == 'email':
            user.email = change['value']['email']
        elif change['field'] == 'name':
            user.name = change['value']['name']
    
    user.save()
```

#### Page Content Management
```python
def handle_page_post(page_id, post_data):
    if post_data['verb'] == 'add':
        create_post_mirror(page_id, post_data['post_id'])
    elif post_data['verb'] == 'remove':
        delete_post_mirror(page_id, post_data['post_id'])
```

### Event-Driven Architecture

#### Message Queue Integration
```python
from celery import Celery

app = Celery('webhook_processor')

@app.task
def process_webhook_async(payload):
    for entry in payload['entry']:
        for change in entry['changes']:
            process_change(entry['id'], change)
```

### Analytics and Insights

#### User Engagement Tracking
```python
def track_user_activity(user_id, activity_type):
    Analytics.objects.create(
        user_id=user_id,
        activity_type=activity_type,
        timestamp=timezone.now()
    )
```

## Platform-Specific Implementations

### Webhook Types by Product

#### Standard Webhooks
- User object changes
- Page activity
- Application events
- Permission changes

#### Specialized Webhooks
- **Messenger Webhooks**: Message events, delivery receipts, read receipts
- **Instagram Webhooks**: Comments, mentions, story insights
- **WhatsApp Webhooks**: Message events, template status updates
- **Payments Webhooks**: Payment status updates, purchase events

### Product-Specific Setup Guides

Each product may have specialized webhook configuration:

#### Messenger Platform
- Different event types and payload structures
- Specialized messaging events
- Page messaging permissions required

#### Instagram Business
- Comment moderation events
- Story insights and metrics
- Creator account events

#### WhatsApp Business API
- Message delivery status
- Template approval notifications
- Phone number quality updates

## Troubleshooting

### Common Issues

#### Verification Failures
- **Token Mismatch**: Verify token in code matches App Dashboard
- **Parameter Handling**: Check for PHP parameter name conversion
- **Response Format**: Ensure exact challenge echo without modification

#### Missing Notifications
- **Permission Issues**: Verify app has required permissions
- **Development Mode**: Check if app is in development mode
- **User Authorization**: Confirm users granted necessary permissions

#### Signature Validation Errors
- **Secret Mismatch**: Verify app secret is correct
- **Encoding Issues**: Ensure proper UTF-8 encoding
- **Timing Attacks**: Use secure comparison functions

### Debugging Tools

#### Facebook Debug Tools
- **Webhook Tester**: Test webhook endpoint from App Dashboard
- **Access Token Debugger**: Verify token permissions
- **Graph API Explorer**: Test API calls and permissions

#### Monitoring Tools
```python
def debug_webhook(request):
    logger.debug(f"Headers: {dict(request.headers)}")
    logger.debug(f"Body: {request.body.decode()}")
    logger.debug(f"Method: {request.method}")
```

## API Version Information

**Current Version**: v23.0

### Version Considerations
- Webhook behavior may change between API versions
- Monitor changelog for webhook-related updates
- Test webhook compatibility with new API versions
- Plan for version migration timelines

### Deprecation Handling
- Monitor deprecation announcements
- Update webhook subscriptions for new field names
- Test changes in development before production deployment

## Related Documentation

- [Graph API Reference](../graph-api-reference.md)
- [Permissions Reference](../permissions-reference.md)
- [Facebook Login & OAuth](../facebook-login-oauth.md)
- [Business Manager](../business-manager.md)
- [Pages API](../pages.md)

## Resources and Tools

### Development Resources
- **Sample Applications**: Pre-built webhook handlers for testing
- **SDK Libraries**: Platform-specific webhook handling libraries
- **Testing Tools**: Webhook validation and testing utilities

### Official Tools
- **Webhook Tester**: Built-in testing tool in App Dashboard
- **Graph API Explorer**: API testing and permission verification
- **Access Token Tool**: Token debugging and inspection

---

*This documentation covers Facebook Webhooks API v23.0. For the most current information, always refer to the official Facebook Developer Documentation.*

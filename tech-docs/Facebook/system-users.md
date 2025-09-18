# Facebook System Users - Complete Technical Documentation

## Overview

System Users are special types of users designed for server-to-server application authentication and automation. They enable businesses to manage API access, automate tasks, and securely interact with Facebook Business APIs without requiring human intervention.

## System User Types

### 1. Admin System User
- **Purpose**: For managing business assets and admin-level operations
- **Capabilities**: Can create custom audiences when the Business Manager (BM) contains an advertiser account
- **Permissions**: Full business management access
- **Limitations**: Subject to app status restrictions for certain operations

### 2. Regular System User
- **Purpose**: For general automation and API access
- **Capabilities**: Limited to specific business assets assigned to them
- **Permissions**: Scoped to assigned assets only
- **Custom Audiences**: Cannot create custom audiences unless BM contains an advertiser account

## Business Assets Access

### Access Requirements
System users can only access business assets that are explicitly assigned to them. This includes:
- **Ad Accounts**: Access to specific advertising accounts
- **Pages**: Management of Facebook Pages
- **Product Catalogs**: E-commerce catalog management
- **WhatsApp Business Accounts**: WhatsApp Business API access
- **Business Asset Groups**: Grouped asset collections

### Asset Assignment
- System users must be explicitly granted access to specific business assets
- Access is managed through Business Manager
- Permissions are granular and can be customized per asset
- Asset access can be revoked without affecting the system user itself

## Permissions Model

### Business Manager Permissions
System users inherit permissions based on:
- **Finance Permission**: Editor, Analyst, and other financial roles
- **IP Permission**: Ads rights like Reviewer permissions
- **Asset-Specific Permissions**: Customized per business asset

### Access Level Limitations
The access level of system users is limited by:
- **App Review Status**: Apps in development mode have restricted capabilities
- **Business Verification**: Business verification status affects permissions
- **API Version**: Different API versions may have different permission models

## Custom Audience Requirements

### Creation Rules
System users can create custom audiences under specific conditions:
1. **Business Manager must contain an advertiser account**
2. **System user must have appropriate permissions**
3. **App must have required permissions approved**

### Limitations
- Regular system users cannot create custom audiences without advertiser account in BM
- Admin system users have broader custom audience creation capabilities
- Custom audience access depends on app review status

## Graph API Reference

### System User Object

#### Reading System Users

**Endpoint**: `GET /v23.0/{system-user-id}`

**Example Request**:
```http
GET /v23.0/{system-user-id} HTTP/1.1
Host: graph.facebook.com
```

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | numeric string | System user ID (default) |
| `created_by` | User | The creator of this system user |
| `created_time` | datetime | The creation time of this system user |
| `finance_permission` | string | Financial permission role (Editor, Analyst, etc.) |
| `ip_permission` | string | Ads rights permission role (Reviewer, etc.) |
| `name` | string | Name used to identify this system user (default) |

#### Edge Connections

| Edge | Type | Description |
|------|------|-------------|
| `assigned_business_asset_groups` | Edge<BusinessAssetGroup> | Business asset groups assigned to this user |
| `assigned_pages` | Edge<Page> | Pages assigned to this business scoped user |
| `assigned_product_catalogs` | Edge<ProductCatalog> | Product catalogs assigned to this user |
| `assigned_whatsapp_business_accounts` | Edge<WhatsAppBusinessAccount> | WhatsApp business accounts assigned to the user |

### Creating System Users

**Endpoint**: `POST /{business_id}/system_users`

#### Parameters
System user creation requires specific parameters (refer to business-specific requirements).

#### Return Type
```json
{
  "id": "numeric string"
}
```

**Note**: This endpoint supports read-after-write and will read the node represented by `id` in the return type.

#### Error Codes for Creation

| Code | Description |
|------|-------------|
| 104001 | In order to create a system user, an app must be part of this business. Please add an app and then try again. |
| 3965 | This Business Manager has reached maximum number of admin system user limit. |
| 3949 | This Business Manager has reached maximum number of system user limit. |
| 100 | Invalid parameter |
| 3972 | System users can not have duplicate names. Use another name. |

### System User Management Operations

#### Update Operations
- **System users cannot be updated** through the Graph API endpoint
- Updates must be performed through Business Manager interface

#### Delete Operations
- **System users cannot be deleted** through the Graph API endpoint
- Deletion must be performed through Business Manager interface

### General Error Codes

| Code | Description |
|------|-------------|
| 100 | Invalid parameter |
| 110 | Invalid user id |

## Access Token Management

### Token Generation
System users can generate access tokens for:
- **Long-term tokens**: For sustained API access
- **Short-term tokens**: For temporary operations
- **Scoped tokens**: Limited to specific permissions

### Token Security
- Tokens are tied to specific system users
- Token permissions are limited by system user permissions
- Tokens can be invalidated without affecting the system user
- Regular token rotation is recommended

## Business Manager Integration

### System User Creation in Business Manager
1. **App Requirement**: App must be added to the business first
2. **Admin Approval**: Business admin must approve system user creation
3. **Naming**: System users must have unique names within the business
4. **Limits**: Businesses have limits on total and admin system users

### Permission Assignment
- System users receive permissions through Business Manager
- Permissions are role-based (Finance, IP, etc.)
- Asset-specific permissions are managed separately
- Permission changes take effect immediately

## Best Practices

### Security
1. **Principle of Least Privilege**: Assign minimum required permissions
2. **Regular Audits**: Review system user permissions regularly
3. **Token Rotation**: Implement regular access token rotation
4. **Asset Scoping**: Limit system users to specific business assets

### Management
1. **Descriptive Naming**: Use clear, descriptive names for system users
2. **Documentation**: Document system user purposes and permissions
3. **Monitoring**: Implement logging and monitoring for system user activities
4. **Access Reviews**: Regular reviews of system user access and usage

### Operational
1. **Error Handling**: Implement robust error handling for API calls
2. **Rate Limiting**: Respect API rate limits and implement appropriate backoff
3. **Version Management**: Stay updated with API version changes
4. **Business Verification**: Ensure business is properly verified for full functionality

## Common Use Cases

### Marketing Automation
- **Campaign Management**: Automated campaign creation and optimization
- **Audience Management**: Custom audience creation and updates
- **Reporting**: Automated performance reporting and analytics

### Business Operations
- **Asset Management**: Automated business asset provisioning
- **User Management**: Automated user and permission management
- **Compliance**: Automated compliance monitoring and reporting

### Integration Scenarios
- **CRM Integration**: Syncing customer data with Facebook audiences
- **E-commerce Integration**: Product catalog synchronization
- **Analytics Integration**: Data pipeline for business intelligence

## API Version Notes

**Current Version**: v23.0

### Version Considerations
- System user functionality may vary between API versions
- Newer versions may include additional features or restrictions
- Always check version-specific documentation for latest features
- Plan for API version migrations and deprecations

## Related Documentation

- [Business Manager API](../business-manager.md)
- [Graph API Reference](../graph-api-reference.md)
- [Permissions Reference](../permissions-reference.md)
- [Facebook Login & OAuth](../facebook-login-oauth.md)

---

*This documentation covers Facebook System Users API v23.0. For the most current information, always refer to the official Facebook Developer Documentation.*

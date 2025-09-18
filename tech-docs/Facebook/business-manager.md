# Business Manager API - Complete Documentation

## Overview

Business Manager API enables businesses and agencies to centrally manage Facebook Pages, ad accounts, and apps. The Business Manager API allows you to manage numerous permissions and ad account objects, as well as automate their creation. See the [Business Manager basics article](https://www.facebook.com/help/113163272211510/) in the Business Help Center.

With Business Manager, you can manage advertising and other business objects for the following tasks:

• Permission management
• Running campaigns on behalf of other businesses  
• Creating ad accounts and allocating advertising purchase credits

## Requirements

• Your app needs appropriate [Marketing API Access Level](https://developers.facebook.com/docs/marketing-api/access#limits) to use the Business Manager API. Please note that in some cases your app may require Advanced Access. [Learn more about different access levels](https://developers.facebook.com/docs/graph-api/overview/access-levels/).
• Your app also needs the `business_management` permission.
• Your user also needs the `business_management` permission.

## Create a New Business Manager

Create a new business manager to represent your business. Only create a new business manager if you are setting up a new business manager for yourself or your clients. If you need another ad account or access to another page, you should use your existing manager and asset permissions. Deleting a business manager is not allowed.

### Requirements

To create a business, you need:

• An access token
• A Page ID
• A vertical
• An app-scoped user ID

The Page ID you provide should be the primary page of your business. This page publicly represents your business on Facebook. Whoever creates the business is a manager of this page. If you don't have a page to represent your business on Facebook, [create one](https://www.facebook.com/pages/create/).

The vertical is one of these string constants:

```
ADVERTISING, AUTOMOTIVE, CONSUMER_PACKAGED_GOODS, ECOMMERCE, EDUCATION, ENERGY_AND_UTILITIES, ENTERTAINMENT_AND_MEDIA, FINANCIAL_SERVICES, GAMING, GOVERNMENT_AND_POLITICS, MARKETING, ORGANIZATIONS_AND_ASSOCIATIONS, PROFESSIONAL_SERVICES, RETAIL, TECHNOLOGY, TELECOM, TRAVEL, OTHER
```

### Example Request

```bash
curl \
  -F "name=Pomni Media" \
  -F "vertical=ADVERTISING" \
  -F "primary_page=<PAGE_ID>" \
  -F "timezone_id=1" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<USER_ID>/businesses"
```

### View Business Properties

To view properties for a business, use its ID. The ID will be a part of the response from the request to create a business manager:

```bash
curl "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>?access_token=<ACCESS_TOKEN>"
```

You can also see a list of the business managers you can access:

```bash
curl "https://graph.facebook.com/<API_VERSION>/me/businesses?access_token=<ACCESS_TOKEN>"
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| id | string | The Business Manager ID |
| name | string | The name of the business |
| primary_page | object | The primary page associated with the business |
| vertical | string | The business vertical/industry |
| timezone_id | int | The timezone ID for the business |
| created_time | datetime | When the business was created |

## Update Business Managers

Update fields in the business manager using a `POST` request to `https://graph.facebook.com/{API_VERSION}/{BUSINESS_ID}`.

### Change Business Name

```bash
curl \
-F "name=My Actual Business Name" \
-F "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/"
```

### Change Business Vertical

```bash
curl \
-F "vertical=RETAIL" \
-F "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/"
```

### Update Primary Page

The primary page has to be owned by business manager.

```bash
curl \
  -F "primary_page=<PAGE_ID>" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/"
```

### Update Multiple Fields

You can also update all fields in one POST request:

```bash
curl \
  -F "name=My Actual Business Name" \
  -F "vertical=RETAIL" \
  -F "primary_page=<PAGE_ID>" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/"
```

## Manage People and Roles

There are two types of roles in Business Manager:

| Role | Description |
|------|-------------|
| Admin | Has full control over the Business Manager account and can manage all assets and users |
| Employee | Has limited access and can only access assigned assets with granted permissions |

For more information about roles, see [Set up catalog roles in Business Manager](https://www.facebook.com/business/help/1953352334878186).

Initially the creator of the Business is the only user on the Business and is an Admin.

### Invite People

To add your coworkers to your business you must invite them. To invite someone, provide a valid email address that they have access to. Sending requests to add employees to a business manager is limited. When you reach this limit you will get error code 17, you should resume 24 hours later.

#### Invite as Admin

```bash
curl \
-F "email=some@email.com" \
-F "role=ADMIN" \
-F "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/business_users"
```

#### Invite as Employee

```bash
curl \
-F "email=some@email.com" \
-F "role=EMPLOYEE" \
-F "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/business_users"
```

Facebook sends an email invitation to the work email address you specified. The invitee must check the email and follow the signup process. Once they are done, you can see them in your list of Users.

### People on Business Manager

As of v2.11 we have separate endpoints to get users based on their status. Make a `GET` request to retrieve each group of users.

#### Get All Business Users

Please note that [Advanced Access is required](https://developers.facebook.com/docs/graph-api/overview/access-levels/).

```bash
curl "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/business_users?access_token=<ACCESS_TOKEN>"
```

#### Get System Users

With system-level access:

```bash
curl "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/system_users?access_token=<ACCESS_TOKEN>"
```

#### Get Pending Users

Who are invited to access a business, but who have not yet accepted:

```bash
curl "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/pending_users?access_token=<ACCESS_TOKEN>"
```

### Example Responses

**Active Users:**

```json
{
  "data": [
    {
      "id": "<BUSINESS_ID>",
      "name": "Alpha MK",
      "email": "some@email.com",
      "role": "EMPLOYEE"
    }
  ]
}
```

**Pending Users:**

```json
{
  "data": [
    {
      "id": "<BUSINESS_ID>",
      "email": "some@email.com",
      "role": "EMPLOYEE",
      "status": "PENDING",
      "owner": {
        "id": "USER_ID",
        "name": "Generic Emporium"
      }
    }
  ]
}
```

### Field Definitions

| Field | Description |
|-------|-------------|
| id | The Business-scoped User ID |
| name | The user's name |
| email | The user's email address |
| role | ADMIN or EMPLOYEE |
| status | ACTIVE, PENDING, or REMOVED |
| owner | Information about who invited the user |

### Change Roles

To change an active user's role on your Business provide the User ID for the user.

#### Upgrade Employee to Admin

```bash
curl \
  -F "role=ADMIN" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_SCOPED_USER_ID>"
```

#### Change Admin to Employee

```bash
curl \
  -F "role=EMPLOYEE" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_SCOPED_USER_ID>"
```

#### Change Pending User Role

```bash
curl \
  -F "role=EMPLOYEE" \
  -F "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<PENDING_USER_ID>"
```

### Remove Users

Remove permissions granted to someone based on membership under your business managers. Limit access to ad accounts and pages. If the user has access to ad accounts or pages outside of your Business Manager those permissions do not change.

#### Remove Active User

```bash
curl \
  -X DELETE \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_SCOPED_USER_ID>"
```

#### Cancel Pending User

```bash
curl \
  -X DELETE \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<PENDING_USER_ID>"
```

This removes the users from your Business and removes access to your Business's assets.

## Manage Business Assets

Business Assets are the Facebook objects (for example, pages, apps, and so on) that an administrator manages. An administrator can be a user or business, or in the case of apps, a developer or advertiser. The types of Business Assets are:

• **Pages** - Facebook Pages owned or managed by the business
• **Ad Accounts** - Advertising accounts for running campaigns  
• **Apps** - Facebook applications
• **Catalogs** - Product catalogs for dynamic ads
• **Facebook Pixels** - Tracking pixels for website events

See a sample queries and learn more at [Business Assets](https://developers.facebook.com/docs/marketing-api/business-asset-management/guides/assets/)

### Asset Management Operations

#### Add Assets

```bash
# Add Page to Business
curl \
  -F "page_id=<PAGE_ID>" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/owned_pages"

# Add Ad Account to Business  
curl \
  -F "adaccount_id=act_<AD_ACCOUNT_ID>" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/owned_ad_accounts"
```

#### Grant Asset Permissions

```bash
# Grant Page permissions to user
curl \
  -F "user=<USER_ID>" \
  -F "tasks=[\"MANAGE\",\"CREATE_CONTENT\"]" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<PAGE_ID>/assigned_users"

# Grant Ad Account permissions
curl \
  -F "user=<USER_ID>" \
  -F "tasks=[\"MANAGE\",\"ADVERTISE\"]" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/act_<AD_ACCOUNT_ID>/assigned_users"
```

#### Remove Assets

```bash
# Remove Page from Business
curl \
  -X DELETE \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/owned_pages?page_id=<PAGE_ID>"
```

### Asset Permission Tasks

#### Page Tasks
- **MANAGE**: Full management access
- **CREATE_CONTENT**: Create posts and content  
- **MODERATE**: Moderate comments and messages
- **ADVERTISE**: Create ads for the page
- **ANALYZE**: View insights and analytics

#### Ad Account Tasks  
- **MANAGE**: Full management access
- **ADVERTISE**: Create and manage ads
- **ANALYZE**: View performance data

## Invoices

Business Manager API enables you to view and manage credit sources associated with a business. The API retrieves all invoices that are visible to a Business Manager. This means all invoices that this Business Manager is liable for are visible via the API, not just the invoices belonging to an individual business ID.

### Business Manager Owned Normal Credit Line

For the Marketing API partners who have invoicing enabled, you can take advantage of Business Manager Owned Normal Credit Line.

Facebook Marketing Partners (FBMP) need to contact their sales rep to get your business manager setup for credit. Please make sure ask for Business Manager Owned Normal Credit Line. Once this is setup, you can start using the ad account creation API to start creating ad accounts. Charges will be against your business manager credit line.

For the ad accounts created via the following API, we will dynamically distribute credit across accounts and update credit limits and spend to avoid hitting the credit limits. You will also be able to see summarized credit available and the amount of credit on each ad account.

Today, we only support normal liability, sequential liability is not supported. The process for setting this up will remain unchanged.

### Month-End Invoicing

Once your credit line is set up for a business and the business uses it to run ads, we generate month-end invoices for the business account. To see the business invoices, you need a finance role. For normal administrators and employees of a business, you can assign permissions under `People` in Business Manager. You can also assign finance permissions to system users using Business Manager.

#### Retrieve Invoices

```bash
curl -G \
-d "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/business_invoices?start_date=2017-01-01&end_date=2017-04-01"
```

#### Sample Response

```json
{
  "business_invoices": {
    "data": [
      {
        "id": "1659175694099710",
        "billing_period": "2017-03-01"
      },
      {
        "id": "1303851778395619",
        "billing_period": "2017-01-01"
      },
      {
        "id": "1415846861611329",
        "billing_period": "2017-02-01"
      }
    ],
    "paging": {
      "cursors": {
        "before": "MAZDZD",
        "after": "MgZDZD"
      }
    }
  },
  "id": "249554531892085"
}
```

#### Get Invoice Details with Campaigns

```bash
curl -G \
-d "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/business_invoices?fields=billed_amount_details,billing_period,entity,id,invoice_id,payment_term,type,campaigns&start_date=2019-06-01&end_date=2019-07-01"
```

#### Detailed Response

```json
{
  "business_invoices": {
    "data": [
      {
        "billed_amount_details": {
          "currency": "USD",
          "net_amount": "387.70",
          "tax_amount": "0.00",
          "total_amount": "387.70"
        },
        "billing_period": "2017-03-01",
        "entity": "FBUS",
        "id": "1659175694099710",
        "invoice_id": "22736800",
        "liability_type": "Normal",
        "invoice_type": "Invoice",
        "payment_term": "CUSTOMER",
        "type": "Invoice",
        "campaigns": {
          "data": [
            {
              "campaign_id": "6056967798500",
              "campaign_name": "Nhận ưu đãi",
              "tags": ["hello2"],
              "billed_amount_details": {
                "currency": "USD",
                "net_amount": "207.62",
                "tax_amount": "0.00",
                "total_amount": "207.62"
              }
            },
            {
              "campaign_id": "6056958052500",
              "campaign_name": "Nhận ưu đãi",
              "billed_amount_details": {
                "currency": "USD",
                "net_amount": "180.08",
                "tax_amount": "0.00",
                "total_amount": "180.08"
              },
              "impressions": 100,
              "clicks": 50,
              "conversions": 30
            }
          ]
        }
      }
    ]
  }
}
```

#### Additional Invoice Fields

You can also retrieve the additional invoicing fields:

• `invoice_date` - Date when Facebook generated the invoice
• `due_date` - Date the invoice is due  
• `payment_status` - Shows whether the invoice is `Paid`, `Unpaid`, or `Partially Paid`
• `amount_due` - How much money is currently due, and outstanding, on the invoice
• `download_uri` - Download a PDF of the invoice at this URI

### Funding Source API

To retrieve the extended credit funding source associated with a business manager, send this GET request:

```bash
curl "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/extendedcredits"
```

To setup a funding source for a business, go to settings section of your business on [Business Manager](https://business.facebook.com/).

### Dynamic Credit Allocation

Dynamic Credit Allocation, also known as DCAF, is our credit allocation system for periodically adjusting available credit on per ad account basis. Our automated script runs approximately every 30 minutes and takes your available credit and evenly distributes it across all your active accounts enabled for DCAF. Available credit includes total approved credit minus total outstanding balance. This helps manage spend at your ad account level and allocate funding for each ad account.

A business can also "inactivate" an invoiced ad account and removing the ad account from the list that needs credit assigned. Businesses no longer need to have Facebook manage this status.

## System Users

System users are special users that represent your apps or scripts instead of an individual person. They're designed for server-to-server API calls and have enhanced security features.

### Creating System Users

```bash
curl \
  -F "name=My System User" \
  -F "system_user_role=ADMIN" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/system_users"
```

### System User Benefits

• **Enhanced Security**: More secure than regular user tokens
• **No Expiration**: Tokens don't expire like user tokens
• **Audit Trail**: Better tracking of API usage
• **Separation**: Keep automated processes separate from human users

## Business-to-Business Functions

### Partner Management

Business Manager enables partner relationships for agencies and their clients:

#### Grant Partner Access

```bash
curl \
  -F "partner_business_id=<PARTNER_BUSINESS_ID>" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/partners"
```

#### Share Assets with Partners

```bash
curl \
  -F "business=<PARTNER_BUSINESS_ID>" \
  -F "access_token=<ACCESS_TOKEN>" \
  "https://graph.facebook.com/<API_VERSION>/act_<AD_ACCOUNT_ID>/agencies"
```

### Client-Agency Relationships

• **Asset Sharing**: Share ad accounts, pages, and pixels
• **Permission Management**: Control what partners can access
• **Billing**: Handle client billing through agency Business Manager
• **Reporting**: Consolidated reporting across client accounts

## Best Practices

### Security
1. **Use System Users**: For automated processes, use system users instead of personal accounts
2. **Minimal Permissions**: Grant only necessary permissions to users
3. **Regular Audits**: Regularly review user access and permissions
4. **Token Management**: Secure and rotate access tokens regularly

### Management
1. **Organized Structure**: Keep assets organized within appropriate Business Managers  
2. **Clear Naming**: Use clear, descriptive names for assets and users
3. **Documentation**: Document access patterns and permission structures
4. **Monitoring**: Monitor usage and costs regularly

### Compliance
1. **Data Privacy**: Ensure compliance with data privacy regulations
2. **Terms of Service**: Follow Facebook's terms of service and policies
3. **Attribution**: Properly attribute shared assets and permissions
4. **Record Keeping**: Maintain records of business relationships and asset sharing

## Error Handling

### Common Error Codes

| Error Code | Description |
|------------|-------------|
| 17 | User request limit reached. Wait 24 hours before retrying |
| 100 | Invalid parameter |
| 190 | Invalid OAuth 2.0 Access Token |
| 200 | Permissions error |
| 368 | The action attempted has been deemed abusive or is otherwise disallowed |

### Rate Limiting

• Business user invitation requests are limited
• Use appropriate delays between requests
• Monitor error responses for rate limit indicators
• Implement exponential backoff for failed requests

---

**API Version:** v23.0

**Documentation URL:** https://developers.facebook.com/docs/marketing-api/business-manager-api/

**Related Documentation:**
- [Business Assets Management](https://developers.facebook.com/docs/marketing-api/business-asset-management)
- [System Users](https://developers.facebook.com/docs/marketing-api/system-users)
- [Business API Reference](https://developers.facebook.com/docs/marketing-api/reference/business)

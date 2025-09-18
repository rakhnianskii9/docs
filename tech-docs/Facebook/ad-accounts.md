# Ad Account - Complete Documentation

## Overview

Represents a business, person or other entity who creates and manages ads on Facebook. Multiple people can manage an account, and each person can have one or more levels of access to an account, see [Business Manager API](https://developers.facebook.com/docs/marketing-api/business-manager-api).

In response to Apple's new policy, we are announcing breaking changes that will affect SDKAdNetwork, Marketing API and Ads Insights API endpoints.

To learn more about how Apple's iOS 14.5 requirements will impact Facebook advertising, visit our Business Help Center articles and changelog:

• [Facebook SDK for iOS, App Events API and Mobile Measurement Partners Updates for Apple's iOS 14 Requirements](https://www.facebook.com/business/help/2750680505215705?id=428636648170202)
• [Facebook Pixel Updates for Apple's iOS 14 Requirements](https://www.facebook.com/business/help/721422165168355)
• [January 19, 2021 - Breaking Changes](https://developers.facebook.com/docs/graph-api/changelog/non-versioned-changes/jan-19-2021)

The `agency_client_declaration` field requires [Admin privileges](https://www.facebook.com/business/help/442345745885606?id=180505742745347) for all operations starting with v10.0 and will be required for all versions on May 25, 2021.

## Ad Volume

You can view the volume of ads running or in review for your ad accounts. These ads will count against the ads limit per page that we will enact in early 2021. Query the number of ads running or in review for a given ad account.

Ad Limit Per Page enforcement begins for when a Page reaches its ad limit enforcement date. Enforcement date can be queried [here](https://developers.facebook.com/docs/marketing-api/insights-api/ads-volume).

When a Page is at its ad limit:
• New ads (or ads scheduled to begin at that time) do not publish successfully.
• Actions on existing ads are limited to pausing and archiving until the number of ads running or in review is below the ad limit.

To see the ads volume for your ad account:

```bash
curl -G \
  -d "access_token=<access_token>" \
  "https://graph.facebook.com/<API_VERSION>/act_<ad_account_ID>/ads_volume"
```

The response looks like this:

```json
{"data":[{"ads_running_or_in_review_count":2}]}
```

For information on managing ads volume, see [About Managing Ad Volume](https://www.facebook.com/business/help/2720085414702598).

### Running Or In Review

To see if an ad is running or in review, we check `effective_status`, `configured_status`, and the ad account's status:

• If an ad has `effective_status` of `1` - `active`, we consider it a running or in review.
• If an ad has `configured_status` of `active` and `effective_status` of `9` - `pending review`, or `17` - `pending processing` we consider it a running or in review.
• The ad can be running or in review only if the ad account status is in `1` - `active`, `8` - `pending settlement`, `9` - `in grace period`.

We also determine if an ad is running or in review based on the ad set's schedule.

• If start time is before current time, and current time is before end time, then we consider the ad running or in review.
• If start time is before current time and the ad set has no end time, we also consider it running or in review.

For example, if the ad set is scheduled to run in the future, the ads are not running or in review. However if the ad set is scheduled to run from now until three months from now, we consider the ads running or in review.

If you are using special ads scheduling features, such as day-parting, we consider the ad running or in review the whole day, not just for the part of the day when the ad starts running.

### Breakdown By Actors

We've added the `show_breakdown_by_actor` parameter to the `act_123/ads_volume` endpoint so you can query ad volume and ad limits-related information for each page. For more details, see [Breakdown by Actors](https://developers.facebook.com/docs/marketing-api/insights-api/ads-volume#breakdown-by-actors).

### Example, Querying

For example, query for all ad sets in this ad account:

### Limits

| Limit Type | Description | Value |
|------------|-------------|--------|
| Daily Budget | Minimum daily budget | Varies by currency |
| Spend Cap | Maximum spending limit | Configurable |
| Ad Volume | Maximum running ads | Page-specific |

## Reading

An ad account is an account used for managing ads on Facebook

Finding people with access to this account:

Get list of accepted Terms of Service, where id is the Facebook terms of service content id:

### Digital Services Act Saved Beneficiary/Payor Information

Use the following code examples to download the beneficiary and payor information.

#### Android SDK

```java
GraphRequest request = GraphRequest.newGraphPathRequest(
 accessToken,
 "/act_<AD_ACCOUNT_ID>",
 new GraphRequest.Callback() {
   @Override
   public void onCompleted(GraphResponse response) {
     // Insert your code here
   }
});

Bundle parameters = new Bundle();
parameters.putString("fields", "default_dsa_payor,default_dsa_beneficiary");
request.setParameters(parameters);
request.executeAsync();
```

#### iOS SDK

```objective-c
FBSDKGraphRequest *request = [[FBSDKGraphRequest alloc]
    initWithGraphPath:@"/act_<AD_ACCOUNT_ID>"
           parameters:@{ @"fields": @"default_dsa_payor,default_dsa_beneficiary",}
           HTTPMethod:@"GET"];
[request startWithCompletionHandler:^(FBSDKGraphRequestConnection *connection, id result, NSError *error) {
    // Insert your code here
}];
```

#### Javascript SDK

```javascript
FB.api(
  '/act_<AD_ACCOUNT_ID>',
  'GET',
  {"fields":"default_dsa_payor,default_dsa_beneficiary"},
  function(response) {
      // Insert your code here
  }
);
```

#### cURL

```bash
curl -X GET \
"https://graph.facebook.com/v23.0/act_<AD_ACCOUNT_ID>?fields=default_dsa_payor%2Cdefault_dsa_beneficiary&access_token=<ACCESS_TOKEN>"
```

The return value is in JSON format. For example:

```json
{"default_dsa_payor":"payor2","default_dsa_beneficiary":"bene2","id":"act_426197654150180"}
```

### Parameters

This endpoint does not contain parameters.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| id | string | The string act_{ad_account_id}. Default |
| account_id | numeric string | The ID of the Ad Account. Default |
| account_status | unsigned int32 | Status of the account: 1 = ACTIVE, 2 = DISABLED, 3 = UNSETTLED, 7 = PENDING_RISK_REVIEW, 8 = PENDING_SETTLEMENT, 9 = IN_GRACE_PERIOD, 100 = PENDING_CLOSURE, 101 = CLOSED, 201 = ANY_ACTIVE, 202 = ANY_CLOSED |
| ad_account_promotable_objects | AdAccountPromotableObjects | Ad Account creation request purchase order fields associated with this Ad Account. |
| age | float | Amount of time the ad account has been open, in days. |
| agency_client_declaration | AgencyClientDeclaration | Details of the agency advertising on behalf of this client account, if applicable. Requires Business Manager Admin privileges. |
| amount_spent | numeric string | Current amount spent by the account with respect to spend_cap. Or total amount in the absence of spend_cap. See why amount spent is different in ad account spending limit for more info. |
| attribution_spec | list<AttributionSpec> | Deprecated due to iOS 14 changes. Please visit the changelog for more information. |
| balance | numeric string | Bill amount due for this Ad Account. |
| brand_safety_content_filter_levels | list<string> | Brand safety content filter levels set for in-content ads (Facebook in-stream videos and Ads on Facebook Reels) and Audience Network along with feed ads (Facebook Feed, Instagram feed, Facebook Reels feed and Instagram Reels feed) if applicable. Refer to Placement Targeting for a list of supported values. |
| business | Business | The Business Manager, if this ad account is owned by one |
| business_city | string | City for business address |
| business_country_code | string | Country code for the business address |
| business_name | string | The business name for the account |
| business_state | string | State abbreviation for business address |
| business_street | string | First line of the business street address for the account |
| business_street2 | string | Second line of the business street address for the account |
| business_zip | string | Zip code for business address |
| can_create_brand_lift_study | bool | If we can create a new automated brand lift study under the Ad Account. |
| capabilities | list<string> | List of capabilities an Ad Account can have. See capabilities |
| created_time | datetime | The time the account was created in ISO 8601 format. |
| currency | string | The currency used for the account, based on the corresponding value in the account settings. See supported currencies |
| default_dsa_beneficiary | string | This is the default value for creating L2 object of dsa_beneficiary |
| default_dsa_payor | string | This is the default value for creating L2 object of dsa_payor |
| direct_deals_tos_accepted | bool | Whether DirectDeals ToS are accepted. |
| disable_reason | unsigned int32 | The reason why the account was disabled. Possible reasons are: 0 = NONE, 1 = ADS_INTEGRITY_POLICY, 2 = ADS_IP_REVIEW, 3 = RISK_PAYMENT, 4 = GRAY_ACCOUNT_SHUT_DOWN, 5 = ADS_AFC_REVIEW, 6 = BUSINESS_INTEGRITY_RAR, 7 = PERMANENT_CLOSE, 8 = UNUSED_RESELLER_ACCOUNT, 9 = UNUSED_ACCOUNT, 10 = UMBRELLA_AD_ACCOUNT, 11 = BUSINESS_MANAGER_INTEGRITY_POLICY, 12 = MISREPRESENTED_AD_ACCOUNT, 13 = AOAB_DESHARE_LEGAL_ENTITY, 14 = CTX_THREAD_REVIEW, 15 = COMPROMISED_AD_ACCOUNT |
| end_advertiser | numeric string | The entity the ads will target. Must be a Facebook Page Alias, Facebook Page ID or an Facebook App ID. |
| end_advertiser_name | string | The name of the entity the ads will target. |
| existing_customers | list<string> | The custom audience ids that are used by advertisers to define their existing customers. This definition is primarily used by Automated Shopping Ads. |
| expired_funding_source_details | FundingSourceDetails | Details of expired funding source including ID, amount, currency, display amount, expiration, start date, display string, campaign IDs, original amount, original display amount, type (0=UNSET, 1=CREDIT_CARD, 2=FACEBOOK_WALLET, 3=FACEBOOK_PAID_CREDIT, 4=FACEBOOK_EXTENDED_CREDIT, 5=ORDER, 6=INVOICE, 7=FACEBOOK_TOKEN, 8=FACEBOOK_COUPON) |
| extended_credit_invoice_group | ExtendedCreditInvoiceGroup | The extended credit invoice group that the ad account belongs to |
| failed_delivery_checks | list<DeliveryCheck> | Failed delivery checks |
| fb_entity | unsigned int32 | fb_entity |
| funding_source | numeric string | ID of the payment method. If the account does not have a payment method it will still be possible to create ads but these ads will get no delivery. Not available if the account is disabled |
| funding_source_details | FundingSourceDetails | Details of current funding source |
| has_migrated_permissions | bool | Whether this account has migrated permissions |
| has_page_authorized_adaccount | bool | Indicates whether a Facebook page has authorized this ad account to place ads with political content. If you try to place an ad with political content using this ad account for this page, and this page has not authorized this ad account for ads with political content, your ad will be disapproved. See Breaking Changes, Marketing API, Ads with Political Content and Facebook Advertising Policies |
| io_number | numeric string | The Insertion Order (IO) number. |
| is_attribution_spec_system_default | bool | If the attribution specification of ad account is generated from system default values |
| is_direct_deals_enabled | bool | Whether the account is enabled to run Direct Deals |
| is_in_3ds_authorization_enabled_market | bool | If the account is in a market requiring to go through payment process going through 3DS authorization |
| is_notifications_enabled | bool | Get the notifications status of the user for this ad account. This will return true or false depending if notifications are enabled or not |
| is_personal | unsigned int32 | Indicates if this ad account is being used for private, non-business purposes. This affects how value-added tax (VAT) is assessed. Note: This is not related to whether an ad account is attached to a business. |
| is_prepay_account | bool | If this ad account is a prepay. Other option would be a postpay account. To access this field, the user making the API call must have a ADVERTISE or MANAGE task permission for that specific ad account. See Ad Account, Assigned Users for more information. |
| is_tax_id_required | bool | If tax id for this ad account is required or not. To access this field, the user making the API call must have a ADVERTISE or MANAGE task permission for that specific ad account. See Ad Account, Assigned Users for more information. |
| line_numbers | list<integer> | The line numbers |
| media_agency | numeric string | The agency, this could be your own business. Must be a Facebook Page Alias, Facebook Page ID or an Facebook App ID. In absence of one, you can use NONE or UNFOUND. |
| min_campaign_group_spend_cap | numeric string | The minimum required spend cap of Ad Campaign. |
| min_daily_budget | unsigned int32 | The minimum daily budget for this Ad Account |
| name | string | Name of the account. If not set, the name of the first admin visible to the user will be returned. |
| offsite_pixels_tos_accepted | bool | Indicates whether the offsite pixel Terms Of Service contract was signed. This feature can be accessible before v2.9 |
| owner | numeric string | The ID of the account owner |
| partner | numeric string | This could be Facebook Marketing Partner, if there is one. Must be a Facebook Page Alias, Facebook Page ID or an Facebook App ID. In absence of one, you can use NONE or UNFOUND. |
| rf_spec | ReachFrequencySpec | Reach and Frequency limits configuration. See Reach and Frequency |
| show_checkout_experience | bool | Whether or not to show the pre-paid checkout experience to an advertiser. If true, the advertiser is eligible for checkout, or they are already locked in to checkout and haven't graduated to postpay. |
| spend_cap | numeric string | The maximum amount that can be spent by this Ad Account. When the amount is reached, all delivery stops. A value of 0 means no spending-cap. Setting a new spend cap only applies to spend AFTER the time at which you set it. Value specified in basic unit of the currency, for example 'cents' for USD. |
| tax_id | string | Tax ID |
| tax_id_status | unsigned int32 | VAT status code for the account. 0: Unknown, 1: VAT not required- US/CA, 2: VAT information required, 3: VAT information submitted, 4: Offline VAT validation failed, 5: Account is a personal account |
| tax_id_type | string | Type of Tax ID |
| timezone_id | unsigned int32 | The timezone ID of this ad account |
| timezone_name | string | Name for the time zone |
| timezone_offset_hours_utc | float | Time zone difference from UTC (Coordinated Universal Time). |
| tos_accepted | map<string, int32> | Checks if this specific ad account has signed the Terms of Service contracts. Returns 1, if terms were accepted. |
| user_tasks | list<string> | user_tasks |
| user_tos_accepted | map<string, int32> | Checks if a user has signed the Terms of Service contracts related to the Business that contains a specific ad account. Must include user's access token to get information. This verification is not valid for system users. |

### Context Edges

| Edge | Description |
|------|-------------|
| account_controls | Account Controls is for Advantage+ shopping campaigns where advertisers can set audience controls for minimum age and excluded geo location. |
| activities | The activities of this ad account |
| adcreatives | The ad creatives of this ad account |
| ads_reporting_mmm_reports | Marketing mix modeling (MMM) reports generated for this ad account. |
| ads_reporting_mmm_schedulers | Get all MMM report schedulers by this ad account |
| advertisable_applications | All advertisable apps associated with this account |
| applications | Applications connected to the ad accounts |
| asyncadcreatives | The async ad creative creation requests associated with this ad account. |
| broadtargetingcategories | Broad targeting categories (BCTs) can be used for targeting |
| customaudiencestos | The custom audiences term of services available to the ad account |
| customconversions | The custom conversions owned by/shared with this ad account |
| delivery_estimate | The delivery estimate for a given ad set configuration for this ad account |
| deprecatedtargetingadsets | Ad sets with deprecating targeting options for this ad account |
| dsa_recommendations | dsa_recommendations |
| generatepreviews | Generate previews for a creative specification |
| impacting_ad_studies | The ad studies that contain this ad account or any of its descendant ad objects |
| instagram_accounts | Instagram accounts connected to the ad accounts |
| mcmeconversions | mcmeconversions |
| minimum_budgets | Returns minimum daily budget values by currency |
| promote_pages | All pages that have been promoted under the ad account |
| reachestimate | The reach estimate of a given targeting spec for this ad account |
| saved_audiences | Saved audiences in the account |
| targetingbrowse | Unified browse |
| targetingsearch | Unified search |

### Error Codes

| Error Code | Description |
|------------|-------------|
| 200 | Permissions error |
| 613 | Calls to this api have exceeded the rate limit. |
| 100 | Invalid parameter |
| 80004 | There have been too many calls to this ad-account. Wait a bit and try again. For more info, please refer to https://developers.facebook.com/docs/graph-api/overview/rate-limiting#ads-management. |
| 190 | Invalid OAuth 2.0 Access Token |
| 3018 | The start date of the time range cannot be beyond 37 months from the current date |
| 1150 | An unknown error occurred. |
| 2500 | Error parsing graph query |
| 2635 | You are calling a deprecated version of the Ads API. Please update to the latest version. |

## Creating

To create a new ad account for your business you must specify `name`, `currency`, `timezone_id`, `end_advertiser`, `media_agency`, and `partner`. Provide `end_advertiser`, `media_agency`, and `partner`:

• They must be Facebook Page Aliases, Facebook Page ID or an Facebook app ID. For example, to provide your company as an end advertiser you specify my company or `20531316728`.
• The End Advertiser ID is the Facebook primary Page ID or Facebook app ID. Further reference to this field (for formatting and acceptable values) may be found [here](https://developers.facebook.com/docs/marketing-api/reference/business/adaccount/).
• If your ad account has no End Advertiser, Media Agency, or Partner, specify `NONE`.
• If your ad account has an End Advertiser, Media Agency, or Partner, that are not represented on Facebook by Page or app, specify `UNFOUND`.

Once you set `end_advertiser` to a value other than `NONE` or `UNFOUND` you cannot change it.

Create an ad account:

```bash
curl \
-F "name=MyAdAccount" \
-F "currency=USD" \
-F "timezone_id=1" \
-F "end_advertiser=<END_ADVERTISER_ID>" \
-F "media_agency=<MEDIA_AGENCY_ID>" \
-F "partner=NONE" \
-F "access_token=<ACCESS_TOKEN>" \
"https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID>/adaccount"
```

If you have an extended credit line with Facebook, you can set `invoice` to `true` and we associate your new ad account to this credit line.

The response:

```json
{
  "id": "act_<ADACCOUNT_ID>",
  "account_id": "<ADACCOUNT_ID>",
  "business_id": "<BUSINESS_ID>",
  "end_advertiser_id": "<END_ADVERTISER_ID>",
  "media_agency_id": "<MEDIA_AGENCY_ID>",
  "partner_id": "NONE"
}
```

### Example

```http
POST /v23.0/act_<AD_ACCOUNT_ID>/product_audiences HTTP/1.1
Host: graph.facebook.com

name=Test+Iphone+Product+Audience&product_set_id=%3CPRODUCT_SET_ID%3E&inclusions=%5B%7B%22retention_seconds%22%3A86400%2C%22rule%22%3A%7B%22and%22%3A%5B%7B%22event%22%3A%7B%22eq%22%3A%22AddToCart%22%7D%7D%2C%7B%22userAgent%22%3A%7B%22i_contains%22%3A%22iPhone%22%7D%7D%5D%7D%7D%5D&exclusions=%5B%7B%22retention_seconds%22%3A172800%2C%22rule%22%3A%7B%22event%22%3A%7B%22eq%22%3A%22Purchase%22
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | Name of the ad account |
| currency | string | Currency for the account |
| timezone_id | int | Timezone ID |
| end_advertiser | string | End advertiser ID |
| media_agency | string | Media agency ID |
| partner | string | Partner ID |

### Return Type

This endpoint supports [read-after-write](https://developers.facebook.com/docs/graph-api/advanced/#read-after-write) and will read the node represented by `id` in the return type.

```json
{
  "id": "numeric string",
  "message": "string"
}
```

### Error Codes

| Error Code | Description |
|------------|-------------|
| 100 | Invalid parameter |
| 2654 | Failed to create custom audience |
| 3979 | You have exceeded the number of allowed ad accounts for your Business Manager at this time. |
| 3994 | Personal accounts that do not have any history of activity are not eligible for migration to a business manager. Instead create an ad account inside your business manager. |
| 3936 | You've already tried to claim this ad account. You'll see a notification if your request is accepted. |
| 368 | The action attempted has been deemed abusive or is otherwise disallowed |
| 3980 | One or more of the ad accounts in your Business Manager are currently in bad standing or in review. All of your accounts must be in good standing in order to create new ad accounts. |
| 3944 | Your Business Manager already has access to this object. |
| 200 | Permissions error |
| 415 | Two factor authentication required. User have to enter a code from SMS or TOTP code generator to pass 2fac. This could happen when accessing a 2fac-protected asset like a page that is owned by a 2fac-protected business manager. |
| 102 | Session key invalid or no longer valid |
| 457 | The session has an invalid origin |
| 3902 | There was a technical issue and your new ad account wasn't created. Please try again. |
| 190 | Invalid OAuth 2.0 Access Token |
| 104 | Incorrect signature |
| 23007 | This credit card can't be set as your account's primary payment method, because your account is set up to be billed after your ads have delivered. This setup can't be changed. Please try a different card or payment method. |
| 1180 | The payment credential is not valid |

## Updating

Notice:

• The `default_dsa_payor` and `default_dsa_beneficiary` values can be set to both of them or none of them. The API does not allow only one of them to exist in the data storage.
• To unset the values: pass two empty strings at the same time, the values will be unset in the data storage. It does not allow you to unset only one of them.

To update an AdAccount, make a POST request to /act_{ad_account_id}.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| default_dsa_payor | string | Default DSA payor |
| default_dsa_beneficiary | string | Default DSA beneficiary |
| name | string | Account name |
| spend_cap | int | Spending cap |

### Return Type

This endpoint supports [read-after-write](https://developers.facebook.com/docs/graph-api/advanced/#read-after-write) and will read the node you posted to.

```json
{
  "success": "bool"
}
```

### Error Codes

| Error Code | Description |
|------------|-------------|
| 100 | Invalid parameter |
| 200 | Permissions error |
| 190 | Invalid OAuth 2.0 Access Token |
| 368 | The action attempted has been deemed abusive or is otherwise disallowed |
| 80004 | There have been too many calls to this ad-account. Wait a bit and try again. For more info, please refer to https://developers.facebook.com/docs/graph-api/overview/rate-limiting#ads-management. |
| 2620 | Invalid call to update account permissions |

## Deleting

You can unlink an AdAccount from a [CustomAudience](https://developers.facebook.com/docs/marketing-api/reference/custom-audience/) by making a DELETE request to [/{custom_audience_id}/ad_accounts](https://developers.facebook.com/docs/marketing-api/reference/custom-audience/ad_accounts/).

You can unlink an AdAccount from an [AdsPixel](https://developers.facebook.com/docs/marketing-api/reference/ads-pixel/) by making a DELETE request to [/{ads_pixel_id}/shared_accounts](https://developers.facebook.com/docs/marketing-api/reference/ads-pixel/shared_accounts/).

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| business | string | Business ID |

### Return Type

```json
{
  "success": "bool"
}
```

### Error Codes

| Error Code | Description |
|------------|-------------|
| 100 | Invalid parameter |

---

**API Version:** v23.0

**Documentation URL:** https://developers.facebook.com/docs/marketing-api/reference/ad-account/

**Related Documentation:**
- [Business Manager API](https://developers.facebook.com/docs/marketing-api/business-manager-api)
- [Marketing API Overview](https://developers.facebook.com/docs/marketing-api/overview)
- [Ads Management](https://developers.facebook.com/docs/marketing-api/get-started)

# Datasets (Pixels) - Complete Documentation

## Overview

A Facebook pixel (also referred to as a dataset) is a small piece of JavaScript code that an advertiser places on every page of their website. This piece of code provides a set of lightweight functionalities for sending user-specific events and event-specific custom data to Facebook. Advertisers can use the Facebook pixel to capture intent information about how people are using their website. A single Facebook pixel is added to all pages of a website, and is then used to create [website custom audiences](https://developers.facebook.com/docs/marketing-api/custom-audience-website/).

## Reading Pixel Data

### Example Request

```http
GET /v23.0/{ads-pixel-id} HTTP/1.1
Host: graph.facebook.com
```

### Parameters

This endpoint does not contain parameters.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| id | numeric string | ID of the pixel. Default |
| automatic_matching_fields | list<enum> | Advanced matching fields which are enabled for automatic advanced matching |
| can_proxy | bool | can_proxy |
| code | string | Pixel code to be placed on the website |
| config | string | The configuration to use for uploads to this dataset. Format determined by the method of upload (eg. UI or SDK) |
| creation_time | datetime | Time at which the pixel was created |
| creator | User | The user who created this pixel |
| data_use_setting | enum | Setting to capture how pixel data should be used |
| description | string | Description of the pixel |
| duplicate_entries | integer | Number of duplicate entries for this dataset |
| enable_auto_assign_to_accounts | bool | Whether the dataset is auto assigned and auto tracked for all accounts that the owner business owns |
| enable_automatic_matching | bool | Represents whether automatic advanced matching is enabled for the pixel for identity matching purposes |
| event_stats | string | Event stats of this dataset |
| event_time_max | integer | Latest entry of this dataset |
| event_time_min | integer | Earliest entry of this dataset |
| first_party_cookie_status | enum | First party cookie status to indicate whether first party cookies can be set for this pixel |
| has_1p_pixel_event | bool | whether pixel has sent us 1p signals |
| is_consolidated_container | bool | A boolean value indicating whether this signal container has unified pixel and offline conversion data set |
| is_created_by_business | bool | Flag stands for if a pixel is created by business |
| is_crm | bool | True if a pixel contains lead gen data source config |
| is_mta_use | bool | Whether the dataset is restricted to MTA only |
| is_restricted_use | bool | Whether the dataset is restricted to Lift only |
| is_unavailable | bool | Whether this pixel is unavailable |
| last_fired_time | datetime | Time at which the pixel was last fired |
| last_upload_app | string | The app that made the most recent upload |
| last_upload_app_changed_time | integer | Time when the app that made the most recent upload last changed |
| match_rate_approx | int32 | Approximate match rate percentage for the entries in this dataset |
| matched_entries | integer | Number of matched entries of this dataset |
| name | string | Name of the pixel |
| owner_business | Business | ID of the business that owns this pixel or null if the pixel has not been claimed by any business yet |
| usage | OfflineConversionDataSetUsage | Usage info for the dataset |
| valid_entries | integer | Number of valid entries of this dataset |

### Context Edges

| Edge | Description |
|------|-------------|
| assigned_users | assigned_users |
| da_checks | A list of results after running Dynamic Ads checks on this pixel |
| offline_event_uploads | The offline uploads associated with this event set |
| openbridge_configurations | Get all the openbridge configurations associated to this Pixel |
| shared_agencies | Agencies or other businesses this pixel is shared with |
| stats | Stats data for this pixel |

### Error Codes

| Error Code | Description |
|------------|-------------|
| 200 | Permissions error |
| 100 | Invalid parameter |
| 368 | The action attempted has been deemed abusive or is otherwise disallowed |
| 190 | Invalid OAuth 2.0 Access Token |
| 80004 | There have been too many calls to this ad-account. Wait a bit and try again. For more info, please refer to https://developers.facebook.com/docs/graph-api/overview/rate-limiting#ads-management. |

## Creating a Pixel

You can make a POST request to the `adspixels` edge from these locations:
• `/act_{ad_account_id}/adspixels`

When posting to this edge, an AdsPixel is created.

### Example

```http
POST /v23.0/act_<AD_ACCOUNT_ID>/adspixels HTTP/1.1
Host: graph.facebook.com

name=My+WCA+Pixel
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | Name of the pixel |

### Return Type

This endpoint supports [read-after-write](https://developers.facebook.com/docs/graph-api/advanced/#read-after-write) and will read the node represented by `id` in the return type.

```json
{
  "id": "numeric string"
}
```

### Error Codes

| Error Code | Description |
|------------|-------------|
| 6202 | More than one pixel exist for this account |
| 6200 | A pixel already exists for this account |
| 100 | Invalid parameter |
| 200 | Permissions error |
| 190 | Invalid OAuth 2.0 Access Token |

## Updating a Pixel

To update an AdsPixel, make a POST request to `/{ads_pixel_id}`.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | Name of the pixel |
| description | string | Description of the pixel |

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
| 190 | Invalid OAuth 2.0 Access Token |
| 100 | Invalid parameter |

## Deleting a Pixel

Cannot perform this operation on this endpoint.

## Dataset Configuration

### Data Use Setting

The `data_use_setting` field captures how pixel data should be used and can have the following values:

- **ADVERTISING_AND_ANALYTICS**: Use data for both advertising and analytics
- **ADVERTISING**: Use data for advertising only
- **ANALYTICS**: Use data for analytics only

### Automatic Matching

Automatic advanced matching can be enabled to improve match rates by automatically detecting and hashing customer information. The `enable_automatic_matching` field controls this setting.

### First Party Cookie Status

The `first_party_cookie_status` field indicates whether first party cookies can be set for this pixel:

- **FIRST_PARTY_COOKIE_ENABLED**: First party cookies are enabled
- **FIRST_PARTY_COOKIE_DISABLED**: First party cookies are disabled

## Dataset Management

### Event Statistics

The `event_stats` field provides statistics about events received by the dataset, including:

- Total events received
- Event breakdown by type
- Time range of events
- Data quality metrics

### Match Rate Monitoring

Monitor the effectiveness of your pixel through:

- `match_rate_approx`: Approximate match rate percentage
- `matched_entries`: Number of successfully matched entries
- `valid_entries`: Total number of valid entries
- `duplicate_entries`: Number of duplicate entries

### Data Quality

Ensure high data quality by monitoring:

- Event time ranges (`event_time_min`, `event_time_max`)
- Last firing time (`last_fired_time`)
- Upload application tracking (`last_upload_app`)

## Offline Event Sets

Datasets can also handle offline events through:

- `offline_event_uploads`: Associated offline upload files
- `is_crm`: Integration with CRM systems
- Custom offline conversion tracking

## Business Integration

### Business Ownership

- `owner_business`: Business that owns the pixel
- `is_created_by_business`: Whether pixel was created by business
- `enable_auto_assign_to_accounts`: Auto-assignment to business accounts

### Sharing and Permissions

- `shared_agencies`: Agencies with access to the pixel
- `assigned_users`: Users with specific pixel permissions
- Business-to-business sharing capabilities

## Advanced Configuration

### OpenBridge Integration

The `openbridge_configurations` edge provides access to OpenBridge configurations for enhanced data connectivity.

### Dynamic Ads Checks

The `da_checks` edge provides results from Dynamic Ads compatibility checks to ensure proper setup for dynamic advertising campaigns.

### Usage Restrictions

Control dataset usage through:

- `is_mta_use`: Restrict to Media Mix Modeling (MTA) only
- `is_restricted_use`: Restrict to Lift studies only
- `data_use_setting`: Overall data usage permissions

## Best Practices

### Pixel Implementation

1. **Single Pixel**: Use one pixel across your entire website
2. **Consistent Naming**: Use descriptive names for easy identification
3. **Regular Monitoring**: Check match rates and data quality regularly
4. **Proper Configuration**: Set appropriate data use settings

### Data Quality

1. **Event Validation**: Monitor duplicate and invalid entries
2. **Match Rate Optimization**: Enable automatic matching when appropriate
3. **Time Accuracy**: Ensure events are fired with correct timestamps
4. **Data Completeness**: Include all relevant customer information

### Security and Compliance

1. **Access Control**: Properly manage assigned users and shared agencies
2. **Data Use Settings**: Configure appropriate data usage restrictions
3. **Privacy Compliance**: Respect user privacy preferences
4. **Business Integration**: Ensure proper business ownership and management

---

**API Version:** v23.0

**Documentation URL:** https://developers.facebook.com/docs/marketing-api/reference/ads-pixel/

**Related Documentation:**
- [Facebook Pixel](https://developers.facebook.com/docs/facebook-pixel)
- [Website Custom Audiences](https://developers.facebook.com/docs/marketing-api/custom-audience-website/)
- [Conversions API](https://developers.facebook.com/docs/marketing-api/conversions-api)

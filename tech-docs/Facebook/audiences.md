# Audiences - Complete Documentation

## Overview

Audience targeting helps you show your ads to the people you care about. There are two general approaches you can take when creating a target audience:

• **Specific** — You set a relatively strict set of parameters, and we create an audience within those parameters. This approach includes Custom Audiences, Lookalike Audiences, and Dynamic Audiences.
• **Broad** — You rely on our delivery system to find the best people to show your ad to. Provide basic restrictions, such as demographics or location, and we deliver ads to people who meet those attributes.

This document provides an overview of the audience targeting options offered by the Marketing API.

## Common Uses

• **[Lookalike Audiences](https://developers.facebook.com/docs/marketing-api/lookalike-audience-targeting)** — Target people most like your established customers.
• **Custom Audiences** — Build your target custom audience with data from [mobile app](https://developers.facebook.com/docs/marketing-api/audiences-api/mobile-apps) and [website](https://developers.facebook.com/docs/marketing-api/audiences-api/websites) behavior, [CRM](https://developers.facebook.com/docs/marketing-api/audiences-api), and [engagement signals](https://developers.facebook.com/docs/marketing-api/audiences-api/engagement). You can also build audiences from [offline conversions](https://developers.facebook.com/docs/marketing-api/audiences-api/offline).
• **[Dynamic Audiences](https://developers.facebook.com/docs/marketing-api/dynamic-product-ads/product-audiences)** — Build an audience from mobile app and website signals.
• **[Targeting Options](https://developers.facebook.com/docs/marketing-api/buying-api/targeting)** — Basic targeting includes demographics and events, location, interests, and behaviors. You can also learn about [advanced targeting](https://developers.facebook.com/docs/marketing-api/targeting-specs).

## Custom Audiences

A Custom Audience is an option that lets you create your own audience from a set of different sources, including customer lists, website or app traffic, and engagement on Facebook.

Some audiences may require that you create [audience rules](https://developers.facebook.com/docs/marketing-api/audiences/overview/audience-rules) to determine whether someone should be added your Custom Audience.

### Permissions

The [ads_management](https://developers.facebook.com/docs/permissions/reference/ads_management/) permission is required to create and update custom audiences.

### Custom Audience Types

#### [Customer File Custom Audiences](https://developers.facebook.com/docs/marketing-api/audiences-api)

Build target Custom Audiences from customer information. This includes email addresses, phone numbers, names, dates of birth, gender, locations, [App User IDs](https://developers.facebook.com/docs/graph-api/reference/user), [Page Scoped User IDs](https://developers.facebook.com/docs/app-events/bots-for-messenger), Apple's Advertising Identifier (IDFA), or [Android Advertising ID](https://developer.android.com/google/play-services/id.html).

#### [Engagement Custom Audiences](https://developers.facebook.com/docs/marketing-api/audiences-api/engagement)

Build audiences based on people who engaged with your content on Facebook or Instagram. Currently supported audience types include Page, Instagram business profile, Lead ad, and Instant Experiences ad.

#### [Mobile App Custom Audiences](https://developers.facebook.com/docs/marketing-api/audiences-api/mobile-apps)

Build audiences based on peoples' actions in your app that meet your criteria. This solution uses logged named events through our [Facebook SDKs](https://developers.facebook.com/docs/app-ads/sdk), [App Events API](https://developers.facebook.com/docs/app-events), or via [Mobile Measurement Partners](https://developers.facebook.com/docs/app-ads/measuring/measurement-partners).

#### [Website Custom Audiences](https://developers.facebook.com/docs/marketing-api/audiences-api/websites)

Create Custom Audiences of users who visited or took specific actions on your website using [Facebook Pixel](https://developers.facebook.com/docs/facebook-pixel), JavaScript [Tag API](https://developers.facebook.com/docs/ads-for-websites/tag-api), and audience rules. Once you create a Custom Audience with website data, reference it in ad targeting as you do with standard [Custom Audiences](https://developers.facebook.com/docs/reference/ads-api/custom-audience-targeting).

#### [Offline Custom Audiences](https://developers.facebook.com/docs/marketing-api/audiences-api/offline)

Group people who visited your store, made calls to your customer service, or took action offline together and target them with Facebook ads. Custom Audiences from Offline Conversions are based on conversion events uploaded to an Offline Event Set.

#### [Dynamic Audiences](https://developers.facebook.com/docs/marketing-api/dynamic-product-ads/product-audiences)

Dynamic Ads enables you to show people ads based on their cross-device purchasing intent. You can collect signals of user intent from mobile apps and websites then use this data to build an audience for targeting prospective customers.

#### [Lookalike Audiences](https://developers.facebook.com/docs/marketing-api/audiences/guides/lookalike-audiences)

Target people most like your already established customers. Lookalike audiences take several sets of people as "seeds" then Facebook builds an audience of similar people. You can use lookalikes for any business objective — targeting people similar your customers for fan acquisition, site registration, off-Facebook purchases, coupon claims, or simply to drive awareness of a brand.

### Sharing a Custom Audience

You may choose to share your Custom Audience with another business, such as a partner managing and running your ads. There are specific requirements and APIs you should follow when doing so. Learn more about [Business-to-Business Functions](https://developers.facebook.com/docs/marketing-api/businessmanager/business-to-business).

### Custom Audiences Deletion

For all advertisers beginning June 8, 2021 and going forward, we will automatically be moving audiences to the "Expiring Audience" stage once they have been inactive for over two years. This means that once an audience meets the threshold of not being used in an active ad set for over two years, it will be automatically flagged as an "Expiring Audience", and the `delete_time` field will be marked with the estimated deletion time (i.e., 90 days from the time of flagging) when the audience is scheduled to be deleted.

You will then be able to either proactively delete the audience or use the audience in an active ad set to prevent deletion. You can see which of your audiences are in the expiring stage at any time by filtering on their `operation_status` or `delete_time` fields.

For [Customer List Custom Audiences](https://developers.facebook.com/docs/marketing-api/audiences/guides/custom-audiences), specifically, you will need to provide us with your instructions before we take any action. Once audiences are moved to the "Expiring Audience" stage and flagged, you will need to provide your instructions by either using the flagged audience in an active ad set, which we will consider an instruction to retain the audience, or by deciding not to use the flagged audience in an active ad set, which we will consider an instruction to delete the audience.

#### Custom Audience

**Endpoint:** `GET /{CUSTOM_AUDIENCE_ID}`

The `operation_status` field of custom audiences now has an `EXPIRING` status option that will be added to audiences that have not been used in an active ad set the last 2 years. When an audience has been marked as expiring, the `delete_time` field tells you when the audience will be deleted.

**Example Response:**

```json
{
  "delete_time": 1620543600, // Unix time in secs for 9 May 2021
  "operation_status": {
    "status": 100,
    "description": "If an audience hasn't been used in an active ad set for over 2 years, it will begin to expire. Expiring audiences that remain unused for 90 days will be deleted."
  }
}
```

#### Ad Account

**Endpoint:** `GET /act_{AD_ACCOUNT_ID}/customaudiences`

You can now filter custom audiences to see those that have an `operation_status` of `EXPIRING` or those that have a `delete_time` indicating the audience will be deleted soon. These new filters will return the audience IDs.

**Example Requests:**

```bash
GET /act_217284683/customaudiences?filtering=[{"field":"delete_time","operator":"GREATER_THAN","value":0}]&fields=["name", "operation_status","delete_time"]

GET /act_217284683/customaudiences?filtering=[{"field":"operation_status.code","operator":"IN","value":[100]}]&fields=["name", "operation_status","delete_time"]
```

#### Saved Audiences

**Endpoint:** `GET /{SAVED_AUDIENCE_ID}`

The `operation_status` field for saved audiences now has an `EXPIRING` status option that will be added to audiences that have not been used in an active ad set the last 2 years. When an audience has been marked as expiring, the `delete_time` field tells you when the audience will be deleted.

**Example Response:**

```json
{
  "delete_time": 1620543600, // Unix time in secs for 9 May 2021
  "operation_status": {
    "status": 100,
    "description": "If an audience hasn't been used in an active ad set for over 2 years, it will begin to expire. Expiring audiences that remain unused for 90 days will be deleted."
  }
}
```

**Endpoint:** `GET /act_{AD_ACCOUNT_ID}/saved_audiences`

You can now filter saved audiences by `delete_time` to see those that will be deleted soon. This filter returns the audience IDs.

**Example Request:**

```bash
GET /act_217284683/saved_audiences?filtering=[{"field":"delete_time","operator":"GREATER_THAN","value":0}]
```

#### FAQ

**Which audiences will be impacted by this change?**
Audiences that have not been used in an active ad set for over 2 years.

**Is there any action I can take to prevent my flagged audiences from being deleted on the planned deletion date?**
Yes, you can use the flagged audience in an active ad set to prevent deletion.

**When does the system automatically delete an audience?**
90 days after the audience is flagged as "Expiring Audience".

## Targeting

Targeting Specs are [ad set](https://developers.facebook.com/docs/reference/ads-api/adset) attributes that define who sees an ad. Basic or core targeting includes:

• **[Demographics and Events](https://developers.facebook.com/docs/marketing-api/buying-api/targeting#demographics)**
• **[Location](https://developers.facebook.com/docs/marketing-api/buying-api/targeting#location)**
• **[Interests](https://developers.facebook.com/docs/marketing-api/buying-api/targeting#interests)**
• **[Behaviors](https://developers.facebook.com/docs/marketing-api/buying-api/targeting#behaviors)**

Typically, you get data to define targeting from a [Targeting Search](https://developers.facebook.com/docs/reference/ads-api/get-autocomplete-data), then specify options in a [Targeting Spec](https://developers.facebook.com/docs/reference/ads-api/targeting-specs). You must specify at least one country in targeting, unless you use [Custom Audiences](https://developers.facebook.com/docs/marketing-api/custom-audience-targeting).

Advertisers running housing, employment and credit ads, who are based in the United States or running ads targeted to the United States have different sets of restrictions. See [Special Ad Category](https://developers.facebook.com/docs/marketing-api/special-ad-category/).

## Documentation Contents

### [Overview](https://developers.facebook.com/docs/marketing-api/audiences/overview)

The basics of audiences and targeting

### [Guides](https://developers.facebook.com/docs/marketing-api/audiences/guides)

Build audiences with data and learn more about our broad targeting options

### [Reference](https://developers.facebook.com/docs/marketing-api/audiences/reference)

Explore our basic and advanced targeting options, targeting search, and the Custom Audience Terms of Service contracts

### [Special Ad Category](https://developers.facebook.com/docs/audiences/special-ad-category)

Targeting options available for advertisers offering housing, employment, or credit opportunities

## See Also

• [Help Center: Custom Audiences](https://www.facebook.com/business/help/744354708981227?id=2469097953376494)
• [Help Center: Lookalike Audiences](https://www.facebook.com/business/help/164749007013531?id=401668390442328)

---

**API Version:** v23.0

**Documentation URL:** https://developers.facebook.com/docs/marketing-api/audiences/

**Related Documentation:**
- [Marketing API Overview](https://developers.facebook.com/docs/marketing-api/overview)
- [Custom Audience Targeting](https://developers.facebook.com/docs/marketing-api/custom-audience-targeting)
- [Targeting Specs](https://developers.facebook.com/docs/marketing-api/targeting-specs)

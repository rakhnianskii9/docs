# Pages - Complete Documentation

## Overview

Represents a [Facebook Page](https://www.facebook.com/help/282489752085908/). Over the coming months, all classic Pages will be migrated to the New Pages Experience. Use the `has_transitioned_to_new_page_experience` Page field to determine if a Page has been migrated. After all Pages have been migrated, the classic Pages experience will no longer be available.

Refer to the [Pages API Guides](https://developers.facebook.com/docs/pages) for additional usage information.

## Reading Page Data

Get information about a Facebook Page.

### Example Request

```bash
curl -i -X GET "https://graph.facebook.com/PAGE-ID?access_token=ACCESS-TOKEN"
```

### Requirements

#### Permissions

- For apps that have been granted the `pages_read_engagement` and `pages_read_user_content` permissions, only data owned by the Page is accessible.
- For apps that have been approved for either the Page Public Content Access (PPCA) or Page Public Metadata Access (PPMA) feature, only public data is accessible. [Learn more.](https://developers.facebook.com/docs/pages/overview/permissions-features#features)
- The `instagram_business_account` field requires a User access token from a User who is able to perform appropriate [tasks](https://developers.facebook.com/docs/instagram-api/overview#tasks) on the Page.
- If using a [business system user](https://www.facebook.com/business/help/327596604689624) in your request, the [business_management permission](https://developers.facebook.com/docs/permissions/reference/business_management) may be required.

#### Limitations

- A Page access token is required for any fields that may include User information.
- All users requesting access to a Page using permissions must be able to perform the `MODERATE` task on the Page being queried.
- If you're using the Page public content access feature, use a [system user access token](https://www.facebook.com/business/help/503306463479099) to avoid [rate limiting](https://developers.facebook.com/docs/graph-api/overview/rate-limiting#pages).
- If the page url is being used as the query input, ensure the page url is set up the following way: facebook.com/<pageusername>. More information on the [page username.](https://www.facebook.com/help/121237621291199)

#### Public Page Data

Requirements vary based on the Page's status, unpublished or published, and unrestricted or restricted. Restrictions include any visibility restrictions such as by age or region. Note that for restricted Pages, the app user must also satisfy any restrictions in order for data to be returned.

### Parameters

This endpoint does not contain parameters.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| id | numeric string | The ID representing a Facebook Page |
| about | string | Information about the Page. Can be read with Page Public Content Access or Page Public Metadata Access. This value maps to the Description setting in the Edit Page Info user interface. Limit of 100 characters |
| access_token | string | The Page's access token. Only returned if the User making the request has a role (other than Live Contributor) on the Page. If your business requires two-factor authentication, the User must also be authenticated |
| ad_campaign | AdSet | The Page's currently running an ad campaign |
| affiliation | string | Affiliation of this person. Applicable to Pages representing people. Can be read with Page Public Content Access or Page Public Metadata Access |
| app_id | id | App ID for app-owned Pages and app Pages |
| artists_we_like | string | Artists the band likes. Applicable to Bands. Can be read with Page Public Content Access or Page Public Metadata Access |
| attire | string | Dress code of the business. Applicable to Restaurants or Nightlife. Can be one of Casual, Dressy or Unspecified. Can be read with Page Public Content Access or Page Public Metadata Access |
| available_promo_offer_ids | list<KeyValue:enum,list<KeyValue:string,string>> | available_promo_offer_ids |
| awards | string | The awards information of the film. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| band_interests | string | Band interests. Applicable to Bands. Can be read with Page Public Content Access or Page Public Metadata Access |
| band_members | string | Members of the band. Applicable to Bands. Can be read with Page Public Content Access or Page Public Metadata Access |
| best_page | Page | The best available Page on Facebook for the concept represented by this Page. The best available Page takes into account authenticity and the number of likes |
| bio | string | Biography of the band. Applicable to Bands. Can be read with Page Public Content Access or Page Public Metadata Access. Limit of 100 characters |
| birthday | string | Birthday of this person. Applicable to Pages representing people. Can be read with Page Public Content Access or Page Public Metadata Access |
| booking_agent | string | Booking agent of the band. Applicable to Bands. Can be read with Page Public Content Access or Page Public Metadata Access |
| breaking_news_usage | null | Information about the availability of daily and monthly usages of the breaking news indicator (Deprecated) |
| built | string | Year vehicle was built. Applicable to Vehicles. Can be read with Page Public Content Access or Page Public Metadata Access |
| business | Business | The Business associated with this Page. Requires business_management permissions, and a page or user access token. The person requesting the access token must be an admin of the page |
| can_checkin | bool | Whether the Page has checkin functionality enabled. Can be read with Page Public Content Access or Page Public Metadata Access |
| can_post | bool | Indicates whether the current app user can post on this Page. Can be read with Page Public Content Access or Page Public Metadata Access |
| category | string | The Page's category. e.g. Product/Service, Computers/Technology. Can be read with Page Public Content Access or Page Public Metadata Access. Default |
| category_list | list<PageCategory> | The Page's sub-categories. This field will not return the parent category |
| checkins | unsigned int32 | Number of checkins at a place represented by a Page. Default |
| company_overview | string | The company overview. Applicable to Companies. Can be read with Page Public Content Access or Page Public Metadata Access |
| connected_instagram_account | IGUser | Instagram account connected to page via page settings |
| contact_address | MailingAddress | The mailing or contact address for this page. This field will be blank if the contact address is the same as the physical address |
| copyright_attribution_insights | CopyrightAttributionInsights | Insight metrics that measures performance of copyright attribution. An example metric would be number of incremental followers from attribution |
| copyright_whitelisted_ig_partners | list<string> | Instagram usernames who will not be reported in copyright match systems |
| country_page_likes | unsigned int32 | If this is a Page in a Global Pages hierarchy, the number of people who are being directed to this Page. Can be read with Page Public Content Access or Page Public Metadata Access |
| cover | CoverPhoto | Information about the page's cover photo |
| culinary_team | string | Culinary team of the business. Applicable to Restaurants or Nightlife. Can be read with Page Public Content Access or Page Public Metadata Access |
| current_location | string | Current location of the Page. Can be read with Page Public Content Access or Page Public Metadata Access. To manage a child Page's location use the /{page-id}/locations endpoint |
| delivery_and_pickup_option_info | list<string> | A Vector of url strings for delivery_and_pickup_option_info of the Page |
| description | string | The description of the Page. Can be read with Page Public Content Access or Page Public Metadata Access. Note that this value is mapped to the Additional Information setting in the Edit Page Info user interface. Default |
| description_html | string | The description of the Page in raw HTML. Can be read with Page Public Content Access or Page Public Metadata Access |
| differently_open_offerings | list<KeyValue:enum,bool> | To be used when temporary_status is set to differently_open to indicate how the business is operating differently than usual, such as a restaurant offering takeout. Enum keys can be one or more of the following: ONLINE_SERVICES, DELIVERY, PICKUP, OTHER with the value set to true or false |
| directed_by | string | The director of the film. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| display_subtext | string | Subtext about the Page being viewed. Can be read with Page Public Content Access or Page Public Metadata Access |
| displayed_message_response_time | string | Page estimated message response time displayed to user. Can be read with Page Public Content Access or Page Public Metadata Access |
| does_viewer_have_page_permission_link_ig | bool | does_viewer_have_page_permission_link_ig |
| emails | list<string> | The emails listed in the About section of a Page. Can be read with Page Public Content Access or Page Public Metadata Access |
| engagement | Engagement | The social sentence and like count information for this Page. This is the same info used for the like button |
| fan_count | unsigned int32 | The number of users who like the Page. For Global Pages this is the count for all Pages across the brand. Can be read with Page Public Content Access or Page Public Metadata Access. For New Page Experience Pages, this field will return followers_count |
| features | string | Features of the vehicle. Applicable to Vehicles. Can be read with Page Public Content Access or Page Public Metadata Access |
| followers_count | unsigned int32 | Number of page followers |
| food_styles | list<string> | The restaurant's food styles. Applicable to Restaurants |
| founded | string | When the company was founded. Applicable to Pages in the Company category. Can be read with Page Public Content Access or Page Public Metadata Access |
| general_info | string | General information provided by the Page. Can be read with Page Public Content Access or Page Public Metadata Access |
| general_manager | string | General manager of the business. Applicable to Restaurants or Nightlife. Can be read with Page Public Content Access or Page Public Metadata Access |
| genre | string | The genre of the film. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| global_brand_page_name | string | The name of the Page with country codes appended for Global Pages. Only visible to the Page admin. Can be read with Page Public Content Access or Page Public Metadata Access |
| global_brand_root_id | numeric string | This brand's global Root ID |
| has_added_app | bool | Indicates whether this Page has added the app making the query in a Page tab. Can be read with Page Public Content Access |
| has_lead_access | HasLeadAccess | has_lead_access |
| has_transitioned_to_new_page_experience | bool | indicates whether a page has transitioned to new page experience or not |
| has_whatsapp_business_number | bool | Indicates whether WhatsApp number connected to this page is a WhatsApp business number. Can be read with Page Public Content Access or Page Public Metadata Access |
| has_whatsapp_number | bool | Indicates whether WhatsApp number connected to this page is a WhatsApp number. Can be read with Page Public Content Access or Page Public Metadata Access |
| hometown | string | Hometown of the band. Applicable to Bands |
| hours | map<string, string> | Indicates a single range of opening hours for a day. Each day can have 2 different hours ranges. The keys in the map are in the form of {day}_{number}_{status}. {day} should be the first 3 characters of the day of the week, {number} should be either 1 or 2 to allow for the two different hours ranges per day. {status} should be either open or close to delineate the start or end of a time range |
| impressum | string | Legal information about the Page publishers. Can be read with Page Public Content Access or Page Public Metadata Access |
| influences | string | Influences on the band. Applicable to Bands. Can be read with Page Public Content Access or Page Public Metadata Access |
| instagram_business_account | IGUser | Instagram account linked to page during Instagram business conversion flow |
| is_always_open | bool | Indicates whether this location is always open. Can be read with Page Public Content Access or Page Public Metadata Access |
| is_calling_eligible | bool | is_calling_eligible |
| is_chain | bool | Indicates whether location is part of a chain. Can be read with Page Public Content Access or Page Public Metadata Access |
| is_community_page | bool | Indicates whether the Page is a community Page. Can be read with Page Public Content Access or Page Public Metadata Access |
| is_eligible_for_branded_content | bool | Indicates whether the page is eligible for the branded content tool |
| is_eligible_for_disable_connect_ig_btn_for_non_page_admin_am_web | bool | is_eligible_for_disable_connect_ig_btn_for_non_page_admin_am_web |
| is_messenger_bot_get_started_enabled | bool | Indicates whether the page is a Messenger Platform Bot with Get Started button enabled |
| is_messenger_platform_bot | bool | Indicates whether the page is a Messenger Platform Bot. Can be read with Page Public Content Access or Page Public Metadata Access |
| is_owned | bool | Indicates whether Page is owned. Can be read with Page Public Content Access or Page Public Metadata Access |
| is_permanently_closed | bool | Whether the business corresponding to this Page is permanently closed. Can be read with Page Public Content Access or Page Public Metadata Access |
| is_published | bool | Indicates whether the Page is published and visible to non-admins |
| is_unclaimed | bool | Indicates whether the Page is unclaimed |
| is_verified | bool | Deprecated, use "verification_status". Pages with a large number of followers can be manually verified by Facebook as having an authentic identity. This field indicates whether the Page is verified by this process. Can be read with Page Public Content Access or Page Public Metadata Access (Deprecated) |
| is_webhooks_subscribed | bool | Indicates whether the application is subscribed for real time updates from this page |
| keywords | null | Deprecated. Returns null (Deprecated) |
| leadgen_tos_acceptance_time | datetime | Indicates the time when the TOS for running LeadGen Ads on the page was accepted |
| leadgen_tos_accepted | bool | Indicates whether a user has accepted the TOS for running LeadGen Ads on the Page |
| leadgen_tos_accepting_user | User | Indicates the user who accepted the TOS for running LeadGen Ads on the page |
| link | string | The Page's Facebook URL. Default |
| location | Location | The location of this place. Applicable to all Places |
| members | string | Members of this org. Applicable to Pages representing Team Orgs. Can be read with Page Public Content Access |
| merchant_id | string | The instant workflow merchant ID associated with the Page. Can be read with Page Public Content Access or Page Public Metadata Access |
| merchant_review_status | enum | Review status of the Page against FB commerce policies, this status decides whether the Page can use component flow |
| messaging_feature_status | MessagingFeatureStatus | messaging_feature_status |
| messenger_ads_default_icebreakers | list<string> | The default ice breakers for a certain page |
| messenger_ads_default_quick_replies | list<string> | The default quick replies for a certain page |
| messenger_ads_quick_replies_type | enum | Indicates what type this page is and we will generate different sets of quick replies based on it. Values include UNKNOWN, PAGE_SHOP, or RETAIL |
| mission | string | The company mission. Applicable to Companies |
| mpg | string | MPG of the vehicle. Applicable to Vehicles. Can be read with Page Public Content Access or Page Public Metadata Access |
| name | string | The name of the Page. Default |
| name_with_location_descriptor | string | The name of the Page with its location and/or global brand descriptor. Only visible to a page admin. Non-page admins will get the same value as name |
| network | string | The TV network for the TV show. Applicable to TV Shows. Can be read with Page Public Content Access or Page Public Metadata Access |
| new_like_count | unsigned int32 | The number of people who have liked the Page, since the last login. Only visible to a Page admin. Can be read with Page Public Content Access or Page Public Metadata Access |
| offer_eligible | bool | Offer eligibility status. Only visible to a page admin |
| overall_star_rating | float | Overall page rating based on rating survey from users on a scale of 1-5. This value is normalized and is not guaranteed to be a strict average of user ratings. If there are 0 or a small number of ratings, this field will not be returned |
| page_token | string | SELF_EXPLANATORY |
| parent_page | Page | Parent Page of this Page. If the Page is part of a Global Root Structure and you have permission to the Global Root, the Global Root Parent Page is returned. If you do not have Global Root permission, the Market Page for your current region is returned as the Parent Page. If your Page is not part of a Global Root Structure, the Parent Page is returned |
| parking | PageParking | Parking information. Applicable to Businesses and Places |
| payment_options | PagePaymentOptions | Payment options accepted by the business. Applicable to Restaurants or Nightlife |
| personal_info | string | Personal information. Applicable to Pages representing People. Can be read with Page Public Content Access |
| personal_interests | string | Personal interests. Applicable to Pages representing People. Can be read with Page Public Content Access or Page Public Metadata Access |
| pharma_safety_info | string | Pharmacy safety information. Applicable to Pharmaceutical companies. Can be read with Page Public Content Access or Page Public Metadata Access |
| phone | string | Phone number provided by a Page. Can be read with Page Public Content Access |
| pickup_options | list<enum> | List of pickup options available at this Page's store location. Values can include CURBSIDE, IN_STORE, and OTHER |
| place_type | enum | For places, the category of the place. Value can be CITY, COUNTRY, EVENT, GEO_ENTITY, PLACE, RESIDENCE, STATE_PROVINCE, or TEXT |
| plot_outline | string | The plot outline of the film. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| preferred_audience | Targeting | Group of tags describing the preferred audience of ads created for the Page |
| press_contact | string | Press contact information of the band. Applicable to Bands |
| price_range | string | Price range of the business, such as a restaurant or salon. Values can be one of $, $$, $$$, $$$$, Not Applicable, or null if no value is set. Can be read with Page Public Content Access or Page Public Metadata Access |
| privacy_info_url | string | Privacy url in page info section |
| produced_by | string | The productor of the film. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| products | string | The products of this company. Applicable to Companies |
| promotion_eligible | bool | Boosted posts eligibility status. Only visible to a page admin |
| promotion_ineligible_reason | string | Reason for which boosted posts are not eligible. Only visible to a page admin |
| public_transit | string | Public transit to the business. Applicable to Restaurants or Nightlife. Can be read with Page Public Content Access or Page Public Metadata Access |
| rating_count | unsigned int32 | Number of ratings for the Page (limited to ratings that are publicly accessible). Can be read with Page Public Content Access or Page Public Metadata Access |
| recipient | numeric string | Messenger page scope id associated with page and a user using account_linking_token |
| record_label | string | Record label of the band. Applicable to Bands. Can be read with Page Public Content Access or Page Public Metadata Access |
| release_date | string | The film's release date. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| restaurant_services | PageRestaurantServices | Services the restaurant provides. Applicable to Restaurants |
| restaurant_specialties | PageRestaurantSpecialties | The restaurant's specialties. Applicable to Restaurants |
| schedule | string | The air schedule of the TV show. Applicable to TV Shows. Can be read with Page Public Content Access or Page Public Metadata Access |
| screenplay_by | string | The screenwriter of the film. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| season | string | The season information of the TV Show. Applicable to TV Shows. Can be read with Page Public Content Access or Page Public Metadata Access |
| single_line_address | string | The Page address, if any, in a simple single line format. Can be read with Page Public Content Access or Page Public Metadata Access |
| starring | string | The cast of the film. Applicable to Films. Can be read with Page Public Content Access or Page Public Metadata Access |
| start_info | PageStartInfo | Information about when the entity represented by the Page was started |
| store_code | string | Unique store code for this location Page. Can be read with Page Public Content Access or Page Public Metadata Access |
| store_location_descriptor | string | Location Page's store location descriptor |
| store_number | unsigned int32 | Unique store number for this location Page |
| studio | string | The studio for the film production. Applicable to Films |
| supports_donate_button_in_live_video | bool | Whether the user can add a Donate Button to their Live Videos |
| talking_about_count | unsigned int32 | The number of people talking about this Page |
| temporary_status | enum | Indicates how the business corresponding to this Page is operating differently than usual. Possible values: differently_open, temporarily_closed, operating_as_usual, no_data. If set to differently_open use with differently_open_offerings to set status |
| unread_message_count | unsigned int32 | Unread message count for the Page. Only visible to a page admin |
| unread_notif_count | unsigned int32 | Number of unread notifications. Only visible to a page admin |
| unseen_message_count | unsigned int32 | Unseen message count for the Page. Only visible to a page admin |
| user_access_expire_time | datetime | user_access_expire_time |
| username | string | The alias of the Page. For example, for www.facebook.com/platform the username is 'platform'. Default |
| verification_status | string | Showing whether this Page is verified. Value can be blue_verified or gray_verified, which represents that Facebook has confirmed that a Page is the authentic presence of the public figure, celebrity, or global brand it represents, or not_verified. This field can be read with the Page Public Metadata Access feature |
| voip_info | VoipInfo | Voip info |
| website | string | The URL of the Page's website. Can be read with Page Public Content Access or Page Public Metadata Access |
| were_here_count | unsigned int32 | The number of visits to this Page's location. If the Page setting Show map, check-ins and star ratings on the Page (under Page Settings > Page Info > Address) is disabled, then this value will also be disabled. Can be read with Page Public Content Access or Page Public Metadata Access |
| whatsapp_number | string | The Page's WhatsApp number. Can be read with Page Public Content Access or Page Public Metadata Access |
| written_by | string | The writer of the TV show. Applicable to TV Shows. Can be read with Page Public Content Access or Page Public Metadata Access |

### Context Edges

| Edge | Description |
|------|-------------|
| ab_tests | ab_tests |
| ads_posts | The ad posts for this Page |
| agencies | Businesses that have agency permissions on the Page |
| albums | Photo albums for this Page. Can be read with Page Public Content Access |
| ar_experience | ar_experience |
| assigned_users | Users assigned to this Page. Can be read with Page Public Content Access |
| audio_media_copyrights | The music copyrights owned by this page (using alacorn) |
| blocked | User or Page Profiles blocked from this Page |
| businessprojects | Business projects |
| call_to_actions | The call-to-action created by this Page. Can be read with Page Public Content Access |
| canvas_elements | The canvas elements associated with this page |
| canvases | The canvas documents associated with this page |
| chat_plugin | customization configuration values of the Page's corresponding Chat Plugin |
| commerce_orders | The commerce orders of this Page |
| conversations | This Page's conversations |
| crosspost_whitelisted_pages | Pages that are allowed to crosspost |
| ctx_optimization_eligibility | ctx_optimization_eligibility |
| custom_labels | custom_labels |
| custom_user_settings | Custom user settings for a page |
| fantasy_games | fantasy_games |
| feed | This Page's feed. Can be read with Page Public Content Access. Default |
| global_brand_children | Children Pages of a Global Pages root Page. Both default and root Page can return children Pages. Can be read with Page Public Content Access |
| image_copyrights | Image copyrights from this page |
| insights | This Page's Insights data |
| instagram_accounts | Linked Instagram accounts for this Page |
| leadgen_forms | A library of lead generation forms created for this page |
| likes | The Pages that this Page has liked. Can be read with Page Public Content Access. For New Page Experience Pages, this field will return followers_count. Default |
| live_videos | Live videos on this Page. Can be read with Page Public Content Access |
| locations | The location Pages that are children of this Page. Can be read with Page Public Content Access. To manage a child Page's location use the /{page-id}/locations endpoint |
| media_fingerprints | Media fingerprints from this page |
| messaging_feature_review | Feature status of the page that has been granted through feature review that show up in the page settings |
| messenger_call_settings | messenger_call_settings |
| messenger_lead_forms | messenger_lead_forms |
| messenger_profile | SELF_EXPLANATORY |
| page_backed_instagram_accounts | Gets the Page Backed Instagram Account (an IGUser) associated with this Page |
| personas | Messenger Platform Bot personas for the Page |
| photos | This Page's Photos. Can be read with Page Public Content Access |
| picture | This Page's profile picture. Default |
| posts | This Page's own Posts, a derivative of the /feed edge. Can be read with Page Public Content Access |
| product_catalogs | Product catalogs owned by this page |
| published_posts | All published posts by this page |
| roles | The Page's Admins |
| scheduled_posts | All posts that are scheduled to a future date by a page |
| secondary_receivers | Secondary Receivers for a page |
| settings | Controllable settings for this page |
| shop_setup_status | Shows the shop setup status |
| store_locations | list of the page's store locations |
| subscribed_apps | Applications that have real time update subscriptions for this Page. Note that we will only return information about the current app |
| tabs | This Page's tabs and the apps in them. Can be read with Page Public Content Access |
| tagged | The Photos, Videos, and Posts in which the Page has been tagged. A derivative of /feeds. Can be read with Page Public Content Access |
| thread_owner | App which owns a thread for Handover Protocol |
| threads | Deprecated. Use conversations instead |
| video_copyright_rules | Video copyright rules from this page |
| video_lists | Video Playlists for this Page |
| videos | Videos for this Page. Can be read with Page Public Content Access |
| visitor_posts | Shows all public Posts published by Page visitors on the Page. Can be read with Page Public Content Access |

### Error Codes

| Error Code | Description |
|------------|-------------|
| 100 | Invalid parameter |
| 80001 | There have been too many calls to this Page account. Wait a bit and try again. For more info, please refer to https://developers.facebook.com/docs/graph-api/overview/rate-limiting. |
| 200 | Permissions error |
| 190 | Invalid OAuth 2.0 Access Token |
| 102 | Session key invalid or no longer valid |
| 80002 | There have been too many calls to this Instagram account. Wait a bit and try again. For more info, please refer to https://developers.facebook.com/docs/graph-api/overview/rate-limiting. |
| 368 | The action attempted has been deemed abusive or is otherwise disallowed |
| 80005 | There have been too many leadgen api calls to this Page account. Wait a bit and try again. For more info, please refer to https://developers.facebook.com/docs/graph-api/overview/rate-limiting. |
| 210 | User not visible |
| 230 | Permissions disallow message to user |

## Creating Page Content

### Feed Posts

Create posts on a Page's feed.

**Example:**

```http
POST /v23.0/{page-id}/feed HTTP/1.1
Host: graph.facebook.com

message=This+is+a+test+value
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| message | string | The main body of the post |
| link | string | The URL of a link to attach to the post |
| published | bool | Whether a post is published |
| scheduled_publish_time | int | Time when the page post should be published |

**Return Type:**

```json
{
  "id": "POST_ID",
  "post_supports_client_mutation_id": true
}
```

### Lead Generation Forms

Create lead generation forms for the page.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | Name of the lead form |
| questions | list | Questions in the lead form |
| privacy_policy_url | string | Privacy policy URL |

### Subscribed Apps

Subscribe apps to page webhooks.

**Example:**

```http
POST /v23.0/{page-id}/subscribed_apps HTTP/1.1
Host: graph.facebook.com

subscribed_fields=leadgen
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| subscribed_fields | list<string> | Fields to subscribe to |

## Updating Page Information

To update a Page, make a POST request to `/{page_id}`.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| about | string | Information about the page |
| description | string | Description of the page |
| general_info | string | General information |
| website | string | Website URL |
| phone | string | Phone number |
| location | object | Location information |
| hours | map | Operating hours |
| price_range | string | Price range ($, $$, $$$, $$$$) |

### Return Type

```json
{
  "success": true
}
```

### Error Codes

| Error Code | Description |
|------------|-------------|
| 100 | Invalid parameter |
| 200 | Permissions error |
| 210 | User not visible |
| 375 | This Page doesn't have a location descriptor. Add one to continue |
| 190 | Invalid OAuth 2.0 Access Token |
| 368 | The action attempted has been deemed abusive or is otherwise disallowed |
| 283 | That action requires the extended permission pages_read_engagement and/or pages_read_user_content and/or pages_manage_ads and/or pages_manage_metadata |
| 320 | Photo edit failure |
| 374 | Invalid store location descriptor update since this Page is not a location Page |
| 160 | Invalid geolocation type |

## Page Management Features

### User Assignment

Manage users assigned to the page with different roles:

- **Admin**: Full control over the page
- **Editor**: Can create posts and respond to comments
- **Moderator**: Can respond to and delete comments, create posts, and send messages
- **Advertiser**: Can create ads and view insights
- **Analyst**: Can view insights only

### Messaging Features

- **Messenger Bot**: Set up automated responses
- **Chat Plugin**: Website chat integration
- **Lead Forms**: Capture leads through Messenger
- **WhatsApp Integration**: Connect WhatsApp Business numbers

### Content Management

- **Posts**: Create and schedule posts
- **Photos/Videos**: Upload and manage media
- **Albums**: Organize photos into albums
- **Live Videos**: Broadcast live content
- **Stories**: Create temporary content

### Business Features

- **Commerce**: Set up shops and catalogs
- **Locations**: Manage multiple business locations
- **Events**: Create and promote events
- **Call-to-Actions**: Add action buttons
- **Reviews**: Manage customer reviews and ratings

### Analytics and Insights

- **Page Insights**: Audience, reach, and engagement metrics
- **Post Performance**: Individual post analytics
- **Audience Demographics**: Age, gender, location data
- **Engagement Tracking**: Likes, comments, shares, reactions

### Verification and Authentication

- **Verification Status**: Blue or gray verification badges
- **Business Verification**: Confirm business authenticity
- **Two-Factor Authentication**: Enhanced security
- **Access Token Management**: Secure API access

## Best Practices

### Content Strategy

1. **Consistent Posting**: Maintain regular posting schedule
2. **Engaging Content**: Create content that encourages interaction
3. **Visual Appeal**: Use high-quality images and videos
4. **Audience Targeting**: Understand your audience demographics

### Security and Compliance

1. **Access Control**: Limit admin access to trusted users
2. **Token Security**: Protect access tokens and rotate regularly
3. **Privacy Settings**: Configure appropriate privacy settings
4. **Content Moderation**: Monitor and moderate user-generated content

### Performance Optimization

1. **Rate Limiting**: Respect API rate limits
2. **Batch Requests**: Use batch requests for efficiency
3. **Caching**: Implement appropriate caching strategies
4. **Error Handling**: Handle API errors gracefully

---

**API Version:** v23.0

**Documentation URL:** https://developers.facebook.com/docs/graph-api/reference/page/

**Related Documentation:**
- [Pages API Overview](https://developers.facebook.com/docs/pages/)
- [Page Access Tokens](https://developers.facebook.com/docs/pages/access-tokens)
- [Page Insights API](https://developers.facebook.com/docs/pages/insights)

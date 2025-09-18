# Conversions API (CAPI) - Complete Documentation

## Overview

Conversions API is designed to create a connection between an advertiser's marketing data (such as website events, app events, business messaging events, and offline conversions) from their server, website platform, mobile app, or CRM to Meta's systems that optimize ad targeting, decrease cost per action, and measure results.

Advertisers can use the Conversions API to send different types of events and simplify their technology stack, rather than having separate endpoints for each data source. In the case of direct integration, a connection is established between the advertiser's server and the Conversions API endpoint on Meta's side.

Server events are associated with a dataset ID and are processed the same as events sent through Meta pixel, Facebook SDK for iOS or Android, Mobile Measurement Partner SDKs, offline event sets, or .csv file uploads. This means they can be used for measurement, reporting, and optimization in the same way as other connection channels. Offline events can be used to measure offline events with attribution, as well as to create and measure custom offline audiences.

To achieve high advertising performance, we recommend advertisers follow our [Conversions API best practices](https://developers.facebook.com/docs/marketing-api/conversions-api/best-practices).

## Getting Started

### Process Overview

The main stages of setting up Conversions API integration:

1. Choose the integration method that's right for you.
2. Complete the necessary prerequisites for this implementation method.
3. Implement using this integration method.
4. Verify setup and follow recommendations to help improve advertising effectiveness.

### Integration Methods

There are several ways to integrate with Conversions API that differ in the level of effort required, cost, and functionality they provide. See [this article](https://www.facebook.com/business/help/433493041367251?id=818859032317965) for an overview of Conversions API setup options.

This developer documentation focuses on providing direct integration.

### Requirements

#### Pixel ID

To use Conversions API, you need to obtain a [Pixel ID](https://www.facebook.com/business/help/952192354843755?id=1205376682832142). If you've already set up a pixel for your website, we recommend using the same Pixel ID for both website events and server events.

#### Business Manager

Using the API also requires [Business Manager](https://business.facebook.com/). With Business Manager, advertisers can integrate Facebook advertising with their internal systems and external partners. If you don't have a Business Manager account yet, create one as described in [this article](https://www.facebook.com/business/help/1710077379203657) in the Help Center.

#### Access Token

To use Conversions API, you need to generate an access token that is passed as a parameter in each API call. You can obtain an access token in two ways:

**Through Events Manager (recommended)**

Step 1. Select the pixel you need to implement.

Step 2. Open the "Settings" tab.

Step 3. Find the Conversions API section and click the "Generate access token" link in the manual setup section, then follow the instructions in the popup window.

Note: The "Generate access token" link is only displayed for users with administrator access rights in the company. Other users won't see it.

After obtaining the access token, click the "Manage integrations" button on the "Overview" tab in Events Manager. In the popup window, click the "Manage" button next to Conversions API. As a result, we will automatically create a Conversions API application and Conversions API system user for you. No app review or permission requests are required.

**Using Your Own Application**

If you already have your own [application](https://developers.facebook.com/docs/apps) and your own [system user](https://developers.facebook.com/docs/marketing-api/system-users/create-retrieve-update), you can generate a token in [Business Manager](https://business.facebook.com/). To do this:

Step 1. Open your business settings.

Step 2. Assign the pixel to the system user (you can also create a new system user at this stage).

Step 3. Select the assigned system user and click "Generate token".

Your app doesn't require review. You also don't need to request any permissions.

Now access tokens created on the Conversions API settings tab in Events Manager can use different versions of the Graph API. Previously, only the newest version available at the time of token creation could be used. [Starting with version 12.0](https://developers.facebook.com/docs/graph-api/changelog/version12.0#conversions-api), new access tokens can use all available Graph API versions.

## Documentation

### [API Parameters](https://developers.facebook.com/docs/marketing-api/conversions-api/parameters)

Required and optional parameters that help improve attribution and ad delivery.

### [Payload Helper](https://developers.facebook.com/docs/marketing-api/conversions-api/payload-helper)

The structure of the payload sent from your server to Facebook.

### [Troubleshooting](https://developers.facebook.com/docs/marketing-api/conversions-api/support)

Handling error codes returned by Conversions API.

## Key Components

### Using the API

Learn how to make POST requests and understand dropped events, batch requests, and transaction time.

### Verifying Setup

Ensure you're receiving appropriate events, performing proper deduplication and matching.

### Parameters

Comprehensive list of required and optional parameters for different event types:

- **Required Parameters**: Event name, event time, user data
- **Optional Parameters**: Custom data, app data, contents, value, currency
- **User Data Parameters**: Email, phone, first name, last name, date of birth, gender, city, state, country, zip code

### Event Types

#### Standard Events
- Purchase
- Add to Cart
- Initiate Checkout
- Add Payment Info
- Complete Registration
- Contact
- Customize Product
- Donate
- Find Location
- Schedule
- Search
- Start Trial
- Submit Application
- Subscribe
- View Content

#### Custom Events
Custom events allow you to track actions specific to your business that aren't covered by standard events.

### App Events Integration

Conversions API for App Events allows you to send app events from your server to improve:
- Attribution accuracy
- Event delivery reliability
- Data collection compliance

### Offline Events Integration

Send offline conversion events to connect your offline business results with your Facebook advertising:
- In-store purchases
- Phone calls
- Customer service interactions
- Other offline conversions

### Business Messaging Integration

Track messaging interactions and conversions:
- Message sends
- Message replies
- Appointment bookings
- Lead conversations

### Dataset Quality API

Monitor and improve the quality of your server events:
- Event match quality scores
- Data validation feedback
- Performance recommendations

### Event Deduplication

Prevent duplicate events when using both Pixel and Conversions API:
- Use `event_id` parameter
- Implement proper deduplication logic
- Monitor deduplication rates

## Best Practices

### Data Quality
- Send complete user information
- Use hashed customer data
- Implement proper event timing
- Include all relevant parameters

### Technical Implementation
- Use HTTPS for all requests
- Implement proper error handling
- Use batch requests for efficiency
- Monitor API response codes

### Event Optimization
- Send events as close to real-time as possible
- Include value and currency for commerce events
- Use consistent event naming conventions
- Implement proper attribution windows

## Error Handling

### Common Error Codes
- **400**: Bad Request - Invalid parameters
- **401**: Unauthorized - Invalid access token
- **429**: Rate Limit - Too many requests
- **500**: Internal Server Error - Temporary issue

### Response Format
```json
{
  "events_received": 1,
  "messages": [],
  "fbtrace_id": "trace_id_here"
}
```

### Debugging
- Use test events for validation
- Monitor Events Manager for event delivery
- Check dataset quality scores
- Review error messages and logs

## Data Processing Options

Learn about Limited Data Use feature and its implementation in Conversions API for compliance with privacy regulations.

## Resources

### Meta Pixel Events
Learn more about [standard](https://developers.facebook.com/docs/facebook-pixel/implementation/conversion-tracking#standard-events) and [custom events](https://developers.facebook.com/docs/facebook-pixel/implementation/conversion-tracking#custom-events) for Meta pixel.

### Business Help Center
Review articles about [Conversions API](https://www.facebook.com/business/help/2041148702652965) and [server event testing](https://www.facebook.com/business/help/1624255387706033) in our Help Center.

### Developer Guide and Webinar
Check out the [developer guide (PDF format)](https://www.facebook.com/gms_hub/share/conversions-api-direct-integration-playbook_english.pdf) and [direct integration developer webinar](https://www.facebook.com/business/m/sessionsforsuccess/conversions-api).

### Additional Resources
- **Business Help Center**: [Business Manager Information](https://www.facebook.com/business/help/113163272211510)
- **Business Help Center**: [Meta Pixel Information](https://www.facebook.com/business/help/742478679120153)
- **Meta Blueprint**: [Getting Started with Conversions API](https://www.facebookblueprint.com/student/path/219713-get-started-with-the-conversions-api)

## IP Addresses

If your company's outgoing requests go through a firewall, obtain Facebook IP addresses as described in the [Crawler IP addresses and user agents](https://developers.facebook.com/docs/sharing/webmasters/crawler#identify) section. Note that the list of addresses changes frequently.

For website, app, or physical store events sent through Conversions API, certain parameters are required. You can see the list in [this article](https://developers.facebook.com/docs/marketing-api/conversions-api/parameters).

---

**API Version:** v23.0

**Documentation URL:** https://developers.facebook.com/docs/marketing-api/conversions-api/

**Related Documentation:**
- [Marketing API Overview](https://developers.facebook.com/docs/marketing-api/overview)
- [Facebook Pixel](https://developers.facebook.com/docs/facebook-pixel)
- [Business Manager API](https://developers.facebook.com/docs/marketing-api/business-manager-api)

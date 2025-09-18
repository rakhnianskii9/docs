# Facebook Permissions Reference - Complete Technical Documentation

## Overview

Permissions are a form of granular, app user-granted Graph API authorization system that controls access to user data and business resources. Before your app can use any API endpoint to access user data, the user must grant your app all permissions required by that endpoint. This comprehensive system ensures user privacy while enabling powerful app functionality.

## Core Principles

### Granular Authorization
- **Specific Data Access**: Each permission grants access to specific types of data
- **User Control**: Users can grant or deny individual permissions
- **Minimal Permissions**: Only request permissions your app actually needs
- **Progressive Requests**: Request additional permissions as needed over time

### User-Centric Control
- **Individual Choice**: Users can grant or deny each permission separately
- **Subset Approval**: Users can approve only some of the requested permissions
- **Revocation Rights**: Users can revoke permissions at any time
- **Transparency**: Clear explanation of what each permission provides access to

### App Review Compliance
- **Legitimate Use**: Only select permissions that your app needs to function
- **Review Requirements**: Unneeded permissions are a common reason for app review rejection
- **Use Case Documentation**: Must demonstrate legitimate use cases for requested permissions

## Access Levels

### Standard Access (No Review Required)
Available immediately without app review:

#### Default Permissions
- **public_profile**: Basic public profile information
- **email**: User's email address

### Advanced Access (Review Required)
Requires Meta App Review for production use:

#### All Other Permissions
- User data beyond basic profile and email
- Business data and management capabilities
- Page management and content access
- Advertising account access
- Instagram business features

## Requirements and Prerequisites

### Meta App Review
Required for apps accessing data not owned or managed by the app:
- **Submission Process**: Submit app with detailed use case documentation
- **Review Timeline**: Variable review time depending on complexity
- **Compliance Verification**: Ensures adherence to platform policies
- **Production Approval**: Required before public app release

### Business Verification
Required for all apps requesting Advanced Access:
- **Business Identity**: Verify legitimate business entity
- **Document Submission**: Provide business registration documents
- **Address Verification**: Confirm business address and contact information
- **Ongoing Compliance**: Maintain verified status throughout app lifecycle

### Data Handling Questions
Required for certain permissions accessing user data:
- **Data Usage Disclosure**: Explain how user data will be used
- **Storage and Security**: Detail data storage and protection measures
- **Third-Party Sharing**: Disclose any data sharing with third parties
- **Retention Policies**: Specify data retention and deletion practices

### Annual Data Use Checkup
Ongoing requirement for maintaining data access:
- **Annual Review**: Yearly verification of data usage practices
- **Compliance Updates**: Update data handling practices as needed
- **Policy Changes**: Adapt to new platform policies and requirements
- **Continuous Monitoring**: Ongoing oversight of app data usage

## Permission Request Methods

### Facebook Login
Standard consumer authentication flow:
- **Consumer Apps**: Individual user authentication
- **Personal Data**: Access to user's personal information
- **Social Features**: Friend connections and social graph access
- **Content Sharing**: Publishing content on behalf of users

### Facebook Login for Business
Business account authentication:
- **Business Users**: Business account holders
- **Professional Data**: Business-related information access
- **Asset Management**: Business asset and resource management
- **Team Collaboration**: Multi-user business account access

### Instagram API Integration
Instagram-specific authentication flows:

#### Instagram API with Facebook Login for Business
- **Business Instagram**: Instagram business account access
- **Cross-Platform**: Integration with Facebook business features
- **Content Management**: Instagram content creation and management
- **Analytics Access**: Instagram business insights and metrics

#### Instagram API with Business Login for Instagram
- **Native Instagram**: Direct Instagram business authentication
- **Platform-Specific**: Instagram-only authentication flow
- **Creator Tools**: Instagram creator and business tools
- **Specialized Features**: Instagram-specific business features

### Meta Business Manager
Enterprise-level business account management:
- **Multi-Account Management**: Manage multiple business accounts
- **Team Permissions**: Granular team member access control
- **Asset Sharing**: Share business assets across organizations
- **Advanced Analytics**: Enterprise-level insights and reporting

## Permission Lifecycle Management

### Permission Expiration
**90-Day Rule**: Automatic permission expiration policy
- **Usage Tracking**: Permissions expire if unused for 90 days
- **User Inactivity**: Typically triggered by user inactivity
- **Re-authorization Required**: Users must re-grant expired permissions
- **Proactive Renewal**: Apps should request renewal before expiration

### Permission Removal
**Dashboard Management**: Remove unnecessary permissions
- **Unused Permissions**: Remove permissions no longer needed
- **Deprecated Permissions**: Remove permissions that have been deprecated
- **Review Optimization**: Reduce permission scope to improve review success
- **User Trust**: Fewer permissions increase user trust and approval rates

### Permission Monitoring
**Usage Analytics**: Track permission usage and effectiveness
- **Permission Utilization**: Monitor which permissions are actively used
- **User Approval Rates**: Track permission grant/deny rates
- **Feature Adoption**: Measure feature usage tied to specific permissions
- **Optimization Opportunities**: Identify permissions that can be removed

## Comprehensive Permissions List

### A - Authentication & Access

#### ads_management
- **Purpose**: Manage advertising campaigns and ad accounts
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Campaign creation, ad optimization, budget management
- **Business Requirements**: Advertising platform integration

#### ads_read
- **Purpose**: Read advertising data and performance metrics
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Analytics platforms, reporting tools, performance monitoring
- **Data Access**: Campaign metrics, ad performance data, audience insights

### B - Business Operations

#### business_management
- **Purpose**: Manage business assets and operations
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Business asset management, team administration
- **Capabilities**: User role management, asset assignment, business configuration

### C - Content & Catalogs

#### catalog_management
- **Purpose**: Manage product catalogs for e-commerce
- **Access Level**: Advanced (Review Required)
- **Use Cases**: E-commerce integration, product synchronization
- **Operations**: Product CRUD operations, catalog updates, inventory management

### E - Email & Essential Data

#### email
- **Purpose**: Access user's email address
- **Access Level**: Standard (No Review Required)
- **Use Cases**: Account creation, user identification, communication
- **Data Provided**: Primary email address, verification status

### G - General User Data

#### gaming_user_picture
- **Purpose**: Access user's gaming profile picture
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Gaming applications, social gaming features
- **Scope**: Gaming-specific profile information

### I - Instagram & Identity

#### instagram_basic
- **Purpose**: Basic Instagram account access
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Instagram content integration, basic profile access
- **Data Access**: Basic profile information, media access

#### instagram_content_publish
- **Purpose**: Publish content to Instagram
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Social media management tools, content automation
- **Capabilities**: Photo/video publishing, story creation

### L - Leads & Location

#### leads_retrieval
- **Purpose**: Access lead generation data
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Lead management systems, CRM integration
- **Data Access**: Lead forms data, contact information

### M - Messaging & Management

#### manage_fundraisers
- **Purpose**: Manage fundraising campaigns
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Nonprofit management, fundraising platforms
- **Capabilities**: Campaign management, donation tracking

### P - Pages & Publishing

#### pages_read_engagement
- **Purpose**: Read Page engagement data
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Social media analytics, engagement monitoring
- **Data Access**: Comments, likes, shares, page interactions

#### pages_manage_posts
- **Purpose**: Manage Page posts and content
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Social media management, content publishing
- **Capabilities**: Post creation, editing, deletion, scheduling

#### pages_messaging
- **Purpose**: Send and receive Page messages
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Customer support, automated messaging
- **Features**: Message sending, conversation management

#### public_profile
- **Purpose**: Access basic public profile information
- **Access Level**: Standard (No Review Required)
- **Use Cases**: User identification, basic personalization
- **Data Provided**: Name, profile picture, age range, gender

### R - Research & Reading

#### research
- **Purpose**: Access research and academic data
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Academic research, data analysis
- **Requirements**: Academic institution verification

### T - Transactions & Tracking

#### user_age_range
- **Purpose**: Access user's age range
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Age-appropriate content, compliance verification
- **Data Format**: Age range categories (e.g., 18-24, 25-34)

#### user_birthday
- **Purpose**: Access user's birthday
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Personalization, age verification, special offers
- **Privacy**: Sensitive personal information requiring justification

### U - User Data & Engagement

#### user_friends
- **Purpose**: Access user's friend list
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Social features, friend connections
- **Limitations**: Limited to friends who also use your app

#### user_hometown
- **Purpose**: Access user's hometown information
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Location-based features, regional content
- **Data Type**: City and geographic information

#### user_location
- **Purpose**: Access user's current location
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Location services, local recommendations
- **Privacy**: High-sensitivity permission requiring strong justification

#### user_photos
- **Purpose**: Access user's photos
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Photo integration, profile enhancement
- **Scope**: Photos user has uploaded or been tagged in

#### user_posts
- **Purpose**: Access user's posts and timeline
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Content analysis, social integration
- **Privacy**: Highly sensitive data requiring strong use case

#### user_videos
- **Purpose**: Access user's videos
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Video integration, content analysis
- **Scope**: Videos user has uploaded or been tagged in

### W - WhatsApp & Web

#### whatsapp_business_management
- **Purpose**: Manage WhatsApp Business accounts
- **Access Level**: Advanced (Review Required)
- **Use Cases**: Business messaging platforms, customer communication
- **Capabilities**: Account management, message templates, webhook configuration

## Implementation Best Practices

### Permission Strategy

#### Minimal Permission Principle
- **Start Small**: Begin with only essential permissions
- **Progressive Enhancement**: Add permissions as features require them
- **User Education**: Explain why each permission is needed
- **Value Proposition**: Clearly communicate benefits to users

#### User Experience Optimization
- **Contextual Requests**: Ask for permissions when features need them
- **Clear Explanations**: Provide clear, non-technical explanations
- **Graceful Degradation**: Function with partial permissions
- **Alternative Flows**: Provide alternatives for denied permissions

### Permission Request Timing

#### Initial Authentication
Request only essential permissions:
- **public_profile**: For basic user identification
- **email**: For account creation and communication
- **Core Functionality**: Permissions absolutely required for basic app function

#### Feature-Based Requests
Request additional permissions when accessing features:
- **Social Sharing**: Request publishing permissions when user wants to share
- **Photo Features**: Request photo access when user wants to upload/edit photos
- **Location Services**: Request location when user enables location features

#### Progressive Enhancement
Gradually increase permission scope:
- **Feature Introduction**: Introduce new features that require additional permissions
- **User Engagement**: Request advanced permissions from engaged users
- **Value Demonstration**: Show value before requesting sensitive permissions

### Error Handling

#### Permission Denied Scenarios
Handle permission denial gracefully:
- **Fallback Functionality**: Provide alternative features when permissions denied
- **User Guidance**: Explain impact of denied permissions
- **Re-request Strategy**: Implement smart re-request logic
- **Graceful Degradation**: Maintain core functionality with minimal permissions

#### Permission Expiration
Handle permission expiration proactively:
- **Expiration Monitoring**: Track permission usage to prevent expiration
- **Renewal Prompts**: Prompt users to renew expiring permissions
- **Re-authentication Flow**: Smooth re-authentication experience
- **State Management**: Maintain app state during permission renewal

## Development and Testing

### Development Environment

#### Testing Permissions
- **Test Users**: Create test users for permission testing
- **Role-Based Testing**: Test different user roles and permission combinations
- **Permission Scenarios**: Test grant, deny, and partial approval scenarios
- **Error Conditions**: Test permission errors and edge cases

#### Debug Tools
- **Access Token Debugger**: Verify token permissions and validity
- **Graph API Explorer**: Test API calls with different permission sets
- **Permission Inspector**: Analyze granted permissions for users
- **Error Analysis**: Debug permission-related API errors

### Production Deployment

#### App Review Preparation
- **Use Case Documentation**: Detailed explanation of permission usage
- **Video Demonstrations**: Show how permissions are used in context
- **Privacy Policy**: Clear privacy policy explaining data usage
- **User Interface**: Screenshots showing permission request flow

#### Launch Strategy
- **Staged Rollout**: Gradually enable permissions for user segments
- **Permission Monitoring**: Monitor permission approval rates and usage
- **User Feedback**: Collect feedback on permission requests and experience
- **Optimization**: Continuously optimize permission strategy based on data

## Security and Compliance

### Data Protection

#### Privacy by Design
- **Data Minimization**: Collect only necessary data
- **Purpose Limitation**: Use data only for stated purposes
- **Storage Limitation**: Retain data only as long as necessary
- **User Rights**: Respect user data deletion and portability rights

#### Security Measures
- **Access Control**: Restrict permission-based data access
- **Audit Trails**: Log permission grants and data access
- **Encryption**: Encrypt sensitive data in transit and at rest
- **Regular Reviews**: Conduct regular security and permission audits

### Regulatory Compliance

#### GDPR Compliance
- **Lawful Basis**: Establish lawful basis for data processing
- **Consent Management**: Properly manage user consent
- **Data Subject Rights**: Implement user rights (access, deletion, portability)
- **Data Protection Impact Assessment**: Conduct DPIA for high-risk processing

#### Platform Policies
- **Developer Policies**: Comply with Facebook Developer Policies
- **Platform Terms**: Adhere to Platform Terms of Service
- **Community Standards**: Respect Facebook Community Standards
- **API Terms**: Follow specific API terms and limitations

## Analytics and Optimization

### Permission Analytics

#### Approval Rate Tracking
- **Permission-Specific Rates**: Track approval rates for each permission
- **User Segmentation**: Analyze approval rates by user demographics
- **Request Context**: Measure impact of request timing and context
- **A/B Testing**: Test different permission request strategies

#### Usage Analytics
- **Permission Utilization**: Monitor which permissions are actively used
- **Feature Adoption**: Correlate permissions with feature usage
- **User Retention**: Analyze permission impact on user retention
- **Value Realization**: Measure business value from different permissions

### Optimization Strategies

#### Conversion Optimization
- **Request Timing**: Optimize when permissions are requested
- **UI/UX Design**: Improve permission request interface design
- **Value Communication**: Better explain permission benefits
- **Progressive Disclosure**: Reveal permission benefits gradually

#### User Experience Enhancement
- **Contextual Help**: Provide help and guidance during permission requests
- **Permission Management**: Allow users to easily manage granted permissions
- **Transparency**: Provide clear information about data usage
- **User Control**: Give users granular control over their data

## API Version Information

**Current Version**: v23.0

### Version Considerations
- Permission availability may vary between API versions
- New permissions may be introduced in newer versions
- Deprecated permissions require migration to newer alternatives
- Version-specific permission behaviors and limitations

### Migration Planning
- Monitor API changelog for permission changes
- Plan migration timelines for deprecated permissions
- Test permission compatibility with new API versions
- Update documentation and user interfaces for permission changes

## Related Documentation

- [Facebook Login & OAuth](../facebook-login-oauth.md)
- [Graph API Reference](../graph-api-reference.md)
- [Business Manager](../business-manager.md)
- [System Users](../system-users.md)
- [Webhooks](../webhooks.md)

## Resources and Tools

### Development Tools
- **Meta App Dashboard**: Manage app permissions and configuration
- **Access Token Debugger**: Debug and inspect access tokens and permissions
- **Graph API Explorer**: Test API calls with different permission sets
- **Permission Inspector**: Analyze granted permissions for test users

### Documentation Resources
- **Permission Glossary**: Detailed descriptions of all available permissions
- **Use Case Examples**: Common implementation patterns and examples
- **Best Practice Guides**: Recommended approaches for permission management
- **Policy Updates**: Latest platform policy changes affecting permissions

---

*This documentation covers Facebook Permissions Reference v23.0. For the most current information and complete permission list, always refer to the official Facebook Developer Documentation.*

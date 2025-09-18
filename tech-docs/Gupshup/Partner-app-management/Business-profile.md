# Gupshup Business Profile API Documentation

This document covers the Business Profile management APIs for Gupshup Partner applications. These APIs allow you to manage business profile information, including details, about section, and profile pictures.

---

## Get Business Profile Details

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/business/profile/`

**Description:** Use this API to fetch business profile details.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/business/profile/' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "profile": {
        "address": "<address>",
        "profileEmail": "<emailId>",
        "desc": "<description>",
        "vertical": "<vertical>",
        "website1": "<website1>",
        "website2": "<website2>"
    },
    "status": "success"
}
```

### Response Fields Description

- `profile`: Object containing business profile information
  - `address`: Business address
  - `profileEmail`: Business email address
  - `desc`: Business description
  - `vertical`: Business vertical/category
  - `website1`: Primary website URL
  - `website2`: Secondary website URL
- `status`: Status of the request ("success")

### Node.js Example

```javascript
const axios = require('axios');

async function getBusinessProfile(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Business profile:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting business profile:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getBusinessProfile('your-app-id', 'your-partner-app-token');
```

---

## Update Business Profile Details

**Endpoint:** `PUT https://partner.gupshup.io/partner/app/{appId}/business/profile`

**Description:** Use this API to update business profile.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| addLine1 | {{ADDRESS_LINE_1}} | Address line 1 of the business | String | Optional |
| addLine2 | {{ADDRESS_LINE_2}} | Address line 2 of the business | String | Optional |
| city | {{CITY}} | City where business is situated | String | Optional |
| state | {{STATE}} | State where business is situated | String | Optional |
| pinCode | {{PINCODE}} | Pincode of the city where business is situated | String | Optional |
| country | {{COUNTRY}} | Country where business is situated | String | Optional |
| vertical | {{VERTICAL}} | Business vertical/category | String | Optional |
| website1 | {{WEBSITE1}} | Primary website of business | String | Optional |
| website2 | {{WEBSITE2}} | Secondary website of business | String | Optional |
| desc | {{DESCRIPTION}} | Description of the business | String | Optional |
| profileEmail | {{PROFILE_EMAIL}} | Email of business | String | Optional |

### Sample Request

```bash
curl --location --request PUT 'https://partner.gupshup.io/partner/app/{{APP_ID}}/business/profile' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'addLine1={{ADDRESS_LINE_1}}' \
--data-urlencode 'addLine2={{ADDRESS_LINE_2}}' \
--data-urlencode 'city={{CITY}}' \
--data-urlencode 'state={{STATE}}' \
--data-urlencode 'pinCode={{PINCODE}}' \
--data-urlencode 'country={{COUNTRY}}' \
--data-urlencode 'vertical={{VERTICAL}}' \
--data-urlencode 'website1={{WEBSITE1}}' \
--data-urlencode 'website2={{WEBSITE2}}' \
--data-urlencode 'desc={{DESCRIPTION}}' \
--data-urlencode 'profileEmail={{PROFILE_EMAIL}}'
```

### Sample Response

```json
{
    "profile": {
        "address": "<address>",
        "profileEmail": "<emailId>",
        "desc": "<description>",
        "vertical": "<vertical>",
        "website1": "<website1>",
        "website2": "<website2>"
    },
    "status": "success"
}
```

### Node.js Example

```javascript
const axios = require('axios');

async function updateBusinessProfile(appId, profileData, partnerAppToken) {
  try {
    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile`,
      new URLSearchParams(profileData),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Business profile updated:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating business profile:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
updateBusinessProfile('your-app-id', {
  addLine1: '123 Main Street',
  addLine2: 'Suite 100',
  city: 'New York',
  state: 'NY',
  pinCode: '10001',
  country: 'USA',
  vertical: 'Technology',
  website1: 'https://example.com',
  website2: 'https://blog.example.com',
  desc: 'A technology company providing innovative solutions',
  profileEmail: 'contact@example.com'
}, 'your-partner-app-token');
```

---

## Get Business Profile About

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/business/profile/about`

**Description:** Use this API to fetch the About section for a Profile.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/business/profile/about' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "about": {
        "message": "<about>"
    },
    "status": "success"
}
```

### Response Fields Description

- `about`: Object containing about information
  - `message`: About message/description
- `status`: Status of the request ("success")

### Node.js Example

```javascript
const axios = require('axios');

async function getBusinessProfileAbout(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/about`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Business profile about:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting business profile about:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getBusinessProfileAbout('your-app-id', 'your-partner-app-token');
```

---

## Update Business Profile About

**Endpoint:** `PUT https://partner.gupshup.io/partner/app/{appId}/business/profile/about`

**Description:** Use this API to update the about section of a business profile.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |
| about | {{ABOUT}} | Business profile about message | String | Required |

### Sample Request

```bash
curl --location --request PUT 'https://partner.gupshup.io/partner/app/{{APP_ID}}/business/profile/about' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'about={{ABOUT}}'
```

### Sample Response

```json
{
    "about": {
        "message": "<about>"
    },
    "status": "success"
}
```

### Node.js Example

```javascript
const axios = require('axios');

async function updateBusinessProfileAbout(appId, aboutMessage, partnerAppToken) {
  try {
    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/about`,
      new URLSearchParams({
        about: aboutMessage
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Business profile about updated:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating business profile about:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
updateBusinessProfileAbout('your-app-id', 'We are a leading technology company focused on innovation and customer satisfaction.', 'your-partner-app-token');
```

---

## Get Business Profile Picture

**Endpoint:** `GET https://partner.gupshup.io/partner/app/{appId}/business/profile/photo`

**Description:** Use this API to fetch the profile picture set for a business profile.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |

### Sample Request

```bash
curl --location --request GET 'https://partner.gupshup.io/partner/app/{{APP_ID}}/business/profile/photo' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}'
```

### Sample Response

```json
{
    "message": "<link>",
    "status": "success"
}
```

### Response Fields Description

- `message`: URL link to the profile picture
- `status`: Status of the request ("success")

### Node.js Example

```javascript
const axios = require('axios');

async function getBusinessProfilePicture(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/photo`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Business profile picture URL:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting business profile picture:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getBusinessProfilePicture('your-app-id', 'your-partner-app-token');
```

---

## Update Business Profile Picture

**Endpoint:** `PUT https://partner.gupshup.io/partner/app/{appId}/business/profile/photo`

**Description:** Use this API to update the profile picture set for a business profile.

### Parameters

| Parameter | Value | Description | Type | Required |
|-----------|-------|-------------|------|----------|
| Authorization | {{PARTNER_APP_TOKEN}} | Access Token for the application | String | Required |
| appId | {{APP_ID}} | Unique Identifier for Gupshup App | String | Required |
| image | {{FILE_PATH}} | Profile photo image file | File | Required |

### Sample Request

```bash
curl --location --request PUT 'https://partner.gupshup.io/partner/app/{{APP_ID}}/business/profile/photo' \
--header 'Authorization: {{PARTNER_APP_TOKEN}}' \
--form 'image=@"/path/to/file"'
```

### Sample Response

```json
{
    "message": "profile picture updated successfully",
    "status": "success"
}
```

### Node.js Example

```javascript
const axios = require('axios');
const FormData = require('form-data');
const fs = require('fs');

async function updateBusinessProfilePicture(appId, imagePath, partnerAppToken) {
  try {
    const formData = new FormData();
    formData.append('image', fs.createReadStream(imagePath));

    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/photo`,
      formData,
      {
        headers: {
          'Authorization': partnerAppToken,
          ...formData.getHeaders()
        }
      }
    );
    
    console.log('Business profile picture updated:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating business profile picture:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
updateBusinessProfilePicture('your-app-id', '/path/to/profile-image.jpg', 'your-partner-app-token');
```

---

## Node.js Code Examples

### Get Business Profile Details - Node.js

```javascript
const axios = require('axios');

async function getBusinessProfile(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Business profile:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting business profile:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getBusinessProfile('your-app-id', 'your-partner-app-token');
```

### Update Business Profile Details - Node.js

```javascript
const axios = require('axios');

async function updateBusinessProfile(appId, profileData, partnerAppToken) {
  try {
    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile`,
      new URLSearchParams({
        addLine1: profileData.addLine1,
        addLine2: profileData.addLine2,
        city: profileData.city,
        state: profileData.state,
        pinCode: profileData.pinCode,
        country: profileData.country,
        vertical: profileData.vertical,
        website1: profileData.website1,
        website2: profileData.website2,
        desc: profileData.desc,
        profileEmail: profileData.profileEmail
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Business profile updated:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating business profile:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
updateBusinessProfile('your-app-id', {
  addLine1: '123 Main Street',
  addLine2: 'Suite 100',
  city: 'New York',
  state: 'NY',
  pinCode: '10001',
  country: 'USA',
  vertical: 'Technology',
  website1: 'https://example.com',
  website2: 'https://blog.example.com',
  desc: 'A technology company providing innovative solutions',
  profileEmail: 'contact@example.com'
}, 'your-partner-app-token');
```

### Get Business Profile About - Node.js

```javascript
const axios = require('axios');

async function getBusinessProfileAbout(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/about`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Business profile about:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting business profile about:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getBusinessProfileAbout('your-app-id', 'your-partner-app-token');
```

### Update Business Profile About - Node.js

```javascript
const axios = require('axios');

async function updateBusinessProfileAbout(appId, aboutMessage, partnerAppToken) {
  try {
    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/about`,
      new URLSearchParams({
        about: aboutMessage
      }),
      {
        headers: {
          'Authorization': partnerAppToken,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
    );
    
    console.log('Business profile about updated:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating business profile about:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
updateBusinessProfileAbout('your-app-id', 'We are a leading technology company focused on innovation and customer satisfaction.', 'your-partner-app-token');
```

### Get Business Profile Picture - Node.js

```javascript
const axios = require('axios');

async function getBusinessProfilePicture(appId, partnerAppToken) {
  try {
    const response = await axios.get(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/photo`,
      {
        headers: {
          'Authorization': partnerAppToken
        }
      }
    );
    
    console.log('Business profile picture URL:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error getting business profile picture:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
getBusinessProfilePicture('your-app-id', 'your-partner-app-token');
```

### Update Business Profile Picture - Node.js

```javascript
const axios = require('axios');
const FormData = require('form-data');
const fs = require('fs');

async function updateBusinessProfilePicture(appId, imagePath, partnerAppToken) {
  try {
    const formData = new FormData();
    formData.append('image', fs.createReadStream(imagePath));

    const response = await axios.put(
      `https://partner.gupshup.io/partner/app/${appId}/business/profile/photo`,
      formData,
      {
        headers: {
          'Authorization': partnerAppToken,
          ...formData.getHeaders()
        }
      }
    );
    
    console.log('Business profile picture updated:', response.data);
    return response.data;
  } catch (error) {
    console.error('Error updating business profile picture:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
updateBusinessProfilePicture('your-app-id', '/path/to/profile-image.jpg', 'your-partner-app-token');
```

### Complete Business Profile Manager Class - Node.js

```javascript
const axios = require('axios');
const FormData = require('form-data');
const fs = require('fs');

class GupshupBusinessProfileManager {
  constructor(partnerAppToken) {
    this.partnerAppToken = partnerAppToken;
    this.baseUrl = 'https://partner.gupshup.io/partner/app';
  }

  async getProfile(appId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/business/profile/`,
        {
          headers: {
            'Authorization': this.partnerAppToken
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }

  async updateProfile(appId, profileData) {
    try {
      const response = await axios.put(
        `${this.baseUrl}/${appId}/business/profile`,
        new URLSearchParams(profileData),
        {
          headers: {
            'Authorization': this.partnerAppToken,
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }

  async getAbout(appId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/business/profile/about`,
        {
          headers: {
            'Authorization': this.partnerAppToken
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }

  async updateAbout(appId, aboutMessage) {
    try {
      const response = await axios.put(
        `${this.baseUrl}/${appId}/business/profile/about`,
        new URLSearchParams({ about: aboutMessage }),
        {
          headers: {
            'Authorization': this.partnerAppToken,
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }

  async getProfilePicture(appId) {
    try {
      const response = await axios.get(
        `${this.baseUrl}/${appId}/business/profile/photo`,
        {
          headers: {
            'Authorization': this.partnerAppToken
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }

  async updateProfilePicture(appId, imagePath) {
    try {
      const formData = new FormData();
      formData.append('image', fs.createReadStream(imagePath));

      const response = await axios.put(
        `${this.baseUrl}/${appId}/business/profile/photo`,
        formData,
        {
          headers: {
            'Authorization': this.partnerAppToken,
            ...formData.getHeaders()
          }
        }
      );
      
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data || error.message,
        status: error.response?.status 
      };
    }
  }
}

// Usage
const profileManager = new GupshupBusinessProfileManager('your-partner-app-token');

async function main() {
  const appId = 'your-app-id';
  
  // Get profile
  const profile = await profileManager.getProfile(appId);
  console.log('Profile:', profile);
  
  // Update profile
  const updateResult = await profileManager.updateProfile(appId, {
    addLine1: '123 Main Street',
    city: 'New York',
    state: 'NY',
    country: 'USA',
    vertical: 'Technology',
    website1: 'https://example.com',
    desc: 'A technology company',
    profileEmail: 'contact@example.com'
  });
  console.log('Update result:', updateResult);
  
  // Get about
  const about = await profileManager.getAbout(appId);
  console.log('About:', about);
  
  // Update about
  const aboutResult = await profileManager.updateAbout(appId, 'We are a leading technology company.');
  console.log('About update result:', aboutResult);
  
  // Get profile picture
  const picture = await profileManager.getProfilePicture(appId);
  console.log('Profile picture:', picture);
  
  // Update profile picture
  const pictureResult = await profileManager.updateProfilePicture(appId, '/path/to/image.jpg');
  console.log('Picture update result:', pictureResult);
}

main();
```

---

## Important Notes

### General

- All APIs require Partner App Access Token for authentication
- Profile updates are optional - you can update individual fields as needed
- Profile picture must be uploaded as a file using multipart/form-data

### Profile Data Management

- Address information can be updated in parts (address lines, city, state, country)
- Multiple websites can be specified (website1, website2)
- Business vertical helps categorize your business
- Profile email should be a valid business contact email

### Profile Picture Requirements

- Profile pictures must be uploaded as files
- Supported formats may vary (typically JPG, PNG)
- Consider image size limitations for optimal performance
- The API returns a success message upon successful upload

### Best Practices

1. **Complete Profile Information**: Fill out all relevant fields for better business representation
2. **Regular Updates**: Keep profile information current and accurate
3. **Professional Image**: Use high-quality, professional profile pictures
4. **Valid Contact Information**: Ensure email addresses and websites are accessible
5. **Error Handling**: Implement proper error handling for all API calls
6. **File Validation**: Validate image files before uploading
7. **About Section**: Keep the about section informative but concise

### Error Handling

- Check response status codes for successful operations
- Handle file upload errors for profile picture updates
- Validate input data before making API calls
- Implement retry logic for network-related failures

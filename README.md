# EyeOTmonitor Customer API Documentation

## Overview

The EyeOTmonitor API provides secure access to device monitoring data. This RESTful API uses JWT (JSON Web Token) authentication and supports account-based access control to ensure users can only access devices within their authorized accounts.

## Base URL

**Production:** `https://api-c.eyeotmonitor.net/customer`

## Authentication

### 1. Login and Get JWT Token

All API requests require authentication. First, obtain a JWT token by logging in with your credentials.

**Endpoint:** `POST /v1/auth/login`

**Request:**
```json
{
  "username": "your_username",
  "password": "your_password"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires": "2025-07-16T10:30:00Z",
    "accounts": [
      {
        "AccountId": "921184C4-5CF0-461D-8889-50E304170076",
        "AccountName": "Gotham Pizza"
      },
      {
        "AccountId": "980e4567-e89b-12d3-a456-426614174000",
        "AccountName": "City of Metropolis"
      }
    ]
  }
}
```

**Important Notes:**
- Save the `token` for use in subsequent API calls
- Note the `accounts` array - you can only access devices within these accounts
- The token expires at the specified `expires` timestamp

### 2. Using the JWT Token

Include the JWT token in the `Authorization` header for all subsequent requests:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Device Management

### Get All Devices

Retrieve metadata for all devices within a specific account.

**Endpoint:** `GET /v1/devices`

**Required Query Parameters:**
- `accountId` - The account ID to filter devices by (must be one of your authorized accounts)

**Request:**
```http
GET /v1/devices?accountId=921184C4-5CF0-461D-8889-50E30417007
Authorization: Bearer your_jwt_token_here
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "DeviceId": "device-123",
      "Name": "Outside Cam 01",
      "ModelName": "Axis 360",
      "SerialNumber": "TS001234",
      "IPAddress": "192.168.1.100",
      "MacAddress": "00:1B:44:11:3A:B7",
      "Description": "Main facility monitoring",
      "TypeId": 1,
      "TypeName": "Axis Digital Camera",
      "Status": "Online",
      "StatusId": 1,
      "LastStatusUpdate": "2025-07-15T14:30:00Z",
      "AccountId": "921184C4-5CF0-461D-8889-50E30417007"
    },
    {
      "DeviceId": "device-456",
      "Name": "Indoor Cam",
      "ModelName": "Axis 360",
      "SerialNumber": "PM002345",
      "IPAddress": "192.168.1.101",
      "MacAddress": "00:1B:44:11:3A:B8",
      "Description": "Secondary monitoring station",
      "TypeId": 2,
      "TypeName": "Axis Digital Camera",
      "Status": "Offline",
      "StatusId": 2,
      "LastStatusUpdate": "2025-07-15T12:15:00Z",
      "AccountId": "921184C4-5CF0-461D-8889-50E30417007"
    }
  ]
}
```

### Get Device by ID

Retrieve detailed metadata for a specific device.

**Endpoint:** `GET /v1/devices/{deviceId}`

**Path Parameters:**
- `deviceId` - The unique identifier of the device

**Required Query Parameters:**
- `accountId` - The account ID that owns the device

**Request:**
```http
GET /v1/devices/device-123?accountId=921184C4-5CF0-461D-8889-50E30417007
Authorization: Bearer your_jwt_token_here
```

**Response:**
```json
{
  "success": true,
  "data": {
    "DeviceId": "device-123",
    "Name": "Outside Cam 01",
    "ModelName": "Axis 360",
    "SerialNumber": "TS001234",
    "IPAddress": "192.168.1.100",
    "MacAddress": "00:1B:44:11:3A:B7",
    "Description": "Main facility monitoring",
    "TypeId": 1,
    "TypeName": "Axis Digital Camera",
    "Status": "Online",
    "StatusId": 1,
    "LastStatusUpdate": "2025-07-15T14:30:00Z",
    "AccountId": "921184C4-5CF0-461D-8889-50E30417007"
  }
}
```

## Account-Based Security

The API implements strict account-based access control:

1. **Account Validation**: You can only access devices within accounts you're authorized for
2. **Required Parameters**: The `accountId` parameter is required for all device operations
3. **Access Denied**: Attempting to access devices in unauthorized accounts returns a 403 Forbidden error

## Error Responses

### Common Error Codes

| Status Code | Description | Example Response |
|-------------|-------------|------------------|
| 400 | Bad Request - Missing required parameters | `{"success": false, "message": "Account ID is required"}` |
| 401 | Unauthorized - Invalid or missing token | `{"success": false, "message": "Invalid or missing authentication token"}` |
| 403 | Forbidden - Access denied to account | `{"success": false, "message": "Access denied to account 'invalid-account-id'"}` |
| 404 | Not Found - Device not found | `{"success": false, "message": "Device with ID 'device-123' not found for account 'account-id'"}` |
| 500 | Internal Server Error | `{"success": false, "message": "An error occurred while retrieving devices"}` |

### Error Response Format

All error responses follow this structure:
```json
{
  "success": false,
  "message": "Error description here"
}
```

## Sample Code Examples

### Python Example

```python
import requests
import json

class EyeOTClient:
    def __init__(self, base_url):
        self.base_url = base_url
        self.token = None
        self.accounts = []
    
    def login(self, username, password):
        """Authenticate and get JWT token"""
        response = requests.post(
            f"{self.base_url}/v1/auth/login",
            json={"username": username, "password": password},
            headers={"Content-Type": "application/json"}
        )
        
        if response.status_code == 200:
            data = response.json()['data']
            self.token = data['token']
            self.accounts = data['accounts']
            return True
        else:
            raise Exception(f"Authentication failed: {response.text}")
    
    def get_devices(self, account_id):
        """Get all devices for an account"""
        headers = {"Authorization": f"Bearer {self.token}"}
        params = {"accountId": account_id}
        
        response = requests.get(
            f"{self.base_url}/v1/devices",
            headers=headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json()['data']
        else:
            raise Exception(f"Failed to get devices: {response.text}")
    
    def get_device(self, device_id, account_id):
        """Get specific device by ID"""
        headers = {"Authorization": f"Bearer {self.token}"}
        params = {"accountId": account_id}
        
        response = requests.get(
            f"{self.base_url}/v1/devices/{device_id}",
            headers=headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json()['data']
        else:
            raise Exception(f"Failed to get device: {response.text}")

# Usage example
client = EyeOTClient("https://api-c.eyeotmonitor.net/customer")

# Login
client.login("your_username", "your_password")

# Get first account ID
account_id = client.accounts[0]['AccountId']

# Get all devices for the account
devices = client.get_devices(account_id)
print(f"Found {len(devices)} devices")

# Get specific device
if devices:
    device = client.get_device(devices[0]['DeviceId'], account_id)
    print(f"Device: {device['Name']}")
```

### JavaScript Example

```javascript
class EyeOTClient {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
        this.token = null;
        this.accounts = [];
    }
    
    async login(username, password) {
        const response = await fetch(`${this.baseUrl}/v1/auth/login`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ username, password })
        });
        
        if (response.ok) {
            const result = await response.json();
            this.token = result.data.token;
            this.accounts = result.data.accounts;
            return true;
        } else {
            throw new Error(`Authentication failed: ${await response.text()}`);
        }
    }
    
    async getDevices(accountId) {
        const url = new URL(`${this.baseUrl}/v1/devices`);
        url.searchParams.append('accountId', accountId);
        
        const response = await fetch(url, {
            headers: {
                'Authorization': `Bearer ${this.token}`
            }
        });
        
        if (response.ok) {
            const result = await response.json();
            return result.data;
        } else {
            throw new Error(`Failed to get devices: ${await response.text()}`);
        }
    }
    
    async getDevice(deviceId, accountId) {
        const url = new URL(`${this.baseUrl}/v1/devices/${deviceId}`);
        url.searchParams.append('accountId', accountId);
        
        const response = await fetch(url, {
            headers: {
                'Authorization': `Bearer ${this.token}`
            }
        });
        
        if (response.ok) {
            const result = await response.json();
            return result.data;
        } else {
            throw new Error(`Failed to get device: ${await response.text()}`);
        }
    }
}

// Usage example
(async () => {
    const client = new EyeOTClient('https://api-c.eyeotmonitor.net/customer');
    
    // Login
    await client.login('your_username', 'your_password');
    
    // Get first account ID
    const accountId = client.accounts[0].AccountId;
    
    // Get all devices
    const devices = await client.getDevices(accountId);
    console.log(`Found ${devices.length} devices`);
    
    // Get specific device
    if (devices.length > 0) {
        const device = await client.getDevice(devices[0].DeviceId, accountId);
        console.log(`Device: ${device.Name}`);
    }
})();
```

### cURL Examples

**Login:**
```bash
curl -X POST "https://api-c.eyeotmonitor.net/customer/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password"
  }'
```

**Get Devices:**
```bash
curl -X GET "https://api-c.eyeotmonitor.net/customer/v1/devices?accountId=894184C4-5CF0-461D-8889-50E304170076" \
  -H "Authorization: Bearer your_jwt_token_here"
```

**Get Specific Device:**
```bash
curl -X GET "https://api-c.eyeotmonitor.net/customer/v1/devices/device-123?accountId=894184C4-5CF0-461D-8889-50E304170076" \
  -H "Authorization: Bearer your_jwt_token_here"
```

## Best Practices

### Security
1. **Secure Token Storage**: Store JWT tokens securely and never expose them in logs or URLs
2. **Token Expiration**: Check token expiration and re-authenticate when necessary
3. **HTTPS Only**: Always use HTTPS for API communication
4. **Account Validation**: Verify you have access to an account before making requests

### Performance
1. **Token Reuse**: Reuse JWT tokens for multiple requests until they expire
2. **Account Caching**: Cache the list of authorized accounts to avoid repeated login calls
3. **Error Handling**: Implement proper error handling and retry logic
4. **Rate Limiting**: Be mindful of API rate limits (contact support for specific limits)

### Error Handling
1. **Check Response Status**: Always check HTTP status codes before processing responses
2. **Parse Error Messages**: Use the `message` field in error responses for user-friendly error handling
3. **Retry Logic**: Implement exponential backoff for transient errors (5xx status codes)
4. **Graceful Degradation**: Handle cases where devices or accounts may not be available

## Changelog

### Version 1.0
- Initial release with device management endpoints
- JWT authentication implementation
- Account-based access control
- Support for device metadata retrieval

---

**Note**: This documentation assumes you have valid credentials and authorized access to at least one account. Contact your us at support@eyeotomonitor.com if you have any questions.

# API Endpoints

Complete reference for all Invoice Collector API endpoints.

## User Management

### Create User

Create a new user for invoice collection.

**Endpoint:** `POST /api/v1/user`

**Authentication:** Bearer Token

**Request Body:**

```json
{
  "remote_id": "user123",
  "locale": "en",
  "email": "user@example.com",
  "ip": "192.168.1.1"
}
```

**Response:**

```json
{
  "user_id": "generated_user_id",
  "token": "ui_access_token"
}
```

### List Users

Get all users for the customer.

**Endpoint:** `GET /api/v1/users`

**Authentication:** Bearer Token

**Response:**

```json
[
  {
    "user_id": "user_id_1",
    "remote_id": "user123",
    "email": "user@example.com",
    "locale": "en",
    "created_at": 1609459200000
  }
]
```

### Delete User

Delete a user and all associated credentials.

**Endpoint:** `DELETE /api/v1/user/{user_id}`

**Authentication:** Bearer Token

**Response:** `200 OK`

## Credential Management

### Add Credential

Add supplier credentials for a user.

**Endpoint:** `POST /api/v1/user/{user_id}/credential`

**Authentication:** Bearer Token or UI Token

**Request Body:**

```json
{
  "collector": "amazon",
  "params": {
    "email": "amazon@example.com",
    "password": "password123"
  },
  "download_from_timestamp": 1609459200000
}
```

**Response:**

```json
{
  "credential_id": "generated_credential_id"
}
```

### List Credentials

Get all credentials for a user.

**Endpoint:** `GET /api/v1/user/{user_id}/credentials`

**Authentication:** Bearer Token or UI Token

**Response:**

```json
[
  {
    "credential_id": "cred123",
    "collector": "amazon",
    "status": "success",
    "last_collection": 1609459200000,
    "invoice_count": 42
  }
]
```

### Get Credential Status

Check the status of a specific credential.

**Endpoint:** `GET /api/v1/user/{user_id}/credential/{credential_id}`

**Authentication:** Bearer Token or UI Token

**Response:**

```json
{
  "credential_id": "cred123",
  "collector": "amazon",
  "status": "success",
  "last_collection": 1609459200000,
  "next_collection": 1609545600000,
  "invoice_count": 42,
  "error_message": null
}
```

### Delete Credential

Remove a credential.

**Endpoint:** `DELETE /api/v1/user/{user_id}/credential/{credential_id}`

**Authentication:** Bearer Token or UI Token

**Response:** `200 OK`

### Submit 2FA Code

Provide two-factor authentication code.

**Endpoint:** `POST /api/v1/user/{user_id}/credential/{credential_id}/2fa`

**Authentication:** Bearer Token or UI Token

**Request Body:**

```json
{
  "code": "123456"
}
```

**Response:** `200 OK`

### Trigger Collection

Manually trigger invoice collection.

**Endpoint:** `POST /api/v1/user/{user_id}/credential/{credential_id}/collect`

**Authentication:** Bearer Token or UI Token

**Response:** `200 OK`

## Collector Information

### List Collectors

Get all available collectors.

**Endpoint:** `GET /api/v1/collectors`

**Authentication:** Bearer Token or UI Token

**Query Parameters:**

- `locale` (optional) - Language for descriptions

**Response:**

```json
[
  {
    "id": "amazon",
    "name": "Amazon",
    "description": "Collect invoices from Amazon",
    "version": "2.1.0",
    "website": "https://amazon.com",
    "logo": "https://...",
    "type": "web",
    "state": "active",
    "params": {
      "email": {
        "type": "text",
        "name": "Email",
        "placeholder": "your-email@example.com",
        "mandatory": true
      },
      "password": {
        "type": "password",
        "name": "Password",
        "placeholder": "••••••••",
        "mandatory": true
      }
    }
  }
]
```

## Customer Management

### Get Customer

Retrieve customer information.

**Endpoint:** `GET /api/v1/customer`

**Authentication:** Bearer Token

**Response:**

```json
{
  "customer_id": "cust123",
  "name": "My Company",
  "callback": "https://api.example.com/invoices",
  "theme": "light",
  "subscribed_collectors": ["amazon", "ovh"],
  "is_subscribed_to_all": false
}
```

### Update Customer

Update customer settings.

**Endpoint:** `POST /api/v1/customer`

**Authentication:** Bearer Token

**Request Body:**

```json
{
  "name": "My Company",
  "callback": "https://api.example.com/invoices",
  "remote_id": "external_id",
  "theme": "dark",
  "subscribed_collectors": ["amazon"],
  "is_subscribed_to_all": true,
  "display_sketch_collectors": false
}
```

**Response:** `200 OK`

### Generate Bearer Token

Create a new bearer token.

**Endpoint:** `POST /api/v1/customer/bearer`

**Authentication:** Bearer Token

**Response:**

```json
{
  "bearer": "new_bearer_token_here"
}
```

### Get Statistics

Retrieve customer statistics.

**Endpoint:** `GET /api/v1/customer/stats`

**Authentication:** Bearer Token

**Response:**

```json
{
  "total_users": 10,
  "total_credentials": 25,
  "total_invoices": 1234,
  "collections_by_collector": {
    "amazon": 500,
    "ovh": 234,
    "leclerc": 500
  },
  "recent_collections": [
    {
      "timestamp": 1609459200000,
      "collector": "amazon",
      "invoice_count": 5
    }
  ]
}
```

## UI Access

### Access Web UI

Open the web interface for a user.

**Endpoint:** `GET /api/v1/ui`

**Authentication:** UI Token (query parameter)

**Query Parameters:**

- `token` (required) - UI access token
- `verificationCode` (optional) - Email verification code

**Response:** HTML page

## Feedback

### Submit Feedback

Send feedback or bug reports.

**Endpoint:** `POST /api/v1/feedback`

**Authentication:** Bearer Token or UI Token

**Request Body:**

```json
{
  "type": "bug",
  "message": "Description of the issue"
}
```

**Response:** `200 OK`

## Testing

### Test Callback

Test your callback endpoint configuration.

**Endpoint:** `GET /api/v1/test/callback/{type}`

**Authentication:** Bearer Token

**Parameters:**

- `type` - Callback type to test (e.g., `invoice`)

**Response:** `200 OK`

## Error Responses

All endpoints may return error responses:

### 400 Bad Request

```json
{
  "type": "error",
  "reason": "Invalid request parameters",
  "message": "Missing required field: email"
}
```

### 401 Unauthorized

```json
{
  "type": "error",
  "reason": "Unauthorized",
  "message": "Invalid or missing authentication token"
}
```

### 404 Not Found

```json
{
  "type": "error",
  "reason": "Resource not found",
  "message": "User not found"
}
```

### 500 Internal Server Error

```json
{
  "type": "error",
  "reason": "Internal server error"
}
```

## Next Steps

- [Authentication Guide](authentication.md)
- [Webhooks Setup](webhooks.md)
- [Quick Start](../getting-started/quick-start.md)

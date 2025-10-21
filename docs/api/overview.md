# API Overview

Invoice Collector provides a RESTful API for managing users, credentials, and collecting invoices.

## Base URL

```
http://localhost:8080/api/v1
```

For production, replace `localhost:8080` with your server address.

## API Versions

Currently available: **v1**

Future versions will be announced and documented separately.

## Content Types

All API requests and responses use JSON:

```
Content-Type: application/json
```

## Response Format

### Success Response

```json
{
  "type": "success",
  "data": {...}
}
```

### Error Response

```json
{
  "type": "error",
  "reason": "Error description",
  "message": "Detailed error message"
}
```

## HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorized | Authentication failed |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 500 | Server Error | Internal server error |

## Rate Limiting

Currently no rate limiting is enforced. This may change in future versions.

## API Endpoints

### Authentication
- `POST /login` - User login
- `POST /signup` - User registration
- `POST /reset` - Password reset

### Customer Management
- `GET /customer` - Get customer details
- `POST /customer` - Update customer
- `POST /customer/bearer` - Generate new bearer token
- `GET /customer/stats` - Get customer statistics

### User Management
- `GET /users` - List all users
- `POST /user` - Create new user
- `DELETE /user/{user_id}` - Delete user

### Credential Management
- `GET /user/{user_id}/credentials` - List credentials
- `POST /user/{user_id}/credential` - Add credential
- `GET /user/{user_id}/credential/{credential_id}` - Get credential status
- `DELETE /user/{user_id}/credential/{credential_id}` - Delete credential
- `POST /user/{user_id}/credential/{credential_id}/2fa` - Submit 2FA code
- `POST /user/{user_id}/credential/{credential_id}/collect` - Trigger collection

### Collector Information
- `GET /collectors` - List available collectors

### UI Access
- `GET /ui` - Access web interface

### Feedback
- `POST /feedback` - Submit feedback

## Quick Start

### 1. Get Bearer Token

First, obtain a bearer token for authentication (setup specific).

### 2. Create a User

```bash
curl -X POST http://localhost:8080/api/v1/user \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "remote_id": "user001",
    "locale": "en",
    "email": "user@example.com",
    "ip": "192.168.1.1"
  }'
```

### 3. Add a Credential

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "collector": "amazon",
    "params": {
      "email": "amazon@example.com",
      "password": "password"
    },
    "download_from_timestamp": 0
  }'
```

### 4. Check Status

```bash
curl -X GET http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## SDKs and Libraries

Currently, no official SDKs are available. The API is REST-based and can be used with any HTTP client.

Community contributions for SDKs are welcome!

## API Stability

- **Stable**: All v1 endpoints are stable
- **Backwards Compatibility**: Breaking changes will result in a new version
- **Deprecation**: Deprecated endpoints will be announced 6 months in advance

## Next Steps

- [Authentication Guide](authentication.md)
- [Endpoint Reference](endpoints.md)
- [Webhooks Configuration](webhooks.md)

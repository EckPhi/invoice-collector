# Managing Credentials

Learn how to add, manage, and secure supplier credentials in Invoice Collector.

## Overview

Credentials are the login information needed to access supplier portals. Invoice Collector stores them securely using Bitwarden encryption.

## Adding Credentials

### Via API

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "collector": "amazon",
    "params": {
      "email": "user@example.com",
      "password": "password123"
    },
    "download_from_timestamp": 1609459200000
  }'
```

### Via Web UI

1. Navigate to `http://localhost:8080/api/v1/ui?token=USER_TOKEN`
2. Click "Add Credential"
3. Select the supplier
4. Enter your credentials
5. Click "Save"

## Credential Parameters

Each collector requires specific parameters:

### Common Parameters

- **email** - Email address for login
- **password** - Account password
- **username** - Username (alternative to email)

### Collector-Specific Parameters

Some collectors need additional information:

- **customer_id** - For OVH and similar services
- **card_number** - For loyalty card-based systems (Leclerc)
- **api_key** - For API-based collectors (OpenAI)

!!! tip
    Use `GET /api/v1/collectors` to see required parameters for each collector.

## Setting Collection Date

The `download_from_timestamp` parameter (in milliseconds) controls from which date to collect invoices:

```javascript
// Collect all invoices
"download_from_timestamp": 0

// Collect from specific date (e.g., Jan 1, 2021)
"download_from_timestamp": 1609459200000

// Collect from one year ago
"download_from_timestamp": Date.now() - (365 * 24 * 60 * 60 * 1000)
```

## Checking Credential Status

Monitor the collection status:

```bash
curl -X GET http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

### Status Values

- `pending` - Waiting to start
- `collecting` - Currently collecting
- `2fa_required` - Needs 2FA code
- `success` - Collection complete
- `error` - An error occurred

## Handling Two-Factor Authentication

If a credential requires 2FA:

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID/2fa \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"code": "123456"}'
```

## Updating Credentials

To update existing credentials, delete the old credential and create a new one:

```bash
# Delete old credential
curl -X DELETE http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"

# Add new credential
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

## Deleting Credentials

Remove credentials when no longer needed:

```bash
curl -X DELETE http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

!!! warning
    Deleting a credential does not delete previously collected invoices.

## Manual Collection Trigger

Trigger collection manually:

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID/collect \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

## Security Best Practices

1. **Use Strong Passwords** - Ensure supplier credentials are strong
2. **Enable 2FA** - Use two-factor authentication when available
3. **Regular Review** - Audit credentials periodically
4. **Remove Unused** - Delete credentials for closed accounts
5. **Monitor Access** - Check credential usage logs

## Troubleshooting

### Credential Not Working

- Verify credentials are correct by logging in manually
- Check if supplier has changed authentication method
- Ensure 2FA is configured if required

### Collection Fails Repeatedly

- Check collector status (may be in maintenance)
- Verify account has active access
- Review error messages in credential status

### 2FA Not Working

- Ensure code is entered quickly (codes expire)
- Verify correct 2FA method is configured
- Check time synchronization on your device

## Next Steps

- [Learn about collectors](collectors.md)
- [Work with invoices](invoices.md)
- [Configure callbacks](../api/webhooks.md)

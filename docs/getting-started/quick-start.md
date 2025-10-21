# Quick Start Guide

This guide will help you get started with Invoice Collector and collect your first invoices.

## Overview

Invoice Collector follows a simple workflow:

1. **Create a User** - Create a user account for each person
2. **Add Credentials** - Add supplier credentials for the user
3. **Collect Invoices** - Automatically collect invoices from suppliers
4. **Retrieve Invoices** - Access collected invoices via API or UI

## Before You Begin

Make sure you have:

- ✅ [Installed Invoice Collector](installation.md)
- ✅ [Configured environment variables](configuration.md)
- ✅ Invoice Collector running at `http://localhost:8080`

## Step 1: Get Your Bearer Token

To interact with the API, you need a bearer token. For the initial setup, you can generate one through your customer configuration.

!!! info "Authentication"
    Invoice Collector uses bearer token authentication for API requests. See the [Authentication Guide](../api/authentication.md) for more details.

## Step 2: Create a User

Create a user account that will be used to store credentials and invoices:

```bash
curl -X POST http://localhost:8080/api/v1/user \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "remote_id": "user123",
    "locale": "en",
    "email": "user@example.com",
    "ip": "192.168.1.1"
  }'
```

The response will include a `user_id` and a `token` that can be used for UI access.

## Step 3: View Available Collectors

List all available collectors to see which suppliers are supported:

```bash
curl -X GET http://localhost:8080/api/v1/collectors \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

This will return a list of collectors with their IDs, names, and required parameters.

## Step 4: Add Credentials

Add credentials for a supplier. For example, to add Amazon credentials:

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "collector": "amazon",
    "params": {
      "email": "your-amazon-email@example.com",
      "password": "your-amazon-password"
    },
    "download_from_timestamp": 1609459200000
  }'
```

!!! tip "Timestamp"
    The `download_from_timestamp` parameter (in milliseconds) determines from which date to start collecting invoices. Use `0` to collect all available invoices.

The response will include a `credential_id`.

## Step 5: Monitor Collection Status

Check the status of the credential to see if invoices are being collected:

```bash
curl -X GET http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

The status can be:

- `pending` - Waiting to start collection
- `collecting` - Currently collecting invoices
- `success` - Collection completed successfully
- `error` - An error occurred
- `2fa_required` - Two-factor authentication code needed

## Step 6: Provide 2FA Code (if required)

If the collector requires a two-factor authentication code:

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID/2fa \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "123456"
  }'
```

## Step 7: Access Collected Invoices

Once collection is complete, the invoices are sent to your configured callback URL. You can also view them through the UI:

```
http://localhost:8080/api/v1/ui?token=USER_TOKEN
```

## Using the Web UI

Instead of using the API directly, you can use the web UI:

1. Open `http://localhost:8080/api/v1/ui?token=USER_TOKEN`
2. Accept the terms and conditions
3. Click "Add Credential" to add a new supplier
4. Select the collector and enter your credentials
5. Click "Save" to start collecting invoices

!!! success "Congratulations!"
    You've successfully set up Invoice Collector and collected your first invoices!

## Automating Collection

Invoice Collector can automatically collect invoices on a schedule. To trigger manual collection:

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential/CREDENTIAL_ID/collect \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

## Next Steps

- [Learn more about collectors](../guides/collectors.md)
- [Configure webhooks for callbacks](../api/webhooks.md)
- [Explore the full API](../api/endpoints.md)
- [Set up automatic collection schedules](../guides/overview.md)

## Common Issues

### Credentials Not Working

- Verify your credentials are correct
- Some suppliers may have additional security requirements
- Check if 2FA is required

### No Invoices Found

- Verify the `download_from_timestamp` is correct
- Some suppliers may have limited invoice history
- Check the collector status for errors

### Collection Takes Too Long

- Initial collection may take time depending on invoice count
- Subsequent collections are usually faster
- Monitor the credential status for progress

For more help, visit our [Discord community](https://discord.gg/dMXTdpxMqY).

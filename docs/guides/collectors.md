# Collectors Guide

Collectors are specialized modules that know how to retrieve invoices from specific suppliers. This guide explains the different types of collectors and how to use them.

## Available Collectors

Invoice Collector supports the following suppliers:

### Core Collectors

These collectors are officially maintained and production-ready:

#### Amazon
- **Type:** Web Collector
- **Region:** Multiple (depends on account)
- **Requirements:** Email and password
- **2FA:** Supported
- **Description:** Retrieves order invoices from Amazon

#### Bureau Vallée
- **Type:** Web Collector
- **Region:** France
- **Requirements:** Email and password
- **Description:** Office supplies and stationery retailer

#### Carrefour
- **Type:** Web Collector
- **Region:** France
- **Requirements:** Email and password
- **Description:** Major supermarket chain

#### Free
- **Type:** Web Collector
- **Region:** France
- **Requirements:** Login credentials
- **Description:** Internet and mobile service provider

#### Intermarché
- **Type:** Web Collector
- **Region:** France
- **Requirements:** Email and password
- **Description:** Supermarket chain

#### Leclerc
- **Type:** Web Collector
- **Region:** France
- **Requirements:** Card number and password
- **Description:** Supermarket chain with loyalty card

#### Leroy Merlin
- **Type:** Web Collector
- **Region:** Multiple
- **Requirements:** Email and password
- **Description:** Home improvement and gardening retailer

#### OpenAI
- **Type:** API Collector
- **Region:** Global
- **Requirements:** API key
- **Description:** AI services invoices

#### OVH
- **Type:** Web Collector
- **Region:** Europe
- **Requirements:** Customer ID and password
- **Description:** Cloud and hosting services

#### U (Système U)
- **Type:** Web Collector
- **Region:** France
- **Requirements:** Email and password
- **Description:** Supermarket cooperative

### Community Collectors

Community-contributed collectors are available but may have varying levels of support.

### Sketch Collectors

Experimental collectors in development. Use with caution in production environments.

## Collector Types

### Web Collectors

Web collectors use browser automation (Puppeteer) to:

1. Navigate to the supplier's website
2. Log in with user credentials
3. Navigate to the invoices section
4. Download all available invoices

**Characteristics:**

- ✅ Works with any website
- ✅ Handles complex authentication flows
- ✅ Supports 2FA when needed
- ⚠️ Can be slower than API collectors
- ⚠️ May be affected by website changes

### API Collectors

API collectors connect directly to supplier APIs:

1. Authenticate using API keys or OAuth
2. Request invoice data via API endpoints
3. Download invoice documents

**Characteristics:**

- ✅ Fast and efficient
- ✅ Stable (APIs change less than websites)
- ✅ Lower resource usage
- ⚠️ Requires API access from supplier
- ⚠️ May have rate limits

### Email Collectors

Email collectors extract invoices from email:

1. Connect to email inbox (IMAP)
2. Search for invoice emails
3. Extract attachments or download links

**Characteristics:**

- ✅ Works when suppliers email invoices
- ✅ Reliable delivery
- ⚠️ Requires email access
- ⚠️ May miss in-portal-only invoices

## Using Collectors

### Listing Available Collectors

Get a list of all available collectors:

```bash
curl -X GET http://localhost:8080/api/v1/collectors \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

Response includes:

- Collector ID
- Display name
- Description
- Required parameters
- Collector state (active, maintenance, development)

### Adding a Credential

To use a collector, add a credential with the required parameters:

```bash
curl -X POST http://localhost:8080/api/v1/user/USER_ID/credential \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "collector": "amazon",
    "params": {
      "email": "your-email@example.com",
      "password": "your-password"
    },
    "download_from_timestamp": 0
  }'
```

### Collector Parameters

Each collector requires specific parameters. Common parameters:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `email` | Email address | `user@example.com` |
| `password` | Account password | `SecurePass123` |
| `username` | Username (alternative to email) | `john_doe` |
| `customer_id` | Customer account ID | `12345678` |
| `card_number` | Loyalty card number | `9876543210` |
| `api_key` | API access key | `sk_live_...` |

!!! tip "Finding Parameters"
    Use the `/api/v1/collectors` endpoint to see exactly which parameters each collector requires.

## Collector States

Collectors can be in different states:

### Active
- Fully functional and production-ready
- Regularly maintained and tested
- Recommended for production use

### Maintenance
- Temporarily unavailable
- Usually due to supplier website changes
- Will be fixed in upcoming updates

### Development
- New collectors being tested
- May have bugs or incomplete features
- Use with caution

## Two-Factor Authentication

Some collectors support or require 2FA:

### Detecting 2FA Requirement

After adding credentials, check the status:

```bash
GET /api/v1/user/USER_ID/credential/CREDENTIAL_ID
```

If status is `2fa_required`, provide the code:

```bash
POST /api/v1/user/USER_ID/credential/CREDENTIAL_ID/2fa
{
  "code": "123456"
}
```

### Supported 2FA Methods

- SMS codes
- Authenticator app (TOTP)
- Email codes
- Push notifications (when available)

## Troubleshooting Collectors

### Collector Not Working

1. **Check Collector State** - Verify it's in `active` state
2. **Verify Credentials** - Ensure login details are correct
3. **Check 2FA** - Some suppliers may have added 2FA
4. **Review Logs** - Look for error messages in collector status

### No Invoices Found

1. **Check Date Range** - Verify `download_from_timestamp` is appropriate
2. **Verify Account** - Ensure account has invoices available
3. **Portal Changes** - Supplier may have changed their website
4. **Collector Updates** - Check for new collector versions

### Collection Taking Long

1. **Large Invoice Count** - Initial collection may take time
2. **Slow Supplier Portal** - Some websites are naturally slow
3. **Anti-Bot Protection** - May require proxy or slower execution
4. **Network Issues** - Check connectivity

## Advanced Features

### Custom Proxies

Some collectors support proxy configuration for:

- Bypassing geo-restrictions
- Avoiding anti-bot detection
- Improving reliability

Configure via environment variables (see [Configuration](../getting-started/configuration.md)).

### AI-Enhanced Extraction

Certain collectors can use AI for:

- Better data extraction
- Handling layout changes
- OCR for scanned documents

Requires Mistral API key configuration.

## Contributing New Collectors

Want to add support for a new supplier? See the [Creating Collectors](../developers/creating-collectors.md) guide!

## Collector Comparison

| Feature | Web | API | Email |
|---------|-----|-----|-------|
| Speed | Medium | Fast | Medium |
| Reliability | Medium | High | High |
| Setup Complexity | Low | Medium | Medium |
| Maintenance | High | Low | Low |
| Resource Usage | High | Low | Medium |

## Best Practices

1. **Start Small** - Test with one supplier first
2. **Monitor Status** - Regularly check credential status
3. **Update Credentials** - Keep passwords current
4. **Use Appropriate Dates** - Don't collect unnecessary historical data
5. **Handle 2FA** - Implement 2FA code handling in your workflow

## Next Steps

- [Managing credentials](credentials.md)
- [Working with invoices](invoices.md)
- [Creating your own collector](../developers/creating-collectors.md)
- [API integration](../api/overview.md)

## Getting Help

- 💬 [Discord](https://discord.gg/dMXTdpxMqY) - Ask about specific collectors
- 🐛 [Report Issues](https://github.com/invoice-collector/invoice-collector/issues) - Collector not working?
- 📖 [Development Guide](../developers/creating-collectors.md) - Build your own

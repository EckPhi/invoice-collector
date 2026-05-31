# Webhooks

Configure webhooks to receive invoice notifications and events from Invoice Collector.

## Overview

Webhooks allow Invoice Collector to send data to your application when events occur, such as:

- New invoice collected
- Collection completed
- Collection failed
- Credential status changed

## Configuration

Set your callback URL when configuring the customer:

```bash
curl -X POST http://localhost:8080/api/v1/customer \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Company",
    "callback": "https://api.yourapp.com/webhooks/invoice-collector"
  }'
```

## Invoice Webhook

### Payload

When a new invoice is collected, Invoice Collector sends:

```json
{
  "type": "invoice",
  "user_id": "user_id_here",
  "credential_id": "credential_id_here",
  "collector": "amazon",
  "invoice": {
    "id": "unique_invoice_id",
    "timestamp": 1609459200000,
    "amount": "99.99",
    "link": "https://supplier.com/invoice/123",
    "data": "JVBERi0xLjQK...",
    "mimetype": "application/pdf",
    "collected_timestamp": 1609459300000,
    "metadata": {
      "order_id": "123456",
      "supplier_name": "Amazon"
    }
  }
}
```

### Fields

- **type** - Event type (`invoice`)
- **user_id** - User identifier
- **credential_id** - Credential identifier
- **collector** - Collector ID
- **invoice** - Invoice data object

## Implementing a Webhook Endpoint

### Requirements

Your webhook endpoint must:

1. Accept POST requests
2. Return HTTP 200 on success
3. Respond within 30 seconds
4. Handle duplicate deliveries idempotently

### Example Implementation

**Node.js/Express:**

```javascript
const express = require('express');
const app = express();

app.post('/webhooks/invoice-collector', express.json(), async (req, res) => {
  try {
    const { type, user_id, invoice } = req.body;
    
    if (type === 'invoice') {
      // Process invoice asynchronously
      processInvoice(user_id, invoice).catch(console.error);
      
      // Return success immediately
      res.status(200).send('OK');
    } else {
      res.status(400).send('Unknown event type');
    }
  } catch (error) {
    console.error('Webhook error:', error);
    res.status(500).send('Error processing webhook');
  }
});

async function processInvoice(userId, invoice) {
  // Decode PDF data
  const pdfData = Buffer.from(invoice.data, 'base64');
  
  // Save to storage
  await saveToStorage(invoice.id, pdfData);
  
  // Store in database
  await db.invoices.create({
    user_id: userId,
    invoice_id: invoice.id,
    date: new Date(invoice.timestamp),
    amount: parseFloat(invoice.amount),
    metadata: invoice.metadata
  });
}
```

**Python/Flask:**

```python
from flask import Flask, request
import base64
import asyncio

app = Flask(__name__)

@app.route('/webhooks/invoice-collector', methods=['POST'])
def webhook():
    try:
        data = request.json
        
        if data['type'] == 'invoice':
            # Process asynchronously
            asyncio.create_task(process_invoice(
                data['user_id'], 
                data['invoice']
            ))
            
            return 'OK', 200
        else:
            return 'Unknown event type', 400
    except Exception as e:
        print(f'Webhook error: {e}')
        return 'Error', 500

async def process_invoice(user_id, invoice):
    # Decode PDF data
    pdf_data = base64.b64decode(invoice['data'])
    
    # Save and process
    await save_to_storage(invoice['id'], pdf_data)
    await save_to_database(user_id, invoice)
```

## Retry Logic

If your webhook endpoint returns an error or times out:

1. Invoice Collector retries with exponential backoff
2. Retries: 1 min, 5 min, 15 min, 1 hour
3. After 4 failed attempts, collection is paused
4. Manual intervention required to resume

!!! tip "Return 200 Quickly"
    Return HTTP 200 immediately and process the invoice asynchronously to avoid timeouts.

## Security

### HTTPS Only

Always use HTTPS for webhook endpoints in production:

```
https://api.yourapp.com/webhooks/invoice-collector  ✅
http://api.yourapp.com/webhooks/invoice-collector   ❌
```

### Signature Verification

Currently, Invoice Collector doesn't sign webhook payloads. Consider:

1. Using HTTPS with valid certificates
2. Restricting access by IP address
3. Implementing your own request validation

### IP Allowlisting

If possible, restrict webhook access to Invoice Collector's IP address.

## Testing Webhooks

### Test Endpoint

Test your callback configuration:

```bash
curl -X GET http://localhost:8080/api/v1/test/callback/invoice \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

This sends a test payload to your configured callback URL.

### Local Development

For local testing, use tools like:

- **ngrok** - Tunnel to localhost
- **RequestBin** - Inspect webhook payloads
- **Postman** - Mock webhook calls

**Using ngrok:**

```bash
# Start ngrok
ngrok http 3000

# Use the ngrok URL as callback
https://abc123.ngrok.io/webhooks/invoice-collector
```

## Handling Duplicates

Your webhook may receive the same invoice multiple times. Implement idempotency:

```javascript
async function processInvoice(userId, invoice) {
  // Check if already processed
  const existing = await db.invoices.findOne({
    invoice_id: invoice.id
  });
  
  if (existing) {
    console.log('Invoice already processed, skipping');
    return;
  }
  
  // Process invoice
  await saveInvoice(invoice);
}
```

## Monitoring

Monitor your webhook endpoint:

1. **Response Times** - Ensure < 30 seconds
2. **Error Rates** - Track 5xx responses
3. **Success Rate** - Monitor 200 responses
4. **Processing Time** - Track invoice processing duration

## Error Handling

### Your Endpoint Fails

If your endpoint consistently fails:

1. Check server logs for errors
2. Verify endpoint is accessible
3. Test with curl/Postman
4. Check firewall rules

### Data Issues

If invoice data is invalid:

```javascript
function validateInvoice(invoice) {
  if (!invoice.id || !invoice.data) {
    throw new Error('Invalid invoice data');
  }
  
  if (invoice.mimetype !== 'application/pdf') {
    console.warn('Unexpected mimetype:', invoice.mimetype);
  }
  
  return true;
}
```

## Best Practices

1. **Idempotency** - Handle duplicate deliveries
2. **Async Processing** - Return 200 quickly
3. **Error Logging** - Log all errors for debugging
4. **Monitoring** - Set up alerts for failures
5. **Validation** - Verify payload structure
6. **Retry Logic** - Implement your own retries for downstream failures

## Next Steps

- [API Endpoints Reference](endpoints.md)
- [Working with Invoices](../guides/invoices.md)
- [Quick Start Guide](../getting-started/quick-start.md)

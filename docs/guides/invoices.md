# Working with Invoices

Learn how to access, process, and manage collected invoices.

## Invoice Structure

Each invoice contains:

```json
{
  "id": "unique_invoice_id",
  "timestamp": 1609459200000,
  "amount": "99.99",
  "link": "https://supplier.com/invoice/123",
  "data": "base64_encoded_pdf_data",
  "mimetype": "application/pdf",
  "collected_timestamp": 1609459300000,
  "metadata": {
    "order_id": "123456",
    "supplier_name": "Amazon"
  }
}
```

### Fields

- **id** - Unique identifier for the invoice
- **timestamp** - Invoice date (milliseconds since epoch)
- **amount** - Total amount (when available)
- **link** - Original URL on supplier's website
- **data** - Base64-encoded document data
- **mimetype** - Document type (usually `application/pdf`)
- **collected_timestamp** - When the invoice was collected
- **metadata** - Additional supplier-specific information

## Receiving Invoices

Invoices are delivered via callbacks (webhooks) to your configured endpoint.

### Callback Configuration

Set the callback URL when creating or updating your customer:

```bash
curl -X POST http://localhost:8080/api/v1/customer \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Company",
    "callback": "https://your-server.com/invoices/callback"
  }'
```

### Callback Format

Invoice Collector sends a POST request to your callback URL:

```json
{
  "type": "invoice",
  "user_id": "user123",
  "credential_id": "cred456",
  "collector": "amazon",
  "invoice": {
    "id": "inv789",
    "timestamp": 1609459200000,
    "amount": "99.99",
    "data": "JVBERi0xLjQK...",
    "mimetype": "application/pdf",
    "metadata": {...}
  }
}
```

## Processing Invoices

### Saving Invoice Files

Decode and save the invoice document:

```python
import base64

# Decode base64 data
invoice_data = base64.b64decode(invoice['data'])

# Save to file
with open(f"invoice_{invoice['id']}.pdf", 'wb') as f:
    f.write(invoice_data)
```

```javascript
// Node.js
const fs = require('fs');

// Decode base64 data
const invoiceData = Buffer.from(invoice.data, 'base64');

// Save to file
fs.writeFileSync(`invoice_${invoice.id}.pdf`, invoiceData);
```

### Extracting Information

Use the metadata for additional information:

```javascript
const {
  order_id,
  supplier_name,
  customer_name
} = invoice.metadata;

// Store in database
await db.invoices.insert({
  invoice_id: invoice.id,
  date: new Date(invoice.timestamp),
  amount: parseFloat(invoice.amount),
  supplier: supplier_name,
  order_id: order_id
});
```

## Viewing Invoices in UI

Access the web UI to view collected invoices:

```
http://localhost:8080/api/v1/ui?token=USER_TOKEN
```

The UI displays:

- List of all credentials
- Collection status for each
- Number of invoices collected
- Last collection date

## Invoice Metadata

Different collectors provide different metadata:

### Amazon
- `order_id` - Order number
- `order_date` - Order placement date
- `invoice_number` - Invoice number

### OVH
- `bill_id` - Bill identifier
- `service` - Service name
- `period` - Billing period

### Leclerc
- `store_name` - Store location
- `transaction_id` - Transaction number

!!! tip
    Check collector-specific documentation for available metadata fields.

## Duplicate Detection

Invoice Collector automatically handles duplicates:

- Each invoice has a unique ID
- Re-collection won't create duplicates
- Previously collected invoices are skipped

## Error Handling

### Callback Failures

If your callback endpoint fails:

1. Invoice Collector retries with exponential backoff
2. After multiple failures, collection is paused
3. Review credential status for error details

Ensure your endpoint:

- Returns HTTP 200 for successful processing
- Responds within 30 seconds
- Handles errors gracefully

### Invalid Documents

Some invoices may fail to download:

- Check `data` field is not null
- Verify `mimetype` is as expected
- Review error logs in credential status

## Best Practices

1. **Idempotent Processing** - Handle duplicate callbacks gracefully
2. **Async Processing** - Return 200 quickly, process in background
3. **Error Recovery** - Implement retry logic for failed processing
4. **Data Validation** - Verify invoice data before storing
5. **Secure Storage** - Store invoice documents securely

## Integration Examples

### Accounting Software

```javascript
// Send to accounting system
async function processInvoice(invoice) {
  const accountingData = {
    date: new Date(invoice.timestamp),
    amount: parseFloat(invoice.amount),
    supplier: invoice.metadata.supplier_name,
    document: invoice.data
  };
  
  await accountingSoftware.createBill(accountingData);
}
```

### Document Management

```python
# Store in document management system
def store_invoice(invoice):
    # Upload to DMS
    document_id = dms.upload(
        data=base64.b64decode(invoice['data']),
        filename=f"invoice_{invoice['id']}.pdf",
        metadata={
            'date': invoice['timestamp'],
            'amount': invoice['amount'],
            'supplier': invoice['metadata']['supplier_name']
        }
    )
    
    # Update database reference
    db.invoices.update(
        invoice['id'],
        {'document_id': document_id}
    )
```

## Monitoring

Track invoice collection:

```bash
# Get customer statistics
curl -X GET http://localhost:8080/api/v1/customer/stats \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN"
```

Returns:

- Total invoices collected
- Collections per collector
- Recent activity

## Next Steps

- [Configure webhooks](../api/webhooks.md)
- [API reference](../api/endpoints.md)
- [Troubleshooting guide](overview.md)

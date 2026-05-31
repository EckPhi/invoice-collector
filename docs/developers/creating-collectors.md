# Creating Collectors

Learn how to create a new collector for Invoice Collector.

## Overview

Creating a collector involves:

1. Understanding the supplier's website/API
2. Creating the collector class
3. Implementing the collection logic
4. Testing the collector
5. Submitting for review

## Before You Start

### Research the Supplier

- Can you access invoices programmatically?
- Is there an API available?
- What authentication is required?
- Are there anti-bot protections?
- What format are the invoices?

### Choose Collector Type

- **Web Collector** - Website scraping with Puppeteer
- **API Collector** - Direct API integration
- **Email Collector** - Extract from emails

## Creating a Web Collector

### Step 1: Create Directory

```bash
mkdir -p src/collectors/core/supplier_name
cd src/collectors/core/supplier_name
```

### Step 2: Create Configuration

Create `supplier_name.ts`:

```typescript
import { WebCollector } from '../webCollector';
import { Config, CollectorState, CollectorType } from '../abstractCollector';

export const config: Config = {
  id: 'supplier_name',
  name: 'Supplier Name',
  description: 'Collect invoices from Supplier',
  version: '1.0.0',
  website: 'https://supplier.com',
  logo: 'https://supplier.com/logo.png',
  type: CollectorType.WEB,
  state: CollectorState.ACTIVE,
  params: {
    email: {
      type: 'text',
      name: 'Email',
      placeholder: 'your-email@example.com',
      mandatory: true
    },
    password: {
      type: 'password',
      name: 'Password',
      placeholder: '••••••••',
      mandatory: true
    }
  }
};
```

### Step 3: Implement Collector

```typescript
export class SupplierCollector extends WebCollector {
  constructor() {
    super(config);
  }

  async collect_new_invoices(
    state: State,
    twofa_promise: TwofaPromise,
    secret: Secret,
    download_from_timestamp: number,
    previousInvoices: any[],
    location: Location | null
  ): Promise<CompleteInvoice[]> {
    const invoices: CompleteInvoice[] = [];
    
    // Launch browser
    const page = await this.launch_browser(location);
    
    try {
      // Login
      await this.login(page, secret);
      
      // Handle 2FA if needed
      if (await this.check_2fa_required(page)) {
        const code = await twofa_promise.wait();
        await this.submit_2fa(page, code);
      }
      
      // Navigate to invoices
      await page.goto('https://supplier.com/invoices');
      
      // Scrape invoice list
      const invoiceLinks = await this.scrape_invoice_list(
        page,
        download_from_timestamp
      );
      
      // Download each invoice
      for (const link of invoiceLinks) {
        const invoice = await this.download_invoice(page, link);
        invoices.push(invoice);
      }
      
    } finally {
      await page.close();
    }
    
    return invoices;
  }
  
  private async login(page: Page, secret: Secret) {
    await page.goto('https://supplier.com/login');
    await page.type('#email', secret.params.email);
    await page.type('#password', secret.params.password);
    await page.click('button[type="submit"]');
    await page.waitForNavigation();
  }
  
  private async check_2fa_required(page: Page): Promise<boolean> {
    return page.$('#2fa-code') !== null;
  }
  
  private async submit_2fa(page: Page, code: string) {
    await page.type('#2fa-code', code);
    await page.click('#submit-2fa');
    await page.waitForNavigation();
  }
  
  private async scrape_invoice_list(
    page: Page,
    from_timestamp: number
  ): Promise<string[]> {
    const links: string[] = [];
    
    // Navigate through pages
    let hasMore = true;
    while (hasMore) {
      const pageLinks = await page.$$eval('.invoice-link', els =>
        els.map(el => el.getAttribute('href'))
      );
      
      links.push(...pageLinks);
      
      // Check for next page
      const nextButton = await page.$('.next-page');
      if (nextButton) {
        await nextButton.click();
        await page.waitForNavigation();
      } else {
        hasMore = false;
      }
    }
    
    return links;
  }
  
  private async download_invoice(
    page: Page,
    link: string
  ): Promise<CompleteInvoice> {
    await page.goto(link);
    
    // Extract metadata
    const timestamp = await page.$eval(
      '.invoice-date',
      el => new Date(el.textContent).getTime()
    );
    const amount = await page.$eval(
      '.invoice-amount',
      el => el.textContent.replace(/[^0-9.]/g, '')
    );
    
    // Download PDF
    const downloadLink = await page.$eval(
      '.download-pdf',
      el => el.getAttribute('href')
    );
    
    const response = await page.goto(downloadLink);
    const buffer = await response.buffer();
    const data = buffer.toString('base64');
    
    return {
      id: `supplier_${timestamp}`,
      timestamp,
      amount,
      link,
      data,
      mimetype: 'application/pdf',
      collected_timestamp: Date.now(),
      metadata: {}
    };
  }
}
```

### Step 4: Create Selectors (Optional)

For complex selectors, create `selectors.ts`:

```typescript
export const SELECTORS = {
  LOGIN: {
    EMAIL: '#email',
    PASSWORD: '#password',
    SUBMIT: 'button[type="submit"]'
  },
  INVOICES: {
    LIST: '.invoice-link',
    DATE: '.invoice-date',
    AMOUNT: '.invoice-amount',
    DOWNLOAD: '.download-pdf'
  },
  PAGINATION: {
    NEXT: '.next-page',
    PREV: '.prev-page'
  }
};
```

## Creating an API Collector

### Step 1: Create Configuration

```typescript
import { APICollector } from '../apiCollector';

export const config: Config = {
  id: 'api_supplier',
  name: 'API Supplier',
  description: 'Collect invoices via API',
  version: '1.0.0',
  website: 'https://api.supplier.com',
  logo: 'https://supplier.com/logo.png',
  type: CollectorType.API,
  state: CollectorState.ACTIVE,
  params: {
    api_key: {
      type: 'text',
      name: 'API Key',
      placeholder: 'sk_live_...',
      mandatory: true
    }
  }
};
```

### Step 2: Implement Collector

```typescript
import axios from 'axios';

export class APISupplierCollector extends APICollector {
  constructor() {
    super(config);
  }

  async collect_new_invoices(
    state: State,
    twofa_promise: TwofaPromise,
    secret: Secret,
    download_from_timestamp: number,
    previousInvoices: any[],
    location: Location | null
  ): Promise<CompleteInvoice[]> {
    const invoices: CompleteInvoice[] = [];
    
    // Create API client
    const client = axios.create({
      baseURL: 'https://api.supplier.com/v1',
      headers: {
        'Authorization': `Bearer ${secret.params.api_key}`
      }
    });
    
    // List invoices
    const response = await client.get('/invoices', {
      params: {
        from_date: new Date(download_from_timestamp).toISOString()
      }
    });
    
    // Download each invoice
    for (const invoice of response.data.invoices) {
      const pdfResponse = await client.get(
        `/invoices/${invoice.id}/pdf`,
        { responseType: 'arraybuffer' }
      );
      
      invoices.push({
        id: invoice.id,
        timestamp: new Date(invoice.date).getTime(),
        amount: invoice.amount.toString(),
        link: invoice.pdf_url,
        data: Buffer.from(pdfResponse.data).toString('base64'),
        mimetype: 'application/pdf',
        collected_timestamp: Date.now(),
        metadata: {
          invoice_number: invoice.number
        }
      });
    }
    
    return invoices;
  }
}
```

## Best Practices

### Error Handling

Always handle errors gracefully:

```typescript
try {
  await this.login(page, secret);
} catch (error) {
  if (error.message.includes('Invalid credentials')) {
    throw new Error('INVALID_CREDENTIALS');
  }
  throw error;
}
```

### Waiting for Elements

Use proper waiting strategies:

```typescript
// Wait for navigation
await page.waitForNavigation({ waitUntil: 'networkidle0' });

// Wait for selector
await page.waitForSelector('.invoice-list', { timeout: 10000 });

// Wait for custom condition
await page.waitForFunction(
  () => document.querySelectorAll('.invoice').length > 0
);
```

### Anti-Bot Evasion

For sites with anti-bot protection:

```typescript
// Use stealth mode
const page = await this.launch_browser(location, {
  stealth: true
});

// Add delays
await page.waitForTimeout(2000);

// Use ghost cursor
import { createCursor } from 'ghost-cursor';
const cursor = createCursor(page);
await cursor.click('#button');
```

### Incremental Collection

Only collect new invoices:

```typescript
private async scrape_invoice_list(
  page: Page,
  from_timestamp: number,
  previousInvoices: any[]
): Promise<string[]> {
  const links: string[] = [];
  const previousIds = new Set(previousInvoices.map(i => i.id));
  
  // Continue until we find a previous invoice
  let foundPrevious = false;
  while (!foundPrevious) {
    const invoices = await this.get_page_invoices(page);
    
    for (const invoice of invoices) {
      if (previousIds.has(invoice.id)) {
        foundPrevious = true;
        break;
      }
      if (invoice.timestamp >= from_timestamp) {
        links.push(invoice.link);
      }
    }
    
    if (!foundPrevious && await this.has_next_page(page)) {
      await this.go_to_next_page(page);
    } else {
      break;
    }
  }
  
  return links;
}
```

## Testing

### Manual Testing

```bash
npm run test.manual supplier_name email@example.com password123
```

### Automated Testing

Create `test/auto.spec.ts` entry:

```typescript
describe('Supplier Collector', () => {
  it('should collect invoices', async () => {
    const collector = new SupplierCollector();
    const invoices = await collector.collect_new_invoices(
      mockState,
      mockTwoFA,
      mockSecret,
      0,
      [],
      null
    );
    
    expect(invoices.length).toBeGreaterThan(0);
    expect(invoices[0].data).toBeDefined();
  });
});
```

## Submitting Your Collector

1. Test thoroughly
2. Document any special requirements
3. Create a pull request
4. Wait for review
5. Address feedback

### Pull Request Template

```markdown
## New Collector: Supplier Name

### Description
Brief description of the supplier and collector.

### Type
- [ ] Web Collector
- [ ] API Collector
- [ ] Email Collector

### Testing
- [ ] Manual testing completed
- [ ] Automated tests added
- [ ] Works with 2FA
- [ ] Handles errors gracefully

### Special Requirements
- Requires proxy for region restrictions
- Needs specific Chrome flags
- Rate limited to X requests per minute

### Checklist
- [ ] Configuration complete
- [ ] Collector implemented
- [ ] Tests passing
- [ ] Documentation updated
```

## Troubleshooting

### Element Not Found

Use more robust selectors:

```typescript
// Instead of
await page.waitForSelector('.button');

// Use
await page.waitForSelector('[data-testid="submit"]', {
  visible: true,
  timeout: 10000
});
```

### Timeout Errors

Increase timeouts for slow sites:

```typescript
await page.goto(url, {
  waitUntil: 'networkidle0',
  timeout: 60000  // 60 seconds
});
```

### Anti-Bot Detection

- Use proxy servers
- Add random delays
- Rotate user agents
- Use ghost cursor for mouse movements

## Next Steps

- [Testing guide](testing.md)
- [Contributing guidelines](contributing.md)
- [Architecture overview](architecture.md)

# Testing Guide

Learn how to test Invoice Collector and your collectors.

## Test Types

### 1. Manual Tests

Interactive testing with real credentials.

**Command:**

```bash
npm run test.manual <collector_id> [param1] [param2] ...
```

**Example:**

```bash
npm run test.manual amazon user@example.com mypassword
```

**Features:**

- Interactive prompts
- Real browser window (visible)
- Step-by-step execution
- 2FA code input

### 2. Automated Tests

Automated testing with Jest.

**Command:**

```bash
npm run test.auto <collector_id>
```

**Example:**

```bash
npm run test.auto amazon
```

**Requirements:**

- Test credentials in environment
- Mock data for assertions
- Automated 2FA handling (if possible)

### 3. Unit Tests

Test individual components.

**Command:**

```bash
npm test
```

## Manual Testing

### Basic Test

Test with credentials:

```bash
npm run test.manual amazon email@example.com password123
```

The test will:

1. Launch browser (visible)
2. Login with credentials
3. Navigate to invoices
4. Display found invoices
5. Ask to download them

### Interactive Prompts

The manual test provides interactive prompts:

```
Found 5 invoices:
1. Invoice from 2024-01-15 - $99.99
2. Invoice from 2024-01-30 - $149.99
...

Download all invoices? (y/n):
```

### Viewing Browser

Manual tests run with visible browser:

- Watch the automation
- Debug selector issues
- Understand the flow
- Capture screenshots

### Custom Parameters

Collectors with custom parameters:

```bash
npm run test.manual leclerc card_number password
npm run test.manual ovh customer_id password
```

## Automated Testing

### Test Structure

Create test file `test/collectors/supplier.spec.ts`:

```typescript
import { SupplierCollector } from '../../src/collectors/core/supplier/supplier';

describe('Supplier Collector', () => {
  let collector: SupplierCollector;
  
  beforeEach(() => {
    collector = new SupplierCollector();
  });

  it('should have correct configuration', () => {
    expect(collector.config.id).toBe('supplier');
    expect(collector.config.type).toBe('web');
  });

  it('should collect invoices', async () => {
    const invoices = await collector.collect_new_invoices(
      mockState(),
      mockTwoFA(),
      mockSecret(),
      Date.now() - 365 * 24 * 60 * 60 * 1000,
      [],
      null
    );
    
    expect(invoices).toBeInstanceOf(Array);
    expect(invoices.length).toBeGreaterThan(0);
    
    const invoice = invoices[0];
    expect(invoice.id).toBeDefined();
    expect(invoice.timestamp).toBeGreaterThan(0);
    expect(invoice.data).toBeDefined();
    expect(invoice.mimetype).toBe('application/pdf');
  });

  it('should handle authentication errors', async () => {
    const badSecret = mockSecret({ password: 'wrong' });
    
    await expect(
      collector.collect_new_invoices(
        mockState(),
        mockTwoFA(),
        badSecret,
        0,
        [],
        null
      )
    ).rejects.toThrow('INVALID_CREDENTIALS');
  });
});
```

### Mock Helpers

Create mock utilities:

```typescript
export function mockState(): State {
  return {
    set_2fa_required: jest.fn(),
    set_status: jest.fn(),
    set_error: jest.fn()
  } as any;
}

export function mockTwoFA(code: string = '123456'): TwofaPromise {
  return {
    wait: jest.fn().mockResolvedValue(code)
  } as any;
}

export function mockSecret(params: any = {}): Secret {
  return {
    params: {
      email: 'test@example.com',
      password: 'password123',
      ...params
    }
  } as any;
}
```

### Running Tests

```bash
# Run all tests
npm test

# Run specific collector
npm run test.auto amazon

# Run with coverage
npm test -- --coverage

# Run in watch mode
npm test -- --watch
```

## Test Environment

### Environment Variables

Set test credentials in `.env.test`:

```env
# Amazon Test Credentials
TEST_AMAZON_EMAIL=test@example.com
TEST_AMAZON_PASSWORD=testpass123

# OVH Test Credentials
TEST_OVH_CUSTOMER_ID=ab12345
TEST_OVH_PASSWORD=testpass123
```

### Test Database

Use a separate test database:

```env
DATABASE_URI=mongodb://localhost:27017
DATABASE_MONGODB_NAME=test
```

### Cleaning Test Data

Clean up after tests:

```typescript
afterEach(async () => {
  await database.clearTestData();
});

afterAll(async () => {
  await database.disconnect();
});
```

## Testing Best Practices

### 1. Test Isolation

Each test should be independent:

```typescript
beforeEach(() => {
  // Fresh instance for each test
  collector = new SupplierCollector();
});
```

### 2. Mock External Services

Mock external dependencies:

```typescript
jest.mock('axios');

const mockAxios = axios as jest.Mocked<typeof axios>;
mockAxios.get.mockResolvedValue({ data: mockInvoices });
```

### 3. Test Edge Cases

```typescript
it('should handle empty invoice list', async () => {
  mockAxios.get.mockResolvedValue({ data: [] });
  const invoices = await collector.collect_new_invoices(...);
  expect(invoices).toEqual([]);
});

it('should handle network errors', async () => {
  mockAxios.get.mockRejectedValue(new Error('Network error'));
  await expect(collector.collect_new_invoices(...)).rejects.toThrow();
});
```

### 4. Verify Assertions

Always verify important properties:

```typescript
expect(invoice.id).toBeTruthy();
expect(invoice.timestamp).toBeGreaterThan(0);
expect(invoice.data).toBeTruthy();
expect(invoice.mimetype).toMatch(/pdf|html/);
```

## Debugging Tests

### VS Code Debugging

Add launch configuration:

```json
{
  "type": "node",
  "request": "launch",
  "name": "Jest Debug",
  "program": "${workspaceFolder}/node_modules/jest/bin/jest",
  "args": ["--runInBand", "--no-cache"],
  "console": "integratedTerminal"
}
```

### Browser Debugging

For manual tests:

```typescript
const browser = await puppeteer.launch({
  headless: false,  // See browser
  devtools: true,   // Open DevTools
  slowMo: 100      // Slow down actions
});
```

### Screenshots

Capture screenshots on failure:

```typescript
try {
  await page.click('#button');
} catch (error) {
  await page.screenshot({ path: 'error.png' });
  throw error;
}
```

### Verbose Logging

Enable detailed logs:

```typescript
console.log('Navigating to:', url);
console.log('Found invoices:', invoices.length);
console.log('Downloading invoice:', invoice.id);
```

## Continuous Integration

Tests run automatically on:

- Pull requests
- Commits to main branch
- Scheduled runs

### GitHub Actions

Example workflow:

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
        env:
          TEST_AMAZON_EMAIL: ${{ secrets.TEST_AMAZON_EMAIL }}
          TEST_AMAZON_PASSWORD: ${{ secrets.TEST_AMAZON_PASSWORD }}
```

## Testing Checklist

Before submitting a collector:

- [ ] Manual test passes with valid credentials
- [ ] Manual test fails gracefully with invalid credentials
- [ ] Automated test passes
- [ ] Handles 2FA correctly
- [ ] Returns correct invoice structure
- [ ] Downloads PDF data successfully
- [ ] Extracts metadata properly
- [ ] No hardcoded credentials
- [ ] Error messages are clear
- [ ] Edge cases handled

## Common Test Issues

### Timeout Errors

Increase timeout for slow sites:

```typescript
it('should collect invoices', async () => {
  // ...
}, 60000); // 60 second timeout
```

### Flaky Tests

Add retries for flaky tests:

```typescript
jest.retryTimes(3);
```

### Selector Changes

Keep selectors up to date:

```typescript
// Use data attributes when possible
await page.waitForSelector('[data-testid="invoice-list"]');
```

## Next Steps

- [Contributing guidelines](contributing.md)
- [Creating collectors](creating-collectors.md)
- [Architecture overview](architecture.md)

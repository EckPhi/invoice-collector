# Invoice Collector - AI Coding Assistant Guide

## Architecture Overview

This is a **TypeScript-based web scraping system** that automatically collects invoices from various supplier portals. The system operates as a multi-tenant Docker application with these core components:

- **Collector Framework**: Plugin architecture with 200+ site-specific scrapers (`src/collectors/`)
- **Web Driver Layer**: Puppeteer-based browser automation (`src/driver/`)
- **Express API**: REST endpoints for UI and webhook integration (`src/index.ts`, `src/server.ts`)
- **Task Scheduler**: Cron-based automated collection (`src/collect/collectTask.ts`)
- **Multi-database Support**: MongoDB primary, extensible via factory pattern (`src/database/`)

## Collector Architecture Patterns

### Collector Hierarchy (Critical)
```
AbstractCollector → V1Collector/V2Collector → WebCollector (V2)/ApiCollector → Specific implementations
```

- **V1Collectors (Legacy)**: Separate `collect()` → `download()` phases. Legacy browser class lives in `src/collectors/webCollector.ts`.
- **V2Collectors (Modern)**: Combined collect+download in a single pass. The modern browser class is named `WebCollector` but lives in `src/collectors/web2Collector.ts`.
- **WebCollector (V2)**: Browser-based scraping (login → collect → download in one flow)
- **ApiCollector**: REST API integration (no browser needed)
- **SketchCollector**: Development templates in `src/collectors/sketch/`

### Core Collector Methods
Every collector must implement:
- `login(driver, params)`: Authentication logic
- `collect(driver, params)`: Extract invoice metadata
- `download(driver, invoice)`: Fetch invoice documents
- `data(driver, params, element)`: Parse invoice data (V2/Web2-only)

## Key Development Workflows

### Testing Collectors
```bash
# Manual testing with prompts
npm run test.manual <collector_id> [params...]

# Automated testing
npm run test.auto <collector_id>

# Advanced tests
npm run test.fingerprints
npm run test.antibot
npm run test.cicd.locales

# Debug container with breakpoints
docker-compose -f docker-compose-debug.yml up --build
```

### Creating New Collectors
1. Copy template from `src/collectors/sketch/`
2. Extend appropriate base class (WebCollector/ApiCollector/Web2Collector)
3. Implement required methods with proper error handling
4. Use `CollectorState.DEVELOPMENT` until production-ready
5. Test with `npm run test.manual <new_collector_id>`

## Critical Code Patterns

### Error Handling Hierarchy
```typescript
CollectorError → AuthenticationError/LoggableError/MaintenanceError/etc.
```
Always include URL, screenshot, and source code context for web errors.

### State Management
`State` class tracks collection progress through defined phases:
`PREPARING → CONNECTING → AUTHENTICATING → TWOFA → COLLECTING → DOWNLOADING → COMPLETED`

### Invoice Data Structure
```typescript
Invoice → DownloadedInvoice → CompleteInvoice
```
- `Invoice`: Metadata (id, timestamp, amount, link)
- `DownloadedInvoice`: + documents[] (base64 PDFs)
- `CompleteInvoice`: + final data, mimetype, collected_timestamp

### Driver Patterns
Use `Driver` class for all browser interactions:
- `driver.goto(url)` with wait conditions
- `driver.waitFor(selector)` with timeouts
- `driver.click()`, `driver.type()` with delays
- Always handle `ElementNotFoundError`

Note: The code imports both legacy and modern browser collectors with aliasing in `src/driver/driver.ts`:
`import { WebCollector as OldWebCollector } from '../collectors/webCollector';`
`import { WebCollector } from '../collectors/web2Collector';`

## Project-Specific Conventions

### File Organization
- Core production collectors: `src/collectors/core/`
- Development sketches: `src/collectors/sketch/`
- Collector configs define: params, entryUrl, loginUrl, state
- One collector per directory with main file + selectors

### Configuration Patterns
```typescript
static CONFIG = {
    id: "unique_name",
    type: CollectorType.WEB,
    params: { email: {mandatory: true}, password: {mandatory: true} },
    loginUrl: "...",
    entryUrl: "...",
    state: CollectorState.PRODUCTION
}
```

### Database Abstraction
Use `DatabaseFactory.getDatabase()` - never directly import MongoDB. Enables swapping database implementations.

### Scheduler Defaults
Cron-based collection is configured in `src/collect/collectTask.ts`:
- `DEFAULT_CRON_TIME = '* * * * *'` (every minute)
- `DEFAULT_TIMEZONE = 'Europe/Paris'`

## Integration Points

### External Systems
- **Secret Management**: Bitwarden integration via `SecretManagerFactory`
- **Proxy Support**: Multiple providers via `ProxyFactory`
- **Registry Server**: Collector metadata and stats reporting
- **Webhook Callbacks**: Invoice delivery via `CallbackHandler`

### Docker Environment
- Chrome/Chromium required for WebDrivers
- XVFB for headless browser support
- Optionally mount volumes for persistence: `/usr/app/media/`, `/usr/app/log/`
- Environment variables in `.env` file (see docs/deployment/environment-variables.md)

Website quick links for operators:
- Installation: https://invoice-collector.com/docs/getting-started/self-hosted/installation
- Configuration: https://invoice-collector.com/docs/getting-started/self-hosted/configuration

Docs references and fallbacks:
- Some developer pages on the public site may be moving. If a link 404s, use the local sources:
    - `docs/developers/architecture.md`
    - `docs/developers/creating-collectors.md`
    - `docs/developers/testing.md`
    - `docs/deployment/environment-variables.md`
    - `docs/guides/*.md` and `docs/api/*.md`

## Common Debugging

- Check collector state with `State` progress tracking
- Browser issues: Enable headful mode, capture screenshots
- Collection failures: Review LoggableError with URL/source context
- 2FA handling: Use `TwofaPromise` for user interaction
- PDF merging: Multiple documents combined via `utils.mergePdfDocuments()`

Focus on the collector pattern when implementing new scrapers - this is the core abstraction that makes the system extensible.
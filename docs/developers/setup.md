# Development Setup

Set up your local development environment for contributing to Invoice Collector.

## Prerequisites

- **Node.js** - Version 18 or higher
- **npm** - Version 8 or higher
- **Google Chrome** - For browser automation
- **MongoDB** - For local database
- **Git** - For version control

## Clone the Repository

```bash
git clone https://github.com/invoice-collector/invoice-collector.git
cd invoice-collector
```

## Install Dependencies

```bash
npm install
```

This installs all required dependencies including:

- TypeScript
- Puppeteer
- Express
- MongoDB driver
- And more...

## Environment Configuration

Create a `.env` file in the root directory:

```bash
cp .env.example .env  # If example exists
# Or create manually
touch .env
```

Add the following environment variables:

```env
# Server
PORT=8080
ENV=debug
FRONTEND=http://localhost:8080

# Database
DATABASE_URI=mongodb://localhost:27017
DATABASE_MONGODB_NAME=dev

# Secret Manager
SECRET_MANAGER_TYPE=bitwarden
SECRET_MANAGER_BITWARDEN_API_URI=https://vault.bitwarden.eu/api
SECRET_MANAGER_BITWARDEN_IDENTITY_URI=https://vault.bitwarden.eu/identity
SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=your_token
SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=your_org_id
SECRET_MANAGER_BITWARDEN_PROJECT_ID=your_project_id

# Registry
REGISTRY_SERVER_ENDPOINT=https://registry.invoice-collector.com

# Optional
DISABLE_VERIFICATION_CODE=true
```

See the [environment variables guide](../deployment/environment-variables.md) for details.

## Start MongoDB

**Using Docker:**

```bash
docker run -d \
  --name mongodb-dev \
  -p 27017:27017 \
  mongo:8.0.3
```

**Or install locally** following MongoDB's [installation guide](https://docs.mongodb.com/manual/installation/).

## Run the Application

### Development Mode

Start the server with hot-reload:

```bash
npm run start
```

The server will start on `http://localhost:8080`.

### Debug Mode

Start with debugging enabled:

```bash
npm run debug
```

Connect your debugger to port 9229.

### Docker Development

Use the debug docker-compose file:

```bash
docker-compose -f docker-compose-debug.yml up --build
```

## Project Structure

```
invoice-collector/
├── src/
│   ├── index.ts              # Entry point
│   ├── server.ts             # Express server
│   ├── collectors/           # Collector implementations
│   │   ├── core/            # Official collectors
│   │   ├── community/       # Community collectors
│   │   └── sketch/          # Experimental collectors
│   ├── database/            # Database layer
│   ├── secret_manager/      # Secret management
│   ├── driver/              # Browser driver
│   ├── collect/             # Collection logic
│   └── model/               # Data models
├── test/                    # Tests
├── views/                   # EJS templates
├── locales/                 # Translations
├── docker-compose.yml       # Production compose
├── docker-compose-debug.yml # Debug compose
└── package.json             # Dependencies
```

## Running Tests

### Manual Tests

Test a specific collector:

```bash
npm run test.manual <collector_id>
```

With parameters:

```bash
npm run test.manual amazon user@example.com password123
```

### Automated Tests

Run the test suite:

```bash
npm run test.auto <collector_id>
```

### All Tests

Run all tests:

```bash
npm test
```

## Code Style

The project uses TypeScript with strict mode enabled.

### Linting

```bash
npm run lint
```

### Formatting

Follow the existing code style:

- 2 spaces for indentation
- Single quotes for strings
- Semicolons required
- Trailing commas in multiline

## Making Changes

### 1. Create a Branch

```bash
git checkout -b feature/your-feature-name
```

### 2. Make Your Changes

Edit the code following the style guide.

### 3. Test Your Changes

```bash
npm run test.manual <collector_id>
```

### 4. Commit

```bash
git add .
git commit -m "Description of changes"
```

### 5. Push and Create PR

```bash
git push origin feature/your-feature-name
```

Then create a Pull Request on GitHub.

## Debugging

### VS Code Configuration

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Server",
      "runtimeArgs": ["-r", "ts-node/register"],
      "args": ["${workspaceFolder}/src/index.ts"],
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Test",
      "runtimeArgs": ["-r", "ts-node/register"],
      "args": [
        "${workspaceFolder}/test/manual.ts",
        "amazon"
      ]
    }
  ]
}
```

### Browser Debugging

Set headless mode to false in collector code:

```typescript
const browser = await puppeteer.launch({
  headless: false,  // See the browser
  devtools: true,   // Open DevTools
});
```

## Common Issues

### Port Already in Use

Change the port in `.env`:

```env
PORT=8081
```

### Chrome Not Found

Install Google Chrome or set the path:

```env
CHROME_PATH=/path/to/chrome
```

### MongoDB Connection Failed

Ensure MongoDB is running:

```bash
docker ps  # Check if container is running
```

### TypeScript Errors

Rebuild TypeScript:

```bash
npm run build
```

## Next Steps

- [Understanding the architecture](architecture.md)
- [Creating a new collector](creating-collectors.md)
- [Running tests](testing.md)
- [Contributing guidelines](contributing.md)

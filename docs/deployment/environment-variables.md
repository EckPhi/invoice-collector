# Environment Variables

Complete reference for all Invoice Collector environment variables.

## Required Variables

### Server Configuration

```env
PORT=8080
```

Port on which the server listens.

- **Type:** Integer
- **Default:** 8080
- **Example:** 8080, 3000, 8000

```env
FRONTEND=http://localhost:8080
```

Frontend URL for generating UI links.

- **Type:** URL
- **Required:** Yes
- **Example:** `https://invoices.yourdomain.com`

### Database Configuration

```env
DATABASE_URI=mongodb://mongodb:27017
```

MongoDB connection string.

- **Type:** MongoDB URI
- **Required:** Yes
- **Format:** `mongodb://[username:password@]host[:port]`
- **Example:** `mongodb://localhost:27017`
- **Production:** `mongodb://user:pass@mongodb:27017`

```env
DATABASE_MONGODB_NAME=prod
```

MongoDB database name.

- **Type:** String
- **Required:** Yes
- **Default:** prod
- **Example:** `production`, `invoices`, `main`

### Secret Manager Configuration

```env
SECRET_MANAGER_TYPE=bitwarden
```

Type of secret manager to use.

- **Type:** String
- **Required:** Yes
- **Values:** `bitwarden` (only supported value)

```env
SECRET_MANAGER_BITWARDEN_API_URI=https://vault.bitwarden.eu/api
```

Bitwarden API endpoint.

- **Type:** URL
- **Required:** Yes
- **EU:** `https://vault.bitwarden.eu/api`
- **US:** `https://vault.bitwarden.com/api`

```env
SECRET_MANAGER_BITWARDEN_IDENTITY_URI=https://vault.bitwarden.eu/identity
```

Bitwarden identity endpoint.

- **Type:** URL
- **Required:** Yes
- **EU:** `https://vault.bitwarden.eu/identity`
- **US:** `https://vault.bitwarden.com/identity`

```env
SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=your_token_here
```

Bitwarden access token.

- **Type:** String
- **Required:** Yes
- **Security:** Keep secret, never commit to version control

```env
SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=your_org_id
```

Bitwarden organization ID.

- **Type:** UUID
- **Required:** Yes

```env
SECRET_MANAGER_BITWARDEN_PROJECT_ID=your_project_id
```

Bitwarden project ID.

- **Type:** UUID
- **Required:** Yes

### Registry Configuration

```env
REGISTRY_SERVER_ENDPOINT=https://registry.invoice-collector.com
```

Registry server URL for collector metadata.

- **Type:** URL
- **Required:** Yes
- **Default:** `https://registry.invoice-collector.com`

## Optional Variables

### Environment Mode

```env
ENV=prod
```

Application environment mode.

- **Type:** String
- **Default:** `prod`
- **Values:** `prod`, `debug`, `dev`
- **Debug mode:** Shows detailed error messages

### Authentication

```env
DISABLE_VERIFICATION_CODE=true
```

Disable email verification codes.

- **Type:** Boolean
- **Default:** `false`
- **Values:** `true`, `false`
- **Development:** Set to `true`
- **Production:** Set to `false`

### Proxy Configuration

```env
PROXY_TYPE=brightdata
```

Type of proxy service.

- **Type:** String
- **Optional:** Yes
- **Values:** `brightdata`, `residential`, `datacenter`

```env
PROXY_HOST=proxy.example.com
```

Proxy server hostname.

- **Type:** String
- **Optional:** Yes

```env
PROXY_PORT=8000
```

Proxy server port.

- **Type:** Integer
- **Optional:** Yes

```env
PROXY_USERNAME=user
```

Proxy authentication username.

- **Type:** String
- **Optional:** Yes

```env
PROXY_PASSWORD=pass
```

Proxy authentication password.

- **Type:** String
- **Optional:** Yes

### AI Configuration

```env
MISTRAL_API_KEY=your_api_key
```

Mistral AI API key for AI-enhanced collection.

- **Type:** String
- **Optional:** Yes
- **Used by:** AI-enhanced collectors

### Chrome Configuration

```env
CHROME_PATH=/path/to/chrome
```

Custom Chrome executable path.

- **Type:** Path
- **Optional:** Yes
- **Auto-detected:** Usually not needed

```env
CHROME_ARGS=--no-sandbox,--disable-dev-shm-usage
```

Additional Chrome command-line arguments.

- **Type:** Comma-separated string
- **Optional:** Yes
- **Common:** `--no-sandbox` for Docker

## Environment Files

### .env File

Create a `.env` file in the project root:

```env
# Server
PORT=8080
ENV=prod
FRONTEND=https://invoices.yourdomain.com

# Database
DATABASE_URI=mongodb://mongodb:27017
DATABASE_MONGODB_NAME=prod

# Secret Manager
SECRET_MANAGER_TYPE=bitwarden
SECRET_MANAGER_BITWARDEN_API_URI=https://vault.bitwarden.eu/api
SECRET_MANAGER_BITWARDEN_IDENTITY_URI=https://vault.bitwarden.eu/identity
SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=token_here
SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=org_id_here
SECRET_MANAGER_BITWARDEN_PROJECT_ID=project_id_here

# Registry
REGISTRY_SERVER_ENDPOINT=https://registry.invoice-collector.com

# Optional
DISABLE_VERIFICATION_CODE=false
```

### .env.example

Create a template:

```env
# Server
PORT=8080
ENV=prod
FRONTEND=

# Database
DATABASE_URI=
DATABASE_MONGODB_NAME=

# Secret Manager (Bitwarden)
SECRET_MANAGER_TYPE=bitwarden
SECRET_MANAGER_BITWARDEN_API_URI=
SECRET_MANAGER_BITWARDEN_IDENTITY_URI=
SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=
SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=
SECRET_MANAGER_BITWARDEN_PROJECT_ID=

# Registry
REGISTRY_SERVER_ENDPOINT=https://registry.invoice-collector.com
```

## Security Best Practices

### Never Commit Secrets

Add to `.gitignore`:

```gitignore
.env
.env.local
.env.production
secrets/
```

### Use Environment Variables

**Good:**

```yaml
environment:
  - SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=${BITWARDEN_TOKEN}
```

**Bad:**

```yaml
environment:
  - SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=actual_token_here
```

### Rotate Secrets Regularly

- Change Bitwarden tokens periodically
- Update database passwords
- Regenerate API keys

### Restrict Permissions

- Limit who can access `.env` files
- Use secrets management in CI/CD
- Encrypt at rest

## Validation

Check configuration:

```bash
# Test connection
docker compose config

# Verify variables
docker compose run invoice-collector env
```

## Troubleshooting

### Missing Variables

Error: `Environment variable X is not defined`

**Solution:** Add the variable to `.env` or `docker-compose.yml`

### Invalid Values

Error: `Invalid value for X`

**Solution:** Check the variable format and allowed values

### Connection Errors

Error: `Cannot connect to database`

**Solution:** Verify `DATABASE_URI` is correct and MongoDB is running

## Next Steps

- [Docker Deployment](docker.md)
- [Database Setup](database.md)
- [Configuration Guide](../getting-started/configuration.md)

# Configuration

Invoice Collector is configured using environment variables. This guide explains all available configuration options.

## Required Environment Variables

These environment variables must be configured for Invoice Collector to work properly.

### Server Configuration

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `PORT` | Port on which the server listens | `8080` | `8080` |
| `ENV` | Environment mode | `prod` | `prod`, `debug` |
| `FRONTEND` | Frontend URL for UI links | - | `http://localhost:8080` |

### Database Configuration

Invoice Collector uses MongoDB for data storage.

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URI` | MongoDB connection URI | `mongodb://mongodb:27017` |
| `DATABASE_MONGODB_NAME` | MongoDB database name | `prod` |

!!! tip "MongoDB Setup"
    The included `docker-compose.yml` automatically sets up MongoDB. For production, consider using MongoDB Atlas or a managed MongoDB instance.

### Secret Manager Configuration

Invoice Collector uses a secret manager to securely store user credentials. Currently, only Bitwarden is supported.

| Variable | Description | Example |
|----------|-------------|---------|
| `SECRET_MANAGER_TYPE` | Type of secret manager | `bitwarden` |
| `SECRET_MANAGER_BITWARDEN_API_URI` | Bitwarden API endpoint | `https://vault.bitwarden.eu/api` |
| `SECRET_MANAGER_BITWARDEN_IDENTITY_URI` | Bitwarden identity endpoint | `https://vault.bitwarden.eu/identity` |
| `SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN` | Bitwarden access token | Your access token |
| `SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID` | Bitwarden organization ID | Your organization ID |
| `SECRET_MANAGER_BITWARDEN_PROJECT_ID` | Bitwarden project ID | Your project ID |

!!! warning "Bitwarden Setup Required"
    You must have a Bitwarden account and configure the secret manager credentials. See the [Bitwarden Setup](#bitwarden-setup) section below.

### Registry Server Configuration

The registry server provides collector metadata and updates.

| Variable | Description | Default |
|----------|-------------|---------|
| `REGISTRY_SERVER_ENDPOINT` | Registry server URL | `https://registry.invoice-collector.com` |

## Optional Environment Variables

### Authentication Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `DISABLE_VERIFICATION_CODE` | Disable email verification codes | `false` |

!!! info "Email Verification"
    When enabled, users must verify their email before accessing the UI. Set to `true` for local development.

### Proxy Configuration

Configure proxy settings for collectors that need to bypass anti-bot protections.

| Variable | Description | Example |
|----------|-------------|---------|
| `PROXY_TYPE` | Type of proxy service | `brightdata`, `residential` |
| `PROXY_HOST` | Proxy host address | `proxy.example.com` |
| `PROXY_PORT` | Proxy port | `8000` |
| `PROXY_USERNAME` | Proxy username | `user` |
| `PROXY_PASSWORD` | Proxy password | `pass` |

### AI Configuration

Some collectors can use AI for enhanced data extraction.

| Variable | Description | Example |
|----------|-------------|---------|
| `MISTRAL_API_KEY` | Mistral AI API key | Your API key |

## Bitwarden Setup

Invoice Collector uses Bitwarden to securely store user credentials. Here's how to set it up:

### Step 1: Create a Bitwarden Account

1. Sign up at [Bitwarden](https://bitwarden.com/)
2. Choose a plan (free tier works for testing)

### Step 2: Create an Organization

1. Log in to Bitwarden
2. Go to "Organizations" and create a new organization
3. Note your **Organization ID**

### Step 3: Create a Project

1. In your organization, go to "Projects"
2. Create a new project
3. Note your **Project ID**

### Step 4: Generate Access Token

1. Go to "Settings" → "Access Tokens"
2. Create a new access token with appropriate permissions
3. Note your **Access Token**

!!! danger "Keep Your Token Secret"
    Never commit your access token to version control. Always use environment variables or secrets management.

### Step 5: Configure Invoice Collector

Add the credentials to your `docker-compose.yml`:

```yaml
environment:
  - SECRET_MANAGER_TYPE=bitwarden
  - SECRET_MANAGER_BITWARDEN_API_URI=https://vault.bitwarden.eu/api
  - SECRET_MANAGER_BITWARDEN_IDENTITY_URI=https://vault.bitwarden.eu/identity
  - SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=your_access_token
  - SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=your_org_id
  - SECRET_MANAGER_BITWARDEN_PROJECT_ID=your_project_id
```

## Example Configuration

Here's a complete example `docker-compose.yml` configuration:

```yaml
services:
  invoice-collector:
    image: ghcr.io/invoice-collector/invoice-collector:master
    ports:
      - 8080:8080
    depends_on:
      - mongodb
    environment:
      # Server
      - PORT=8080
      - ENV=prod
      - FRONTEND=http://localhost:8080
      
      # Database
      - DATABASE_URI=mongodb://mongodb:27017
      - DATABASE_MONGODB_NAME=prod
      
      # Secret Manager
      - SECRET_MANAGER_TYPE=bitwarden
      - SECRET_MANAGER_BITWARDEN_API_URI=https://vault.bitwarden.eu/api
      - SECRET_MANAGER_BITWARDEN_IDENTITY_URI=https://vault.bitwarden.eu/identity
      - SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=${BITWARDEN_TOKEN}
      - SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=${BITWARDEN_ORG_ID}
      - SECRET_MANAGER_BITWARDEN_PROJECT_ID=${BITWARDEN_PROJECT_ID}
      
      # Registry
      - REGISTRY_SERVER_ENDPOINT=https://registry.invoice-collector.com
      
      # Authentication
      - DISABLE_VERIFICATION_CODE=true
    command: npm run start

  mongodb:
    image: mongo:8.0.3
    container_name: mongodb
    volumes:
      - ./mongodb/data:/data/db
    ports:
      - 27017:27017
    command: --logpath /dev/null
```

!!! tip "Environment Variables from File"
    You can store sensitive variables in a `.env` file and reference them using `${VARIABLE_NAME}` syntax.

## Security Best Practices

1. **Use Strong Credentials** - Ensure all passwords and tokens are strong and unique
2. **Enable HTTPS** - Use a reverse proxy (nginx, Traefik) with SSL/TLS certificates
3. **Restrict Access** - Use firewall rules to limit access to the application
4. **Regular Backups** - Backup MongoDB data regularly
5. **Update Regularly** - Keep Invoice Collector and dependencies up to date

## Next Steps

- [Start collecting invoices](quick-start.md)
- [Learn about deployment options](../deployment/docker.md)
- [Configure webhooks](../api/webhooks.md)

## Troubleshooting

### Cannot Connect to Bitwarden

- Verify your API URI is correct for your region
- Check that your access token is valid
- Ensure your organization and project IDs are correct

### Database Connection Failed

- Verify MongoDB is running: `docker ps`
- Check the database URI is correct
- Ensure MongoDB is accessible from the Invoice Collector container

### Port Already in Use

Change the external port in `docker-compose.yml`:

```yaml
ports:
  - 8081:8080  # Use port 8081 instead
```

For more help, join our [Discord community](https://discord.gg/dMXTdpxMqY).

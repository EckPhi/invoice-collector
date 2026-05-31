# Installation

This guide will walk you through the process of installing Invoice Collector using Docker.

## Prerequisites

Before you begin, make sure you have the following installed on your system:

- **Docker Engine** - [Installation guide](https://docs.docker.com/engine/install/)
- **Docker Compose** - Usually included with Docker Desktop

!!! tip "Docker Desktop"
    If you're on Windows or macOS, we recommend installing Docker Desktop, which includes both Docker Engine and Docker Compose.

## Installation Methods

=== "Docker Compose (Recommended)"

    ### Step 1: Download docker-compose.yml
    
    Download the `docker-compose.yml` file from the GitHub repository:
    
    ```bash
    curl -O https://raw.githubusercontent.com/invoice-collector/invoice-collector/refs/heads/master/docker-compose.yml
    ```
    
    ### Step 2: Configure Environment Variables
    
    Edit the `docker-compose.yml` file and set the required environment variables:
    
    ```yaml
    environment:
      - PORT=8080
      - DISABLE_VERIFICATION_CODE=true
      - REGISTRY_SERVER_ENDPOINT=https://registry.invoice-collector.com
      - DATABASE_URI=mongodb://mongodb:27017
      - DATABASE_MONGODB_NAME=prod
      - SECRET_MANAGER_TYPE=bitwarden
      - SECRET_MANAGER_BITWARDEN_API_URI=https://vault.bitwarden.eu/api
      - SECRET_MANAGER_BITWARDEN_IDENTITY_URI=https://vault.bitwarden.eu/identity
      - SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=your_access_token_here
      - SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=your_org_id_here
      - SECRET_MANAGER_BITWARDEN_PROJECT_ID=your_project_id_here
      - FRONTEND=http://localhost:8080
    ```
    
    !!! warning "Required Configuration"
        You must configure the Bitwarden credentials to use the secret manager. See the [Configuration Guide](configuration.md) for details.
    
    ### Step 3: Start the Containers
    
    Build and run the containers using Docker Compose:
    
    ```bash
    sudo docker compose up -d
    ```
    
    This command will:
    
    - Download the Invoice Collector image
    - Start MongoDB container
    - Start Invoice Collector container
    - Create necessary volumes for data persistence
    
    ### Step 4: Verify Installation
    
    Check that the containers are running:
    
    ```bash
    docker compose ps
    ```
    
    You should see both `invoice-collector` and `mongodb` containers in the running state.
    
    ### Step 5: Access the Application
    
    Open your browser and navigate to:
    
    ```
    http://localhost:8080
    ```

=== "Docker Run"

    If you prefer to use `docker run` commands directly:
    
    ### Step 1: Start MongoDB
    
    ```bash
    docker run -d \
      --name mongodb \
      -v ./mongodb/data:/data/db \
      -p 27017:27017 \
      mongo:8.0.3 \
      --logpath /dev/null
    ```
    
    ### Step 2: Start Invoice Collector
    
    ```bash
    docker run -d \
      --name invoice-collector \
      --link mongodb:mongodb \
      -p 8080:8080 \
      -e PORT=8080 \
      -e DATABASE_URI=mongodb://mongodb:27017 \
      -e DATABASE_MONGODB_NAME=prod \
      -e SECRET_MANAGER_TYPE=bitwarden \
      -e SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=your_token \
      -e SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=your_org_id \
      -e SECRET_MANAGER_BITWARDEN_PROJECT_ID=your_project_id \
      ghcr.io/invoice-collector/invoice-collector:master
    ```

## Updating

To update to the latest version:

```bash
# Pull the latest image
docker compose pull

# Restart containers
docker compose up -d
```

## Uninstallation

To remove Invoice Collector:

```bash
# Stop and remove containers
docker compose down

# Remove volumes (this will delete all data)
docker compose down -v
```

!!! danger "Data Loss"
    Using the `-v` flag will delete all stored data including invoices and credentials. Make sure to backup your data before running this command.

## Next Steps

- [Configure your installation](configuration.md)
- [Start collecting invoices](quick-start.md)
- [Learn about available collectors](../guides/collectors.md)

## Troubleshooting

### Port Already in Use

If port 8080 is already in use, change the port mapping in `docker-compose.yml`:

```yaml
ports:
  - 8081:8080  # Changed external port to 8081
```

### Cannot Connect to MongoDB

Ensure the MongoDB container is running:

```bash
docker logs mongodb
```

### Permission Issues

If you encounter permission issues with volumes, ensure the directories have the correct permissions:

```bash
sudo chown -R $(whoami) ./mongodb
```

For more help, join our [Discord community](https://discord.gg/dMXTdpxMqY).

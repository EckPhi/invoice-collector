# Docker Deployment

Deploy Invoice Collector using Docker in production environments.

## Production Deployment

### Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- 2GB RAM minimum
- 10GB disk space

### Quick Start

1. **Download docker-compose.yml**

```bash
curl -O https://raw.githubusercontent.com/invoice-collector/invoice-collector/master/docker-compose.yml
```

2. **Configure Environment**

Edit `docker-compose.yml` and set all required environment variables.

3. **Start Services**

```bash
docker compose up -d
```

## Production Configuration

### Using .env File

Create a `.env` file:

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
SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN=your_token
SECRET_MANAGER_BITWARDEN_ORGANIZATION_ID=your_org
SECRET_MANAGER_BITWARDEN_PROJECT_ID=your_project

# Other settings
DISABLE_VERIFICATION_CODE=false
```

Reference in `docker-compose.yml`:

```yaml
services:
  invoice-collector:
    image: ghcr.io/invoice-collector/invoice-collector:master
    env_file: .env
    # ...
```

### HTTPS with Reverse Proxy

Use nginx or Traefik for SSL/TLS:

**nginx example:**

```nginx
server {
    listen 443 ssl http2;
    server_name invoices.yourdomain.com;

    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Resource Limits

Set resource limits:

```yaml
services:
  invoice-collector:
    image: ghcr.io/invoice-collector/invoice-collector:master
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
```

## Data Persistence

### MongoDB Volumes

```yaml
services:
  mongodb:
    image: mongo:8.0.3
    volumes:
      - mongodb_data:/data/db
      - mongodb_config:/data/configdb

volumes:
  mongodb_data:
  mongodb_config:
```

### Backup MongoDB

```bash
# Backup
docker exec mongodb mongodump --out /backup

# Restore
docker exec mongodb mongorestore /backup
```

## Scaling

### Multiple Instances

Run multiple Invoice Collector instances:

```yaml
services:
  invoice-collector:
    image: ghcr.io/invoice-collector/invoice-collector:master
    deploy:
      replicas: 3
    # ...
```

### Load Balancer

Use nginx for load balancing:

```nginx
upstream invoice_collector {
    server app1:8080;
    server app2:8080;
    server app3:8080;
}

server {
    location / {
        proxy_pass http://invoice_collector;
    }
}
```

## Monitoring

### Health Checks

Add health checks:

```yaml
services:
  invoice-collector:
    image: ghcr.io/invoice-collector/invoice-collector:master
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/api/v1/collectors"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Logging

Configure logging:

```yaml
services:
  invoice-collector:
    image: ghcr.io/invoice-collector/invoice-collector:master
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

View logs:

```bash
docker compose logs -f invoice-collector
```

## Security

### Network Isolation

```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

services:
  invoice-collector:
    networks:
      - frontend
      - backend
  
  mongodb:
    networks:
      - backend
```

### Secrets Management

Use Docker secrets:

```yaml
secrets:
  bitwarden_token:
    file: ./secrets/bitwarden_token.txt

services:
  invoice-collector:
    secrets:
      - bitwarden_token
    environment:
      - SECRET_MANAGER_BITWARDEN_ACCESS_TOKEN_FILE=/run/secrets/bitwarden_token
```

## Updates

### Rolling Updates

```bash
# Pull new image
docker compose pull

# Recreate containers
docker compose up -d
```

### Zero-Downtime Updates

```bash
# Scale up with new version
docker compose up -d --scale invoice-collector=2

# Remove old version
docker compose up -d --scale invoice-collector=1
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker compose logs invoice-collector

# Check configuration
docker compose config

# Verify image
docker images | grep invoice-collector
```

### Database Connection Issues

```bash
# Check MongoDB is running
docker compose ps mongodb

# Test connection
docker exec mongodb mongo --eval "db.serverStatus()"
```

### Out of Memory

Increase memory limit:

```yaml
deploy:
  resources:
    limits:
      memory: 4G
```

## Production Checklist

- [ ] Environment variables configured
- [ ] HTTPS enabled with valid certificate
- [ ] Database backups configured
- [ ] Resource limits set
- [ ] Health checks enabled
- [ ] Logging configured
- [ ] Monitoring set up
- [ ] Firewall rules configured
- [ ] Secrets secured
- [ ] Update strategy defined

## Next Steps

- [Environment Variables](environment-variables.md)
- [Database Setup](database.md)
- [Configuration Guide](../getting-started/configuration.md)

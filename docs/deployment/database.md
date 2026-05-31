# Database Setup

MongoDB configuration and management for Invoice Collector.

## MongoDB Setup

Invoice Collector uses MongoDB for data storage.

### Using Docker Compose

The included `docker-compose.yml` sets up MongoDB automatically:

```yaml
services:
  mongodb:
    image: mongo:8.0.3
    volumes:
      - ./mongodb/data:/data/db
    ports:
      - 27017:27017
    command: --logpath /dev/null
```

### Manual Installation

Install MongoDB locally:

**Ubuntu/Debian:**

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-8.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
```

**macOS:**

```bash
brew tap mongodb/brew
brew install mongodb-community@8.0
brew services start mongodb-community
```

## Database Configuration

### Connection String

Configure the MongoDB URI:

```env
DATABASE_URI=mongodb://localhost:27017
```

**With authentication:**

```env
DATABASE_URI=mongodb://user:password@localhost:27017
```

**Replica set:**

```env
DATABASE_URI=mongodb://host1:27017,host2:27017,host3:27017/?replicaSet=rs0
```

### Database Name

```env
DATABASE_MONGODB_NAME=prod
```

## Database Structure

### Collections

Invoice Collector creates the following collections:

#### customers

Customer accounts.

```javascript
{
  _id: ObjectId,
  name: String,
  bearer: String,
  callback: String,
  theme: String,
  created_at: Date
}
```

#### users

End users.

```javascript
{
  _id: ObjectId,
  customer_id: ObjectId,
  remote_id: String,
  email: String,
  locale: String,
  token: String,
  created_at: Date
}
```

#### credentials

Stored credentials.

```javascript
{
  _id: ObjectId,
  user_id: ObjectId,
  collector: String,
  secret_id: String,
  status: String,
  last_collection: Date,
  next_collection: Date,
  invoice_count: Number,
  error_message: String
}
```

#### invoices

Invoice metadata.

```javascript
{
  _id: ObjectId,
  credential_id: ObjectId,
  invoice_id: String,
  timestamp: Date,
  amount: String,
  link: String,
  collected_at: Date,
  metadata: Object
}
```

### Indexes

Important indexes:

```javascript
// Users
db.users.createIndex({ customer_id: 1 });
db.users.createIndex({ token: 1 });

// Credentials
db.credentials.createIndex({ user_id: 1 });
db.credentials.createIndex({ status: 1, next_collection: 1 });

// Invoices
db.invoices.createIndex({ credential_id: 1 });
db.invoices.createIndex({ timestamp: -1 });
```

## Backup and Restore

### Backup

**Full backup:**

```bash
mongodump --uri="mongodb://localhost:27017" --db=prod --out=/backup
```

**Specific collection:**

```bash
mongodump --uri="mongodb://localhost:27017" --db=prod --collection=invoices --out=/backup
```

**Compressed backup:**

```bash
mongodump --uri="mongodb://localhost:27017" --db=prod --gzip --archive=/backup/prod.gz
```

### Restore

**Full restore:**

```bash
mongorestore --uri="mongodb://localhost:27017" --db=prod /backup/prod
```

**From compressed:**

```bash
mongorestore --uri="mongodb://localhost:27017" --db=prod --gzip --archive=/backup/prod.gz
```

### Automated Backups

Create a backup script:

```bash
#!/bin/bash
BACKUP_DIR="/backups/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="invoice-collector-$DATE"

# Create backup
mongodump --uri="mongodb://localhost:27017" --db=prod --out="$BACKUP_DIR/$BACKUP_NAME"

# Compress
tar -czf "$BACKUP_DIR/$BACKUP_NAME.tar.gz" -C "$BACKUP_DIR" "$BACKUP_NAME"
rm -rf "$BACKUP_DIR/$BACKUP_NAME"

# Keep only last 7 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete
```

Schedule with cron:

```cron
0 2 * * * /path/to/backup-script.sh
```

## Monitoring

### Connection Status

Check MongoDB is running:

```bash
mongo --eval "db.serverStatus()"
```

### Database Stats

```javascript
// Connect to MongoDB
mongo

// Switch to database
use prod

// View statistics
db.stats()

// Collection sizes
db.invoices.stats()
db.credentials.stats()
```

### Slow Queries

Enable profiling:

```javascript
// Enable profiling for slow queries (>100ms)
db.setProfilingLevel(1, { slowms: 100 })

// View slow queries
db.system.profile.find().pretty()
```

## Performance Optimization

### Connection Pooling

Configure in the application:

```typescript
const client = new MongoClient(uri, {
  maxPoolSize: 50,
  minPoolSize: 10,
  maxIdleTimeMS: 30000
});
```

### Indexes

Create additional indexes for common queries:

```javascript
// Faster user lookups
db.users.createIndex({ email: 1 });

// Faster credential queries
db.credentials.createIndex({ collector: 1, status: 1 });

// Faster invoice searches
db.invoices.createIndex({ user_id: 1, timestamp: -1 });
```

### Query Optimization

Use projection to limit returned fields:

```javascript
// Instead of
db.invoices.find({ user_id: id })

// Use
db.invoices.find(
  { user_id: id },
  { _id: 1, timestamp: 1, amount: 1 }
)
```

## Security

### Authentication

Enable authentication:

```javascript
// Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "secure_password",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})

// Create application user
use prod
db.createUser({
  user: "invoice_app",
  pwd: "app_password",
  roles: [ { role: "readWrite", db: "prod" } ]
})
```

### Network Security

Bind to localhost only:

```yaml
# mongod.conf
net:
  bindIp: 127.0.0.1
  port: 27017
```

### Encryption

Enable encryption at rest:

```yaml
# mongod.conf
security:
  enableEncryption: true
  encryptionKeyFile: /path/to/keyfile
```

## Troubleshooting

### Connection Refused

Check MongoDB is running:

```bash
sudo systemctl status mongod
```

Start if needed:

```bash
sudo systemctl start mongod
```

### Out of Disk Space

Check disk usage:

```bash
df -h
```

Compact databases:

```javascript
db.runCommand({ compact: 'invoices' })
```

### Slow Performance

Check indexes:

```javascript
db.invoices.getIndexes()
```

Analyze queries:

```javascript
db.invoices.find({ user_id: id }).explain("executionStats")
```

## Production Recommendations

- [ ] Enable authentication
- [ ] Use replica set for high availability
- [ ] Configure automated backups
- [ ] Monitor disk space
- [ ] Enable SSL/TLS
- [ ] Create indexes for common queries
- [ ] Set up monitoring and alerts
- [ ] Document restore procedures

## Next Steps

- [Docker Deployment](docker.md)
- [Environment Variables](environment-variables.md)
- [Configuration Guide](../getting-started/configuration.md)

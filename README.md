# Docker Compose Setup for Rails Dependencies

> ⚠️ **SECURITY WARNING**: This configuration is for **LOCAL DEVELOPMENT ONLY**.
> Services are configured with minimal security (no passwords, disabled authentication)
> for development convenience. **DO NOT use in production environments.**

This repository contains a Docker Compose configuration for running MySQL, Redis, and Elasticsearch services that can be used by Rails applications running on the host macOS system.

## Services

### MySQL 8.0
- **Image**: `mysql:8.0`
- **Port**: `127.0.0.1:33308` (host) → `3306` (container)
- **Authentication**: Passwordless root login
- **Data Volume**: `mysql_data`
- **Security**: Bound to loopback address only

### Redis 6.2
- **Image**: `redis:6.2-alpine`
- **Port**: `127.0.0.1:6379` (host) → `6379` (container)
- **Data Volume**: `redis_data`
- **Security**: Bound to loopback address only

### Elasticsearch 7.16
- **Image**: `blacktop/elasticsearch:7.16`
- **Port**: `127.0.0.1:9200` (host) → `9200` (container)
- **Data Volume**: `elasticsearch_data`
- **Security**: Bound to loopback address only
- **Configuration**:
  - Java heap: 256MB min/max
  - Single-node discovery
  - Security disabled
  - Destructive operations allowed
  - Deprecation warnings suppressed

## Quick Start

### Start Services
```bash
docker-compose up -d
```

### Stop Services
```bash
docker-compose down
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f mysql
docker-compose logs -f redis
docker-compose logs -f elasticsearch
```

### Check Status
```bash
docker-compose ps
```

## Connection Details for Rails Applications

### MySQL
```yaml
# database.yml
production:
  adapter: mysql2
  host: 127.0.0.1
  port: 33308
  username: root
  password: ""
  database: your_app_production
```

**Connection String**: `mysql2://root@127.0.0.1:33308/your_database`

### Redis
```yaml
# redis.yml or environment variables
REDIS_URL: redis://127.0.0.1:6379
```

**Connection String**: `redis://127.0.0.1:6379`

### Elasticsearch
```yaml
# elasticsearch.yml
production:
  host: '127.0.0.1:9200'
  log: true
```

**URL**: `http://127.0.0.1:9200`

## Data Persistence

All services use named Docker volumes for data persistence:
- `mysql_data` - MySQL database files
- `redis_data` - Redis data files
- `elasticsearch_data` - Elasticsearch indices and data

Data will persist across container restarts and system reboots.

## Management Commands

### Restart Services
```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart mysql
docker-compose restart redis
docker-compose restart elasticsearch
```

### Update Services
```bash
# Pull latest images and restart
docker-compose pull
docker-compose up -d
```

### Clean Up
```bash
# Stop and remove containers (keeps data)
docker-compose down

# Stop, remove containers AND delete all data
docker-compose down -v
```

### Access Container Shells
```bash
# MySQL
docker-compose exec mysql mysql -u root

# Redis
docker-compose exec redis redis-cli

# Elasticsearch
docker-compose exec elasticsearch curl http://localhost:9200
```

## Health Checks

### MySQL
```bash
# Test connection
mysql -h 127.0.0.1 -P 33308 -u root -e "SELECT 1;"
```

### Redis
```bash
# Test connection
redis-cli -h 127.0.0.1 -p 6379 ping
```

### Elasticsearch
```bash
# Test connection
curl http://127.0.0.1:9200
```

### Environment Variables

The Docker Compose configuration uses environment variables for flexible port and host configuration. All services support custom host and port settings with sensible defaults.

#### Default Configuration
If no environment variables are set, services use these defaults:
- **MySQL**: `127.0.0.1:33308`
- **Redis**: `127.0.0.1:6379`
- **Elasticsearch**: `127.0.0.1:9200`

#### Custom Configuration
Override any service configuration by setting environment variables:

```bash
# Set custom ports and hosts
export MYSQL_HOST=127.0.0.1
export MYSQL_PORT=33308
export REDIS_HOST=127.0.0.1
export REDIS_PORT=6379
export ELASTICSEARCH_HOST=127.0.0.1
export ELASTICSEARCH_PORT=9200

# Start services with custom configuration
docker-compose up -d
```

#### How It Works
The `docker-compose.yml` uses the `${VAR:-default}` syntax:
- `${MYSQL_HOST:-127.0.0.1}` means "use MYSQL_HOST if set, otherwise use 127.0.0.1"
- This allows for flexible configuration while maintaining sensible defaults
- Services will always start even without environment variables set

#### Examples
```bash
# Use different ports
export MYSQL_PORT=33309
export REDIS_PORT=6380
export ELASTICSEARCH_PORT=9201
docker-compose up -d

# Use different host (for external access - not recommended for security)
export MYSQL_HOST=0.0.0.0
export REDIS_HOST=0.0.0.0
export ELASTICSEARCH_HOST=0.0.0.0
docker-compose up -d
```

## Auto-Start on Boot

All services are configured with `restart: always`, which means:
- Services will automatically start when Docker starts
- Services will restart if they crash
- Services will restart after system reboot (if Docker is configured to start on boot)

## Troubleshooting

### Port Conflicts
If you get "address already in use" errors:
1. Check what's using the port: `lsof -i :PORT_NUMBER`
2. Stop conflicting services or change ports in `docker-compose.yml`

### Permission Issues
If you encounter permission issues with volumes:
```bash
# Fix volume permissions
sudo chown -R $(id -u):$(id -g) /var/lib/docker/volumes/
```

### Memory Issues
If Elasticsearch fails to start due to memory issues:
1. Increase Docker memory allocation in Docker Desktop
2. Adjust `ES_JAVA_OPTS` in `docker-compose.yml`

### View Detailed Logs
```bash
# Show last 100 lines of logs
docker-compose logs --tail=100

# Follow logs in real-time
docker-compose logs -f --tail=50
```

## File Structure

```
dockers/
├── docker-compose.yml    # Main configuration file
└── README.md            # This file
```

## ⚠️ Security Notice

**This configuration is designed for LOCAL DEVELOPMENT ONLY**

### Development Security Features
- **Loopback Binding**: All services are bound exclusively to `127.0.0.1` (loopback address)
- **No External Access**: Services cannot be accessed from external networks or other machines
- **Local Development Only**: Perfect for local development environments
- **No Authentication Required**: Services use passwordless authentication for development convenience

### Network Security Verification

You can verify the security configuration by checking that services are only bound to localhost:

```bash
# Check port bindings (should show 127.0.0.1 only)
netstat -an | grep -E "(33308|6379|9200)"

# Expected output:
# tcp4  0  0  127.0.0.1.33308  *.*  LISTEN
# tcp4  0  0  127.0.0.1.6379   *.*  LISTEN
# tcp4  0  0  127.0.0.1.9200   *.*  LISTEN
```

## Requirements

- Docker Desktop for Mac
- Docker Compose (included with Docker Desktop)
- At least 2GB RAM available for containers
- Ports 33308, 6379, and 9200 available on host system

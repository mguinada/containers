# Docker Compose Setup for Development Dependencies

> ⚠️ **SECURITY WARNING**: This configuration is for **LOCAL DEVELOPMENT ONLY**.
> Services are configured with minimal security (no passwords, disabled authentication)
> for development convenience. **DO NOT use in production environments.**

This repository contains a Docker Compose configuration for running services that can be used by applications running on the host system.

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

### Automated Setup
```bash
# Run the setup script - it handles everything automatically
./bin/setup
```

The setup script will:
- ✅ **Detect configuration conflicts** and help you choose the right approach
- ✅ **Create .env file** from template if needed
- ✅ **Start all services** with proper configuration
- ✅ **Show connection details** for your applications

### Configuration Options

This setup supports two **mutually exclusive** ways to configure service ports and hosts:

> ⚠️ **IMPORTANT**: Choose **ONE** approach only. Mixing both can cause confusion and unexpected behavior.

#### Option 1: .env File
**Best for**: Project-specific configuration, team consistency, persistent settings

```bash
# The setup script creates this automatically, or you can:
cp env.example .env

# Edit .env file to customize ports
MYSQL_PORT=33308
REDIS_PORT=6379
ELASTICSEARCH_PORT=9200
```

**Benefits**:
- ✅ **Project-specific**: Variables tied to the project, not your shell
- ✅ **Persistent**: Survives shell restarts and new terminal sessions
- ✅ **Team-friendly**: Other developers can use the same configuration
- ✅ **Clean workflow**: Just run `docker-compose up -d`

#### Option 2: Shell Environment Variables
**Best for**: Application config sync, multi-project workflows, CI/CD automation

```bash
# Set in your shell (e.g., .zshrc, .bashrc)
export MYSQL_HOST=127.0.0.1
export MYSQL_PORT=33308
export REDIS_HOST=127.0.0.1
export REDIS_PORT=6379
export ELASTICSEARCH_HOST=127.0.0.1
export ELASTICSEARCH_PORT=9200

# Then run setup
./bin/setup
```

**Benefits**:
- ✅ **App config sync**: Framework configs can directly read these variables
- ✅ **Multi-project**: Different projects can use different ports
- ✅ **CI/CD ready**: Perfect for automated environments
- ✅ **No file management**: No `.env` files to track

#### How It Works
Docker Compose uses `${VAR:-default}` syntax, which means:
- **Shell variables take precedence** over `.env` file
- **`.env` file provides defaults** when shell vars not set
- **Choose one approach** to avoid configuration conflicts

#### Example: Framework Configuration Sync
When using shell variables, your application configuration files can directly reference them:

```yaml
# database.yml
default: &default
  adapter: mysql2
  host: <%= ENV['MYSQL_HOST'] || 'localhost' %>
  port: <%= ENV['MYSQL_PORT'] || 3306 %>
  username: root
  password: ""
```

**Result**: Change ports in one place (shell), everything updates automatically! 🎯

### Stop Services
```bash
# Stop and remove containers (preserves data)
./bin/teardown

# Or use docker-compose directly
docker-compose down
```

## Management Commands

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

### Run Tests
```bash
# Run comprehensive tests
./bin/test
```

## Connection Details for Applications

### MySQL
```yaml
# Example configuration (adjust for your framework)
host: 127.0.0.1
port: 33308
username: root
password: ""
database: your_app_production
```

**Connection String**: `mysql://root@127.0.0.1:33308/your_database`

### Redis
```yaml
# Example configuration (adjust for your framework)
REDIS_URL: redis://127.0.0.1:6379
```

**Connection String**: `redis://127.0.0.1:6379`

### Elasticsearch
```yaml
# Example configuration (adjust for your framework)
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

## Testing

### Automated Testing
Run the comprehensive test suite to validate all services and configurations:

```bash
./bin/test
```

The test script validates:
- ✅ **Connectivity**: All services respond correctly
- ✅ **Security**: Ports are bound to localhost only
- ✅ **Configuration**: .env file is properly configured
- ✅ **Service Health**: All containers are running

### Manual Health Checks

#### MySQL
```bash
# Test connection
mysql -h 127.0.0.1 -P 33308 -u root -e "SELECT 1;"
```

#### Redis
```bash
# Test connection
redis-cli -h 127.0.0.1 -p 6379 ping
```

#### Elasticsearch
```bash
# Test connection
curl http://127.0.0.1:9200
```

### Environment Variables

The Docker Compose configuration uses environment variables for flexible port and host configuration. The easiest way to manage these is with a `.env` file.

#### Quick Setup
1. **Copy the environment template:**
   ```bash
   cp env.example .env
   ```

2. **Start services:**
   ```bash
   docker-compose up -d
   ```

#### Default Configuration
The `env.example` file contains these defaults:
- **MySQL**: `127.0.0.1:33308`
- **Redis**: `127.0.0.1:6379`
- **Elasticsearch**: `127.0.0.1:9200`

#### Custom Configuration
Edit the `.env` file to customize any settings:

```bash
# .env
MYSQL_HOST=127.0.0.1
MYSQL_PORT=33308
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
ELASTICSEARCH_HOST=127.0.0.1
ELASTICSEARCH_PORT=9200
```

#### How It Works
- Docker Compose automatically reads the `.env` file
- The `docker-compose.yml` uses the `${VAR:-default}` syntax
- `${MYSQL_HOST:-127.0.0.1}` means "use MYSQL_HOST if set, otherwise use 127.0.0.1"
- Services will always start with sensible defaults

#### Examples
```bash
# Use different ports - edit .env file
MYSQL_PORT=33309
REDIS_PORT=6380
ELASTICSEARCH_PORT=9201

# Use different host (for external access - not recommended for security)
MYSQL_HOST=0.0.0.0
REDIS_HOST=0.0.0.0
ELASTICSEARCH_HOST=0.0.0.0
```

#### Benefits of .env File
- **Project-specific**: Variables tied to the project, not your shell
- **Persistent**: Survives shell restarts and new terminal sessions
- **Team-friendly**: Other developers can use the same configuration
- **Clean workflow**: Just run `docker-compose up -d`

#### Scripts
The `bin/` directory contains helpful scripts:
- **`bin/setup`**: Automated setup script that creates `.env` file and starts services
- **`bin/test`**: Comprehensive test script that validates all services and configurations
- **`bin/teardown`**: Stops and removes containers while preserving data volumes

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
├── bin/
│   ├── setup            # Automated setup script
│   ├── test             # Comprehensive test script
│   └── teardown         # Container teardown script
├── docker-compose.yml    # Main configuration file
├── env.example          # Environment variables template
├── .gitignore          # Git ignore file
└── README.md           # This file
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

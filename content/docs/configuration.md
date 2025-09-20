---
title: Configuration
weight: 3
type: docs
---

TagCache is designed to work well with minimal configuration, but offers extensive customization options for advanced use cases.

## Configuration File Location

TagCache looks for configuration files in the following locations (in order):

1. File specified by `--config` command line argument
2. `./tagcache.conf` (current directory)
3. Platform-specific default locations:
   - **Linux**: `/etc/tagcache/tagcache.conf`
   - **macOS**: `/opt/homebrew/etc/tagcache.conf` or `/usr/local/etc/tagcache.conf`
   - **Windows**: `C:\ProgramData\tagcache\tagcache.conf`

## Generating a Sample Configuration

```bash
# Generate a sample configuration file
tagcache config generate > tagcache.conf

# Or save to default location
sudo tagcache config generate > /etc/tagcache/tagcache.conf
```

## Complete Configuration Reference

```toml
# TagCache Configuration File

[server]
# Network binding
http_port = 8080
tcp_port = 1984
bind_address = "0.0.0.0"

# Server behavior
max_connections = 10000
request_timeout_ms = 30000
shutdown_timeout_ms = 10000

# Enable/disable protocols
enable_http = true
enable_tcp = true
enable_web_ui = true

[auth]
# Authentication settings
enabled = true
username = "admin"
password = "password"

# Password hashing (bcrypt)
hash_rounds = 12

[cache]
# Memory management
max_memory = "1GB"              # 1GB, 500MB, 2GB, etc.
max_keys = 1000000              # Maximum number of keys
default_ttl_ms = 3600000        # Default TTL: 1 hour

# Eviction policy when memory limit is reached
eviction_policy = "lru"         # lru, lfu, random, none

[performance]
# Sharding configuration
num_shards = 256                # Number of cache shards (power of 2)

# Background tasks
cleanup_interval_ms = 60000     # Cleanup expired keys every minute
stats_interval_ms = 5000        # Update stats every 5 seconds

# Connection pooling
tcp_pool_size = 100
http_pool_size = 1000

[logging]
# Log level: error, warn, info, debug, trace
level = "info"

# Log format: json, pretty
format = "pretty"

# Log output
output = "stdout"               # stdout, stderr, or file path

# Enable request logging
log_requests = false

[monitoring]
# Health check endpoint
health_check = true
health_endpoint = "/health"

# Metrics endpoint
metrics_enabled = true
metrics_endpoint = "/metrics"

# Prometheus format
prometheus_format = true

[security]
# CORS settings for HTTP API
cors_enabled = true
cors_origins = ["*"]
cors_methods = ["GET", "POST", "PUT", "DELETE"]
cors_headers = ["Content-Type", "Authorization"]

# Rate limiting
rate_limiting = false
rate_limit_requests = 1000      # Requests per minute
rate_limit_window_ms = 60000    # 1 minute window

# TLS/SSL settings (optional)
tls_enabled = false
tls_cert_file = "/path/to/cert.pem"
tls_key_file = "/path/to/key.pem"

[clustering]
# Clustering support (enterprise feature)
enabled = false
node_id = "node-1"
cluster_port = 7946
peers = ["node-2:7946", "node-3:7946"]

[persistence]
# Data persistence (enterprise feature)
enabled = false
storage_path = "/var/lib/tagcache"
snapshot_interval_ms = 300000   # 5 minutes
```

## Environment Variables

Configuration values can be overridden using environment variables with the prefix `TAGCACHE_`:

```bash
# Server settings
export TAGCACHE_SERVER_HTTP_PORT=9080
export TAGCACHE_SERVER_TCP_PORT=2984
export TAGCACHE_SERVER_BIND_ADDRESS="127.0.0.1"

# Authentication
export TAGCACHE_AUTH_USERNAME="myuser"
export TAGCACHE_AUTH_PASSWORD="secure-password"

# Cache settings
export TAGCACHE_CACHE_MAX_MEMORY="2GB"
export TAGCACHE_CACHE_DEFAULT_TTL_MS=7200000

# Logging
export TAGCACHE_LOGGING_LEVEL="debug"
export TAGCACHE_LOGGING_FORMAT="json"

# Start with environment variables
tagcache server
```

## Command Line Arguments

Many configuration options can be specified via command line:

```bash
# Basic server options
tagcache server \
  --http-port 9080 \
  --tcp-port 2984 \
  --bind-address 127.0.0.1 \
  --config /path/to/custom.conf

# Authentication
tagcache server \
  --auth-username myuser \
  --auth-password mypassword

# Memory and performance
tagcache server \
  --max-memory 2GB \
  --num-shards 512 \
  --default-ttl 7200000

# Logging
tagcache server \
  --log-level debug \
  --log-format json
```

## Configuration Sections

### Server Configuration

Controls network binding and server behavior:

```toml
[server]
http_port = 8080                # HTTP API port
tcp_port = 1984                 # TCP protocol port
bind_address = "0.0.0.0"        # Bind to all interfaces
max_connections = 10000         # Maximum concurrent connections
request_timeout_ms = 30000      # Request timeout (30 seconds)
shutdown_timeout_ms = 10000     # Graceful shutdown timeout
enable_http = true              # Enable HTTP API
enable_tcp = true               # Enable TCP protocol
enable_web_ui = true            # Enable web dashboard
```

### Authentication

Configure user authentication:

```toml
[auth]
enabled = true                  # Enable authentication
username = "admin"              # Default username
password = "password"           # Default password (change this!)
hash_rounds = 12                # BCrypt hash rounds
```

{{< callout type="warning" >}}
**Security Warning**: Always change the default credentials in production!
{{< /callout >}}

### Cache Configuration

Memory and cache behavior settings:

```toml
[cache]
max_memory = "1GB"              # Maximum memory usage
max_keys = 1000000              # Maximum number of keys
default_ttl_ms = 3600000        # Default TTL (1 hour)
eviction_policy = "lru"         # Eviction policy
```

**Eviction Policies:**
- `lru`: Least Recently Used (recommended)
- `lfu`: Least Frequently Used
- `random`: Random eviction
- `none`: No eviction (may cause OOM)

### Performance Tuning

Optimize for your workload:

```toml
[performance]
num_shards = 256                # Cache shards (power of 2)
cleanup_interval_ms = 60000     # Cleanup frequency
stats_interval_ms = 5000        # Stats update frequency
tcp_pool_size = 100             # TCP connection pool
http_pool_size = 1000           # HTTP connection pool
```

**Tuning Guidelines:**
- Increase `num_shards` for high concurrency (256-1024)
- Decrease `cleanup_interval_ms` for more frequent cleanup
- Adjust pool sizes based on expected load

### Security Settings

Configure security features:

```toml
[security]
cors_enabled = true             # Enable CORS
cors_origins = ["*"]            # Allowed origins
rate_limiting = false           # Enable rate limiting
rate_limit_requests = 1000      # Requests per window
rate_limit_window_ms = 60000    # Rate limit window
tls_enabled = false             # Enable TLS/HTTPS
tls_cert_file = "/path/to/cert.pem"
tls_key_file = "/path/to/key.pem"
```

## Production Configuration Examples

### High-Performance Setup

```toml
[server]
http_port = 8080
tcp_port = 1984
bind_address = "0.0.0.0"
max_connections = 50000

[cache]
max_memory = "8GB"
max_keys = 10000000
eviction_policy = "lru"

[performance]
num_shards = 1024
cleanup_interval_ms = 30000
tcp_pool_size = 500
http_pool_size = 2000

[logging]
level = "warn"
format = "json"
log_requests = false
```

### Security-Focused Setup

```toml
[auth]
enabled = true
username = "cache_admin"
password = "very-secure-password-123!"
hash_rounds = 15

[security]
cors_enabled = true
cors_origins = ["https://yourdomain.com"]
rate_limiting = true
rate_limit_requests = 500
tls_enabled = true
tls_cert_file = "/etc/ssl/certs/tagcache.crt"
tls_key_file = "/etc/ssl/private/tagcache.key"

[server]
bind_address = "127.0.0.1"      # Bind to localhost only
```

### Memory-Constrained Setup

```toml
[cache]
max_memory = "256MB"
max_keys = 100000
eviction_policy = "lru"

[performance]
num_shards = 64
cleanup_interval_ms = 30000

[logging]
level = "error"
```

## Validation

Validate your configuration before starting:

```bash
# Check configuration syntax
tagcache config validate /path/to/tagcache.conf

# Show effective configuration
tagcache config show
```

## Dynamic Configuration

Some settings can be updated at runtime via the HTTP API:

```bash
# Update log level
curl -X POST http://localhost:8080/api/admin/config \
  -u "admin:password" \
  -H "Content-Type: application/json" \
  -d '{"logging.level": "debug"}'

# Update cache settings
curl -X POST http://localhost:8080/api/admin/config \
  -u "admin:password" \
  -H "Content-Type: application/json" \
  -d '{"cache.default_ttl_ms": 7200000}'
```

## Best Practices

1. **Security**: Always change default credentials
2. **Memory**: Set appropriate memory limits for your system
3. **Sharding**: Use more shards for high-concurrency workloads
4. **Monitoring**: Enable metrics and health checks
5. **Logging**: Use appropriate log levels for production
6. **Backup**: Keep configuration files in version control

{{< callout type="info" >}}
Configuration changes require a server restart unless they support dynamic updates.
{{< /callout >}}

## Next Steps

{{< cards >}}
  {{< card link="../cli" title="CLI Reference" icon="terminal" subtitle="Learn the command-line interface" >}}
  {{< card link="../api" title="API Reference" icon="book-open" subtitle="HTTP and TCP API documentation" >}}
  {{< card link="../deployment" title="Deployment" icon="server" subtitle="Production deployment guides" >}}
{{< /cards >}}

---
title: CLI Reference
weight: 4
type: docs
---

TagCache provides a comprehensive command-line interface for server management and cache operations.

## Global Options

These options are available for all commands:

```bash
tagcache [OPTIONS] <COMMAND>
```

### Authentication Options

```bash
--username <USERNAME>    # Username for authentication
--password <PASSWORD>    # Password for authentication
--auth-token <TOKEN>     # Alternative to username/password
```

### Connection Options

```bash
--host <HOST>           # Server hostname (default: localhost)
--port <PORT>           # HTTP port (default: 8080)
--tcp-port <PORT>       # TCP port (default: 1984)
--protocol <PROTOCOL>   # Protocol: http or tcp (default: http)
--timeout <SECONDS>     # Request timeout (default: 30)
```

### Output Options

```bash
--format <FORMAT>       # Output format: json, table, yaml (default: table)
--quiet                 # Suppress non-essential output
--verbose               # Enable verbose output
--no-color             # Disable colored output
```

## Server Management

### Start Server

```bash
# Start with default configuration
tagcache server

# Start with custom configuration
tagcache server --config /path/to/config.toml

# Start with inline options
tagcache server \
  --http-port 9080 \
  --tcp-port 2984 \
  --auth-username admin \
  --auth-password secure123
```

**Server Options:**
```bash
--config <FILE>             # Configuration file path
--http-port <PORT>          # HTTP port (default: 8080)
--tcp-port <PORT>           # TCP port (default: 1984)
--bind-address <ADDRESS>    # Bind address (default: 0.0.0.0)
--auth-username <USER>      # Authentication username
--auth-password <PASS>      # Authentication password
--max-memory <SIZE>         # Maximum memory (e.g., 1GB, 500MB)
--num-shards <COUNT>        # Number of cache shards
--log-level <LEVEL>         # Log level: error, warn, info, debug, trace
--log-format <FORMAT>       # Log format: json, pretty
--daemonize                 # Run as daemon (Unix only)
--pid-file <FILE>          # PID file path (when daemonizing)
```

### Configuration Management

```bash
# Generate sample configuration
tagcache config generate

# Validate configuration file
tagcache config validate /path/to/config.toml

# Show effective configuration
tagcache config show

# Show configuration schema
tagcache config schema
```

## Cache Operations

### Set Operations

```bash
# Set a simple key-value pair
tagcache put "key" "value"

# Set with TTL (time-to-live) in milliseconds
tagcache put "session:123" "user_data" --ttl 3600000  # 1 hour

# Set with tags for group invalidation
tagcache put "user:123" '{"name":"John","email":"john@example.com"}' \
  --tags "user,profile"

# Set multiple tags
tagcache put "product:456" '{"name":"Widget","price":19.99}' \
  --tags "product,inventory,widget"

# Set with both TTL and tags
tagcache put "temp:789" "temporary_data" \
  --ttl 60000 \
  --tags "temp,session"
```

### Get Operations

```bash
# Get a single key
tagcache get key "user:123"

# Get multiple keys
tagcache get keys "user:123" "user:456" "user:789"

# Get all keys (use with caution on large datasets)
tagcache get all

# Get keys by pattern
tagcache get pattern "user:*"

# Get keys by tag
tagcache get tag "user"
tagcache get tag "product"

# Get with metadata (TTL, tags, etc.)
tagcache get key "user:123" --with-meta
```

### Atomic Operations

```bash
# Add a key only if it doesn't exist
tagcache add "counter" "100"
tagcache add "unique:token" "abc123" --ttl 60000

# Increment operations
tagcache increment "page_views"              # Increment by 1
tagcache increment "page_views" --by 5       # Increment by 5
tagcache increment "api_calls" --by 10 --ttl 3600000

# Decrement operations
tagcache decrement "quota_remaining"         # Decrement by 1
tagcache decrement "quota_remaining" --by 3  # Decrement by 3
tagcache decrement "credits" --by 50

# Atomic operations with initialization
tagcache increment "new_counter" --by 1 --initial 0    # Initialize if not exists
tagcache decrement "countdown" --by 1 --initial 100
```

### Delete Operations

```bash
# Delete a single key
tagcache delete key "user:123"

# Delete multiple keys
tagcache delete keys "user:123" "user:456" "temp:789"

# Delete by pattern
tagcache delete pattern "temp:*"
tagcache delete pattern "session:*"

# Delete by tag (powerful!)
tagcache delete tag "user"          # Delete all keys tagged with "user"
tagcache delete tag "expired"       # Delete all keys tagged with "expired"

# Delete with confirmation
tagcache delete tag "product" --confirm

# Clear all cache (use with extreme caution!)
tagcache delete all --confirm
```

### Information Commands

```bash
# Check if key exists
tagcache exists "user:123"

# Get key metadata (TTL, tags, size)
tagcache info "user:123"

# List all tags
tagcache tags list

# Get tag information
tagcache tags info "user"

# Count keys
tagcache count all
tagcache count tag "user"
tagcache count pattern "session:*"

# Get server statistics
tagcache stats

# Get detailed performance metrics
tagcache stats --detailed

# Monitor real-time statistics
tagcache stats --watch
```

## Administrative Commands

### User Management

```bash
# Change password
tagcache admin passwd --current old_password --new new_password

# Reset authentication (requires server restart)
tagcache admin reset-auth

# Show current user info
tagcache admin whoami
```

### Server Control

```bash
# Check server health
tagcache health

# Get server info
tagcache info

# Graceful shutdown (if running as daemon)
tagcache shutdown

# Force garbage collection
tagcache admin gc

# Flush expired keys
tagcache admin flush-expired

# Backup cache to file
tagcache admin backup /path/to/backup.json

# Restore cache from file
tagcache admin restore /path/to/backup.json
```

## Batch Operations

### Import/Export

```bash
# Export all data
tagcache export all > backup.json

# Export by tag
tagcache export tag "user" > users.json

# Export by pattern
tagcache export pattern "session:*" > sessions.json

# Import data
tagcache import < backup.json

# Import with options
tagcache import --skip-existing --ttl 3600000 < data.json
```

### Bulk Operations

```bash
# Bulk set from file (JSON format)
tagcache bulk set --file data.json

# Bulk delete from file (list of keys)
tagcache bulk delete --file keys.txt

# Bulk operations with concurrency
tagcache bulk set --file data.json --workers 10
```

## Monitoring and Debugging

### Real-time Monitoring

```bash
# Watch cache statistics
tagcache stats --watch --interval 1

# Monitor specific metrics
tagcache monitor memory
tagcache monitor ops
tagcache monitor connections

# Real-time key operations
tagcache monitor keys --pattern "user:*"
```

### Debugging

```bash
# Trace specific key operations
tagcache trace "user:123"

# Debug connection issues
tagcache debug connection

# Show internal cache state
tagcache debug shards

# Test server connectivity
tagcache ping
```

## Examples

### Common Use Cases

**Session Management:**
```bash
# Create session
tagcache put "session:abc123" '{"user_id":123,"expires":"2024-12-31"}' \
  --ttl 1800000 --tags "session,user:123"

# Check session
tagcache get key "session:abc123"

# Invalidate user sessions
tagcache delete tag "user:123"
```

**API Rate Limiting:**
```bash
# Initialize rate limit counter
tagcache add "rate:api:user123" "0" --ttl 60000  # 1 minute window

# Increment on each request
tagcache increment "rate:api:user123"

# Check current count
tagcache get key "rate:api:user123"
```

**Cache Warming:**
```bash
# Warm cache with user data
tagcache put "user:123" '$(curl -s api/users/123)' --tags "user,profile"
tagcache put "user:456" '$(curl -s api/users/456)' --tags "user,profile"

# Invalidate all user cache when user data changes
tagcache delete tag "user"
```

**Performance Testing:**
```bash
# Set test data
for i in {1..1000}; do
  tagcache put "test:$i" "value_$i" --tags "test" &
done
wait

# Get performance stats
tagcache stats

# Clean up
tagcache delete tag "test"
```

## Output Formats

### JSON Format

```bash
tagcache get key "user:123" --format json
```

```json
{
  "key": "user:123",
  "value": "{\"name\":\"John\",\"email\":\"john@example.com\"}",
  "ttl_ms": 3540000,
  "tags": ["user", "profile"],
  "created_at": "2024-01-15T10:30:00Z",
  "accessed_at": "2024-01-15T10:35:00Z"
}
```

### Table Format (Default)

```bash
tagcache stats --format table
```

```
┌─────────────────────┬──────────────┐
│ Metric              │ Value        │
├─────────────────────┼──────────────┤
│ Total Keys          │ 1,234        │
│ Total Memory        │ 45.2 MB      │
│ Operations/sec      │ 12,543       │
│ Hit Rate            │ 94.5%        │
│ Uptime              │ 2d 5h 30m    │
└─────────────────────┴──────────────┘
```

### YAML Format

```bash
tagcache get key "user:123" --format yaml
```

```yaml
key: user:123
value: '{"name":"John","email":"john@example.com"}'
ttl_ms: 3540000
tags:
  - user
  - profile
created_at: "2024-01-15T10:30:00Z"
accessed_at: "2024-01-15T10:35:00Z"
```

## Error Handling

TagCache CLI provides clear error messages and appropriate exit codes:

- `0`: Success
- `1`: General error
- `2`: Authentication error
- `3`: Connection error
- `4`: Not found
- `5`: Validation error

```bash
# Check exit code
tagcache get key "nonexistent"
echo $?  # Will be 4 (not found)
```

## Configuration via Environment

CLI options can be set via environment variables:

```bash
export TAGCACHE_HOST="cache.example.com"
export TAGCACHE_USERNAME="admin"
export TAGCACHE_PASSWORD="secure123"
export TAGCACHE_FORMAT="json"

# Now commands use these defaults
tagcache get key "user:123"
```

## Next Steps

{{< cards >}}
  {{< card link="../api" title="API Reference" icon="book-open" subtitle="HTTP and TCP API documentation" >}}
  {{< card link="../examples" title="Examples" icon="code" subtitle="Real-world usage examples" >}}
  {{< card link="../performance" title="Performance" icon="chart-bar" subtitle="Benchmarking and optimization" >}}
{{< /cards >}}

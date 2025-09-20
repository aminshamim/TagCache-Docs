---
title: Getting Started
weight: 1
type: docs
---

This guide will help you quickly get TagCache up and running on your system.

## What is TagCache?

TagCache is a high-performance, sharded, tag-aware in-memory cache server written in Rust that offers:

- **JSON HTTP API** (port 8080) - RESTful interface for web applications
- **TCP Protocol** (port 1984) - Ultra-low latency binary protocol
- **Tag-based Invalidation** - Organize and clear related data efficiently
- **Atomic Operations** - ADD, INCR, DECR with race-condition protection
- **Built-in Web Dashboard** - Beautiful React UI for monitoring and management

## Quick Installation

Choose your preferred installation method:

{{< tabs items="Homebrew,Binary Download,Docker" >}}

{{< tab >}}
**macOS/Linux via Homebrew** (Recommended)

```bash
# Add the tap
brew tap aminshamim/tap

# Install TagCache
brew install tagcache

# Verify installation
tagcache --version
```
{{< /tab >}}

{{< tab >}}
**Download Pre-built Binary**

```bash
# Linux x86_64
curl -L -o tagcache-linux-x86_64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/latest/download/tagcache-linux-x86_64.tar.gz
tar xzf tagcache-linux-x86_64.tar.gz
sudo cp tagcache /usr/local/bin/

# macOS Intel
curl -L -o tagcache-macos-x86_64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/latest/download/tagcache-macos-x86_64.tar.gz
tar xzf tagcache-macos-x86_64.tar.gz
sudo cp tagcache /usr/local/bin/

# macOS Apple Silicon
curl -L -o tagcache-macos-arm64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/latest/download/tagcache-macos-arm64.tar.gz
tar xzf tagcache-macos-arm64.tar.gz
sudo cp tagcache /usr/local/bin/
```
{{< /tab >}}

{{< tab >}}
**Docker**

```bash
# Run with default settings
docker run -p 1984:1984 -p 8080:8080 aminshamim/tagcache

# Or with Docker Compose
curl -O https://raw.githubusercontent.com/aminshamim/tagcache/main/docker-compose.yml
docker-compose up
```
{{< /tab >}}

{{< /tabs >}}

## Starting the Server

Start TagCache with default settings:

```bash
tagcache server
```

You should see output like:

```
🚀 TagCache v1.0.8 starting...
🔧 Config: /path/to/tagcache.conf
🌐 HTTP Server: http://0.0.0.0:8080
⚡ TCP Server: 0.0.0.0:1984
🔐 Authentication: enabled (user: admin)
✅ Server started successfully!
```

{{< callout type="info" >}}
**Default Credentials**: Username: `admin`, Password: `password`
Change these in production!
{{< /callout >}}

## First Steps

### 1. Access the Web Dashboard

Open your browser to [http://localhost:8080](http://localhost:8080)

The web dashboard provides:
- Real-time server statistics
- Key-value browser and editor
- Performance monitoring
- Cache management tools

### 2. Basic CLI Operations

Test the CLI with basic operations:

```bash
# Set a simple key-value pair
tagcache --username admin --password password put "hello" "world"

# Get the value
tagcache --username admin --password password get key "hello"

# Set with expiration (10 seconds)
tagcache --username admin --password password put "temp" "data" --ttl 10000

# Set with tags for group invalidation
tagcache --username admin --password password put "user:123" '{"name":"John"}' --tags "user,profile"

# Atomic operations
tagcache --username admin --password password add "counter" "100"
tagcache --username admin --password password increment "counter" --by 5
tagcache --username admin --password password decrement "counter" --by 2

# View server statistics
tagcache --username admin --password password stats
```

### 3. HTTP API Examples

Test the HTTP API with curl:

```bash
# Set a value
curl -X POST http://localhost:8080/api/set \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{"key": "api-test", "value": "Hello from HTTP!", "ttl_ms": 60000}'

# Get a value
curl -X GET http://localhost:8080/api/get/api-test \
  -u "admin:password"

# Get server stats
curl -X GET http://localhost:8080/api/stats \
  -u "admin:password"
```

## Configuration

TagCache uses a configuration file located at:
- Linux: `/etc/tagcache/tagcache.conf`
- macOS: `/opt/homebrew/etc/tagcache.conf` or `/usr/local/etc/tagcache.conf`
- Windows: `C:\ProgramData\tagcache\tagcache.conf`

Basic configuration:

```toml
[server]
http_port = 8080
tcp_port = 1984
bind_address = "0.0.0.0"

[auth]
username = "admin"
password = "your-secure-password"

[cache]
max_memory = "1GB"
default_ttl_ms = 3600000  # 1 hour

[performance]
num_shards = 256
cleanup_interval_ms = 60000
```

## What's Next?

{{< cards >}}
  {{< card link="../installation" title="Detailed Installation" icon="download" subtitle="Complete installation guide for all platforms" >}}
  {{< card link="../configuration" title="Configuration" icon="cog" subtitle="Configure TagCache for your needs" >}}
  {{< card link="../api" title="API Reference" icon="book-open" subtitle="Complete API documentation" >}}
  {{< card link="../examples" title="Examples" icon="code" subtitle="Real-world usage examples" >}}
{{< /cards >}}

## Common Use Cases

- **Session Storage**: Store user sessions with automatic expiration
- **API Response Caching**: Cache expensive API responses with tag-based invalidation
- **Rate Limiting**: Use atomic counters for request rate limiting
- **Database Query Caching**: Cache query results and invalidate by related tags
- **Real-time Analytics**: Store and aggregate metrics data

{{< callout type="warning" >}}
**Production Usage**: Don't forget to change the default credentials and configure proper authentication for production deployments!
{{< /callout >}}

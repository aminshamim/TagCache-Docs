---
title: API Reference
weight: 5
type: docs
---

TagCache provides two high-performance APIs: a JSON HTTP API for web applications and a binary TCP protocol for ultra-low latency operations.

## HTTP JSON API

The HTTP API runs on port 8080 (configurable) and provides a RESTful interface with JSON payloads.

### Base URL

```
http://localhost:8080/api
```

### Authentication

All API endpoints require HTTP Basic Authentication:

```bash
curl -u "username:password" http://localhost:8080/api/stats
```

### Content Type

All POST/PUT requests should include:

```
Content-Type: application/json
```

### Response Format

All responses follow this structure:

```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

Error responses:

```json
{
  "success": false,
  "data": null,
  "error": "Key not found",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## Cache Operations

### Set Key

Set a key-value pair with optional TTL and tags.

```http
POST /api/set
```

**Request Body:**

```json
{
  "key": "user:123",
  "value": "{\"name\":\"John\",\"email\":\"john@example.com\"}",
  "ttl_ms": 3600000,
  "tags": ["user", "profile"]
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "user:123",
    "stored": true
  },
  "error": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**cURL Example:**

```bash
curl -X POST http://localhost:8080/api/set \
  -u "admin:password" \
  -H "Content-Type: application/json" \
  -d '{
    "key": "user:123",
    "value": "{\"name\":\"John\",\"email\":\"john@example.com\"}",
    "ttl_ms": 3600000,
    "tags": ["user", "profile"]
  }'
```

### Get Key

Retrieve a single key.

```http
GET /api/get/{key}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "user:123",
    "value": "{\"name\":\"John\",\"email\":\"john@example.com\"}",
    "ttl_ms": 3540000,
    "tags": ["user", "profile"],
    "created_at": "2024-01-15T10:30:00Z",
    "accessed_at": "2024-01-15T10:35:00Z"
  },
  "error": null
}
```

**cURL Example:**

```bash
curl -X GET http://localhost:8080/api/get/user:123 \
  -u "admin:password"
```

### Get Multiple Keys

Retrieve multiple keys in a single request.

```http
POST /api/get/batch
```

**Request Body:**

```json
{
  "keys": ["user:123", "user:456", "session:abc"]
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "user:123": {
      "value": "{\"name\":\"John\"}",
      "ttl_ms": 3540000,
      "tags": ["user"]
    },
    "user:456": null,
    "session:abc": {
      "value": "session_data",
      "ttl_ms": 1800000,
      "tags": ["session"]
    }
  }
}
```

### Get by Tag

Retrieve all keys associated with a specific tag.

```http
GET /api/get/tag/{tag}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "tag": "user",
    "keys": {
      "user:123": {
        "value": "{\"name\":\"John\"}",
        "ttl_ms": 3540000
      },
      "user:456": {
        "value": "{\"name\":\"Jane\"}",
        "ttl_ms": 3500000
      }
    }
  }
}
```

**cURL Example:**

```bash
curl -X GET http://localhost:8080/api/get/tag/user \
  -u "admin:password"
```

### Delete Key

Delete a single key.

```http
DELETE /api/delete/{key}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "user:123",
    "deleted": true
  }
}
```

**cURL Example:**

```bash
curl -X DELETE http://localhost:8080/api/delete/user:123 \
  -u "admin:password"
```

### Delete by Tag

Delete all keys associated with a tag.

```http
DELETE /api/delete/tag/{tag}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "tag": "user",
    "deleted_count": 25
  }
}
```

**cURL Example:**

```bash
curl -X DELETE http://localhost:8080/api/delete/tag/user \
  -u "admin:password"
```

## Atomic Operations

### Add (Set if Not Exists)

Set a key only if it doesn't already exist.

```http
POST /api/add
```

**Request Body:**

```json
{
  "key": "counter",
  "value": "100",
  "ttl_ms": 3600000,
  "tags": ["counter"]
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "counter",
    "added": true,
    "existed": false
  }
}
```

### Increment

Atomically increment a numeric value.

```http
POST /api/increment
```

**Request Body:**

```json
{
  "key": "page_views",
  "by": 5,
  "initial": 0,
  "ttl_ms": 86400000
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "page_views",
    "old_value": 100,
    "new_value": 105
  }
}
```

**cURL Example:**

```bash
curl -X POST http://localhost:8080/api/increment \
  -u "admin:password" \
  -H "Content-Type: application/json" \
  -d '{
    "key": "page_views",
    "by": 1,
    "initial": 0
  }'
```

### Decrement

Atomically decrement a numeric value.

```http
POST /api/decrement
```

**Request Body:**

```json
{
  "key": "quota_remaining",
  "by": 10,
  "ttl_ms": 3600000
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "quota_remaining",
    "old_value": 1000,
    "new_value": 990
  }
}
```

## Information & Monitoring

### Check if Key Exists

```http
GET /api/exists/{key}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "user:123",
    "exists": true
  }
}
```

### Get Key Information

Get metadata about a key without retrieving its value.

```http
GET /api/info/{key}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "key": "user:123",
    "exists": true,
    "ttl_ms": 3540000,
    "tags": ["user", "profile"],
    "size_bytes": 45,
    "created_at": "2024-01-15T10:30:00Z",
    "accessed_at": "2024-01-15T10:35:00Z",
    "access_count": 15
  }
}
```

### List Tags

Get all tags currently in use.

```http
GET /api/tags
```

**Response:**

```json
{
  "success": true,
  "data": {
    "tags": [
      {
        "name": "user",
        "key_count": 1250,
        "total_size_bytes": 125000
      },
      {
        "name": "session",
        "key_count": 500,
        "total_size_bytes": 50000
      }
    ]
  }
}
```

### Server Statistics

Get comprehensive server statistics.

```http
GET /api/stats
```

**Response:**

```json
{
  "success": true,
  "data": {
    "version": "1.0.8",
    "uptime_seconds": 86400,
    "cache": {
      "total_keys": 12500,
      "total_memory_bytes": 524288000,
      "total_tags": 50,
      "hit_rate": 0.945,
      "miss_rate": 0.055
    },
    "performance": {
      "operations_per_second": 12543,
      "avg_response_time_ms": 0.25,
      "total_operations": 1084320,
      "total_hits": 1024602,
      "total_misses": 59718
    },
    "memory": {
      "used_bytes": 524288000,
      "available_bytes": 549755813888,
      "usage_percent": 0.095
    },
    "connections": {
      "active_http": 25,
      "active_tcp": 10,
      "total_http": 5420,
      "total_tcp": 1250
    }
  }
}
```

### Health Check

Simple health check endpoint.

```http
GET /api/health
```

**Response:**

```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

## TCP Protocol

The TCP protocol provides ultra-low latency operations using a binary format. It runs on port 1984 (configurable).

### Connection

```python
import socket

# Connect to TagCache
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('localhost', 1984))
```

### Protocol Format

All TCP messages follow this binary format:

```
[Length: 4 bytes][Command: 1 byte][Data: variable]
```

- **Length**: Total message length (big-endian uint32)
- **Command**: Operation code (uint8)
- **Data**: Command-specific payload

### Command Codes

| Command | Code | Description |
|---------|------|-------------|
| AUTH    | 0x01 | Authenticate |
| SET     | 0x02 | Set key-value |
| GET     | 0x03 | Get value |
| DELETE  | 0x04 | Delete key |
| EXISTS  | 0x05 | Check if key exists |
| ADD     | 0x06 | Add if not exists |
| INCR    | 0x07 | Increment |
| DECR    | 0x08 | Decrement |
| STATS   | 0x09 | Get statistics |
| PING    | 0x0A | Ping server |

### Authentication

```python
import struct

def auth(sock, username, password):
    # Prepare auth payload
    auth_data = f"{username}:{password}".encode('utf-8')
    
    # Create message: [length][command][data]
    message = struct.pack('>I', len(auth_data) + 1) + b'\x01' + auth_data
    
    sock.send(message)
    
    # Read response
    response_len = struct.unpack('>I', sock.recv(4))[0]
    response = sock.recv(response_len)
    
    return response[0] == 0x01  # Success if first byte is 0x01
```

### Set Operation

```python
def set_key(sock, key, value, ttl_ms=0, tags=None):
    # Prepare payload
    key_bytes = key.encode('utf-8')
    value_bytes = value.encode('utf-8')
    tags_bytes = ','.join(tags or []).encode('utf-8')
    
    # Format: [key_len][key][value_len][value][ttl][tags_len][tags]
    payload = (
        struct.pack('>H', len(key_bytes)) + key_bytes +
        struct.pack('>I', len(value_bytes)) + value_bytes +
        struct.pack('>Q', ttl_ms) +
        struct.pack('>H', len(tags_bytes)) + tags_bytes
    )
    
    # Create message
    message = struct.pack('>I', len(payload) + 1) + b'\x02' + payload
    
    sock.send(message)
    
    # Read response
    response_len = struct.unpack('>I', sock.recv(4))[0]
    response = sock.recv(response_len)
    
    return response[0] == 0x01  # Success
```

### Get Operation

```python
def get_key(sock, key):
    # Prepare payload
    key_bytes = key.encode('utf-8')
    payload = struct.pack('>H', len(key_bytes)) + key_bytes
    
    # Create message
    message = struct.pack('>I', len(payload) + 1) + b'\x03' + payload
    
    sock.send(message)
    
    # Read response
    response_len = struct.unpack('>I', sock.recv(4))[0]
    response = sock.recv(response_len)
    
    if response[0] == 0x01:  # Success
        # Parse response: [status][value_len][value][ttl][tags_len][tags]
        offset = 1
        value_len = struct.unpack('>I', response[offset:offset+4])[0]
        offset += 4
        value = response[offset:offset+value_len].decode('utf-8')
        offset += value_len
        ttl = struct.unpack('>Q', response[offset:offset+8])[0]
        offset += 8
        tags_len = struct.unpack('>H', response[offset:offset+2])[0]
        offset += 2
        tags = response[offset:offset+tags_len].decode('utf-8').split(',') if tags_len > 0 else []
        
        return {
            'value': value,
            'ttl_ms': ttl,
            'tags': tags
        }
    
    return None  # Not found or error
```

## Client Libraries

### Python

```python
import tagcache

# HTTP client
client = tagcache.HTTPClient('http://localhost:8080', username='admin', password='password')

# Set a value
client.set('user:123', '{"name":"John"}', ttl_ms=3600000, tags=['user'])

# Get a value
value = client.get('user:123')

# Atomic increment
new_value = client.increment('counter', by=5)

# TCP client (for high performance)
tcp_client = tagcache.TCPClient('localhost', 1984, username='admin', password='password')
tcp_client.set('fast:key', 'value')
```

### JavaScript/Node.js

```javascript
const TagCache = require('tagcache-client');

// HTTP client
const client = new TagCache.HTTPClient({
  url: 'http://localhost:8080',
  username: 'admin',
  password: 'password'
});

// Set a value
await client.set('user:123', {name: 'John'}, {
  ttl_ms: 3600000,
  tags: ['user']
});

// Get a value
const value = await client.get('user:123');

// Atomic operations
const newValue = await client.increment('counter', {by: 5});
```

### Go

```go
package main

import (
    "github.com/aminshamim/tagcache-go"
)

func main() {
    // HTTP client
    client := tagcache.NewHTTPClient("http://localhost:8080", "admin", "password")
    
    // Set a value
    err := client.Set("user:123", `{"name":"John"}`, tagcache.SetOptions{
        TTL: 3600000,
        Tags: []string{"user"},
    })
    
    // Get a value
    value, err := client.Get("user:123")
    
    // Atomic increment
    newValue, err := client.Increment("counter", 5)
}
```

## Error Codes

### HTTP Status Codes

- `200 OK`: Success
- `400 Bad Request`: Invalid request format
- `401 Unauthorized`: Authentication failed
- `404 Not Found`: Key not found
- `409 Conflict`: Key already exists (for ADD operations)
- `500 Internal Server Error`: Server error

### TCP Response Codes

- `0x01`: Success
- `0x02`: Key not found
- `0x03`: Authentication failed
- `0x04`: Invalid command
- `0x05`: Server error
- `0x06`: Key already exists

## Performance Considerations

### HTTP API

- Use HTTP/1.1 keep-alive for connection reuse
- Batch multiple operations when possible
- Consider TCP protocol for high-frequency operations

### TCP Protocol

- Maintain persistent connections
- Use connection pooling for concurrent applications
- Implement proper error handling and reconnection logic

### General Tips

- Use appropriate TTL values to prevent memory bloat
- Leverage tags for efficient batch operations
- Monitor cache hit rates and adjust strategies accordingly

## Next Steps

{{< cards >}}
  {{< card link="../examples" title="Examples" icon="code" subtitle="Real-world usage examples and patterns" >}}
  {{< card link="../performance" title="Performance" icon="chart-bar" subtitle="Benchmarking and optimization guides" >}}
  {{< card link="../deployment" title="Deployment" icon="server" subtitle="Production deployment strategies" >}}
{{< /cards >}}

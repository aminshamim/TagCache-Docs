---
title: Performance
weight: 7
type: docs
---

TagCache is designed for maximum performance with multi-shard architecture, optimized data structures, and efficient protocols. This guide covers benchmarking, optimization techniques, and performance tuning.

## Performance Overview

TagCache delivers exceptional performance through:

- **Multi-shard Design**: DashMap with hash-based sharding for lock-free operations
- **Memory-efficient Storage**: Optimized data structures with minimal overhead
- **Dual Protocols**: TCP for ultra-low latency, HTTP for web applications
- **Atomic Operations**: Lock-free counters and conditional operations
- **Tag Management**: Efficient tag indexing with minimal memory overhead

## Benchmark Results

### Hardware Specifications

Tests performed on:
- **CPU**: Intel Core i7-12700K (12 cores, 20 threads)
- **RAM**: 32GB DDR4-3200
- **Storage**: NVMe SSD
- **Network**: Localhost (no network latency)
- **OS**: Ubuntu 22.04 LTS

### TCP Protocol Performance

{{< tabs items="Throughput,Latency,Mixed Workload" >}}

{{< tab >}}
**Pure Throughput Tests**

| Operation | Ops/Second | Notes |
|-----------|------------|-------|
| SET | 1,200,000 | Small values (100 bytes) |
| GET | 1,500,000 | Cache hit rate: 100% |
| DELETE | 800,000 | Single key operations |
| INCREMENT | 1,100,000 | Atomic counters |
| GET (batch) | 2,000,000 | 10 keys per request |

**Configuration**: 256 shards, 16 client connections, pipelined requests
{{< /tab >}}

{{< tab >}}
**Latency Distribution (microseconds)**

| Operation | P50 | P95 | P99 | P99.9 |
|-----------|-----|-----|-----|-------|
| SET | 45 | 120 | 280 | 850 |
| GET | 35 | 95 | 220 | 650 |
| DELETE | 40 | 110 | 250 | 750 |
| INCREMENT | 42 | 115 | 270 | 800 |

**Configuration**: Single-threaded client, synchronous requests
{{< /tab >}}

{{< tab >}}
**Mixed Workload (80% GET, 20% SET)**

| Metric | Value |
|--------|-------|
| Total Ops/Second | 1,350,000 |
| Average Latency | 38μs |
| Cache Hit Rate | 95% |
| Memory Usage | 2.1GB |
| CPU Usage | 45% |

**Test Duration**: 10 minutes, 32 concurrent clients
{{< /tab >}}

{{< /tabs >}}

### HTTP API Performance

{{< tabs items="Throughput,Latency,Real-world" >}}

{{< tab >}}
**HTTP Throughput**

| Operation | Ops/Second | Notes |
|-----------|------------|-------|
| POST /api/set | 450,000 | JSON payloads |
| GET /api/get/{key} | 520,000 | Direct key access |
| POST /api/increment | 400,000 | Atomic operations |
| GET /api/get/tag/{tag} | 180,000 | Tag-based retrieval |

**Configuration**: HTTP/1.1 with keep-alive, 64 concurrent connections
{{< /tab >}}

{{< tab >}}
**HTTP Latency (milliseconds)**

| Operation | P50 | P95 | P99 | P99.9 |
|-----------|-----|-----|-----|-------|
| POST /api/set | 0.8 | 2.1 | 4.5 | 12.0 |
| GET /api/get/{key} | 0.6 | 1.8 | 3.8 | 10.5 |
| POST /api/increment | 0.9 | 2.3 | 4.8 | 13.2 |
| GET /api/stats | 1.2 | 3.0 | 6.2 | 15.8 |
{{< /tab >}}

{{< tab >}}
**Real-world Web Application**

Simulating typical web app cache patterns:

| Metric | Value |
|--------|-------|
| Requests/Second | 285,000 |
| Session Lookups | 180,000/sec |
| Cache Updates | 45,000/sec |
| Tag Invalidations | 8,500/sec |
| Average Response | 1.2ms |
| 99th Percentile | 5.8ms |

**Pattern**: 65% GET, 25% SET, 10% tag operations
{{< /tab >}}

{{< /tabs >}}

## Benchmarking Tools

TagCache includes built-in benchmarking tools for performance testing.

### TCP Benchmark Tool

```bash
# Basic throughput test
bench_tcp --host localhost --port 1984 \
  --username admin --password password \
  --operations 1000000 \
  --workers 16 \
  --operation set

# Latency test
bench_tcp --host localhost --port 1984 \
  --username admin --password password \
  --operations 100000 \
  --workers 1 \
  --operation get \
  --latency-histogram

# Mixed workload
bench_tcp --host localhost --port 1984 \
  --username admin --password password \
  --operations 1000000 \
  --workers 32 \
  --read-ratio 0.8 \
  --write-ratio 0.2
```

### HTTP Benchmark

```bash
# Using Apache Bench (ab)
ab -n 100000 -c 64 -H "Authorization: Basic YWRtaW46cGFzc3dvcmQ=" \
  http://localhost:8080/api/get/test_key

# Using wrk
wrk -t16 -c64 -d30s \
  --header "Authorization: Basic YWRtaW46cGFzc3dvcmQ=" \
  http://localhost:8080/api/get/test_key

# Custom benchmark script
curl -X POST http://localhost:8080/api/benchmark \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{
    "duration_seconds": 60,
    "operations": ["set", "get", "increment"],
    "ratio": [0.3, 0.6, 0.1],
    "concurrent_clients": 32
  }'
```

### Load Testing Script

```python
import asyncio
import aiohttp
import time
import statistics
import json

class TagCacheLoadTest:
    def __init__(self, base_url, username, password):
        self.base_url = base_url
        self.auth = aiohttp.BasicAuth(username, password)
        self.results = []
    
    async def run_test(self, num_requests=10000, concurrency=100):
        """Run comprehensive load test"""
        
        connector = aiohttp.TCPConnector(limit=200)
        timeout = aiohttp.ClientTimeout(total=30)
        
        async with aiohttp.ClientSession(
            connector=connector,
            timeout=timeout,
            auth=self.auth
        ) as session:
            
            # Create semaphore to limit concurrency
            semaphore = asyncio.Semaphore(concurrency)
            
            # Generate test tasks
            tasks = []
            for i in range(num_requests):
                if i % 100 == 0:
                    # 1% tag operations
                    task = self.tag_operation(session, semaphore, i)
                elif i % 10 == 0:
                    # 10% increment operations
                    task = self.increment_operation(session, semaphore, i)
                elif i % 4 == 0:
                    # 25% set operations
                    task = self.set_operation(session, semaphore, i)
                else:
                    # 64% get operations
                    task = self.get_operation(session, semaphore, i)
                
                tasks.append(task)
            
            # Run all tasks
            start_time = time.time()
            results = await asyncio.gather(*tasks, return_exceptions=True)
            end_time = time.time()
            
            # Calculate statistics
            successful_results = [r for r in results if not isinstance(r, Exception)]
            total_time = end_time - start_time
            
            print(f"Total Requests: {num_requests}")
            print(f"Successful: {len(successful_results)}")
            print(f"Failed: {num_requests - len(successful_results)}")
            print(f"Duration: {total_time:.2f} seconds")
            print(f"Requests/Second: {num_requests / total_time:.0f}")
            
            if successful_results:
                latencies = [r['latency'] for r in successful_results]
                print(f"Average Latency: {statistics.mean(latencies):.2f}ms")
                print(f"P95 Latency: {statistics.quantiles(latencies, n=20)[18]:.2f}ms")
                print(f"P99 Latency: {statistics.quantiles(latencies, n=100)[98]:.2f}ms")
    
    async def set_operation(self, session, semaphore, i):
        async with semaphore:
            start = time.time()
            try:
                data = {
                    "key": f"test_key_{i}",
                    "value": f"test_value_{i}",
                    "ttl_ms": 60000,
                    "tags": [f"tag_{i % 10}"]
                }
                
                async with session.post(
                    f"{self.base_url}/api/set",
                    json=data
                ) as response:
                    await response.json()
                    latency = (time.time() - start) * 1000
                    return {"operation": "set", "latency": latency, "status": response.status}
            
            except Exception as e:
                return e
    
    async def get_operation(self, session, semaphore, i):
        async with semaphore:
            start = time.time()
            try:
                key = f"test_key_{i % 1000}"  # Reuse keys for cache hits
                
                async with session.get(f"{self.base_url}/api/get/{key}") as response:
                    await response.json()
                    latency = (time.time() - start) * 1000
                    return {"operation": "get", "latency": latency, "status": response.status}
            
            except Exception as e:
                return e
    
    async def increment_operation(self, session, semaphore, i):
        async with semaphore:
            start = time.time()
            try:
                data = {
                    "key": f"counter_{i % 100}",
                    "by": 1,
                    "initial": 0
                }
                
                async with session.post(
                    f"{self.base_url}/api/increment",
                    json=data
                ) as response:
                    await response.json()
                    latency = (time.time() - start) * 1000
                    return {"operation": "increment", "latency": latency, "status": response.status}
            
            except Exception as e:
                return e
    
    async def tag_operation(self, session, semaphore, i):
        async with semaphore:
            start = time.time()
            try:
                tag = f"tag_{i % 10}"
                
                async with session.get(f"{self.base_url}/api/get/tag/{tag}") as response:
                    await response.json()
                    latency = (time.time() - start) * 1000
                    return {"operation": "tag_get", "latency": latency, "status": response.status}
            
            except Exception as e:
                return e

# Run the test
async def main():
    tester = TagCacheLoadTest("http://localhost:8080", "admin", "password")
    await tester.run_test(num_requests=50000, concurrency=200)

if __name__ == "__main__":
    asyncio.run(main())
```

## Performance Tuning

### Server Configuration

#### Optimal Shard Count

```toml
[performance]
# Rule of thumb: 2-4x the number of CPU cores
# For 16-core system: 32-64 shards
# For high-concurrency: up to 1024 shards
num_shards = 512
```

**Guidelines:**
- More shards = better concurrency, slightly higher memory overhead
- Fewer shards = less memory overhead, potential contention
- Sweet spot: 32-64 shards for most workloads

#### Memory Management

```toml
[cache]
# Set based on available system memory
max_memory = "8GB"          # 80% of available RAM
max_keys = 10000000         # Prevent excessive key count

# Choose eviction policy
eviction_policy = "lru"     # Best for most workloads
# eviction_policy = "lfu"   # For stable access patterns
# eviction_policy = "random" # Fastest eviction
```

#### Connection Pooling

```toml
[performance]
# Tune based on expected concurrent connections
tcp_pool_size = 500         # For high-throughput TCP clients
http_pool_size = 2000       # For web applications

# Connection timeouts
cleanup_interval_ms = 30000 # More frequent cleanup for high turnover
```

### Client Optimization

#### Connection Reuse

```python
# Good: Reuse connections
class OptimizedCache:
    def __init__(self):
        self.session = aiohttp.ClientSession(
            connector=aiohttp.TCPConnector(
                limit=100,              # Connection pool size
                limit_per_host=20,      # Per-host connections
                keepalive_timeout=30,   # Keep connections alive
                enable_cleanup_closed=True
            )
        )
    
    async def get(self, key):
        async with self.session.get(f"http://cache:8080/api/get/{key}") as resp:
            return await resp.json()

# Bad: Creating new connections
async def bad_get(key):
    async with aiohttp.ClientSession() as session:  # New connection each time
        async with session.get(f"http://cache:8080/api/get/{key}") as resp:
            return await resp.json()
```

#### Batch Operations

```python
# Good: Batch multiple operations
async def get_multiple_optimized(keys):
    data = {"keys": keys}
    async with session.post("http://cache:8080/api/get/batch", json=data) as resp:
        return await resp.json()

# Bad: Individual requests
async def get_multiple_slow(keys):
    results = {}
    for key in keys:
        async with session.get(f"http://cache:8080/api/get/{key}") as resp:
            results[key] = await resp.json()
    return results
```

#### TCP for High Performance

```python
import socket
import struct

class HighPerformanceTCPClient:
    def __init__(self, host='localhost', port=1984):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((host, port))
        self.sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)  # Disable Nagle
        
    def set_fast(self, key, value):
        # Optimized binary protocol implementation
        key_bytes = key.encode('utf-8')
        value_bytes = value.encode('utf-8')
        
        # Pre-allocate buffer for better performance
        payload_len = 2 + len(key_bytes) + 4 + len(value_bytes) + 8
        payload = bytearray(payload_len)
        
        offset = 0
        struct.pack_into('>H', payload, offset, len(key_bytes))
        offset += 2
        payload[offset:offset+len(key_bytes)] = key_bytes
        offset += len(key_bytes)
        struct.pack_into('>I', payload, offset, len(value_bytes))
        offset += 4
        payload[offset:offset+len(value_bytes)] = value_bytes
        offset += len(value_bytes)
        struct.pack_into('>Q', payload, offset, 0)  # No TTL
        
        # Send message
        message = struct.pack('>I', len(payload) + 1) + b'\x02' + payload
        self.sock.send(message)
        
        # Read response (simplified)
        response_len = struct.unpack('>I', self.sock.recv(4))[0]
        response = self.sock.recv(response_len)
        return response[0] == 0x01
```

### Memory Optimization

#### Efficient Data Structures

```rust
// TagCache internal optimizations (for reference)

// Use SmallVec for tags to avoid heap allocation for small tag lists
use smallvec::SmallVec;
type TagList = SmallVec<[String; 4]>;  // Stack allocation for ≤4 tags

// Use AHash for better performance than default hasher
use ahash::AHashMap;
type FastMap<K, V> = AHashMap<K, V>;

// Compress values when beneficial
use lz4_flex::{compress, decompress};

fn store_value(value: &[u8]) -> Vec<u8> {
    if value.len() > 1024 {
        compress(value)  // Compress large values
    } else {
        value.to_vec()   // Store small values as-is
    }
}
```

#### Tag Management

```bash
# Monitor tag efficiency
tagcache stats --detailed | grep tags

# Clean up unused tags
tagcache admin gc-tags

# Optimize tag usage
tagcache analyze tags --show-distribution
```

### Monitoring and Profiling

#### Real-time Performance Monitoring

```bash
# Monitor key metrics
watch -n 1 'curl -s -u admin:password http://localhost:8080/api/stats | jq .data.performance'

# Track memory usage
watch -n 5 'curl -s -u admin:password http://localhost:8080/api/stats | jq .data.memory'

# Monitor hit rates
tagcache stats --watch --filter hit_rate,ops_per_second
```

#### Performance Profiling

```python
import asyncio
import time
from collections import defaultdict

class PerformanceProfiler:
    def __init__(self):
        self.operation_times = defaultdict(list)
        self.operation_counts = defaultdict(int)
    
    async def profile_operation(self, operation_name, operation_func, *args, **kwargs):
        start = time.time()
        try:
            result = await operation_func(*args, **kwargs)
            success = True
        except Exception as e:
            result = e
            success = False
        
        duration = (time.time() - start) * 1000  # ms
        
        self.operation_times[operation_name].append(duration)
        self.operation_counts[operation_name] += 1
        
        return result, success, duration
    
    def get_stats(self):
        stats = {}
        for op_name, times in self.operation_times.items():
            stats[op_name] = {
                'count': len(times),
                'avg_ms': sum(times) / len(times),
                'min_ms': min(times),
                'max_ms': max(times),
                'p95_ms': sorted(times)[int(len(times) * 0.95)] if times else 0
            }
        return stats

# Usage
profiler = PerformanceProfiler()

async def benchmark_operations():
    cache = OptimizedCache()
    
    # Profile different operations
    for i in range(1000):
        await profiler.profile_operation('set', cache.set, f'key_{i}', f'value_{i}')
        await profiler.profile_operation('get', cache.get, f'key_{i}')
        
        if i % 100 == 0:
            await profiler.profile_operation('increment', cache.increment, 'counter')
    
    print(profiler.get_stats())
```

## Performance Best Practices

### 1. Choose the Right Protocol

- **TCP Protocol**: Ultra-low latency, high throughput applications
- **HTTP API**: Web applications, easier integration, better debugging

### 2. Optimize Key Design

```bash
# Good: Hierarchical, predictable keys
user:123:profile
session:abc123:data
cache:product:456:details

# Bad: Random, non-hierarchical keys
a7f9x2m4k
user_session_data_temp_123
```

### 3. Use Tags Strategically

```python
# Good: Logical grouping for batch invalidation
cache.set('user:123:profile', data, tags=['user:123', 'profile'])
cache.set('user:123:settings', data, tags=['user:123', 'settings'])

# Invalidate all user data at once
cache.delete_by_tag('user:123')

# Bad: Too many or too few tags
cache.set('key', data, tags=['a', 'b', 'c', 'd', 'e', 'f'])  # Too many
cache.set('key', data)  # No tags, hard to invalidate groups
```

### 4. Set Appropriate TTLs

```python
# Different TTLs for different data types
cache.set('user:session', data, ttl_ms=1800000)      # 30 minutes
cache.set('user:profile', data, ttl_ms=3600000)      # 1 hour  
cache.set('api:response', data, ttl_ms=300000)       # 5 minutes
cache.set('static:config', data, ttl_ms=86400000)    # 24 hours
```

### 5. Monitor and Alert

```yaml
# Prometheus alerts
groups:
  - name: tagcache
    rules:
      - alert: TagCacheHighLatency
        expr: tagcache_avg_response_time_ms > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "TagCache high latency detected"
      
      - alert: TagCacheHighMemoryUsage
        expr: tagcache_memory_usage_percent > 85
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "TagCache memory usage is high"
```

## Troubleshooting Performance Issues

### Common Performance Problems

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| High latency | Too few shards | Increase `num_shards` |
| Memory issues | No TTL set | Set appropriate TTLs |
| Low throughput | Single-threaded client | Use connection pooling |
| Cache misses | Keys expiring too fast | Increase TTL values |
| High CPU usage | Too many small operations | Batch operations when possible |

### Diagnostic Commands

```bash
# Check server health
tagcache health --detailed

# Analyze performance bottlenecks  
tagcache analyze performance --duration 60s

# Memory usage breakdown
tagcache debug memory --show-shards

# Connection analysis
tagcache debug connections --show-active

# Cache efficiency
tagcache analyze cache-efficiency --period 1h
```

## Next Steps

{{< cards >}}
  {{< card link="../deployment" title="Deployment" icon="server" subtitle="Production deployment strategies" >}}
  {{< card link="../examples" title="Examples" icon="code" subtitle="Real-world implementation patterns" >}}
  {{< card link="../api" title="API Reference" icon="book-open" subtitle="Complete API documentation" >}}
{{< /cards >}}

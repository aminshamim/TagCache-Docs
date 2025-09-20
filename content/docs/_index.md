---
title: Documentation
type: docs
sidebar:
  open: true
---

Welcome to the comprehensive documentation for TagCache, a lightning-fast, tag-aware in-memory cache server written in Rust.

## Overview

TagCache is designed for high-performance applications that need:
- **Ultra-fast data access** with sub-millisecond latency
- **Tag-based cache invalidation** for complex data relationships
- **Dual protocol support** (TCP and HTTP) for different use cases
- **Production-ready features** including authentication and monitoring

## Documentation Structure

{{< cards >}}
  {{< card link="getting-started" title="Getting Started" icon="play" subtitle="Quick installation and first steps" >}}
  {{< card link="installation" title="Installation" icon="download" subtitle="Detailed installation methods" >}}
  {{< card link="configuration" title="Configuration" icon="cog" subtitle="Server configuration options" >}}
  {{< card link="cli" title="CLI Reference" icon="terminal" subtitle="Command-line interface guide" >}}
{{< /cards >}}

{{< cards >}}
  {{< card link="api" title="API Reference" icon="book-open" subtitle="HTTP and TCP API documentation" >}}
  {{< card link="examples" title="Examples" icon="code" subtitle="Real-world usage examples" >}}
  {{< card link="performance" title="Performance" icon="chart-bar" subtitle="Benchmarks and optimization" >}}
  {{< card link="deployment" title="Deployment" icon="server" subtitle="Production deployment guides" >}}
{{< /cards >}}

## Key Features

- **Multi-shard Architecture**: DashMap with hash sharding for maximum concurrency
- **Tag-based Invalidation**: Organize and clear related data efficiently
- **Atomic Operations**: ADD, INCR, DECR with race-condition protection
- **Flexible TTL**: Precise expiration control in milliseconds or seconds
- **Cross-platform**: Native support for macOS, Linux, and Windows
- **Web Dashboard**: Beautiful React UI for monitoring and management

## Quick Example

```bash
# Start TagCache server
tagcache server

# Set a value with tags
tagcache put "user:123" '{"name": "John", "email": "john@example.com"}' --tags "user,profile"

# Get the value
tagcache get key "user:123"

# Invalidate all user-related cache
tagcache delete tag "user"
```

## Community and Support

- **GitHub**: [aminshamim/tagcache](https://github.com/aminshamim/tagcache)
- **Issues**: Report bugs and request features on GitHub
- **Discussions**: Join the community discussions

{{< callout type="info" >}}
TagCache is actively maintained and production-ready. For enterprise support or custom development, please reach out through GitHub.
{{< /callout >}}

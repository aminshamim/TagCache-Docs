---
title: Installation
weight: 2
type: docs
---

TagCache supports multiple installation methods across different platforms. Choose the method that best fits your environment.

## Pre-built Binaries (Recommended)

Download the latest binaries from our [GitHub Releases](https://github.com/aminshamim/tagcache/releases/latest) page.

### macOS

{{< tabs items="Apple Silicon (M1/M2/M3),Intel" >}}

{{< tab >}}
```bash
# Download for Apple Silicon (ARM64)
curl -L -o tagcache-macos-arm64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-macos-arm64.tar.gz

# Extract
tar xzf tagcache-macos-arm64.tar.gz

# Install globally
sudo cp tagcache bench_tcp /usr/local/bin/

# Verify installation
tagcache --version
```
{{< /tab >}}

{{< tab >}}
```bash
# Download for Intel (x86_64)
curl -L -o tagcache-macos-x86_64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-macos-x86_64.tar.gz

# Extract
tar xzf tagcache-macos-x86_64.tar.gz

# Install globally
sudo cp tagcache bench_tcp /usr/local/bin/

# Verify installation
tagcache --version
```
{{< /tab >}}

{{< /tabs >}}

### Linux

{{< tabs items="x86_64,ARM64" >}}

{{< tab >}}
```bash
# Download for Linux x86_64
curl -L -o tagcache-linux-x86_64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-linux-x86_64.tar.gz

# Extract
tar xzf tagcache-linux-x86_64.tar.gz

# Install globally
sudo cp tagcache bench_tcp /usr/local/bin/

# Verify installation
tagcache --version
```
{{< /tab >}}

{{< tab >}}
```bash
# Download for Linux ARM64
curl -L -o tagcache-linux-arm64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-linux-arm64.tar.gz

# Extract
tar xzf tagcache-linux-arm64.tar.gz

# Install globally
sudo cp tagcache bench_tcp /usr/local/bin/

# Verify installation
tagcache --version
```
{{< /tab >}}

{{< /tabs >}}

### Windows

{{< tabs items="x86_64,ARM64" >}}

{{< tab >}}
```powershell
# Download for Windows x86_64
Invoke-WebRequest -Uri "https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-windows-x86_64.zip" -OutFile "tagcache-windows-x86_64.zip"

# Extract (Windows 10+)
Expand-Archive -Path "tagcache-windows-x86_64.zip" -DestinationPath "C:\tagcache"

# Add to PATH (optional)
$env:PATH += ";C:\tagcache"

# Verify installation
tagcache --version
```
{{< /tab >}}

{{< tab >}}
```powershell
# Download for Windows ARM64
Invoke-WebRequest -Uri "https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-windows-arm64.zip" -OutFile "tagcache-windows-arm64.zip"

# Extract (Windows 10+)
Expand-Archive -Path "tagcache-windows-arm64.zip" -DestinationPath "C:\tagcache"

# Add to PATH (optional)
$env:PATH += ";C:\tagcache"

# Verify installation
tagcache --version
```
{{< /tab >}}

{{< /tabs >}}

## Package Managers

### Homebrew (macOS/Linux)

```bash
# Add the TagCache tap
brew tap aminshamim/tap

# Install TagCache
brew install tagcache

# Verify installation
tagcache --version

# Start service (optional)
brew services start tagcache
```

### Debian/Ubuntu (APT)

```bash
# Download the .deb package
curl -L -o tagcache.deb \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache_1.0.8_amd64.deb

# Install
sudo dpkg -i tagcache.deb

# Fix dependencies if needed
sudo apt-get install -f

# Enable and start service
sudo systemctl enable tagcache
sudo systemctl start tagcache

# Check status
sudo systemctl status tagcache
```

## Docker

### Docker Run

```bash
# Basic run with default settings
docker run -d \
  --name tagcache \
  -p 1984:1984 \
  -p 8080:8080 \
  aminshamim/tagcache

# With custom configuration
docker run -d \
  --name tagcache \
  -p 1984:1984 \
  -p 8080:8080 \
  -v /path/to/your/tagcache.conf:/etc/tagcache/tagcache.conf:ro \
  aminshamim/tagcache

# With environment variables
docker run -d \
  --name tagcache \
  -p 1984:1984 \
  -p 8080:8080 \
  -e TAGCACHE_AUTH_USERNAME=myuser \
  -e TAGCACHE_AUTH_PASSWORD=mypassword \
  aminshamim/tagcache
```

### Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  tagcache:
    image: aminshamim/tagcache:latest
    container_name: tagcache
    ports:
      - "1984:1984"  # TCP port
      - "8080:8080"  # HTTP port
    environment:
      - TAGCACHE_AUTH_USERNAME=admin
      - TAGCACHE_AUTH_PASSWORD=secure-password
    volumes:
      - ./tagcache.conf:/etc/tagcache/tagcache.conf:ro
      - tagcache_data:/var/lib/tagcache
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  tagcache_data:
```

Run with Docker Compose:

```bash
docker-compose up -d
```

## Building from Source

### Prerequisites

- [Rust](https://rustup.rs/) 1.70 or later
- Git

### Build Steps

```bash
# Clone the repository
git clone https://github.com/aminshamim/tagcache.git
cd tagcache

# Build release version
cargo build --release

# The binary will be available at
./target/release/tagcache

# Optionally, install globally
cargo install --path .
```

### Build with UI Embedded

```bash
# Build with embedded web UI (default)
cargo build --release --features embed-ui

# Build without embedded UI (smaller binary)
cargo build --release --no-default-features
```

### Cross-compilation

TagCache supports cross-compilation for different architectures:

```bash
# Install cross-compilation tool
cargo install cross

# Build for different targets
cross build --release --target x86_64-unknown-linux-gnu
cross build --release --target aarch64-unknown-linux-gnu
cross build --release --target x86_64-pc-windows-gnu
```

## Verification

After installation, verify TagCache is working correctly:

```bash
# Check version
tagcache --version

# Start server in background
tagcache server &

# Wait a moment for startup
sleep 2

# Test basic functionality
tagcache --username admin --password password put "test" "hello"
tagcache --username admin --password password get key "test"

# Stop the server
pkill tagcache
```

## Next Steps

After successful installation:

{{< cards >}}
  {{< card link="../configuration" title="Configuration" icon="cog" subtitle="Configure TagCache for your environment" >}}
  {{< card link="../getting-started" title="Getting Started" icon="play" subtitle="Learn basic usage and operations" >}}
  {{< card link="../deployment" title="Deployment" icon="server" subtitle="Production deployment guides" >}}
{{< /cards >}}

## Troubleshooting

### Common Issues

**Permission Denied**
```bash
# Make sure the binary is executable
chmod +x tagcache
```

**Port Already in Use**
```bash
# Check what's using the ports
lsof -i :8080
lsof -i :1984

# Kill processes if needed
sudo pkill -f tagcache
```

**Configuration File Not Found**
```bash
# Create default config directory
sudo mkdir -p /etc/tagcache

# Generate sample config
tagcache config generate > /etc/tagcache/tagcache.conf
```

### Getting Help

- Check the [GitHub Issues](https://github.com/aminshamim/tagcache/issues)
- Review the [Configuration Guide](../configuration)
- Join our community discussions

{{< callout type="info" >}}
For enterprise support or custom installations, please contact us through GitHub.
{{< /callout >}}

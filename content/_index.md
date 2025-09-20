---
title: TagCache - High-Performance Tagged Caching System
layout: hextra-home
description: Ultra-fast tag-driven cache invalidation with dual protocol support. 1M+ ops/s throughput, <0.8ms P95 latency. Written in Rust for maximum performance.
keywords: cache, caching, tag invalidation, rust, high performance, atomic operations, redis alternative, memcached alternative, distributed cache, microservices cache, backend cache, in-memory cache, performance optimization

# SEO Configuration
noindex: false
robots: index, follow
canonical: https://tagcache.io/
---

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "TagCache",
  "applicationCategory": "DeveloperApplication",
  "operatingSystem": "Linux, macOS, Windows",
  "description": "High-performance, tag-aware in-memory cache server written in Rust with dual protocol support (HTTP/TCP) and atomic operations. Delivers 1M+ ops/sec throughput with sub-millisecond latency.",
  "url": "https://tagcache.io",
  "downloadUrl": "https://github.com/aminshamim/tagcache/releases",
  "codeRepository": "https://github.com/aminshamim/tagcache",
  "programmingLanguage": "Rust",
  "license": "https://github.com/aminshamim/tagcache/blob/main/LICENSE",
  "version": "1.0.8",
  "releaseNotes": "https://github.com/aminshamim/tagcache/releases/tag/v1.0.8",
  "author": {
    "@type": "Person",
    "name": "Amin Shamim",
    "url": "https://github.com/aminshamim"
  },
  "maintainer": {
    "@type": "Person",
    "name": "Amin Shamim",
    "url": "https://github.com/aminshamim"
  },
  "offers": {
    "@type": "Offer",
    "price": "0",
    "priceCurrency": "USD"
  },
  "applicationSubCategory": "Cache Server",
  "featureList": [
    "Tag-based cache invalidation",
    "Dual protocol support (HTTP/TCP)",
    "Atomic operations (INCR/DECR/ADD)",
    "Multi-shard architecture",
    "Sub-millisecond latency",
    "1M+ operations per second",
    "Built-in observability",
    "Cross-platform support"
  ],
  "requirements": "Rust runtime environment",
  "installUrl": "https://tagcache.io/docs/installation",
  "screenshot": "https://tagcache.io/images/dashboard.png",
  "softwareHelp": {
    "@type": "CreativeWork",
    "url": "https://tagcache.io/docs"
  },
  "discussionUrl": "https://github.com/aminshamim/tagcache/discussions",
  "issueTracker": "https://github.com/aminshamim/tagcache/issues"
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "TechArticle",
  "headline": "TagCache - Ultra-Fast Tag-Based Cache for Modern Applications",
  "description": "Complete guide to TagCache, a high-performance Rust-based caching solution with tag invalidation, atomic operations, and dual protocol support.",
  "author": {
    "@type": "Person",
    "name": "Amin Shamim"
  },
  "datePublished": "2024-01-01",
  "dateModified": "2024-09-20",
  "publisher": {
    "@type": "Organization",
    "name": "TagCache",
    "url": "https://tagcache.io"
  },
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://tagcache.io"
  },
  "about": [
    {
      "@type": "Thing",
      "name": "Cache Systems"
    },
    {
      "@type": "Thing",
      "name": "Rust Programming"
    },
    {
      "@type": "Thing",
      "name": "Performance Optimization"
    },
    {
      "@type": "Thing",
      "name": "Distributed Systems"
    }
  ]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "TagCache",
  "url": "https://tagcache.io",
  "logo": "https://tagcache.io/images/logo.png",
  "description": "Open source high-performance caching solution with tag-based invalidation",
  "sameAs": [
    "https://github.com/aminshamim/tagcache"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "contactType": "technical support",
    "url": "https://github.com/aminshamim/tagcache/issues"
  }
}
</script>

<div class="homepage-container" role="main">

<!-- HERO SECTION WITH ANIMATED BACKGROUND -->
<section class="tc-hero-section" aria-label="Hero">
  <div class="tc-hero-bg">
    <div class="tc-hero-gradient"></div>
    <div class="tc-hero-particles" aria-hidden="true"></div>
    <div class="tc-hero-mesh" aria-hidden="true"></div>
  </div>
  <div class="tc-hero-content">
    <div class="tc-badge-wrapper">
      <span class="tc-badge-new" role="status">🦀 Built with Rust</span>
      <span class="tc-badge-stars" aria-label="Open Source Project">
        <svg class="tc-icon" viewBox="0 0 16 16" width="14" height="14" aria-hidden="true">
          <path fill="currentColor" d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/>
        </svg>
        Open Source
      </span>
    </div>
    <h1 class="tc-hero-title">
      <span class="tc-title-line">The Fastest</span>
      <span class="tc-title-gradient">Tag-Based Cache</span>
      <span class="tc-title-line">Ever Built</span>
    </h1>
    <p class="tc-hero-description">
      Surgical cache invalidation through intelligent tagging.<br class="tc-hero-break"/>
      Atomic operations at wire speed for distributed systems.<br class="tc-hero-break"/>
      <strong>1M+ ops/sec</strong> throughput with <strong>&lt;0.8ms</strong> P95 latency.
    </p>
    <div class="tc-hero-actions">
      <a href="/docs/getting-started" class="tc-btn tc-btn-hero-primary" aria-label="Get started with TagCache">
        <span>Get Started</span>
        <svg class="tc-btn-arrow" viewBox="0 0 20 20" width="16" height="16" aria-hidden="true">
          <path fill="currentColor" d="M10.293 3.293a1 1 0 011.414 0l6 6a1 1 0 010 1.414l-6 6a1 1 0 01-1.414-1.414L14.586 11H3a1 1 0 110-2h11.586l-4.293-4.293a1 1 0 010-1.414z"/>
        </svg>
      </a>
      <a href="https://github.com/aminshamim/tagcache" class="tc-btn tc-btn-hero-secondary" target="_blank" rel="noopener noreferrer" aria-label="View TagCache on GitHub">
        <svg class="tc-btn-icon" viewBox="0 0 16 16" width="16" height="16" aria-hidden="true">
          <path fill="currentColor" d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/>
        </svg>
        <span>Star on GitHub</span>
      </a>
      <a href="/docs/performance" class="tc-btn tc-btn-hero-ghost" aria-label="View performance benchmarks">
        <span>View Benchmarks</span>
      </a>
    </div>
  </div>
</section>

<!-- PERFORMANCE METRICS -->
<section class="tc-metrics" aria-label="Performance metrics">
  <div class="tc-metrics-inner">
    <h2 class="tc-section-title sr-only">Performance Metrics</h2>
    <div class="tc-metrics-grid">
      <div class="tc-metric">
        <div class="tc-metric-value" aria-describedby="metric-throughput">
          <span class="tc-metric-number" data-value="1M+">1M+</span>
          <span class="tc-metric-unit">ops/sec</span>
        </div>
        <h3 class="tc-metric-label" id="metric-throughput">TCP Throughput</h3>
      </div>
      <div class="tc-metric">
        <div class="tc-metric-value" aria-describedby="metric-latency">
          <span class="tc-metric-number" data-value="0.8">&lt;0.8</span>
          <span class="tc-metric-unit">ms</span>
        </div>
        <h3 class="tc-metric-label" id="metric-latency">P95 Latency</h3>
      </div>
      <div class="tc-metric">
        <div class="tc-metric-value" aria-describedby="metric-coldstart">
          <span class="tc-metric-number" data-value="40">&lt;40</span>
          <span class="tc-metric-unit">ms</span>
        </div>
        <h3 class="tc-metric-label" id="metric-coldstart">Cold Start</h3>
      </div>
      <div class="tc-metric">
        <div class="tc-metric-value" aria-describedby="metric-memory">
          <span class="tc-metric-number" data-value="4">&lt;4</span>
          <span class="tc-metric-unit">%</span>
        </div>
        <h3 class="tc-metric-label" id="metric-memory">Memory Overhead</h3>
      </div>
    </div>
  </div>
</section>

<!-- KEY FEATURES -->
<section class="tc-features" aria-label="Key features">
  <div class="tc-features-inner">
    <div class="tc-section-header">
      <h2 class="tc-section-title">Why Development Teams Choose TagCache</h2>
      <p class="tc-section-subtitle">Built from the ground up for modern cloud-native applications and distributed microservices architectures</p>
    </div>
    <div class="tc-features-grid">
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-speed">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M13 2.05v2.02c3.95.49 7 3.85 7 7.93 0 3.21-1.92 6-4.72 7.28L13 17v5h5l-1.22-1.22C19.91 19.07 22 15.76 22 12c0-5.18-3.95-9.45-9-9.95zM11 2c-1.95.2-3.8.96-5.32 2.21L7.1 5.63A8.195 8.195 0 0111 4.06V2zM4.21 5.68C2.96 7.2 2.2 9.05 2 11h2.06c.19-1.42.75-2.77 1.64-3.9L4.21 5.68zM2.06 13c.2 1.96.97 3.81 2.21 5.33l1.42-1.43A8.002 8.002 0 014.06 13H2.06zM5 12c0 3.86 3.14 7 7 7s7-3.14 7-7-3.14-7-7-7-7 3.14-7 7z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Blazing Fast Cache Performance</h3>
        <p class="tc-feature-desc">Lock-minimized architecture with sharded maps and async IO, optimized for multi-core systems and maximum throughput in distributed cache clusters.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-tag">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M21.41 11.58l-9-9C12.05 2.22 11.55 2 11 2H4c-1.1 0-2 .9-2 2v7c0 .55.22 1.05.59 1.42l9 9c.36.36.86.58 1.41.58.55 0 1.05-.22 1.41-.59l7-7c.37-.36.59-.86.59-1.41 0-.55-.23-1.06-.59-1.42zM5.5 7C4.67 7 4 6.33 4 5.5S4.67 4 5.5 4 7 4.67 7 5.5 6.33 7 5.5 7z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Smart Cache Tag Invalidation</h3>
        <p class="tc-feature-desc">Attach multiple tags to cache entries for surgical purges. Invalidate by user, tenant, or any custom dimension instantly across distributed backend systems.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-atomic">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M12 10.11c1.03 0 1.87.84 1.87 1.89 0 1-.84 1.85-1.87 1.85S10.13 13 10.13 12c0-1.05.84-1.89 1.87-1.89M7.37 20c.63.38 2.01-.2 3.6-1.7-.52-.59-1.03-1.23-1.51-1.9a22.7 22.7 0 01-2.4-.36c-.51 2.14-.32 3.61.31 3.96m.71-5.74l-.29-.51c-.11.29-.22.58-.29.86.27.06.57.11.88.16l-.3-.51m6.54-.76l.81-1.5-.81-1.5c-.3-.53-.62-1-.91-1.47C13.17 9 12.6 9 12 9s-1.17 0-1.71.03c-.29.47-.61.94-.91 1.47L8.57 12l.81 1.5c.3.53.62 1 .91 1.47.54.03 1.11.03 1.71.03s1.17 0 1.71-.03c.29-.47.61-.94.91-1.47M12 6.78c-.19.22-.39.45-.59.72h1.18c-.2-.27-.4-.5-.59-.72m0 10.44c.19-.22.39-.45.59-.72h-1.18c.2.27.4.5.59.72M16.62 4c-.62-.38-2 .2-3.59 1.7.52.59 1.03 1.23 1.51 1.9.82.08 1.63.2 2.4.36.51-2.14.32-3.61-.32-3.96m-.7 5.74l.29.51c.11-.29.22-.58.29-.86-.27-.06-.57-.11-.88-.16l.3.51m1.45-7.05c1.47.84 1.63 3.05 1.01 5.63 2.54.75 4.37 1.99 4.37 3.68s-1.83 2.93-4.37 3.68c.62 2.58.46 4.79-1.01 5.63-1.46.84-3.45-.12-5.37-1.95-1.92 1.83-3.91 2.79-5.38 1.95-1.46-.84-1.62-3.05-1-5.63-2.54-.75-4.37-1.99-4.37-3.68s1.83-2.93 4.37-3.68c-.62-2.58-.46-4.79 1-5.63 1.47-.84 3.46.12 5.38 1.95 1.92-1.83 3.91-2.79 5.37-1.95M17.08 12c.34.75.68 1.53.98 2.29.76-.21 1.42-.43 1.89-.64-.26-.81-.65-1.68-1.18-2.57-.36.34-.75.69-1.17 1.04l-.52-.12m-.63-3.12c.42.35.84.73 1.26 1.14.53-.9.92-1.76 1.18-2.58-.47-.2-1.13-.43-1.89-.64-.3.77-.64 1.54-.98 2.29l.43-.21M6.92 12c-.34-.75-.68-1.53-.98-2.29-.76.21-1.42.43-1.89.64.26.81.65 1.68 1.18 2.57.36-.34.75-.69 1.17-1.04l.52.12m.63 3.12c-.42-.35-.84-.73-1.26-1.14-.53.9-.92 1.76-1.18 2.58.47.2 1.13.43 1.89.64.3-.77.64-1.54.98-2.29l-.43.21z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Atomic Operations</h3>
        <p class="tc-feature-desc">Thread-safe INCR, DECR, and ADD operations perfect for rate limiting, counters, and distributed coordination.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-protocol">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M4.93 4.93C3.12 6.74 2 9.24 2 12s1.12 5.26 2.93 7.07L12 12 4.93 4.93m14.14 0L12 12l7.07 7.07C20.88 17.26 22 14.76 22 12s-1.12-5.26-2.93-7.07M13 17v-2h-2v2l-3 3 3 3v-2h2v2l3-3-3-3m-2-10V5l3-3-3-3v2H9V3L6 6l3 3V7h2z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Dual Protocol Support</h3>
        <p class="tc-feature-desc">Choose HTTP with JSON for simplicity or binary TCP protocol for maximum performance and minimal overhead.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-metrics">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Full Observability</h3>
        <p class="tc-feature-desc">Beautiful React-based admin dashboard with real-time metrics, structured logging, health checks, and comprehensive <a href="/docs/performance#monitoring" class="tc-internal-link">performance monitoring</a> for complete cache visibility.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-secure">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M12 1L3 5v6c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V5l-9-4zm0 10.99h7c-.53 4.12-3.28 7.79-7 8.94V12H5V6.3l7-3.11v8.8z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Enterprise Security</h3>
        <p class="tc-feature-desc">Authentication, network hardening, and TLS support ensure your cached data remains protected. <a href="/docs/configuration#security" class="tc-internal-link">Learn about security configuration</a>.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-deploy">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M7 2v11h3v9l7-12h-4l4-8z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Deploy Anywhere</h3>
        <p class="tc-feature-desc">Docker images, <a href="/docs/deployment#kubernetes" class="tc-internal-link">Kubernetes manifests</a>, systemd units, and cloud templates for seamless deployment.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-rust">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Rust Powered</h3>
        <p class="tc-feature-desc">Memory-safe, zero-cost abstractions, and fearless concurrency for unmatched reliability and performance.</p>
      </article>
      <article class="tc-feature-card">
        <div class="tc-feature-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-concurrent">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M14,2A8,8 0 0,1 22,10A8,8 0 0,1 14,18A8,8 0 0,1 6,10A8,8 0 0,1 14,2M14,4A6,6 0 0,0 8,10A6,6 0 0,0 14,16A6,6 0 0,0 20,10A6,6 0 0,0 14,4Z M14,6A4,4 0 0,1 18,10A4,4 0 0,1 14,14A4,4 0 0,1 10,10A4,4 0 0,1 14,6Z M8,12A6,6 0 0,1 2,18A6,6 0 0,1 8,12Z M8,14A4,4 0 0,0 4,18A4,4 0 0,0 8,22A4,4 0 0,0 12,18A4,4 0 0,0 8,14Z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-feature-title">Multi-Shard Concurrency</h3>
        <p class="tc-feature-desc">DashMap-powered sharding eliminates lock contention, enabling true parallel access across CPU cores for maximum throughput.</p>
      </article>
    </div>
  </div>
</section>

<!-- QUICK START -->
<section class="tc-quickstart" aria-label="Quick installation">
  <div class="tc-quickstart-inner">
    <div class="tc-section-header">
      <h2 class="tc-section-title">Get Started in Seconds</h2>
      <p class="tc-section-subtitle">Choose your preferred installation method</p>
    </div>

{{< tabs items="Homebrew,Docker,Binary,Package Managers,Source" >}}
{{< tab >}}
```bash
# Install via Homebrew
brew tap aminshamim/tap
brew install tagcache

# Start the server
tagcache server
```
{{< /tab >}}
{{< tab >}}
```bash
# Run with Docker
docker run -d --name tagcache \
  -p 8080:8080 -p 1984:1984 \
  ghcr.io/aminshamim/tagcache:latest

# Check status
docker logs tagcache
```
{{< /tab >}}
{{< tab >}}
```bash
# Linux x86_64
curl -L -o tagcache-linux-x86_64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-linux-x86_64.tar.gz
tar xzf tagcache-linux-x86_64.tar.gz
sudo cp tagcache /usr/local/bin/

# macOS Intel
curl -L -o tagcache-macos-x86_64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-macos-x86_64.tar.gz
tar xzf tagcache-macos-x86_64.tar.gz
sudo cp tagcache /usr/local/bin/

# macOS Apple Silicon
curl -L -o tagcache-macos-arm64.tar.gz \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-macos-arm64.tar.gz
tar xzf tagcache-macos-arm64.tar.gz
sudo cp tagcache /usr/local/bin/
```
{{< /tab >}}
{{< tab >}}
```bash
# Debian/Ubuntu (APT)
curl -L -o tagcache.deb \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache_1.0.8_amd64.deb
sudo dpkg -i tagcache.deb

# RHEL/CentOS/Fedora (RPM)
curl -L -o tagcache.rpm \
  https://github.com/aminshamim/tagcache/releases/download/v1.0.8/tagcache-1.0.8-1.x86_64.rpm
sudo rpm -ivh tagcache.rpm
```
{{< /tab >}}
{{< tab >}}
```bash
# From source with Cargo
cargo install --git https://github.com/aminshamim/tagcache.git

# Or clone and build
git clone https://github.com/aminshamim/tagcache.git
cd tagcache
cargo build --release
sudo cp target/release/tagcache /usr/local/bin/
```
{{< /tab >}}
{{< /tabs >}}
  </div>
</section>

<!-- CODE EXAMPLES -->
<section class="tc-code-section" aria-label="Code examples">
  <div class="tc-code-inner">
    <div class="tc-section-header">
      <h2 class="tc-section-title">Simple Yet Powerful API</h2>
      <p class="tc-section-subtitle">Tag-based invalidation makes cache management intuitive</p>
    </div>
    <div class="tc-code-spotlight">
  <div class="title"><span class="dot"></span><span class="dot"></span><span class="dot"></span></div>

```bash
# Set with tag(s)
curl -X POST :8080/set \
  -H 'Content-Type: application/json' \
  -d '{"key":"user:42:profile","value":{"name":"Ada"},"tags":["user:42","profile"]}'

# Invalidate all profile data for user 42
curl -X POST :8080/tag/delete \
  -H 'Content-Type: application/json' \
  -d '{"tag":"user:42"}'

# Atomic increment (rate limit / counter)
curl -X POST :8080/increment -d '{"key":"ratelimit:login","delta":1}'
```
    </div>
  </div>
</section>

<!-- ECOSYSTEM -->
<section class="tc-ecosystem-section" aria-label="Ecosystem and integrations">
  <div class="tc-ecosystem-inner">
    <div class="tc-section-header">
      <h2 class="tc-section-title">Integrates With Your Technology Stack</h2>
      <p class="tc-section-subtitle">Native cache client support for all major programming languages and cloud platforms</p>
    </div>
    <div class="tc-ecosystem-grid">
      <article class="tc-eco-card">
        <div class="tc-eco-icon rust" aria-hidden="true">🦀</div>
        <h3 class="tc-eco-title">Rust</h3>
        <p class="tc-eco-desc">First-class runtime and native async client library</p>
        <a href="/docs/api" class="tc-eco-link" aria-label="View Rust API documentation">View API →</a>
      </article>
      <article class="tc-eco-card">
        <div class="tc-eco-icon node" aria-hidden="true">📦</div>
        <h3 class="tc-eco-title">Node.js</h3>
        <p class="tc-eco-desc">TypeScript client with connection pooling</p>
        <a href="/docs/examples#node" class="tc-eco-link" aria-label="View Node.js guide">Get Started →</a>
      </article>
      <article class="tc-eco-card">
        <div class="tc-eco-icon python" aria-hidden="true">🐍</div>
        <h3 class="tc-eco-title">Python</h3>
        <p class="tc-eco-desc">Async support via httpx and aiohttp</p>
        <a href="/docs/examples#python" class="tc-eco-link" aria-label="View Python examples">Examples →</a>
      </article>
      <article class="tc-eco-card">
        <div class="tc-eco-icon go" aria-hidden="true">🐹</div>
        <h3 class="tc-eco-title">Go</h3>
        <p class="tc-eco-desc">High-performance client with connection reuse</p>
        <a href="/docs/examples#go" class="tc-eco-link" aria-label="View Go samples">Samples →</a>
      </article>
      <article class="tc-eco-card">
        <div class="tc-eco-icon php" aria-hidden="true">🐘</div>
        <h3 class="tc-eco-title">PHP</h3>
        <p class="tc-eco-desc">PSR-compliant cache adapter with tag support</p>
        <a href="/docs/examples#php" class="tc-eco-link" aria-label="View PHP examples">Examples →</a>
      </article>
      <article class="tc-eco-card">
        <div class="tc-eco-icon k8s" aria-hidden="true">☸️</div>
        <h3 class="tc-eco-title">Kubernetes</h3>
        <p class="tc-eco-desc">Helm charts and horizontal scaling guides</p>
        <a href="/docs/deployment#kubernetes" class="tc-eco-link" aria-label="View Kubernetes deployment guide">Deploy →</a>
      </article>
      <article class="tc-eco-card">
        <div class="tc-eco-icon java" aria-hidden="true">☕</div>
        <h3 class="tc-eco-title">Java & Spring</h3>
        <p class="tc-eco-desc">Spring Boot integration with connection pooling</p>
        <a href="/docs/examples#java" class="tc-eco-link" aria-label="View Java examples">Examples →</a>
      </article>
      <article class="tc-eco-card">
        <div class="tc-eco-icon csharp" aria-hidden="true">🔷</div>
        <h3 class="tc-eco-title">.NET & C#</h3>
        <p class="tc-eco-desc">Async HttpClient patterns for microservices</p>
        <a href="/docs/examples#dotnet" class="tc-eco-link" aria-label="View .NET examples">Examples →</a>
      </article>
    </div>
  </div>
</section>

<!-- LIMITATIONS & ROADMAP -->
<section class="tc-limitations" aria-label="Current limitations and roadmap">
  <div class="tc-limitations-inner">
    <div class="tc-section-header">
      <h2 class="tc-section-title">Current Limitations & Roadmap</h2>
      <p class="tc-section-subtitle">Transparent about current constraints and exciting features coming soon</p>
    </div>
    <div class="tc-limitations-grid">
      <article class="tc-limitation-card">
        <div class="tc-limitation-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-memory">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M17,9H7V7H17M17,13H7V11H17M17,17H7V15H17M12,3A1,1 0 0,1 13,4A1,1 0 0,1 12,5A1,1 0 0,1 11,4A1,1 0 0,1 12,3M19,3H14.82C14.4,1.84 13.3,1 12,1C10.7,1 9.6,1.84 9.18,3H5A2,2 0 0,0 3,5V19A2,2 0 0,0 5,21H19A2,2 0 0,0 21,19V5A2,2 0 0,0 19,3Z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-limitation-title">In-Memory Only</h3>
        <p class="tc-limitation-desc">Currently no persistence to disk. Cache data is lost on restart. <strong>Future:</strong> Optional persistence with configurable write-ahead logging.</p>
      </article>
      <article class="tc-limitation-card">
        <div class="tc-limitation-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-cluster">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M12,1L21,5V11C21,16.55 18.84,21.74 12,23C5.16,21.74 3,16.55 3,11V5L12,1M12,7C13.66,7 15,8.34 15,10C15,11.66 13.66,13 12,13C10.34,13 9,11.66 9,10C9,8.34 10.34,7 12,7Z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-limitation-title">Single Node Architecture</h3>
        <p class="tc-limitation-desc">No built-in replication or clustering support. <strong>Roadmap:</strong> Consistent hashing with peer discovery and automatic failover.</p>
      </article>
      <article class="tc-limitation-card">
        <div class="tc-limitation-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-compress">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M5,12L10,8V11H14V8L19,12L14,16V13H10V16L5,12M2,6H4V18H2V6M20,6H22V18H20V6Z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-limitation-title">No Compression</h3>
        <p class="tc-limitation-desc">Binary protocol lacks compression layer for large payloads. <strong>Planned:</strong> Optional compression with configurable algorithms (LZ4, Zstd).</p>
      </article>
      <article class="tc-limitation-card">
        <div class="tc-limitation-icon" aria-hidden="true">
          <div class="tc-icon-wrapper tc-icon-tags">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M17.63,5.84C17.27,5.33 16.67,5 16,5L5,5.01C3.9,5.01 3,5.9 3,7V17C3,18.1 3.9,19 5,19H16C16.67,19 17.27,18.67 17.63,18.16L22,12L17.63,5.84Z"/>
            </svg>
          </div>
        </div>
        <h3 class="tc-limitation-title">Unbounded Tag Cardinality</h3>
        <p class="tc-limitation-desc">No automatic limits on unique tag counts. Monitor memory usage with high tag diversity. <strong>Future:</strong> Configurable tag limits and cleanup policies.</p>
      </article>
    </div>
    <div class="tc-roadmap-note">
      <p><strong>🗺️ Roadmap Priority:</strong> Clustering and replication are top priorities for v2.0, followed by persistence options and compression enhancements.</p>
    </div>
  </div>
</section>

<!-- CTA -->
<section class="tc-cta-section" aria-label="Call to action">
  <div class="tc-cta">
  <h2>Ready to accelerate your stack?</h2>
  <p>Adopt TagCache as your unified low-latency caching layer with surgical invalidation and observability from day one.</p>
  <div class="tc-cta-actions">
    <a class="tc-btn tc-btn-primary" href="/docs/getting-started">Start Building →</a>
    <a class="tc-btn tc-btn-ghost" href="/docs/deployment">Deploy to Production</a>
    </div>
  </div>
</section>

<!-- EXPLORE MORE -->
<section class="tc-explore" aria-label="Explore documentation">
  <div class="tc-explore-inner">
    <div class="tc-section-header">
      <h2 class="tc-section-title">Explore the Documentation</h2>
      <p class="tc-section-subtitle">Everything you need to build with TagCache</p>
    </div>

{{< cards >}}
  {{< card link="docs/getting-started" title="Getting Started" icon="play" subtitle="Install & first requests" >}}
  {{< card link="docs/configuration" title="Configuration" icon="cog" subtitle="Tune memory & shards" >}}
  {{< card link="docs/api" title="API Reference" icon="book-open" subtitle="HTTP & TCP endpoints" >}}
  {{< card link="docs/performance" title="Performance" icon="chart-bar" subtitle="Benchmarks & tuning" >}}
  {{< card link="docs/examples" title="Examples" icon="code" subtitle="Real-world patterns" >}}
  {{< card link="docs/deployment" title="Deployment" icon="cloud" subtitle="Docker, K8s, systemd" >}}
{{< /cards >}}
  </div>
</section>

{{< callout type="info" >}}
**Need help?** Search the docs, check our [GitHub discussions](https://github.com/aminshamim/tagcache/discussions), or [open an issue](https://github.com/aminshamim/tagcache/issues).
{{< /callout >}}

</div>
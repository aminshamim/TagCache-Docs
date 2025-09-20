---
title: Deployment
weight: 8
type: docs
---

This guide covers production deployment strategies for TagCache across different environments and platforms.

## Production Checklist

Before deploying TagCache to production, ensure you have:

- [ ] **Changed default credentials** (`admin`/`password`)
- [ ] **Configured appropriate memory limits**
- [ ] **Set up monitoring and alerting**
- [ ] **Configured backup strategies**
- [ ] **Implemented health checks**
- [ ] **Set up proper logging**
- [ ] **Configured security settings**
- [ ] **Tested failover scenarios**

## Deployment Methods

### Docker Deployment

#### Single Container

```yaml
# docker-compose.yml
version: '3.8'

services:
  tagcache:
    image: aminshamim/tagcache:latest
    container_name: tagcache
    restart: unless-stopped
    
    ports:
      - "1984:1984"  # TCP port
      - "8080:8080"  # HTTP port
    
    environment:
      - TAGCACHE_AUTH_USERNAME=cache_admin
      - TAGCACHE_AUTH_PASSWORD=${TAGCACHE_PASSWORD}
      - TAGCACHE_CACHE_MAX_MEMORY=4GB
      - TAGCACHE_PERFORMANCE_NUM_SHARDS=512
      - TAGCACHE_LOGGING_LEVEL=info
      - TAGCACHE_LOGGING_FORMAT=json
    
    volumes:
      - ./tagcache.conf:/etc/tagcache/tagcache.conf:ro
      - tagcache_logs:/var/log/tagcache
    
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # Resource limits
    deploy:
      resources:
        limits:
          memory: 5G
          cpus: '2.0'
        reservations:
          memory: 2G
          cpus: '1.0'

volumes:
  tagcache_logs:
```

#### High Availability Setup

```yaml
# docker-compose.ha.yml
version: '3.8'

services:
  tagcache-primary:
    image: aminshamim/tagcache:latest
    container_name: tagcache-primary
    restart: unless-stopped
    
    ports:
      - "1984:1984"
      - "8080:8080"
    
    environment:
      - TAGCACHE_AUTH_USERNAME=cache_admin
      - TAGCACHE_AUTH_PASSWORD=${TAGCACHE_PASSWORD}
      - TAGCACHE_CACHE_MAX_MEMORY=8GB
      - TAGCACHE_CLUSTERING_ENABLED=true
      - TAGCACHE_CLUSTERING_NODE_ID=primary
      - TAGCACHE_CLUSTERING_PEERS=tagcache-secondary:7946
    
    volumes:
      - ./config/primary.conf:/etc/tagcache/tagcache.conf:ro
      - tagcache_primary_data:/var/lib/tagcache

  tagcache-secondary:
    image: aminshamim/tagcache:latest
    container_name: tagcache-secondary
    restart: unless-stopped
    
    ports:
      - "1985:1984"
      - "8081:8080"
    
    environment:
      - TAGCACHE_AUTH_USERNAME=cache_admin
      - TAGCACHE_AUTH_PASSWORD=${TAGCACHE_PASSWORD}
      - TAGCACHE_CACHE_MAX_MEMORY=8GB
      - TAGCACHE_CLUSTERING_ENABLED=true
      - TAGCACHE_CLUSTERING_NODE_ID=secondary
      - TAGCACHE_CLUSTERING_PEERS=tagcache-primary:7946
    
    volumes:
      - ./config/secondary.conf:/etc/tagcache/tagcache.conf:ro
      - tagcache_secondary_data:/var/lib/tagcache

  # Load balancer
  nginx:
    image: nginx:alpine
    container_name: tagcache-lb
    restart: unless-stopped
    
    ports:
      - "80:80"
      - "1983:1983"
    
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    
    depends_on:
      - tagcache-primary
      - tagcache-secondary

volumes:
  tagcache_primary_data:
  tagcache_secondary_data:
```

### Kubernetes Deployment

#### Basic Deployment

```yaml
# tagcache-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tagcache
  labels:
    app: tagcache
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tagcache
  template:
    metadata:
      labels:
        app: tagcache
    spec:
      containers:
      - name: tagcache
        image: aminshamim/tagcache:latest
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 1984
          name: tcp
        
        env:
        - name: TAGCACHE_AUTH_USERNAME
          valueFrom:
            secretKeyRef:
              name: tagcache-auth
              key: username
        - name: TAGCACHE_AUTH_PASSWORD
          valueFrom:
            secretKeyRef:
              name: tagcache-auth
              key: password
        - name: TAGCACHE_CACHE_MAX_MEMORY
          value: "4GB"
        - name: TAGCACHE_PERFORMANCE_NUM_SHARDS
          value: "512"
        
        resources:
          requests:
            memory: "2Gi"
            cpu: "500m"
          limits:
            memory: "5Gi"
            cpu: "2000m"
        
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        volumeMounts:
        - name: config
          mountPath: /etc/tagcache
          readOnly: true
        
      volumes:
      - name: config
        configMap:
          name: tagcache-config

---
apiVersion: v1
kind: Service
metadata:
  name: tagcache-service
spec:
  selector:
    app: tagcache
  ports:
  - name: http
    protocol: TCP
    port: 8080
    targetPort: 8080
  - name: tcp
    protocol: TCP
    port: 1984
    targetPort: 1984
  type: ClusterIP

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: tagcache-config
data:
  tagcache.conf: |
    [server]
    http_port = 8080
    tcp_port = 1984
    bind_address = "0.0.0.0"
    
    [cache]
    max_memory = "4GB"
    eviction_policy = "lru"
    
    [performance]
    num_shards = 512
    cleanup_interval_ms = 60000
    
    [logging]
    level = "info"
    format = "json"
    output = "stdout"

---
apiVersion: v1
kind: Secret
metadata:
  name: tagcache-auth
type: Opaque
data:
  username: Y2FjaGVfYWRtaW4=  # cache_admin (base64)
  password: c2VjdXJlX3Bhc3N3b3JkXzEyMw==  # secure_password_123 (base64)
```

#### StatefulSet with Persistence

```yaml
# tagcache-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: tagcache
spec:
  serviceName: tagcache-headless
  replicas: 3
  selector:
    matchLabels:
      app: tagcache
  template:
    metadata:
      labels:
        app: tagcache
    spec:
      containers:
      - name: tagcache
        image: aminshamim/tagcache:latest
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 1984
          name: tcp
        - containerPort: 7946
          name: cluster
        
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: TAGCACHE_CLUSTERING_ENABLED
          value: "true"
        - name: TAGCACHE_CLUSTERING_NODE_ID
          value: "$(POD_NAME)"
        - name: TAGCACHE_PERSISTENCE_ENABLED
          value: "true"
        - name: TAGCACHE_PERSISTENCE_STORAGE_PATH
          value: "/var/lib/tagcache"
        
        volumeMounts:
        - name: data
          mountPath: /var/lib/tagcache
        - name: config
          mountPath: /etc/tagcache
        
        resources:
          requests:
            memory: "4Gi"
            cpu: "1000m"
          limits:
            memory: "8Gi"
            cpu: "2000m"
      
      volumes:
      - name: config
        configMap:
          name: tagcache-config
  
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
      storageClassName: fast-ssd

---
apiVersion: v1
kind: Service
metadata:
  name: tagcache-headless
spec:
  clusterIP: None
  selector:
    app: tagcache
  ports:
  - name: http
    port: 8080
  - name: tcp
    port: 1984
  - name: cluster
    port: 7946
```

### Traditional Server Deployment

#### Systemd Service

```bash
# Install TagCache
sudo curl -L -o /usr/local/bin/tagcache \
  https://github.com/aminshamim/tagcache/releases/latest/download/tagcache-linux-x86_64

sudo chmod +x /usr/local/bin/tagcache

# Create system user
sudo useradd --system --home /var/lib/tagcache --shell /bin/false tagcache
sudo mkdir -p /var/lib/tagcache /var/log/tagcache /etc/tagcache
sudo chown tagcache:tagcache /var/lib/tagcache /var/log/tagcache
```

```ini
# /etc/systemd/system/tagcache.service
[Unit]
Description=TagCache Server
Documentation=https://tagcache.dev
After=network.target
Wants=network.target

[Service]
Type=simple
User=tagcache
Group=tagcache
ExecStart=/usr/local/bin/tagcache server --config /etc/tagcache/tagcache.conf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=30
Restart=always
RestartSec=5
StartLimitInterval=60
StartLimitBurst=3

# Security settings
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/tagcache /var/log/tagcache
CapabilityBoundingSet=CAP_NET_BIND_SERVICE

# Resource limits
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start service
sudo systemctl enable tagcache
sudo systemctl start tagcache
sudo systemctl status tagcache
```

#### Production Configuration

```toml
# /etc/tagcache/tagcache.conf
[server]
http_port = 8080
tcp_port = 1984
bind_address = "0.0.0.0"
max_connections = 10000
request_timeout_ms = 30000

[auth]
enabled = true
username = "cache_admin"
password = "your_secure_password_here"
hash_rounds = 12

[cache]
max_memory = "16GB"
max_keys = 10000000
default_ttl_ms = 3600000
eviction_policy = "lru"

[performance]
num_shards = 1024
cleanup_interval_ms = 30000
stats_interval_ms = 5000
tcp_pool_size = 500
http_pool_size = 2000

[logging]
level = "info"
format = "json"
output = "/var/log/tagcache/tagcache.log"
log_requests = false

[monitoring]
health_check = true
metrics_enabled = true
prometheus_format = true

[security]
cors_enabled = true
cors_origins = ["https://yourdomain.com"]
rate_limiting = true
rate_limit_requests = 1000
rate_limit_window_ms = 60000
```

## Load Balancing

### Nginx Configuration

```nginx
# /etc/nginx/sites-available/tagcache
upstream tagcache_http {
    least_conn;
    server tagcache-1:8080 max_fails=3 fail_timeout=30s;
    server tagcache-2:8080 max_fails=3 fail_timeout=30s;
    server tagcache-3:8080 max_fails=3 fail_timeout=30s;
    
    keepalive 32;
}

upstream tagcache_tcp {
    least_conn;
    server tagcache-1:1984 max_fails=3 fail_timeout=30s;
    server tagcache-2:1984 max_fails=3 fail_timeout=30s;
    server tagcache-3:1984 max_fails=3 fail_timeout=30s;
}

# HTTP API Load Balancer
server {
    listen 80;
    server_name cache.yourdomain.com;
    
    # Health check endpoint
    location /health {
        access_log off;
        proxy_pass http://tagcache_http/health;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 10s;
    }
    
    # API endpoints
    location /api/ {
        proxy_pass http://tagcache_http;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Connection pooling
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        
        # Timeouts
        proxy_connect_timeout 10s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
        
        # Rate limiting
        limit_req zone=api burst=100 nodelay;
    }
    
    # Web dashboard
    location / {
        proxy_pass http://tagcache_http;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# TCP Load Balancer
stream {
    upstream tagcache_tcp_backend {
        least_conn;
        server tagcache-1:1984 max_fails=3 fail_timeout=30s;
        server tagcache-2:1984 max_fails=3 fail_timeout=30s;
        server tagcache-3:1984 max_fails=3 fail_timeout=30s;
    }
    
    server {
        listen 1984;
        proxy_pass tagcache_tcp_backend;
        proxy_timeout 1s;
        proxy_responses 1;
        proxy_connect_timeout 1s;
    }
}

# Rate limiting
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=1000r/m;
}
```

### HAProxy Configuration

```
# /etc/haproxy/haproxy.cfg
global
    daemon
    maxconn 4096
    log stdout local0

defaults
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s
    option httplog
    option dontlognull
    option redispatch
    retries 3

frontend tagcache_http_frontend
    bind *:80
    mode http
    default_backend tagcache_http_backend
    
    # Health check
    acl health_check path_beg /health
    http-request return status 200 content-type "application/json" string '{"status":"ok"}' if health_check

backend tagcache_http_backend
    mode http
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    
    server tagcache-1 tagcache-1:8080 check inter 30s rise 2 fall 3
    server tagcache-2 tagcache-2:8080 check inter 30s rise 2 fall 3
    server tagcache-3 tagcache-3:8080 check inter 30s rise 2 fall 3

frontend tagcache_tcp_frontend
    bind *:1984
    mode tcp
    default_backend tagcache_tcp_backend

backend tagcache_tcp_backend
    mode tcp
    balance leastconn
    option tcp-check
    
    server tagcache-1 tagcache-1:1984 check inter 10s rise 2 fall 3
    server tagcache-2 tagcache-2:1984 check inter 10s rise 2 fall 3
    server tagcache-3 tagcache-3:1984 check inter 10s rise 2 fall 3

listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 30s
```

## Monitoring and Observability

### Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'tagcache'
    static_configs:
      - targets: ['tagcache-1:8080', 'tagcache-2:8080', 'tagcache-3:8080']
    metrics_path: /metrics
    scrape_interval: 10s
    basic_auth:
      username: 'cache_admin'
      password: 'your_password'

rule_files:
  - "tagcache_alerts.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
```

### Grafana Dashboard

```json
{
  "dashboard": {
    "title": "TagCache Monitoring",
    "panels": [
      {
        "title": "Operations per Second",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(tagcache_total_operations[5m])",
            "legendFormat": "{{instance}} - Ops/sec"
          }
        ]
      },
      {
        "title": "Cache Hit Rate",
        "type": "stat",
        "targets": [
          {
            "expr": "tagcache_hit_rate * 100",
            "legendFormat": "Hit Rate %"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "tagcache_memory_used_bytes / 1024 / 1024 / 1024",
            "legendFormat": "{{instance}} - Memory GB"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "tagcache_avg_response_time_ms",
            "legendFormat": "{{instance}} - Avg Latency"
          }
        ]
      }
    ]
  }
}
```

### Alert Rules

```yaml
# tagcache_alerts.yml
groups:
  - name: tagcache
    rules:
      - alert: TagCacheDown
        expr: up{job="tagcache"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "TagCache instance is down"
          description: "TagCache instance {{ $labels.instance }} has been down for more than 1 minute."

      - alert: TagCacheHighMemoryUsage
        expr: (tagcache_memory_used_bytes / tagcache_memory_available_bytes) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "TagCache high memory usage"
          description: "TagCache instance {{ $labels.instance }} memory usage is above 85%"

      - alert: TagCacheHighLatency
        expr: tagcache_avg_response_time_ms > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "TagCache high latency"
          description: "TagCache instance {{ $labels.instance }} average response time is above 10ms"

      - alert: TagCacheLowHitRate
        expr: tagcache_hit_rate < 0.8
        for: 10m
        labels:
          severity: info
        annotations:
          summary: "TagCache low hit rate"
          description: "TagCache instance {{ $labels.instance }} hit rate is below 80%"
```

## Backup and Recovery

### Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/tagcache-backup.sh

set -euo pipefail

BACKUP_DIR="/var/backups/tagcache"
RETENTION_DAYS=7
TAGCACHE_URL="http://localhost:8080"
TAGCACHE_USER="cache_admin"
TAGCACHE_PASS="your_password"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Generate timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/tagcache_backup_$TIMESTAMP.json"

echo "Starting TagCache backup at $(date)"

# Export all cache data
curl -s -u "$TAGCACHE_USER:$TAGCACHE_PASS" \
  "$TAGCACHE_URL/api/export/all" > "$BACKUP_FILE"

# Verify backup
if [[ -s "$BACKUP_FILE" ]]; then
    echo "Backup completed successfully: $BACKUP_FILE"
    
    # Compress backup
    gzip "$BACKUP_FILE"
    echo "Backup compressed: ${BACKUP_FILE}.gz"
    
    # Remove old backups
    find "$BACKUP_DIR" -name "tagcache_backup_*.json.gz" -mtime +$RETENTION_DAYS -delete
    echo "Old backups cleaned up"
else
    echo "Backup failed: $BACKUP_FILE is empty"
    rm -f "$BACKUP_FILE"
    exit 1
fi

echo "Backup completed at $(date)"
```

### Cron Job

```bash
# Add to crontab (crontab -e)
# Backup every 6 hours
0 */6 * * * /usr/local/bin/tagcache-backup.sh >> /var/log/tagcache/backup.log 2>&1
```

### Recovery Procedure

```bash
#!/bin/bash
# /usr/local/bin/tagcache-restore.sh

BACKUP_FILE="$1"
TAGCACHE_URL="http://localhost:8080"
TAGCACHE_USER="cache_admin"
TAGCACHE_PASS="your_password"

if [[ -z "$BACKUP_FILE" ]]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

if [[ ! -f "$BACKUP_FILE" ]]; then
    echo "Backup file not found: $BACKUP_FILE"
    exit 1
fi

echo "Starting TagCache restore from: $BACKUP_FILE"

# Clear existing cache (optional)
read -p "Clear existing cache before restore? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    curl -s -u "$TAGCACHE_USER:$TAGCACHE_PASS" \
      -X DELETE "$TAGCACHE_URL/api/clear/all"
    echo "Existing cache cleared"
fi

# Restore data
if [[ "$BACKUP_FILE" == *.gz ]]; then
    zcat "$BACKUP_FILE" | curl -s -u "$TAGCACHE_USER:$TAGCACHE_PASS" \
      -X POST "$TAGCACHE_URL/api/import" \
      -H "Content-Type: application/json" \
      --data-binary @-
else
    curl -s -u "$TAGCACHE_USER:$TAGCACHE_PASS" \
      -X POST "$TAGCACHE_URL/api/import" \
      -H "Content-Type: application/json" \
      --data-binary "@$BACKUP_FILE"
fi

echo "Restore completed"
```

## Security Hardening

### Firewall Configuration

```bash
# UFW (Ubuntu)
sudo ufw allow from 10.0.0.0/8 to any port 8080 comment "TagCache HTTP - Internal"
sudo ufw allow from 10.0.0.0/8 to any port 1984 comment "TagCache TCP - Internal"
sudo ufw allow from 172.16.0.0/12 to any port 8080 comment "TagCache HTTP - Docker"
sudo ufw allow from 172.16.0.0/12 to any port 1984 comment "TagCache TCP - Docker"

# iptables
iptables -A INPUT -s 10.0.0.0/8 -p tcp --dport 8080 -j ACCEPT
iptables -A INPUT -s 10.0.0.0/8 -p tcp --dport 1984 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j DROP
iptables -A INPUT -p tcp --dport 1984 -j DROP
```

### TLS/SSL Configuration

```toml
# tagcache.conf with TLS
[security]
tls_enabled = true
tls_cert_file = "/etc/ssl/certs/tagcache.crt"
tls_key_file = "/etc/ssl/private/tagcache.key"
```

### Network Policies (Kubernetes)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: tagcache-netpol
spec:
  podSelector:
    matchLabels:
      app: tagcache
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web-app
    - podSelector:
        matchLabels:
          app: api-server
    ports:
    - protocol: TCP
      port: 8080
    - protocol: TCP
      port: 1984
  egress:
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

## Troubleshooting

### Common Issues

**Service Won't Start:**
```bash
# Check service status
sudo systemctl status tagcache

# Check logs
sudo journalctl -u tagcache -f

# Validate configuration
tagcache config validate /etc/tagcache/tagcache.conf
```

**High Memory Usage:**
```bash
# Check memory stats
curl -u admin:password http://localhost:8080/api/stats | jq .data.memory

# Force garbage collection
curl -u admin:password -X POST http://localhost:8080/api/admin/gc

# Check for memory leaks
sudo systemctl restart tagcache
```

**Connection Issues:**
```bash
# Test connectivity
telnet localhost 1984
curl -I http://localhost:8080/health

# Check listening ports
sudo netstat -tlnp | grep tagcache

# Check firewall
sudo ufw status
```

## Next Steps

{{< cards >}}
  {{< card link="../performance" title="Performance Guide" icon="chart-bar" subtitle="Optimize for your workload" >}}
  {{< card link="../examples" title="Examples" icon="code" subtitle="Implementation patterns" >}}
  {{< card link="../configuration" title="Configuration" icon="cog" subtitle="Detailed configuration options" >}}
{{< /cards >}}

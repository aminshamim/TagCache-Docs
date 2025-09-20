---
title: Examples
weight: 6
type: docs
---

This page provides real-world examples of using TagCache in various scenarios and programming languages.

## Web Application Session Management

### PHP Session Handler

```php
<?php
class TagCacheSessionHandler implements SessionHandlerInterface
{
    private $client;
    private $ttl;
    
    public function __construct($host = 'localhost', $port = 8080, $ttl = 1800)
    {
        $this->client = new TagCacheHTTPClient($host, $port, 'admin', 'password');
        $this->ttl = $ttl * 1000; // Convert to milliseconds
    }
    
    public function open($savePath, $sessionName)
    {
        return true;
    }
    
    public function read($sessionId)
    {
        $response = $this->client->get("session:$sessionId");
        return $response ? $response['value'] : '';
    }
    
    public function write($sessionId, $sessionData)
    {
        return $this->client->set(
            "session:$sessionId",
            $sessionData,
            ['ttl_ms' => $this->ttl, 'tags' => ['session']]
        );
    }
    
    public function destroy($sessionId)
    {
        return $this->client->delete("session:$sessionId");
    }
    
    public function gc($maxlifetime)
    {
        // TagCache handles TTL automatically
        return true;
    }
    
    public function close()
    {
        return true;
    }
}

// Register the session handler
session_set_save_handler(new TagCacheSessionHandler());
session_start();

// Use sessions normally
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'john_doe';
```

### Express.js Session Store

```javascript
const express = require('express');
const session = require('express-session');
const TagCacheStore = require('tagcache-session-store');

const app = express();

// Configure TagCache session store
app.use(session({
  store: new TagCacheStore({
    host: 'localhost',
    port: 8080,
    username: 'admin',
    password: 'password',
    ttl: 1800000, // 30 minutes in milliseconds
    prefix: 'session:',
    tags: ['session']
  }),
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: { maxAge: 1800000 }
}));

app.get('/login', (req, res) => {
  req.session.userId = 123;
  req.session.username = 'john_doe';
  res.json({ message: 'Logged in successfully' });
});

app.get('/profile', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  
  res.json({
    userId: req.session.userId,
    username: req.session.username
  });
});

app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Could not log out' });
    }
    res.json({ message: 'Logged out successfully' });
  });
});

app.listen(3000);
```

## API Response Caching

### Python Django Cache Backend

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'tagcache_django.TagCacheBackend',
        'LOCATION': 'http://localhost:8080',
        'OPTIONS': {
            'USERNAME': 'admin',
            'PASSWORD': 'password',
            'DEFAULT_TTL': 3600,  # 1 hour
        }
    }
}

# views.py
from django.core.cache import cache
from django.http import JsonResponse
from django.views.decorators.cache import cache_page
import json

# Cache with automatic invalidation
def get_user_profile(request, user_id):
    cache_key = f"user:profile:{user_id}"
    
    # Try to get from cache
    profile = cache.get(cache_key)
    if profile:
        return JsonResponse(json.loads(profile))
    
    # Fetch from database
    try:
        user = User.objects.get(id=user_id)
        profile_data = {
            'id': user.id,
            'name': user.name,
            'email': user.email,
            'last_login': user.last_login.isoformat() if user.last_login else None
        }
        
        # Cache with tags for easy invalidation
        cache.set(
            cache_key,
            json.dumps(profile_data),
            timeout=3600,
            tags=[f"user:{user_id}", "profile"]
        )
        
        return JsonResponse(profile_data)
        
    except User.DoesNotExist:
        return JsonResponse({'error': 'User not found'}, status=404)

# Invalidate user cache when profile is updated
def update_user_profile(request, user_id):
    # Update user in database
    user = User.objects.get(id=user_id)
    user.name = request.POST.get('name', user.name)
    user.email = request.POST.get('email', user.email)
    user.save()
    
    # Invalidate all cache entries for this user
    cache.delete_many_by_tag(f"user:{user_id}")
    
    return JsonResponse({'message': 'Profile updated'})

# Cache expensive database queries
@cache_page(300, tags=['stats', 'dashboard'])  # 5 minutes
def dashboard_stats(request):
    stats = {
        'total_users': User.objects.count(),
        'active_sessions': Session.objects.filter(expire_date__gt=timezone.now()).count(),
        'daily_signups': User.objects.filter(
            date_joined__date=timezone.now().date()
        ).count()
    }
    return JsonResponse(stats)
```

### Go HTTP API with Cache

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "time"
    
    "github.com/gorilla/mux"
    "github.com/aminshamim/tagcache-go"
)

type APIServer struct {
    cache *tagcache.HTTPClient
}

type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    Created  string `json:"created"`
}

func NewAPIServer() *APIServer {
    cache := tagcache.NewHTTPClient("http://localhost:8080", "admin", "password")
    return &APIServer{cache: cache}
}

func (s *APIServer) getUserHandler(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID := vars["id"]
    
    cacheKey := fmt.Sprintf("user:%s", userID)
    
    // Try cache first
    if cached, err := s.cache.Get(cacheKey); err == nil {
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("X-Cache", "HIT")
        w.Write([]byte(cached.Value))
        return
    }
    
    // Simulate database fetch
    user := s.fetchUserFromDB(userID)
    if user == nil {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    
    // Serialize user
    userData, _ := json.Marshal(user)
    
    // Cache for 1 hour with tags
    s.cache.Set(cacheKey, string(userData), tagcache.SetOptions{
        TTL:  3600000, // 1 hour in milliseconds
        Tags: []string{fmt.Sprintf("user:%s", userID), "profile"},
    })
    
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("X-Cache", "MISS")
    w.Write(userData)
}

func (s *APIServer) updateUserHandler(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID := vars["id"]
    
    // Update user in database
    s.updateUserInDB(userID, r)
    
    // Invalidate cache
    s.cache.DeleteByTag(fmt.Sprintf("user:%s", userID))
    
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]string{"status": "updated"})
}

func (s *APIServer) fetchUserFromDB(userID string) *User {
    // Simulate database lookup
    time.Sleep(50 * time.Millisecond) // Simulate DB latency
    
    id, _ := strconv.Atoi(userID)
    return &User{
        ID:      id,
        Name:    fmt.Sprintf("User %s", userID),
        Email:   fmt.Sprintf("user%s@example.com", userID),
        Created: time.Now().Format(time.RFC3339),
    }
}

func (s *APIServer) updateUserInDB(userID string, r *http.Request) {
    // Simulate database update
    time.Sleep(30 * time.Millisecond)
}

func main() {
    server := NewAPIServer()
    
    r := mux.NewRouter()
    r.HandleFunc("/users/{id}", server.getUserHandler).Methods("GET")
    r.HandleFunc("/users/{id}", server.updateUserHandler).Methods("PUT")
    
    log.Println("Server starting on :8000")
    log.Fatal(http.ListenAndServe(":8000", r))
}
```

## Rate Limiting

### Redis-style Rate Limiter

```python
import time
import json
from tagcache import HTTPClient

class TagCacheRateLimiter:
    def __init__(self, cache_client):
        self.cache = cache_client
    
    def is_allowed(self, key, limit, window_seconds):
        """
        Sliding window rate limiter
        
        Args:
            key: Unique identifier (e.g., user ID, IP address)
            limit: Maximum requests allowed
            window_seconds: Time window in seconds
        
        Returns:
            (allowed: bool, remaining: int, reset_time: float)
        """
        cache_key = f"rate_limit:{key}"
        current_time = time.time()
        window_start = current_time - window_seconds
        
        # Get current request log
        cached = self.cache.get(cache_key)
        if cached:
            requests = json.loads(cached['value'])
        else:
            requests = []
        
        # Remove old requests outside the window
        requests = [req_time for req_time in requests if req_time > window_start]
        
        # Check if limit exceeded
        if len(requests) >= limit:
            oldest_request = min(requests)
            reset_time = oldest_request + window_seconds
            return False, 0, reset_time
        
        # Add current request
        requests.append(current_time)
        
        # Update cache
        ttl_ms = int(window_seconds * 1000)
        self.cache.set(
            cache_key,
            json.dumps(requests),
            ttl_ms=ttl_ms,
            tags=['rate_limit', f'user:{key}']
        )
        
        remaining = limit - len(requests)
        reset_time = current_time + window_seconds
        
        return True, remaining, reset_time

# Usage example
cache = HTTPClient('http://localhost:8080', 'admin', 'password')
limiter = TagCacheRateLimiter(cache)

def api_endpoint(user_id):
    allowed, remaining, reset_time = limiter.is_allowed(
        f"user:{user_id}", 
        limit=100,        # 100 requests
        window_seconds=60 # per minute
    )
    
    if not allowed:
        return {
            'error': 'Rate limit exceeded',
            'retry_after': int(reset_time - time.time())
        }, 429
    
    # Process request
    return {
        'data': 'success',
        'rate_limit': {
            'remaining': remaining,
            'reset': int(reset_time)
        }
    }
```

### Token Bucket Rate Limiter

```javascript
class TokenBucketRateLimiter {
    constructor(cacheClient) {
        this.cache = cacheClient;
    }
    
    async isAllowed(key, capacity, refillRate, requestTokens = 1) {
        const cacheKey = `token_bucket:${key}`;
        const now = Date.now();
        
        try {
            // Get current bucket state
            const cached = await this.cache.get(cacheKey);
            let bucket;
            
            if (cached) {
                bucket = JSON.parse(cached.value);
            } else {
                bucket = {
                    tokens: capacity,
                    lastRefill: now
                };
            }
            
            // Calculate tokens to add based on time elapsed
            const timeElapsed = (now - bucket.lastRefill) / 1000; // seconds
            const tokensToAdd = Math.floor(timeElapsed * refillRate);
            
            // Refill bucket (up to capacity)
            bucket.tokens = Math.min(capacity, bucket.tokens + tokensToAdd);
            bucket.lastRefill = now;
            
            // Check if enough tokens available
            if (bucket.tokens >= requestTokens) {
                bucket.tokens -= requestTokens;
                
                // Update cache
                await this.cache.set(cacheKey, JSON.stringify(bucket), {
                    ttl_ms: 3600000, // 1 hour
                    tags: ['rate_limit', `bucket:${key}`]
                });
                
                return {
                    allowed: true,
                    tokensRemaining: bucket.tokens
                };
            } else {
                return {
                    allowed: false,
                    tokensRemaining: bucket.tokens,
                    retryAfter: Math.ceil((requestTokens - bucket.tokens) / refillRate)
                };
            }
            
        } catch (error) {
            // On error, allow request (fail open)
            console.error('Rate limiter error:', error);
            return { allowed: true, error: true };
        }
    }
}

// Express.js middleware
function createRateLimitMiddleware(cacheClient, options = {}) {
    const limiter = new TokenBucketRateLimiter(cacheClient);
    const {
        capacity = 100,
        refillRate = 10,     // tokens per second
        keyGenerator = (req) => req.ip,
        onLimitReached = (req, res) => {
            res.status(429).json({ error: 'Rate limit exceeded' });
        }
    } = options;
    
    return async (req, res, next) => {
        const key = keyGenerator(req);
        const result = await limiter.isAllowed(key, capacity, refillRate);
        
        // Add rate limit headers
        res.set('X-RateLimit-Limit', capacity);
        res.set('X-RateLimit-Remaining', result.tokensRemaining || 0);
        
        if (!result.allowed) {
            if (result.retryAfter) {
                res.set('Retry-After', result.retryAfter);
            }
            return onLimitReached(req, res);
        }
        
        next();
    };
}

// Usage
const express = require('express');
const TagCache = require('tagcache-client');

const app = express();
const cache = new TagCache.HTTPClient({
    url: 'http://localhost:8080',
    username: 'admin',
    password: 'password'
});

// Apply rate limiting
app.use('/api/', createRateLimitMiddleware(cache, {
    capacity: 1000,      // 1000 tokens
    refillRate: 10,      // 10 tokens per second
    keyGenerator: (req) => `api:${req.ip}`
}));

app.get('/api/data', (req, res) => {
    res.json({ message: 'API response' });
});
```

## Real-time Analytics

### Page View Counter

```python
import asyncio
import json
from datetime import datetime, timedelta
from tagcache import HTTPClient

class PageViewAnalytics:
    def __init__(self, cache_client):
        self.cache = cache_client
    
    def track_page_view(self, page_path, user_id=None, metadata=None):
        """Track a page view with optional user and metadata"""
        timestamp = datetime.now()
        
        # Increment total page views
        self.cache.increment(f"page_views:total:{page_path}")
        
        # Increment hourly counter
        hour_key = timestamp.strftime("%Y-%m-%d:%H")
        self.cache.increment(
            f"page_views:hourly:{page_path}:{hour_key}",
            initial=0,
            ttl_ms=7200000  # 2 hours TTL
        )
        
        # Increment daily counter
        day_key = timestamp.strftime("%Y-%m-%d")
        self.cache.increment(
            f"page_views:daily:{page_path}:{day_key}",
            initial=0,
            ttl_ms=2592000000  # 30 days TTL
        )
        
        # Track unique users (if user_id provided)
        if user_id:
            unique_key = f"unique_users:daily:{page_path}:{day_key}"
            self.cache.set(
                f"{unique_key}:{user_id}",
                "1",
                ttl_ms=86400000,  # 24 hours
                tags=["analytics", f"page:{page_path}", "unique_users"]
            )
        
        # Store detailed event (for recent activity)
        event = {
            "timestamp": timestamp.isoformat(),
            "page": page_path,
            "user_id": user_id,
            "metadata": metadata or {}
        }
        
        event_key = f"events:recent:{timestamp.timestamp()}"
        self.cache.set(
            event_key,
            json.dumps(event),
            ttl_ms=3600000,  # 1 hour
            tags=["analytics", "events", f"page:{page_path}"]
        )
    
    def get_page_stats(self, page_path, period="daily"):
        """Get aggregated stats for a page"""
        if period == "hourly":
            # Get last 24 hours
            stats = {}
            current_hour = datetime.now().replace(minute=0, second=0, microsecond=0)
            
            for i in range(24):
                hour = current_hour - timedelta(hours=i)
                hour_key = hour.strftime("%Y-%m-%d:%H")
                cache_key = f"page_views:hourly:{page_path}:{hour_key}"
                
                try:
                    result = self.cache.get(cache_key)
                    count = int(result['value']) if result else 0
                except:
                    count = 0
                
                stats[hour_key] = count
            
            return stats
        
        elif period == "daily":
            # Get last 30 days
            stats = {}
            current_date = datetime.now().date()
            
            for i in range(30):
                date = current_date - timedelta(days=i)
                day_key = date.strftime("%Y-%m-%d")
                cache_key = f"page_views:daily:{page_path}:{day_key}"
                
                try:
                    result = self.cache.get(cache_key)
                    count = int(result['value']) if result else 0
                except:
                    count = 0
                
                stats[day_key] = count
            
            return stats
    
    def get_top_pages(self, limit=10):
        """Get top pages by total views"""
        # This would typically require a separate index
        # For demo, we'll return mock data
        # In production, you'd maintain a sorted set or use a different approach
        
        # Get all page view keys
        all_keys = self.cache.get_by_pattern("page_views:total:*")
        page_stats = []
        
        for key, data in all_keys.items():
            page_path = key.replace("page_views:total:", "")
            views = int(data['value'])
            page_stats.append((page_path, views))
        
        # Sort by views and return top pages
        page_stats.sort(key=lambda x: x[1], reverse=True)
        return page_stats[:limit]

# Usage in a web application
analytics = PageViewAnalytics(cache)

# Flask example
from flask import Flask, request, jsonify
app = Flask(__name__)

@app.before_request
def track_request():
    # Track page view for each request
    user_id = request.headers.get('X-User-ID')  # From auth system
    metadata = {
        'user_agent': request.headers.get('User-Agent'),
        'referer': request.headers.get('Referer'),
        'ip': request.remote_addr
    }
    
    analytics.track_page_view(
        request.path,
        user_id=user_id,
        metadata=metadata
    )

@app.route('/analytics/page/<path:page_path>')
def page_analytics(page_path):
    period = request.args.get('period', 'daily')
    stats = analytics.get_page_stats(f"/{page_path}", period)
    return jsonify(stats)

@app.route('/analytics/top-pages')
def top_pages():
    limit = int(request.args.get('limit', 10))
    pages = analytics.get_top_pages(limit)
    return jsonify([{"page": page, "views": views} for page, views in pages])
```

## Microservices Cache Coordination

### Service Mesh Cache Pattern

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "time"
    
    "github.com/aminshamim/tagcache-go"
)

// CacheCoordinator manages cache across multiple services
type CacheCoordinator struct {
    cache      *tagcache.HTTPClient
    serviceName string
}

func NewCacheCoordinator(serviceName, cacheURL, username, password string) *CacheCoordinator {
    cache := tagcache.NewHTTPClient(cacheURL, username, password)
    return &CacheCoordinator{
        cache:       cache,
        serviceName: serviceName,
    }
}

// UserService manages user data
type UserService struct {
    coordinator *CacheCoordinator
}

func (us *UserService) GetUser(ctx context.Context, userID string) (*User, error) {
    cacheKey := fmt.Sprintf("user:%s", userID)
    
    // Try cache first
    if cached, err := us.coordinator.cache.Get(cacheKey); err == nil {
        var user User
        json.Unmarshal([]byte(cached.Value), &user)
        log.Printf("Cache HIT for user:%s", userID)
        return &user, nil
    }
    
    log.Printf("Cache MISS for user:%s", userID)
    
    // Fetch from database
    user, err := us.fetchUserFromDB(userID)
    if err != nil {
        return nil, err
    }
    
    // Cache user data with tags
    userData, _ := json.Marshal(user)
    us.coordinator.cache.Set(cacheKey, string(userData), tagcache.SetOptions{
        TTL:  3600000, // 1 hour
        Tags: []string{
            fmt.Sprintf("user:%s", userID),
            "service:user",
            fmt.Sprintf("svc:%s", us.coordinator.serviceName),
        },
    })
    
    return user, nil
}

func (us *UserService) UpdateUser(ctx context.Context, userID string, updates map[string]interface{}) error {
    // Update in database
    err := us.updateUserInDB(userID, updates)
    if err != nil {
        return err
    }
    
    // Invalidate user cache across all services
    us.coordinator.InvalidateUserCache(userID)
    
    // Notify other services about user update
    us.coordinator.NotifyUserUpdate(userID, updates)
    
    return nil
}

// OrderService manages orders
type OrderService struct {
    coordinator *CacheCoordinator
}

func (os *OrderService) GetUserOrders(ctx context.Context, userID string) ([]Order, error) {
    cacheKey := fmt.Sprintf("orders:user:%s", userID)
    
    // Try cache first
    if cached, err := os.coordinator.cache.Get(cacheKey); err == nil {
        var orders []Order
        json.Unmarshal([]byte(cached.Value), &orders)
        return orders, nil
    }
    
    // Fetch from database
    orders, err := os.fetchOrdersFromDB(userID)
    if err != nil {
        return nil, err
    }
    
    // Cache orders with user tag
    ordersData, _ := json.Marshal(orders)
    os.coordinator.cache.Set(cacheKey, string(ordersData), tagcache.SetOptions{
        TTL:  1800000, // 30 minutes
        Tags: []string{
            fmt.Sprintf("user:%s", userID),
            "service:order",
            fmt.Sprintf("svc:%s", os.coordinator.serviceName),
        },
    })
    
    return orders, nil
}

// Cache coordination methods
func (cc *CacheCoordinator) InvalidateUserCache(userID string) {
    // Invalidate all cache entries for this user across all services
    tag := fmt.Sprintf("user:%s", userID)
    err := cc.cache.DeleteByTag(tag)
    if err != nil {
        log.Printf("Failed to invalidate cache for user %s: %v", userID, err)
    } else {
        log.Printf("Invalidated all cache for user:%s", userID)
    }
}

func (cc *CacheCoordinator) InvalidateServiceCache(serviceName string) {
    // Invalidate all cache entries for a specific service
    tag := fmt.Sprintf("service:%s", serviceName)
    err := cc.cache.DeleteByTag(tag)
    if err != nil {
        log.Printf("Failed to invalidate cache for service %s: %v", serviceName, err)
    }
}

func (cc *CacheCoordinator) NotifyUserUpdate(userID string, updates map[string]interface{}) {
    // Store update notification for other services
    notification := map[string]interface{}{
        "user_id":   userID,
        "updates":   updates,
        "timestamp": time.Now().Unix(),
        "service":   cc.serviceName,
    }
    
    notificationData, _ := json.Marshal(notification)
    notificationKey := fmt.Sprintf("notification:user_update:%s:%d", userID, time.Now().UnixNano())
    
    cc.cache.Set(notificationKey, string(notificationData), tagcache.SetOptions{
        TTL:  300000, // 5 minutes
        Tags: []string{"notifications", "user_updates", fmt.Sprintf("user:%s", userID)},
    })
}

// Service deployment coordination
func (cc *CacheCoordinator) RegisterService() {
    serviceInfo := map[string]interface{}{
        "name":      cc.serviceName,
        "started":   time.Now().Unix(),
        "version":   "1.0.0",
        "pid":       fmt.Sprintf("%d", 12345), // os.Getpid()
    }
    
    serviceData, _ := json.Marshal(serviceInfo)
    serviceKey := fmt.Sprintf("service:registry:%s", cc.serviceName)
    
    cc.cache.Set(serviceKey, string(serviceData), tagcache.SetOptions{
        TTL:  60000, // 1 minute (heartbeat)
        Tags: []string{"service_registry", fmt.Sprintf("svc:%s", cc.serviceName)},
    })
}

func (cc *CacheCoordinator) Heartbeat() {
    ticker := time.NewTicker(30 * time.Second)
    go func() {
        for range ticker.C {
            cc.RegisterService()
        }
    }()
}

// Data structures
type User struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

type Order struct {
    ID     string `json:"id"`
    UserID string `json:"user_id"`
    Total  float64 `json:"total"`
}

// Mock database methods
func (us *UserService) fetchUserFromDB(userID string) (*User, error) {
    // Simulate database fetch
    time.Sleep(50 * time.Millisecond)
    return &User{ID: userID, Name: "John Doe", Email: "john@example.com"}, nil
}

func (us *UserService) updateUserInDB(userID string, updates map[string]interface{}) error {
    // Simulate database update
    time.Sleep(30 * time.Millisecond)
    return nil
}

func (os *OrderService) fetchOrdersFromDB(userID string) ([]Order, error) {
    // Simulate database fetch
    time.Sleep(100 * time.Millisecond)
    return []Order{
        {ID: "order1", UserID: userID, Total: 99.99},
        {ID: "order2", UserID: userID, Total: 149.99},
    }, nil
}

func main() {
    // Initialize services
    coordinator := NewCacheCoordinator("user-service", "http://localhost:8080", "admin", "password")
    userService := &UserService{coordinator: coordinator}
    orderService := &OrderService{coordinator: coordinator}
    
    // Register service and start heartbeat
    coordinator.RegisterService()
    coordinator.Heartbeat()
    
    // Example usage
    ctx := context.Background()
    
    // Fetch user (will cache)
    user, _ := userService.GetUser(ctx, "123")
    fmt.Printf("User: %+v\n", user)
    
    // Fetch orders (will cache with user tag)
    orders, _ := orderService.GetUserOrders(ctx, "123")
    fmt.Printf("Orders: %+v\n", orders)
    
    // Update user (will invalidate all related cache)
    userService.UpdateUser(ctx, "123", map[string]interface{}{
        "name": "John Smith",
    })
    
    // Fetch user again (cache miss, fresh data)
    user, _ = userService.GetUser(ctx, "123")
    fmt.Printf("Updated User: %+v\n", user)
}
```

These examples demonstrate real-world usage patterns for TagCache across different scenarios and programming languages. Each example shows how to leverage TagCache's key features like tag-based invalidation, atomic operations, and TTL management for building robust, high-performance applications.

## Next Steps

{{< cards >}}
  {{< card link="../performance" title="Performance Guide" icon="chart-bar" subtitle="Benchmarking and optimization techniques" >}}
  {{< card link="../deployment" title="Deployment" icon="server" subtitle="Production deployment strategies" >}}
  {{< card link="../cli" title="CLI Reference" icon="terminal" subtitle="Complete command-line interface guide" >}}
{{< /cards >}}

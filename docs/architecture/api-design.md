# API Design Documentation

## 1. Rate Limiting

Purpose: Prevent abuse, ensure fair usage, and protect backend resources.

Implementation Strategies:
yaml
```yaml
# Example rate limit configuration
rate_limits:
  anonymous: 100 requests/hour
  authenticated: 1000 requests/hour
  premium: 10000 requests/hour
```

HTTP Headers:
```http

X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
Retry-After: 60
```
Common Algorithms:

    Token Bucket: Flexible burst handling

    Leaky Bucket: Smooths out traffic

    Fixed Window: Simple but allows bursts at window edges

    Sliding Window: More accurate, prevents window-edge bursts

## 2. Pagination

Cursor-based (Recommended):
```json

{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MjV9",
    "has_more": true,
    "total_count": 150
  }
}
```
Offset-based (Traditional):
```json

{
  "data": [...],
  "pagination": {
    "offset": 20,
    "limit": 10,
    "total": 150,
    "total_pages": 15
  }
}
```
API Usage:
```http

# Cursor-based
GET /api/users?cursor=eyJpZCI6MjV9&limit=10

# Offset-based
GET /api/users?offset=20&limit=10

# Page-based
GET /api/users?page=2&page_size=10
```

## 3. Error Handling

Standard Error Response Format:
```json

{
  "error": {
    "code": "validation_error",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      }
    ],
    "trace_id": "req_123456789",
    "timestamp": "2023-01-15T10:30:00Z",
    "documentation_url": "https://api.example.com/docs/errors/validation_error"
  }
}
```
HTTP Status Codes:
```http

# Success
200 OK - Successful GET/PUT
201 Created - Resource created
204 No Content - Successful DELETE

# Client Errors
400 Bad Request - General client error
401 Unauthorized - Authentication required
403 Forbidden - Insufficient permissions
404 Not Found - Resource doesn't exist
409 Conflict - Resource state conflict
422 Unprocessable Entity - Validation errors
429 Too Many Requests - Rate limit exceeded

# Server Errors
500 Internal Server Error - General server error
502 Bad Gateway - Upstream service error
503 Service Unavailable - Maintenance or overloaded
```

## 4. Idempotency

Purpose: Ensure safe retries for POST, PATCH, and other non-idempotent operations.

Implementation with Idempotency Keys:
```http

POST /api/payments
Idempotency-Key: payment_123456789
Content-Type: application/json

{
  "amount": 1000,
  "currency": "USD"
}
```
Server-side Handling:
```python

def handle_idempotent_request(request):
    idempotency_key = request.headers.get('Idempotency-Key')
    
    if idempotency_key:
        # Check if we've seen this key before
        cached_response = cache.get(f"idempotency:{idempotency_key}")
        if cached_response:
            return cached_response
        
        # Process request and cache response
        response = process_request(request)
        cache.set(f"idempotency:{idempotency_key}", response, timeout=24*3600)
        return response
```
## 5. Filtering and Sorting

Filtering Syntax:
```http

# Basic filtering
GET /api/users?status=active&role=admin

# Advanced filtering
GET /api/products?price[gte]=100&price[lte]=500&category[in]=electronics,books

# Date ranges
GET /api/orders?created_at[after]=2023-01-01&created_at[before]=2023-01-31

# Search
GET /api/articles?q=api+design
```
Sorting:
```http

GET /api/users?sort=created_at:desc,name:asc
GET /api/products?sort=-price,category  # '-' prefix for descending
```
## 6. API Versioning

Strategies:

    URL Path: /api/v1/users, /api/v2/users

    Query Parameter: /api/users?version=1

    Custom Header: Accept: application/vnd.example.v1+json

    Content Negotiation: Accept: application/json; version=1

Recommended (URL Path):
```http

GET /api/v1/users/123
GET /api/v2/users/123
```
## 7. Caching Strategies

Client-side Caching:
```http

# Response headers for caching
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```
Conditional Requests:
```http

GET /api/users/123
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT
```
## 8. HATEOAS (Hypermedia as the Engine of Application State)

Response with Links:
```json

{
  "user": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "_links": {
      "self": { "href": "/api/users/123" },
      "orders": { "href": "/api/users/123/orders" },
      "update": { 
        "href": "/api/users/123", 
        "method": "PUT",
        "schema": { "$ref": "/schemas/user-update.json" }
      },
      "delete": { 
        "href": "/api/users/123", 
        "method": "DELETE" 
      }
    }
  }
}
```
## 9. Request/Response Formatting

Standard Request:
```http

POST /api/users
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>

{
  "name": "John Doe",
  "email": "john@example.com"
}
```
Standard Response:
```json

{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "created_at": "2023-01-15T10:30:00Z"
  },
  "meta": {
    "version": "1.0",
    "server_time": "2023-01-15T10:30:00Z"
  }
}
```
## 10. Security Best Practices

Authentication & Authorization:
```http

# JWT Bearer Token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# API Key
X-API-Key: sk_live_123456789abcdef

# Basic Auth (only over HTTPS)
Authorization: Basic base64(username:password)
```
Security Headers:
```http

Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```
## 11. API Documentation

OpenAPI/Swagger Example:
```yaml

openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      summary: Get a user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found
```
## 12. Additional Important Topics

Batch Operations:
```http

POST /api/batch
[
  {"method": "GET", "path": "/users/1"},
  {"method": "POST", "path": "/users", "body": {"name": "Jane"}},
  {"method": "DELETE", "path": "/users/2"}
]
```
Webhooks:
```json

{
  "event": "user.created",
  "data": {
    "user_id": 123,
    "email": "user@example.com"
  },
  "timestamp": "2023-01-15T10:30:00Z",
  "webhook_id": "wh_123456789"
}
```
Partial Responses:
```http

GET /api/users/123?fields=id,name,email
GET /api/users?fields=id,name&limit=10
```
Compression:
```http

Accept-Encoding: gzip, deflate
Content-Encoding: gzip
```
## 13. Monitoring and Analytics

Important Metrics:

    Request rate and latency (p50, p95, p99)

    Error rate (4xx, 5xx responses)

    API usage by endpoint, customer, plan

    Rate limit hits and throttling

Structured Logging:
```json

{
  "timestamp": "2023-01-15T10:30:00Z",
  "level": "INFO",
  "method": "GET",
  "path": "/api/users/123",
  "status_code": 200,
  "response_time_ms": 45,
  "user_id": "user_123",
  "request_id": "req_123456789",
  "user_agent": "Mozilla/5.0...",
  "ip_address": "192.168.1.1"
}
```

## 14.API Naming Conventions & Additional Topics
1. Resource Naming Conventions
Singular vs Plural Debate

Plural (Most Common & Recommended)
```http

GET    /api/users          # Get all users
POST   /api/users          # Create a user
GET    /api/users/123      # Get user with ID 123
PUT    /api/users/123      # Update user with ID 123
DELETE /api/users/123      # Delete user with ID 123
```

Arguments for Plural:

    Consistent across collections and individual resources

    More intuitive for collections

    Follows REST principles more closely

    Used by major APIs (GitHub, Stripe, Twitter)

Singular (Less Common)
```http

GET    /api/user           # Get all users (inconsistent)
POST   /api/user           # Create a user
GET    /api/user/123       # Get specific user
```

Problems with Singular:

    /api/user for collection vs /api/user/123 for specific is inconsistent

    Can be confusing for non-English speakers

    Less intuitive for nested resources

Consistency Rules
```http

# ✅ Recommended - Plural throughout
/api/users
/api/users/123
/api/users/123/orders
/api/users/123/orders/456

# ❌ Avoid - Mixed singular/plural
/api/user
/api/users/123
/api/user/123/order
/api/users/123/orders/456
```

2. Case Conventions

Snake Case (Recommended for URLs)
```http

GET /api/user_profiles
GET /api/order_items/123
```
Kebab Case (Also Good for URLs)
```http

GET /api/user-profiles
GET /api/order-items/123
```
Camel Case (Avoid in URLs)
```http

GET /api/userProfiles     # ❌ Less readable in URLs
```
3. Nested Resources & Relationships

Proper Nesting
```http

# User's orders
GET /api/users/123/orders

# Specific order for a user
GET /api/users/123/orders/456

# But avoid deep nesting
GET /api/users/123/orders/456/items/789/products/999  # ❌ Too deep!

# Instead, flatten when possible
GET /api/order-items/789?include=product  # ✅ Better

Relationships vs Actions
http

# Relationships (nested resources)
GET /api/users/123/orders

# Actions (use verbs carefully)
POST /api/users/123/activate
POST /api/orders/456/cancel
```
4. Additional Important API Design Topics We Missed
## 15. Content Negotiation

Multiple Formats Support:
```http

GET /api/users/123
Accept: application/json
Accept: application/xml
Accept: text/csv
```
Versioning with Accept Header:
```http

GET /api/users/123
Accept: application/vnd.company.v1+json
Accept: application/vnd.company.v2+json
```
## 16. Partial Updates (PATCH)

JSON Patch (RFC 6902):
```http

PATCH /api/users/123
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/email", "value": "new@email.com" },
  { "op": "add", "path": "/phone", "value": "+1234567890" },
  { "op": "remove", "path": "/old_field" }
]
```
JSON Merge Patch (RFC 7386):
```http

PATCH /api/users/123
Content-Type: application/merge-patch+json

{
  "email": "new@email.com",
  "phone": "+1234567890"
}
```
## 17. Bulk Operations

Creating Multiple Resources:
```http

POST /api/users/bulk
{
  "operations": [
    { "name": "John", "email": "john@example.com" },
    { "name": "Jane", "email": "jane@example.com" }
  ]
}
```
Batch Requests:
```http

POST /api/batch
{
  "requests": [
    {
      "method": "GET",
      "path": "/users/123"
    },
    {
      "method": "PUT", 
      "path": "/users/456",
      "body": { "name": "Updated" }
    }
  ]
}
```
## 18. Async Operations

Long-Running Operations:
```http

POST /api/reports
{
  "type": "yearly_sales",
  "format": "csv"
}

# Response
{
  "job_id": "job_123",
  "status": "processing",
  "estimated_completion": "2023-01-15T11:00:00Z",
  "progress": 0,
  "links": {
    "self": "/api/jobs/job_123",
    "cancel": "/api/jobs/job_123/cancel"
  }
}
```
Polling for Status:
```http

GET /api/jobs/job_123

{
  "job_id": "job_123",
  "status": "completed",
  "result_url": "/api/reports/report_456/download"
}
```
## 19. Field Selection & Sparse Fieldsets

Request Specific Fields:
```http

GET /api/users/123?fields=id,name,email
GET /api/users?fields=id,name&limit=10

# Nested field selection
GET /api/users/123?fields=id,name,profile.picture,orders.id
```
Response:
```json

{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
    // Other fields are omitted
  }
}
```
## 20. Resource Expansion (Includes)

Include Related Resources:
```http

GET /api/orders/456?include=customer,items.product
```
Response:
```json

{
  "data": {
    "id": 456,
    "total": 99.99,
    "customer": {
      "id": 123,
      "name": "John Doe"
    },
    "items": [
      {
        "id": 789,
        "product": {
          "id": 999,
          "name": "Widget"
        }
      }
    ]
  },
  "included": {
    "customers": [...],
    "products": [...]
  }
}
```
## 21. API Health Checks & Monitoring

Health Endpoints:
```http

GET /health
GET /health/ready
GET /health/live
GET /metrics  # Prometheus metrics
```
Response:
```json

{
  "status": "healthy",
  "timestamp": "2023-01-15T10:30:00Z",
  "services": {
    "database": "healthy",
    "cache": "healthy", 
    "external_api": "degraded"
  },
  "version": "1.2.3",
  "uptime": "30d 5h 23m"
}
```
## 22. Deprecation & Sunset Policies

Deprecation Headers:
```http

Deprecation: true
Sunset: Wed, 21 Oct 2024 07:28:00 GMT
Link: </api/v2/users>; rel="successor-version"
Link: </docs/deprecation>; rel="deprecation"
```
## 23. CORS (Cross-Origin Resource Sharing)

CORS Headers:
```http

Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```
## 24. File Upload & Download

File Upload:
```http

POST /api/documents
Content-Type: multipart/form-data

-- Response
{
  "file_id": "file_123",
  "filename": "document.pdf",
  "size": 1024000,
  "url": "/api/files/file_123/download"
}
```
File Download:
```http

GET /api/files/file_123/download
Content-Disposition: attachment; filename="document.pdf"
Content-Type: application/pdf
```
## 25. Real-time APIs (WebSockets/SSE)

WebSocket Example:
```javascript

// Client connection
const ws = new WebSocket('wss://api.example.com/ws');

// Subscribe to updates
ws.send(JSON.stringify({
  "action": "subscribe",
  "channel": "user:123:notifications"
}));
```
Server-Sent Events (SSE):
```http

GET /api/events

data: {"type": "notification", "data": {...}}
data: {"type": "price_update", "data": {...}}
```
## 26. Comprehensive Naming Guidelines Summary
DOs:

    Use plural nouns for collections (/api/users)

    Use consistent case (snake_case or kebab-case)

    Use nouns for resources, verbs for actions

    Keep URLs short and meaningful

    Use query parameters for filtering, sorting, pagination

DON'Ts:

    Don't use verbs in resource URLs (/api/getUsers)

    Don't expose database internals (/api/user_table)

    Don't use file extensions (/api/users.json)

    Avoid deeply nested URLs (> 3 levels)

    Don't change URL structure frequently

Good Examples:

```http

# Collections
GET /api/users
POST /api/users

# Individual resources  
GET /api/users/123
PUT /api/users/123
DELETE /api/users/123

# Actions
POST /api/users/123/activate
POST /api/users/123/deactivate

# Nested resources
GET /api/users/123/orders
GET /api/users/123/orders/456

# Filtering, sorting, pagination
GET /api/users?status=active&sort=name&page=2&limit=20

```

This comprehensive coverage ensures your API design documentation is complete and follows industry best practices. The naming conventions section specifically addresses the important decision between singular vs plural resource names.

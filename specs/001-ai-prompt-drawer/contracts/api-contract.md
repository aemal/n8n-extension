# API Contract: Prompt History Service

**Branch**: `001-ai-prompt-drawer` | **Date**: 2026-03-25
**Contract Version**: 1.0.0

## Base URL
```
https://api.n8n-prompt-history.com/api/v1
```

## Authentication
All API requests require Bearer token authentication:
```http
Authorization: Bearer {jwt_token}
```

## Endpoints

### Get Prompt History
Retrieve prompt history for a specific AI Agent node.

```http
GET /nodes/{nodeId}/prompts/history
```

**Path Parameters:**
- `nodeId` (required): String - Unique identifier of the AI Agent node

**Query Parameters:**
- `limit`: Number (optional, default: 25, max: 100) - Number of prompts to return
- `cursor`: String (optional) - Cursor for pagination
- `sort`: String (optional, default: "timestamp:desc") - Sort order
- `after`: String (optional) - ISO 8601 timestamp filter (inclusive)
- `before`: String (optional) - ISO 8601 timestamp filter (inclusive)
- `status`: String (optional) - Filter by status ("completed", "failed", "pending")

**Example Request:**
```bash
curl -X GET \
  'https://api.n8n-prompt-history.com/api/v1/nodes/node-456/prompts/history?limit=25&sort=timestamp:desc' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...'
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "prompt-uuid-123",
      "nodeId": "node-456",
      "workflowId": "workflow-789",
      "title": "Customer Support Response",
      "content": "You are a helpful customer support agent. Respond to the following customer inquiry...",
      "status": "completed",
      "createdAt": "2025-03-25T10:30:00.000Z",
      "updatedAt": "2025-03-25T10:30:01.200Z",
      "metadata": {
        "model": "gpt-4",
        "temperature": 0.7,
        "maxTokens": 1000,
        "executionTime": 1.2,
        "inputTokens": 150,
        "outputTokens": 200,
        "cost": 0.0045
      },
      "executionContext": {
        "workflowExecutionId": "exec-999",
        "inputVariables": {
          "customerMessage": "I need help with my order",
          "customerName": "John Doe"
        },
        "outputSummary": "Generated helpful response (200 tokens)",
        "errorMessage": null,
        "retryCount": 0,
        "parentPromptId": null
      }
    }
  ],
  "pagination": {
    "cursor": "eyJjcmVhdGVkQXQiOiIyMDI1LTAzLTI1VDEwOjMwOjAwWiIsImlkIjoicHJvbXB0LXV1aWQtMTIzIn0=",
    "hasMore": true,
    "limit": 25,
    "total": 1500
  },
  "meta": {
    "nodeInfo": {
      "nodeType": "ai-agent",
      "nodeName": "Customer Support AI"
    },
    "requestTime": "2025-03-25T11:00:00.000Z"
  }
}
```

**Error Responses:**

**400 Bad Request:**
```json
{
  "error": {
    "status": "BAD_REQUEST",
    "code": 400,
    "message": "Invalid request parameters",
    "details": {
      "field": "limit",
      "reason": "EXCEEDS_MAXIMUM",
      "maxValue": 100
    },
    "timestamp": "2025-03-25T11:00:00.000Z",
    "requestId": "req-uuid-789"
  }
}
```

**401 Unauthorized:**
```json
{
  "error": {
    "status": "UNAUTHORIZED",
    "code": 401,
    "message": "Invalid or expired authentication token",
    "details": {
      "reason": "TOKEN_EXPIRED"
    },
    "timestamp": "2025-03-25T11:00:00.000Z",
    "requestId": "req-uuid-789"
  }
}
```

**404 Not Found:**
```json
{
  "error": {
    "status": "NOT_FOUND",
    "code": 404,
    "message": "Node not found or no prompt history available",
    "details": {
      "nodeId": "node-456",
      "reason": "NODE_NOT_FOUND"
    },
    "timestamp": "2025-03-25T11:00:00.000Z",
    "requestId": "req-uuid-789"
  }
}
```

**429 Too Many Requests:**
```json
{
  "error": {
    "status": "TOO_MANY_REQUESTS",
    "code": 429,
    "message": "Rate limit exceeded",
    "details": {
      "limit": 1000,
      "remaining": 0,
      "resetAt": "2025-03-25T12:00:00.000Z"
    },
    "timestamp": "2025-03-25T11:00:00.000Z",
    "requestId": "req-uuid-789"
  }
}
```

**500 Internal Server Error:**
```json
{
  "error": {
    "status": "INTERNAL_SERVER_ERROR",
    "code": 500,
    "message": "An unexpected error occurred",
    "details": {
      "reason": "DATABASE_CONNECTION_FAILED"
    },
    "timestamp": "2025-03-25T11:00:00.000Z",
    "requestId": "req-uuid-789"
  }
}
```

## Rate Limiting

**Headers included in all responses:**
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1679875200
```

**Limits:**
- 1000 requests per hour per authenticated user
- 5000 requests per hour per IP address
- Burst protection: 10 requests per 10 seconds

**Rate limit exceeded response includes:**
```http
Retry-After: 60
```

## CORS Configuration

**Allowed Origins:**
- `chrome-extension://*` (for Chrome extensions)
- Development origins (configurable per environment)

**Allowed Headers:**
- `Authorization`
- `Content-Type`
- `X-Requested-With`

**Allowed Methods:**
- `GET`
- `OPTIONS`

## Caching Headers

**Successful responses include:**
```http
Cache-Control: public, max-age=300, stale-while-revalidate=60
ETag: "prompt-history-hash-123"
Last-Modified: Thu, 25 Mar 2025 10:30:00 GMT
```

**Client caching strategy:**
- Cache responses for 5 minutes
- Use ETags for conditional requests
- Implement stale-while-revalidate for better UX

## Data Types

### PromptEntry Object
```typescript
interface PromptEntry {
  id: string;                    // UUID format
  nodeId: string;                // Max 255 characters
  workflowId: string;            // Max 255 characters
  title: string | null;          // Max 255 characters
  content: string;               // Max 50,000 characters
  status: 'pending' | 'completed' | 'failed' | 'cancelled';
  createdAt: string;             // ISO 8601 timestamp
  updatedAt: string;             // ISO 8601 timestamp
  metadata: PromptMetadata;
  executionContext: ExecutionContext;
}
```

### PromptMetadata Object
```typescript
interface PromptMetadata {
  model: string;                 // Required, max 100 characters
  temperature?: number;          // 0-2 range
  maxTokens?: number;            // Positive integer
  executionTime: number;         // Seconds, non-negative
  inputTokens?: number;          // Non-negative integer
  outputTokens?: number;         // Non-negative integer
  cost?: number;                 // Non-negative decimal
}
```

### ExecutionContext Object
```typescript
interface ExecutionContext {
  workflowExecutionId: string;   // Required
  inputVariables: Record<string, any>;
  outputSummary?: string;        // Max 500 characters
  errorMessage?: string;         // Max 1000 characters
  retryCount: number;            // Non-negative integer
  parentPromptId?: string;       // UUID format if present
}
```

### Pagination Object
```typescript
interface Pagination {
  cursor: string;                // Base64 encoded cursor
  hasMore: boolean;
  limit: number;                 // 1-100 range
  total?: number;                // Total count if available
}
```

## Security Requirements

### Request Validation
- All input parameters validated against type constraints
- SQL injection prevention through parameterized queries
- Input sanitization for XSS prevention
- Request size limits (max 1MB)

### Token Management
- JWT tokens with 1 hour expiration
- Refresh token mechanism for automatic renewal
- Secure token storage requirements for clients
- Token revocation capability

### Content Security
- Response content sanitized before transmission
- No sensitive data in error messages
- Audit logging for all API access
- Rate limiting to prevent abuse

This contract defines the complete interface between the Chrome extension and the backend service, ensuring reliable and secure communication for prompt history retrieval.
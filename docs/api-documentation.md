---
layout: default
title: API Documentation
nav_order: 4
parent: Developer Guide
---

# API Documentation

This document provides comprehensive API reference documentation for the platform's RESTful endpoints.

## Overview

| Property | Value |
|----------|-------|
| Base Path | `/api/v1` |
| Content-Type | `application/json` |
| Authentication | Bearer JWT tokens via Neon Auth |

All API requests must include a valid JWT token in the Authorization header:

```
Authorization: Bearer <token>
```

---

## Endpoints

### AIController (MVP1)

#### Generate Chat Response

Creates a new AI-generated response within a conversation.

```
POST /api/v1/generate
```

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `conversation_id` | string | Yes | Unique identifier for the conversation |
| `user_id` | string | Yes | Unique identifier for the user |
| `content` | string | Yes | The user's message content |

**Example Request**

```json
{
  "conversation_id": "conv_abc123",
  "user_id": "user_xyz789",
  "content": "What are the key requirements for this feature?"
}
```

**Response Body**

| Field | Type | Description |
|-------|------|-------------|
| `conversation_id` | string | Unique identifier for the conversation |
| `user_id` | string | Unique identifier for the user |
| `content` | string | The AI-generated response |
| `type` | MessageType | The type of message |
| `created_at` | datetime | Timestamp when the message was created |
| `updated_at` | datetime | Timestamp when the message was last updated |

**Example Response**

```json
{
  "conversation_id": "conv_abc123",
  "user_id": "user_xyz789",
  "content": "Based on the context, here are the key requirements...",
  "type": "assistant",
  "created_at": "2026-02-05T10:30:00Z",
  "updated_at": "2026-02-05T10:30:00Z"
}
```

**Status Codes**

| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Bad Request - Invalid input parameters |
| 401 | Unauthorized - Missing or invalid authentication |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Conversation or user not found |
| 408 | Request Timeout - AI service timeout |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error |
| 503 | Service Unavailable - AI service unavailable |

---

### Future Endpoints

The following endpoints are planned for future releases:

| Controller | Endpoints | Description |
|------------|-----------|-------------|
| PersonaController | `GET/POST/PUT/DELETE /api/v1/personas` | Manage AI personas |
| ProjectController | `GET/POST/PUT/DELETE /api/v1/project` | Manage projects |
| ScenarioController | `GET/POST/PUT/DELETE /api/v1/scenario` | Manage scenarios |
| RequirementController | `GET/POST/PUT/DELETE /api/v1/requirement` | Manage requirements |

---

## Rate Limiting

Rate limits are enforced to ensure fair usage and system stability.

### Limits for `/api/v1/generate`

| Scope | Limit |
|-------|-------|
| Per conversation | 1 request per 3 seconds |
| Per user | 20 requests per minute |

When rate limits are exceeded, the API returns a `429 Too Many Requests` response with a `Retry-After` header indicating when the client can retry.

**Example Rate Limit Response**

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 3
Content-Type: application/json

{
  "error_code": "RATE_LIMIT_EXCEEDED",
  "message": "Rate limit exceeded. Please retry after 3 seconds.",
  "details": null,
  "correlation_id": "corr_abc123",
  "timestamp": "2026-02-05T10:30:00Z"
}
```

---

## Error Handling

### Error Response Format

All error responses follow a consistent structure:

| Field | Type | Description |
|-------|------|-------------|
| `error_code` | string | Machine-readable error code |
| `message` | string | Human-readable error message |
| `details` | object | Additional error details (optional) |
| `correlation_id` | string | Unique identifier for request tracing |
| `timestamp` | datetime | Time when the error occurred |

**Example Error Response**

```json
{
  "error_code": "VALIDATION_ERROR",
  "message": "Invalid request body",
  "details": {
    "field": "content",
    "reason": "Content cannot be empty"
  },
  "correlation_id": "corr_def456",
  "timestamp": "2026-02-05T10:30:00Z"
}
```

### Exceptions

| Exception            | HTTP Status | Description                         |
|----------------------|-------------|-------------------------------------|
| ValidationError      | 400         | Request validation failed           |
| UnauthorizedError    | 401         | Authentication required or invalid  |
| ForbiddenError       | 403         | Authenticated but not authorized    |
| NotFoundError        | 404         | Requested resource not found        |
| ConflictError        | 409         | Resource conflict (e.g., duplicate) |
| RateLimitError       | 429         | Rate limit exceeded                 |
| LlmResponseException | 500         | Error in LLM response               |
| ContextLoadException | 501         | Error loading agent context         |

---

## Next Steps

| Goal                           | Documentation                                                         |
|--------------------------------|-----------------------------------------------------------------------|
| Understand system architecture | [Architecture]({% link docs/architecture.md %})                       |
| Start implementing features    | [Implementation Guidance]({% link docs/implementation-guidance.md %}) |
| Learn testing practices        | [Testing Strategy]({% link docs/testing-strategy.md %})               |
| Troubleshoot common issues     | [Troubleshooting]({% link docs/troubleshooting.md %})                 |

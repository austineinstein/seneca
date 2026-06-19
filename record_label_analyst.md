# Record Label Docs

This repository supports record label users requesting a usable access token through the `https://www.base-jobs.vercel.app/api/get-token` 

# Access Token Endpoint & Overview

This document describes the `/api/get-token` endpoint, including its purpose, request/response formats, deployment behavior, and testing procedures.

The endpoint provides access tokens for authorized users and can be exposed either directly by the application or through a proxy implementation.

---

# Endpoint Specification

## Production Endpoint

```http
POST https://www.fancradle.com/api/get-token
```

## Local Development Endpoint

```http
POST http://localhost:5000/api/get-token
```

---

# Purpose

The endpoint generates a usable access token that can be used for authenticated access to protected API routes.

Generated tokens:

* Are returned as JSON.
* Expire after a configured period (typically 24 hours).
* Can be used via:

```http
Authorization: Bearer <token>
```

or

```http
x-api-key: <token>
```

---

# Request Format

## POST /api/get-token

### Request Body

```json
{
  "email": "user@example.com",
  "name": "Optional Name"
}
```

### Field Requirements

| Field | Required | Description           |
| ----- | -------- | --------------------- |
| email | Yes      | Valid email address   |
| name  | No       | Optional display name |

### Notes

* Email addresses are normalised to lowercase.
* Name is optional and may be returned as `null`.

---

# Successful Response

## HTTP 201 Created

```json
{
  "token": "string",
  "expires_at": "ISO 8601 timestamp",
  "expires_in_seconds": 86400,
  "user": {
    "email": "user@example.com",
    "name": "Optional Name"
  },
  "message": "Keep this token safe. Use it in Authorisation: Bearer <token> or x-api-key on protected routes."
}
```

### Response Fields

| Field              | Description                     |
| ------------------ | ------------------------------- |
| token              | Generated access token          |
| expires_at         | Expiration timestamp (ISO 8601) |
| expires_in_seconds | Token lifetime in seconds       |
| user               | User information                |
| message            | Usage guidance                  |

---

# Error Responses

## HTTP 400 Bad Request

Missing email:

```json
{
  "error": "Email is required."
}
```

Invalid email format:

```json
{
  "error": "Valid email address is required."
}
```

## HTTP 503 Service Unavailable

```json
{
  "error": "Service unavailable"
}
```

---

# Example Request 1

## Successful Token Generation

```bash
curl -X POST https://www.fancradle.com/api/get-token \
  -H "Content-Type: application/json" \
  -d '{"email”: "contact@fancradle.com", "name": "Test User"}'
```

### Example Response

```json
{
  "token": {Your_Own_Special_Key_To_Query_Our_Website},
  "expires_at": "2026-06-11T09:33:20.830195Z",
  "expires_in_seconds": 86400,
  "user": {
    "email": "contact@fancradle.com",
    "name": "Test User"
  },
  "message": "Keep this token safe. Use it in Authorisation: Bearer <token> or x-api-key on protected routes."
}
```

# Example Request 2
After you have a valid key, generate your first artist brief

# Purpose
The analyst should:


Identify artists worth reviewing.


Explain why the artist matters.


Provide supporting evidence.


Highlight risks.


Recommend next actions.


## Query A Genre/Subculture Interesting To You

```bash

```

### Example Response

```json
`get_acts` returns a JSON object with this shape on success:

```json
{
  "token": "string",
  "user": {..
  },
  "results": [
    {
      "topic": "...",
      "article_count": 3,
      "summary": "### {artists list}\n...",
      "links": "[]"
    }
  ]
}
```
---

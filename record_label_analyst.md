# Record Label Access Token Documentation

This repository supports record label users requesting a usable access token through the `https://www.base-jobs.vercel.app/api/get-token` endpoint.

# Access Token Endpoint Documentation & Test Guide

## Overview

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

* Email addresses are normalized to lowercase.
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
  "message": "Keep this token safe. Use it in Authorization: Bearer <token> or x-api-key on protected routes."
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

# Example Requests

## Successful Token Generation

```bash
curl -X POST https://www.fancradle.com/api/get-token \
  -H "Content-Type: application/json" \
  -d '{"email":"contact@fancradle.com","name":"Test User"}'
```

### Example Response

```json
{
  "token": "Euk-qMNWZCky4GU4xCM6q_UzJT6En-YdGuqvV7pIdx8",
  "expires_at": "2026-06-11T09:33:20.830195Z",
  "expires_in_seconds": 86400,
  "user": {
    "email": "contact@fancradle.com",
    "name": "Test User"
  },
  "message": "Keep this token safe. Use it in Authorization: Bearer <token> or x-api-key on protected routes."
}
```

---

# Proxy Implementation

Some deployments expose a proxy handler via:

```text
server/server.js
```

The proxy:

1. Accepts requests to `/api/get-token`.
2. Forwards requests to the upstream token service.
3. Returns the upstream JSON response unchanged.

This allows clients to use a consistent endpoint regardless of infrastructure changes.

---

# User Flow

1. Client submits a POST request to `/api/get-token`.
2. Server validates the request payload.
3. Server generates or retrieves a token.
4. JSON response is returned with token and expiration details.
5. Client uses the token on protected endpoints.

---

# Test Scenarios

## 1. Successful Token Generation

### Request

```bash
curl -X POST http://localhost:5000/api/get-token \
  -H "Content-Type: application/json" \
  -d '{"email":"contact@fancradle.com","name":"Test User"}' \
  -w "\nStatus: %{http_code}\n"
```

### Expected Result

* HTTP Status: `201`
* Token generated
* Expiration set to `86400` seconds
* Email normalized to lowercase
* Name preserved

---

## 2. Missing Email

### Request

```bash
curl -X POST http://localhost:5000/api/get-token \
  -H "Content-Type: application/json" \
  -d '{}' \
  -w "\nStatus: %{http_code}\n"
```

### Expected Response

```json
{
  "error": "Email is required."
}
```

### Validation

* HTTP Status: `400`

---

## 3. Invalid Email Format

### Request

```bash
curl -X POST http://localhost:5000/api/get-token \
  -H "Content-Type: application/json" \
  -d '{"email":"invalid-email"}' \
  -w "\nStatus: %{http_code}\n"
```

### Expected Response

```json
{
  "error": "Valid email address is required."
}
```

### Validation

* HTTP Status: `400`

---

## 4. Optional Name Field

### Request

```bash
curl -X POST http://localhost:5000/api/get-token \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com"}' \
  -w "\nStatus: %{http_code}\n"
```

### Expected Response

```json
{
  "token": "...",
  "expires_at": "...",
  "expires_in_seconds": 86400,
  "user": {
    "email": "user@example.com",
    "name": null
  }
}
```

### Validation

* HTTP Status: `201`
* Token generated successfully
* Name returned as `null`

---

# Unit Tests

## Run All Tests

### Expected Output

```text
tests/test_app.py::test_health PASSED
tests/test_app.py::test_alert_missing_message PASSED
tests/test_app.py::test_home PASSED
tests/test_app.py::test_get_token_missing_email PASSED
tests/test_app.py::test_get_token_endpoint_post PASSED

============================== 5 passed ==============================
```

---

## Run Token Tests Only

```bash
cd /workspaces/base-jobs
python -m pytest tests/test_app.py -k "get_token" -v
```

### Expected Output

```text
tests/test_app.py::test_get_token_missing_email PASSED
tests/test_app.py::test_get_token_endpoint_post PASSED

============================== 2 passed ==============================
```

---

# Redis Integration Test

## Run Test

```bash
cd /workspaces/magic
python -m pytest tests/test_app.py::test_get_token_with_redis -v -s
```

### Expected Output

```text
tests/test_app.py::test_get_token_with_redis PASSED
```

### Validation

* Token creation succeeds
* Redis storage succeeds
* Token retrieval succeeds
* 24-hour TTL is enforced

---

# Local vs Production Verification

## Local

```bash
curl -X POST http://localhost:5000/api/get-token \
  -H "Content-Type: application/json" \
  -d '{"email":"contact@fancradle.com","name":"Test User"}'
```

## Production

```bash
curl -X POST https://www.fancradle.com/api/get-token \
  -H "Content-Type: application/json" \
  -d '{"email":"contact@fancradle.com","name":"Test User"}'
```

Both environments should return the same response structure and HTTP `201 Created`.

---

# Verification Checklist

* [ ] Email validation works
* [ ] Missing email returns HTTP 400
* [ ] Invalid email returns HTTP 400
* [ ] Token is generated successfully
* [ ] Token is unique
* [ ] Expiration is set correctly
* [ ] Email is normalized to lowercase
* [ ] Name is preserved when provided
* [ ] Name is null when omitted
* [ ] Response matches specification
* [ ] Local environment returns HTTP 201
* [ ] Production environment returns HTTP 201
* [ ] Unit tests pass
* [ ] Redis integration test passes
* [ ] Error responses match specification

---

# Important Notes

* The endpoint uses the **POST** method.
* Requests must include a JSON body.
* A valid email address is required.
* Tokens should be stored securely by clients.
* The token may be used via either `Authorization: Bearer <token>` or `x-api-key`.
* Deployments may expose the endpoint directly or through a proxy implementation.



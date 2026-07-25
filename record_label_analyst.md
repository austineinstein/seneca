# Record Label Docs

This repository supports record label users requesting a usable access token through the `https://www.base-jobs.vercel.app/api/get-token` endpoint and provides additional routes for researching artists and generating analyst briefs.

# Access Token Endpoint & Overview

This document describes the `/api/get-token` endpoint, including its purpose, request/response formats, deployment behavior, and testing procedures.

The endpoint provides access tokens for authorized users and can be exposed either directly by the application or through a proxy implementation.

---

# Endpoint Specification

## Production Endpoints

```http
POST https://www.api.fancradle.com/api/get-token
GET https://www.api.fancradle.com/api/get-acts
POST https://www.api.fancradle.com/api/artist-brief
GET https://www.api.fancradle.com/api/artist-profile/{artist_id}
POST https://www.api.fancradle.com/api/validate-token
```

## Local Development Endpoints

```http
POST http://localhost:5000/api/get-token
GET http://localhost:5000/api/get-acts
POST http://localhost:5000/api/artist-brief
GET http://localhost:5000/api/artist-profile/{artist_id}
POST http://localhost:5000/api/validate-token
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
  -d '{"email": "contact@fancradle.com", "name": "Test User"}'
```

### Example Response

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_at": "2026-07-11T09:33:20.830195Z",
  "expires_in_seconds": 86400,
  "user": {
    "email": "contact@fancradle.com",
    "name": "Test User"
  },
  "message": "Keep this token safe. Use it in Authorisation: Bearer <token> or x-api-key on protected routes."
}
```

---

# Example Request 2: Query Artists by Genre/Subculture

After you have a valid token, query for artists and topics in genres/subcultures of interest.

## GET /api/get-acts

### Purpose

The analyst should use this endpoint to:

* Identify artists worth reviewing
* Discover trending topics and subcultures
* Get article counts and summaries for validation
* Retrieve supporting links for further research

### Query Parameters

| Parameter    | Required | Description                                                                  | Example                |
| ------------ | -------- | ---------------------------------------------------------------------------- | ---------------------- |
| genre        | Yes      | Genre or subculture to query                                                 | "hyperpop", "trap"     |
| limit        | No       | Maximum results to return (default: 10, max: 50)                            | 20                     |
| offset       | No       | Pagination offset (default: 0)                                               | 0                      |
| sort_by      | No       | Sort results by "relevance" or "trending" (default: "relevance")            | "trending"             |

### Authentication

**Required**: Include valid token via one of:

```http
Authorization: Bearer <token>
```

or

```http
x-api-key: <token>
```

### Example Request

```bash
curl -X GET "https://www.fancradle.com/api/get-acts?genre=hyperpop&limit=10&sort_by=trending" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Successful Response

## HTTP 200 OK

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "email": "contact@fancradle.com",
    "name": "Test User"
  },
  "results": [
    {
      "topic": "Hyperpop Rising Stars 2026",
      "article_count": 12,
      "summary": "### Featured Artists\n\n- **100 gecs**: Continues to dominate the hyperpop scene with experimental production\n- **Laura Les**: Solo breakout success with critical acclaim\n- **Arca**: Genre-pushing contributions to electronic music\n\n### Key Trends\n\n- Mainstream crossover gaining momentum\n- Festival circuit expansion\n- Collaborative projects increasing",
      "links": [
        "https://pitchfork.com/reviews/albums/hyperpop-2026",
        "https://genius.com/artists/100gecs",
        "https://www.stereogum.com/hyperpop-analysis"
      ],
      "last_updated": "2026-07-10T15:30:00Z"
    },
    {
      "topic": "Underground Hyperpop Collectives",
      "article_count": 8,
      "summary": "### Emerging Collectives\n\n- **PC Music Collective**: Continues experimental direction\n- **YEAR0001**: A&R focused label with emerging talent\n- **Opium AR15**: International rising stars\n\n### Investment Opportunities\n\n- Emerging producers under $50k endorsement deals\n- Remix culture driving artist exposure\n- TikTok-to-mainstream pipeline acceleration",
      "links": [
        "https://www.genius.com/hyperpop-collectives",
        "https://pitchfork.com/reviews/best-new-artists"
      ],
      "last_updated": "2026-07-09T08:22:00Z"
    }
  ],
  "pagination": {
    "offset": 0,
    "limit": 10,
    "total": 24
  }
}
```

### Response Fields

| Field         | Description                              |
| ------------- | ---------------------------------------- |
| topic         | Research topic/theme                     |
| article_count | Number of articles analyzed              |
| summary       | Markdown-formatted summary with insights |
| links         | Supporting URLs for verification         |
| last_updated  | Timestamp of data collection             |

### Error Responses

## HTTP 400 Bad Request

```json
{
  "error": "Genre parameter is required."
}
```

## HTTP 401 Unauthorized

```json
{
  "error": "Invalid or expired token."
}
```

## HTTP 429 Too Many Requests

```json
{
  "error": "Rate limit exceeded. Please try again in 60 seconds."
}
```

---

# Example Request 3: Generate Artist Brief

## POST /api/artist-brief

### Purpose

Generate a comprehensive analyst brief for a specific artist, including:

* Market positioning and relevance
* Risk assessment and opportunities
* Supporting evidence from multiple sources
* Recommended next actions

### Request Body

```json
{
  "artist_id": "artist_uuid_or_name",
  "genre": "hyperpop",
  "depth": "full",
  "include_comparables": true
}
```

### Field Requirements

| Field               | Required | Description                                                  | Type    | Example                |
| ------------------- | -------- | ------------------------------------------------------------ | ------- | ---------------------- |
| artist_id           | Yes      | Artist identifier or name                                    | string  | "100gecs" or "uuid..."  |
| genre               | Yes      | Primary genre for context                                    | string  | "hyperpop"             |
| depth               | No       | Brief depth: "summary", "standard", "full" (default: "standard") | string  | "full"                 |
| include_comparables | No       | Include comparable artists (default: true)                   | boolean | true                   |

### Authentication

**Required**: Include valid token via one of:

```http
Authorization: Bearer <token>
```

or

```http
x-api-key: <token>
```

### Example Request

```bash
curl -X POST https://www.fancradle.com/api/artist-brief \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -d '{
    "artist_id": "100gecs",
    "genre": "hyperpop",
    "depth": "full",
    "include_comparables": true
  }'
```

### Successful Response

## HTTP 200 OK

```json
{
  "brief_id": "brief_uuid_12345",
  "artist": {
    "id": "100gecs_uuid",
    "name": "100 gecs",
    "genre": "hyperpop",
    "established": 2017,
    "origin": "Canada"
  },
  "analysis": {
    "market_position": {
      "relevance_score": 9.2,
      "trend_momentum": "ascending",
      "mainstream_appeal": "high",
      "summary": "100 gecs has achieved significant mainstream breakthrough while maintaining artistic credibility within hyperpop community. Consistent touring and merchandise presence."
    },
    "why_they_matter": [
      "Genre ambassadors bringing experimental hyperpop to mainstream audiences",
      "Demonstrated commercial viability of niche electronic music",
      "Strong social media engagement and fanbase loyalty",
      "Collaborations spanning major labels and independent producers"
    ],
    "supporting_evidence": {
      "streaming_metrics": {
        "spotify_monthly_listeners": 8500000,
        "spotify_growth_ytd": "32%",
        "youtube_views": 250000000,
        "tiktok_trend_count": 1200
      },
      "critical_reception": [
        "Pitchfork: 7.8/10 - 'Messy but genuinely innovative'",
        "Guardian: 'Generation-defining electronic act'",
        "Rolling Stone: Included in Top 100 Artists of 2025"
      ],
      "chart_performance": {
        "billboard_hot_100_entries": 4,
        "peak_position": 18,
        "weeks_on_chart": 45
      }
    },
    "risk_assessment": {
      "risks": [
        {
          "type": "Genre Saturation",
          "severity": "medium",
          "description": "Hyperpop market becoming crowded with similar sounds",
          "mitigation": "Monitor for unique sonic evolution; support experimental direction"
        },
        {
          "type": "Artist Burnout",
          "severity": "low",
          "description": "Intensive touring schedule may impact creative output",
          "mitigation": "Track release cycle consistency and tour announcement patterns"
        },
        {
          "type": "Label Dependency",
          "severity": "low",
          "description": "Multiple label relationships create potential creative conflicts",
          "mitigation": "Maintain relationship with core management team"
        }
      ],
      "opportunities": [
        {
          "type": "International Expansion",
          "potential": "high",
          "description": "Asian markets showing strong hyperpop interest",
          "timeline": "6-12 months"
        },
        {
          "type": "Brand Partnerships",
          "potential": "high",
          "description": "Fashion and lifestyle brand alignment opportunities",
          "timeline": "3-6 months"
        },
        {
          "type": "Film/Gaming Soundtracks",
          "potential": "medium",
          "description": "Aesthetic fits emerging cyberpunk-themed content",
          "timeline": "ongoing"
        }
      ]
    },
    "comparable_artists": [
      {
        "name": "Grimes",
        "relevance": 0.85,
        "reason": "Experimental electronic with mainstream crossover"
      },
      {
        "name": "SOPHIE",
        "relevance": 0.78,
        "reason": "Genre-pioneering production techniques"
      },
      {
        "name": "Arca",
        "relevance": 0.72,
        "reason": "Art-forward electronic music approach"
      }
    ]
  },
  "recommendations": [
    "Consider A&R relationship for potential collaborative projects",
    "Monitor upcoming album/EP releases for investment opportunities",
    "Track international festival circuit for partnership possibilities",
    "Evaluate merchandise and licensing revenue potential",
    "Maintain quarterly check-ins on chart performance and streaming growth"
  ],
  "next_actions": [
    {
      "action": "Schedule label meeting",
      "owner": "User",
      "deadline": "2026-07-17",
      "priority": "high"
    },
    {
      "action": "Analyze Q3 streaming data",
      "owner": "Auto",
      "deadline": "2026-08-01",
      "priority": "medium"
    },
    {
      "action": "Review comparable artist valuations",
      "owner": "User",
      "deadline": "2026-07-20",
      "priority": "medium"
    }
  ],
  "generated_at": "2026-07-10T16:45:30Z",
  "expires_at": "2026-08-10T16:45:30Z"
}
```

### Error Responses

## HTTP 400 Bad Request

```json
{
  "error": "Artist not found. Please verify artist_id."
}
```

## HTTP 401 Unauthorized

```json
{
  "error": "Invalid or expired token."
}
```

## HTTP 422 Unprocessable Entity

```json
{
  "error": "Invalid depth parameter. Must be 'summary', 'standard', or 'full'."
}
```

---

# Example Request 4: Get Artist Profile

## GET /api/artist-profile/{artist_id}

### Purpose

Retrieve detailed profile information for a specific artist, including social metrics, discography, and recent activity.

### Path Parameters

| Parameter | Required | Description                      | Example     |
| --------- | -------- | -------------------------------- | ----------- |
| artist_id | Yes      | Artist identifier or URL slug    | "100gecs"   |

### Query Parameters

| Parameter   | Required | Description                           | Example |
| ----------- | -------- | ------------------------------------- | ------- |
| include_bio | No       | Include biographical information      | true    |
| include_social | No   | Include social media metrics          | true    |
| include_discography | No | Include albums/EPs/singles (limited to 20 most recent) | true    |

### Authentication

**Required**: Include valid token via one of:

```http
Authorization: Bearer <token>
```

or

```http
x-api-key: <token>
```

### Example Request

```bash
curl -X GET "https://www.fancradle.com/api/artist-profile/100gecs?include_bio=true&include_social=true&include_discography=true" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Successful Response

## HTTP 200 OK

```json
{
  "artist": {
    "id": "100gecs_uuid",
    "name": "100 gecs",
    "aliases": ["100 gecs", "GECS"],
    "genre": ["hyperpop", "electronic", "pop"],
    "established": 2017,
    "origin": "Vancouver, Canada",
    "bio": "100 gecs is the brainchild of Mat and Dylan Brady. Known for genre-bending hyperpop production and maximalist aesthetics.",
    "social": {
      "spotify_followers": 8500000,
      "instagram_followers": 2100000,
      "twitter_followers": 890000,
      "youtube_subscribers": 450000
    },
    "discography": [
      {
        "title": "Memesongs Vol. 3",
        "type": "EP",
        "release_date": "2026-03-15",
        "label": "YEAR0001",
        "tracks": 6,
        "streaming_urls": {
          "spotify": "https://open.spotify.com/album/...",
          "apple_music": "https://music.apple.com/..."
        }
      }
    ],
    "recent_activity": {
      "last_release": "2026-03-15",
      "last_tour_announcement": "2026-06-20",
      "last_social_post": "2026-07-09",
      "active": true
    }
  },
  "retrieved_at": "2026-07-10T16:50:00Z"
}
```

---

# Example Request 5: Validate Token

## POST /api/validate-token

### Purpose

Validate that a token is still active and retrieve token metadata.

### Request Body

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

or pass via headers:

```http
Authorization: Bearer <token>
```

### Example Request

```bash
curl -X POST https://www.fancradle.com/api/validate-token \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Successful Response

## HTTP 200 OK

```json
{
  "valid": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_at": "2026-07-11T09:33:20.830195Z",
  "expires_in_seconds": 82400,
  "user": {
    "email": "contact@fancradle.com",
    "name": "Test User"
  }
}
```

### Error Response

## HTTP 401 Unauthorized

```json
{
  "valid": false,
  "error": "Token is invalid or expired."
}
```

---

# Rate Limiting

All endpoints are subject to rate limiting:

| Endpoint | Limit | Window |
| -------- | ----- | ------ |
| /api/get-token | 5 requests | 1 hour |
| /api/get-acts | 100 requests | 1 hour |
| /api/artist-brief | 50 requests | 1 hour |
| /api/artist-profile | 200 requests | 1 hour |
| /api/validate-token | 1000 requests | 1 hour |

Rate limit headers included in all responses:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1657477200
```

---

# Authentication Best Practices

1. **Store tokens securely**: Never commit tokens to version control
2. **Rotate regularly**: Request new tokens at least weekly
3. **Use Bearer format**: Prefer `Authorization: Bearer <token>` over `x-api-key`
4. **Monitor expiration**: Check `expires_at` field and refresh before expiry
5. **Validate on client**: Use `/api/validate-token` before making requests if token age is unknown

---

# Support & Troubleshooting

For issues or questions:

* Check token expiration: Use `/api/validate-token`
* Verify genre spelling: Genre queries are case-insensitive but must match known genres
* Review rate limits: Check response headers for remaining quota
* Contact support: support@fancradle.com

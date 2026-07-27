# SearchForge REST API — Complete Specification

> **Base URL:** `http://localhost:8000/api/v1`
> **Version:** 1.0.0
> **Format:** JSON (application/json)
> **Frontend:** Vanilla JS SPA (no HTMX initially; HTMX fragments specified for future server-rendered mode)

---

## Table of Contents

1. [API Design Principles](#1-api-design-principles)
2. [Authentication & Authorization](#2-authentication--authorization)
3. [Rate Limiting](#3-rate-limiting)
4. [Error Handling](#4-error-handling)
5. [Pagination](#5-pagination)
6. [Core Data Models](#6-core-data-models)
7. [Listing Endpoints](#7-listing-endpoints)
8. [Vertical/Domain Endpoints](#8-verticallist-domain-endpoints)
9. [Scrape Endpoints](#9-scrape-endpoints)
10. [Enrichment Endpoints](#10-enrichment-endpoints)
11. [Valuation Endpoints](#11-valuation-endpoints)
12. [Alert Endpoints](#12-alert-endpoints)
13. [Search Endpoints](#13-search-endpoints)
14. [Compare Endpoints](#14-compare-endpoints)
15. [System Endpoints](#15-system-endpoints)
16. [HTMX Fragment Endpoints](#16-htmx-fragment-endpoints)
17. [OpenAPI Schema](#17-openapi-schema)

---

## 1. API Design Principles

### 1.1 RESTful Conventions

| Principle | Implementation |
|-----------|---------------|
| Resource naming | Nouns, plural (`/listings`, `/verticals`) |
| HTTP methods | GET=read, POST=create, PUT=update, PATCH=partial-update, DELETE=remove |
| Status codes | 200 OK, 201 Created, 204 No Content, 400/404/409/422/429 |
| Nesting | Max 1 level (`/verticals/{id}/filters`) |
| Filtering | Query parameters on collection endpoints |
| Sorting | `sort` + `order` query params |
| Pagination | `page` + `per_page` with Link headers |

### 1.2 Versioning

```
/api/v1/listings    ← current version
/api/v2/listings    ← future breaking changes
```

- Path-based versioning (`/api/v1/`)
- Current version: `v1`
- Deprecation: `Sunset` header + 6-month support window

### 1.3 Content Negotiation

```
Accept: application/json          (default)
Accept: text/html                 (HTMX fragments)
```

### 1.4 Timestamps

All timestamps are ISO 8601 UTC: `"2026-07-11T14:30:00Z"`

---

## 2. Authentication & Authorization

### 2.1 Single-User API Key

```http
X-API-Key: sfk_live_abc123def456
```

- Set in `.env` as `SEARCHFORGE_API_KEY`
- Optional for local development (`AUTH_ENABLED=false`)
- Required in production (`AUTH_ENABLED=true`)

### 2.2 API Key Management

```
POST   /api/v1/auth/keys         # Generate new key (admin only)
DELETE /api/v1/auth/keys/{key}   # Revoke key
GET    /api/v1/auth/keys         # List active keys
```

### 2.3 Authentication Errors

| Code | Meaning |
|------|---------|
| 401 | Missing or invalid API key |
| 403 | Valid key but insufficient permissions |
| 429 | Rate limit exceeded (see §3) |

---

## 3. Rate Limiting

### 3.1 Rate Limits

| Endpoint Category | Limit | Window |
|-------------------|-------|--------|
| Read endpoints (GET) | 300 req/min | Sliding window |
| Write endpoints (POST/PUT/DELETE) | 60 req/min | Sliding window |
| Scrape triggers | 10 req/min | Sliding window |
| Enrichment triggers | 5 req/min | Sliding window |

### 3.2 Rate Limit Headers

```http
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 287
X-RateLimit-Reset: 1720713060
Retry-After: 30
```

---

## 4. Error Handling

### 4.1 Standard Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "status": 422,
    "details": [
      {
        "field": "price_min",
        "message": "Must be a positive number",
        "code": "INVALID_TYPE"
      }
    ],
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:30:00Z"
  }
}
```

### 4.2 Error Codes

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | `BAD_REQUEST` | Malformed request body or parameters |
| 400 | `INVALID_JSON` | Request body is not valid JSON |
| 401 | `UNAUTHORIZED` | Missing or invalid API key |
| 403 | `FORBIDDEN` | Insufficient permissions |
| 404 | `NOT_FOUND` | Resource does not exist |
| 404 | `VERTICAL_NOT_FOUND` | Vertical/plugin not found |
| 404 | `LISTING_NOT_FOUND` | Listing does not exist |
| 404 | `ALERT_NOT_FOUND` | Alert rule does not exist |
| 404 | `JOB_NOT_FOUND` | Scrape job does not exist |
| 409 | `CONFLICT` | Resource already exists (duplicate URL, duplicate alert) |
| 409 | `JOB_ALREADY_RUNNING` | Scrape job already in progress for this vertical |
| 422 | `VALIDATION_ERROR` | Request body failed validation |
| 422 | `VERTICAL_DISABLED` | Target vertical is disabled |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Unexpected server error |
| 503 | `SERVICE_UNAVAILABLE` | Service temporarily unavailable (starting up) |
| 503 | `SCRAPER_UNAVAILABLE` | All scrapers blocked or down |
| 504 | `AI_TIMEOUT` | AI enrichment request timed out |

### 4.3 Error Detail Object

```json
{
  "field": "price_min",
  "message": "Must be a positive number",
  "code": "INVALID_TYPE"
}
```

| Detail Code | Description |
|-------------|-------------|
| `REQUIRED` | Field is required |
| `INVALID_TYPE` | Wrong type (string where number expected) |
| `INVALID_FORMAT` | Format mismatch (bad UUID, bad date) |
| `OUT_OF_RANGE` | Value outside allowed range |
| `TOO_LONG` | String exceeds max length |
| `INVALID_ENUM` | Value not in allowed enum |
| `NOT_FOUND` | Referenced resource doesn't exist |

---

## 5. Pagination

### 5.1 Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number (1-indexed) |
| `per_page` | integer | 25 | Items per page (max 100) |

### 5.2 Paginated Response Envelope

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "total_items": 1847,
    "total_pages": 74,
    "has_next": true,
    "has_prev": false
  },
  "meta": {
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:30:00Z"
  }
}
```

### 5.3 Link Headers

```http
Link: </api/v1/listings?page=2>; rel="next",
      </api/v1/listings?page=74>; rel="last"
```

---

## 6. Core Data Models

### 6.1 Listing (Product) — Universal

```json
{
  "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
  "domain": "cnc_machinery",
  "title": "Haas VF-2 2019 CNC Vertical Machining Center",
  "make": "Haas",
  "model": "VF-2",
  "year": 2019,
  "price_usd": 45000.00,
  "price_original": "€42,500",
  "price_currency": "EUR",
  "price_range_low": null,
  "price_range_high": null,
  "condition_raw": "Used - Good",
  "condition_score": 0.72,
  "condition_notes": "Light use, well maintained",
  "location_text": "Detroit, MI",
  "location_country": "US",
  "location_state": "MI",
  "location_city": "Detroit",
  "location_lat": 42.3314,
  "location_lng": -83.0458,
  "distance_miles": 127.3,
  "source": "machinio",
  "source_url": "https://www.machinio.com/listings/123456",
  "source_id": "machinio_123456",
  "seller_type": "dealer",
  "seller_name": "Midwest Machinery",
  "deal_score": 0.85,
  "deal_label": "Great Deal",
  "deal_color": "#4ecb71",
  "image_urls": [
    "https://cdn.example.com/img1.jpg",
    "https://cdn.example.com/img2.jpg"
  ],
  "primary_image_url": "https://cdn.example.com/img1.jpg",
  "thumbnail_url": "https://cdn.example.com/img1_thumb.jpg",
  "first_seen_at": "2026-07-11T10:00:00Z",
  "last_seen_at": "2026-07-11T14:30:00Z",
  "is_active": true,
  "is_sold": false,
  "is_bookmarked": false,
  "notes": null,
  "data_quality": 0.88,
  "enrichment_tier": "spec_db",
  "created_at": "2026-07-11T10:00:00Z",
  "updated_at": "2026-07-11T14:30:00Z"
}
```

### 6.2 Domain-Specific Extension Data

Extension data is nested under `domain_data`:

**CNC Machinery:**
```json
{
  "domain_data": {
    "spindle_rpm_max": 12000,
    "spindle_taper": "CT40",
    "spindle_power_kw": 15.0,
    "has_tsc": true,
    "tsc_psi": 1000,
    "travel_x_mm": 762,
    "travel_y_mm": 406,
    "travel_z_mm": 508,
    "travel_format": "3axis",
    "table_diameter_mm": null,
    "max_workpiece_kg": 340,
    "tool_changer_cap": 24,
    "tool_changer_type": "umbrella",
    "control_system": "Haas NGC",
    "control_model": "VF Series",
    "machine_type": "mill",
    "machine_config": "vertical",
    "axis_count": 3,
    "has_pallet_changer": false,
    "has_probing": true,
    "cutting_hours": 4500,
    "operating_hours": 8200,
    "specs_json": null
  }
}
```

**Endmills:**
```json
{
  "domain_data": {
    "diameter_mm": 10.0,
    "diameter_inches": 0.3937,
    "length_of_cut_mm": 30.0,
    "overall_length_mm": 75.0,
    "shank_diameter_mm": 10.0,
    "flute_count": 4,
    "flute_type": "variable_helix",
    "helix_angle": 38.0,
    "end_type": "square",
    "corner_radius_mm": null,
    "substrate_material": "carbide",
    "coating": "TiAlN",
    "material_target": ["steel", "stainless"],
    "application": "roughing",
    "cut_type": "plunge",
    "quantity": 1,
    "brand": "OSG",
    "sku": "EX-SUS-TICN-D10.0",
    "series": "EX-SUS",
    "in_stock": true,
    "specs_json": null
  }
}
```

**Used Cars:**
```json
{
  "domain_data": {
    "vin": "1HGBH41JXMN109186",
    "trim": "SE",
    "body_style": "sedan",
    "fuel_type": "gas",
    "transmission": "automatic",
    "drivetrain": "fwd",
    "engine_size": "1.8L",
    "engine_cylinders": 4,
    "engine_hp": 140,
    "mpg_city": 26,
    "mpg_highway": 34,
    "curb_weight_lbs": 2750,
    "color": "Silver",
    "interior_color": "Gray",
    "doors": 4,
    "accident_history": false,
    "owner_count": 2,
    "vehicle_history": null,
    "recalls_json": null,
    "safety_rating": 4.0,
    "specs_json": null
  }
}
```

### 6.3 Vertical (Domain)

```json
{
  "id": "cnc_machinery",
  "display_name": "CNC Machinery",
  "icon": "🏭",
  "enabled": true,
  "total_listings": 12450,
  "active_listings": 8920,
  "sources": ["machinio", "equipnet", "ebay", "surplus_record"],
  "last_scrape_at": "2026-07-11T12:00:00Z",
  "facets": [...],
  "sort_options": ["relevance", "price_asc", "price_desc", "year_desc", "newest"],
  "created_at": "2026-07-11T00:00:00Z"
}
```

### 6.4 Scrape Job

```json
{
  "job_id": "job_7f8e9d0c1b2a",
  "vertical_id": "cnc_machinery",
  "status": "running",
  "progress": {
    "sources_total": 5,
    "sources_completed": 3,
    "sources_failed": 0,
    "listings_found": 234,
    "listings_new": 45,
    "listings_updated": 12
  },
  "started_at": "2026-07-11T14:00:00Z",
  "estimated_completion": "2026-07-11T14:15:00Z",
  "elapsed_seconds": 432,
  "error": null,
  "created_at": "2026-07-11T14:00:00Z"
}
```

### 6.5 Alert Rule

```json
{
  "id": 1,
  "name": "Cheap Haas VF-2",
  "enabled": true,
  "domain": "cnc_machinery",
  "conditions": {
    "make": { "op": "equals", "value": "Haas" },
    "model": { "op": "contains", "value": "VF-2" },
    "price_usd": { "op": "lte", "value": 30000 },
    "condition_score": { "op": "gte", "value": 0.5 }
  },
  "actions": [
    { "type": "webhook", "url": "https://hooks.slack.com/services/T00/B00/xxx" }
  ],
  "notify_on": "new_match",
  "last_triggered_at": null,
  "trigger_count": 0,
  "created_at": "2026-07-11T10:00:00Z",
  "updated_at": "2026-07-11T10:00:00Z"
}
```

### 6.6 Enrichment Stats

```json
{
  "total_cached": 15420,
  "cache_hit_rate": 0.87,
  "total_cost_usd": 12.45,
  "cost_today_usd": 0.32,
  "cost_by_model": {
    "deepseek/deepseek-chat": 8.20,
    "openai/gpt-4o-mini": 4.25
  },
  "tokens_by_model": {
    "deepseek/deepseek-chat": 2048000,
    "openai/gpt-4o-mini": 512000
  },
  "enrichment_tiers": {
    "none": 2340,
    "rules": 1200,
    "spec_db": 8920,
    "llm_text": 3100,
    "llm_vision": 860
  },
  "last_enrichment_at": "2026-07-11T14:25:00Z"
}
```

### 6.7 Price History Entry

```json
{
  "history_id": 4521,
  "product_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
  "price_usd": 45000.00,
  "price_original": "€42,500",
  "source": "machinio",
  "recorded_at": "2026-07-11T10:00:00Z"
}
```

### 6.8 Facet Definition

```json
{
  "name": "spindle_rpm_max",
  "label": "Spindle RPM",
  "type": "range",
  "min_value": 0,
  "max_value": 24000,
  "step": 1000,
  "options": null
}
```

For categorical facets:
```json
{
  "name": "coating",
  "label": "Coating",
  "type": "categorical",
  "min_value": null,
  "max_value": null,
  "step": null,
  "options": [
    { "value": "TiN", "label": "TiN", "count": 234 },
    { "value": "TiAlN", "label": "TiAlN", "count": 189 },
    { "value": "AlCrN", "label": "AlCrN", "count": 67 },
    { "value": "uncoated", "label": "Uncoated", "count": 312 }
  ]
}
```

### 6.9 Connector Status

```json
{
  "connector_id": "machinio",
  "display_name": "Machinio",
  "type": "web_scraper",
  "enabled": true,
  "healthy": true,
  "status": "idle",
  "last_check_at": "2026-07-11T14:30:00Z",
  "last_error": null,
  "consecutive_failures": 0,
  "total_scrapes": 1247,
  "total_listings_found": 8920,
  "avg_response_ms": 320,
  "rate_limit": {
    "rpm": 30,
    "current_rpm": 5
  }
}
```

---

## 7. Listing Endpoints

### 7.1 List Listings

```
GET /api/v1/listings
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | string | — | Full-text search query |
| `domain` | string | — | Filter by domain (`cnc_machinery`, `endmill`, `used_car`) |
| `make` | string | — | Filter by make/brand (exact or partial) |
| `model` | string | — | Filter by model (exact or partial) |
| `year_min` | integer | — | Minimum year |
| `year_max` | integer | — | Maximum year |
| `price_min` | number | — | Minimum price (USD) |
| `price_max` | number | — | Maximum price (USD) |
| `condition_min` | number | — | Minimum condition score (0.0-1.0) |
| `condition_max` | number | — | Maximum condition score (0.0-1.0) |
| `source` | string | — | Filter by source site |
| `seller_type` | string | — | `dealer` or `private` |
| `location_lat` | number | — | Latitude for distance calculation |
| `location_lng` | number | — | Longitude for distance calculation |
| `location_radius_miles` | number | 50 | Search radius (miles) |
| `deal_score_min` | number | — | Minimum deal score |
| `is_bookmarked` | boolean | — | Filter bookmarked only |
| `is_active` | boolean | true | Include inactive listings |
| `sort` | string | `relevance` | Sort field (see below) |
| `order` | string | `desc` | `asc` or `desc` |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 25 | Items per page (max 100) |

**Domain-Specific Parameters:**

| Domain | Parameter | Type | Description |
|--------|-----------|------|-------------|
| CNC | `spindle_rpm_min` | integer | Minimum spindle RPM |
| CNC | `spindle_rpm_max` | integer | Maximum spindle RPM |
| CNC | `travel_x_min` | number | Minimum X travel (mm) |
| CNC | `travel_x_max` | number | Maximum X travel (mm) |
| CNC | `travel_y_min` | number | Minimum Y travel (mm) |
| CNC | `travel_y_max` | number | Maximum Y travel (mm) |
| CNC | `travel_z_min` | number | Minimum Z travel (mm) |
| CNC | `travel_z_max` | number | Maximum Z travel (mm) |
| CNC | `spindle_taper` | string | Spindle taper type (CT40, BT40, etc.) |
| CNC | `machine_type` | string | `mill`, `lathe`, `router` |
| CNC | `axis_count` | integer | Number of axes (3, 4, 5) |
| CNC | `control_system` | string | Control system brand |
| CNC | `tool_changer_min` | integer | Minimum tool changer capacity |
| Endmill | `diameter_min` | number | Minimum diameter (mm) |
| Endmill | `diameter_max` | number | Maximum diameter (mm) |
| Endmill | `flute_count` | integer | Number of flutes |
| Endmill | `coating` | string | Coating type |
| Endmill | `substrate_material` | string | `hss`, `carbide`, `ceramic` |
| Endmill | `end_type` | string | `square`, `ball_nose`, `bull_nose` |
| Endmill | `in_stock` | boolean | In-stock only |
| Car | `vin` | string | VIN partial match |
| Car | `body_style` | string | `sedan`, `suv`, `truck`, etc. |
| Car | `fuel_type` | string | `gas`, `diesel`, `electric`, `hybrid` |
| Car | `transmission` | string | `manual`, `automatic`, `cvt` |
| Car | `drivetrain` | string | `fwd`, `rwd`, `awd` |
| Car | `mileage_max` | integer | Maximum mileage |
| Car | `mpg_min` | number | Minimum combined MPG |

**Sort Options:**

| Value | Description |
|-------|-------------|
| `relevance` | Search relevance (FTS5 rank) |
| `price_asc` | Price low → high |
| `price_desc` | Price high → low |
| `year_desc` | Newest first |
| `year_asc` | Oldest first |
| `newest` | First seen most recently |
| `deal_score` | Best deals first |
| `condition` | Best condition first |
| `distance` | Nearest first (requires location params) |

**Request Example:**
```http
GET /api/v1/listings?q=haas+vf&domain=cnc_machinery&price_max=50000&sort=deal_score&page=1&per_page=25
```

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
      "domain": "cnc_machinery",
      "title": "Haas VF-2 2019 CNC Vertical Machining Center",
      "make": "Haas",
      "model": "VF-2",
      "year": 2019,
      "price_usd": 45000.00,
      "condition_raw": "Used - Good",
      "condition_score": 0.72,
      "location_text": "Detroit, MI",
      "source": "machinio",
      "source_url": "https://www.machinio.com/listings/123456",
      "deal_score": 0.85,
      "deal_label": "Great Deal",
      "deal_color": "#4ecb71",
      "primary_image_url": "https://cdn.example.com/img1_thumb.jpg",
      "is_bookmarked": false,
      "domain_data": {
        "spindle_rpm_max": 12000,
        "travel_x_mm": 762,
        "travel_y_mm": 406,
        "travel_z_mm": 508,
        "machine_type": "mill",
        "axis_count": 3,
        "tool_changer_cap": 24
      },
      "created_at": "2026-07-11T10:00:00Z",
      "updated_at": "2026-07-11T14:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "total_items": 127,
    "total_pages": 6,
    "has_next": true,
    "has_prev": false
  },
  "meta": {
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:30:00Z",
    "query_time_ms": 42,
    "search_type": "fts5"
  }
}
```

---

### 7.2 Get Single Listing

```
GET /api/v1/listings/{id}
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | Listing ID |

**Request Example:**
```http
GET /api/v1/listings/0197e8a1-2b3c-4d5e-6f78-9abcdef01234
```

**Response: 200 OK**
```json
{
  "data": {
    "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "domain": "cnc_machinery",
    "title": "Haas VF-2 2019 CNC Vertical Machining Center",
    "make": "Haas",
    "model": "VF-2",
    "year": 2019,
    "price_usd": 45000.00,
    "price_original": "€42,500",
    "price_currency": "EUR",
    "condition_raw": "Used - Good",
    "condition_score": 0.72,
    "condition_notes": "Light use, well maintained",
    "location_text": "Detroit, MI",
    "location_country": "US",
    "location_state": "MI",
    "location_city": "Detroit",
    "location_lat": 42.3314,
    "location_lng": -83.0458,
    "source": "machinio",
    "source_url": "https://www.machinio.com/listings/123456",
    "source_id": "machinio_123456",
    "seller_type": "dealer",
    "seller_name": "Midwest Machinery",
    "deal_score": 0.85,
    "deal_label": "Great Deal",
    "deal_color": "#4ecb71",
    "deal_score_detail": {
      "price_vs_market": 0.90,
      "condition_factor": 0.72,
      "age_factor": 0.85,
      "hours_factor": 0.80,
      "source_reputation": 0.95,
      "weighted_total": 0.85
    },
    "image_urls": [
      "https://cdn.example.com/img1.jpg",
      "https://cdn.example.com/img2.jpg",
      "https://cdn.example.com/img3.jpg"
    ],
    "primary_image_url": "https://cdn.example.com/img1.jpg",
    "thumbnail_url": "https://cdn.example.com/img1_thumb.jpg",
    "first_seen_at": "2026-07-11T10:00:00Z",
    "last_seen_at": "2026-07-11T14:30:00Z",
    "is_active": true,
    "is_sold": false,
    "is_bookmarked": false,
    "notes": null,
    "data_quality": 0.88,
    "enrichment_tier": "spec_db",
    "domain_data": {
      "spindle_rpm_max": 12000,
      "spindle_taper": "CT40",
      "spindle_power_kw": 15.0,
      "has_tsc": true,
      "tsc_psi": 1000,
      "travel_x_mm": 762,
      "travel_y_mm": 406,
      "travel_z_mm": 508,
      "travel_format": "3axis",
      "table_diameter_mm": null,
      "max_workpiece_kg": 340,
      "tool_changer_cap": 24,
      "tool_changer_type": "umbrella",
      "control_system": "Haas NGC",
      "control_model": "VF Series",
      "machine_type": "mill",
      "machine_config": "vertical",
      "axis_count": 3,
      "has_pallet_changer": false,
      "has_probing": true,
      "cutting_hours": 4500,
      "operating_hours": 8200,
      "specs_json": null
    },
    "price_history": [
      { "price_usd": 48000.00, "recorded_at": "2026-07-09T10:00:00Z" },
      { "price_usd": 45000.00, "recorded_at": "2026-07-11T10:00:00Z" }
    ],
    "comparables_count": 12,
    "created_at": "2026-07-11T10:00:00Z",
    "updated_at": "2026-07-11T14:30:00Z"
  }
}
```

**Response: 404 Not Found**
```json
{
  "error": {
    "code": "LISTING_NOT_FOUND",
    "message": "Listing with id '0197e8a1-2b3c-4d5e-6f78-9abcdef01234' not found",
    "status": 404,
    "details": [],
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:30:00Z"
  }
}
```

---

### 7.3 Create Listing (Manual)

```
POST /api/v1/listings
```

**Request Body:**
```json
{
  "domain": "cnc_machinery",
  "title": "Custom Listing - Haas VF-2",
  "make": "Haas",
  "model": "VF-2",
  "year": 2019,
  "price_usd": 45000.00,
  "condition_raw": "Used - Good",
  "location_text": "Detroit, MI",
  "source_url": "https://example.com/listing/123",
  "seller_type": "dealer",
  "seller_name": "Midwest Machinery",
  "domain_data": {
    "spindle_rpm_max": 12000,
    "travel_x_mm": 762,
    "travel_y_mm": 406,
    "travel_z_mm": 508
  }
}
```

**Response: 201 Created**
```json
{
  "data": {
    "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "domain": "cnc_machinery",
    "title": "Custom Listing - Haas VF-2",
    "make": "Haas",
    "model": "VF-2",
    "year": 2019,
    "price_usd": 45000.00,
    "condition_raw": "Used - Good",
    "source_url": "https://example.com/listing/123",
    "created_at": "2026-07-11T14:30:00Z",
    "updated_at": "2026-07-11T14:30:00Z"
  }
}
```

**Response: 409 Conflict**
```json
{
  "error": {
    "code": "CONFLICT",
    "message": "Listing with source_url 'https://example.com/listing/123' already exists",
    "status": 409,
    "details": [],
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:30:00Z"
  }
}
```

---

### 7.4 Update Listing

```
PUT /api/v1/listings/{id}
```

**Request Body:** (same schema as Create, all fields optional for partial update)
```json
{
  "price_usd": 42000.00,
  "condition_raw": "Used - Excellent",
  "notes": "Seller confirmed low hours"
}
```

**Response: 200 OK** — returns updated listing (same schema as Get Single)

---

### 7.5 Delete Listing (Soft)

```
DELETE /api/v1/listings/{id}
```

**Response: 204 No Content**

Sets `is_active = false`. Listing remains in database for history.

---

### 7.6 Toggle Bookmark

```
PUT /api/v1/listings/{id}/bookmark
```

**Request Body:**
```json
{
  "is_bookmarked": true
}
```

**Response: 200 OK**
```json
{
  "data": {
    "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "is_bookmarked": true
  }
}
```

---

### 7.7 Update Notes

```
PUT /api/v1/listings/{id}/notes
```

**Request Body:**
```json
{
  "notes": "Seller confirmed low hours. Comes with tooling package. Best offer accepted at $42k."
}
```

**Response: 200 OK**
```json
{
  "data": {
    "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "notes": "Seller confirmed low hours. Comes with tooling package. Best offer accepted at $42k.",
    "updated_at": "2026-07-11T14:35:00Z"
  }
}
```

---

### 7.8 Bulk Operations

```
POST /api/v1/listings/bulk
```

**Request Body:**
```json
{
  "action": "bookmark",
  "listing_ids": [
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01236"
  ],
  "params": {
    "is_bookmarked": true
  }
}
```

**Actions:** `bookmark`, `unbookmark`, `delete`, `enrich`, `set_notes`

**Response: 200 OK**
```json
{
  "data": {
    "processed": 3,
    "succeeded": 3,
    "failed": 0,
    "errors": []
  }
}
```

---

## 8. Vertical/Domain Endpoints

### 8.1 List Verticals

```
GET /api/v1/verticals
```

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "cnc_machinery",
      "display_name": "CNC Machinery",
      "icon": "🏭",
      "enabled": true,
      "total_listings": 12450,
      "active_listings": 8920,
      "sources": ["machinio", "equipnet", "ebay", "surplus_record"],
      "last_scrape_at": "2026-07-11T12:00:00Z"
    },
    {
      "id": "endmill",
      "display_name": "Endmills & Tooling",
      "icon": "🔧",
      "enabled": true,
      "total_listings": 34200,
      "active_listings": 28100,
      "sources": ["ebay"],
      "last_scrape_at": "2026-07-11T13:00:00Z"
    },
    {
      "id": "used_car",
      "display_name": "Used Cars",
      "icon": "🚗",
      "enabled": true,
      "total_listings": 0,
      "active_listings": 0,
      "sources": ["nhtsa", "cargurus"],
      "last_scrape_at": null
    }
  ]
}
```

---

### 8.2 Get Vertical Details

```
GET /api/v1/verticals/{id}
```

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Vertical ID (`cnc_machinery`, `endmill`, `used_car`) |

**Response: 200 OK**
```json
{
  "data": {
    "id": "cnc_machinery",
    "display_name": "CNC Machinery",
    "icon": "🏭",
    "enabled": true,
    "total_listings": 12450,
    "active_listings": 8920,
    "sources": ["machinio", "equipnet", "ebay", "surplus_record"],
    "last_scrape_at": "2026-07-11T12:00:00Z",
    "facets": [
      {
        "name": "spindle_rpm_max",
        "label": "Spindle RPM",
        "type": "range",
        "min_value": 0,
        "max_value": 24000,
        "step": 1000
      },
      {
        "name": "spindle_taper",
        "label": "Spindle Taper",
        "type": "categorical",
        "options": [
          { "value": "CT40", "label": "CT40", "count": 3420 },
          { "value": "BT40", "label": "BT40", "count": 1890 },
          { "value": "CT30", "label": "CT30", "count": 890 },
          { "value": "HSK-A63", "label": "HSK-A63", "count": 234 }
        ]
      },
      {
        "name": "axis_count",
        "label": "Axes",
        "type": "categorical",
        "options": [
          { "value": "3", "label": "3-Axis", "count": 6200 },
          { "value": "4", "label": "4-Axis", "count": 1200 },
          { "value": "5", "label": "5-Axis", "count": 890 }
        ]
      },
      {
        "name": "machine_type",
        "label": "Machine Type",
        "type": "categorical",
        "options": [
          { "value": "mill", "label": "Vertical Mill", "count": 5400 },
          { "value": "lathe", "label": "Lathe", "count": 2100 },
          { "value": "router", "label": "Router", "count": 340 }
        ]
      }
    ],
    "sort_options": ["relevance", "price_asc", "price_desc", "year_desc", "newest", "deal_score"],
    "price_statistics": {
      "min": 2500.00,
      "max": 450000.00,
      "median": 45000.00,
      "mean": 62500.00,
      "count": 8920
    },
    "created_at": "2026-07-11T00:00:00Z"
  }
}
```

**Response: 404 Not Found**
```json
{
  "error": {
    "code": "VERTICAL_NOT_FOUND",
    "message": "Vertical 'invalid' not found",
    "status": 404,
    "details": [],
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:30:00Z"
  }
}
```

---

### 8.3 Get Vertical Filters

```
GET /api/v1/verticals/{id}/filters
```

**Response: 200 OK**
```json
{
  "data": {
    "vertical_id": "cnc_machinery",
    "filters": [
      {
        "name": "spindle_rpm_max",
        "label": "Spindle RPM",
        "type": "range",
        "min_value": 0,
        "max_value": 24000,
        "step": 1000,
        "options": null
      },
      {
        "name": "spindle_taper",
        "label": "Spindle Taper",
        "type": "categorical",
        "options": [
          { "value": "CT40", "label": "CT40", "count": 3420 },
          { "value": "BT40", "label": "BT40", "count": 1890 },
          { "value": "CT30", "label": "CT30", "count": 890 },
          { "value": "HSK-A63", "label": "HSK-A63", "count": 234 }
        ]
      },
      {
        "name": "axis_count",
        "label": "Axes",
        "type": "categorical",
        "options": [
          { "value": "3", "label": "3-Axis", "count": 6200 },
          { "value": "4", "label": "4-Axis", "count": 1200 },
          { "value": "5", "label": "5-Axis", "count": 890 }
        ]
      },
      {
        "name": "control_system",
        "label": "Control System",
        "type": "categorical",
        "options": [
          { "value": "Haas", "label": "Haas", "count": 3100 },
          { "value": "Fanuc", "label": "Fanuc", "count": 2800 },
          { "value": "Siemens", "label": "Siemens", "count": 1200 },
          { "value": "Mitsubishi", "label": "Mitsubishi", "count": 890 }
        ]
      },
      {
        "name": "has_tsc",
        "label": "Through-Spindle Coolant",
        "type": "boolean",
        "options": [
          { "value": "true", "label": "Yes", "count": 4200 },
          { "value": "false", "label": "No", "count": 4720 }
        ]
      }
    ]
  }
}
```

---

### 8.4 Get Vertical Statistics

```
GET /api/v1/verticals/{id}/stats
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `period` | string | `30d` | Time period: `7d`, `30d`, `90d`, `1y`, `all` |

**Response: 200 OK**
```json
{
  "data": {
    "vertical_id": "cnc_machinery",
    "period": "30d",
    "total_listings": 12450,
    "active_listings": 8920,
    "new_listings": 234,
    "sold_listings": 89,
    "price_changes": 156,
    "price_statistics": {
      "min": 2500.00,
      "max": 450000.00,
      "median": 45000.00,
      "mean": 62500.00,
      "std_dev": 38200.00
    },
    "price_trend": {
      "direction": "down",
      "change_pct": -2.3,
      "change_usd": -1400.00
    },
    "top_makes": [
      { "make": "Haas", "count": 3100, "avg_price": 52000 },
      { "make": "DMG Mori", "count": 1800, "avg_price": 95000 },
      { "make": "Mazak", "count": 1200, "avg_price": 78000 },
      { "make": "Okuma", "count": 890, "avg_price": 85000 },
      { "make": "Doosan", "count": 670, "avg_price": 62000 }
    ],
    "top_sources": [
      { "source": "machinio", "count": 5400 },
      { "source": "equipnet", "count": 3200 },
      { "source": "ebay", "count": 2100 },
      { "source": "surplus_record", "count": 1750 }
    ],
    "deal_distribution": {
      "great_deal": 1230,
      "fair_price": 5400,
      "above_market": 2290
    },
    "scrape_activity": {
      "last_24h": 45,
      "last_7d": 312,
      "last_30d": 1247
    }
  }
}
```

---

## 9. Scrape Endpoints

### 9.1 Trigger Scrape

```
POST /api/v1/scrape/{vertical_id}
```

**Request Body:**
```json
{
  "sources": ["machinio", "equipnet"],
  "query": "haas vf",
  "max_pages": 5,
  "force": false,
  "priority": "normal"
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `sources` | string[] | all | Specific sources to scrape (omit for all) |
| `query` | string | — | Custom search query (omit for default) |
| `max_pages` | integer | 3 | Max pages per source |
| `force` | boolean | false | Re-scrape even if recently scraped |
| `priority` | string | `normal` | `low`, `normal`, `high` |

**Response: 202 Accepted**
```json
{
  "data": {
    "job_id": "job_7f8e9d0c1b2a",
    "vertical_id": "cnc_machinery",
    "status": "queued",
    "sources": ["machinio", "equipnet"],
    "created_at": "2026-07-11T14:00:00Z"
  }
}
```

**Response: 409 Conflict**
```json
{
  "error": {
    "code": "JOB_ALREADY_RUNNING",
    "message": "A scrape job is already running for vertical 'cnc_machinery'",
    "status": 409,
    "details": [],
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:00:00Z"
  }
}
```

---

### 9.2 List Scrape Jobs

```
GET /api/v1/scrape/jobs
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `status` | string | — | Filter by status: `queued`, `running`, `completed`, `failed`, `cancelled` |
| `vertical_id` | string | — | Filter by vertical |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Response: 200 OK**
```json
{
  "data": [
    {
      "job_id": "job_7f8e9d0c1b2a",
      "vertical_id": "cnc_machinery",
      "status": "running",
      "progress": {
        "sources_total": 4,
        "sources_completed": 2,
        "sources_failed": 0,
        "listings_found": 234,
        "listings_new": 45,
        "listings_updated": 12
      },
      "started_at": "2026-07-11T14:00:00Z",
      "elapsed_seconds": 432,
      "error": null,
      "created_at": "2026-07-11T14:00:00Z"
    },
    {
      "job_id": "job_3e2d1c0b9a8f",
      "vertical_id": "endmill",
      "status": "completed",
      "progress": {
        "sources_total": 1,
        "sources_completed": 1,
        "sources_failed": 0,
        "listings_found": 1247,
        "listings_new": 89,
        "listings_updated": 234
      },
      "started_at": "2026-07-11T13:00:00Z",
      "elapsed_seconds": 287,
      "error": null,
      "created_at": "2026-07-11T13:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total_items": 15,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  }
}
```

---

### 9.3 Get Job Status

```
GET /api/v1/scrape/jobs/{job_id}
```

**Response: 200 OK**
```json
{
  "data": {
    "job_id": "job_7f8e9d0c1b2a",
    "vertical_id": "cnc_machinery",
    "status": "running",
    "progress": {
      "sources_total": 4,
      "sources_completed": 3,
      "sources_failed": 0,
      "listings_found": 312,
      "listings_new": 67,
      "listings_updated": 23
    },
    "sources": [
      {
        "source_id": "machinio",
        "status": "completed",
        "listings_found": 145,
        "listings_new": 32,
        "elapsed_seconds": 120
      },
      {
        "source_id": "equipnet",
        "status": "completed",
        "listings_found": 89,
        "listings_new": 18,
        "elapsed_seconds": 98
      },
      {
        "source_id": "ebay",
        "status": "running",
        "listings_found": 78,
        "listings_new": 17,
        "elapsed_seconds": 156
      },
      {
        "source_id": "surplus_record",
        "status": "queued",
        "listings_found": 0,
        "listings_new": 0,
        "elapsed_seconds": 0
      }
    ],
    "started_at": "2026-07-11T14:00:00Z",
    "estimated_completion": "2026-07-11T14:12:00Z",
    "elapsed_seconds": 432,
    "error": null,
    "created_at": "2026-07-11T14:00:00Z"
  }
}
```

---

### 9.4 Cancel Job

```
DELETE /api/v1/scrape/jobs/{job_id}
```

**Response: 200 OK**
```json
{
  "data": {
    "job_id": "job_7f8e9d0c1b2a",
    "status": "cancelled",
    "cancelled_at": "2026-07-11T14:07:12Z"
  }
}
```

**Response: 409 Conflict** (if already completed/failed)
```json
{
  "error": {
    "code": "JOB_ALREADY_COMPLETED",
    "message": "Job 'job_7f8e9d0c1b2a' has already completed with status 'completed'",
    "status": 409,
    "details": [],
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:07:12Z"
  }
}
```

---

## 10. Enrichment Endpoints

### 10.1 Trigger Enrichment

```
POST /api/v1/enrich/{vertical_id}
```

**Request Body:**
```json
{
  "listing_ids": [
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01235"
  ],
  "tier": "auto",
  "force": false,
  "batch_size": 50,
  "max_cost_usd": 1.00
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `listing_ids` | UUID[] | all in vertical | Specific listings to enrich |
| `tier` | string | `auto` | `auto`, `rules_only`, `spec_db`, `llm_text`, `llm_vision` |
| `force` | boolean | false | Re-enrich even if already enriched |
| `batch_size` | integer | 50 | Listings per batch |
| `max_cost_usd` | number | 1.00 | Max AI cost for this run |

**Response: 202 Accepted**
```json
{
  "data": {
    "job_id": "job_enrich_a1b2c3d4",
    "vertical_id": "cnc_machinery",
    "status": "queued",
    "total_listings": 2,
    "estimated_cost_usd": 0.004,
    "created_at": "2026-07-11T14:30:00Z"
  }
}
```

---

### 10.2 Get Enrichment Stats

```
GET /api/v1/enrich/stats
```

**Response: 200 OK**
```json
{
  "data": {
    "total_listings": 46550,
    "enrichment_tiers": {
      "none": 2340,
      "rules": 1200,
      "spec_db": 8920,
      "llm_text": 3100,
      "llm_vision": 860
    },
    "total_cached": 15420,
    "cache_hit_rate": 0.87,
    "total_cost_usd": 12.45,
    "cost_today_usd": 0.32,
    "cost_by_model": {
      "deepseek/deepseek-chat": 8.20,
      "openai/gpt-4o-mini": 4.25
    },
    "tokens_used_by_model": {
      "deepseek/deepseek-chat": 2048000,
      "openai/gpt-4o-mini": 512000
    },
    "avg_enrichment_time_ms": 342,
    "last_enrichment_at": "2026-07-11T14:25:00Z"
  }
}
```

---

### 10.3 Get Cache Statistics

```
GET /api/v1/enrich/cache
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | string | — | Filter by cache type: `text`, `vision` |

**Response: 200 OK**
```json
{
  "data": {
    "total_entries": 15420,
    "total_size_bytes": 204857600,
    "total_size_human": "195 MB",
    "entries_by_type": {
      "text": 14200,
      "vision": 1220
    },
    "avg_entry_size_bytes": 13280,
    "oldest_entry_at": "2026-01-15T10:00:00Z",
    "newest_entry_at": "2026-07-11T14:25:00Z",
    "hit_rate_7d": 0.89,
    "hit_rate_30d": 0.87,
    "cost_saved_usd": 45.20
  }
}
```

---

### 10.4 Clear Cache

```
DELETE /api/v1/enrich/cache
```

**Request Body:**
```json
{
  "older_than_days": 90,
  "type": "text",
  "confirm": true
}
```

**Response: 200 OK**
```json
{
  "data": {
    "deleted_count": 3420,
    "freed_bytes": 45678900,
    "freed_human": "43.6 MB"
  }
}
```

---

## 11. Valuation Endpoints

### 11.1 Estimate Price

```
POST /api/v1/valuation/estimate
```

**Request Body:**
```json
{
  "domain": "cnc_machinery",
  "make": "Haas",
  "model": "VF-2",
  "year": 2019,
  "condition_score": 0.72,
  "domain_data": {
    "spindle_rpm_max": 12000,
    "travel_x_mm": 762,
    "cutting_hours": 4500
  },
  "location_lat": 42.3314,
  "location_lng": -83.0458
}
```

**Response: 200 OK**
```json
{
  "data": {
    "estimated_price_usd": 42500.00,
    "confidence": 0.82,
    "confidence_label": "High",
    "price_range": {
      "low": 35000.00,
      "high": 52000.00
    },
    "comparables_used": 12,
    "market_average": 48000.00,
    "market_median": 45000.00,
    "depreciation_curve": {
      "2020": 65000.00,
      "2021": 58000.00,
      "2022": 52000.00,
      "2023": 48000.00,
      "2024": 45000.00,
      "2025": 43000.00,
      "2026": 42500.00
    },
    "factors": {
      "base_value": 48000.00,
      "age_adjustment": -5200.00,
      "hours_adjustment": -300.00,
      "condition_adjustment": 0.00,
      "location_adjustment": 0.00,
      "market_trend_adjustment": 0.00
    },
    "methodology": "multi_factor_weighted"
  }
}
```

---

### 11.2 Get Comparables

```
GET /api/v1/valuation/comparables/{listing_id}
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 10 | Max comparables to return |
| `max_distance_miles` | number | 200 | Max distance for location-based |
| `max_price_deviation` | number | 0.3 | Max price deviation (30%) |

**Response: 200 OK**
```json
{
  "data": {
    "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "original_price_usd": 45000.00,
    "comparables": [
      {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
        "title": "Haas VF-2 2018 CNC Vertical Machining Center",
        "make": "Haas",
        "model": "VF-2",
        "year": 2018,
        "price_usd": 38000.00,
        "condition_score": 0.65,
        "location_text": "Chicago, IL",
        "source": "machinio",
        "similarity_score": 0.94,
        "price_vs_original": -0.156,
        "domain_data": {
          "spindle_rpm_max": 12000,
          "travel_x_mm": 762,
          "cutting_hours": 6200
        }
      },
      {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01236",
        "title": "Haas VF-2 2020 CNC Vertical Machining Center",
        "make": "Haas",
        "model": "VF-2",
        "year": 2020,
        "price_usd": 52000.00,
        "condition_score": 0.82,
        "location_text": "Columbus, OH",
        "source": "equipnet",
        "similarity_score": 0.91,
        "price_vs_original": 0.156,
        "domain_data": {
          "spindle_rpm_max": 12000,
          "travel_x_mm": 762,
          "cutting_hours": 2800
        }
      }
    ],
    "statistics": {
      "count": 12,
      "avg_price": 44500.00,
      "median_price": 43000.00,
      "min_price": 35000.00,
      "max_price": 55000.00,
      "std_dev": 5200.00
    }
  }
}
```

---

## 12. Alert Endpoints

### 12.1 List Alerts

```
GET /api/v1/alerts
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `domain` | string | — | Filter by domain |
| `enabled` | boolean | — | Filter by enabled status |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 25 | Items per page |

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": 1,
      "name": "Cheap Haas VF-2",
      "enabled": true,
      "domain": "cnc_machinery",
      "conditions": {
        "make": { "op": "equals", "value": "Haas" },
        "model": { "op": "contains", "value": "VF-2" },
        "price_usd": { "op": "lte", "value": 30000 }
      },
      "actions": [
        { "type": "webhook", "url": "https://hooks.slack.com/services/T00/B00/xxx" }
      ],
      "notify_on": "new_match",
      "last_triggered_at": "2026-07-10T08:15:00Z",
      "trigger_count": 3,
      "created_at": "2026-07-01T10:00:00Z",
      "updated_at": "2026-07-01T10:00:00Z"
    },
    {
      "id": 2,
      "name": "Any 5-Axis Under $100k",
      "enabled": true,
      "domain": "cnc_machinery",
      "conditions": {
        "domain_data.axis_count": { "op": "gte", "value": 5 },
        "price_usd": { "op": "lte", "value": 100000 }
      },
      "actions": [
        { "type": "webhook", "url": "https://hooks.slack.com/services/T00/B00/yyy" }
      ],
      "notify_on": "new_match",
      "last_triggered_at": null,
      "trigger_count": 0,
      "created_at": "2026-07-05T14:00:00Z",
      "updated_at": "2026-07-05T14:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "total_items": 2,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  }
}
```

---

### 12.2 Create Alert

```
POST /api/v1/alerts
```

**Request Body:**
```json
{
  "name": "Cheap Haas VF-2",
  "enabled": true,
  "domain": "cnc_machinery",
  "conditions": {
    "make": { "op": "equals", "value": "Haas" },
    "model": { "op": "contains", "value": "VF-2" },
    "price_usd": { "op": "lte", "value": 30000 }
  },
  "actions": [
    { "type": "webhook", "url": "https://hooks.slack.com/services/T00/B00/xxx" }
  ],
  "notify_on": "new_match"
}
```

**Condition Operators:**

| Operator | Description | Value Type |
|----------|-------------|------------|
| `equals` | Exact match | string, number |
| `not_equals` | Not equal | string, number |
| `contains` | Substring match | string |
| `gt` | Greater than | number |
| `gte` | Greater than or equal | number |
| `lt` | Less than | number |
| `lte` | Less than or equal | number |
| `in` | Value in list | array |
| `between` | Value between [min, max] | array[2] |

**Action Types:**

| Type | Required Fields | Optional Fields |
|------|----------------|-----------------|
| `webhook` | `url` | `headers`, `method` |
| `log` | — | `level` |

**Response: 201 Created**
```json
{
  "data": {
    "id": 3,
    "name": "Cheap Haas VF-2",
    "enabled": true,
    "domain": "cnc_machinery",
    "conditions": {
      "make": { "op": "equals", "value": "Haas" },
      "model": { "op": "contains", "value": "VF-2" },
      "price_usd": { "op": "lte", "value": 30000 }
    },
    "actions": [
      { "type": "webhook", "url": "https://hooks.slack.com/services/T00/B00/xxx" }
    ],
    "notify_on": "new_match",
    "last_triggered_at": null,
    "trigger_count": 0,
    "created_at": "2026-07-11T14:30:00Z",
    "updated_at": "2026-07-11T14:30:00Z"
  }
}
```

**Response: 409 Conflict**
```json
{
  "error": {
    "code": "CONFLICT",
    "message": "Alert with name 'Cheap Haas VF-2' already exists",
    "status": 409,
    "details": [],
    "request_id": "req_abc123def456",
    "timestamp": "2026-07-11T14:30:00Z"
  }
}
```

---

### 12.3 Update Alert

```
PUT /api/v1/alerts/{id}
```

**Request Body:** (same as Create, all fields optional)
```json
{
  "enabled": false,
  "conditions": {
    "make": { "op": "equals", "value": "Haas" },
    "model": { "op": "contains", "value": "VF-2" },
    "price_usd": { "op": "lte", "value": 25000 }
  }
}
```

**Response: 200 OK** — returns updated alert (same schema as Create)

---

### 12.4 Delete Alert

```
DELETE /api/v1/alerts/{id}
```

**Response: 204 No Content**

---

### 12.5 Test Alert

```
POST /api/v1/alerts/{id}/test
```

**Response: 200 OK**
```json
{
  "data": {
    "alert_id": 1,
    "alert_name": "Cheap Haas VF-2",
    "matches_found": 3,
    "matches": [
      {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
        "title": "Haas VF-2 2019 CNC Vertical Machining Center",
        "price_usd": 28000.00,
        "source": "machinio",
        "matched_conditions": ["make", "model", "price_usd"]
      },
      {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
        "title": "Haas VF-2SS 2018 CNC Vertical Machining Center",
        "price_usd": 29500.00,
        "source": "equipnet",
        "matched_conditions": ["make", "model", "price_usd"]
      },
      {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01236",
        "title": "Haas VF-2 2017 CNC Vertical Machining Center",
        "price_usd": 22000.00,
        "source": "ebay",
        "matched_conditions": ["make", "model", "price_usd"]
      }
    ],
    "would_notify": true,
    "would_notify_reason": "3 new matches found since last check"
  }
}
```

---

### 12.6 Get Alert History

```
GET /api/v1/alerts/history
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `alert_id` | integer | — | Filter by specific alert |
| `page` | integer | 1 | Page number |
| `per_page` | integer | 25 | Items per page |

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": 15,
      "alert_id": 1,
      "alert_name": "Cheap Haas VF-2",
      "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
      "listing_title": "Haas VF-2 2019 CNC Vertical Machining Center",
      "listing_price_usd": 28000.00,
      "matched_conditions": ["make", "model", "price_usd"],
      "action_taken": "webhook",
      "action_result": "success",
      "notified_at": "2026-07-10T08:15:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 25,
    "total_items": 15,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  }
}
```

---

## 13. Search Endpoints

### 13.1 Autocomplete Suggestions

```
GET /api/v1/search/suggest
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | string | — | Partial search query (min 2 chars) |
| `domain` | string | — | Limit to specific domain |
| `limit` | integer | 8 | Max suggestions |

**Response: 200 OK**
```json
{
  "data": {
    "suggestions": [
      { "text": "Haas VF-2", "type": "make_model", "domain": "cnc_machinery", "count": 342 },
      { "text": "Haas VF-2SS", "type": "make_model", "domain": "cnc_machinery", "count": 89 },
      { "text": "Haas VF-4", "type": "make_model", "domain": "cnc_machinery", "count": 234 },
      { "text": "Haas VF-6", "type": "make_model", "domain": "cnc_machinery", "count": 156 },
      { "text": "Haas Mini Mill", "type": "make_model", "domain": "cnc_machinery", "count": 112 },
      { "text": "Haas UMC-750", "type": "make_model", "domain": "cnc_machinery", "count": 67 },
      { "text": "Haas ST-20", "type": "make_model", "domain": "cnc_machinery", "count": 45 },
      { "text": "Haas EC-1600", "type": "make_model", "domain": "cnc_machinery", "count": 23 }
    ],
    "query_time_ms": 12
  }
}
```

---

### 13.2 Faceted Counts

```
GET /api/v1/search/facets
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | string | — | Base search query |
| `domain` | string | — | Limit to specific domain |
| `filters` | string | — | JSON-encoded active filters (for recomputing counts) |

**Response: 200 OK**
```json
{
  "data": {
    "total_results": 12450,
    "facets": {
      "domain": [
        { "value": "cnc_machinery", "label": "CNC Machinery", "count": 8920 },
        { "value": "endmill", "label": "Endmills & Tooling", "count": 34200 },
        { "value": "used_car", "label": "Used Cars", "count": 0 }
      ],
      "price_range": [
        { "value": "0-10000", "label": "Under $10,000", "count": 1230 },
        { "value": "10000-25000", "label": "$10,000 - $25,000", "count": 2340 },
        { "value": "25000-50000", "label": "$25,000 - $50,000", "count": 3100 },
        { "value": "50000-100000", "label": "$50,000 - $100,000", "count": 2890 },
        { "value": "100000-500000", "label": "$100,000+", "count": 1890 }
      ],
      "condition": [
        { "value": "0.8-1.0", "label": "Excellent", "count": 2340 },
        { "value": "0.6-0.8", "label": "Good", "count": 4560 },
        { "value": "0.4-0.6", "label": "Fair", "count": 1230 },
        { "value": "0.0-0.4", "label": "Poor", "count": 790 }
      ],
      "source": [
        { "value": "machinio", "label": "Machinio", "count": 5400 },
        { "value": "equipnet", "label": "EquipNet", "count": 3200 },
        { "value": "ebay", "label": "eBay", "count": 2100 },
        { "value": "surplus_record", "label": "Surplus Record", "count": 1750 }
      ]
    }
  }
}
```

---

### 13.3 Price History

```
GET /api/v1/search/price-history
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `domain` | string | — | Domain filter |
| `make` | string | — | Make/brand filter |
| `model` | string | — | Model filter |
| `period` | string | `30d` | Time period: `7d`, `30d`, `90d`, `1y`, `all` |
| `bucket` | string | `auto` | Bucket size: `day`, `week`, `month`, `auto` |

**Response: 200 OK**
```json
{
  "data": {
    "domain": "cnc_machinery",
    "make": "Haas",
    "model": "VF-2",
    "period": "30d",
    "bucket": "week",
    "summary": {
      "min_price": 28000.00,
      "max_price": 55000.00,
      "avg_price": 42500.00,
      "median_price": 41000.00,
      "trend": "down",
      "trend_pct": -2.3,
      "sample_size": 89
    },
    "buckets": [
      {
        "date": "2026-06-14",
        "min": 30000.00,
        "max": 52000.00,
        "avg": 43500.00,
        "median": 42000.00,
        "count": 22
      },
      {
        "date": "2026-06-21",
        "min": 28000.00,
        "max": 55000.00,
        "avg": 43000.00,
        "median": 41500.00,
        "count": 25
      },
      {
        "date": "2026-06-28",
        "min": 29000.00,
        "max": 53000.00,
        "avg": 42500.00,
        "median": 41000.00,
        "count": 21
      },
      {
        "date": "2026-07-05",
        "min": 28000.00,
        "max": 54000.00,
        "avg": 42000.00,
        "median": 40500.00,
        "count": 21
      }
    ]
  }
}
```

---

## 14. Compare Endpoints

### 14.1 Compare Listings

```
POST /api/v1/compare
```

**Request Body:**
```json
{
  "listing_ids": [
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
    "0197e8a1-2b3c-4d5e-6f78-9abcdef01236"
  ]
}
```

**Response: 200 OK**
```json
{
  "data": {
    "listings": [
      {
        "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
        "title": "Haas VF-2 2019 CNC Vertical Machining Center",
        "make": "Haas",
        "model": "VF-2",
        "year": 2019,
        "price_usd": 45000.00,
        "condition_score": 0.72,
        "location_text": "Detroit, MI",
        "source": "machinio",
        "deal_score": 0.85,
        "primary_image_url": "https://cdn.example.com/img1_thumb.jpg",
        "domain_data": {
          "spindle_rpm_max": 12000,
          "travel_x_mm": 762,
          "travel_y_mm": 406,
          "travel_z_mm": 508,
          "axis_count": 3,
          "tool_changer_cap": 24,
          "cutting_hours": 4500,
          "control_system": "Haas NGC"
        }
      },
      {
        "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
        "title": "Haas VF-2SS 2020 CNC Vertical Machining Center",
        "make": "Haas",
        "model": "VF-2SS",
        "year": 2020,
        "price_usd": 58000.00,
        "condition_score": 0.88,
        "location_text": "Chicago, IL",
        "source": "equipnet",
        "deal_score": 0.72,
        "primary_image_url": "https://cdn.example.com/img2_thumb.jpg",
        "domain_data": {
          "spindle_rpm_max": 12000,
          "travel_x_mm": 762,
          "travel_y_mm": 406,
          "travel_z_mm": 508,
          "axis_count": 3,
          "tool_changer_cap": 24,
          "cutting_hours": 1800,
          "control_system": "Haas NGC"
        }
      },
      {
        "id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01236",
        "title": "DMG Mori CMX 50U 2019 5-Axis VMC",
        "make": "DMG Mori",
        "model": "CMX 50U",
        "year": 2019,
        "price_usd": 95000.00,
        "condition_score": 0.82,
        "location_text": "Milwaukee, WI",
        "source": "machinio",
        "deal_score": 0.68,
        "primary_image_url": "https://cdn.example.com/img3_thumb.jpg",
        "domain_data": {
          "spindle_rpm_max": 18000,
          "travel_x_mm": 500,
          "travel_y_mm": 450,
          "travel_z_mm": 400,
          "axis_count": 5,
          "tool_changer_cap": 60,
          "cutting_hours": 3200,
          "control_system": "Siemens"
        }
      }
    ],
    "comparison": {
      "common_fields": ["make", "model", "year", "spindle_rpm_max", "axis_count"],
      "best_price": {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
        "value": 45000.00
      },
      "best_condition": {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
        "value": 0.88
      },
      "best_deal": {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01234",
        "value": 0.85
      },
      "newest": {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
        "value": 2020
      },
      "lowest_hours": {
        "listing_id": "0197e8a1-2b3c-4d5e-6f78-9abcdef01235",
        "value": 1800
      }
    }
  }
}
```

---

## 15. System Endpoints

### 15.1 Health Check

```
GET /api/v1/health
```

**Response: 200 OK**
```json
{
  "data": {
    "status": "healthy",
    "version": "1.0.0",
    "uptime_seconds": 86400,
    "database": {
      "status": "connected",
      "size_bytes": 104857600,
      "size_human": "100 MB",
      "wal_mode": true
    },
    "scraper": {
      "status": "idle",
      "active_jobs": 0,
      "blocked_sources": 0
    },
    "ai": {
      "status": "available",
      "model": "deepseek/deepseek-chat",
      "cost_today_usd": 0.32
    }
  }
}
```

---

### 15.2 System Statistics

```
GET /api/v1/stats
```

**Response: 200 OK**
```json
{
  "data": {
    "listings": {
      "total": 46550,
      "active": 37020,
      "sold": 8200,
      "inactive": 1330,
      "by_domain": {
        "cnc_machinery": 12450,
        "endmill": 34100,
        "used_car": 0
      }
    },
    "scraping": {
      "total_jobs_run": 2340,
      "total_listings_found": 125000,
      "avg_listings_per_job": 53,
      "last_scrape_at": "2026-07-11T14:00:00Z"
    },
    "enrichment": {
      "total_enriched": 15420,
      "total_cost_usd": 12.45,
      "cache_entries": 15420,
      "cache_hit_rate": 0.87
    },
    "alerts": {
      "total_rules": 12,
      "active_rules": 8,
      "total_triggers": 45,
      "last_trigger_at": "2026-07-10T08:15:00Z"
    },
    "database": {
      "size_bytes": 104857600,
      "size_human": "100 MB",
      "tables": {
        "products": 46550,
        "cnc_machines": 12450,
        "endmills": 34100,
        "used_cars": 0,
        "price_history": 8920,
        "ai_cache": 15420,
        "alert_rules": 12,
        "alert_history": 45
      }
    },
    "system": {
      "python_version": "3.12.4",
      "fastapi_version": "0.115.0",
      "uptime_seconds": 86400,
      "last_restart_at": "2026-07-10T14:30:00Z"
    }
  }
}
```

---

### 15.3 Connector Status

```
GET /api/v1/connectors
```

**Response: 200 OK**
```json
{
  "data": [
    {
      "connector_id": "machinio",
      "display_name": "Machinio",
      "type": "web_scraper",
      "vertical": "cnc_machinery",
      "enabled": true,
      "healthy": true,
      "status": "idle",
      "last_check_at": "2026-07-11T14:30:00Z",
      "last_scrape_at": "2026-07-11T12:00:00Z",
      "last_error": null,
      "consecutive_failures": 0,
      "total_scrapes": 1247,
      "total_listings_found": 5400,
      "avg_response_ms": 320,
      "rate_limit": {
        "rpm": 30,
        "current_rpm": 5
      }
    },
    {
      "connector_id": "equipnet",
      "display_name": "EquipNet",
      "type": "web_scraper",
      "vertical": "cnc_machinery",
      "enabled": true,
      "healthy": true,
      "status": "idle",
      "last_check_at": "2026-07-11T14:30:00Z",
      "last_scrape_at": "2026-07-11T11:30:00Z",
      "last_error": null,
      "consecutive_failures": 0,
      "total_scrapes": 890,
      "total_listings_found": 3200,
      "avg_response_ms": 450,
      "rate_limit": {
        "rpm": 20,
        "current_rpm": 3
      }
    },
    {
      "connector_id": "ebay",
      "display_name": "eBay",
      "type": "api",
      "vertical": "endmill",
      "enabled": true,
      "healthy": true,
      "status": "idle",
      "last_check_at": "2026-07-11T14:30:00Z",
      "last_scrape_at": "2026-07-11T13:00:00Z",
      "last_error": null,
      "consecutive_failures": 0,
      "total_scrapes": 450,
      "total_listings_found": 34100,
      "avg_response_ms": 180,
      "rate_limit": {
        "rpm": 5000,
        "current_rpm": 12
      }
    },
    {
      "connector_id": "nhtsa",
      "display_name": "NHTSA",
      "type": "api",
      "vertical": "used_car",
      "enabled": true,
      "healthy": true,
      "status": "idle",
      "last_check_at": "2026-07-11T14:30:00Z",
      "last_scrape_at": null,
      "last_error": null,
      "consecutive_failures": 0,
      "total_scrapes": 0,
      "total_listings_found": 0,
      "avg_response_ms": null,
      "rate_limit": {
        "rpm": 1000,
        "current_rpm": 0
      }
    }
  ]
}
```

---

## 16. HTMX Fragment Endpoints

> **Note:** SearchForge initially ships as a Vanilla JS SPA. These HTMX endpoints are specified for a future server-rendered mode where the frontend uses HTMX for partial page updates. All endpoints return HTML fragments, not JSON.

### 16.1 Main Page (Full HTML)

```
GET /
```

**Response: 200 OK** — Full HTML page (`text/html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>SearchForge — Universal Shopping Search</title>
  <link rel="stylesheet" href="/css/tokens.css">
  <link rel="stylesheet" href="/css/tokens-searchforge.css">
  <link rel="stylesheet" href="/css/app.css">
</head>
<body>
  <div class="sf-app" id="app">
    <header class="sf-titlebar">...</header>
    <nav class="sf-sidebar">...</nav>
    <main class="sf-content" id="content">
      <!-- HTMX swaps happen here -->
      <div hx-get="/search" hx-trigger="load" hx-target="#content"></div>
    </main>
    <footer class="sf-statusbar">...</footer>
  </div>
  <script src="https://unpkg.com/htmx.org@2.0.0"></script>
</body>
</html>
```

---

### 16.2 Search Results Fragment

```
GET /search
```

**Query Parameters:** Same as `GET /api/v1/listings` (q, domain, filters, sort, page, per_page)

**Response: 200 OK** — HTML fragment

```html
<div class="sf-search-results"
     hx-get="/search"
     hx-trigger="keyup changed delay:300ms from:[name='q']"
     hx-include="[name='q'], [name='domain'], [name='sort']"
     hx-target="#search-results"
     hx-swap="innerHTML">

  <div class="sf-results-header">
    <span class="sf-results-count">Showing 1–25 of 127 results</span>
    <select name="sort" class="sf-select"
            hx-get="/search"
            hx-include="[name='q'], [name='domain'], [name='sort']"
            hx-target="#search-results">
      <option value="relevance">Relevance</option>
      <option value="price_asc">Price: Low → High</option>
      <option value="price_desc">Price: High → Low</option>
      <option value="deal_score">Best Deals</option>
      <option value="newest">Newest First</option>
    </select>
  </div>

  <div id="search-results" class="sf-results-grid">
    <!-- Result cards rendered here -->
    <div class="sf-result-card" data-domain="cnc_machinery">
      <img src="https://cdn.example.com/img1_thumb.jpg" alt="Haas VF-2" class="sf-result-image">
      <div class="sf-result-body">
        <span class="sf-domain-badge sf-domain-cnc">CNC Machinery</span>
        <h3 class="sf-result-title">
          <a href="/listing/0197e8a1-2b3c-4d5e-6f78-9abcdef01234"
             hx-get="/listing/0197e8a1-2b3c-4d5e-6f78-9abcdef01234"
             hx-target="#detail-drawer"
             hx-swap="innerHTML">
            Haas VF-2 2019 CNC Vertical Machining Center
          </a>
        </h3>
        <div class="sf-result-meta">
          <span class="sf-result-price">$45,000</span>
          <span class="sf-result-condition sf-condition-good">Good</span>
          <span class="sf-result-deal sf-deal-great">Great Deal</span>
        </div>
        <div class="sf-result-specs">
          <span>12,000 RPM</span>
          <span>762×406×508 mm</span>
          <span>3-Axis</span>
        </div>
        <div class="sf-result-footer">
          <span class="sf-result-source">Machinio</span>
          <span class="sf-result-location">Detroit, MI</span>
          <button class="sf-btn-bookmark"
                  hx-put="/api/v1/listings/0197e8a1-2b3c-4d5e-6f78-9abcdef01234/bookmark"
                  hx-vals='{"is_bookmarked": true}'
                  hx-swap="outerHTML">
            ☆ Bookmark
          </button>
        </div>
      </div>
    </div>
  </div>

  <div class="sf-pagination">
    <a href="/search?page=2"
       hx-get="/search?page=2"
       hx-target="#search-results"
       hx-swap="innerHTML">
      Next Page →
    </a>
  </div>
</div>
```

---

### 16.3 Filter Panel Fragment

```
GET /filters
```

**Query Parameters:** Same as search (domain, active filters)

**Response: 200 OK** — HTML fragment

```html
<div class="sf-filter-panel"
     hx-get="/filters"
     hx-trigger="load"
     hx-target="#filter-panel"
     hx-swap="innerHTML">

  <h3 class="sf-filter-title">Filters</h3>

  <!-- Cross-domain filters -->
  <div class="sf-filter-group">
    <label class="sf-filter-label">Domain</label>
    <div class="sf-filter-options">
      <label class="sf-checkbox">
        <input type="checkbox" name="domain" value="cnc_machinery"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>CNC Machinery <span class="sf-filter-count">(8,920)</span></span>
      </label>
      <label class="sf-checkbox">
        <input type="checkbox" name="domain" value="endmill"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>Endmills & Tooling <span class="sf-filter-count">(34,200)</span></span>
      </label>
      <label class="sf-checkbox">
        <input type="checkbox" name="domain" value="used_car"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>Used Cars <span class="sf-filter-count">(0)</span></span>
      </label>
    </div>
  </div>

  <!-- Price Range -->
  <div class="sf-filter-group">
    <label class="sf-filter-label">Price Range</label>
    <div class="sf-range-inputs">
      <input type="number" name="price_min" placeholder="Min" class="sf-input-sm"
             hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
      <span class="sf-range-separator">–</span>
      <input type="number" name="price_max" placeholder="Max" class="sf-input-sm"
             hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
    </div>
  </div>

  <!-- CNC-specific filters (shown only when domain=cnc_machinery) -->
  <div class="sf-filter-group" data-domain="cnc_machinery">
    <label class="sf-filter-label">Spindle RPM</label>
    <div class="sf-range-inputs">
      <input type="number" name="spindle_rpm_min" placeholder="Min" class="sf-input-sm"
             hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
      <span class="sf-range-separator">–</span>
      <input type="number" name="spindle_rpm_max" placeholder="Max" class="sf-input-sm"
             hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
    </div>
  </div>

  <div class="sf-filter-group" data-domain="cnc_machinery">
    <label class="sf-filter-label">Axes</label>
    <div class="sf-filter-options">
      <label class="sf-checkbox">
        <input type="checkbox" name="axis_count" value="3"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>3-Axis <span class="sf-filter-count">(6,200)</span></span>
      </label>
      <label class="sf-checkbox">
        <input type="checkbox" name="axis_count" value="4"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>4-Axis <span class="sf-filter-count">(1,200)</span></span>
      </label>
      <label class="sf-checkbox">
        <input type="checkbox" name="axis_count" value="5"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>5-Axis <span class="sf-filter-count">(890)</span></span>
      </label>
    </div>
  </div>

  <!-- Car-specific filters (shown only when domain=used_car) -->
  <div class="sf-filter-group" data-domain="used_car">
    <label class="sf-filter-label">Body Style</label>
    <div class="sf-filter-options">
      <label class="sf-checkbox">
        <input type="checkbox" name="body_style" value="sedan"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>Sedan</span>
      </label>
      <label class="sf-checkbox">
        <input type="checkbox" name="body_style" value="suv"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>SUV</span>
      </label>
      <label class="sf-checkbox">
        <input type="checkbox" name="body_style" value="truck"
               hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
        <span>Truck</span>
      </label>
    </div>
  </div>

  <div class="sf-filter-group" data-domain="used_car">
    <label class="sf-filter-label">Mileage</label>
    <div class="sf-range-inputs">
      <input type="number" name="mileage_max" placeholder="Max miles" class="sf-input-sm"
             hx-get="/search" hx-target="#search-results" hx-swap="innerHTML">
    </div>
  </div>

  <button class="sf-btn sf-btn-secondary"
          hx-get="/filters?reset=true"
          hx-target="#filter-panel"
          hx-swap="innerHTML">
    Clear All Filters
  </button>
</div>
```

---

### 16.4 Listing Detail Fragment

```
GET /listing/{id}
```

**Response: 200 OK** — HTML fragment

```html
<div class="sf-detail-drawer" id="detail-drawer">
  <button class="sf-close-btn"
          hx-get="/search"
          hx-target="#content"
          hx-swap="innerHTML">
    ✕
  </button>

  <div class="sf-detail-header">
    <span class="sf-domain-badge sf-domain-cnc">CNC Machinery</span>
    <h2 class="sf-detail-title">Haas VF-2 2019 CNC Vertical Machining Center</h2>
    <div class="sf-detail-price-row">
      <span class="sf-detail-price">$45,000</span>
      <span class="sf-detail-deal sf-deal-great">Great Deal (0.85)</span>
    </div>
  </div>

  <div class="sf-detail-images">
    <img src="https://cdn.example.com/img1.jpg" alt="Haas VF-2" class="sf-detail-image-main">
    <div class="sf-detail-thumbnails">
      <img src="https://cdn.example.com/img1_thumb.jpg" alt="" class="sf-thumb active">
      <img src="https://cdn.example.com/img2_thumb.jpg" alt="" class="sf-thumb">
      <img src="https://cdn.example.com/img3_thumb.jpg" alt="" class="sf-thumb">
    </div>
  </div>

  <div class="sf-detail-actions">
    <button class="sf-btn sf-btn-primary"
            hx-put="/api/v1/listings/0197e8a1-2b3c-4d5e-6f78-9abcdef01234/bookmark"
            hx-vals='{"is_bookmarked": true}'>
      ☆ Bookmark
    </button>
    <a href="https://www.machinio.com/listings/123456" target="_blank" class="sf-btn sf-btn-secondary">
      View Source →
    </a>
  </div>

  <div class="sf-detail-specs">
    <h3>Specifications</h3>
    <table class="sf-spec-table">
      <tr><td>Make</td><td>Haas</td></tr>
      <tr><td>Model</td><td>VF-2</td></tr>
      <tr><td>Year</td><td>2019</td></tr>
      <tr><td>Spindle RPM</td><td>12,000</td></tr>
      <tr><td>Spindle Taper</td><td>CT40</td></tr>
      <tr><td>Travel X</td><td>762 mm</td></tr>
      <tr><td>Travel Y</td><td>406 mm</td></tr>
      <tr><td>Travel Z</td><td>508 mm</td></tr>
      <tr><td>Axes</td><td>3</td></tr>
      <tr><td>Tool Changer</td><td>24-position umbrella</td></tr>
      <tr><td>Control</td><td>Haas NGC</td></tr>
      <tr><td>Cutting Hours</td><td>4,500</td></tr>
      <tr><td>Operating Hours</td><td>8,200</td></tr>
    </table>
  </div>

  <div class="sf-detail-comparables"
       hx-get="/api/v1/valuation/comparables/0197e8a1-2b3c-4d5e-6f78-9abcdef01234?limit=3"
       hx-target="#comparables-section"
       hx-swap="innerHTML"
       hx-trigger="load">
    <h3>Comparables</h3>
    <div id="comparables-section">Loading...</div>
  </div>

  <div class="sf-detail-notes">
    <h3>Notes</h3>
    <textarea class="sf-textarea"
              hx-put="/api/v1/listings/0197e8a1-2b3c-4d5e-6f78-9abcdef01234/notes"
              hx-trigger="blur"
              hx-vals='{"notes": this.value}'
              placeholder="Add notes..."></textarea>
  </div>
</div>
```

---

### 16.5 Comparison View Fragment

```
GET /compare
```

**Query Parameters:** `ids` — comma-separated listing IDs

**Response: 200 OK** — HTML fragment

```html
<div class="sf-comparison-view">
  <h2 class="sf-comparison-title">Compare Listings</h2>

  <div class="sf-comparison-table-wrapper">
    <table class="sf-comparison-table">
      <thead>
        <tr>
          <th class="sf-comparison-label-col">Feature</th>
          <th class="sf-comparison-item-col">
            <img src="https://cdn.example.com/img1_thumb.jpg" alt="" class="sf-comparison-thumb">
            <span>Haas VF-2 (2019)</span>
            <button class="sf-comparison-remove"
                    hx-get="/compare?ids=2,3"
                    hx-target="#content"
                    hx-swap="innerHTML">✕</button>
          </th>
          <th class="sf-comparison-item-col">
            <img src="https://cdn.example.com/img2_thumb.jpg" alt="" class="sf-comparison-thumb">
            <span>Haas VF-2SS (2020)</span>
            <button class="sf-comparison-remove"
                    hx-get="/compare?ids=1,3"
                    hx-target="#content"
                    hx-swap="innerHTML">✕</button>
          </th>
          <th class="sf-comparison-item-col">
            <img src="https://cdn.example.com/img3_thumb.jpg" alt="" class="sf-comparison-thumb">
            <span>DMG Mori CMX 50U (2019)</span>
            <button class="sf-comparison-remove"
                    hx-get="/compare?ids=1,2"
                    hx-target="#content"
                    hx-swap="innerHTML">✕</button>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td class="sf-comparison-label">Price</td>
          <td class="sf-comparison-value sf-best">$45,000</td>
          <td class="sf-comparison-value">$58,000</td>
          <td class="sf-comparison-value">$95,000</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Condition</td>
          <td class="sf-comparison-value">0.72 (Good)</td>
          <td class="sf-comparison-value sf-best">0.88 (Excellent)</td>
          <td class="sf-comparison-value">0.82 (Good)</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Deal Score</td>
          <td class="sf-comparison-value sf-best">0.85 (Great Deal)</td>
          <td class="sf-comparison-value">0.72 (Fair)</td>
          <td class="sf-comparison-value">0.68 (Above Market)</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Year</td>
          <td class="sf-comparison-value">2019</td>
          <td class="sf-comparison-value sf-best">2020</td>
          <td class="sf-comparison-value">2019</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Spindle RPM</td>
          <td class="sf-comparison-value">12,000</td>
          <td class="sf-comparison-value">12,000</td>
          <td class="sf-comparison-value sf-best">18,000</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Travel (X×Y×Z)</td>
          <td class="sf-comparison-value sf-best">762×406×508</td>
          <td class="sf-comparison-value sf-best">762×406×508</td>
          <td class="sf-comparison-value">500×450×400</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Axes</td>
          <td class="sf-comparison-value">3</td>
          <td class="sf-comparison-value">3</td>
          <td class="sf-comparison-value sf-best">5</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Tool Changer</td>
          <td class="sf-comparison-value">24-pos</td>
          <td class="sf-comparison-value">24-pos</td>
          <td class="sf-comparison-value sf-best">60-pos</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Control</td>
          <td class="sf-comparison-value">Haas NGC</td>
          <td class="sf-comparison-value">Haas NGC</td>
          <td class="sf-comparison-value">Siemens</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Cutting Hours</td>
          <td class="sf-comparison-value">4,500</td>
          <td class="sf-comparison-value sf-best">1,800</td>
          <td class="sf-comparison-value">3,200</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Location</td>
          <td class="sf-comparison-value">Detroit, MI</td>
          <td class="sf-comparison-value">Chicago, IL</td>
          <td class="sf-comparison-value">Milwaukee, WI</td>
        </tr>
        <tr>
          <td class="sf-comparison-label">Source</td>
          <td class="sf-comparison-value">Machinio</td>
          <td class="sf-comparison-value">EquipNet</td>
          <td class="sf-comparison-value">Machinio</td>
        </tr>
      </tbody>
    </table>
  </div>

  <div class="sf-comparison-verdict">
    <h3>Verdict</h3>
    <div class="sf-verdict-card sf-verdict-best-deal">
      <span class="sf-verdict-label">Best Deal</span>
      <span class="sf-verdict-value">Haas VF-2 (2019) — $45,000, Score 0.85</span>
    </div>
    <div class="sf-verdict-card">
      <span class="sf-verdict-label">Best Condition</span>
      <span class="sf-verdict-value">Haas VF-2SS (2020) — Score 0.88, 1,800 hours</span>
    </div>
    <div class="sf-verdict-card">
      <span class="sf-verdict-label">Most Capable</span>
      <span class="sf-verdict-value">DMG Mori CMX 50U — 5-axis, 18K RPM, 60-tool ATC</span>
    </div>
  </div>
</div>
```

---

## 17. OpenAPI Schema

The FastAPI application auto-generates the OpenAPI 3.1.0 schema at:

```
GET /api/v1/openapi.json
GET /api/v1/docs        (Swagger UI)
GET /api/v1/redoc       (ReDoc)
```

### 17.1 Complete Endpoint Summary

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| **Listings** | | | |
| GET | `/api/v1/listings` | Search/filter listings | P0 |
| GET | `/api/v1/listings/{id}` | Get single listing | P0 |
| POST | `/api/v1/listings` | Create listing (manual) | P1 |
| PUT | `/api/v1/listings/{id}` | Update listing | P0 |
| DELETE | `/api/v1/listings/{id}` | Soft-delete listing | P1 |
| PUT | `/api/v1/listings/{id}/bookmark` | Toggle bookmark | P0 |
| PUT | `/api/v1/listings/{id}/notes` | Update notes | P0 |
| POST | `/api/v1/listings/bulk` | Bulk operations | P1 |
| **Verticals** | | | |
| GET | `/api/v1/verticals` | List all verticals | P0 |
| GET | `/api/v1/verticals/{id}` | Get vertical details | P0 |
| GET | `/api/v1/verticals/{id}/filters` | Get filter definitions | P0 |
| GET | `/api/v1/verticals/{id}/stats` | Get vertical statistics | P0 |
| **Scraping** | | | |
| POST | `/api/v1/scrape/{vertical_id}` | Trigger scrape | P0 |
| GET | `/api/v1/scrape/jobs` | List active jobs | P0 |
| GET | `/api/v1/scrape/jobs/{job_id}` | Get job status | P0 |
| DELETE | `/api/v1/scrape/jobs/{job_id}` | Cancel job | P0 |
| **Enrichment** | | | |
| POST | `/api/v1/enrich/{vertical_id}` | Trigger enrichment | P0 |
| GET | `/api/v1/enrich/stats` | Enrichment statistics | P0 |
| GET | `/api/v1/enrich/cache` | Cache statistics | P1 |
| DELETE | `/api/v1/enrich/cache` | Clear cache | P1 |
| **Valuation** | | | |
| POST | `/api/v1/valuation/estimate` | Estimate price | P1 |
| GET | `/api/v1/valuation/comparables/{listing_id}` | Get comparables | P0 |
| **Alerts** | | | |
| GET | `/api/v1/alerts` | List alerts | P0 |
| POST | `/api/v1/alerts` | Create alert | P0 |
| PUT | `/api/v1/alerts/{id}` | Update alert | P0 |
| DELETE | `/api/v1/alerts/{id}` | Delete alert | P0 |
| POST | `/api/v1/alerts/{id}/test` | Test alert rule | P0 |
| GET | `/api/v1/alerts/history` | Alert history | P1 |
| **Search** | | | |
| GET | `/api/v1/search/suggest` | Autocomplete | P0 |
| GET | `/api/v1/search/facets` | Faceted counts | P0 |
| GET | `/api/v1/search/price-history` | Price history data | P1 |
| **Compare** | | | |
| POST | `/api/v1/compare` | Compare listings | P0 |
| **System** | | | |
| GET | `/api/v1/health` | Health check | P0 |
| GET | `/api/v1/stats` | System statistics | P0 |
| GET | `/api/v1/connectors` | Connector status | P0 |
| **Auth** | | | |
| POST | `/api/v1/auth/keys` | Generate API key | P2 |
| DELETE | `/api/v1/auth/keys/{key}` | Revoke API key | P2 |
| GET | `/api/v1/auth/keys` | List API keys | P2 |
| **HTMX** | | | |
| GET | `/` | Main page (HTML) | P0 |
| GET | `/search` | Search results fragment | P0 |
| GET | `/filters` | Filter panel fragment | P0 |
| GET | `/listing/{id}` | Listing detail fragment | P0 |
| GET | `/compare` | Comparison view fragment | P0 |
| **Docs** | | | |
| GET | `/api/v1/openapi.json` | OpenAPI schema | Auto |
| GET | `/api/v1/docs` | Swagger UI | Auto |
| GET | `/api/v1/redoc` | ReDoc | Auto |

### 17.2 Endpoint Priority Legend

| Priority | Description |
|----------|-------------|
| P0 | Must-have for MVP |
| P1 | Should-have, can defer 1 sprint |
| P2 | Nice-to-have, can defer longer |

### 17.3 Total Endpoint Count

- **API Endpoints (JSON):** 33
- **HTMX Fragments (HTML):** 5
- **Auto-generated (OpenAPI):** 3
- **Grand Total:** 41 endpoints

---

## Appendix A: CORS Configuration

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:8080", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    expose_headers=["X-RateLimit-Limit", "X-RateLimit-Remaining", "X-RateLimit-Reset"],
)
```

## Appendix B: Request ID Generation

Every response includes a unique `request_id`:

```python
import uuid
request_id = f"req_{uuid.uuid4().hex[:12]}"
```

## Appendix C: Response Wrapper Pattern

All successful responses use a consistent envelope:

```python
class ApiResponse(BaseModel, Generic[T]):
    data: T
    meta: dict = {
        "request_id": "req_abc123def456",
        "timestamp": "2026-07-11T14:30:00Z"
    }
```

## Appendix D: FastAPI Router Registration

```python
# src/main.py
from fastapi import FastAPI
from src.api.routers import listings, verticals, scrape, enrichment, valuation, alerts, search, compare, health, auth

app = FastAPI(
    title="SearchForge API",
    version="1.0.0",
    docs_url="/api/v1/docs",
    redoc_url="/api/v1/redoc",
    openapi_url="/api/v1/openapi.json",
)

app.include_router(listings.router, prefix="/api/v1", tags=["Listings"])
app.include_router(verticals.router, prefix="/api/v1", tags=["Verticals"])
app.include_router(scrape.router, prefix="/api/v1", tags=["Scraping"])
app.include_router(enrichment.router, prefix="/api/v1", tags=["Enrichment"])
app.include_router(valuation.router, prefix="/api/v1", tags=["Valuation"])
app.include_router(alerts.router, prefix="/api/v1", tags=["Alerts"])
app.include_router(search.router, prefix="/api/v1", tags=["Search"])
app.include_router(compare.router, prefix="/api/v1", tags=["Compare"])
app.include_router(health.router, prefix="/api/v1", tags=["System"])
app.include_router(auth.router, prefix="/api/v1", tags=["Auth"])
```

## Appendix E: HTMX Configuration

```html
<script>
  document.addEventListener('htmx:configRequest', (e) => {
    e.detail.headers['X-API-Key'] = localStorage.getItem('sf_api_key') || '';
  });

  document.addEventListener('htmx:responseError', (e) => {
    const error = JSON.parse(e.detail.xhr.responseText);
    showToast(error.error.message, 'error');
  });
</script>
```

## Appendix F: Middleware Stack Order

```python
# 1. Request ID injection
# 2. Request logging (method, path, duration, status)
# 3. Security headers (CSP, X-Frame-Options, X-Content-Type-Options)
# 4. Rate limiting (token bucket per IP)
# 5. CORS
# 6. API key authentication (if enabled)
```

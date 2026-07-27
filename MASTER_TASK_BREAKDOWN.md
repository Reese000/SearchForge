# SearchForge — Master Task Breakdown

> **Every task, every file, every dependency, every test.**
> Generated: 2026-07-11 | ~95 tasks across 14 phases

---

## Dependency Graph Overview

```
Phase 0 (Setup) ──────────────────────────────────────────────────┐
    │                                                              │
Phase 1 (Schema Engine) ──────────────────────────────────────────┤
    │                                                              │
Phase 2 (Database) ──┐                                            │
    │                 │                                            │
Phase 3 (Connectors) ┼── depends on Phase 0, 1, 2                 │
    │                 │                                            │
Phase 4 (AI) ────────┘── depends on Phase 0, 1, 2                 │
    │                                                              │
Phase 5 (Valuation) ── depends on Phase 2, 4                      │
    │                                                              │
Phase 6 (Search) ────── depends on Phase 2                        │
    │                                                              │
Phase 7 (Alerts) ────── depends on Phase 2, 6                     │
    │                                                              │
Phase 8 (API) ────────── depends on Phase 2, 5, 6, 7              │
    │                                                              │
Phase 9 (Frontend) ───── depends on Phase 8                       │
    │                                                              │
Phase 10 (Verticals) ─── depends on Phase 1, 4, 5, 8             │
    │                                                              │
Phase 11 (Migration) ─── depends on Phase 2, 8, 10               │
    │                                                              │
Phase 12 (Testing) ────── depends on ALL previous phases          │
    │                                                              │
Phase 13 (Polish) ─────── depends on Phase 12                     │
```

---

## Phase 0: Project Setup (Day 1)

### Task 0.1 — Create project directory structure

- **Files to create:**
  - `SearchForge/` (root)
  - `SearchForge/src/` (+ `__init__.py`)
  - `SearchForge/src/core/` (+ `__init__.py`)
  - `SearchForge/src/api/` (+ `__init__.py`)
  - `SearchForge/src/api/routers/` (+ `__init__.py`)
  - `SearchForge/src/api/utils/` (+ `__init__.py`)
  - `SearchForge/src/domains/` (+ `__init__.py`, `base.py`)
  - `SearchForge/src/domains/cnc_machinery/` (+ `__init__.py`)
  - `SearchForge/src/domains/endmills/` (+ `__init__.py`)
  - `SearchForge/src/domains/used_cars/` (+ `__init__.py`)
  - `SearchForge/src/migrations/` (+ `__init__.py`)
  - `SearchForge/frontend/`
  - `SearchForge/frontend/css/`
  - `SearchForge/frontend/js/`
  - `SearchForge/frontend/js/components/`
  - `SearchForge/frontend/js/views/`
  - `SearchForge/config/`
  - `SearchForge/scripts/`
  - `SearchForge/tests/` (+ `__init__.py`)
  - `SearchForge/tests/test_core/` (+ `__init__.py`)
  - `SearchForge/tests/test_api/` (+ `__init__.py`)
  - `SearchForge/tests/test_domains/` (+ `__init__.py`)
  - `SearchForge/tests/test_integration/` (+ `__init__.py`)
  - `SearchForge/data/` (empty, .gitkeep)
- **Accomplishes:** Full project skeleton ready for development
- **Dependencies:** None
- **Estimated time:** 15 min
- **Acceptance criteria:** All directories exist with `__init__.py` files, `tree` output matches spec
- **Test:** `ls -R SearchForge/ | grep __init__ | wc -l` → ≥18

---

### Task 0.2 — pyproject.toml with all dependencies

- **Files to create:**
  - `SearchForge/pyproject.toml`
- **Dependencies listed:**
  - `fastapi`, `uvicorn[standard]`, `sqlalchemy[asyncio]`, `aiosqlite`
  - `pydantic`, `pydantic-settings`
  - `curl-cffi`, `playwright`
  - `httpx`
  - `openai` (for OpenRouter)
  - `sentence-transformers` (for embeddings)
  - `pytest`, `pytest-asyncio`, `pytest-cov`
  - `ruff` (linting)
  - `uvicorn` (dev server)
- **Accomplishes:** Single source of truth for all Python dependencies
- **Dependencies:** Task 0.1
- **Estimated time:** 20 min
- **Acceptance criteria:** `uv sync` or `pip install -e ".[dev]"` succeeds with zero errors
- **Test:** `cd SearchForge && python -c "import fastapi; import sqlalchemy; import pydantic; print('OK')"`

---

### Task 0.3 — Makefile with all targets

- **Files to create:**
  - `SearchForge/Makefile`
- **Targets:**
  - `install` — install all dependencies
  - `dev` — start uvicorn dev server with --reload
  - `test` — run pytest with coverage
  - `test-unit` — run only unit tests
  - `test-integration` — run integration tests
  - `lint` — run ruff check + ruff format --check
  - `format` — run ruff format
  - `migrate` — run migration runner
  - `seed` — run seed script
  - `scrape-all` — trigger all domain scrapers
  - `clean` — remove __pycache__, .pytest_cache, *.pyc
- **Accomplishes:** Standardized development workflow
- **Dependencies:** Task 0.1
- **Estimated time:** 15 min
- **Acceptance criteria:** `make test` runs pytest, `make lint` runs ruff
- **Test:** `cd SearchForge && make test 2>&1 | head -5`

---

### Task 0.4 — config/settings.json + .env.example

- **Files to create:**
  - `SearchForge/config/settings.json`
  - `SearchForge/.env.example`
  - `SearchForge/config/search_queries.json`
  - `SearchForge/config/brands.json`
  - `SearchForge/config/exclusions.json`
  - `SearchForge/config/alerts.json`
- **settings.json contents:**
  ```json
  {
    "app_name": "SearchForge",
    "host": "127.0.0.1",
    "port": 8000,
    "database_path": "data/searchforge.db",
    "meilisearch_url": "http://127.0.0.1:7700",
    "meilisearch_api_key": "",
    "ai_provider": "openrouter",
    "ai_model_default": "deepseek/deepseek-chat",
    "ai_model_vision": "google/gemini-flash-1.5",
    "ai_daily_budget_usd": 5.0,
    "scrape_max_concurrent": 3,
    "scrape_default_rpm": 30,
    "embedding_model": "all-MiniLM-L6-v2",
    "embedding_dimensions": 384,
    "log_level": "INFO"
  }
  ```
- **.env.example contents:**
  ```
  OPENROUTER_API_KEY=sk-or-...
  EBAY_APP_ID=
  EBAY_CERT_ID=
  EDMUNDS_API_KEY=
  NHTSA_BASE_URL=https://vpic.nhtsa.dot.gov/api
  ```
- **Accomplishes:** All configuration files ready, secrets externalized
- **Dependencies:** Task 0.1
- **Estimated time:** 20 min
- **Acceptance criteria:** JSON is valid, .env.example has all required keys
- **Test:** `python -c "import json; json.load(open('SearchForge/config/settings.json'))"`

---

### Task 0.5 — .gitignore + pre-commit config

- **Files to create:**
  - `SearchForge/.gitignore`
  - `SearchForge/.pre-commit-config.yaml`
  - `SearchForge/CLAUDE.md` (project context for AI agents)
- **.gitignore contents:**
  ```
  __pycache__/
  *.pyc
  .env
  *.db
  data/
  .pytest_cache/
  .ruff_cache/
  dist/
  *.egg-info/
  node_modules/
  .coverage
  htmlcov/
  ```
- **pre-commit hooks:**
  - ruff lint + format
  - trailing whitespace
  - end-of-file fixer
  - check-json
  - check-yaml
- **Accomplishes:** Clean repo from day one, automated code quality
- **Dependencies:** Task 0.1
- **Estimated time:** 10 min
- **Acceptance criteria:** `pre-commit run --all-files` passes
- **Test:** `cd SearchForge && git init && pre-commit install`

---

## Phase 1: Schema Engine (Days 2-3)

### Task 1.1 — YAML schema loader + validator

- **Files to create:**
  - `SearchForge/src/core/schema_engine/__init__.py`
  - `SearchForge/src/core/schema_engine/loader.py`
  - `SearchForge/src/core/schema_engine/validator.py`
- **Accomplishes:** Load and validate YAML vertical pack files with schema versioning
- **Dependencies:** Task 0.1, 0.2
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Loader reads YAML files from `config/verticals/`
  - Validator checks required fields: `name`, `version`, `fields`, `scrapers`, `search_config`
  - Raises `SchemaValidationError` with line numbers on invalid YAML
  - Supports schema versioning (v1, v2, etc.)
- **Test:** `cd SearchForge && pytest tests/test_core/test_schema_loader.py -v`

---

### Task 1.2 — VerticalSchema Pydantic model

- **Files to create:**
  - `SearchForge/src/core/schema_engine/models.py`
  - `SearchForge/src/core/schema_engine/models_test.py` (inline)
- **Accomplishes:** Pydantic models for the full schema definition: `VerticalSchema`, `ScrapingConfig`, `SearchConfig`, `EnrichmentConfig`, `ValuationConfig`
- **Dependencies:** Task 0.2, 1.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `VerticalSchema` has: name, version, display_name, icon, fields, scrapers, search, enrichment, valuation
  - Field definitions support: string, integer, float, boolean, enum, range, currency
  - Model validates against sample YAML correctly
- **Test:** `cd SearchForge && pytest tests/test_core/test_schema_models.py -v`

---

### Task 1.3 — FieldDefinition types

- **Files to create:**
  - `SearchForge/src/core/schema_engine/field_types.py`
- **Accomplishes:** Complete type system for field definitions: `FieldType` enum, `FieldDefinition`, `EnumField`, `RangeField`, `CurrencyField`, `ConditionMapping`
- **Dependencies:** Task 0.2, 1.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - All field types: `string`, `integer`, `float`, `boolean`, `enum`, `range`, `currency`, `location`, `date`, `json`
  - Each type has validation rules
  - ConditionMapping normalizes condition strings to 0.0-1.0 scores
- **Test:** `cd SearchForge && pytest tests/test_core/test_field_types.py -v`

---

### Task 1.4 — Schema registry + auto-discovery

- **Files to create:**
  - `SearchForge/src/core/schema_engine/registry.py`
- **Accomplishes:** Auto-discover YAML schemas from `config/verticals/`, cache parsed schemas, provide lookup by name
- **Dependencies:** Task 1.1, 1.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `SchemaRegistry.discover()` scans directory and loads all valid schemas
  - `SchemaRegistry.get("cnc_machines")` returns the CNC schema
  - `SchemaRegistry.all()` returns all registered schemas
  - Invalid schemas logged with warnings, not fatal
- **Test:** `cd SearchForge && pytest tests/test_core/test_schema_registry.py -v`

---

### Task 1.5 — AI prompt generator from schema

- **Files to create:**
  - `SearchForge/src/core/schema_engine/prompt_generator.py`
- **Accomplishes:** Auto-generate LLM extraction prompts from field definitions (field name, type, description, examples → structured JSON extraction prompt)
- **Dependencies:** Task 1.2, 1.3
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Generates valid extraction prompts for each vertical
  - Prompts include field descriptions, types, and example values
  - Output is valid JSON schema for structured extraction
- **Test:** `cd SearchForge && pytest tests/test_core/test_prompt_generator.py -v`

---

### Task 1.6 — Filter spec generator from schema

- **Files to create:**
  - `SearchForge/src/core/schema_engine/filter_generator.py`
- **Accomplishes:** Auto-generate filter panel specs (facet definitions) from field definitions
- **Dependencies:** Task 1.2, 1.3
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - Generates faceted filter specs for enum fields
  - Generates range filter specs for numeric fields
  - Generates boolean filter specs for flag fields
  - Excludes non-filterable fields (IDs, timestamps, raw text)
- **Test:** `cd SearchForge && pytest tests/test_core/test_filter_generator.py -v`

---

### Task 1.7 — Search index config generator from schema

- **Files to create:**
  - `SearchForge/src/core/schema_engine/index_generator.py`
- **Accomplishes:** Auto-generate SQLite FTS5 config and Meilisearch index settings from field definitions
- **Dependencies:** Task 1.2, 1.3
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - Generates FTS5 column list from searchable fields
  - Generates Meilisearch filterableAttributes and sortableAttributes
  - Generates Meilisearch searchableAttributes with weights
- **Test:** `cd SearchForge && pytest tests/test_core/test_index_generator.py -v`

---

## Phase 2: Core Database (Days 3-4)

### Task 2.1 — SQLAlchemy async engine setup

- **Files to create:**
  - `SearchForge/src/database.py`
- **Accomplishes:** Async SQLAlchemy engine with WAL mode, session factory, dependency injection for FastAPI
- **Dependencies:** Task 0.2, 0.4
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `get_engine()` returns async engine pointing to SQLite file
  - WAL mode enabled via PRAGMA
  - `get_session()` yields async session
  - `init_db()` creates all tables from metadata
  - Connection pool size = 1 (SQLite)
- **Test:** `cd SearchForge && pytest tests/test_core/test_database.py -v`

---

### Task 2.2 — Listing model (universal fields)

- **Files to create:**
  - `SearchForge/src/models/__init__.py`
  - `SearchForge/src/models/listing.py`
- **Accomplishes:** SQLAlchemy ORM model for the `products` table (universal fields: identity, pricing, location, condition, metadata, timestamps)
- **Dependencies:** Task 2.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - All columns from schema.sql implemented
  - Proper types: `String`, `Float`, `Integer`, `Text`, `Boolean`
  - Indexes defined on: domain, make+model, year, price, condition_score, location, active+sold
  - `source_url` has UNIQUE constraint
- **Test:** `cd SearchForge && pytest tests/test_core/test_models_listing.py -v`

---

### Task 2.3 — Vertical-specific specs models

- **Files to create:**
  - `SearchForge/src/models/cnc_machine.py`
  - `SearchForge/src/models/endmill.py`
  - `SearchForge/src/models/used_car.py`
- **Accomplishes:** SQLAlchemy ORM models for `cnc_machines`, `endmills`, `used_cars` extension tables
- **Dependencies:** Task 2.1, 2.2
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - All columns from schema.sql implemented per table
  - Foreign key to `products.product_id` with CASCADE
  - Indexes on key filter fields (taper, diameter, VIN, etc.)
  - `specs_json` TEXT column for overflow data
- **Test:** `cd SearchForge && pytest tests/test_core/test_models_extensions.py -v`

---

### Task 2.4 — FTS5 virtual table + triggers

- **Files to create:**
  - `SearchForge/src/migrations/001_initial_schema.sql`
  - `SearchForge/src/migrations/002_cnc_extension.sql`
  - `SearchForge/src/migrations/003_endmill_extension.sql`
  - `SearchForge/src/migrations/004_car_extension.sql`
  - `SearchForge/src/migrations/base.py`
- **Accomplishes:** Full migration system: FTS5 virtual table, INSERT/UPDATE/DELETE triggers for sync, migration runner
- **Dependencies:** Task 2.2, 2.3
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - FTS5 `products_fts` table created with porter unicode61 tokenizer
  - INSERT trigger syncs new rows to FTS5
  - UPDATE trigger syncs changes
  - DELETE trigger removes from FTS5
  - Migration runner tracks applied migrations in `schema_migrations` table
  - `run_migrations()` applies all pending in order
- **Test:** `cd SearchForge && pytest tests/test_core/test_migrations.py -v`

---

### Task 2.5 — AiCache model

- **Files to create:**
  - `SearchForge/src/models/ai_cache.py`
- **Accomplishes:** SQLAlchemy ORM model for `ai_cache` table (SHA-256 hash-keyed LLM response cache)
- **Dependencies:** Task 2.1
- **Estimated time:** 45 min
- **Acceptance criteria:**
  - Columns: cache_id, input_hash (UNIQUE), input_text, output_json, cache_type, model_used, tokens_used, created_at
  - Index on input_hash for fast lookup
- **Test:** `cd SearchForge && pytest tests/test_core/test_models_ai_cache.py -v`

---

### Task 2.6 — Alert model

- **Files to create:**
  - `SearchForge/src/models/alert.py`
- **Accomplishes:** SQLAlchemy ORM model for `saved_searches` and `alerts` tables
- **Dependencies:** Task 2.1
- **Estimated time:** 45 min
- **Acceptance criteria:**
  - `saved_searches` table: search_id, name, filters_json, domain, alert_enabled, alert_frequency, last_alert_at
  - `alerts` table: alert_id, search_id (FK), product_id (FK), message, sent_at
  - Indexes on alert_enabled, last_alert_at
- **Test:** `cd SearchForge && pytest tests/test_core/test_models_alert.py -v`

---

### Task 2.7 — PriceHistory model

- **Files to create:**
  - `SearchForge/src/models/price_history.py`
- **Accomplishes:** SQLAlchemy ORM model for `price_history` table
- **Dependencies:** Task 2.1
- **Estimated time:** 30 min
- **Acceptance criteria:**
  - Columns: product_id (FK), recorded_at, price_usd, source
  - Composite primary key (product_id, recorded_at)
  - Index on (product_id, recorded_at DESC) for recent price lookup
- **Test:** `cd SearchForge && pytest tests/test_core/test_models_price_history.py -v`

---

### Task 2.8 — ScrapeJob model

- **Files to create:**
  - `SearchForge/src/models/scrape_job.py`
- **Accomplishes:** SQLAlchemy ORM model for `scrape_jobs` table
- **Dependencies:** Task 2.1
- **Estimated time:** 45 min
- **Acceptance criteria:**
  - Columns: job_id, domain, status (pending/running/completed/failed), query, params_json, items_found, items_new, items_updated, error_message, started_at, completed_at, created_at
  - Index on status, created_at
- **Test:** `cd SearchForge && pytest tests/test_core/test_models_scrape_job.py -v`

---

### Task 2.9 — Repository (CRUD + search)

- **Files to create:**
  - `SearchForge/src/core/repository.py`
- **Accomplishes:** Generic repository pattern with merge-safe upsert, soft delete, dedup via source_url hash, bulk operations
- **Dependencies:** Task 2.2, 2.3, 2.4
- **Estimated time:** 2.5 hours
- **Acceptance criteria:**
  - `upsert_by_source_url()` — insert or update based on source_url uniqueness
  - `upsert_batch()` — bulk upsert with transaction
  - `get_by_id()` — fetch with extension table JOIN
  - `list_by_domain()` — paginated listing with filters
  - `soft_delete()` — set is_active=0
  - `search_text()` — FTS5 search returning product IDs + scores
  - `get_price_history()` — return price history for product
  - All operations async
- **Test:** `cd SearchForge && pytest tests/test_core/test_repository.py -v`

---

## Phase 3: Connectors (Days 4-6)

### Task 3.1 — Connector protocol + registry

- **Files to create:**
  - `SearchForge/src/core/connector.py`
- **Accomplishes:** Abstract connector protocol, connector registry, auto-discovery from config
- **Dependencies:** Task 0.2, 1.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `Connector` Protocol: `async def fetch(query, params) -> list[RawListing]`
  - `ConnectorConfig` dataclass: name, base_url, rpm, requires_browser, headers
  - `ConnectorRegistry` discovers connectors from config
- **Test:** `cd SearchForge && pytest tests/test_core/test_connector.py -v`

---

### Task 3.2 — Rate limiter (token bucket)

- **Files to create:**
  - `SearchForge/src/core/rate_limiter.py`
  - `SearchForge/tests/test_core/test_rate_limiter.py`
- **Accomplishes:** Token bucket rate limiter with async support, per-domain limits, jitter
- **Dependencies:** Task 0.2
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `acquire()` blocks until token available
  - Configurable max_tokens and refill_rate
  - ±20% random jitter on wait times
  - `available()` returns current token count
- **Test:** `cd SearchForge && pytest tests/test_core/test_rate_limiter.py -v`

---

### Task 3.3 — Circuit breaker

- **Files to create:**
  - `SearchForge/src/core/circuit_breaker.py`
  - `SearchForge/tests/test_core/test_circuit_breaker.py`
- **Accomplishes:** Circuit breaker pattern: closed → open → half-open, configurable thresholds
- **Dependencies:** Task 0.2
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - States: CLOSED, OPEN, HALF_OPEN
  - After N consecutive failures → OPEN
  - After timeout → HALF_OPEN
  - After success in HALF_OPEN → CLOSED
  - `record_success()` and `record_failure()` methods
- **Test:** `cd SearchForge && pytest tests/test_core/test_circuit_breaker.py -v`

---

### Task 3.4 — eBay API connector

- **Files to create:**
  - `SearchForge/src/connectors/ebay_api.py`
  - `SearchForge/tests/test_connectors/test_ebay_api.py`
- **Accomplishes:** eBay Browse API connector with OAuth 2.0 auto-refresh, rate limiting, typed errors
- **Dependencies:** Task 3.1, 3.2, 3.3
- **Estimated time:** 3 hours
- **Acceptance criteria:**
  - OAuth 2.0 client_credentials flow with auto-refresh
  - Browse API search endpoint integration
  - Rate limiting (5000 calls/day sliding window)
  - Returns `list[RawListing]` in normalized format
  - Handles pagination (50 items/page)
  - Circuit breaker on 5xx errors
- **Test:** `cd SearchForge && pytest tests/test_connectors/test_ebay_api.py -v`

---

### Task 3.5 — eBay Playwright fallback

- **Files to create:**
  - `SearchForge/src/connectors/ebay_playwright.py`
  - `SearchForge/tests/test_connectors/test_ebay_playwright.py`
- **Accomplishes:** eBay HTML scraping via Playwright for when API quota is exhausted
- **Dependencies:** Task 3.4
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Launches headless Chromium with stealth scripts
  - Parses search results page
  - Handles cookie consent popups
  - Returns `list[RawListing]`
  - Rate limited to 10 RPM
- **Test:** `cd SearchForge && pytest tests/test_connectors/test_ebay_playwright.py -v`

---

### Task 3.6 — Machinio connector

- **Files to create:**
  - `SearchForge/src/connectors/machinio.py`
  - `SearchForge/tests/test_connectors/test_machinio.py`
- **Accomplishes:** Machinio CNC marketplace connector (HTTP-first, Playwright fallback)
- **Dependencies:** Task 3.1, 3.2
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Searches by category (CNC mills, lathes, etc.)
  - Parses listing cards (title, price, location, image)
  - Handles pagination
  - Rate limited
- **Test:** `cd SearchForge && pytest tests/test_connectors/test_machinio.py -v`

---

### Task 3.7 — CarGurus connector

- **Files to create:**
  - `SearchForge/src/connectors/cargurus.py`
  - `SearchForge/tests/test_connectors/test_cargurus.py`
- **Accomplishes:** CarGurus used car listings connector (Playwright, heavy anti-bot)
- **Dependencies:** Task 3.1, 3.2, 3.3
- **Estimated time:** 3 hours
- **Acceptance criteria:**
  - Searches by make, model, year, zip code, radius
  - Parses: title, price, mileage, dealer info, photos
  - Handles anti-bot detection (retry with browser)
  - Rate limited (conservative: 5 RPM)
- **Test:** `cd SearchForge && pytest tests/test_connectors/test_cargurus.py -v`

---

### Task 3.8 — Autotrader connector

- **Files to create:**
  - `SearchForge/src/connectors/autotrader.py`
  - `SearchForge/tests/test_connectors/test_autotrader.py`
- **Accomplishes:** Autotrader used car listings connector
- **Dependencies:** Task 3.1, 3.2
- **Estimated time:** 2.5 hours
- **Acceptance criteria:**
  - Searches by make, model, year, zip code, distance
  - Parses: title, price, mileage, VIN (when available), dealer
  - Handles pagination
  - Rate limited
- **Test:** `cd SearchForge && pytest tests/test_connectors/test_autotrader.py -v`

---

### Task 3.9 — Raw snapshot storage

- **Files to create:**
  - `SearchForge/src/core/raw_storage.py`
  - `SearchForge/tests/test_core/test_raw_storage.py`
- **Accomplishes:** Store raw scraped HTML/API responses for debugging and re-parsing
- **Dependencies:** Task 2.1
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - Stores raw content with metadata (source, URL, timestamp, content_type)
  - File-based storage in `data/raw/` directory
  - `store()` saves with content-hash filename
  - `retrieve()` loads by hash
  - `list_recent()` returns last N snapshots
- **Test:** `cd SearchForge && pytest tests/test_core/test_raw_storage.py -v`

---

## Phase 4: AI Enrichment (Days 6-8)

### Task 4.1 — Regex extractor framework

- **Files to create:**
  - `SearchForge/src/core/extraction/__init__.py`
  - `SearchForge/src/core/extraction/regex_engine.py`
- **Accomplishes:** Generic regex extraction engine: define patterns → apply to text → return structured fields
- **Dependencies:** Task 0.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `RegexExtractor` accepts list of `ExtractionPattern(name, pattern, type)`
  - `extract(text) -> dict[str, Any]` returns matched fields
  - Handles unit conversion (e.g., "10,000 RPM" → 10000)
  - Returns confidence score per field
- **Test:** `cd SearchForge && pytest tests/test_core/test_regex_engine.py -v`

---

### Task 4.2 — CNC regex patterns + tests

- **Files to create:**
  - `SearchForge/src/domains/cnc_machinery/regex_patterns.py`
  - `SearchForge/tests/test_domains/test_cnc_regex.py`
- **Accomplishes:** 20+ regex patterns for CNC specs: spindle RPM, taper, travel XYZ, control system, hours, etc.
- **Dependencies:** Task 4.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Patterns for: spindle_rpm, spindle_taper, travel_xyz, control_system, cutting_hours, axis_count, tool_changer_capacity
  - Each pattern has ≥5 test cases
  - Extraction accuracy ≥90% on test data
- **Test:** `cd SearchForge && pytest tests/test_domains/test_cnc_regex.py -v`

---

### Task 4.3 — Endmill regex patterns + tests

- **Files to create:**
  - `SearchForge/src/domains/endmills/regex_patterns.py`
  - `SearchForge/tests/test_domains/test_endmill_regex.py`
- **Accomplishes:** 15+ regex patterns for endmill specs: diameter, flute count, coating, length, end type, etc.
- **Dependencies:** Task 4.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Patterns for: diameter, flute_count, coating, end_type, length_of_cut, substrate, helix_angle
  - Handles both metric and imperial ("1/2 inch" and "12.7mm")
  - Each pattern has ≥5 test cases
- **Test:** `cd SearchForge && pytest tests/test_domains/test_endmill_regex.py -v`

---

### Task 4.4 — Car regex patterns + tests

- **Files to create:**
  - `SearchForge/src/domains/used_cars/regex_patterns.py`
  - `SearchForge/tests/test_domains/test_car_regex.py`
- **Accomplishes:** 15+ regex patterns for car specs: mileage, VIN, engine size, transmission, year, etc.
- **Dependencies:** Task 4.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Patterns for: mileage, vin (17-char validation), year, engine, transmission, drivetrain, body_style
  - VIN validation includes check digit
  - Each pattern has ≥5 test cases
- **Test:** `cd SearchForge && pytest tests/test_domains/test_car_regex.py -v`

---

### Task 4.5 — LLM extractor (OpenRouter)

- **Files to create:**
  - `SearchForge/src/core/extraction/llm_extractor.py`
  - `SearchForge/tests/test_core/test_llm_extractor.py`
- **Accomplishes:** LLM-based structured extraction using OpenRouter API with JSON mode
- **Dependencies:** Task 0.2, 1.5
- **Estimated time:** 2.5 hours
- **Acceptance criteria:**
  - `extract(text, schema) -> dict` calls OpenRouter with structured output
  - Uses schema-generated prompts (from Task 1.5)
  - Handles: timeout, rate limit, invalid JSON, empty response
  - Returns typed `ExtractionResult` with confidence scores
  - Logs token usage to cost tracker
- **Test:** `cd SearchForge && pytest tests/test_core/test_llm_extractor.py -v`

---

### Task 4.6 — Vision analyzer (Gemini)

- **Files to create:**
  - `SearchForge/src/core/extraction/vision_analyzer.py`
  - `SearchForge/tests/test_core/test_vision_analyzer.py`
- **Accomplishes:** Gemini Vision API for extracting specs from listing images (nameplates, data plates)
- **Dependencies:** Task 0.2, 4.5
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `analyze(image_url) -> dict` sends image to Gemini Flash
  - Extracts text from nameplate images
  - Parses extracted text into structured fields
  - Handles: no image, invalid image, text not found
  - Cost tracked separately
- **Test:** `cd SearchForge && pytest tests/test_core/test_vision_analyzer.py -v`

---

### Task 4.7 — Tiered pipeline orchestrator

- **Files to create:**
  - `SearchForge/src/core/extraction/pipeline.py`
  - `SearchForge/tests/test_core/test_extraction_pipeline.py`
- **Accomplishes:** Orchestrates the 3-tier extraction cascade: regex → LLM → vision
- **Dependencies:** Task 4.1, 4.5, 4.6, 2.5
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Tier 1: Apply regex patterns (free, instant)
  - Tier 2: If gaps remain → LLM extraction (paid, cached)
  - Tier 3: If image available and text fields missing → Vision analysis (paid, cached)
  - Stops early when all required fields filled
  - Returns `EnrichmentResult` with tier_used and confidence
- **Test:** `cd SearchForge && pytest tests/test_core/test_extraction_pipeline.py -v`

---

### Task 4.8 — Hash-keyed cache

- **Files to create:**
  - `SearchForge/src/core/extraction/cache.py`
  - `SearchForge/tests/test_core/test_extraction_cache.py`
- **Accomplishes:** Content-addressable cache for AI results, keyed by SHA-256 of input text
- **Dependencies:** Task 2.5
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `get(input_text) -> dict | None` fast lookup
  - `store(input_text, output, metadata)` persists to DB
  - Cache hit rate tracked
  - TTL-based expiration (configurable, default 30 days)
  - `stats()` returns hit/miss counts and cost savings
- **Test:** `cd SearchForge && pytest tests/test_core/test_extraction_cache.py -v`

---

### Task 4.9 — Cost tracker

- **Files to create:**
  - `SearchForge/src/core/extraction/cost_tracker.py`
  - `SearchForge/tests/test_core/test_cost_tracker.py`
- **Accomplishes:** Singleton cost tracker for all AI API calls across the system
- **Dependencies:** Task 0.2
- **Estimated time:** 45 min
- **Acceptance criteria:**
  - `add(model, input_tokens, output_tokens, cost_usd)`
  - `daily_total()` returns today's total spend
  - `by_model()` returns per-model breakdown
  - `check_budget()` returns True if under daily budget
  - Persists to DB (survives restarts)
- **Test:** `cd SearchForge && pytest tests/test_core/test_cost_tracker.py -v`

---

### Task 4.10 — Field mapping + merge logic

- **Files to create:**
  - `SearchForge/src/core/extraction/merger.py`
  - `SearchForge/tests/test_core/test_merger.py`
- **Accomplishes:** Merge extracted fields from multiple sources (regex, LLM, vision) into a single NormalizedListing
- **Dependencies:** Task 1.2, 4.7
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Priority: LLM > regex > vision > raw listing
  - Merges into NormalizedListing.domain_data
  - Handles conflicts (same field, different values) with confidence-based resolution
  - Preserves source attribution (which tier filled each field)
- **Test:** `cd SearchForge && pytest tests/test_core/test_merger.py -v`

---

## Phase 5: Valuation (Days 8-9)

### Task 5.1 — Comparable-sales engine

- **Files to create:**
  - `SearchForge/src/core/valuation/__init__.py`
  - `SearchForge/src/core/valuation/engine.py`
- **Accomplishes:** Find comparable listings for a given product and compute market value estimate
- **Dependencies:** Task 2.2, 2.3, 2.9
- **Estimated time:** 2.5 hours
- **Acceptance criteria:**
  - `find_comparables(product_id, domain, n=20) -> list[ComparableSale]`
  - Matching by: make+model, year range, domain-specific specs
  - Returns: product_id, price, similarity_score, recency_weight
  - SQL-based query for performance
- **Test:** `cd SearchForge && pytest tests/test_core/test_valuation_engine.py -v`

---

### Task 5.2 — Similarity scoring

- **Files to create:**
  - `SearchForge/src/core/valuation/similarity.py`
- **Accomplishes:** Compute similarity scores between listings based on shared attributes
- **Dependencies:** Task 1.2, 5.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Weighted Jaccard similarity for categorical fields
  - Normalized distance for numeric fields (min-max within domain)
  - Domain-specific weights (e.g., taper heavily weighted for CNC, engine for cars)
  - Returns 0.0-1.0 score
- **Test:** `cd SearchForge && pytest tests/test_core/test_valuation_similarity.py -v`

---

### Task 5.3 — Outlier filtering

- **Files to create:**
  - `SearchForge/src/core/valuation/outliers.py`
- **Accomplishes:** Filter outlier prices using IQR method and domain-specific rules
- **Dependencies:** Task 5.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - IQR method: remove prices < Q1-1.5*IQR or > Q3+1.5*IQR
  - Minimum 5 comparable sales required
  - Domain-specific rules (e.g., endmills under $1 = likely wrong, CNC under $500 = parts only)
  - Returns filtered list + outlier count
- **Test:** `cd SearchForge && pytest tests/test_core/test_valuation_outliers.py -v`

---

### Task 5.4 — Confidence scoring

- **Files to create:**
  - `SearchForge/src/core/valuation/confidence.py`
- **Accomplishes:** Compute confidence level for valuation based on: number of comparables, recency, data completeness
- **Dependencies:** Task 5.1, 5.3
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - Factors: comparable_count (0-1), recency (0-1), data_completeness (0-1), similarity_avg (0-1)
  - Weighted combination → 0.0-1.0 confidence
  - Labels: "high" (≥0.8), "medium" (≥0.5), "low" (<0.5)
- **Test:** `cd SearchForge && pytest tests/test_core/test_valuation_confidence.py -v`

---

### Task 5.5 — Price breakdown format

- **Files to create:**
  - `SearchForge/src/core/valuation/format.py`
  - `SearchForge/src/schemas/valuation.py`
- **Accomplishes:** Format valuation results into a structured response with breakdown
- **Dependencies:** Task 5.1, 5.4
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `ValuationResult` schema: estimated_value, confidence, comparable_count, price_range_low, price_range_high, breakdown (list of factors), comparables (top 5 with prices)
  - Domain-specific formatting (CNC shows hours, cars show mileage)
- **Test:** `cd SearchForge && pytest tests/test_core/test_valuation_format.py -v`

---

## Phase 6: Search (Days 9-10)

### Task 6.1 — FTS5 index setup

- **Files to create:**
  - `SearchForge/src/core/search/fts5.py`
  - `SearchForge/tests/test_core/test_fts5.py`
- **Accomplishes:** FTS5 index management: create, rebuild, query with ranking
- **Dependencies:** Task 2.4
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Creates FTS5 virtual table with porter unicode61 tokenizer
  - `rebuild()` re-indexes all products
  - `search(query, filters, sort, page, per_page)` returns paginated results
  - BM25 ranking by default
  - Highlights matching terms in results
- **Test:** `cd SearchForge && pytest tests/test_core/test_fts5.py -v`

---

### Task 6.2 — Meilisearch integration

- **Files to create:**
  - `SearchForge/src/core/search/meilisearch_client.py`
  - `SearchForge/tests/test_core/test_meilisearch.py`
- **Accomplishes:** Optional Meilisearch integration for enhanced search (facets, typo tolerance, geosearch)
- **Dependencies:** Task 0.2, 6.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `MeilisearchClient` with async httpx
  - `index_product(product)` adds/updates document
  - `search(query, filters, sort, facets)` returns results with facet counts
  - `delete_product(product_id)` removes document
  - Graceful fallback to FTS5 when Meilisearch unavailable
  - Auto-sync: products added to FTS5 also indexed in Meilisearch
- **Test:** `cd SearchForge && pytest tests/test_core/test_meilisearch.py -v`

---

### Task 6.3 — Query builder (filters → SQL)

- **Files to create:**
  - `SearchForge/src/core/search/query_builder.py`
  - `SearchForge/tests/test_core/test_query_builder.py`
- **Accomplishes:** Translate filter parameters into SQL WHERE clauses for domain extension tables
- **Dependencies:** Task 2.2, 2.3, 1.6
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `build_query(filters, domain) -> (where_clause, params)`
  - Supports: equals, contains, range (min/max), in-list, boolean
  - Joins products with domain extension table when domain filters used
  - SQL injection safe (parameterized queries)
  - Supports cross-domain filters (price, condition, location)
- **Test:** `cd SearchForge && pytest tests/test_core/test_query_builder.py -v`

---

### Task 6.4 — Faceted search

- **Files to create:**
  - `SearchForge/src/core/search/facets.py`
  - `SearchForge/tests/test_core/test_facets.py`
- **Accomplishes:** Compute faceted filter counts for current search results
- **Dependencies:** Task 2.4, 6.3, 1.6
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `get_facets(query, domain, active_filters) -> dict[str, list[FacetOption]]`
  - Each facet shows available values with counts
  - Active filters don't reduce their own counts (drill-down behavior)
  - Domain-specific facets loaded from schema
  - Cross-domain facets: price_range, condition, source, date
- **Test:** `cd SearchForge && pytest tests/test_core/test_facets.py -v`

---

### Task 6.5 — Autocomplete/suggest

- **Files to create:**
  - `SearchForge/src/core/search/suggest.py`
  - `SearchForge/tests/test_core/test_suggest.py`
- **Accomplishes:** Fast autocomplete suggestions combining FTS5 prefix search + brand/model lookup
- **Dependencies:** Task 2.4
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `suggest(prefix, domain) -> list[Suggestion]`
  - Suggestions include: text, type (brand/model/category), domain
  - Response time <50ms
  - Fuzzy matching (1 edit distance)
  - Returns top 8 suggestions
- **Test:** `cd SearchForge && pytest tests/test_core/test_suggest.py -v`

---

### Task 6.6 — Price histogram

- **Files to create:**
  - `SearchForge/src/core/search/histogram.py`
  - `SearchForge/tests/test_core/test_histogram.py`
- **Accomplishes:** Generate price distribution histograms for search results
- **Dependencies:** Task 2.2
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `get_histogram(query, domain, bins=20) -> list[HistogramBin]`
  - Auto-calculates bin width from price range
  - Returns: min, max, count per bin
  - Supports domain-specific price ranges (e.g., endmills $0-$500, CNC $5K-$500K)
- **Test:** `cd SearchForge && pytest tests/test_core/test_histogram.py -v`

---

## Phase 7: Alerts (Days 10-11)

### Task 7.1 — Alert rule engine

- **Files to create:**
  - `SearchForge/src/core/alerts/__init__.py`
  - `SearchForge/src/core/alerts/engine.py`
  - `SearchForge/src/core/alerts/models.py`
- **Accomplishes:** Rule-based alert system with configurable conditions
- **Dependencies:** Task 2.6, 6.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `AlertRule` model: name, conditions (JSON), actions (JSON), enabled
  - Conditions: price_below, price_above, make_equals, model_contains, domain_equals, new_listing, price_drop
  - `evaluate(product) -> list[AlertMatch]` checks all rules
  - Composable conditions (AND logic within a rule)
- **Test:** `cd SearchForge && pytest tests/test_core/test_alert_engine.py -v`

---

### Task 7.2 — Rule evaluation

- **Files to create:**
  - `SearchForge/src/core/alerts/evaluator.py`
- **Accomplishes:** Evaluate alert conditions against product fields with type-safe comparison
- **Dependencies:** Task 7.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Type-aware comparison (string equals, numeric range, date before/after)
  - Supports nested field access (e.g., `cnc_machines.spindle_rpm_max > 10000`)
  - Returns match details (which conditions matched, values)
  - Performance: evaluate 1000 rules against 1 product in <100ms
- **Test:** `cd SearchForge && pytest tests/test_core/test_alert_evaluator.py -v`

---

### Task 7.3 — Notification dispatch (webhook)

- **Files to create:**
  - `SearchForge/src/core/alerts/dispatcher.py`
- **Accomplishes:** Send alert notifications via webhook (Slack, Discord, generic HTTP)
- **Dependencies:** Task 7.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `dispatch(alert_match, action)` sends HTTP POST to configured URL
  - Slack-formatted messages (blocks)
  - Discord-formatted messages (embeds)
  - Generic JSON payload for custom endpoints
  - Retry on failure (3 attempts with exponential backoff)
- **Test:** `cd SearchForge && pytest tests/test_core/test_alert_dispatcher.py -v`

---

### Task 7.4 — Deduplication

- **Files to create:**
  - `SearchForge/src/core/alerts/dedup.py`
- **Accomplishes:** Prevent duplicate alert notifications for the same product+rule combination
- **Dependencies:** Task 2.6
- **Estimated time:** 45 min
- **Acceptance criteria:**
  - `is_duplicate(rule_id, product_id) -> bool`
  - `mark_sent(rule_id, product_id)` records notification
  - Dedup window: configurable (default 24 hours)
  - Uses DB table for persistence
- **Test:** `cd SearchForge && pytest tests/test_core/test_alert_dedup.py -v`

---

### Task 7.5 — Alert history

- **Files to create:**
  - `SearchForge/src/core/alerts/history.py`
- **Accomplishes:** Track alert notification history with status and details
- **Dependencies:** Task 2.6, 7.3
- **Estimated time:** 45 min
- **Acceptance criteria:**
  - `record(match, action, status, details)`
  - `get_history(rule_id, limit) -> list[AlertHistoryEntry]`
  - `get_stats() -> AlertStats` (total sent, success rate, by rule)
- **Test:** `cd SearchForge && pytest tests/test_core/test_alert_history.py -v`

---

## Phase 8: API (Days 11-13)

### Task 8.1 — FastAPI app factory

- **Files to create:**
  - `SearchForge/src/main.py`
  - `SearchForge/src/api/__init__.py`
  - `SearchForge/src/api/dependencies.py`
  - `SearchForge/src/api/middleware.py`
- **Accomplishes:** FastAPI application with lifespan, middleware stack, CORS, static file serving
- **Dependencies:** Task 0.2, 0.4, 2.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `create_app()` factory function
  - Lifespan: init DB, run migrations, load plugins on startup
  - Middleware: request logging, security headers, CORS
  - Static files: serve `frontend/` at `/`
  - Health check at `/api/health`
  - OpenAPI docs at `/docs`
- **Test:** `cd SearchForge && pytest tests/test_api/test_app_factory.py -v`

---

### Task 8.2 — Listing endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/products.py`
  - `SearchForge/src/api/utils/pagination.py`
  - `SearchForge/src/api/utils/errors.py`
  - `SearchForge/src/api/utils/responses.py`
  - `SearchForge/tests/test_api/test_products.py`
- **Accomplishes:** CRUD endpoints for products: list, get, create, update, delete
- **Dependencies:** Task 8.1, 2.9
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `GET /api/products` — paginated list with filters
  - `GET /api/products/{id}` — single product with extension data
  - `POST /api/products` — create product
  - `PUT /api/products/{id}` — update product
  - `DELETE /api/products/{id}` — soft delete
  - Standardized error responses (404, 422, 500)
  - Pagination: page, per_page, total, has_next
- **Test:** `cd SearchForge && pytest tests/test_api/test_products.py -v`

---

### Task 8.3 — Vertical endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/domains.py`
  - `SearchForge/tests/test_api/test_domains.py`
- **Accomplishes:** Domain registry endpoints: list domains, get domain config, get domain facets
- **Dependencies:** Task 8.1, 1.4
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `GET /api/domains` — list all registered domains with config
  - `GET /api/domains/{name}` — single domain config
  - `GET /api/domains/{name}/facets` — domain-specific facets
  - `GET /api/domains/{name}/filters` — domain filter definitions
- **Test:** `cd SearchForge && pytest tests/test_api/test_domains.py -v`

---

### Task 8.4 — Scrape endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/scrape.py`
  - `SearchForge/tests/test_api/test_scrape.py`
- **Accomplishes:** Scrape job management: trigger, status, list jobs
- **Dependencies:** Task 8.1, 2.8
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `POST /api/scrape/trigger` — start scrape job (domain, query, params)
  - `GET /api/scrape/jobs` — list jobs with status
  - `GET /api/scrape/jobs/{id}` — job status + progress
  - `DELETE /api/scrape/jobs/{id}` — cancel job
  - Jobs run in background via task queue
- **Test:** `cd SearchForge && pytest tests/test_api/test_scrape.py -v`

---

### Task 8.5 — Enrichment endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/enrichment.py`
  - `SearchForge/tests/test_api/test_enrichment.py`
- **Accomplishes:** AI enrichment endpoints: batch enrich, cost stats, cache stats
- **Dependencies:** Task 8.1, 4.7, 4.9
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `POST /api/enrich/batch` — enrich N products (background)
  - `GET /api/enrich/stats` — cost breakdown, cache hit rate
  - `POST /api/enrich/force/{id}` — force re-enrichment (bypass cache)
- **Test:** `cd SearchForge && pytest tests/test_api/test_enrichment.py -v`

---

### Task 8.6 — Valuation endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/valuation.py`
  - `SearchForge/tests/test_api/test_valuation.py`
- **Accomplishes:** Valuation endpoints: estimate price, comparable sales
- **Dependencies:** Task 8.1, 5.5
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `GET /api/valuation/{id}` — valuation for single product
  - `GET /api/valuation/{id}/comparables` — list comparable sales
  - `POST /api/valuation/batch` — batch valuation
  - Returns: estimated_value, confidence, range, breakdown, top comparables
- **Test:** `cd SearchForge && pytest tests/test_api/test_valuation.py -v`

---

### Task 8.7 — Alert endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/alerts.py`
  - `SearchForge/tests/test_api/test_alerts.py`
- **Accomplishes:** Alert rule CRUD and history endpoints
- **Dependencies:** Task 8.1, 7.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `GET /api/alerts/rules` — list all rules
  - `POST /api/alerts/rules` — create rule
  - `PUT /api/alerts/rules/{id}` — update rule
  - `DELETE /api/alerts/rules/{id}` — delete rule
  - `GET /api/alerts/history` — notification history
  - `POST /api/alerts/test` — test alert with sample product
- **Test:** `cd SearchForge && pytest tests/test_api/test_alerts.py -v`

---

### Task 8.8 — Search endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/search.py`
  - `SearchForge/tests/test_api/test_search.py`
- **Accomplishes:** Search endpoints: full-text, faceted, autocomplete, histogram
- **Dependencies:** Task 8.1, 6.1, 6.3, 6.4, 6.5, 6.6
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `GET /api/search` — full search with filters, sort, pagination
  - `GET /api/search/suggest` — autocomplete
  - `GET /api/search/facets` — faceted counts
  - `GET /api/search/histogram` — price distribution
  - Cross-domain search (no domain filter = search all)
  - Domain-specific search (filtered to one domain)
- **Test:** `cd SearchForge && pytest tests/test_api/test_search.py -v`

---

### Task 8.9 — System endpoints

- **Files to create:**
  - `SearchForge/src/api/routers/health.py`
  - `SearchForge/src/api/routers/statistics.py`
  - `SearchForge/src/api/routers/compare.py`
  - `SearchForge/tests/test_api/test_health.py`
  - `SearchForge/tests/test_api/test_statistics.py`
- **Accomplishes:** System health, analytics, and cross-domain comparison endpoints
- **Dependencies:** Task 8.1, 2.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - `GET /api/health` — DB connectivity, Meilisearch connectivity, uptime
  - `GET /api/stats/overview` — counts by domain, avg prices, recent activity
  - `GET /api/stats/domain/{name}` — domain-specific analytics
  - `GET /api/stats/trends` — price trends over time
  - `POST /api/compare` — compare multiple products side-by-side
- **Test:** `cd SearchForge && pytest tests/test_api/test_health.py -v`

---

### Task 8.10 — HTMX endpoints (if applicable)

- **Files to create:**
  - `SearchForge/src/api/routers/htmx.py`
- **Accomplishes:** HTMX-specific endpoints returning HTML fragments for partial page updates
- **Dependencies:** Task 8.1
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - `GET /htmx/search-results` — returns HTML card grid
  - `GET /htmx/filter-panel` — returns HTML filter panel
  - `GET /htmx/product-detail/{id}` — returns HTML detail drawer
  - `POST /htmx/scrape/trigger` — returns job status HTML
  - Content-Type: text/html for all responses
- **Test:** `cd SearchForge && pytest tests/test_api/test_htmx.py -v`

---

## Phase 9: Frontend (Days 13-16)

### Task 9.1 — Base layout template

- **Files to create:**
  - `SearchForge/frontend/index.html`
  - `SearchForge/frontend/css/tokens.css`
  - `SearchForge/frontend/css/tokens-searchforge.css`
  - `SearchForge/frontend/css/app.css`
  - `SearchForge/frontend/js/app.js`
  - `SearchForge/frontend/js/api.js`
  - `SearchForge/frontend/js/store.js`
- **Accomplishes:** App shell with titlebar, sidebar, main content area, status bar; design system tokens; SPA router; API client; state management
- **Dependencies:** Task 0.4
- **Estimated time:** 4 hours
- **Acceptance criteria:**
  - Dark theme with all 13 semantic colors
  - Windows-style titlebar with min/max/close
  - Sidebar with navigation
  - Hash-based SPA router (#search, #dashboard, #compare, #alerts)
  - API client with error handling
  - Client-side state store (current domain, filters, results)
  - Keyboard shortcuts: Ctrl+K (command palette), / (search focus)
- **Test:** Open in browser, verify layout renders, check console for JS errors

---

### Task 9.2 — SearchBar component

- **Files to create:**
  - `SearchForge/frontend/js/components/SearchBar.js`
- **Accomplishes:** Universal search bar with autocomplete dropdown
- **Dependencies:** Task 9.1, 8.8
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Text input with debounced search (300ms)
  - Autocomplete dropdown with suggestions
  - Domain filter chips (search within domain)
  - Keyboard navigation (↑↓ to select, Enter to search, Esc to close)
  - Clear button
  - Focus shortcut (/)
- **Test:** Type query, verify autocomplete appears, select suggestion triggers search

---

### Task 9.3 — FilterPanel component

- **Files to create:**
  - `SearchForge/frontend/js/components/FilterPanel.js`
- **Accomplishes:** Faceted filter panel that adapts to current domain
- **Dependencies:** Task 9.1, 8.3, 6.4
- **Estimated time:** 3 hours
- **Acceptance criteria:**
  - Loads facets from API for current domain
  - Categorical: checkbox groups with counts
  - Range: dual-handle slider with min/max inputs
  - Boolean: toggle switches
  - Active filters shown as removable chips
  - "Clear all" button
  - Filters update results in real-time (debounced)
  - Collapsible filter groups
- **Test:** Select filters, verify URL params update, verify results filter

---

### Task 9.4 — ListingCard component

- **Files to create:**
  - `SearchForge/frontend/js/components/ResultCard.js`
- **Accomplishes:** Domain-aware product card showing key fields per domain
- **Dependencies:** Task 9.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Shows: image thumbnail, title, price, location, domain-specific badges
  - Domain accent color border/highlight
  - Deal quality badge (green/amber/red)
  - "Compare" checkbox
  - Click opens detail drawer
  - Hover shows quick actions (save, share, compare)
- **Test:** Render card for each domain type, verify fields display correctly

---

### Task 9.5 — ListingTable component

- **Files to create:**
  - `SearchForge/frontend/js/components/ResultGrid.js`
- **Accomplishes:** Grid/list/map view toggle for search results
- **Dependencies:** Task 9.4
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Grid view: card layout (3-4 columns)
  - List view: table layout with sortable columns
  - View toggle buttons
  - Loading skeleton animation
  - Empty state message
  - Result count display
- **Test:** Toggle views, verify rendering, check responsive behavior

---

### Task 9.6 — DetailView

- **Files to create:**
  - `SearchForge/frontend/js/components/DetailDrawer.js`
- **Accomplishes:** Slide-in detail panel showing full product information
- **Dependencies:** Task 9.1, 8.2, 5.5
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Slides in from right (300px wide)
  - Shows: all universal fields + domain-specific fields
  - Image gallery (if available)
  - Price history chart
  - Valuation section (estimated value, confidence, comparables)
  - External link to original listing
  - Close button + Esc key
- **Test:** Click card, verify drawer opens with full data

---

### Task 9.7 — ComparisonView

- **Files to create:**
  - `SearchForge/frontend/js/components/ComparisonTray.js`
  - `SearchForge/frontend/js/views/CompareView.js`
- **Accomplishes:** Fixed bottom comparison tray + full comparison page
- **Dependencies:** Task 9.1, 8.9
- **Estimated time:** 3 hours
- **Acceptance criteria:**
  - Tray: fixed bottom bar showing selected products (max 4)
  - Tray: remove individual, clear all, "Compare" button
  - CompareView: side-by-side table with all fields
  - Highlight differences between products
  - Cross-domain comparison (CNC vs car = different fields shown)
  - Export comparison as CSV/PDF
- **Test:** Select products, compare, verify field differences highlighted

---

### Task 9.8 — CommandPalette

- **Files to create:**
  - `SearchForge/frontend/js/components/CommandPalette.js`
- **Accomplishes:** Ctrl+K global command palette for quick navigation and search
- **Dependencies:** Task 9.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Opens on Ctrl+K
  - Search input with fuzzy matching
  - Commands: switch domain, navigate, toggle theme, trigger scrape
  - Results: products + commands + recent searches
  - Keyboard navigation
  - Esc to close
- **Test:** Press Ctrl+K, verify palette opens, search for products/commands

---

### Task 9.9 — Pagination

- **Files to create:**
  - `SearchForge/frontend/js/components/Pagination.js`
- **Accomplishes:** Reusable pagination component
- **Dependencies:** Task 9.1
- **Estimated time:** 45 min
- **Acceptance criteria:**
  - Page numbers with ellipsis for large ranges
  - Previous/Next buttons
  - "Showing X-Y of Z" text
  - Jump to page input
  - Syncs with URL params
- **Test:** Navigate pages, verify URL updates, verify data loads

---

### Task 9.10 — Status components (loading, empty, error)

- **Files to create:**
  - `SearchForge/frontend/js/components/StatusBar.js`
- **Accomplishes:** Loading skeletons, empty states, error messages, status bar
- **Dependencies:** Task 9.1
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - Loading: skeleton placeholders matching card layout
  - Empty: illustration + "No results found" + suggestions
  - Error: error icon + message + retry button
  - StatusBar: bottom bar with: domain, result count, AI cost today, connection status
- **Test:** Trigger each state, verify rendering

---

### Task 9.11 — Dashboard view

- **Files to create:**
  - `SearchForge/frontend/js/views/DashboardView.js`
- **Accomplishes:** Analytics dashboard with domain stats, recent activity, market trends
- **Dependencies:** Task 9.1, 8.9
- **Estimated time:** 2.5 hours
- **Acceptance criteria:**
  - Stats cards: total products, active domains, listings today, AI cost
  - Per-domain breakdown (mini charts)
  - Recent listings feed
  - Price trend chart (line chart, last 30 days)
  - Top deals list
- **Test:** Navigate to #dashboard, verify stats load from API

---

### Task 9.12 — Alert management view

- **Files to create:**
  - `SearchForge/frontend/js/views/AlertView.js`
- **Accomplishes:** Alert rule management UI: create, edit, toggle, view history
- **Dependencies:** Task 9.1, 8.7
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Rule list with enable/disable toggle
  - Create/edit rule form (domain, conditions, webhook URL)
  - Test alert button (send test notification)
  - Alert history table with status icons
  - Delete with confirmation
- **Test:** Create rule, toggle, test notification, view history

---

## Phase 10: Vertical Packs (Days 16-18)

### Task 10.1 — CNC schema YAML

- **Files to create:**
  - `SearchForge/config/verticals/cnc_machines.yaml`
- **Accomplishes:** Complete CNC machinery vertical definition
- **Dependencies:** Task 1.2
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Defines all CNC fields (spindle, travel, control, dimensions, etc.)
  - Condition scale mapping (new → as_is → for_parts)
  - Scraping configs (Machinio, EquipNet, eBay CNC)
  - Search config (searchable fields, weights)
  - Passes schema validation (Task 1.1)
- **Test:** `cd SearchForge && python -c "from src.core.schema_engine import SchemaRegistry; r = SchemaRegistry(); s = r.get('cnc_machines'); print(s.name)"`

---

### Task 10.2 — Endmill schema YAML

- **Files to create:**
  - `SearchForge/config/verticals/endmills.yaml`
- **Accomplishes:** Complete endmill vertical definition
- **Dependencies:** Task 1.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Defines all endmill fields (diameter, flutes, coating, material, etc.)
  - Condition scale mapping
  - Scraping config (eBay Browse API)
  - Search config
  - Passes schema validation
- **Test:** `cd SearchForge && python -c "from src.core.schema_engine import SchemaRegistry; r = SchemaRegistry(); s = r.get('endmills'); print(s.name)"`

---

### Task 10.3 — Car schema YAML

- **Files to create:**
  - `SearchForge/config/verticals/used_cars.yaml`
- **Accomplishes:** Complete used car vertical definition
- **Dependencies:** Task 1.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Defines all car fields (VIN, engine, transmission, body, etc.)
  - Condition scale mapping (certified_preowned → salvage)
  - Scraping config (NHTSA, CarGurus, Autotrader)
  - Search config
  - Passes schema validation
- **Test:** `cd SearchForge && python -c "from src.core.schema_engine import SchemaRegistry; r = SchemaRegistry(); s = r.get('used_cars'); print(s.name)"`

---

### Task 10.4 — CNC AI prompts

- **Files to create:**
  - `SearchForge/src/domains/cnc_machinery/prompts.py`
- **Accomplishes:** Domain-specific AI extraction prompts for CNC machines
- **Dependencies:** Task 1.5, 10.1
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - System prompt for CNC specification extraction
  - JSON schema for output format
  - Handles: missing data, ambiguous specs, different notations
  - Tested with sample CNC listing text
- **Test:** `cd SearchForge && pytest tests/test_domains/test_cnc_prompts.py -v`

---

### Task 10.5 — Endmill AI prompts

- **Files to create:**
  - `SearchForge/src/domains/endmills/prompts.py`
- **Accomplishes:** Domain-specific AI extraction prompts for endmills
- **Dependencies:** Task 1.5, 10.2
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - System prompt for endmill specification extraction
  - JSON schema for output format
  - Handles: multi-piece listings, missing coatings, imperial/metric confusion
- **Test:** `cd SearchForge && pytest tests/test_domains/test_endmill_prompts.py -v`

---

### Task 10.6 — Car AI prompts

- **Files to create:**
  - `SearchForge/src/domains/used_cars/prompts.py`
- **Accomplishes:** Domain-specific AI extraction prompts for used cars
- **Dependencies:** Task 1.5, 10.3
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - System prompt for car specification extraction from listing text
  - JSON schema for output format
  - Handles: trim identification, feature list extraction, condition assessment
- **Test:** `cd SearchForge && pytest tests/test_domains/test_car_prompts.py -v`

---

### Task 10.7 — CNC valuation config

- **Files to create:**
  - `SearchForge/src/domains/cnc_machinery/scorer.py`
- **Accomplishes:** CNC deal scoring: multi-factor weighted model
- **Dependencies:** Task 5.1, 5.2, 10.1
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Weights: age (15%), hours (25%), brand (10%), spindle_power (15%), features (15%), condition (20%)
  - Comparable-sales based pricing
  - Returns 0.0-1.0 deal score + detail breakdown
  - "Great deal" / "Fair" / "Overpriced" labels
- **Test:** `cd SearchForge && pytest tests/test_domains/test_cnc_scorer.py -v`

---

### Task 10.8 — Endmill valuation config

- **Files to create:**
  - `SearchForge/src/domains/endmills/scorer.py`
- **Accomplishes:** Endmill deal scoring: per-unit price comparison
- **Dependencies:** Task 5.1, 5.2, 10.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Normalize to per-unit price (handle multi-packs)
  - Compare against same brand + diameter + coating
  - Weight: price_per_unit (40%), brand (20%), coating (20%), condition (20%)
  - Returns deal score + detail
- **Test:** `cd SearchForge && pytest tests/test_domains/test_endmill_scorer.py -v`

---

### Task 10.9 — Car valuation config

- **Files to create:**
  - `SearchForge/src/domains/used_cars/scorer.py`
- **Accomplishes:** Car deal scoring: market-value based
- **Dependencies:** Task 5.1, 5.2, 10.3
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Compare against same make + model + year range + mileage range
  - Weight: price_vs_market (35%), mileage (20%), age (15%), condition (15%), history (15%)
  - Returns deal score + detail
  - Special handling: certified pre-owned, low-mileage premium
- **Test:** `cd SearchForge && pytest tests/test_domains/test_car_scorer.py -v`

---

## Phase 11: Migration (Days 18-19)

### Task 11.1 — CNC data migration script

- **Files to create:**
  - `SearchForge/scripts/import_legacy_cnc.py`
- **Accomplishes:** Import existing data from CNC mill search SQLite database
- **Dependencies:** Task 2.2, 2.3, 2.9, 10.7
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Reads legacy `machines.db` table
  - Maps 60+ columns to SearchForge schema
  - Normalizes condition scores
  - Parses price strings to USD float
  - Extracts domain fields into `cnc_machines` extension table
  - Generates FTS5 search_text
  - Computes deal scores
  - Dry-run mode (`--dry-run` flag)
  - Reports: imported, skipped, errors
- **Test:** `cd SearchForge && python scripts/import_legacy_cnc.py --dry-run --db-path "../../CNC mill search/backend/searchforge.db"`

---

### Task 11.2 — eBay data migration script

- **Files to create:**
  - `SearchForge/scripts/import_legacy_ebay.py`
- **Accomplishes:** Import existing data from eBay endmills SQLite database
- **Dependencies:** Task 2.2, 2.3, 2.9, 10.8
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - Reads legacy endmill listings table
  - Maps parsed specs to endmill extension columns
  - Normalizes condition scores
  - Parses price to USD
  - Generates FTS5 search_text
  - Deduplicates by source_url
  - Reports: imported, skipped, errors
- **Test:** `cd SearchForge && python scripts/import_legacy_ebay.py --dry-run --db-path "../../Ebay endmills/data/endmills.db"`

---

### Task 11.3 — Embedding migration

- **Files to create:**
  - `SearchForge/scripts/generate_embeddings.py`
- **Accomplishes:** Generate sentence-transformer embeddings for all products for semantic search
- **Dependencies:** Task 2.2, 0.2
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Loads `all-MiniLM-L6-v2` model
  - Batch-processes products (128 at a time)
  - Stores 384-dim vectors in `products.embedding` column
  - Progress bar with ETA
  - Skip products that already have embeddings
  - Estimated: ~100 products/second on CPU
- **Test:** `cd SearchForge && python scripts/generate_embeddings.py --limit 100`

---

### Task 11.4 — Validation checks

- **Files to create:**
  - `SearchForge/scripts/validate_migration.py`
- **Accomplishes:** Post-migration data integrity checks
- **Dependencies:** Task 11.1, 11.2
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - Checks: row counts match between legacy and SearchForge
  - Checks: all products have search_text
  - Checks: all products have valid domain
  - Checks: no orphaned extension records
  - Checks: all extension records have matching product
  - Checks: deal scores are 0.0-1.0
  - Checks: condition scores are 0.0-1.0
  - Reports: passed, warnings, failures
- **Test:** `cd SearchForge && python scripts/validate_migration.py`

---

## Phase 12: Testing (Days 19-21)

### Task 12.1 — Unit test suite

- **Files to create/verify:**
  - All `tests/test_core/test_*.py` files (15+ test files)
  - All `tests/test_domains/test_*.py` files (6+ test files)
  - `SearchForge/tests/conftest.py`
- **Accomplishes:** Comprehensive unit test coverage for all core modules
- **Dependencies:** All Phase 1-5 tasks
- **Estimated time:** 4 hours
- **Acceptance criteria:**
  - ≥80% coverage on core modules
  - 100% coverage on normalization and pricing
  - All tests pass
  - No test takes >5 seconds
- **Test:** `cd SearchForge && pytest tests/ --cov=src --cov-report=html -v`

---

### Task 12.2 — Integration test suite

- **Files to create:**
  - `SearchForge/tests/test_api/test_products.py` (full CRUD flows)
  - `SearchForge/tests/test_api/test_search.py` (end-to-end search)
  - `SearchForge/tests/test_api/test_scrape.py` (job lifecycle)
  - `SearchForge/tests/test_api/test_enrichment.py` (enrichment pipeline)
  - `SearchForge/tests/test_integration/test_full_pipeline.py`
- **Accomplishes:** Integration tests using FastAPI TestClient with in-memory SQLite
- **Dependencies:** All Phase 1-8 tasks
- **Estimated time:** 3 hours
- **Acceptance criteria:**
  - 10-15 integration tests
  - Tests use in-memory SQLite (no disk I/O)
  - Tests verify: API → DB → Search → Response round-trip
  - All tests pass
- **Test:** `cd SearchForge && pytest tests/test_api/ tests/test_integration/ -v`

---

### Task 12.3 — E2E test suite

- **Files to create:**
  - `SearchForge/tests/test_integration/test_e2e_scrape_to_search.py`
  - `SearchForge/tests/test_integration/test_e2e_enrich_to_valuation.py`
- **Accomplishes:** End-to-end tests: scrape → parse → enrich → store → search → value
- **Dependencies:** All previous phases
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - 2-3 E2E tests covering full pipeline
  - Mock external APIs (no real network calls in CI)
  - Verify: raw data → normalized listing → enriched listing → searchable → valued
- **Test:** `cd SearchForge && pytest tests/test_integration/test_e2e_*.py -v`

---

### Task 12.4 — Golden dataset creation

- **Files to create:**
  - `SearchForge/tests/fixtures/golden_cnc_listings.json` (20 listings)
  - `SearchForge/tests/fixtures/golden_endmill_listings.json` (20 listings)
  - `SearchForge/tests/fixtures/golden_car_listings.json` (20 listings)
  - `SearchForge/tests/fixtures/golden_search_results.json`
- **Accomplishes:** Curated test datasets for regression testing
- **Dependencies:** All Phase 1-10 tasks
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - 20 real-world examples per domain (hand-verified)
  - Expected parsed output for each
  - Expected search results for common queries
  - Used in integration tests as fixtures
- **Test:** `cd SearchForge && pytest tests/test_integration/ -v --fixtures`

---

### Task 12.5 — Performance benchmarks

- **Files to create:**
  - `SearchForge/tests/benchmarks/test_bench_search.py`
  - `SearchForge/tests/benchmarks/test_bench_upsert.py`
  - `SearchForge/tests/benchmarks/benchmark_results.md`
- **Accomplishes:** Performance benchmarks for critical paths
- **Dependencies:** All previous phases
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - Search latency: <100ms for 10K products
  - Upsert: >50 products/second bulk insert
  - Faceted search: <200ms for 10K products
  - Autocomplete: <50ms
  - Results documented in `benchmark_results.md`
- **Test:** `cd SearchForge && pytest tests/benchmarks/ -v --benchmark`

---

## Phase 13: Polish (Days 21-22)

### Task 13.1 — Error handling audit

- **Files to modify:**
  - All `src/api/routers/*.py`
  - `SearchForge/src/api/utils/errors.py`
- **Accomplishes:** Ensure consistent error handling across all API endpoints
- **Dependencies:** All Phase 8 tasks
- **Estimated time:** 1.5 hours
- **Acceptance criteria:**
  - All endpoints return proper HTTP status codes
  - All errors return structured JSON (`{"error": "...", "detail": "...", "code": "..."}`)
  - No unhandled exceptions reach the client
  - Rate limit errors return 429 with Retry-After header
  - Validation errors return 422 with field-level details
- **Test:** `cd SearchForge && pytest tests/test_api/ -v -k "error"`

---

### Task 13.2 — Logging configuration

- **Files to create:**
  - `SearchForge/src/logging_config.py`
- **Accomplishes:** Structured logging with rotation, correlation IDs, request tracing
- **Dependencies:** Task 0.2
- **Estimated time:** 1 hour
- **Acceptance criteria:**
  - JSON structured logs (production)
  - Colored console logs (development)
  - Log rotation: 10MB per file, 5 backups
  - Request correlation ID in all request logs
  - Separate log levels: DEBUG (core), INFO (API), WARNING (scraping)
- **Test:** `cd SearchForge && python -c "from src.logging_config import setup_logging; setup_logging('DEBUG'); print('OK')"`

---

### Task 13.3 — README + documentation

- **Files to create:**
  - `SearchForge/README.md`
  - `SearchForge/docs/architecture.md`
  - `SearchForge/docs/api-reference.md`
  - `SearchForge/docs/adding-a-domain.md`
- **Accomplishes:** Complete project documentation
- **Dependencies:** All previous phases
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - README: project overview, quick start, tech stack, screenshots
  - Architecture: system diagram, data flow, design decisions
  - API Reference: all endpoints with request/response examples
  - Adding a Domain: step-by-step guide with checklist
- **Test:** All markdown files pass lint (`markdownlint`)

---

### Task 13.4 — Final testing pass

- **Files to modify:**
  - Any failing tests from previous phases
  - Any broken imports or references
- **Accomplishes:** Full test suite passes, all code quality checks green
- **Dependencies:** All previous tasks
- **Estimated time:** 2 hours
- **Acceptance criteria:**
  - `make test` passes 100%
  - `make lint` passes 100%
  - `make format` shows no changes
  - No TODO/FIXME in production code
  - All docstrings present for public APIs
- **Test:** `cd SearchForge && make test && make lint`

---

## Summary Statistics

| Phase | Tasks | Estimated Time | Key Dependencies |
|-------|-------|----------------|------------------|
| 0: Setup | 5 | 1.25 hours | None |
| 1: Schema Engine | 7 | 10.5 hours | Phase 0 |
| 2: Database | 9 | 11.25 hours | Phase 0 |
| 3: Connectors | 9 | 17 hours | Phase 0, 1, 2 |
| 4: AI Enrichment | 10 | 14.25 hours | Phase 0, 1, 2 |
| 5: Valuation | 5 | 8 hours | Phase 2, 4 |
| 6: Search | 6 | 8.5 hours | Phase 2 |
| 7: Alerts | 5 | 6.25 hours | Phase 2, 6 |
| 8: API | 10 | 14.5 hours | Phase 2, 5, 6, 7 |
| 9: Frontend | 12 | 22.5 hours | Phase 8 |
| 10: Verticals | 9 | 13 hours | Phase 1, 4, 5, 8 |
| 11: Migration | 4 | 6.5 hours | Phase 2, 8, 10 |
| 12: Testing | 5 | 12.5 hours | All |
| 13: Polish | 4 | 6.5 hours | Phase 12 |
| **Total** | **100** | **~153 hours** | |

---

## File Count by Category

| Category | Files | Directories |
|----------|-------|-------------|
| Source (src/) | ~50 | 8 |
| Frontend | ~18 | 4 |
| Config | ~8 | 2 |
| Tests | ~35 | 4 |
| Scripts | ~6 | 1 |
| Docs | ~4 | 1 |
| Root | ~8 | — |
| **Total** | **~129** | **~20** |

---

## Parallelization Opportunities

| Wave | Tasks | Can Run In Parallel |
|------|-------|---------------------|
| Wave 1 | 0.1-0.5 | All 5 tasks |
| Wave 2 | 1.1-1.7 | Tasks 1.1→1.2→1.3 (sequential), 1.4 (parallel with 1.3), 1.5-1.7 (after 1.3) |
| Wave 3 | 2.1-2.9 | 2.1→2.2→2.3→2.4 (sequential), 2.5-2.8 (parallel after 2.1), 2.9 (after 2.4) |
| Wave 4 | 3.1-3.9 | 3.1→3.2, 3.3 (parallel), then 3.4-3.8 (parallel), 3.9 (parallel) |
| Wave 5 | 4.1-4.10 | 4.1→4.2-4.4 (parallel), 4.5-4.6 (parallel), 4.7-4.10 (sequential chain) |
| Wave 6 | 5.1-5.5, 6.1-6.6 | Phase 5 and 6 can run in parallel |
| Wave 7 | 7.1-7.5, 8.1-8.10 | Phase 7 and 8 can partially overlap |
| Wave 8 | 9.1-9.12 | Mostly sequential (build on each other) |
| Wave 9 | 10.1-10.9 | 10.1-10.3 parallel, 10.4-10.6 parallel, 10.7-10.9 parallel |
| Wave 10 | 11.1-11.4 | 11.1 and 11.2 parallel, then 11.3, then 11.4 |
| Wave 11 | 12.1-12.5 | Sequential (each builds on previous) |
| Wave 12 | 13.1-13.4 | 13.1-13.3 parallel, 13.4 last |

---

*Master task breakdown complete. Ready for execution.*

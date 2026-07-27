# SearchForge — Design Documents & Specifications

> **Planning repo for the SearchForge / Universal Shopping Search Engine.**
> This repository contains the master task breakdown and complete REST API specification that guided the implementation in [Universal-Shopping-Search](https://github.com/Reese000/Universal-Shopping-Search).

---

## What's Here

| File | Lines | Purpose |
|------|-------|---------|
| `MASTER_TASK_BREAKDOWN.md` | 1,944 | ~95 tasks across 14 phases — every file, dependency, and test specified |
| `docs/api-specification.md` | 3,173 | Complete REST API spec — 41 endpoints, auth, rate limiting, error handling, OpenAPI |

## Relationship to Other Repos

| Repository | Relationship |
|------------|-------------|
| **Universal-Shopping-Search** | The implementation. This repo's specs drove its architecture. The USS README cross-references these files via `../SearchForge/`. |
| **SearchForge-UI** | The UI design system and component library for the same product. Should arguably be a subfolder here (see recommendation below). |

### Recommendation: SearchForge-UI as Subfolder

SearchForge-UI contains UI architecture specs, HTML component references, and CSS design tokens — all planning/design artifacts for the same SearchForge product. It would be more logical as `SearchForge/ui/` rather than a standalone repo. The user has been advised to consider merging it into this repo before publishing SearchForge-UI separately.

## Product Overview

**SearchForge** (also known as **Universal Shopping Search Engine**) is a vertical-agnostic shopping search engine that:

- **Aggregates** listings from multiple marketplaces (eBay, Machinio, AutoTrader, CarGurus)
- **Normalizes** data across product verticals using YAML schema definitions
- **AI-enriches** listings via a tiered pipeline (regex → LLM → vision) for 82% cost reduction
- **Values** items using comparable-sales analysis
- **Searches** with SQLite FTS5 + Meilisearch (typo-tolerant, faceted)
- **Alerts** users on price drops and new listings

### Supported Verticals

| Vertical | Accent Color | Key Attributes |
|----------|-------------|----------------|
| CNC Machines | Orange `#ff9a5c` | Type, manufacturer, travel envelope, spindle speed, control brand |
| Endmills | Teal `#5ce1e6` | Diameter, flute count, coating, material, type |
| Used Cars | Blue `#6c8aff` | Make/model, year, mileage, VIN, drivetrain, title status |

## Tech Stack (Implementation)

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.11, FastAPI, SQLAlchemy (async), SQLite (WAL), Pydantic v2 |
| Search | SQLite FTS5 + Meilisearch |
| Semantic | sentence-transformers (all-MiniLM-L6-v2) + sqlite-vec |
| Scraping | Playwright (stealth), httpx, BeautifulSoup4 |
| AI | OpenRouter (DeepSeek text + Gemini vision) + local Ollama |
| Frontend | Jinja2 + HTMX + Alpine.js, Reese Design System tokens.css |
| Testing | pytest, pytest-asyncio, pytest-cov |

## Status

**Planning: COMPLETE.** All architectural decisions debated and resolved. Implementation is in the [Universal-Shopping-Search](https://github.com/Reese000/Universal-Shopping-Search) repo.

*Planning completed: 2026-07-11*

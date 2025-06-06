Ancient-Megalith Knowledge Graph — Build Specification

1 — Project Goal

Create a queryable, extensible database + API + map UI that unifies archaeological, geometric, acoustic, astronomical, mythological, and materials data for ~100 key megalithic sites worldwide.
The system must let users:
1.Search or filter sites by any attribute (e.g., “quartz > 25 % AND resonance ≈ 110 Hz”).
2.Visualise spatial patterns (Mapbox layer).
3.Drill down to primary sources (PDFs / citation links).
4.Add new evidence rows without touching backend code (CSV upload or admin UI).

2 — Data Model

2.1 Core Tables

TablePurposeKey Fields
SitesOne record per monument/complexsite_id PK, name, country, lat, lng, era_bp, status
GeometryRatios & layout metricsgeo_id PK, site_id FK, phi_ratio, pi_ratio, three_four_five (bool), concavity
MaterialStone/mineral datamat_id PK, site_id FK, primary_lithology, quartz_pct, piezo_flag
AlignmentAstronomical & cardinalalign_id PK, site_id FK, azimuth_deg, target_body (sun, star), precision_arcmin
AcousticsResonance testsacous_id PK, site_id FK, fundamental_hz, secondary_hz, method
MythsLocal flood / origin mythsmyth_id PK, site_id FK, tradition, summary, cataclysm_type
SourcesPrimary literaturesrc_id PK, doi_or_url, title, authors, year
ClaimsEvery research claimclaim_id PK, site_id FK, table_ref, field_ref, claim_text, src_id FK, confidence_lvl

PK = primary key, FK = foreign key.

2.2 Schema format

Start with SQLite for dev; upgrade to Postgres when scale needs (> 1 k sites). Use SQLAlchemy models so switching DB is seamless.

3 — Data Ingestion Pipeline
1.Seed CSV → ingest/seed_sites.csv (columns match Sites table).
2.Python ETL (scripts/etl.py) reads CSV, validates lat/lng & era, inserts into DB.
3.Secondary CSVs (geometry.csv, materials.csv, etc.) follow the same pattern, referencing site_id.
4.scripts/link_sources.py scrapes metadata (DOI → title, authors) via CrossRef API and fills Sources table.
5.Unit tests ensure referential integrity after each load (pytest tests/).

4 — API Layer
•Framework: FastAPI.
•Endpoints:
•GET /sites – filter by country, era, material, etc.
•GET /site/{id} – full join across all tables.
•POST /claim – add or update claims with source linkage.
•GET /search?q= – MeiliSearch query across site names, myths, claims.
•Output format: JSON & GeoJSON.

5 — Frontend
•Tech: React + Mapbox GL JS.
•Components:
1.Global map layer (<SiteMap>) – coloured pins by Category.
2.<SiteDrawer> – slides out with all joined data, link to PDFs.
3.<FilterPanel> – multi-select chips (quartz %, resonance range, alignment).
4.<TimelineSlider> – filter by era (drag 30 000 BP → 0 BP).
•Deploy on Vercel (static build) or Netlify; env vars store API URL.

6 — Admin / Data-Entry UI (Phase 2)

Simple Next.js or SvelteKit form gated by GitHub OAuth:
•Upload new CSV or single-row form entry.
•Inline PDF/URL validation and DOI resolution.
•Auto-generates pull-request on data/ folder for review.

7 — Testing & CI/CD

LayerToolGoal
DB migrationsAlembicSchema versioning
APIPytest + HTTPXEndpoint health & auth
FrontendPlaywrightMap & filter rendering
PipelineGitHub ActionsLint, test, build, deploy to staging

8 — File / Repo Structure

ancient-megalith-kg/
├─ backend/
│  ├─ app/              # FastAPI
│  ├─ models.py
│  ├─ routers/
│  ├─ scripts/
│  └─ tests/
├─ frontend/
│  ├─ src/
│  ├─ public/
│  └─ tests/
├─ data/
│  ├─ seed_sites.csv
│  ├─ geometry.csv
│  └─ ...
├─ ingest/
│  └─ notebooks/        # Jupyter exploration
├─ docs/
│  └─ research_papers/
└─ README.md            # This spec

9 — Stretch Goals
•Graph-DB mirror (Neo4j) to query “find all sites with Φ-ratio AND flood myth”.
•Real-time sensor feed: plug a portable seismograph into King’s Chamber resonance dataset when field access becomes possible.
•Automated citation crawler: monitor arXiv & Nature-Comms for new megalith studies; webhook into database.

Hand-off

Commit this spec as README.md in the repo root.
First sprint: build SQLite schema + ETL scripts and push the seed CSV (download link above).

Ping me once the skeleton repo is live—I’ll prepare the Mapbox starter and the first batch of verified PDF citations.

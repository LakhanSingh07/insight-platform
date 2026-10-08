# Insight — Organizational Intelligence · v0.3.1 accuracy & UX hardening

An incremental v0.3.1 release on the existing v0.3 source. See [HARDENING_V031.md](HARDENING_V031.md) for the workbook regression fixes, automatic bento dashboards, run commands and verification limits. See [PHASE3.md](PHASE3.md) for evidence-based understanding, runtime concepts, semantic compilation, schema drift and verification limits. [PHASE2.md](PHASE2.md) documents the PostgreSQL, hybrid retrieval, analytical tools and dashboard foundation.

A working organizational intelligence platform. Core code is ours: Next.js frontend, FastAPI backend, SQLAlchemy metadata, Polars profiling, immutable Parquet snapshots, governed metrics, structured planning, SQLGlot validation and DuckDB execution.

## What works

- Universal value-based understanding, revisioned semantic review, organization concepts, safe anonymous-field exploration, concept-backed metrics and explicit snapshot remapping. No invented business meaning for unknown columns.

- Account registration/sign-in, organizations and default workspaces; authenticated, tenant-scoped APIs.
- Real CSV, XLSX/XLS, JSON and Parquet uploads. Every populated workbook sheet becomes an independent dataset; manual header-row override and paginated preview.
- Schema inference, numeric/date normalization, duplicate/null/range checks, a disclosed quality score, basic PII masking and semantic metadata.
- Relationship suggestions from unique target keys and actual value overlap. Analyst approval required; target uniqueness is rechecked before executing joins.
- KPI suggestions, typed custom metrics, formula editing, approval/certification/version attribution and business glossary aliases.
- Natural-language rules planner: approved aggregates, category groupings, monthly trends, first/last-month comparison, last N complete calendar months and conversation follow-ups. Optional schema-constrained OpenAI planner for additional phrasing; no API key is needed for supported local analyses.
- Compiled SQL, SQLGlot statement/relation/function validation, DuckDB execution, result checks and source/formula/filter/version/confidence panel. No LLM arithmetic.
- Multi-query root-cause evidence: endpoint comparison, reporting/denominator change, entity declines and weighted district contribution. Results are associations, not proof of causes.
- ECharts chart/table/KPI results, saved snapshot dashboards, governed dashboard generation and editable widget order/type/size/duplication/removal.
- PDF reports, CSV/XLSX result exports, threshold rules, scheduled alert worker, notification outbox and configured SMTP delivery with PDF attachments.
- Branded PNG data graphics in seven canvas presets including custom sizes. Numbers/charts render deterministically; no automatic social publishing.
- Audit trail and interpretation feedback.

No seeded accounts, hardcoded dashboard statistics or fake server responses. The provided education fixture is synthetic test data and must be uploaded explicitly.

## Quick local start (no Docker)

Requirements: Python 3.12+, Node 22+, npm. From this repository root:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
npm --prefix apps/web ci
python -m uvicorn apps.api.main:app --host 127.0.0.1 --port 8000
```

In a second terminal, from the same repository root:

```bash
npm --prefix apps/web run dev
```

Open http://localhost:3000. Create your own account. API docs: http://localhost:8000/docs.

Without `DATABASE_URL`, local mode **explicitly uses SQLite metadata in `var/development.db`**. This is disclosed by `/health` and the UI. The analytics engine is real DuckDB/Polars in this mode. Do not copy `.env.example` unchanged for this mode; its DATABASE_URL points to PostgreSQL. A per-installation secret is generated privately in `var/.app-secret` if APP_SECRET is absent. Persist `var` to retain accounts, files and reports.

To use a native PostgreSQL instance locally, set `DATABASE_URL=postgresql+psycopg://USER:PASSWORD@HOST:5432/DATABASE` and ensure the vector extension is installed and that the initialization account may enable it. Never silently fall back when a configured database fails.

Optional alert worker:

```bash
python -m services.alerts.worker
```

Only activated rules are scheduled. Tests do not send email. With `EMAIL_PROVIDER=outbox`, messages are persisted as `queued_outbox`; nothing is falsely marked delivered.

## Docker local environment

```bash
cp .env.example .env
# Edit .env: replace APP_SECRET with a newly generated secret.
# Example generation: python -c "import secrets; print(secrets.token_urlsafe(48))"
docker compose up --build
```

Open http://localhost:3000. Compose starts PostgreSQL 16 + pgvector, API, web and alert worker. The web image embeds the Docker API origin during the build; the local npm build uses localhost. PostgreSQL and upload volumes are persistent. `docker compose down` stops services without deleting data. Do not use `down -v` unless you intentionally want to erase it.

Optional Redis/MinIO containers (their runtime adapters are later work):

```bash
docker compose --profile integrations up --build
```

**Verification limit:** native Docker/PostgreSQL startup has not been executed here. The actual psycopg/PostgreSQL application path, schema, pgvector/FTS, browser flow and restart persistence are tested using PGlite wire protocol. This is not equivalent to native deployment or concurrency validation.

## Run the education flow

1. Upload `tests/fixtures/education.xlsx` (synthetic sample; three schools, July–September 2026 attendance).
2. In Data workspace, select Attendance and approve its school_id relationship to School Master.
3. Approve or certify Student Attendance Rate in Metric Registry.
4. In Ask Data, choose that metric and ask:
   `Which districts in Nagaland had the biggest attendance decline during the last three months?`
5. Inspect chart, table, plan, SQL, source hashes and formula. On 6 October 2026, the fixture yields Dimapur −30 percentage points and Kohima +10 percentage points over July–September.
6. Add the result to a dashboard, export a report or create a threshold rule/data graphic.

Relative dates are resolved against the server's calendar date. This fixture is fixed in 2026; for repeatable testing on another date, ask `Which districts in Nagaland had the biggest attendance decline from 2026-07-01 to 2026-09-30?`. Sparse sample records do not demonstrate complete daily reporting coverage.

For a larger multi-file test, upload the four workbooks under `tests/fixtures/education-demo/`: 30 synthetic schools and 1,980 daily attendance records, plus enrollment and assessment. Regenerate with `python scripts/generate_demo.py`. These are optional fixtures, never default product data.

## AI configuration

```dotenv
AI_PROVIDER=openai
OPENAI_API_KEY=your-key
OPENAI_MODEL=a-structured-output-capable-model-you-have-access-to
AI_BASE_URL=https://api.openai.com/v1
```

Provider adapter sends approved semantic definitions, permitted entity values and structured conversation state, not source rows or database credentials. It requests strict JSON Schema output using Chat Completions. All plans are validated and compiled by our engine. Provider errors/unsupported local requests return clarification; never fabricated figures. Phase 2 also implements the Gemini adapter and optional pgvector embeddings plus PostgreSQL FTS/exact/entity retrieval. See PHASE2.md for configuration and indexing. Live paid-provider calls have not been verified.

## Phase 2 additions

Versioned PostgreSQL migrations and workspace constraints; OpenAI/Gemini adapters; scoped hybrid retrieval and verified question memory; eight deterministic analytical tools and evidence synthesis; versioned canvas sections, global filters, metric/dimension editing and resizing; diagnostic PDFs and advanced alert conditions.

## Checks

```bash
python -m pytest -q
npm --prefix apps/web run typecheck
npm --prefix apps/web run build
cd infrastructure/schema-check
npm ci
npm test
```

Optional Linux headless browser integration test, from repository root:

```bash
npm --prefix infrastructure/browser-check ci
python scripts/runtime-smoke.py
```

The browser script starts both servers, runs the actual upload-to-dashboard UI, and stops those test servers. See `TESTING.md` for exact observed results and limitations.

## Important scope boundaries

This is a functional first milestone, **not a production-certified government analytics service**. Native PostgreSQL runtime, SSO/team invitations, full ABAC/row policies, comprehensive PII detection, connector refresh, live embedding/provider quality, inferential statistics, forecast/cohort/correlation, report scheduling, PPTX/DOCX/maps and social auto-posting are not finished. Alerts/dashboard results refer to immutable selected dataset snapshots; uploads create new versions but do not automatically rebind existing metrics. See `ROADMAP.md`, `SECURITY.md` and `DEPLOYMENT.md` before external deployment.

PostgreSQL dialect + driver + browser + restart check (WASM, not native):

```bash
npm --prefix infrastructure/schema-check ci
npm --prefix infrastructure/browser-check ci
mkdir -p var
python scripts/postgres-wire-check.py --browser
```

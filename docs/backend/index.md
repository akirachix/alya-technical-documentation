# Backend

## 1. Overview

The KNode backend is the application and data layer for a platform that helps African-focused organisations understand funding requirements, identify complementary partners, and organise collaboration around funding opportunities. It exposes JSON HTTP endpoints and coordinates authentication and authorisation, profile management, opportunity management, evidence processing, matching, collectives, referrals, workspaces, trend analysis, and external integrations.

The backend structures and surfaces evidence to support human decision-making. It does not decide funding outcomes, certify organisations, guarantee eligibility, or establish that a potential partner has agreed to participate. Missing information must remain distinguishable from negative evidence.

This document describes the backend's main components and points to the detailed references:

- [Database and Migrations](database.md)
- [API Reference](api-reference.md)
- [Authentication and Security](authentication-security.md)
- [Matching Engine](matching-engine.md)
- [Testing and Operations](testing.md)

## 2. Technology stack

| Area                     | Technology or library                                               | Notes                                                                                              |
| ------------------------ | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Runtime                  | Python 3.11 or newer                                                | Declared by project configuration.                                                                 |
| HTTP API                 | FastAPI, Uvicorn                                                    | JSON endpoints and generated OpenAPI documentation.                                                |
| Validation               | Pydantic v2, pydantic-settings                                      | Request/response schemas and environment settings.                                                 |
| ORM                      | SQLAlchemy 2.x                                                      | Models, sessions, queries and persistence.                                                         |
| Database                 | PostgreSQL                                                          | Configured using `DATABASE_URL`; tests also use SQLite.                                            |
| Driver                   | `psycopg2-binary`                                                   | Used by the PostgreSQL SQLAlchemy URL.                                                             |
| Migrations               | Alembic                                                             | Versioned schema changes.                                                                          |
| Authentication           | PyJWT, Argon2                                                       | JWTs and password hashing.                                                                         |
| AI                       | Google Gen AI SDK / Gemini provider                                 | Provider use depends on configuration and feature path.                                            |
| HTTP/document processing | Requests, BeautifulSoup, Playwright, pypdf, python-docx, lxml       | Used by selected scraping and document-processing paths.                                           |
| Testing                  | pytest, pytest-cov, FastAPI TestClient/httpx-compatible test client | Automated tests and coverage. Verify the declared HTTP client dependency against the test imports. |
| Lint/format              | Ruff                                                                | Linting and formatting checks.                                                                     |

**Dependency verification note:** The repository's declared versions and package names should be checked against the current `pyproject.toml` and lock/dependency resolution before setting up a clean environment. The audit material contains dependency declarations that warrant verification, including the `httpx2` entry and differing Alembic version constraints. Do not treat this table as proof that every declared package is installed in a production environment.

## 3. Backend architecture

The application uses a layered structure. Routers define HTTP endpoints, Pydantic schemas validate incoming and outgoing data, services implement business workflows, repositories encapsulate persistence queries, SQLAlchemy models define database entities, and core/infrastructure modules provide configuration, security, dependencies, logging, and error handling.

A typical request follows this path:

1. FastAPI matches the HTTP request to a registered route.
2. Pydantic validates path, query, and request-body values.
3. Dependencies resolve the database session and, for protected routes, the current user.
4. The router delegates business operations to a service.
5. The service applies business rules, access checks, repository operations, and any required external-provider calls.
6. Data is persisted through SQLAlchemy where applicable.
7. A response schema shapes the returned result.
8. Exception handlers and logging convert failures into client-safe responses and operational records.

Application wiring in `main.py` includes CORS configuration, correlation-ID middleware, error handlers, router registration, and the public `/health` endpoint.

## 4. Repository structure

The source tree includes these main areas:

| Path                                                             | Responsibility                                                                                    |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `main.py`                                                        | Creates the FastAPI app, registers middleware and routers, and exposes health checking.           |
| `core/config.py`                                                 | Application settings and environment configuration.                                               |
| `core/dependencies.py`                                           | Authentication, role and resource-scope dependencies.                                             |
| `core/security.py`                                               | Password hashing, JWT helpers, OTP and token utilities.                                           |
| `core/errors.py`, `core/error_handlers.py`, `core/exceptions.py` | Error types and HTTP exception handling; overlapping handlers should be consolidated or verified. |
| `core/logging.py`                                                | Logging configuration and correlation-related support.                                            |
| `database.py`                                                    | SQLAlchemy engine and session factory.                                                            |
| `models/`                                                        | ORM entities and custom database types.                                                           |
| `repositories/`                                                  | Database query and persistence helpers.                                                           |
| `routers/`                                                       | HTTP routes grouped by feature.                                                                   |
| `schemas/`                                                       | Pydantic request and response contracts.                                                          |
| `services/`                                                      | Business logic and integration workflows.                                                         |
| `services/matching_engine/`                                      | Requirement assessment, eligibility, scoring, partner discovery and collective coverage helpers.  |
| `services/jobs/`                                                 | Crawling, processing and batch-job entry points.                                                  |
| `alembic/`                                                       | Migration environment and versioned schema changes.                                               |
| `tests/`                                                         | API, service, repository, security, AI, matching and other tests.                                 |
| `.github/workflows/`                                             | CI, pull-request checks and deployment workflow definitions.                                      |
| `pyproject.toml`, `alembic.ini`, `Procfile`                      | Project configuration, migration settings and process declaration.                                |

## 5. Backend feature coverage

The backend supports multiple product areas; the Matching Engine is only one of them.

- **Authentication and user management:** registration, verification, login, token refresh/logout, current-user access, password and OTP-related flows.
- **Organisation management:** organisation profiles, team management, documents, and organisation evidence.
- **Funder management:** funder profiles, funding interests and funder evidence.
- **Funding opportunities:** opportunity creation, retrieval, update/deletion, extraction, and requirement management.
- **Evidence and claims:** source records, extracted claims, review status and provenance.
- **Matching and partner discovery:** requirement assessments, eligibility outputs, score/evidence summaries, and potential partner suggestions.
- **Collectives and coverage:** collective records, membership, invitations/responses, coverage and lead suggestions.
- **Referrals:** creation, retrieval, response and withdrawal workflows.
- **Workspaces:** collective-associated workspaces and workspace items.
- **Trend analysis:** analysis over available funding-related data.
- **Reports and audit:** report-related data structures and audit-event services exist; the presence of these structures does not prove every report or audit workflow is exposed or complete.

For detailed endpoint paths, see the [API Reference](api-reference.md). For business logic and limitations of matching, see the [Matching Engine](matching-engine.md).

## 6. Configuration and environment

`core/config.py` uses `pydantic-settings`, loads settings from `.env`, uses case-insensitive names, and ignores extra settings. `database.py` separately calls `load_dotenv()` and reads `DATABASE_URL` directly, so database setup and application settings have two access paths.

Important configuration names identified in the source include:

| Variable                                                                 | Purpose / observed default                                                                                                                  |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `DATABASE_URL`                                                           | Required database connection string.                                                                                                        |
| `JWT_SECRET`                                                             | Required secret; minimum length is configured as 32 characters. Never commit it.                                                            |
| `CORS_ALLOW_ORIGINS`                                                     | Required allowed-origin configuration.                                                                                                      |
| `APP_BASE_URL`                                                           | Required application base URL.                                                                                                              |
| `ENV`                                                                    | Environment such as `dev`, `test`, `staging`, or `prod`.                                                                                    |
| `DEBUG`                                                                  | Debug flag; observed default `true`. Review before production.                                                                              |
| `APP_NAME`                                                               | Observed default `KNode`.                                                                                                                   |
| `API_V1_PREFIX`                                                          | Observed default `/api/v1`; router prefixes are not fully uniform.                                                                          |
| `JWT_ALGORITHM`                                                          | Observed default `HS256`.                                                                                                                   |
| `ACCESS_TOKEN_TTL_SECONDS`                                               | Observed default 900 seconds.                                                                                                               |
| `REFRESH_TOKEN_TTL_SECONDS`                                              | Observed default 43200 seconds.                                                                                                             |
| `OTP_TTL_SECONDS`                                                        | Observed default 600 seconds.                                                                                                               |
| `OTP_LENGTH`                                                             | Observed default 6.                                                                                                                         |
| `OTP_MAX_ATTEMPTS`                                                       | Observed default 5.                                                                                                                         |
| `LOGIN_MAX_ATTEMPTS`                                                     | Observed default 5.                                                                                                                         |
| `LOGIN_LOCKOUT_SECONDS`                                                  | Observed default 900 seconds.                                                                                                               |
| `PASSWORD_MIN_LENGTH`                                                    | Observed default 12.                                                                                                                        |
| `REFRESH_COOKIE_NAME`                                                    | Observed default `knode_refresh`.                                                                                                           |
| `REFRESH_COOKIE_SECURE`                                                  | Observed default `true`; production transport should be verified.                                                                           |
| `REFRESH_COOKIE_SAMESITE`                                                | Observed default `lax`.                                                                                                                     |
| `REFRESH_COOKIE_DOMAIN`                                                  | Optional cookie domain.                                                                                                                     |
| `EMAIL_BACKEND`                                                          | Observed options include SMTP and Resend; verify the active implementation.                                                                 |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_FROM_NAME` | Email delivery configuration.                                                                                                               |
| `REFERRAL_DISCLAIMER_VERSION`                                            | Observed default `v1`.                                                                                                                      |
| `GCP_PROJECT_ID`, `GCP_REGION`                                           | Google Cloud / Vertex AI configuration; observed region `africa-south1`.                                                                    |
| `GEMINI_API_KEY`, `GEMINI_MODEL`                                         | Gemini provider configuration; observed model default `gemini-3.8-flash` in settings. Confirm the actual supported model and provider path. |
| `AI_ASSESSMENT_ENABLED`                                                  | Observed default `false`.                                                                                                                   |
| `AI_BULK_ASSESSMENT_ENABLED`                                             | Observed default `false`.                                                                                                                   |
| `GEMINI_TIMEOUT_SECONDS`, `GEMINI_MAX_RETRIES`, `AI_MAX_TEXT_LENGTH`     | AI timeout, retry and text-size controls; observed defaults include 30 seconds, 2 retries and 20000 characters.                             |

Never put actual credentials in documentation, examples, screenshots, logs, or version control. Use placeholders in examples and supply secrets through an appropriate local or deployment secret mechanism.

## 7. Startup and local development

The application entry point is `main.py`, and the FastAPI app is titled `KNode`, version `0.1.0`. The public `GET /health` route returns `{"status":"ok"}`. FastAPI's default interactive documentation is available at `/docs`, and the OpenAPI schema at `/openapi.json` when enabled by the application.

The repository README documents these commands:

```bash
uv venv
uv pip install -r pyproject.toml --group dev
uv run --no-sync uvicorn main:app --reload
```

The commands should be verified in a clean environment before being treated as definitive. See [Testing and Operations](testing.md) for the complete local setup and verification workflow.

## 8. AI integrations and external services

External-service use varies by feature and configuration:

| Integration                     | Code area                                                            | Configuration / limitations                                                                                                                |
| ------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Profile/opportunity extraction  | `services/ai_service.py`                                             | Uses `GEMINI_API_KEY` or a GCP project/region path depending on the module. External calls were not verified in a live environment.        |
| Document extraction             | `services/document_extraction.py`                                    | Defines `ExtractionUnavailable` (503) and `ExtractionFailed` (502). Review supported MIME types and conversion branches in code.           |
| Matching assessment             | `services/matching_engine/gemini_provider.py`, `provider_factory.py` | Depends on assessment enablement, Gemini configuration, timeout and retry settings. Provider failures are translated to assessment errors. |
| Email                           | `services/email_service.py`                                          | SMTP configuration; uses STARTTLS. Missing credentials are treated differently in development and production.                              |
| Crawling and scraping           | `services/jobs/`, `services/scraper_service.py`                      | Source permissions, scheduler and live external-source behaviour require verification.                                                     |
| Vertex AI embeddings / pgvector | No matching implementation was found in the inspected matching path  | Do not describe embeddings, vector search, or pgvector-based retrieval as implemented in that path without new code evidence.              |

Treat source documents, web content, and extracted text as untrusted. Model output must not grant access, bypass backend validation, or write arbitrary database values. Preserve source and claim IDs, validate quotes against stored evidence, and avoid logging raw sensitive documents or secrets.

## 9. Background jobs

Relevant modules include `services/jobs/opportunity_matching.py`, `process_grants.py`, `funder_crawler.py`, `midnight_grants.py`, `midnight_scraper.py`, `nightly.py`, `scraper.py`, `services/jobs/base.py`, and `services/matching_batch_service.py`.

`opportunity_matching.py` provides a CLI entry point that opens a database session, runs `run_opportunity_matching_batch`, logs a summary, and exits through `run_job`. The batch service loads open opportunities and organisations, applies the configured provider policy, and iterates through candidates. A production scheduler/orchestrator was **not verified** in the inspected source.

Before running jobs in a shared environment, confirm the target database, job locks, failure recovery, provider-cost limits, retry behaviour, duplicate-execution safety, sensitive logging, and run-outcome recording.

## 10. Errors and observability

Pydantic and FastAPI handle request validation, services apply business checks, and database constraints provide an additional integrity boundary. The codebase contains both `core/error_handlers.py` and `core/errors.py`; `main.py` registration order may cause one `AppError` handler to replace another. Verify the effective handler and standardise the response contract.

Logging is configured in `main.py` through `configure_logging`, with `DEBUG` in development and `INFO` otherwise. Correlation-ID middleware is installed. The unhandled exception handler logs the exception and request path. The matching provider logs prompt version, attempt count, status, elapsed time and failure class rather than the raw prompt/evidence. The email service logs whether email was sent or skipped. Audit-event models and service modules exist, but complete audit coverage is not established.

Not verified: metrics backend, distributed tracing exporter, alerting, production log aggregation, database performance dashboard, complete sensitive-action audit coverage, or a database-readiness endpoint. `/health` returns a constant status object and is not a database-readiness check.

## 11. Deployment and operations

Before release:

1. Build from a clean dependency resolution.
2. Configure secrets through the deployment platform.
3. Run Alembic migrations as a controlled release step.
4. Verify the application start command and environment.
5. Confirm `/health` responds after deployment.
6. Smoke-test authentication, organisation/opportunity access and matching.
7. Schedule batch jobs only if intended and ensure locks/retries are safe.
8. Verify SMTP and AI credentials, network access, quotas and provider failure behaviour.
9. Confirm backup/restore, log retention and incident-response procedures.

## 12. Developer maintenance

When adding an endpoint, use the existing router and prefix convention, define request/response schemas, keep business logic in services, enforce resource-level authorisation, add repository methods where appropriate, test success and failure paths, and confirm the route appears at the intended path in `/openapi.json`.

When changing a database field, update the SQLAlchemy model, create and review an Alembic migration, test it on disposable PostgreSQL, update schemas and service logic, and add persistence/API tests. `Base.metadata.create_all()` in tests is not a substitute for testing Alembic migrations.

For matching changes, define status semantics and evidence requirements first, keep provider access behind the assessment-provider interface, preserve claim/source IDs and quotes, validate provider output, keep eligibility separate from capability scoring, add labelled evaluation cases, record versions where supported, and verify authorisation before retrieving or returning organisation/collective information.

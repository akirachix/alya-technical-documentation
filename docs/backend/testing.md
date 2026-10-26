# Testing and Operations

## 1. Test suite overview

The inspected repository contains tests for API routes, services, repositories, authentication, matching, collectives, evidence, audit, AI, email and error handlers. The extracted tree contained 71 files matching the `test*.py` naming pattern. This is a file count, not a test-case count and not evidence that the tests pass.

Key test support files include:

- `tests/conftest.py`
- `tests/factories.py`
- `tests/helpers.py`
- Matching-specific test helpers and feature test modules.

## 2. Test database and fixtures

`tests/conftest.py` creates an in-memory SQLite engine, enables foreign keys, calls `Base.metadata.create_all`, and provides a rollback transaction fixture. API tests use FastAPI `TestClient` and override `get_db`. Authentication stores are cleared around tests.

SQLite fixtures are useful for fast unit/API tests, but do not prove PostgreSQL UUID, enum, JSON, migration DDL, locking, concurrent transactions or query behaviour. Maintain a separate disposable PostgreSQL integration-test workflow for migration and database-specific behaviour.

## 3. Commands

The README/project configuration documents the following commands. Confirm the required environment variables are available before running the suite; the audit environment could not import the test suite without `DATABASE_URL`.

```bash
# Full test suite with coverage
uv run --no-sync pytest --cov --cov-report=term-missing

# Individual test module (replace with an existing test filename)
uv run --no-sync pytest tests/test_matching_engine.py -v

# Verbose full suite
uv run --no-sync pytest -vv

# Ruff lint
uv run --no-sync ruff check .

# Check formatting without changing files
uv run --no-sync ruff format --check .

# Apply Ruff formatting
uv run --no-sync ruff format .
```

To inspect actual test filenames:

```bash
find tests -maxdepth 2 -type f -name 'test*.py'
```

The sample module name is illustrative only; use an existing filename in the checkout.

## 4. Recommended coverage matrix

| Area | Tests to verify or add |
|---|---|
| Authentication | Registration/login success and failure, email/login verification, expired/invalid tokens, refresh/logout, inactive users, lockout and OTP attempt limits. |
| Authorization | Cross-organisation reads/writes, cross-funder match reads, admin scope, body/query ID attacks, evidence access, collective membership and workspace access. |
| API contract | OpenAPI schema, response-model serialization, expected status codes, consistent error payloads and duplicate coverage route behaviour. |
| Matching statuses | `MET`, `PARTIALLY_MET`, `NOT_MET`, `UNKNOWN`, `NOT_CHECKED`, `ERROR`, and legacy status mapping. |
| Evidence grounding | Exact/sub-string quote, wrong claim ID, quote from another organisation, empty evidence and unconfirmed claim. |
| Retrieval | No overlap, ties/order, token normalisation, result limit and equivalent requirements phrased differently. |
| Scoring | No scorable requirements, unknown category, partial published weights, duplicate/conflicting dedupe keys, missing evidence references and rounding. |
| Eligibility | Passed/current/future deadlines, geography overlap/no overlap, organisation-type unknown and empty conditions. |
| Partner suggestions | Candidate covers gap, no candidate, participation remains unconfirmed and evidence references are preserved. |
| Collective coverage | Accepted vs proposed member, covered/partial/unresolved/uncovered states, duplicate claim IDs and unconfirmed participants. |
| PostgreSQL | Migration upgrade/downgrade on a disposable database; JSON, enum, UUID, FK/check constraints and concurrent transactions. |
| AI integration | Provider disabled, missing key, timeout, malformed JSON, retry handling, provider failure, prompt injection and cost limits. |
| Email | Missing credentials in development/production, SMTP auth/TLS failure and no credential logging. |
| Database transactions | Rollback after service/provider failure, atomic persistence, duplicate requests and idempotency. |

## 5. Postman setup

Create a local Postman environment with:

- `baseUrl` — for example `http://127.0.0.1:8000`.
- `accessToken` — obtained from the real login/verification flow.
- `organisationId`.
- `opportunityId`.
- `matchId`.
- `collectiveId`.

Use a local test account and test database. Do not store production credentials in shared workspaces.

### Health check

```http
GET {{baseUrl}}/health
```

Expected source-level response:

```json
{ "status": "ok" }
```

The route returns a constant status object; it is not a database-readiness check.

### Current user

```http
GET {{baseUrl}}/api/v1/auth/me
Authorization: Bearer {{accessToken}}
```

Use the response schema in `schemas/auth.py`; do not assume exact response fields without checking the schema.

### Create match

```http
POST {{baseUrl}}/api/v1/matches
Authorization: Bearer {{accessToken}}
Content-Type: application/json
```

```json
{
  "organisation_id": "{{organisationId}}",
  "opportunity_id": "{{opportunityId}}"
}
```

Verify the exact request contract in `schemas/matching.py`.

### Read match

```http
GET {{baseUrl}}/api/v1/matches/{{matchId}}
Authorization: Bearer {{accessToken}}
```

### Partner suggestions

```http
GET {{baseUrl}}/api/v1/matches/{{matchId}}/partners
Authorization: Bearer {{accessToken}}
```

### Import OpenAPI

With the application running, retrieve `/openapi.json` and use Postman's OpenAPI import. Inspect the generated collection afterwards because generated requests may not fully describe authentication, examples, errors or edge cases.

## 6. Essential API scenarios

| Scenario | Expected check |
|---|---|
| Valid authenticated request | Correct schema/status and authorised resource. |
| Missing bearer token | 401-style response. |
| Malformed UUID | 422 validation response. |
| Nonexistent resource | Not-found response. |
| User from another organisation | Access denied or not-found according to route policy. |
| Funder requests unrelated opportunity match | No cross-funder data exposure. |
| Requirement has no evidence | `UNKNOWN`, not `NOT_MET`. |
| Evidence exists but AI provider is disabled | `NOT_CHECKED` in the requirement-assessment path where applicable. |
| AI provider fails | `ERROR` or documented safe fallback; never silently mark a requirement unmet. |
| AI quote is not in cited passage | Assessment downgraded to `UNKNOWN`. |
| Candidate organisation covers lead gap | “May complement” label; participation remains false. |
| Coverage route contract | Verify versioned matching coverage and unversioned deterministic coverage are both intentional. |
| PostgreSQL unavailable | Safe failure and no partial persisted result. |

## 7. Local development setup

### Prerequisites

- Python 3.11 or newer, as declared by project configuration.
- `uv`, as documented by the repository README.
- PostgreSQL for integration development.
- Git.
- Optional Google Gen AI credentials and SMTP credentials for external integration tests.

### Setup

```bash
git clone <trusted-repository-url>
cd alya-backend
uv venv
uv pip install -r pyproject.toml --group dev
```

Replace the repository URL with the trusted project URL. Create a local `.env` file and never commit it:

```dotenv
ENV=dev
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@localhost:5432/<DB_NAME>
JWT_SECRET=<LOCAL_SECRET_AT_LEAST_32_CHARACTERS>
CORS_ALLOW_ORIGINS=["http://localhost:3000"]
APP_BASE_URL=http://localhost:3000
AI_ASSESSMENT_ENABLED=false
AI_BULK_ASSESSMENT_ENABLED=false
```

Create the local database, then inspect and apply migrations:

```bash
uv run alembic heads
uv run alembic current
uv run alembic upgrade head
```

Start the API:

```bash
uv run --no-sync uvicorn main:app --reload
```

Then visit:

- `http://127.0.0.1:8000/docs`
- `http://127.0.0.1:8000/health`

Check the required environment variables before invoking pytest. Use a disposable PostgreSQL database for migration and database-specific tests.

## 8. Seed data and development databases

The archive includes test factories and matching helper modules. Matching-related services can create match records from stored organisation/opportunity/requirement/evidence records. A standalone seed script with a verified public command was not established in the inspected repository tree. Do not assume `seed_matching_demo.py` exists in this archive.

Use test factories for repeatable automated test data. For manual local data, use the API or a project-owned seed script only after checking its database target, access scope and idempotency. Record whether the workflow writes to PostgreSQL, SQLite or another target, and check that rerunning it does not create duplicates.

## 9. Logging and operational verification

The application configures logging in `main.py`, installs correlation-ID middleware, and logs unhandled exceptions. The matching provider logs prompt version, attempt count, status, elapsed time and failure class rather than raw evidence prompts. The email service logs whether email was sent or skipped.

The following were not verified in the inspected source:

- Metrics backend or distributed tracing exporter.
- Alerting integration or production log aggregation.
- Database performance dashboard.
- Complete audit coverage for sensitive actions.
- Dedicated readiness endpoint that verifies database/provider connectivity.

Do not treat `/health` as a readiness check for the database or external providers.

## 10. Deployment checklist

The repository includes `Procfile` and GitHub workflows named `ci.yml`, `pr-checks.yml` and `deploy.yml`. The README describes a Heroku workflow, but no deployed instance was inspected. Before release:

1. Build from a clean dependency resolution.
2. Configure secrets through the deployment platform's secret mechanism.
3. Run Alembic migrations as a controlled release step.
4. Verify the app process and start command.
5. Confirm `/health` responds after deployment.
6. Run smoke tests for authentication, organisation/opportunity access and matching.
7. Schedule batch jobs only if intended; verify locks, retries and duplicate execution safety.
8. Verify SMTP and AI credentials, network access, quotas and provider failure handling.
9. Confirm backup/restore, log retention, monitoring and incident-response procedures.

## 11. Developer change checklist

### Add an endpoint

1. Find the existing router for the functional area.
2. Follow the router's existing path-prefix convention.
3. Add request/response schemas if existing contracts do not fit.
4. Put business logic in the service layer.
5. Add authentication and resource-level authorization explicitly.
6. Add repository methods where they fit the persistence pattern.
7. Test success, invalid input, missing resources and cross-tenant access.
8. Confirm the route appears at the intended full path in `/openapi.json`.

### Change a database field

1. Update the SQLAlchemy model.
2. Generate an Alembic migration.
3. Review nullability, defaults, constraints, data migration and downgrade behaviour.
4. Apply it to disposable PostgreSQL.
5. Update schemas and service logic.
6. Add persistence and API tests.
7. Do not rely on `Base.metadata.create_all()` as a substitute for migration testing.

### Extend matching behaviour

1. Define status semantics and evidence requirements before changing logic.
2. Keep pure logic in `services/matching_engine/` where possible.
3. Keep provider access behind `AssessmentProvider` so it can be mocked.
4. Preserve claim IDs, source IDs, quotes and status notes.
5. Validate provider output and verify quotes against cited passages.
6. Keep eligibility separate from capability score calculation.
7. Test missing/contradictory evidence, invalid output, provider failure, duplicate requirements and unknown categories.
8. Add labelled evaluation cases before changing prompts, model configuration or retrieval.
9. Record model/prompt/algorithm versions where the schema supports it.
10. Verify access control before retrieving or returning partner and collective information.

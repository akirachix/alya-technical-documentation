# Database and Migrations

## 1. Overview

KNode uses SQLAlchemy models and a PostgreSQL database configured through `DATABASE_URL`. The database stores users, organisations, funders, opportunities, requirements, evidence sources and claims, matching runs and results, collectives, referrals, workspaces, reports, and audit events.

## 2. Connection and sessions

`database.py` calls `load_dotenv()`, reads `DATABASE_URL`, raises an error if it is absent, and normalises `postgres://` and `postgresql://` URLs to `postgresql+psycopg2://`. The SQLAlchemy engine uses `pool_pre_ping=True` and `echo=False`. `SessionLocal` uses `autocommit=False` and `autoflush=False`. `get_db()` yields a session and closes it in a `finally` block.

Example DSN (replace every placeholder locally):

```text
postgresql://<DB_USER>:<DB_PASSWORD>@localhost:5432/<DB_NAME>
```

Closing a session does not automatically commit a transaction. Transaction boundaries differ by service; inspect `repositories/transaction.py` and relevant service/repository code, especially rollback behaviour after exceptions.

Tests use an in-memory SQLite database with `sqlite://` and `StaticPool`, enable foreign keys, call `Base.metadata.create_all`, and provide a rollback transaction fixture.

## 3. Main entities

| Table / model                | Purpose and key relationships                                                                                                             |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `users`                      | User accounts, role, status and optional organisation/funder association.                                                                 |
| `organisations`              | Organisation identity, mission, focus areas, countries of operation, capabilities and verification metadata.                              |
| `funders`                    | Funder profile, funding interests, geography and typical grant range.                                                                     |
| `funding_opportunities`      | Opportunity details, funder/source identifiers, eligibility information, coalition policy, deadlines and status.                          |
| `opportunity_requirements`   | Individual opportunity requirements and optional structured values/source association.                                                    |
| `sources`                    | Evidence provenance, source URL/type, retrieval/publication information, content/storage references and hash.                             |
| `extracted_claims`           | Claims extracted from sources, evidence text, confidence, review status and reviewer metadata.                                            |
| `match_runs`                 | Matching workflow lifecycle, requester/target references, run type, status, algorithm version and errors.                                 |
| `matches`                    | Match identity, organisation/opportunity association, score, score breakdown, requirement assessments and explanation.                    |
| `match_partners`             | Partner suggestions associated with matching results.                                                                                     |
| `collectives`                | Collaboration group and opportunity context.                                                                                              |
| `collective_members`         | Membership, participation and response state for collectives.                                                                             |
| `funding_awards`             | Award records used by funding/trend-related functionality.                                                                                |
| `organisation_email_domains` | Organisation email-domain associations.                                                                                                   |
| `ai_files`                   | AI/document file records; the polymorphic `entity_type` + `entity_id` association does not use a normal foreign key to every entity type. |
| `reports`                    | Stored report content, including JSON content. A public report router was not found in the inspected source.                              |
| `audit_events`               | Audit event data. The ORM attribute `event_metadata` maps to the database column `metadata`.                                              |
| `referrals`                  | Referral records targeting an organisation or collective. A check constraint requires exactly one target.                                 |
| `workspaces`                 | Workspace associated with a collective.                                                                                                   |
| `workspace_items`            | Items within a workspace, with optional assignee/author relationships using `SET NULL` where configured.                                  |

## 4. Key column-level data dictionary

### `users`

| Field                     | Observed type / constraint | Meaning                                                 |
| ------------------------- | -------------------------- | ------------------------------------------------------- |
| `user_id`                 | GUID/UUID primary key      | User identifier.                                        |
| `organisation_id`         | Nullable foreign key       | Associated organisation, when applicable.               |
| `funder_id`               | Nullable foreign key       | Associated funder, when applicable.                     |
| `first_name`, `last_name` | `String(100)`              | User name fields.                                       |
| `email`                   | `Text`, unique             | Login/contact email.                                    |
| `password_hash`           | `String(255)`              | Argon2 password hash; never store a plaintext password. |
| `role`                    | Enum `UserRole`            | Includes `ORGANISATION`, `FUNDER`, `ADMIN`.             |
| `status`                  | Enum `UserStatus`          | Includes `ACTIVE`, `INACTIVE`, `SUSPENDED`.             |
| timestamps                | DateTime                   | Creation/update lifecycle.                              |

### `organisations`

| Field                                                   | Observed type / constraint               | Meaning                                                |
| ------------------------------------------------------- | ---------------------------------------- | ------------------------------------------------------ |
| `organisation_id`                                       | GUID/UUID primary key                    | Organisation identifier.                               |
| `legal_name`                                            | `String(255)`, required                  | Legal organisation name.                               |
| `display_name`                                          | Nullable string                          | Display name.                                          |
| `ein`                                                   | `String(9)`, nullable unique             | External registration identifier in the current model. |
| `website_url`, `mission`                                | Nullable `Text`                          | Website and mission statement.                         |
| `focus_areas`, `countries_of_operation`, `capabilities` | Non-null generic JSON with list defaults | Organisation focus, geography and capability values.   |
| `registration_status`                                   | `String(100)`                            | Registration status text.                              |
| `annual_revenue`                                        | `Numeric(19,4)`                          | Revenue amount.                                        |
| `revenue_currency`                                      | `CHAR(3)`                                | Currency code.                                         |
| timestamps, `last_verified_at`                          | DateTime                                 | Lifecycle and verification metadata.                   |

### `funders`

| Field                                    | Observed type / constraint | Meaning                              |
| ---------------------------------------- | -------------------------- | ------------------------------------ |
| `funder_id`                              | GUID/UUID primary key      | Funder identifier.                   |
| `name`                                   | `String(255)`              | Funder name.                         |
| website/profile summary                  | Nullable `Text`            | Funder description and web presence. |
| `funder_type`                            | Enum                       | Funder classification.               |
| `focus_areas`, `geographic_focus`        | Generic JSON               | Funding themes and geographies.      |
| `typical_grant_min`, `typical_grant_max` | `Numeric(19,4)`            | Typical grant range.                 |
| `grant_currency`                         | `CHAR(3)`                  | Currency code.                       |
| timestamps, `last_verified_at`           | DateTime                   | Lifecycle and verification metadata. |

### `funding_opportunities`

| Field                                                 | Observed type / constraint                    | Meaning                                           |
| ----------------------------------------------------- | --------------------------------------------- | ------------------------------------------------- |
| `opportunity_id`                                      | GUID/UUID primary key                         | Opportunity identifier.                           |
| `funder_id`                                           | Nullable foreign key                          | Associated funder.                                |
| `source_system`, `external_id`                        | `String(50)`, `String(255)`; composite unique | External source identity.                         |
| `title`                                               | `String(500)`                                 | Opportunity title.                                |
| `description`, `source_url`                           | `Text`                                        | Description and source link.                      |
| `sector`                                              | `String(100)`                                 | Sector/category.                                  |
| `coalition_policy`                                    | Enum, default `UNSPECIFIED`                   | Coalition policy.                                 |
| `eligible_geographies`, `eligible_organisation_types` | Generic JSON                                  | Eligibility metadata.                             |
| `grant_min`, `grant_max`                              | `Numeric(19,4)`                               | Grant range.                                      |
| `currency`                                            | `CHAR(3)`                                     | Grant currency.                                   |
| `deadline_at`                                         | Timezone-aware DateTime                       | Deadline.                                         |
| `status`                                              | Enum                                          | Includes `OPEN`, `CLOSED`, `UPCOMING`, `UNKNOWN`. |
| timestamps                                            | DateTime                                      | Lifecycle metadata.                               |

### `opportunity_requirements`

| Field              | Observed type / constraint     | Meaning                               |
| ------------------ | ------------------------------ | ------------------------------------- |
| `requirement_id`   | GUID/UUID primary key          | Requirement identifier.               |
| `opportunity_id`   | Foreign key, cascade on delete | Parent opportunity.                   |
| `requirement_type` | `String(100)`                  | Requirement category.                 |
| `requirement_text` | `Text`                         | Requirement wording.                  |
| `is_mandatory`     | Boolean, default true          | Whether the requirement is mandatory. |
| `structured_value` | Generic JSON                   | Optional structured interpretation.   |
| `source_id`        | Nullable foreign key           | Supporting source, when present.      |
| `created_at`       | DateTime                       | Creation timestamp.                   |

### `sources` and `extracted_claims`

` sources` records source associations with organisations, funders, or opportunities and includes source name/URL/type, storage key, retrieval/publication metadata, content text and hash fields. Exact nullability and foreign-key rules should be verified against models and migrations.

`extracted_claims` stores claim type/value, `evidence_text`, `confidence_score` (observed `Numeric(5,2)`), review status (default `PENDING`), reviewer and review timestamp, source/organisation association and creation time. The matching assessment path accepts only claims with `review_status == "CONFIRMED"` and non-empty evidence text.

### `match_runs` and `matches`

`match_runs` stores `run_id`, optional requester/organisation/opportunity references, run type (opportunity matching, partner discovery, collective building), lifecycle status (`PENDING`, `RUNNING`, `COMPLETED`, `FAILED`), `algorithm_version`, start/completion timestamps, error details and creation time.

`matches` stores `match_id`, run and organisation references, optional opportunity, match type, nullable numeric score, `score_breakdown` JSON, `requirement_assessments` JSON, explanation and creation time. The response builder also reads missing-information fields, strengths/gaps, score status, evidence coverage, eligibility status, run status and algorithm version. It detects legacy placeholder records so a placeholder zero is not presented as a real score.

### Other constraints and relationships

- `match_partners` stores partner suggestions associated with matching results.
- `collectives` and `collective_members` represent groups and their member participation. Membership uniqueness and response timestamps are constrained in the schema/migrations.
- `funding_awards` supports award and trend-related data.
- `organisation_email_domains` stores organisation-domain associations.
- `ai_files` uses a polymorphic entity association rather than a foreign key covering every target type.
- `reports` stores report content as JSON; the inspected code did not establish a public report router.
- `audit_events` stores audit metadata; the Python attribute `event_metadata` maps to the SQL column `metadata`.
- `referrals` has a check constraint requiring exactly one of organisation or collective target.
- `workspaces` and `workspace_items` associate work items with collectives/workspaces; optional author/assignee references may use `SET NULL`.

## 5. Types, constraints and implementation notes

- A custom GUID/UUID type and UUID factory are defined in `models/types.py`; verify behaviour on both SQLite and PostgreSQL.
- Several fields use SQLAlchemy generic `JSON`, not PostgreSQL `JSONB`. Identified JSON fields include `focus_areas`, `capabilities`, `structured_value`, `score_breakdown`, `requirement_assessments`, `report_content`, and audit metadata.
- Many enums use `native_enum=False`.
- DateTime fields are timezone-aware in the model declarations and may use Python/server defaults.
- Grant and revenue values use `Numeric(19,4)`; score/confidence values use their own declared precisions.
- Constraints include external identifiers, collective membership uniqueness, referral target exclusivity, collective response timestamps and funding-award rules.
- Do not infer the live schema solely from SQLAlchemy models. Inspect PostgreSQL catalogs and Alembic state before making production changes.

## 6. Alembic migrations

Migration files live under `alembic/versions/`; configuration is in `alembic.ini` and `alembic/env.py`. The history includes more than one initial-schema migration, a later no-op-looking initial migration, a merge revision, nullable match-score work, organisation-email-domain changes, collective/referral constraints, sector and coalition-policy changes. The noted latest migration is `b4f783d2d110_add_coalition_policy_to_opportunities.py`, which merges two branches. Verify current heads/history rather than assuming this is still the latest revision.

Run against the intended database only:

```bash
uv run alembic heads
uv run alembic history --verbose
uv run alembic current
```

Generate and apply migrations only after review:

```bash
uv run alembic revision --autogenerate -m "describe_schema_change"
uv run alembic upgrade head
```

A downgrade can be destructive; inspect the revision's `downgrade()` implementation before running:

```bash
uv run alembic downgrade -1
```

### Migration workflow

1. Update the SQLAlchemy model.
2. Generate a candidate migration.
3. Review the migration for nullability, defaults, constraints, indexes, data backfills and downgrade behaviour.
4. Apply it to a disposable PostgreSQL database.
5. Verify the resulting schema and existing data.
6. Update Pydantic schemas and service logic.
7. Add migration, repository and API tests.

`Base.metadata.create_all()` in the SQLite test setup is not a substitute for validating Alembic migrations. Keep model and migration history aligned and resolve multiple-head conditions deliberately.

## 7. Safe read-only inspection queries

Run these only against a database you are authorised to inspect. The queries are read-only and use model table names:

```sql
SELECT organisation_id, legal_name, display_name, registration_status
FROM organisations
ORDER BY legal_name
LIMIT 50;

SELECT opportunity_id, title, status, deadline_at, coalition_policy
FROM funding_opportunities
ORDER BY created_at DESC
LIMIT 50;

SELECT collective_id, name, opportunity_id, created_at
FROM collectives
ORDER BY created_at DESC
LIMIT 50;

SELECT run_id, status, run_type, algorithm_version, created_at
FROM match_runs
ORDER BY created_at DESC
LIMIT 50;

SELECT match_id, organisation_id, opportunity_id, score, created_at
FROM matches
ORDER BY created_at DESC
LIMIT 50;
```

For any seed workflow, establish whether it writes to PostgreSQL, SQLite or another target, and whether rerunning it creates duplicates.

# KNode Master Glossary

Last updated: 26 October 2026

## Contents

- [About this glossary](#about-this-glossary)
- Master glossary
  - [A. General software engineering](#a-general-software-engineering)
  - [B. Frontend and user interface](#b-frontend-and-user-interface)
  - [C. Backend and API development](#c-backend-and-api-development)
  - [D. Database and data modelling](#d-database-and-data-modelling)
  - [E. Authentication, authorisation and security](#e-authentication-authorisation-and-security)
  - [F. AI, evidence and matching](#f-ai-evidence-and-matching)
  - [G. Funding and nonprofit domain](#g-funding-and-nonprofit-domain)
  - [H. Collectives, workspaces and collaboration](#h-collectives-workspaces-and-collaboration)
  - [I. Data ingestion, evidence and reporting](#i-data-ingestion-evidence-and-reporting)
  - [J. Testing and quality assurance](#j-testing-and-quality-assurance)
  - [K. Infrastructure, deployment and operations](#k-infrastructure-deployment-and-operations)
- [Acronym and abbreviation index](#acronym-and-abbreviation-index)

## About this glossary

This glossary defines the terms used across the KNode technical documentation, grouped by system area. It also contains an abbreviation index.

**Sources.** Terms come from the Architecture, Backend (index, Database, API Reference, Authentication and Security, Matching Engine, Testing), AI Service, Frontend Web and Integrations pages of the `alya-technical-documentation` repository (`main` branch). The Overview page was an empty stub when these sources were read. No source code, ERD, PRD, MVP document, OpenAPI file or Figma file was available, so project-specific claims are checked against the documentation, not against code. See [Evidence basis and limitations](#evidence-basis-and-limitations).

**Status labels in the "Context in KNode" column.** The documentation mixes target design with inspected code, so entries that describe KNode-specific behaviour say which it is:

| Label | Meaning |
|---|---|
| **Implemented** | The Backend, AI Service, Frontend or Integrations pages describe it as present in inspected code. |
| **Design only** | Only the Architecture page describes it; the Backend or Frontend pages do not confirm it. |
| **Not found** | A code-based page states it was looked for and not found. Do not describe it as built. |
| **Unverified** | The documentation itself says the behaviour needs checking. |

Entries without a label define a general concept and note how KNode uses it.

## Master glossary

Entries are alphabetical within each category. Where a concept has both a database table entry (section D2) and a domain entry, the table entry describes the stored record and the domain entry describes the concept.

### A. General software engineering

| Term | Definition | Context in KNode |
|---|---|---|
| API (Application Programming Interface) | A defined interface through which one piece of software requests data or actions from another. | **Implemented.** The Next.js frontend calls the FastAPI backend's JSON endpoints under the `/api/v1` prefix and a few unversioned paths. See also *Endpoint*, *OpenAPI*. |
| Build | The step that turns source code into something runnable, such as compiling TypeScript or bundling a web app. | **Implemented.** Frontend: `npm run build`, then `npm start`. Backend: dependencies are installed with uv; there is no compile step. |
| Client-server architecture | A design where a client (here a browser) sends requests to a server that holds the data and rules. | **Implemented.** The browser frontend talks only to the KNode backend; third-party services sit behind the backend, never behind the browser. |
| Component | A self-contained, reusable piece of a system. In the frontend it means a React UI element; in the architecture it means a major function such as Matching Engine or Referrals. | **Implemented.** The Architecture page lists 14 system components. The Frontend page groups UI components into ui, features, layout and auth. |
| Configuration | Settings that change how an application behaves without changing its code. | **Implemented.** Backend settings load from `.env` via pydantic-settings; frontend settings are `NEXT_PUBLIC_*` variables. See *Environment variable*. |
| Correlation ID | A unique identifier attached to a request so every log line and error response for that request can be linked. | **Implemented.** Backend middleware installs it; the documentation says it is not yet returned consistently. |
| Dependency | An external library or package that code needs in order to run. Also, in FastAPI, a function that supplies a value to a route (see *Dependency injection*). | **Implemented.** Python dependencies are declared in `pyproject.toml` and locked in `uv.lock`; npm packages in `package.json`. The Backend page flags the httpx2 entry for verification. |
| Environment | A separate running copy of the system with its own database, keys and settings. | **Implemented** as the ENV setting (dev, test, staging, prod), which changes email and cookie behaviour. The Architecture page describes separate dev, staging and production stacks (**Design only**). |
| Integration | A connection between KNode and a third-party service or data source. | **Implemented.** Documented integrations: Google Gemini, SMTP email, PostgreSQL, USAspending, Grants.gov, IRS data, ProPublica, Candid, GlobalGiving. See the Integrations page. |
| Layered architecture | Organising code into layers that each have one job and call only the layer beneath. | **Implemented.** Backend: routers, then Pydantic schemas, services, repositories, SQLAlchemy models, with core modules for config, security and errors. Frontend: pages, feature components, UI components, then lib. The frontend page notes its boundaries are convention, not enforced. |
| Middleware | Code that runs on every request before or after the route handler. | **Implemented.** Correlation-ID middleware and CORS are registered in `main.py`. The frontend has no Next.js middleware. |
| Module | A file or folder of related code with a clear responsibility. Also used in the Architecture page for a product capability. | **Implemented.** For example `services/matching_engine/` is a backend module. |
| Repository (data-access layer) | A class or module that wraps database queries so business code does not write them directly. | **Implemented.** The backend repositories/ folder, for example `matching_repository.py`. Not the same as a source-control repository. |
| Repository (source control) | A version-controlled project folder, such as a Git repository. | The documentation refers to the alya-technical-documentation, alya-frontend-develop and backend repositories. |
| Runtime | The environment in which code executes, or the period while it executes. | Backend: Python 3.11 or newer with Uvicorn. Frontend: Node.js with Next.js (version not pinned by an engines field). |
| Service (backend business-logic layer) | A module that implements a business workflow, applying rules and calling repositories and external providers. | **Implemented.** services/ holds `auth_service`, `matching_service`, `email_service` and others. Routers delegate to services. |
| Service (external) | A third-party system KNode calls, such as Gemini or an SMTP server. | See *Integration*. |
| uv | A Python package and environment manager. | **Implemented.** README commands: `uv venv`, `uv pip install -r pyproject.toml --group dev`, `uv run ...` |

### B. Frontend and user interface

| Term | Definition | Context in KNode |
|---|---|---|
| Absent / Integrated / Integrated, path mismatch / Placeholder / Static data | The five implementation-status labels used on the Frontend page. **Integrated**: the UI calls a backend endpoint that exists in the bundled OpenAPI snapshot. **Integrated, path mismatch**: it calls a path that differs from the snapshot. **Static data**: it uses data embedded in the frontend. **Placeholder**: a control or page exists but is disabled or hard-coded. **Absent**: no code found. | Project-specific labels. Use them only for the frontend; the status labels in this glossary (Implemented, Design only, and so on) are a different scheme. |
| `ApiError` / `authedRequest` | `authedRequest`<T>() is the frontend's main HTTP helper: it adds the bearer token, JSON headers and `cache: "no-store"`, then throws `ApiError`(message, status) on failure. | **Implemented** in `lib/api.ts`. Some opportunity and grant calls bypass it with raw fetch and the `NEXT_PUBLIC_API_URL` base. |
| App Router | The Next.js routing system in which folders under app/ become URL routes and `layout.tsx` files wrap them. | **Implemented.** Routes include `/auth/*`, `/ngo/*` and `/funder/*`. |
| AppShell | The shared frame (sidebar, top bar, page header) used by the NGO and funder areas. | **Implemented** in `components/layout/`. Below 64rem the sidebar becomes an off-canvas drawer. |
| Client-side rendering (CSR) | Building the page in the browser using JavaScript, with data fetched from the browser. | **Implemented.** Almost every page is a client component that calls the API directly. |
| Debounce | Waiting a short time after the last input before acting, to avoid a request per keystroke. | **Implemented**: 250 ms in `usePagedOpportunities` and `useAllGrants`. |
| Design tokens | Named values for colours, fonts and spacing that keep styling consistent. | **Implemented** as Tailwind `@theme` tokens in `app/globals.css`: navy-*, brand-*, canvas, surface, line, ink. Auth pages still use hard-coded hex values. |
| Draft autosave | Saving an unfinished form so it survives a refresh. | **Implemented** for the grant form in `localStorage` under `knode_grant_draft_v3`. The draft persists after logout. |
| Empty state | What a screen shows when a list has no items. | **Implemented** on several lists ("No matches yet", "No suggestions yet"). Coverage across pages was not exhaustively checked. |
| Error boundary | A framework mechanism that catches rendering errors and shows a fallback screen. In Next.js: `error.tsx` and `global-error.tsx`. | **Not found.** The Frontend page states none exist, so an uncaught render error shows the Next.js default. |
| Funder area / NGO area | The two role-specific parts of the frontend: `/funder/*` and `/ngo/*`. Both share one application shell. | **Implemented.** The NGO area serves ORGANISATION users; the funder area serves FUNDER users. |
| Hook | A React function (`useState`, `useEffect`, or a custom use... function) that adds state or behaviour to a component. | **Implemented.** Custom hooks: `usePagedOpportunities`, `useAllGrants`, `useMediaQuery`. |
| Layout | A Next.js file that wraps all pages beneath it. | `app/layout.tsx` sets the font and metadata. `app/ngo/layout.tsx` redirects to login when no user resolves. The funder layout has no guard. |
| Loading state | What the UI shows while data is being fetched. | **Implemented** as text ("Loading dashboard…") or flags. No skeletons and no `loading.tsx`. |
| Next.js | A React framework providing routing, layouts, rendering modes and builds. | **Implemented.** Declared version 16.4.1 using the App Router. |
| `NEXT_PUBLIC_` variable | A Next.js environment variable that is copied into the browser bundle at build time and is therefore public. | `NEXT_PUBLIC_API_BASE_URL` (authenticated calls) and `NEXT_PUBLIC_API_URL` (opportunities). Never put secrets in them. |
| Pagination (limit / offset) | Returning a long list in pages. limit is the page size; offset is how many items to skip. Some endpoints use page and `page_size` instead. | **Implemented.** Opportunities, matches, collectives and workspace items use `limit/offset`; funder organisation search and evidence use `page/page_size`. |
| Publish recovery | Logic that prevents duplicate records when a multi-request save fails part-way. | **Implemented** in `publishGrant`: it creates the opportunity, then each requirement; on failure it deletes the partial opportunity, and a stored id lets a retry update instead of duplicating. |
| React | A JavaScript library for building user interfaces from components. | **Implemented.** Declared version 19.3.0. |
| React Hook Form | A library for managing form state and validation. | **Implemented** for the grant form only; other forms use plain `useState`. |
| Recharts | A charting library for React. | **Implemented**: the trend `LineChart` on `/ngo/trends`. |
| Responsive design | Layouts that adapt to screen width. | **Implemented** with Tailwind breakpoints. No visual or responsive tests exist. |
| Route guard | Client-side code that redirects users who should not see a page. | **Implemented only for** `/ngo/*`, and it checks that a user exists, not their role. It controls what the interface shows; it does not protect data. |
| Routing | Mapping a URL to a page. | See *App Router*. |
| `sessionStorage` / `localStorage` | Browser storage. `sessionStorage` is cleared when the tab closes; `localStorage` persists until removed. | Tokens and the pending email are in `sessionStorage` (`knode_access_token`, `knode_refresh_token`, `knode_pending_email`); the grant draft is in `localStorage`. |
| State management | How an application stores and shares data that changes. | **Implemented** with React hooks and component state. No Redux, Zustand, SWR or React Query. An `AuthProvider` context exists but is unused. |
| Tailwind CSS | A utility-first CSS framework. | **Implemented**, version 4. Several features also use separate plain .css files. |
| TypeScript | JavaScript with static types. | **Implemented**, `strict: true`, path alias `@/*`. |
| Zod | A TypeScript schema-validation library. | **Implemented** for auth fields (`lib/validation.ts`) and the grant form schema. |

### C. Backend and API development

| Term | Definition | Context in KNode |
|---|---|---|
| API contract | The agreed shape of requests, responses, status codes and errors between client and server. | The live contract is the backend's `/openapi.json`. The frontend keeps a snapshot at `docs/openapi.json`; `openapiContract.test.ts` checks that snapshot, not the live backend. The Backend page says the endpoint inventory is a map, not the contract. |
| Business logic | The rules that decide what the system does, as opposed to plumbing such as HTTP or storage. | **Implemented** in backend services/ and the pure functions in `services/matching_engine/`. The Backend page says to keep it out of routers. |
| CORS (Cross-Origin Resource Sharing) | A browser rule that blocks a page from calling a different origin unless that server explicitly allows it. | **Implemented.** The backend allows only origins in `CORS_ALLOW_ORIGINS` (default `http://localhost:3000`). Production values must be set explicitly. |
| CRUD | Create, Read, Update, Delete: the four basic data operations. | Used for opportunities, requirements, organisations, funders, collectives and workspace items. Not every entity exposes all four. |
| Dependency injection | A pattern where a function declares what it needs and the framework supplies it. | **Implemented** through FastAPI Depends. `get_db` supplies the database session; `get_current_user`, `require_role` and the scope helpers supply the caller. |
| Endpoint / route | A URL path plus HTTP method that the server handles, such as POST `/api/v1/matches`. A route is the code definition; an endpoint is the resulting URL. | **Implemented.** The API Reference inventories them by feature. |
| Exception handler | Code that converts a raised error into a client-safe HTTP response. | **Implemented**, but registered in two places, `core/errors.py` and `core/error_handlers.py`. |
| FastAPI | A Python web framework that builds JSON APIs from typed functions and generates OpenAPI documentation. | **Implemented.** App title KNode, version 0.1.0, served by Uvicorn. |
| HTTP method | The verb of a request: GET (read), POST (create or act), PATCH (partial update), DELETE. | Used throughout. Some paths accept several methods (for example GET/POST `/collectives`). |
| JSON (JavaScript Object Notation) | A text format for structured data. | All API bodies, Gemini responses and several database columns. |
| OpenAPI | A standard machine-readable description of an HTTP API. | FastAPI generates it at `/openapi.json`, with an interactive page at `/docs`. Postman can import it. |
| Path / query parameter | A value in the URL path (`/matches/{match_id}`) or after the ? (?limit=20). | limit is 1 to 100 (default 20) and offset at least 0 on GET `/api/v1/matches`. |
| Pydantic model / schema | A Python class that declares fields and types, validates data against them, and can export a JSON schema. | **Implemented.** In schemas/ for requests and responses (MatchCreateRequest, MatchResponse). Also used to force Gemini output into a shape (ExtractedProfile, ExtractedEvidence). |
| REST API | An HTTP API organised around resources and standard methods. | KNode describes itself as exposing JSON HTTP endpoints. Route prefixes are not fully uniform. |
| Request schema / response schema | The Pydantic models that define what a client may send and what the server returns. | `response_model` shapes outputs, so internal fields are not leaked. |
| Router | A FastAPI object that groups related routes under one prefix. | **Implemented**: `routers/auth.py`, `matching.py`, `collectives.py` and so on. |
| Status code | A three-digit HTTP result. | 201 created; 401 bad token; 403 forbidden; 404 not found (sometimes used deliberately instead of 403); 409 conflict; 422 validation; 423 account locked; 429 too many OTP attempts; 502 extraction failed; 503 extraction unavailable; 500 unhandled. |
| Uvicorn | An ASGI server that runs FastAPI apps. | `uv run --no-sync uvicorn main:app --reload` for local runs. |
| Versioned route | A route under `/api/v1`. | Most routes are versioned; collectives, referrals, workspaces and evidence are documented as unversioned. |

### D. Database and data modelling

#### D1. General database concepts

| Term | Definition | Context in KNode |
|---|---|---|
| Alembic | A migration tool for SQLAlchemy that keeps a versioned history of schema changes. | **Implemented.** Files in `alembic/versions/`; commands `alembic heads`, `alembic current`, `alembic upgrade head`. The latest noted revision, `b4f783d2d110`, merges two branches; verify current heads. |
| Cascade / SET NULL | Rules for what happens to child rows when a parent row is deleted: CASCADE deletes them; SET NULL clears the reference. | `opportunity_requirements` cascade when their opportunity is deleted. `workspace_items` assignee and author references may use SET NULL. |
| Check constraint | A database rule that every row must satisfy a condition. | referrals requires exactly one target: an organisation or a collective. |
| Constraint | A rule enforced by the database: not null, unique, foreign key, check. | Provides an integrity boundary beyond Pydantic and service checks. |
| Entity / table / model | An entity is a thing the system stores. A table holds its rows; a SQLAlchemy model is the Python class mapped to the table. | The documentation uses the table name (organisations) and the model name (Organisation) interchangeably. |
| Enum (database) | A column limited to a fixed set of values. | Many enums use `native_enum=False`, stored as text rather than a PostgreSQL enum type. Examples: UserRole, UserStatus, opportunity status, `coalition_policy`. |
| Foreign key | A column that references another table's primary key, so the database can reject orphaned references. | Used throughout, for example `opportunity_requirements`.`opportunity_id`. `ai_files` is the exception (see *Polymorphic association*). |
| GUID / UUID | A 128-bit identifier that is practically unique without central coordination. | Primary keys such as `user_id` and `match_id`. A custom GUID type in `models/types.py` works on both PostgreSQL and SQLite; verify on both. |
| Index | A structure that speeds up lookups on a column. | Index definitions are not described in the documentation; the migration checklist says to review them. |
| JSON vs JSONB | JSON stores text; PostgreSQL JSONB stores a binary, indexable form. | **KNode uses SQLAlchemy generic JSON, not JSONB**, for `focus_areas`, capabilities, `structured_value`, `score_breakdown`, `requirement_assessments` and report content. |
| Migration | A versioned script that changes the database schema. | See *Alembic*. Run as a controlled release step; `create_all()` in tests is not a substitute. |
| `Numeric(19,4)` | An exact decimal with up to 19 digits and 4 after the point, used for money to avoid floating-point error. | Grant minimums and maximums, revenue. Currency is a separate `CHAR(3)` column. |
| ORM (Object-Relational Mapping) | A technique that maps Python classes to database tables so queries are written in Python. | **Implemented** with SQLAlchemy 2.x. |
| pgvector | A PostgreSQL extension for storing embeddings and searching by vector similarity. | **Design only.** The Architecture page lists it; the Backend page says no pgvector or embedding code was found in the matching path. |
| Polymorphic association | One table referencing several kinds of parent through a type column plus an id column, without a normal foreign key. | `ai_files.entity_type` + `entity_id`. The database cannot enforce that the target exists. |
| PostgreSQL | An open-source relational database. | **Implemented.** The system of record, configured with `DATABASE_URL`. |
| Primary key | The column that uniquely identifies a row. | A UUID per table, such as `organisation_id`. |
| Referential integrity | The guarantee that references between rows stay valid. | Enforced by foreign keys; SQLite tests enable foreign keys explicitly. |
| Relationship | A link between two tables. One-to-many: one parent row, many child rows (an opportunity has many requirements). Many-to-many: rows on both sides can link to many on the other, through a join table. | `collective_members` is the membership link between collectives and members. The documentation does not name the member column; the coverage code uses Organisation, so organisations are the members (verify). |
| Schema (database) vs schema (Pydantic) | A database schema is the set of tables and columns. A Pydantic schema is an API request or response shape. | Context decides which is meant. The Backend page treats the live database schema and the models as potentially different. |
| Session (SQLAlchemy) | A unit of work holding database connections and pending changes. | `SessionLocal` has autocommit=False and autoflush=False; `get_db()` closes it in a finally block. Closing does not commit. |
| SQLAlchemy | The Python ORM and SQL toolkit. | **Implemented**, version 2.x. |
| SQLite | A file or in-memory database. | **Implemented for tests only**, with StaticPool. It does not prove PostgreSQL behaviour. |
| Timestamp | A date-time value. | Models declare timezone-aware DateTime columns. `deadline_at` is explicitly timezone-aware. |
| Transaction | A group of database changes that all succeed or all roll back. | Boundaries differ by service; `repositories/transaction.py` is the place to inspect. Atomicity of matching writes is unverified. |
| Unique constraint | A rule that a value, or combination, cannot repeat. | users.email, organisations.ein, and the composite (`source_system`, `external_id`) on opportunities and awards. |

#### D2. KNode tables, fields and enumerations

| Term | Definition | Context in KNode |
|---|---|---|
| `ai_files` | Records of files handled by the AI and document pipeline, linked to an organisation, funder or opportunity through a polymorphic association. | **Implemented.** Distinct from sources, which records evidence provenance. How the two relate is not fully documented. |
| `algorithm_version` | A label recording which scoring logic produced a match run. | Stored on `match_runs`. The AI Service page names the scoring weights set `match-score-v1`; whether that string is the stored value is unverified. |
| `audit_events` | Rows recording who did what, to which target, when. | Model and service **Implemented**; complete audit coverage is **Unverified**. ORM attribute `event_metadata` maps to the column metadata. |
| `coalition_policy` | A field on a funding opportunity stating the funder's stance on applicants forming coalitions. Enum, default UNSPECIFIED. | **Implemented.** Only UNSPECIFIED is documented; the other values are not listed. Read via GET `/api/v1/matches/opportunities/{id}/coalition-policy`. |
| `collective_members` | One row per member of a collective, with participation and response state. | **Implemented.** Membership uniqueness and response timestamps are constrained. Status values are not enumerated. |
| collectives | A group of organisations assembled around a funding opportunity. | **Implemented.** See *Collective* in section H. |
| `confidence_score` | A number (`Numeric(5,2)`) on an extracted claim indicating how sure the extraction was. | **Implemented** in the schema. How it is produced and whether it is used downstream is not documented; the scoring formula does not reference it. |
| `dedupe_key` | An optional key inside a requirement's structured value, used to merge duplicate requirements before scoring. | **Implemented** in the scoring module. The first requirement with the key is kept; conflicts produce warnings. |
| ein | Employer Identification Number: a US tax identifier, nine characters. | Stored on organisations (unique, nullable). Required at sign-up for organisations. Used by verification (**Design only**) against IRS data. |
| `extracted_claims` | A single fact about an organisation, extracted from a source, with evidence text, confidence and review status. | **Implemented.** See *Extracted claim* in section F. |
| funders | Organisations that offer funding. | **Implemented.** Fields: name, `funder_type`, focus areas, geographic focus, typical grant range, currency. |
| `funding_awards` | Historical records of grants already awarded. | **Implemented.** Source for the Funding Trends analysis. Unique on (`source_system`, `external_id`) and linked to a source. Not the same as a funding opportunity. |
| `funding_opportunities` | An open or upcoming call for applications from a funder. | **Implemented.** Fields include title, sector, grant range, currency, `deadline_at`, status (OPEN, CLOSED, UPCOMING, UNKNOWN), eligible geographies and organisation types. |
| `is_mandatory` | A boolean on a requirement saying whether it is compulsory. | Default true. Whether scoring treats mandatory and optional requirements differently is not documented. |
| `last_verified_at` | Timestamp of the last verification of an organisation or funder. | **Implemented** on both tables. The funder organisation search has a `verified_only` filter, but the documentation does not say which field it reads. |
| `match_partners` | Suggested partner organisations attached to a match. | **Implemented.** Suggestions are computed from stored assessments and always carry `participation_confirmed` = false. |
| `match_runs` | One execution of a matching workflow: who asked, what type, status, algorithm version, timing, errors. | **Implemented.** Statuses: PENDING, RUNNING, COMPLETED, FAILED. Run types: opportunity matching, partner discovery, collective building. |
| matches | The result of assessing one organisation (optionally against one opportunity): score, score breakdown, per-requirement assessments, explanation. | **Implemented.** Score is nullable; a legacy placeholder zero is detected and not shown as a real score. |
| `opportunity_requirements` | One requirement from a funding opportunity, with text, type, mandatory flag, optional structured value and source. | **Implemented.** Types written by extraction: eligibility, geography, `thematic_fit`, `financial_capacity`, partnership, evidence. |
| `organisation_email_domains` | The email domains associated with an organisation. | **Implemented.** Supports the planned domain check in verification (**Design only**). |
| organisations | NGOs or other implementing organisations that seek funding. | **Implemented.** Fields: legal and display name, EIN, mission, focus areas, countries, capabilities, registration status, revenue, `last_verified_at`. |
| `published_weight` | An optional weight the funder publishes for a requirement. | When all requirements have positive published weights they replace the default category weights. |
| referrals | A record directed at exactly one target, an organisation or a collective, with respond and withdraw actions and a versioned disclaimer (`REFERRAL_DISCLAIMER_VERSION`). | **Implemented** in schema and routes. A check constraint requires exactly one target. Statuses are not documented. |
| reports | Stored report content as JSON. | Table exists; **a public report router was not found**. |
| `requirement_assessments` | A JSON column on matches holding the status, evidence and quote for each requirement. | **Implemented.** Source of the per-requirement view shown in the UI. |
| `review_status` | The state of an extracted claim: PENDING (default), CONFIRMED, DISPUTED. | **Implemented.** Only CONFIRMED claims with non-empty evidence text are used in matching. |
| `score_breakdown` | A JSON column on matches holding the contribution of each scoring category. | **Implemented.** |
| `source_system` / `external_id` | A code naming where a record came from, plus that source's own id. | Values documented: USASPENDING, `GRANTS_GOV`, CRAWLER, MANUAL. Together they prevent duplicate imports. |
| sources | A record of a document or web page that evidence came from: URL or file reference, type, retrieval and publication dates, content, hash. | **Implemented.** See *Source document* in section I. |
| `structured_value` | A JSON field on a requirement that holds machine-readable detail such as category, published weight and dedupe key. | **Implemented**, optional. |
| users | Accounts. Each has a role (ORGANISATION, FUNDER, ADMIN), a status (ACTIVE, INACTIVE, SUSPENDED) and an optional organisation or funder link. | **Implemented.** The password is stored only as an Argon2 hash. |
| `workspace_items` | The things inside a workspace: tasks, documents and notes, with optional author and assignee. | **Implemented.** The frontend also stores discussion threads as NOTE items (see *Discussion thread*). |
| workspaces | A shared work area attached to one collective. | **Implemented.** See *Consortium workspace* in section H. |

### E. Authentication, authorisation and security

The Architecture page describes a larger security design than the Backend and Frontend pages confirm. Entries marked **Design only** should not be presented as built.

| Term | Definition | Context in KNode |
|---|---|---|
| Access token | A short-lived credential sent with each request to prove who the caller is. | **Implemented.** A JWT valid 900 seconds (15 minutes) by default (`ACCESS_TOKEN_TTL_SECONDS`), sent as `Authorization: Bearer`. |
| Argon2 | A deliberately slow, memory-hungry password-hashing algorithm. | **Implemented** through `PasswordHasher` in `core/security.py`. |
| Audit log / audit event | An append-only record of security-relevant actions. | Model and service **Implemented**; coverage **Unverified**. The Architecture page describes 12-month append-only retention (**Design only**). |
| Authentication vs authorisation | Authentication proves who you are (password plus one-time code). Authorisation decides what that identity may do (role and ownership checks). | Authentication: `/api/v1/auth/*`. Authorisation: `require_role`, `require_organisation_scope`, `require_funder_scope`. A valid token alone never implies access to a given record. |
| Bearer token | A token that grants access to whoever presents it. | Sent in the Authorization header by `authedRequest`. The frontend also sends `credentials: "include"`. |
| Cookie attributes (HttpOnly, Secure, SameSite) | Flags that keep a cookie away from page scripts, restrict it to HTTPS, and limit cross-site sending. | Backend settings: cookie `knode_refresh`, `REFRESH_COOKIE_SECURE=true`, SameSite=lax. The frontend stores the refresh token in `sessionStorage` instead. |
| Cross-tenant access | One organisation or funder reading or changing another's private data. | The central authorisation risk. The Backend page lists cross-organisation and cross-funder tests as required checks. See *Tenant isolation*. |
| Data privacy / redaction | Removing personal or sensitive data before it is stored, logged or sent elsewhere. | **Implemented** for Gemini evidence prompts: email-like strings become `[email removed]`, and database ids and organisation names are not sent. Logs hold error types, not document text. |
| Environment variable | A named value supplied by the environment rather than written in code. | **Implemented.** Examples: `DATABASE_URL`, `JWT_SECRET`, `GEMINI_API_KEY`. |
| Input validation | Checking that incoming data has the expected type, length and format. | Pydantic on the backend; zod on the frontend; file type, size and signature checks on uploads. |
| JWT (JSON Web Token) | A signed token carrying claims. | **Implemented** with PyJWT. Claims: sub (user), type (access, refresh, email verification, password reset), iat, exp, jti. Signed with `JWT_SECRET` using HS256 by default. |
| jti / revocation | jti is a random id for one token, so a specific token can be revoked. | **Implemented.** Dependencies check the jti and a per-user revoked-after-issued-at time. Revocation state is process-local unless stored shared. |
| KMS (Key Management Service) | A managed service that holds encryption keys. | **Design only.** Architecture: SSE-KMS for uploads and key rotation. Current uploads go to `UPLOAD_DIR`. |
| Least privilege | Giving each user or component only the access it needs. | Stated as a principle for secrets and keys (**Design only**). Backend roles are broad (organisation, funder, admin). |
| Lockout | Temporarily blocking an account after repeated failed logins. | **Implemented**: `LOGIN_MAX_ATTEMPTS=5`, `LOGIN_LOCKOUT_SECONDS=900`; returns 423. |
| OTP (one-time password) | A short numeric code valid once, delivered by email. | **Implemented.** Six digits, 600 seconds, 5 attempts. Stored hashed (SHA-256, constant-time compare). Used for sign-up verification and for every login (a second step). |
| Password hashing | Storing a one-way transform of the password, never the password. | **Implemented** with Argon2. Minimum length setting is 12. |
| Permission / role | A role is a named bundle of permissions. | Database roles: ORGANISATION, FUNDER, ADMIN. The Architecture page lists five. |
| Prompt injection | Text inside a document that tries to give the AI new instructions. | **Implemented** mitigations: untrusted-text markers, marker stripping, no database ids in prompts, quote verification. See *Untrusted input*. |
| Protected route | An endpoint that requires a valid token. | **Implemented** through the `get_current_user` dependency. The frontend's `/ngo/*` guard is cosmetic. |
| RBAC (Role-Based Access Control) | Granting access according to a user's role. | **Implemented** by role dependencies, which the Backend page says do not replace object-level checks. |
| Refresh token | A longer-lived credential used to obtain new access tokens. | Backend default 43,200 seconds (12 hours). The frontend stores but never calls the refresh endpoint. |
| Resource scope | A check that the caller may access this particular organisation or funder. | `require_organisation_scope`, `require_funder_scope`. `require_self_or_admin` does not currently compare ids, so it is not a reliable ownership check. |
| Row-Level Security (RLS) | A PostgreSQL feature that filters rows by policy for each database user. | **Design only.** Not mentioned in the Backend page. |
| Secret | A value that grants access if disclosed: keys, passwords, tokens. | `JWT_SECRET` (at least 32 characters), `GEMINI_API_KEY`, `SMTP_PASSWORD`. Never commit them or show real values in documentation. |
| Tenant isolation | Ensuring each customer organisation's private data is invisible to others. | Intended to be enforced by application role and scope checks, whose route-by-route verification is still outstanding. Some routes return 404 to avoid revealing that another tenant's record exists. Database-level RLS is **Design only**. |
| TOTP / passkey | Time-based one-time passwords from an authenticator app; passkeys are device-bound credentials replacing passwords. | **Design only.** Planned for funders and admins. |
| Untrusted input | Any text from uploads, websites or model output, treated as data and never as instructions or permissions. | **Implemented** principle across AI paths. Model output cannot grant access or write arbitrary database values. |
| User session | The period during which a user is signed in. | The frontend keeps it as tokens in `sessionStorage`. Logout is not implemented in the frontend (the sidebar link neither clears tokens nor calls the logout endpoint). |
| WAF / DDoS protection | A web application firewall filters malicious requests; DDoS protection absorbs floods of traffic. | **Design only.** Architecture lists Cloudflare and an API gateway. Not mentioned in code-based pages. |

### F. AI, evidence and matching

KNode's documented design is that AI reads and checks evidence while **fixed rules** compute eligibility, score and explanation. The AI never computes the score. No machine-learning training, embeddings or vector search is documented in the matching path.

| Term | Definition | Context in KNode |
|---|---|---|
| AI (artificial intelligence) / LLM (large language model) | Software that performs tasks needing language understanding; an LLM is a model trained on text to generate and interpret language. | **Implemented.** Google Gemini is the only model provider documented. Used for three jobs: extracting organisation facts, extracting funding calls, and checking evidence against a requirement. |
| Assessment provider | The component that decides a requirement's status from retrieved evidence. | **Implemented** behind an `AssessmentProvider` interface so it can be mocked; `provider_factory.py` returns the Gemini provider or disables it. Off by default (`AI_ASSESSMENT_ENABLED=false`). |
| Capability gap | A requirement the lead organisation does not clearly meet. | **Implemented** as a rule: a lead requirement with status UNKNOWN, `NOT_CHECKED`, ERROR, `NOT_MET` or `PARTIALLY_MET` is treated as a gap. A gap is not a proven weakness, because missing evidence is counted too. |
| Complementary organisation | An organisation with MET or `PARTIALLY_MET` evidence for a requirement the lead has as a gap. | **Implemented** as deterministic comparison of stored assessments, labelled "may complement". It does not use semantic search and does not confirm the partner is willing or available. |
| Data extraction / evidence extraction | Turning unstructured text or files into structured fields with a source. | **Implemented.** Website text, uploaded PDF, DOCX, PNG and TXT go to Gemini and return schema-validated JSON. Document facts become PENDING claims. |
| Eligibility (condition and overall status) | Whether the organisation meets hard conditions of an opportunity. Each condition is SATISFIED, FAILED, UNKNOWN or ERROR; the overall result is ELIGIBLE, INELIGIBLE, `NEEDS_VERIFICATION`, ERROR or `NOT_ASSESSED`. | **Implemented** with three conditions only: deadline not passed, geography overlap (no overlap is UNKNOWN, not a failure), organisation type (always UNKNOWN; no logic yet). A satisfied or failed condition with no evidence reference is downgraded to UNKNOWN. Kept separate from the score. |
| Evidence coverage | The share of applicable requirement weight that could be assessed. | **Implemented**: 100 x assessed weight / total applicable weight. Score status is COMPLETE, PROVISIONAL or `NOT_CALCULABLE`. |
| Evidence-backed matching / explainability | Every match result can be traced to the quotes and sources behind it, so a person can see why. | The project's central principle. **Implemented** through claim and source ids kept on each assessment and a template explanation built from the score breakdown. The AI does not write the explanation. |
| Extracted claim | A single fact about an organisation (for example a country of operation or a capability) with the supporting evidence text, its source, a confidence value and a review status. | **Implemented** (`extracted_claims`). Claims from uploads start PENDING; a person sets CONFIRMED or DISPUTED. Only CONFIRMED claims with non-empty evidence text reach matching. Website-scraped profile data skips this step. |
| False positive / false negative | A false positive is a match or status wrongly reported as good; a false negative is a real fit wrongly missed or marked bad. | Not measured. The Backend page lists measuring retrieval quality, false positives and false negatives as outstanding work. The design guards against false negatives by using UNKNOWN for missing evidence. |
| Gemini | Google's family of generative AI models. | Model name in `GEMINI_MODEL`; documented default `gemini-3.8-flash`, which the Backend page asks to confirm. Called with an API key or, as fallback, Vertex AI. |
| Grounding / quote verification | Checking that the model's cited quote really appears in the passage it cites. | **Implemented.** The cited passage label must map to a retrieved claim and the quote must appear after whitespace and case normalisation; otherwise the status becomes UNKNOWN. A valid quote does not prove correct interpretation. |
| Human-in-the-loop (human oversight) | A person must approve AI output before it takes effect. | **Implemented** at two points: confirming or disputing PENDING claims, and reviewing a draft opportunity before it is saved. |
| Lexical retrieval / information retrieval | Finding candidate evidence by matching words rather than meaning. | **Implemented.** Text is tokenised; shared alphanumeric tokens of at least three characters count as overlap; at most five passages go to the provider. May miss differently worded evidence. |
| Match | The stored result of assessing an organisation against an opportunity. | **Implemented** (matches). Carries a nullable score, score breakdown, per-requirement assessments, strengths and gaps, missing information, eligibility status and explanation. |
| Match run | One execution of a matching workflow that produces matches. | **Implemented** (`match_runs`): types opportunity matching, partner discovery, collective building; statuses PENDING, RUNNING, COMPLETED, FAILED. |
| Match score | A number from 0 to 100 summarising how well assessed requirements are met. | **Implemented** as a deterministic weighted formula, rounded to one decimal (`ROUND_HALF_UP`). It measures capability alignment, not funding probability or eligibility. Whether showing a number is approved product policy is unconfirmed. |
| Matching engine | The backend module that assesses requirements, checks eligibility, scores, finds partners and summarises collective coverage. | **Implemented** in `services/matching_engine/`. The Architecture page describes it as scoring "using fixed rules". |
| Partner discovery | Suggesting organisations that may fill a lead organisation's gaps. | **Implemented** at GET `/api/v1/matches/{match_id}/partners`. Suggestions always set `participation_confirmed` to false and include a disclaimer. |
| Passage | A piece of confirmed evidence text with its claim, source and organisation ids, sent to the provider. | **Implemented.** Labelled P1, P2 and so on in prompts, and mapped back to ids afterwards. |
| Requirement assessment status | The verdict on one requirement: MET, `PARTIALLY_MET`, `NOT_MET`, UNKNOWN, `NOT_CHECKED` (no provider configured), ERROR (provider failure or invalid output). | **Implemented.** `NOT_MET` requires an explicit contradiction, never just missing evidence. A legacy contract maps SUPPORTED, `PARTIALLY_SUPPORTED`, UNVERIFIED to MET, `PARTIALLY_MET`, UNKNOWN. |
| Scoring categories and weights | Five groups of requirements with default weights: programme strategic alignment 0.25, programme delivery capability 0.25, experience and track record 0.20, target population and context 0.15, team capacity 0.15. | **Implemented** (set called `match-score-v1` in the AI page). Points: MET 4, `PARTIALLY_MET` 2, `NOT_MET` 0, maximum 4. Only these three statuses are scored. Funder-published weights replace defaults when complete. |
| Semantic similarity / embedding / vector search | Comparing meaning using numeric vectors (embeddings) and finding nearest matches, often with a vector index such as pgvector. | **Not found** in the matching path; the Backend page warns not to describe it as implemented. The Architecture page lists a pgvector store for chunks and embeddings (**Design only**). |
| Structured output / structured data | Data in a fixed machine-readable shape. Structured output means forcing the model to reply in JSON matching a schema. | **Implemented**: `response_mime_type=application/json` plus Pydantic validation; invalid replies are rejected and nothing is saved. |
| Temperature | A model setting controlling randomness; 0 is most repeatable. | Assessment uses 0; extraction uses 0.1. |
| Untrusted-text markers | Delimiters such as `<untrusted_text>` or `<<<EVIDENCE ... EVIDENCE>>>` that tell the model content is data to read, not instructions. | **Implemented.** Marker sequences found inside input are stripped first so content cannot close a block early. |

### G. Funding and nonprofit domain

Terms are used as KNode's data model uses them. Industry usage may treat some as interchangeable; KNode does not.

| Term | Definition | Context in KNode |
|---|---|---|
| Candid / ProPublica Nonprofit Explorer / GlobalGiving (IATI) | Third-party sources of nonprofit profiles, financial data, and project or funder data. IATI is the International Aid Transparency Initiative data standard. | Listed as organisation and registry sources in the Integrations page. Whether each is currently ingested is not established. |
| EIN (Employer Identification Number) | A nine-digit US tax identifier for an organisation. | Required at sign-up for organisations, stored in organisations.ein, and the key to IRS verification (**Design only**). |
| Eligibility requirement | A condition an applicant must meet to be considered for an opportunity. | In the data model eligibility appears twice: as structured opportunity fields (eligible geographies and organisation types, deadline) assessed by rules, and as a requirement of type eligibility assessed like any other requirement. |
| Form 990 / 990-PF | The US annual information return filed by tax-exempt organisations (990-PF for private foundations). | Source of financial and geographic information (IRS data, ProPublica). |
| Funder | An organisation that provides grants. | funders table and FUNDER user role. A funder owns opportunities, discovers organisations, and reviews collectives. |
| Funding award | A grant that has already been made. | `funding_awards`. Used for historical data and the Funding Trends analysis. Not the same as an opportunity. |
| Funding opportunity | An open, upcoming or closed call to which organisations can apply. | `funding_opportunities`. Has requirements, a deadline, a grant range and a status. Called "grant" in funder screens and "funding call" in AI prompts. |
| Funding trends / funding landscape | An analysis of historical awards by region, sector and timeframe. | **Implemented**: POST `/api/v1/trends/analysis`, shown as a line chart for NGOs. Totals are reported per currency. The MVP's grant gap analysis is absent from the frontend. |
| Grant | A sum of money given for a purpose, normally without repayment. | The word in funder screens for what the database calls a funding opportunity. A grant amount range is `grant_min` / `grant_max`. |
| Grants.gov / USAspending | US government sites publishing grant opportunities (Grants.gov) and historical federal awards (USAspending). | Ingested as `source_system` values `GRANTS_GOV` and USASPENDING. The Grants.gov job searches for the keyword "Africa" over the last 60 days. |
| Historical funding record | A past award stored with its source. | See *Funding award*. A missing amount or date stays empty, never zero. |
| IRS Business Master File (BMF) / TEOS | The IRS list of exempt organisations and its search service. | Used to load organisations for profile building (up to 100 per run) and, by design, to verify registration. |
| Mandatory requirement | A requirement the funder marks as compulsory. | `is_mandatory`, default true. |
| Nonprofit organisation / NGO (non-governmental organisation) | An organisation that works for public benefit, not profit, independent of government. | KNode targets African-focused NGOs. The code and UI call them organisations (ORGANISATION role) and `/ngo/*`; the Architecture page says "NGO" and "organization". |
| Organisation | A nonprofit that seeks funding. One row in organisations, identified by a UUID. | Distinct from a **collective**, which is a group of organisations around one opportunity. Registration in the US does not show that an organisation works in Africa. |
| Organisation profile / organisational capability | The stored description of an organisation: mission, focus areas, countries of operation and capabilities. Capabilities are the things it can do. | **Implemented** as JSON lists on organisations. The documented assessment path reads confirmed claims and their evidence text, not these lists directly. |
| Programme area / focus area / sector | The thematic field of work. KNode uses three related words: `focus_areas` for organisations and funders, sector for an opportunity. | AI extraction is limited to the values HEALTH and LIVELIHOODS; the database fields are free JSON. |
| Registration status | A text field on an organisation describing its legal registration. | `registration_status`, `String(100)`. Its allowed values are not documented. |
| Requirement (opportunity) | One condition or expectation from a funding call, stored with text, type and weight. | `opportunity_requirements`. Types: eligibility, geography, `thematic_fit`, `financial_capacity`, partnership, evidence. At most 30 per extracted call. |
| Verified organisation | An organisation whose legal registration and domain have been checked. | **Design only** in the Architecture page (EIN against IRS data, email domain match, OTP to a domain address, admin review). The Backend page documents only `last_verified_at` and an ein field. |

### H. Collectives, workspaces and collaboration

The documentation uses **collective**, **coalition** and **consortium** for overlapping ideas. The data model knows only collectives. The documentation does not say whether the three words differ, so treat them as one concept until the team decides.

| Term | Definition | Context in KNode |
|---|---|---|
| Accepted vs proposed member | Whether a collective member has accepted an invitation or is only suggested. | The deterministic coverage route takes an `include_proposed` option, and the test plan covers accepted versus proposed members. The full status vocabulary is not documented. |
| Coalition | The funder-side word for a collective. Also in `coalition_policy`, a field on opportunities. | **Implemented** in the funder UI (`/funder/coalitions`, `lib/funderCoalitions.ts`) and the API (coalition-policy). The funder list reads the same collectives records (GET `/collectives`?`opportunity_id`=). The documentation does not state this identity explicitly. |
| Collective | A group of organisations formed around one funding opportunity so that together they can meet its requirements. | **Implemented** (collectives, `collective_members`). Has members, a coverage view, a lead suggestion and one workspace. Not a legal entity and not a single organisation. A collective-building match run is one of the three run types. |
| Collective coverage (COVERED / PARTIAL / UNRESOLVED / UNCOVERED) | For each requirement, whether the members together meet it. COVERED: at least one member is MET. PARTIAL: best status is `PARTIALLY_MET`. UNRESOLVED: no result yet and some assessments are unresolved. UNCOVERED: all relevant assessments are negative. | **Implemented** in `services/matching_engine/collective.py`, plus counts of unconfirmed participants. The frontend calls the same idea capability coverage. Two coverage routes exist. |
| Collective member / participation status | A member organisation of a collective, with its response to an invitation. | **Implemented.** Membership uniqueness and response timestamps are database constraints. The frontend tabs derive from `my_membership_status` (all, active, invited). |
| Consortium | The word the Architecture and Frontend pages use for the collaboration between collective members, as in "consortium workspace". | **Not a stored entity.** There is no consortia table and no stated difference from a collective. |
| Consortium workspace / workspace | A shared work area belonging to one collective, holding workspace items. | **Implemented.** workspaces has one row per collective; the frontend creates it on first visit (GET, then POST on 404). The word "consortium" appears only in the UI and the Frontend page. |
| Discussion thread | A conversation among collective members inside the workspace. | **Frontend convention only.** Threads are stored as NOTE workspace items whose title starts with `[t:<id>]`. The backend has no thread concept. |
| Lead organisation / lead suggestion | The organisation treated as the lead when partner discovery looks for gaps; a lead suggestion proposes one for a collective. | **Implemented** at `/collectives/{id}/lead-suggestion` (method and schema to be confirmed); no frontend caller. Partner discovery treats the lead's gaps as the starting point. |
| Member invitation / respond | A collective's invitation to an organisation and that organisation's reply. | **Implemented**: POST `/collectives/{id}/members/invite` and `.../respond`. The NGO UI can accept or decline; no UI exists to send invitations (`inviteMember` is unused). |
| Participation confirmed | A flag stating whether a suggested partner has agreed to take part. | Always false for partner suggestions. A suggestion means evidence may complement a gap, not that anyone has agreed. |
| Referral | A funder-initiated record directed to one organisation or one collective, which the recipient can respond to or the funder withdraw. | **Implemented** in schema and routes (`/referrals`, respond, withdraw); no frontend caller. The Architecture page describes "funder-to-NGO interactions"; the schema also allows a collective target. Status values are not documented. |
| Shortlist / Select | Funder actions on a coalition's detail page. | **Placeholder**: the buttons are disabled. |
| Workspace item / task | One entry in a workspace. Types seen: tasks, documents and notes. A task is an item with an assignee and status. | **Implemented** (`workspace_items`). Item creation is in the NGO UI; edit and delete exist in the API but have no UI caller. Whether task status changes are supported in the UI was not traced. |

### I. Data ingestion, evidence and reporting

| Term | Definition | Context in KNode |
|---|---|---|
| Background job / batch job | A task run outside a user request, usually on a schedule or from a command line. | **Implemented** as Python entry points in `services/jobs/` (scraper, `midnight_scraper`, `process_grants`, `midnight_grants`, `opportunity_matching`, nightly). Run by command; a production scheduler was **not verified**. |
| Data ingestion / data pipeline | Collecting data from outside sources, cleaning it and storing it as structured records. | **Implemented** for Grants.gov grants, IRS organisation lists and organisation websites. Each record keeps its source and `source_system`. The Architecture page's queued "Data Ingestion Service" is **Design only**. |
| Data validation | Rejecting or cleaning data that does not meet rules. | Imported data is validated and sanitised; AI output is validated against Pydantic schemas with length, number and allowed-value checks; HTML tags are stripped. |
| Draft opportunity | An AI-extracted opportunity returned for a funder to review before anything is saved. | **Implemented**: POST `/opportunities/extract` (PDF, DOCX or TXT, up to 10 MB). Calls with no title, a past deadline, a non-African region or a non-open call are rejected. |
| Email delivery (SMTP, STARTTLS) | SMTP is the standard protocol for sending email; STARTTLS upgrades the connection to encrypted. | **Implemented** in `services/email_service.py` on port 587 with a 15-second timeout. Used for sign-up and login codes only. In prod missing credentials raise an error; in dev the code is printed to the log. Backend settings also list Resend. |
| Evidence ledger | The frontend's name for the list of evidence entries held for an organisation. | Served by GET `/organisations/{id}/evidence` and shown as documents (`ledgerEntryToDocument`). Its exact relation to sources and `extracted_claims` is not documented. |
| File signature check / SHA-256 hash | A file signature check compares the file's first bytes with its claimed type (%PDF, PK). A SHA-256 hash is a fingerprint that detects identical files. | **Implemented** for uploads (PDF, DOCX, PNG). Duplicates are detected by hash. Originals are stored under a random name inside the organisation's own folder. |
| Idempotency | Repeating a request has the same effect as doing it once. | Not established for matching. The Backend page lists duplicate-request and idempotency tests as outstanding. Import uniqueness on (`source_system`, `external_id`) gives it for imported records. |
| Match report / report generation | A generated document summarising matches. | **Not found as an exposed workflow.** A reports table with JSON content exists and the Architecture page lists a Reports component that emails reports; no public report router was found. |
| Provenance / source traceability | Knowing where each fact came from and when. | **Implemented** as a core rule. Every extracted claim links to a source; imported records keep source and retrieval date; assessments keep claim and source ids and the verified quote. |
| Rate limiting / politeness delay | Limiting how fast requests are made. | Jobs pause between items (4 seconds per organisation, 7 seconds per grant). Website collection respects `robots.txt`. A gateway-level rate limit is **Design only**. |
| Retry / backoff | Repeating a failed call, waiting longer each time. | **Implemented** for requirement assessment: up to 2 retries with exponential backoff (`GEMINI_MAX_RETRIES`), then ERROR. No automatic retry on the frontend, which offers manual retry on two hooks. |
| Scraping / crawling | Fetching web pages automatically and extracting content. | **Implemented** with Requests, BeautifulSoup and Playwright. A Serper search finds an organisation's website; pages under 200 characters are discarded. Playwright here is for scraping, not testing. |
| Source document | A file or web page from which evidence was taken. | **Implemented** as a sources row: name, URL or storage key, type, dates, content and hash. An uploaded document is kept as a source even when extraction fails. |
| Worker / job queue | A separate process that takes queued tasks so users do not wait. | **Design only.** The Architecture page describes queued workers; the Backend page documents command-line jobs and says a scheduler is unverified. |

### J. Testing and quality assurance

| Term | Definition | Context in KNode |
|---|---|---|
| Contract test | A test checking that two sides agree on an interface. | `openapiContract.test.ts` checks matching endpoints and fields against the frontend's snapshot of the OpenAPI file. It does not test the live backend. |
| Coverage (test coverage) | The share of code executed by tests. | Backend: `pytest --cov --cov-report=term-missing`. A coverage figure is not given; 71 backend files match test*.py, which is a file count, not a pass result or case count. |
| CI (continuous integration) | Automatically building and testing every change. | Backend workflows `ci.yml` and `pr-checks.yml` exist. The frontend's only workflow checks that the pull-request template is ticked and does not run build, lint or tests, although its README says it does. |
| End-to-end (E2E) test / Playwright | An E2E test drives the whole app through a real browser. Playwright is a browser-automation tool. | **No E2E tests are documented.** Playwright appears in the backend stack for scraping, not testing. |
| Factory / fixture / mock | A factory builds test records; a fixture is shared test setup; a mock replaces a real dependency. | `tests/factories.py`, `tests/conftest.py` (in-memory SQLite, rollback transaction), `AssessmentProvider` mocked so no Gemini call is needed. |
| Integration test | A test of several parts working together, such as a route plus its database. | Backend API tests use FastAPI `TestClient` with `get_db` overridden. SQLite does not prove PostgreSQL behaviour, so a separate PostgreSQL test workflow is recommended. |
| Linting / formatting | Automatic checks for code-style problems and layout. | Backend: Ruff (`ruff check`, `ruff format --check`). Frontend: ESLint and Prettier, run before commits by Husky and lint-staged. |
| pytest / Vitest | The Python and JavaScript test runners. | Backend uses pytest with pytest-cov. Frontend uses Vitest with jsdom and Testing Library: 10 test files. |
| Postman | An API testing tool. | The docs describe importing `/openapi.json` and a local environment with `baseUrl` and `accessToken`. Use test accounts only. |
| Smoke test | A quick check that the main paths work after a deploy. | Recommended after release: authentication, organisation and opportunity access, matching, `/health`. |
| Type checking | Verifying that values match declared types before running. | Frontend: next build type-checks; no project script exists (`npx tsc --noEmit` works). Backend type checking is not documented. |
| Unit test | A test of one small function in isolation. | Scoring, eligibility and quote validation are pure functions and suit unit tests. |

### K. Infrastructure, deployment and operations

| Term | Definition | Context in KNode |
|---|---|---|
| CD / deployment | Delivering a tested build to a running environment. | Frontend hosting: Vercel. Backend: Heroku.|
| Containers / infrastructure as code | Containers package an application with its dependencies; infrastructure as code defines servers and networks in files. | **Design only** (Architecture). |
| Development / test / staging / production | Environments of increasing realism and risk. | The ENV setting accepts dev, test, staging, prod. Dev prints one-time codes to logs; prod requires SMTP credentials. |
| Health check | An endpoint that reports whether the service is up. | GET `/health` returns `{"status":"ok"}`. It is a constant and **not** a database or provider readiness check. |
| Logging / monitoring | Logging records events; monitoring watches metrics and alerts. | Logging **Implemented** via `configure_logging` (DEBUG in dev, INFO otherwise). Metrics, tracing, alerting and log aggregation were not verified. |
| MkDocs | A static-site generator for documentation. | Builds this documentation site (Material theme), deployed from GitHub Actions to GitHub Pages. |
| Object storage / pre-signed URL | Cloud file storage; a pre-signed URL is a time-limited link to one file. | **Design only.** Current uploads are stored in a local `UPLOAD_DIR`. |
| Release step (migrations) | Applying database migrations deliberately when releasing. | Recommended: run Alembic as a controlled release step, confirm `/health`, smoke-test, then schedule jobs only if intended. |
| Secrets manager | A service that stores secrets and hands them to applications. | 
## Acronym and abbreviation index

Only abbreviations that appear in the documentation are listed. ERD and PRD do not appear in any page provided. Where a concept has a master-glossary entry, the entry holds the full definition; this table gives the expansion and one line of use.

| Abbreviation | Full form | Meaning or use in KNode |
|---|---|---|
| AI | Artificial intelligence | Gemini-based extraction and requirement assessment. See section F. |
| API | Application Programming Interface | The backend's JSON endpoints under `/api/v1`. |
| ARIA | Accessible Rich Internet Applications | `aria-*` attributes used in the sidebar drawer, tabs and alerts. |
| BMF | Business Master File | IRS list of exempt organisations used to seed organisations. |
| CD | Continuous deployment | Delivering tested builds automatically. See section K. |
| CI | Continuous integration | Automated build and test on each change. See section J. |
| CORS | Cross-Origin Resource Sharing | `CORS_ALLOW_ORIGINS` allow-list. |
| CRUD | Create, Read, Update, Delete | Basic data operations on most entities. |
| CSP | Content Security Policy | Not set by the frontend; noted as a gap. |
| CSR | Client-side rendering | How almost all frontend pages render. |
| DDoS | Distributed denial of service | Perimeter protection in the security design . |
| DKIM / DMARC / SPF | Email authentication standards | Email sender protection . |
| DSN | Data source name | The database connection string form of `DATABASE_URL`. |
| E2E | End-to-end | No E2E tests documented. |
| EIN | Employer Identification Number | US tax id on organisations.ein. |
| GCP | Google Cloud Platform | `GCP_PROJECT_ID` and `GCP_REGION` for the Vertex AI fallback. |
| GUID / UUID | Globally / Universally Unique Identifier | Primary key type. |
| HSTS | HTTP Strict Transport Security | Forces HTTPS . |
| HTTP / HTTPS | Hypertext Transfer Protocol (Secure) | Transport for the API. |
| IaC | Infrastructure as code | **Design only.** |
| IATI | International Aid Transparency Initiative | Data standard used by GlobalGiving. |
| IRS | Internal Revenue Service (US) | Source of exempt-organisation data. |
| JSON | JavaScript Object Notation | Request, response and column format. |
| JTI | JWT ID | Per-token id used for revocation. |
| JWT | JSON Web Token | Access and refresh tokens. |
| KMS | Key Management Service | **Design only.** |
| LLM | Large language model | Gemini. |
| MFA | Multi-factor authentication | The frontend reads an `mfa_required` flag from the login response; the second factor is the emailed OTP. |
| MIME | Multipurpose Internet Mail Extensions | Media type, for example `application/json` or supported upload types. |
| MVP | Minimum viable product | The scope document referenced by the Frontend page; not provided. |
| NGO | Non-governmental organisation | The product's primary user. |
| ORM | Object-Relational Mapping | SQLAlchemy. |
| OTP | One-time password | Six-digit emailed code for sign-up and login. |
| RBAC | Role-Based Access Control | Role dependencies in the backend. |
| REST | Representational State Transfer | The API style. |
| RLS | Row-Level Security | **Design only.** |
| SAD / SSAD | System Architecture Diagram / Security System Architecture Diagram | Figures 1 and 2 in the Architecture page. |
| SDK | Software development kit | Google Gen AI SDK used for Gemini. |
| SMTP | Simple Mail Transfer Protocol | Email delivery. |
| SQL | Structured Query Language | Database query language. |
| SSE-KMS | Server-side encryption with KMS keys | **Design only.** |
| SSRF | Server-side request forgery | Ingestion protection (**Design only**). |
| TEOS | Tax Exempt Organization Search | IRS search service. |
| TLS | Transport Layer Security | Encrypted connections. |
| TOTP | Time-based one-time password | **Design only**, funders and admins. |
| TTL | Time to live | Lifetime in seconds of tokens and OTPs. |
| UI | User interface | The Next.js frontend. |
| WAF | Web application firewall | **Design only.** |
| WCAG | Web Content Accessibility Guidelines | The Frontend page states conformance is not claimed. |
| XSS | Cross-site scripting | Risk noted for tokens in `sessionStorage`. |
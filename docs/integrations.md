# Integrations

KNode connects to a small set of third-party services. This page describes each one: what it does, what data it handles, how it is configured, and how the system behaves when it is unavailable.

## Overview

| Integration                                                             | Purpose in KNode                                                             |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| [Google Gemini](#google-gemini-ai-document-extraction)                  | Reads uploaded documents and extracts structured facts about an organisation |
| [Email (SMTP)](#email-delivery-smtp)                                    | Delivers sign-up and login verification codes                                |
| [PostgreSQL](#postgresql)                                               | Stores all application data                                                  |
| [Funding data: USAspending and Grants.gov](#funding-data-sources)       | Supplies historical awards and funding opportunities                         |
| [Organisation and registry sources](#organisation-and-registry-sources) | Builds organisation profiles from public information                         |
| [Uploaded documents](#uploaded-documents)                               | Evidence supplied directly by organisations                                  |

## How the integrations fit together

```text
                      ┌───────────────────────────┐
  Browser ──────────▶ │  Next.js frontend         │
  (NGO / Funder)      └─────────────┬─────────────┘
                                    │  HTTPS, JWT, /api/v1/*
                                    ▼
                      ┌───────────────────────────┐
                      │  FastAPI backend          │
                      └───┬───────────┬───────┬───┘
                          │           │       │
              ┌───────────▼──┐  ┌─────▼────┐  ┌▼─────────────────┐
              │ PostgreSQL   │  │ SMTP     │  │ Google Gemini    │
              │ (data)       │  │ (email)  │  │ (extraction)     │
              └──────▲───────┘  └──────────┘  └──────────────────┘
                     │
        USAspending · Grants.gov · IRS · ProPublica ·
        Candid · GlobalGiving · organisation websites
```

Every integration sits behind the backend. The frontend never calls a third party directly, and no third-party key is ever sent to the browser.

---

## Google Gemini (AI document extraction)

### Purpose

When an organisation uploads a report, Gemini reads it and returns structured facts: mission, focus areas, countries of operation, capabilities, annual revenue, registration status and headline impact metrics. The backend stores these facts as claims in the organisation's evidence ledger.

### Flow

1. The user uploads a PDF, DOCX or PNG to `POST /api/v1/organisations/{id}/documents`.
2. The backend checks the file type, size and file signature, then stores the file.
3. `services/document_extraction.py` sends the document to Gemini with a fixed instruction prompt and a JSON schema describing the expected answer.
4. The response is validated again in the backend against the `ExtractedProfile` model.
5. Each extracted fact is saved as a **`PENDING`** claim linked to its source document. A person must confirm a claim before it counts as verified.

### Data sent to Gemini

| Sent                                                                                             | Never sent                           |
| ------------------------------------------------------------------------------------------------ | ------------------------------------ |
| The document content (PDF and PNG as files, DOCX as extracted text capped at 100,000 characters) | Account details, passwords or tokens |
| The fixed instruction prompt                                                                     | Other organisations' data            |
| The JSON schema for the answer                                                                   | Database records                     |

### Configuration

| Setting            | Meaning                                                                         |
| ------------------ | ------------------------------------------------------------------------------- |
| `GEMINI_API_KEY`   | API key for Gemini. Without it, the upload API answers `503 AI_NOT_CONFIGURED`. |
| `GEMINI_MODEL`     | Gemini model used for extraction                                                |
| `MAX_UPLOAD_BYTES` | Largest file accepted                                                           |
| `UPLOAD_DIR`       | Folder where uploaded files are stored                                          |

### Safeguards

- **Untrusted input.** The prompt instructs the model to treat the document as data and ignore any instructions written inside it.
- **Structured output.** The model must answer in JSON that matches the schema. The backend validates the result again, because the model does not enforce length and pattern limits.
- **No guessing.** The model uses only what the document states and leaves unknown fields empty.
- **Human review.** Extracted facts remain `PENDING` until a person confirms them.
- **Additive only.** Extraction adds claims. It never overwrites what an organisation or an administrator has entered.
- **Private logs.** Failures log the type of error only, never document text.

### Failure behaviour

| Situation                                        | API response                        |
| ------------------------------------------------ | ----------------------------------- |
| No API key configured                            | `503` with code `AI_NOT_CONFIGURED` |
| Network error, quota exceeded or unreadable file | `502` with code `EXTRACTION_FAILED` |
| Model output does not match the schema           | `502` with code `EXTRACTION_FAILED` |

The uploaded document is kept as a source in every case, so a failed extraction never loses the file.

---

## Email delivery (SMTP)

### Purpose

KNode sends one-time codes by email for two purposes:

| Purpose              | When it is sent                                       |
| -------------------- | ----------------------------------------------------- |
| Sign-up verification | After registration, to activate the account           |
| Login verification   | After a correct email and password, to complete login |

Codes have six digits, expire after 10 minutes (`OTP_TTL_SECONDS`) and allow five attempts (`OTP_MAX_ATTEMPTS`). Codes are stored hashed.

### Configuration

| Setting          | Default          | Meaning                                             |
| ---------------- | ---------------- | --------------------------------------------------- |
| `SMTP_HOST`      | `smtp.gmail.com` | SMTP server                                         |
| `SMTP_PORT`      | `587`            | Port. The connection is upgraded with STARTTLS.     |
| `SMTP_USER`      | none             | Sender account                                      |
| `SMTP_PASSWORD`  | none             | Sender password. For Gmail this is an app password. |
| `SMTP_FROM_NAME` | `KNode Alya`     | Sender display name                                 |

### Behaviour by environment

| `ENV`                    | When SMTP credentials are not set                                                                      |
| ------------------------ | ------------------------------------------------------------------------------------------------------ |
| `dev`, `test`, `staging` | The email is skipped and a warning is logged. In `dev` the one-time code is printed in the server log. |
| `prod`                   | Sending raises an error. Credentials are mandatory.                                                    |

### Design

- Emails use a branded HTML template with a plain-text fallback.
- Messages contain the code and the recipient's name only.
- Connections use TLS and a 15 second timeout.

---

## PostgreSQL

PostgreSQL is the system of record. It holds users, organisations, funders, sources and claims, funding opportunities and awards, matches, collectives, workspaces, referrals, reports and audit events.

| Setting        | Meaning                                 |
| -------------- | --------------------------------------- |
| `DATABASE_URL` | PostgreSQL connection string (required) |

All schema changes go through Alembic migrations. See the [Backend](backend/index.md) page for the migration workflow.

---

## Funding data sources

Historical awards power the Funding Trends dashboard, and open calls power opportunity search. Every record stores where it came from, so any figure can be traced to its source.

| Source          | Provides                                         | Recorded as                     |
| --------------- | ------------------------------------------------ | ------------------------------- |
| USAspending.gov | Historical awards                                | `source_system = 'USASPENDING'` |
| Grants.gov      | Opportunities and awards                         | `source_system = 'GRANTS_GOV'`  |
| Web crawler     | Opportunities and awards from funder sites       | `source_system = 'CRAWLER'`     |
| Manual entry    | Opportunities added by a funder or administrator | `source_system = 'MANUAL'`      |

### Data integrity rules

- Every award links to a source record, and `(source_system, external_id)` is unique, so the same award can never be stored twice.
- A missing amount or date stays empty. It is never stored as zero.
- Currencies are never mixed. The trends API reports totals per currency.
- Corrections to an award require a reason, and each correction is written to the audit log.

---

## Organisation and registry sources

Organisation profiles are built from public information. Each fact keeps a link to its source and the date it was retrieved.

| Source                                      | Provides                                |
| ------------------------------------------- | --------------------------------------- |
| IRS Business Master File (BMF)              | Registration and tax-exempt status      |
| IRS Tax Exempt Organization Search (TEOS)   | Status of US tax-exempt organisations   |
| IRS Form 990 and 990-PF filings             | Financial information                   |
| ProPublica Nonprofit Explorer               | Organisation financial data             |
| Candid                                      | Organisation profile data               |
| GlobalGiving and GlobalGiving IATI          | Projects, opportunities and funder data |
| Organisation websites and published reports | Programmes, impact and evidence         |

### Rules that apply to every source

- Imported data is validated and sanitised before it is stored.
- US registration or tax-exempt status alone does not show that an organisation works in Africa. Evidence of operations comes from programme descriptions, project records, filings or reports.
- Personal contact details are not collected beyond an organisation's published contact point.
- Collection from websites respects `robots.txt` and site terms, and is rate limited.
- Imported data extends a profile. It never overwrites what an organisation has claimed.

---

## Uploaded documents

Organisations can upload impact reports and similar documents as evidence.

| Item           | Rule                                                                                                                        |
| -------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Accepted types | PDF, DOCX and PNG                                                                                                           |
| Validation     | The file extension and the file signature must match (`%PDF` for PDF, `PK` for DOCX). Size is capped by `MAX_UPLOAD_BYTES`. |
| Duplicates     | A SHA-256 hash detects the same file uploaded twice                                                                         |
| File names     | The original name is sanitised for display. Files are stored under a random name inside the organisation's own folder.      |
| Visibility     | Only the owning organisation's members and administrators can see an uploaded document                                      |
| Audit          | Every upload is written to the audit log                                                                                    |
| Storage        | `UPLOAD_DIR`                                                                                                                |

---

## Frontend and backend

The Next.js frontend talks only to the KNode backend, through the versioned `/api/v1` prefix.

| Concern        | How it works                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Authentication | A 15-minute access token is sent as a bearer token. A refresh token is kept in an HTTP-only cookie named `knode_refresh`. |
| Cookies        | Controlled by `COOKIE_SECURE`, `COOKIE_SAMESITE` and `COOKIE_DOMAIN`                                                      |
| CORS           | Only origins listed in `CORS_ALLOW_ORIGINS` are allowed. The default is `http://localhost:3000`.                          |
| Errors         | Every error has the shape `{"error": {"code", "message", "details"}}`                                                     |
| API contract   | The OpenAPI specification is exported from the backend and describes every endpoint                                       |

---

## Configuration reference

Set these in `.env`. Never commit real values.

| Variable                                                                 | Required                    | Used by                    |
| ------------------------------------------------------------------------ | --------------------------- | -------------------------- |
| `ENV`                                                                    | No (default `dev`)          | Email and cookie behaviour |
| `DATABASE_URL`                                                           | Yes                         | PostgreSQL                 |
| `JWT_SECRET`                                                             | Yes (32 or more characters) | Token signing              |
| `GEMINI_API_KEY`                                                         | For document extraction     | Gemini                     |
| `GEMINI_MODEL`                                                           | No                          | Gemini                     |
| `MAX_UPLOAD_BYTES`, `UPLOAD_DIR`                                         | No                          | Uploads                    |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_FROM_NAME` | In production               | Email                      |
| `CORS_ALLOW_ORIGINS`                                                     | In production               | Frontend access            |

## Failure modes

| If this is unavailable | What users see                                            | What keeps working                             |
| ---------------------- | --------------------------------------------------------- | ---------------------------------------------- |
| Gemini                 | The upload succeeds and extraction answers `502` or `503` | Profile, evidence ledger, trends, login        |
| SMTP                   | Sign-up and login codes are not delivered                 | Everything for users who are already signed in |
| PostgreSQL             | All API calls fail                                        | Static pages only                              |
| A public data source   | Nothing. Data stays as last imported.                     | Everything                                     |

## Testing integrations locally

1. **Email.** Leave `SMTP_USER` empty with `ENV=dev`, register a user, and read the code from the server log. To test real delivery, set the `SMTP_*` values with a Gmail app password.
2. **Gemini.** Set `GEMINI_API_KEY` in `.env`, restart the server, upload a short test PDF, and check that the new claims appear as `PENDING`.
3. **No AI key.** Remove `GEMINI_API_KEY` and confirm the upload API answers `503 AI_NOT_CONFIGURED`.
4. **Automated tests.** Run `uv run --no-sync --active pytest`.

See also: [Architecture](architecture.md), [Backend](backend/index.md), [AI Service](ai.md) and the [Glossary](glossary.md).

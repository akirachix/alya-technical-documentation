# API Reference

## 1. API conventions

The FastAPI application is identified as KNode version `0.1.0`. For local development, the base URL is commonly `http://127.0.0.1:8000`. When enabled, interactive documentation is available at `/docs` and the generated OpenAPI document at `/openapi.json`.

Protected routes generally use a bearer access token. UUID fields are validated by FastAPI/Pydantic where typed as UUIDs. Request and response schemas are primarily under `schemas/`.

## 2. Endpoint inventory

This inventory reflects paths identified in the source review. Check `/openapi.json` for the exact method, path, schema, status code and authentication requirement for the current build.

### Authentication — `/api/v1/auth`

| Method | Path                        | Purpose                        |
| ------ | --------------------------- | ------------------------------ |
| POST   | `/api/v1/auth/register`     | Register a user.               |
| POST   | `/api/v1/auth/verify-email` | Verify email.                  |
| POST   | `/api/v1/auth/login`        | Start login flow.              |
| POST   | `/api/v1/auth/verify-login` | Verify login challenge/OTP.    |
| POST   | `/api/v1/auth/refresh`      | Refresh tokens.                |
| POST   | `/api/v1/auth/logout`       | Log out/revoke session.        |
| GET    | `/api/v1/auth/me`           | Return the authenticated user. |

### Organisations

| Method   | Path                                                | Purpose                                                                           |
| -------- | --------------------------------------------------- | --------------------------------------------------------------------------------- |
| POST/GET | `/api/v1/organisations`                             | Create/list organisations, subject to actual registered methods and access rules. |
| GET      | `/api/v1/organisations/{organisation_id}`           | Retrieve an organisation.                                                         |
| PATCH    | `/api/v1/organisations/{organisation_id}`           | Update an organisation.                                                           |
| GET      | `/api/v1/organisations/{organisation_id}/team`      | Retrieve organisation team information.                                           |
| POST     | `/api/v1/organisations/{organisation_id}/documents` | Submit/upload organisation documents.                                             |
| GET      | `/organisations/{organisation_id}/evidence`         | Retrieve organisation evidence; identified as an unversioned route.               |

### Funders

| Method   | Path                            | Purpose                                                                     |
| -------- | ------------------------------- | --------------------------------------------------------------------------- |
| POST/GET | `/api/v1/funders`               | Create/list funders, subject to actual registered methods and access rules. |
| GET      | `/api/v1/funders/{funder_id}`   | Retrieve a funder.                                                          |
| PATCH    | `/api/v1/funders/{funder_id}`   | Update a funder.                                                            |
| GET      | `/funders/{funder_id}/evidence` | Retrieve funder evidence; identified as an unversioned route.               |

### Opportunities and requirements

| Method       | Path                                                                   | Purpose                                                                                  |
| ------------ | ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| POST         | `/api/v1/opportunities/extract`                                        | Extract opportunity information.                                                         |
| POST/GET     | `/api/v1/opportunities`                                                | Create/list opportunities.                                                               |
| GET          | `/api/v1/opportunities/mine`                                           | Retrieve opportunities associated with the current user/context.                         |
| GET          | `/api/v1/opportunities/{opportunity_id}`                               | Retrieve an opportunity.                                                                 |
| PATCH        | `/api/v1/opportunities/{opportunity_id}`                               | Update an opportunity.                                                                   |
| DELETE       | `/api/v1/opportunities/{opportunity_id}`                               | Delete an opportunity.                                                                   |
| POST/GET     | `/api/v1/opportunities/{opportunity_id}/requirements`                  | Create/list requirements; verify exact prefix and whether reads are public or protected. |
| PATCH/DELETE | `/api/v1/opportunities/{opportunity_id}/requirements/{requirement_id}` | Update/delete a requirement; verify exact path in OpenAPI.                               |

### Matching

| Method | Path                                                              | Purpose                                       |
| ------ | ----------------------------------------------------------------- | --------------------------------------------- |
| POST   | `/api/v1/matches`                                                 | Create a match assessment.                    |
| GET    | `/api/v1/matches`                                                 | List accessible matches.                      |
| GET    | `/api/v1/matches/{match_id}`                                      | Retrieve an accessible match.                 |
| GET    | `/api/v1/matches/{match_id}/partners`                             | Retrieve potential partner suggestions.       |
| GET    | `/api/v1/matches/opportunities/{opportunity_id}/coalition-policy` | Retrieve coalition policy for an opportunity. |
| GET    | `/api/v1/collectives/{collective_id}/coverage`                    | Matching-service coverage route.              |

### Collectives and coverage

| Method           | Path                                            | Purpose                                                                                                        |
| ---------------- | ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| POST/GET         | `/collectives`                                  | Create/list collectives; identified as an unversioned route.                                                   |
| GET/PATCH/DELETE | `/collectives/{collective_id}`                  | Retrieve/update/delete a collective, where registered.                                                         |
| POST/GET         | `/collectives/{collective_id}/members`          | Add/list collective members.                                                                                   |
| POST             | `/collectives/{collective_id}/members/invite`   | Invite a member.                                                                                               |
| POST             | `/collectives/{collective_id}/members/respond`  | Respond to membership invitation.                                                                              |
| POST             | `/collectives/{collective_id}/coverage/refresh` | Refresh coverage.                                                                                              |
| GET              | `/collectives/{collective_id}/coverage`         | Deterministic coverage route with `include_proposed`; unversioned and distinct from the versioned route above. |
| GET/POST         | `/collectives/{collective_id}/lead-suggestion`  | Lead suggestion; verify actual method and schema in OpenAPI.                                                   |

The two coverage routes have different implementations and are an important API-contract verification item. Confirm both are intentional and documented clearly.

### Evidence and claims

| Method     | Path                                 | Purpose                                         |
| ---------- | ------------------------------------ | ----------------------------------------------- |
| GET        | `/evidence/{source_id}/claims`       | Retrieve claims associated with a source.       |
| POST/PATCH | `/evidence/claims/{claim_id}/review` | Review a claim; verify exact supported methods. |

These routes were identified as unversioned. Confirm the complete evidence route set in OpenAPI.

### Referrals

| Method   | Path                                | Purpose                |
| -------- | ----------------------------------- | ---------------------- |
| POST/GET | `/referrals`                        | Create/list referrals. |
| GET      | `/referrals/{referral_id}`          | Retrieve a referral.   |
| POST     | `/referrals/{referral_id}/respond`  | Respond to a referral. |
| POST     | `/referrals/{referral_id}/withdraw` | Withdraw a referral.   |

### Trends, workspaces and health

| Method           | Path                                         | Purpose                                      |
| ---------------- | -------------------------------------------- | -------------------------------------------- |
| POST             | `/api/v1/trends/analysis`                    | Run trend analysis.                          |
| POST/GET         | `/collectives/{collective_id}/workspace`     | Create/get a collective workspace.           |
| GET              | `/workspaces/{workspace_id}`                 | Retrieve a workspace.                        |
| POST/GET         | `/workspaces/{workspace_id}/items`           | Create/list workspace items.                 |
| GET/PATCH/DELETE | `/workspaces/{workspace_id}/items/{item_id}` | Retrieve/update/delete a workspace item.     |
| GET              | `/health`                                    | Health endpoint returning `{"status":"ok"}`. |

The endpoint table is a navigational inventory, not a substitute for the live OpenAPI contract. Verify exact methods, prefixes, response schemas and status codes before integrating a client.

## 3. Detailed matching endpoint contracts

### Create a match assessment

`POST /api/v1/matches`

- Request schema: `schemas.matching.MatchCreateRequest`.
- Response schema: `MatchResponse`.
- Expected success status: `201` in the inspected route.
- Service call: `matching_service.create_match_assessment(db, user, organisation_id, opportunity_id, provider)`.
- Persistence may create `match_runs` and `matches` records.
- Invalid body or malformed UUID: typically `422`.
- Missing resources: not-found response.
- Access violation: forbidden or not-found, depending on the path's disclosure policy.
- Provider failures may appear in the assessment status rather than always failing the HTTP request.

Example body (confirm exact schema before use):

```json
{
  "organisation_id": "<organisation-uuid>",
  "opportunity_id": "<opportunity-uuid>"
}
```

### List matches

`GET /api/v1/matches`

Query parameters identified in the source include `organisation_id`, `opportunity_id`, `limit` (1–100, default 20) and `offset` (minimum 0, default 0). Authentication is required. The response uses `MatchListResponse` and should filter results by the caller's permitted scope.

### Retrieve a match

`GET /api/v1/matches/{match_id}`

Returns `MatchResponse`. Access is resolved through `_get_accessible_match`; the inspected policy considers organisation management scope or a funder associated with the opportunity. A not-found response may be intentional to avoid disclosing that another tenant's resource exists.

### Partner suggestions

`GET /api/v1/matches/{match_id}/partners`

Returns `PartnerSuggestionsResponse`. The view logic looks for organisations with `MET` or `PARTIALLY_MET` assessments for requirements that remain gaps for the lead organisation. The response should say that a candidate **may complement** the lead; it must not imply availability, agreement or confirmed participation.

## 4. Errors and response contracts

Exact response formats vary because both `core.errors` and `core.error_handlers` register handlers. Capture real responses from the running application before treating the examples as definitive.

Typical categories include:

| Situation                                | Typical status | Notes                                                             |
| ---------------------------------------- | -------------: | ----------------------------------------------------------------- |
| Missing, invalid or expired access token |            401 | Token/dependency validation.                                      |
| Insufficient role or resource scope      |            403 | Permission check.                                                 |
| Resource not found                       |            404 | Some paths use not-found to reduce resource-existence disclosure. |
| Request validation error                 |            422 | FastAPI/Pydantic.                                                 |
| Conflict                                 |            409 | `ConflictError`, where used.                                      |
| Account lockout                          |            423 | `AccountLockedError`.                                             |
| Too many OTP attempts                    |            429 | `OtpTooManyAttemptsError`.                                        |
| AI extraction unavailable                |            503 | `ExtractionUnavailable`.                                          |
| AI extraction failed                     |            502 | `ExtractionFailed`.                                               |
| Unhandled exception                      |            500 | Generic client response; exception should be logged.              |

A validation response may contain an error code, message and field-level details. Other responses may contain an HTTP-style code and correlation ID. Do not expose raw database errors, provider payloads, credentials or source document contents in client-facing errors.

## 5. OpenAPI and Postman

When the application is running, retrieve `/openapi.json` and use Postman's OpenAPI import to generate a collection. Inspect the generated requests after import: OpenAPI generation does not guarantee that authentication, examples, response errors or every edge case are documented correctly.

Recommended environment variables in Postman:

- `baseUrl`, for example `http://127.0.0.1:8000`
- `accessToken`, obtained from the real login/verification flow
- `organisationId`
- `opportunityId`
- `matchId`
- `collectiveId`

Do not store production credentials in shared workspaces. Use a local test account and a test database. Example requests and scenario coverage are in [Testing and Operations](testing.md).

## 6. Endpoint-to-code reference

| Feature        | Router(s)                               | Schema(s)                 | Service/repository areas                                    | Model areas                                                                     |
| -------------- | --------------------------------------- | ------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Authentication | `routers/auth.py`                       | `schemas/auth.py`         | `services/auth_service.py`, token/OTP stores                | `User`                                                                          |
| Organisations  | `routers/organisations.py`              | `schemas/organisation.py` | organisation and document services                          | `Organisation`, `User`, `AIFile`, `Source`                                      |
| Funders        | `routers/funders.py`                    | `schemas/funder.py`       | `funder_service.py`                                         | `Funder`, `User`                                                                |
| Opportunities  | `routers/opportunities.py`              | `schemas/opportunity.py`  | opportunity and extraction services                         | `FundingOpportunity`, `Funder`, `Source`                                        |
| Requirements   | `routers/requirements.py`               | `schemas/requirement.py`  | requirement service/repository                              | `OpportunityRequirement`                                                        |
| Evidence       | `routers/evidence.py`                   | `schemas/evidence.py`     | evidence service/repository                                 | `Source`, `ExtractedClaim`                                                      |
| Matching       | `routers/matching.py`                   | `schemas/matching.py`     | matching service/repository and `services/matching_engine/` | `MatchRun`, `Match`, `MatchPartner`, `OpportunityRequirement`, `ExtractedClaim` |
| Collectives    | `routers/collectives.py`, member router | `schemas/collective.py`   | collective service/repository                               | `Collective`, `CollectiveMember`                                                |
| Coverage       | `routers/coverage.py`                   | `schemas/coverage.py`     | coverage service, collective coverage, lead suggestion      | `Collective`, `CollectiveMember`, `Organisation`                                |
| Referrals      | `routers/referrals.py`                  | `schemas/referral.py`     | referral service/repository                                 | `Referral`, `Funder`, `Collective`, `Organisation`                              |
| Workspaces     | workspace and workspace-item routers    | workspace schemas         | workspace/item services                                     | `Workspace`, `WorkspaceItem`                                                    |
| Trends         | `routers/trends.py`                     | Confirm imported schema   | trend service and repository                                | `FundingAward`, `Funder`, `Organisation`                                        |
| Audit          | Service-level                           | N/A                       | audit service/repository modules                            | `AuditEvent`                                                                    |

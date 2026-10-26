# Authentication and Security

## 1. Security overview

The backend uses token-based authentication and role/resource-scope checks. The main controls identified in the source include Argon2 password hashing, JWT expiry and token-type validation, token JTI revocation checks, inactive-account rejection, dependency helpers for role and organisation/funder scope, Pydantic validation, generic handling of unexpected errors, correlation-ID middleware, an audit-event model/service, and AI prompt/evidence-grounding safeguards.

The presence of a helper or model does not prove every route uses it correctly. Resource-level authorization must be verified route by route, including list endpoints, request-body identifiers, query parameters, uploads, evidence retrieval, and workspace access.

## 2. Passwords and tokens

### Password hashing

`core/security.py` uses Argon2's `PasswordHasher` with explicit parameters. The database stores a password hash, not a plaintext password. Never log passwords or verification codes. Confirm that password policy and hashing configuration are applied consistently across registration and password-reset flows.

### JWTs

The token helpers create JWTs with claims including:

- `sub`: subject/user identifier.
- `type`: token purpose, such as access, refresh, email verification or password reset.
- `iat`: issued-at time.
- `exp`: expiry time.
- `jti`: random token identifier used by revocation checks.

`JWT_SECRET` and `JWT_ALGORITHM` configure signing/verification. The secret must be strong, private, and rotated through a controlled procedure. Do not commit secrets or include them in error messages, logs or documentation.

### OTPs and opaque tokens

The code includes OTP generation and hashing using SHA-256 with constant-time comparison, along with opaque-token creation and hashing helpers. Settings include OTP lifetime, OTP length and maximum attempts, plus login attempt and lockout controls. Verify that attempt counters and revocation/OTP state are shared safely across multiple processes and survive restarts when required. In-memory state is not shared automatically between workers.

## 3. Authentication request flow

`core/dependencies.py` reads the bearer token, decodes an access token, handles invalid/expired tokens, checks JTI revocation and user revocation-after-issued-at, loads the user, and rejects missing or inactive accounts. Additional dependencies enforce roles and resource scopes.

The main helper categories are:

| Helper | Intended responsibility | Review note |
|---|---|---|
| `get_current_user` | Resolve the authenticated user from the access token. | Confirm consistent use on protected endpoints. |
| `require_role` / `require_any_role` | Restrict routes to specified roles. | Role checks do not replace object-level authorization. |
| `require_organisation_scope` | Check organisation-related access. | Verify the resource ID used is the one being accessed. |
| `require_funder_scope` | Check funder-related access. | Verify cross-funder isolation. |
| `require_self_or_admin` | Intended to permit self-access or admin access. | The inspected implementation returns the current user without comparing the supplied target user ID; it is not a reliable ownership check until fixed or guarded by a separate verified check. |

## 4. Authorization requirements

Use both role-level and resource-level authorization. A valid token or correct role alone does not mean a user can access every organisation, funder, opportunity, match, collective, evidence source, referral or workspace.

Required verification includes:

1. A user from one organisation cannot read or modify another organisation's private records.
2. A funder cannot access unrelated funder data or match results outside its permitted scope.
3. Resource identifiers in a body or query string cannot bypass access checks.
4. List endpoints filter records by caller scope rather than returning broad database results.
5. Evidence ledgers and uploaded documents are protected against cross-tenant access.
6. Collective membership, invitations, referral targets and workspace items have appropriate access rules.
7. Administrative privileges are explicit and tested.
8. Sensitive routes return consistent 401/403/404 behaviour according to a deliberate disclosure policy.

Some routes may return `404` rather than `403` to avoid revealing that another tenant's resource exists. Document and test that behaviour rather than applying it inconsistently.

## 5. Cookies, CORS and production configuration

The observed settings include refresh-cookie name `knode_refresh`, `REFRESH_COOKIE_SECURE=true`, `SameSite=lax`, and an optional cookie domain. Verify the actual refresh-token transport and cookie attributes in the deployed configuration. Production must use HTTPS, appropriate secure-cookie settings, narrowly scoped CORS origins, and a strong secret managed outside source control.

The configuration includes `CORS_ALLOW_ORIGINS`, `APP_BASE_URL`, `ENV`, and `DEBUG`. Review production values explicitly; do not rely on development defaults. Ensure secrets are not accidentally included in logs or exception traces.

## 6. Evidence and AI safety

Treat source documents, web pages, extracted text, and model outputs as untrusted. AI output must not determine permissions, bypass service validation, or write arbitrary database fields. Preserve source/claim identifiers and verify cited quotations against the stored passage. Distinguish provider failure from unknown evidence and from a negative finding.

Before sending sensitive organisational material to an external AI provider, review the provider, region, retention, contractual and data-processing policies. Validate file type, size, storage location and access rules before upload, retrieval or AI submission. Avoid logging raw documents, credentials, tokens or full prompts containing sensitive evidence.

## 7. Error handling and information disclosure

The repository contains both `core/error_handlers.py` and `core/errors.py`. `main.py` registration order may cause one `AppError` handler to replace another. Verify which handler is effective and standardise error payloads and correlation IDs.

Client-facing errors should not expose raw database exceptions, SQL, provider payloads, secrets, or source-document contents. Unexpected exceptions should be logged with suitable context while returning a generic client-safe response. Do not include personal or sensitive data in correlation IDs.

## 8. Audit logging and operational security

The codebase contains an `audit_events` model and audit service/repository modules. This is not proof that all security-sensitive actions are audited. Verify coverage for account changes, profile changes, evidence ingestion/review, matching runs, collective membership changes, referral actions, workspace operations and administrative actions.

Check that:

- Logs exclude passwords, access/refresh tokens, OTPs and secrets.
- Audit records identify actor, action, target, time and relevant outcome without storing unnecessary sensitive content.
- Audit data has appropriate access controls and retention.
- Correlation IDs are returned consistently and allow request tracing.
- Revocation/OTP/lockout state behaves correctly across worker processes and restarts.
- Backup, restore and incident-response procedures are documented.

## 9. Security review checklist

- [ ] Fix or replace `require_self_or_admin` so it compares the target user ID with the authenticated user's ID, or restrict its use to contexts where another verified check performs that comparison.
- [ ] Complete a route-by-route object-authorization review, including list routes and body/query IDs.
- [ ] Test cross-organisation and cross-funder access denial.
- [ ] Verify organisation/funder evidence ledgers cannot be accessed across tenants.
- [ ] Review upload validation, storage protections and document retrieval permissions.
- [ ] Standardise error handlers and confirm correlation IDs are returned consistently.
- [ ] Verify token/OTP stores are appropriate for multi-process deployment; process-local state is not automatically shared.
- [ ] Confirm secret loading, rotation, production cookie and CORS settings.
- [ ] Check SQLAlchemy query construction remains parameterized. The source review did not establish a raw SQL injection finding.
- [ ] Review AI provider data handling, region, retention and contractual policy.
- [ ] Verify that all security-sensitive actions that require audit records actually produce them.
- [ ] Test authentication expiration, refresh, logout/revocation, lockout and inactive-account handling.

## 10. Relevant API and test references

Authentication endpoints are listed in [API Reference](api-reference.md). Use the tests in the `tests/` tree to identify existing coverage, but do not infer complete security coverage from the existence of test files. The [Testing and Operations](testing.md) document provides recommended negative and cross-tenant scenarios.

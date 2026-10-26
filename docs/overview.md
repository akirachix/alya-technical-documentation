# KNode

## 1. Executive summary

KNode is an AI-assisted platform intended to help African-focused nonprofit organisations discover relevant funding opportunities, present evidence of their capabilities, and identify organisations whose capabilities complement their own. It also supports funders who need to review organisations and proposed collectives against opportunity requirements. Its goal is to make funding discovery and collaboration more evidence-based, explainable, and accessible without replacing human judgement.

The product combines several layers:

- **Web frontend:** the user-facing experience for account access and the organisation- and funder-facing workflows. The supplied hosted entry point is the sign-up page. Project context indicates a React/Next.js frontend; the frontend source was not part of the backend repository archive reviewed for this document.
- **Backend API:** a FastAPI application that validates requests, applies business rules, coordinates feature services, enforces access controls where implemented, and returns data to the frontend.
- **Data layer:** PostgreSQL is the configured target database. The product data model covers users, organisations, funders, opportunities, requirements, evidence, matching runs and results, collectives, referrals, workspaces, reports, and audit events.
- **AI and document intelligence:** AI-assisted extraction and requirement assessment can structure information from funding calls and organisation documents. Results should remain traceable to source evidence and visibly distinguish known facts from generated or uncertain content.
- **Operational integrations:** external AI providers, email delivery, and permitted public information sources support extraction, reporting, and discovery when configured and approved.

KNode should be understood as a **decision-support platform**, not an automated grant-award system. A match or partner suggestion does not confirm legal eligibility, organisational legitimacy, partner consent, or the likelihood of receiving funding.

## 2. Problem and product purpose

Organisations often have to search across multiple websites, reports, registries, and informal networks to find suitable funding. Funding calls are commonly written as long, unstructured documents, while evidence of an organisation's work can be spread across annual reports, project pages, financial filings, and other public material. Funders face the corresponding challenge of assessing organisations and combinations of organisations without having enough time to reconstruct the evidence manually.

KNode's core problem statement is:

> Organisations struggle to match their capabilities with complementary partners to meet funding requirements, while funders struggle to assess the combined capabilities and track record of collaboratives.

The product is intended to help users answer questions such as:

- What funding opportunities are relevant, and where did each opportunity come from?
- What does a funding call require, including eligibility, geography, thematic priorities, deadlines, and partnership rules?
- What evidence supports an organisation's profile and its ability to meet a requirement?
- Which requirements appear met, partly met, unknown, or unsupported by the available evidence?
- Which other organisations might complement a capability gap for a particular opportunity?
- How could a proposed collective cover the opportunity's requirements, and what gaps remain?
- What should a funder review before deciding whether to engage with an organisation or collective?

## 3. Users and product areas

### 3.1 Primary users

| User group                     | Main needs                                                                                                              | Intended product areas                                                                                                                                 |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Organisation users (NGOs/NPOs) | Maintain an evidence-backed profile, discover opportunities, understand gaps, and find potential complementary partners | Organisation profile, opportunity discovery, matching results, partner suggestions, collective builder/workspace, reports, and trend/gap analysis      |
| Funder users                   | Publish or review opportunities, assess organisations and proposed collectives, and refer relevant organisations        | Funder profile, opportunity management, organisation/collective discovery, evidence review, shortlists/referrals, and reports                          |
| KNode administrators/reviewers | Support platform operations, investigate flagged or uncertain records, and oversee access or quality processes          | Administrative and review workflows, audit-related records, and operational controls; the exact UI and permissions require implementation verification |
| Collective members             | Coordinate contributions and responsibilities around a funding opportunity                                              | Collective membership, invitations, roles, tasks, notes, documents/links, requirement checklists, and gap tracking                                     |

### 3.2 Main product areas

1. **Account access and profiles** — account registration and authentication, organisation/funder profile information, and role-aware access.
2. **Opportunity intelligence and search** — discover opportunities, preserve their source and verification state, and structure long calls into comparable requirements.
3. **Evidence profiles and evidence ledger** — collect or upload source material, extract claims, preserve provenance and evidence status, and allow appropriate review/correction.
4. **Opportunity matching** — compare opportunity requirements with available organisational evidence and show the reasoning and information gaps.
5. **Complementary organisation discovery** — suggest potential partners based on capabilities, geography, programmes, and evidence gaps, not popularity or network size.
6. **Collective builder and workspace** — assemble potential collaborators, understand collective-level coverage, and coordinate work for a shared opportunity.
7. **Funder discovery and referrals** — help funders review relevant organisations or collectives and create referrals with clear status and context.
8. **Reports and notifications** — deliver useful results and updates, including email reports when the relevant delivery integration is configured.
9. **Trend and gap analysis** — present funding trends and identify information or capability gaps relevant to an organisation and an opportunity.

These areas describe the product scope from the requirements and data model. They do not imply that every screen or end-to-end workflow is currently complete in the deployed frontend.

## 4. Frontend experience

### 4.1 Hosted entry point

The hosted frontend URL supplied for this document is:

- [KNode sign-up page](https://alya-frontend-dashboard.vercel.app/auth/signup)

The URL identifies the sign-up route of the deployed frontend. Frontend source context available for this project also includes a KNode application shell with navigation entries for **Dashboard**, **My Collectives**, **Trend Analysis**, and **Organisation Profile**. These entries establish intended frontend areas, but a navigation link does not prove that the destination page is complete or connected to live data. The hosted page could not be fetched in the current review environment, so this document does not make claims about its current visual layout, exact fields, validation messages, or successful connection to the production API. Those details should be verified in a browser and against the current frontend source before being treated as confirmed behaviour.

### 4.2 Frontend responsibilities

At a solution level, the frontend is the interaction layer. It should:

- present account registration and sign-in flows appropriate to user roles;
- collect and validate user input before sending API requests;
- display organisation/funder information, opportunities, evidence, matching results, and collective/referral workflows as those screens become available;
- make evidence provenance, review status, unknown information, and AI-generated content understandable to users;
- show loading, empty, success, validation, permission, and service-error states rather than treating every response as success;
- respect the access decisions made by the backend and avoid relying on client-side role checks as the security boundary; and
- use the KNode API rather than connecting directly to the database, AI model, or source-data providers.

### 4.3 Frontend status and evidence

| Item                               | Current evidence                                                                                                                                                                                                 | Documentation status                                                                              |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Hosted web entry point             | User-provided Vercel sign-up URL                                                                                                                                                                                 | URL supplied; runtime page not independently inspected during this pass                           |
| Frontend framework                 | Project context indicates React/Next.js                                                                                                                                                                          | Confirm against the frontend repository's `package.json` and application configuration            |
| Frontend source and page inventory | Project context identifies the application shell navigation (Dashboard, My Collectives, Trend Analysis, Organisation Profile) and a sign-up page; the frontend repository is not included in the backend archive | Partial source context; route behaviour and completeness not verified                             |
| Frontend-to-API integration        | Intended architecture is frontend → KNode API → services/data                                                                                                                                                    | Verify each page's API calls, authentication state, error handling, and environment configuration |
| Full user journeys                 | Product requirements and epics define intended journeys                                                                                                                                                          | End-to-end acceptance testing still required                                                      |

## 5. End-to-end solution architecture

KNode is designed as a web application backed by an API and a relational data store. The frontend should communicate with the backend API only. The API owns access checks, business rules, persistence, and coordination with AI and external services.

### 5.1 Frontend layer

The frontend is responsible for the user-facing experience and API consumption. The deployed sign-up URL is known, but a complete screen inventory and verified API contract require review of the frontend repository and the running application.

### 5.2 Backend/API layer

The inspected backend uses Python and FastAPI. Its main responsibilities include:

- exposing HTTP endpoints and OpenAPI documentation;
- validating requests and response structures with Pydantic;
- applying business logic and access checks through routers, dependencies, and services;
- managing account and profile operations;
- coordinating opportunity, evidence, matching, collective, referral, workspace, trend, and report functions; and
- returning errors in API responses and recording operational information.

The implementation uses a layered structure with routers, schemas, services, repositories, SQLAlchemy models, and shared core/infrastructure modules. The separation is not absolute in every module, so this should be understood as the current repository convention rather than a claim of strict Clean Architecture.

### 5.3 Data layer

PostgreSQL is the configured target database for the backend. The data model defines persistent records for users, organisations, funders, sources, extracted claims, opportunities, requirements, matches and matching runs, collectives and their members, referrals, workspace items, reports, AI-related files/passages, and audit events. Alembic is used for schema migrations.

The data model specifies source traceability and explicit handling of unknown values. Missing information should remain missing or unknown rather than being filled with a guess. The deployed schema and migration state still need to be validated against the actual project database before the model document is treated as proof of production state.

### 5.4 AI and external services

The backend repository includes Google Gen AI/Gemini-related integrations for selected extraction and matching tasks. Provider availability depends on environment configuration. The code review found keyword-overlap retrieval in the matching path; the backend implementation inspected did not establish that vector search or pgvector retrieval was wired into that path. The separate backend technical document should remain the source for implementation-level detail.

The product requirements describe external data collection and email/report delivery as part of the intended system. Use of any external provider or public data source must respect access terms, privacy requirements, and approved configuration. A provider appearing in a requirements document is not proof that a production integration is enabled.

## 6. Core user workflows

The following workflows summarize the intended product behaviour. Individual steps may be implemented to different levels and must be verified through frontend/API integration tests.

### 6.1 Registration and account access

1. A user opens the hosted sign-up page.
2. The frontend collects registration information and submits it to the KNode API.
3. The backend validates the request and handles account creation and authentication-related processes.
4. The user is directed into the appropriate organisation or funder experience when the relevant frontend journey is implemented.

The backend contains authentication-related routes and services. The current hosted sign-up page's fields, validation behaviour, email verification experience, and successful post-sign-up journey were not independently verified in this pass.

### 6.2 Opportunity discovery and structuring

1. An opportunity is created, collected, or submitted from a permitted source.
2. Its source/provenance and verification state are preserved.
3. The opportunity content is converted into structured requirements such as geography, applicant type, thematic priorities, financial criteria, partnership conditions, and deadline where the source supports them.
4. Unclear fields remain “Not specified” or another approved unknown state.
5. Users review the opportunity and its source before relying on it.

The requirements emphasise that an aggregator-only opportunity must not be presented as verified without suitable confirmation and that a deadline must not be inferred when the source is ambiguous.

### 6.3 Evidence profile and review

1. Public-source information or an uploaded document enters the evidence-processing workflow.
2. The system extracts candidate claims and retains a link to the source and relevant passage where available.
3. Extracted claims are structured and assigned an appropriate evidence/review status.
4. Organisation users review, correct, or remove extracted claims where the product workflow supports it.
5. The displayed profile distinguishes source-backed information, organisation-supplied information, AI-generated suggestions, and unknowns.

Uploaded documents should not be publicly accessible by default. Visibility and sharing permissions must be enforced by the backend and verified before sensitive evidence is exposed to funders or partners.

### 6.4 Opportunity matching and gap analysis

1. The system identifies relevant opportunity requirements and available organisational evidence.
2. Hard eligibility conditions, where implemented and supported by reliable data, are checked separately from evidence interpretation.
3. Relevant evidence passages are retrieved and assessed against individual requirements.
4. Results show the supporting evidence and explain gaps or uncertainty.
5. The organisation uses the results to decide whether to investigate or pursue the opportunity.

A result is a decision-support signal. It does not prove eligibility or predict a funding award. Unknown or unavailable evidence must not automatically be interpreted as proof that the organisation lacks a capability.

### 6.5 Complementary partner discovery and collectives

1. An organisation identifies an opportunity or capability gap.
2. KNode considers whether other organisations have evidence that could complement the gap.
3. Suggested partners are presented as possibilities (for example, “may complement”), not as confirmed collaborators or definitive “best” partners.
4. The user can evaluate and invite potential members.
5. A collective-level view helps identify which requirements have evidence coverage and what remains unresolved.
6. Members coordinate roles, contributions, tasks, documents, and notes in a shared workspace where those features are available.

The product requirements explicitly reject discovery ordering based on connection count, profile views, popularity, or network size. Complementarity should be grounded in capability, geography, programme experience, and evidence—not similarity alone.

### 6.6 Funder discovery, referrals, and reports

Funders are intended to review organisations and potential collectives with supporting evidence, gaps, and unknowns; create referrals; and receive or send reports through the relevant delivery workflow. A referral is an introduction, not an endorsement or funding decision. Email delivery depends on configured and tested provider integration.

### 6.7 Trends and gap analysis

The product requirements include trend analysis for funding opportunities and analysis of what an organisation needs to strengthen for an opportunity. The presence of trend-related backend modules does not establish that a complete dashboard visualization or end-to-end frontend workflow is deployed. The available data, aggregation logic, UI, and expected outputs need acceptance testing.

## 7. AI boundaries and evidence principles

AI is used to reduce the effort of structuring information and interpreting evidence. It must not become the sole authority for access, eligibility, evidence truth, partner consent, or funding decisions.

Key principles from the product and security requirements are:

- **Source traceability:** important claims and assessments should link to their origin and retain retrieval or verification context where available.
- **Explicit uncertainty:** missing, conflicting, outdated, blocked, and unassessed information should remain distinguishable.
- **Human review:** organisations should be able to review relevant extracted claims; uncertain or high-impact information should be routed for appropriate review.
- **Separation of responsibilities:** deterministic rules handle hard constraints and application permissions; AI helps extract or interpret text and draft explanations.
- **No invented facts:** if a requirement or claim cannot be supported, it should be marked unknown or not specified instead of inferred.
- **No false assurance:** a match does not confirm legal eligibility, legitimacy, willingness to collaborate, or likelihood of funding.
- **Fair discovery:** connection counts, popularity, and network size must not determine which organisation is surfaced as a potential partner.
- **Visible AI provenance:** AI-generated content must be distinguishable from organisation-confirmed or independently verified information.

## 8. Security, privacy, and governance

The Security and Privacy Requirements document sets expectations for authentication, least-privilege authorization, secure handling of personal and organisational data, safe document processing, source collection controls, logging, backup/recovery, incident handling, and security testing. These are requirements, not a certification that every control is currently implemented.

Areas the solution must address include:

- role-based access for organisation users, funders, and administrators/reviewers;
- object-level and tenant-level authorization, including prevention of insecure direct object reference (IDOR) access;
- secure password storage and appropriately protected authentication tokens/cookies;
- private-by-default uploaded documents and controlled field-level sharing;
- secrets management and restricted access to external provider credentials;
- validation and safe processing of uploaded documents;
- server-side protections for any URL fetching or crawling, including SSRF controls, rate limiting, and robots.txt/terms compliance;
- privacy-conscious logs, audit records for sensitive operations, and data minimisation;
- encryption, backup, recovery, retention, and breach-response controls; and
- security review and automated testing before deployment.

The backend source contains authentication/security utilities and access-control components, but implementation coverage must be assessed per route and workflow. This document does not claim that the system has passed a full security audit, achieved OWASP ASVS Level 2, or completed all privacy requirements.

## 9. Technology overview

| Layer                    | Technology / approach                                                                | Status and qualification                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| Web frontend             | React/Next.js according to project context; hosted sign-up entry point on Vercel     | Confirm exact framework/version, deployment configuration, screen coverage, and API wiring against frontend source           |
| Backend API              | Python, FastAPI, Pydantic                                                            | Found in the inspected backend repository                                                                                    |
| Persistence              | PostgreSQL, SQLAlchemy ORM, Alembic migrations                                       | PostgreSQL is the configured target; deployed schema and migrations require environment-level verification                   |
| Authentication           | JWT-related utilities, password hashing, role/scope dependencies                     | Present in backend source; production configuration and route-by-route authorization need verification                       |
| AI integration           | Google Gen AI/Gemini integration; Vertex AI support in selected configuration paths  | External availability depends on configuration; the exact provider/model configuration should be confirmed before deployment |
| Retrieval                | Keyword-overlap retrieval in the inspected matching path                             | Vector retrieval/pgvector integration in the matching path was not established by the source review                          |
| Testing and code quality | Pytest, Ruff; API testing can be performed with an HTTP client/Postman               | Test results must be associated with their database/configuration and should include PostgreSQL and end-to-end coverage      |
| Delivery                 | Hosted frontend link supplied; backend deployment configuration exists in repository | Deployment configuration is not proof of production health, uptime, or full integration                                      |

## 10. Implementation status and verification matrix

The following matrix deliberately separates product scope from implementation evidence.

| Capability                   | Requirements say                                                   | Evidence available for this review                                                                                                    | Status to communicate                                                                   |
| ---------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Hosted sign-up entry         | Users can register for KNode                                       | User supplied the Vercel `/auth/signup` URL; page itself could not be fetched in this environment                                     | Hosted route supplied; live behaviour not verified                                      |
| Authentication and accounts  | Role-aware organisation/funder access                              | Auth routes/services and security utilities found in backend source                                                                   | Implemented in backend code; end-to-end frontend journey needs testing                  |
| Organisation/funder profiles | Maintain profile information and evidence                          | Profile routers/services/models found in backend source                                                                               | Backend capability present; complete UI and permission behaviour require verification   |
| Opportunity intelligence     | Discover opportunities and structure requirements                  | Opportunity and requirement modules found; PRD/Epics define provenance and verification expectations                                  | Partially verified; data-source refresh and full workflow require testing               |
| Evidence extraction          | Extract source-linked claims and expose review states              | Evidence/document-processing modules found; upload and external provider behaviour depend on configuration                            | Partial/configuration-dependent; review and privacy workflow require acceptance tests   |
| Matching                     | Requirement-level comparison with evidence and gaps                | Matching modules include assessment and retrieval components; quality and complete integration not established by code presence alone | Partially implemented; verify outputs, provider availability, and evidence traceability |
| Scores/ranking               | Conflicting statements across PRD, Epics, and backend design       | Scoring-related code is present, while Epics prohibit numeric scores/ranking labels                                                   | Product decision required before UI exposure                                            |
| Partner suggestions          | Complementary partners, not popularity-based similarity            | Partner suggestion helpers are present in backend source                                                                              | Code path present; relevance, fairness, and consent expectations require tests          |
| Collective builder/workspace | Assemble collaborators and coordinate work                         | Collective, membership, workspace-related modules/models are present                                                                  | Backend support found; full frontend workflow not verified                              |
| Funder referrals             | Create and manage referrals                                        | Referral-related backend components are present                                                                                       | Backend support found; permissions, status transitions, and UI need tests               |
| Reports/email                | Deliver source-linked reports and notifications                    | Report/email-related code and configuration are present in project context                                                            | Provider configuration and successful delivery not verified here                        |
| Trend/gap analysis           | Surface funding trends and gap advice                              | Trend-related backend modules are present                                                                                             | Full data pipeline and frontend visualizations not verified                             |
| Security/privacy controls    | Private documents, access control, auditability, secure processing | Security utilities and access-related code exist; requirements document lists broader controls                                        | Must be checked against a route-by-route and deployment security checklist              |

### Status vocabulary

- **Implemented in code:** relevant code exists; this does not prove production readiness or acceptance.
- **Partially implemented:** some parts exist, but integration, coverage, quality, or supporting infrastructure remains incomplete or unverified.
- **Planned / specified:** described in the product or security requirements without sufficient implementation evidence in the reviewed source.
- **Not verified:** evidence available in this documentation pass is insufficient to make a reliable claim.
- **Requires decision:** product or technical sources conflict and an authorised owner must resolve the conflict.

## 11. Key limitations and open decisions

1. **Frontend source and runtime validation:** review the frontend repository, route inventory, API client, authentication state, environment variables, and actual deployed flows. The supplied signup URL could not be loaded during this pass.
2. **Scoring and ranking policy:** reconcile the Product Requirements Document with the Epics before exposing numeric scores or ranked organisation labels.
3. **Requirement status vocabulary:** agree the canonical labels for met, partially met, not met, unknown, not checked, and technical error, and map backend schemas to the approved user-facing terminology.
4. **Evidence review and visibility:** verify that unreviewed claims and uploaded documents are not exposed beyond their permitted audience.
5. **AI and retrieval configuration:** document the exact model/provider, credentials configuration, retrieval implementation, output validation, failure behaviour, and version tracking used in each environment.
6. **Database state:** run and verify migrations against the target PostgreSQL database and test with the intended production driver.
7. **End-to-end workflows:** test registration through to authenticated dashboard access, opportunity discovery, evidence review, matching, partner suggestions, collective workflows, referrals, and report delivery.
8. **Security readiness:** perform route-level authorization testing, dependency and secret scanning, upload/SSRF testing where applicable, and deployment configuration review.
9. **Data source permissions:** confirm the legal basis, licence terms, access restrictions, robots.txt behaviour, and refresh policy for each public or third-party source.

## 12. Recommended validation plan

Before describing the MVP as production-ready, validate the complete user journey rather than testing the API in isolation:

1. Open the hosted sign-up page and verify layout, required fields, client-side validation, API errors, and success behaviour.
2. Register and authenticate test organisation and funder accounts; confirm the correct role-specific landing experience.
3. Verify that a user cannot access another organisation's private profile, evidence, documents, reports, or collective workspace by changing identifiers.
4. Create or retrieve an opportunity and confirm that provenance, verification state, deadline, and unknown fields are shown accurately.
5. Process representative documents and confirm every extracted claim links to its source and displays the correct review/visibility status.
6. Run matching against representative evidence, including positive, partial, conflicting, missing, and technically failed cases. Confirm the result does not turn missing evidence into a negative fact.
7. Test complementary partner suggestions for evidence relevance and ensure connection counts, popularity, and network size do not affect discovery.
8. Test collective coverage and invitations, referral state changes, report generation, and email delivery with approved test accounts.
9. Run backend tests against PostgreSQL as well as isolated test databases, and run frontend-to-backend integration tests against the deployed environment.
10. Complete the security and privacy review required by the project's Security and Privacy Requirements document.

## 13. Glossary

| Term                     | Meaning in KNode                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| Organisation / NGO / NPO | A nonprofit organisation whose mission, capabilities, programmes, and evidence can be represented in KNode         |
| Funder                   | A foundation, donor, agency, or programme offering or managing funding opportunities                               |
| Funding opportunity      | A grant or funding call with eligibility conditions, requirements, deadlines, and a source                         |
| Evidence ledger          | Records of claims and their source, status, and relevant verification/review information                           |
| Requirement assessment   | An evidence-based view of how the available information relates to one opportunity requirement                     |
| Complementary partner    | An organisation whose demonstrated capabilities may address a gap or add a needed contribution                     |
| Collective / consortium  | A proposed group of organisations working together around an opportunity                                           |
| Referral                 | An introduction of an organisation or collective to a funder, opportunity, or partner; not an endorsement or award |
| Unknown / missing        | Information that is unavailable, unsupported, or not assessed; it is not automatically proof of incapability       |
| Decision support         | Information intended to help a person make a decision, without replacing the person's judgement                    |

## 14. Source documents and related technical pages

This overview synthesizes the following project sources:

- **Alya Product Requirements Document** - product problem, personas, intended features, user flows, AI boundaries, and acceptance expectations. [Alya PRD](https://docs.google.com/document/d/1SnDUWyaqpQn6qto0l3FhOfaXV68t53vlg-IahBFJmC4/edit?tab=t.0)
- **Alya Epics** - prioritised product behaviours and acceptance criteria, including fairness, evidence states, privacy, and the prohibition on numeric trust/risk scores or ranking labels. [Alya Epics](https://jmainamercy.atlassian.net/jira/software/projects/AFP/boards/100/backlog)
- **Alya Data Model** - intended entities, relationships, status conventions, provenance, and persistence requirements. [Alya Data Model](https://docs.google.com/document/d/1IL2LW0MAJ8w4G44G8gHASjvoCF5_DEXZlWgZ30Mujio/edit?tab=t.v5bembk3ww3c#heading=h.v24apw3k80jz)
- **Alya Security and Privacy Requirements** - intended security, privacy, source-collection, audit, and operational controls. [Alya Security Requirements document](https://docs.google.com/document/d/1mWBRcvSwhOXP-jwyWXGbA8QizYxowvPNwhAjLR8o2Hc/edit?tab=t.0#heading=h.f5a41wqh2006)
- **KNode Backend Technical Documentation** - proposed and current backend architecture, modules, API behaviour, AI integrations, and implementation limitations.[Alya Backend Technical Documentation](https://docs.google.com/document/d/1cw60eD3mxYD7QDEuGNA00ISGBNeES6LKqsPkTp2tWHY/edit?tab=t.0)
- **KNode Frontend Technical Documentation** - [Alya Frontend Technical Documentation](https://docs.google.com/document/d/1tRqpXvkQxtaqKVMut1KsTFm55PbRKPrtbaA-9BQwin8/edit?tab=t.0#heading=h.xvkcfdleebgz)
- **KNode SAD** - [Alya Security Requirements document](https://lucid.app/lucidchart/d8b8a5e1-e092-4d29-83c0-0a202c6dba92/edit?invitationId=inv_278130e4-6b41-4e3d-9ff5-0ff720c45ab7&page=0_0#)
- **KNode SSAD** - [Alya Security Requirements document](https://miro.com/app/board/uXjVEcegNEg=/?share_link_id=531272794233)
- **Hosted frontend** - [Hosted dashboard](https://alya-frontend-dashboard.vercel.app/auth/signup)

For implementation detail, use the dedicated **Backend** page. Use the **Frontend Web**, **AI Service**, **Architecture**, and **Integrations** pages for their respective concerns as they are filled in and validated against the source repositories.

name: technical-documentation-review
description: Reviews technical documentation for completeness, accuracy, clarity, consistency, usability, security, and alignment with the actual project. Use when validating documentation before submission, reviewing documentation after project changes, or checking documentation against project files. This skill reviews and reports findings; it does not write or rewrite documentation.

# Technical Documentation Review Skill

## Purpose

Review technical documentation to determine whether an unfamiliar but technically competent reader can:

- Understand what the project is and why it exists
- Understand who uses it and what it does
- Understand how the system is structured
- Understand how components interact
- Set up and run the project
- Use or interact with its APIs where applicable
- Test and troubleshoot it
- Understand its security model
- Understand its AI/ML behavior where applicable
- Deploy and maintain it where applicable
- Understand its limitations and implementation status

The review evaluates the documentation against the **available project evidence** and reports what is missing, incorrect, unclear, inconsistent, unsupported, or unsafe.

It does not write, rewrite, or invent documentation.

# When to Use

Use this skill when:

- Reviewing documentation before submission
- Reviewing documentation before a project demonstration
- Reviewing documentation written by another contributor
- Checking documentation after implementation changes
- Checking whether documentation reflects the current implementation
- Reviewing a README, technical specification, API documentation, architecture documentation, developer guide, or project wiki
- Validating documentation against a repository or project files
- Performing a final documentation quality review

Do not use it to:

- Write or rewrite documentation
- Invent missing implementation details
- Redesign the architecture
- Select or replace technologies
- Perform a full code, security, UX, or performance audit
- Judge visual design, branding, fonts, layouts, or aesthetics

Code may be inspected **only to verify documentation claims**.

# Review Context and Evidence Rules

The AI is a **project-grounded reviewer**.

Its job is to determine:

> **Does this documentation accurately and sufficiently describe this specific project, based on the evidence available?**

It is not being asked:

> What would good documentation normally say?

## Allowed project context

Use only project-relevant material available for the review, such as:

- Documentation
- README files
- Source code
- Repository structure
- Configuration files
- Environment templates
- Dependency/package files
- Database schemas and migrations
- API definitions
- Tests
- Infrastructure/deployment configuration
- AI/ML model artifacts
- Model evaluation results
- Architecture diagrams
- Project specifications or issue documents explicitly included in the review

External knowledge may be used only to understand terminology or interpret technical material. It must not be used to establish facts about the project.

Do not use information from unrelated projects or assume that a framework's typical behavior exists in this project.

## Evidence hierarchy

When verifying a claim, prefer available evidence in roughly this order:

1. Runtime behavior, when available
2. Source code
3. Tests
4. Configuration/infrastructure
5. Database schema and migrations
6. API specifications
7. Dependency/package configuration
8. Model artifacts/evaluation results
9. Project specifications
10. Documentation

When sources conflict, identify the conflict and report it. Do not silently choose whichever source appears more convenient.

## Evidence status

Distinguish between:

- **Verified** — available evidence supports the claim
- **Contradicted** — available evidence conflicts with the claim
- **Partially verified** — some of the claim is supported, but some cannot be confirmed
- **Unverified** — the documentation makes a claim but sufficient evidence is unavailable
- **Missing** — required information is absent from the documentation
- **Not applicable** — the information genuinely does not apply to the project

Never turn an unverified claim into a factual statement.

If implementation evidence is unavailable, explicitly state the review boundary.

Example:

> Implementation accuracy could not be fully verified because source code and deployment configuration were unavailable.

# Core Review Criteria

Evaluate each relevant section for:

### Purpose

Does the section actually accomplish its intended purpose?

### Completeness

Does it contain the information a reader needs for that purpose?

### Accuracy

Do documented claims match available project evidence?

### Specificity

Does it describe the actual project rather than a generic implementation?

### Clarity

Can the intended reader understand it without guessing?

### Actionability

Can the reader actually perform the documented task?

### Traceability

Can important claims be connected to project evidence?

### Consistency

Does the section agree with the rest of the documentation and implementation?

### Security

Does it avoid exposing sensitive information and accurately describe important security behavior?

### Maintainability

Does it provide enough information for another developer to work on the project?

Do not judge documentation quality by word count. Concise documentation can be complete; long documentation can still be inadequate.

# Documentation Structure

Review these sections when applicable:

- Home
- Overview
- Architecture
- Backend / API
- Frontend Web
- Frontend Mobile
- Security
- AI Module
- Deployment
- Developer Guide

A section may be intentionally omitted when the project does not contain that component.

Distinguish between **missing**, **incomplete**, and **not applicable**.

# Section Requirements

## Home

The entry point should:

- Identify the project
- Briefly explain what it does
- State its primary purpose
- Orient a first-time reader
- Provide useful navigation
- Avoid unnecessary implementation detail

Flag misleading navigation, unsupported claims, inconsistent project naming, or planned features presented as implemented.

## Overview

The overview should explain:

- Problem being addressed
- Project purpose
- Target users
- Main use cases
- Core functionality
- System boundaries
- Important assumptions
- Relevant limitations

A reader should be able to answer:

- What is this?
- Who is it for?
- What problem does it address?
- What can I do with it?
- What does it not do?

Do not accept a feature list as a sufficient overview.

## Architecture

The architecture section should explain:

- Overall architecture
- Major components
- Component responsibilities
- Data flow
- Request/response flow where applicable
- Frontend/backend communication
- Database interaction
- Storage
- External services
- Authentication flow
- AI/ML flow where applicable
- Asynchronous/background processes where applicable
- Deployment relationships
- Important architectural decisions and assumptions

For diagrams, verify that:

- Components exist
- Names match the implementation
- Relationships are accurate
- Data flows are understandable
- External services are identified
- Diagram and prose agree

Flag vague architecture, missing major components, misleading data flow, or diagrams that contradict the implementation.

## Backend / API

Verify:

### Technology

- Language
- Framework
- Major libraries
- Database
- ORM/data-access layer
- Authentication
- External integrations

### Structure

- Entry point
- Routes/controllers
- Services
- Repositories/data-access
- Models/schemas
- Configuration
- Middleware
- Utilities
- Background processes where applicable

### API

Where applicable, verify:

- Endpoint categories
- HTTP methods
- Paths
- Parameters
- Request bodies
- Authentication requirements
- Responses
- Error behavior
- Relevant status codes

Complete endpoint details may live in a dedicated API reference.

### Database

Verify:

- Database type
- Main entities/tables
- Relationships
- Important constraints
- Migrations
- Seed data where applicable

### Running

Verify:

- Prerequisites
- Dependency installation
- Environment configuration
- Database setup
- Migration/setup commands
- Startup command
- Verification step

### Testing

Verify:

- Test framework
- Test locations
- How tests are run
- Relevant test categories
- Expected behavior

Flag documentation that references nonexistent endpoints, files, commands, configuration, or behavior.

## Frontend Web

Verify:

- Framework and language
- Build tooling
- Styling approach
- State management where applicable
- Forms
- Routing
- API integration
- Authentication
- Main pages
- Main user workflows
- Validation
- Loading/error states
- Testing
- Build process
- Deployment

### Project structure

Documented files and directories should correspond to the actual project.

### Authentication

Verify:

- Login
- Logout
- Session/token handling
- Protected routes
- Expiration where applicable
- Authentication errors

### API integration

Verify:

- Base URL/configuration
- Request mechanism
- Authentication mechanism
- Response handling
- Error handling

## Frontend Mobile

Review only when a mobile application exists.

Verify:

- Platform
- Framework
- Language
- Application structure
- Navigation
- API integration
- Authentication
- Local storage
- Offline behavior where applicable
- Synchronization
- Permissions
- Security
- Installation
- Build process
- Testing
- Deployment
- Relevant device-specific issues

Do not require mobile-specific material when the project has no mobile application.

## Security

The security section should accurately describe relevant protections for:

### Authentication

- Authentication mechanism
- Credential handling
- Password storage
- Sessions/tokens
- Expiration
- Account recovery where applicable

### Authorization

- Roles
- Permissions
- Protected resources
- Enforcement

### Data protection

- Sensitive data
- Encryption in transit
- Encryption at rest where applicable
- Password hashing
- Secret management
- Database protection

### Application security

Evaluate controls relevant to the actual architecture, such as:

- Input validation
- Authentication
- Authorization
- Rate limiting
- CORS
- CSRF where applicable
- Injection prevention
- File upload restrictions
- Error disclosure
- Dependency risks

### Integrations

Where applicable:

- External-service authentication
- API-key handling
- Webhook verification
- Service permissions

### Incident handling

Where applicable:

- Logging
- Monitoring
- Detection
- Response
- Credential rotation
- Recovery

Flag unsupported security claims, missing important security behavior, confusion between authentication and authorization, or exposed sensitive information.

# AI Module

Review only when AI/ML functionality exists.

The AI section should explain:

### Purpose

- What task the AI performs
- Where it fits in the product
- What input it receives
- What output it produces

### Model and method

Where applicable:

- Model type/name/version
- Architecture
- Training approach
- Fine-tuning
- Prompting
- Retrieval
- Embeddings
- Inference
- Preprocessing
- Postprocessing

### Data

Where relevant:

- Data source
- Format
- Labels
- Preprocessing
- Dataset size
- Training/validation/test strategy
- Data limitations

### Pipeline

The documented flow should reflect the actual implementation, for example:

```text
Input
→ preprocessing
→ model/retrieval
→ postprocessing
→ application logic
→ user-visible output
```

### Evaluation

Verify, where applicable:

- Evaluation dataset
- Metrics
- Results
- Baseline
- Per-class performance
- Known failure cases
- Evaluation limitations

Performance claims must be supported by project evidence.

### Boundaries

Documentation should distinguish between:

- AI predictions
- AI recommendations
- AI-generated content/explanations
- Deterministic application logic
- Retrieved information
- External data
- Human decisions

It should clearly state relevant limitations, failure modes, and human-review requirements.

Do not infer model behavior from its name.

Do not accept unsupported claims such as:

- Highly accurate
- Reliable
- Production-ready
- Safe
- Guarantees security
- Makes definitive decisions
- Provides professional advice
- Detects wrongdoing with certainty

For high-stakes applications, flag documentation that presents AI outputs as authoritative decisions without appropriate evidence or boundaries.

# Recommended refinements

- Add an explicit severity/risk taxonomy:
  - Critical — blocks safe use or verification (e.g., missing architecture, missing security model).
  - High — major gaps preventing developer or operator tasks (e.g., no API docs, no deployment instructions).
  - Medium — important but not blocking (e.g., missing examples, partial test coverage).
  - Low — minor clarity, formatting, or style issues.

- Define pass/fail criteria and a minimum evidence checklist per section (Home, Overview, Architecture, API, Deployment, Security, Tests).

- Provide an automated evidence-collector script to enumerate:
  - README, docs pages, mkdocs/nav, src/, api/, deployment files, CI configs, Dockerfiles, tests.
  - Produce a manifest of missing expected files.

- Support diff-based reviews: focus verification on files changed since the last accepted review or baseline commit.

- Require traceability: all significant claims should link to a file path or evidence item (code, test, config). Flag unsupported claims as Unverified.

- Automate link and example validation:
  - Run mkdocs build to detect broken links.
  - Lint code blocks and attempt to run safe, short runnable snippets in a sandbox where feasible.

- Provide templates/checklists for missing sections (headers only; no invented implementation content) to guide authors.

- Add lightweight static checks for security/privacy risks in docs:
  - Detect embedded secrets, PII, credentials, or insecure defaults.

- Run available unit/integration tests and report results; map failing or missing tests to documentation claims.

- Add AI/ML-specific checks:
  - Verify presence of model artifacts, evaluation metrics, dataset provenance, intended behaviour and monitoring instructions.

- Produce multiple report formats:
  - Concise summary, section-by-section full report with evidence pointers, and machine-readable JSON for CI.

- Integrate with CI:
  - Provide a job that runs the evidence collector, mkdocs build, link checks, and posts findings as PR annotations.
  - Support baseline/suppression for accepted gaps.

- Include provenance and audit metadata in reports:
  - Reviewer identity, timestamp, commit/ref, and snapshot of evidence used.

- Provide prioritized remediation steps (minimal actionable fixes) and example commands to reproduce verification.

- Add plain-language clarity and glossary checks; flag missing term definitions and accessibility issues.

- Add project policy hooks to allow maintainers to configure regulatory/compliance requirements and review strictness.

# Documentation scoring & rating

- Score scale: 0.0 — 10.0 (higher is better).
- Weighted categories (suggested):
  - Home 10%
  - Overview 10%
  - Architecture 15%
  - Backend/API 15%
  - Deployment/CI 10%
  - Security 10%
  - Testing 10%
  - Traceability/Maintainability 10%
  - Completeness/Consistency 10%

- Scoring rules:
  - Rate each category 0–10.
  - Apply weights and sum to compute overall score.
  - Apply score floor reductions for Critical missing items (e.g., missing Architecture or Security may cap max score to a low value until addressed).

- Report must include:
  - Numeric overall score (0.0–10.0).
  - Category subscores and weights.
  - 1–2 sentence rationale explaining the score and top remediation priorities.

Example (applied to minimal-evidence review):

- Overall score: 2.8 / 10.0
- Short rationale: Only a brief README and mkdocs inventory were present; architecture, API, deployment, security, and testing documentation were missing or unverifiable, causing multiple critical gaps.
- Quick remediation priorities: add Architecture, Backend/API reference, Setup & Run guide, Deployment/CI docs, Security section, and runnable tests; run mkdocs build and fix broken/missing pages.

# Report formats

- summary: 6–10 bullet high-level items with severity and one-line remediation.
- full: section-by-section findings, evidence pointers (file paths and line references where available), status (Verified/Contradicted/Partially Verified/Unverified/Missing), and prioritized remediation.
- json: machine-readable output including scores, categories, findings, evidence list, and provenance metadata.

# Automation artifacts

- evidence_collector.sh (enumerates expected files and produces a manifest).
- mkdocs_build_and_link_check.sh (runs mkdocs build, captures warnings, dead links).
- ci-review.yml (CI job to run the above and upload findings as artifacts or PR annotations).
- templates/ (header-only doc templates for Architecture, API, Deployment, Security, Tests).

# Integration & Usage notes

- The reviewer must always declare the evidence boundary at the start of each review (what was available to inspect).
- The reviewer should not invent missing implementation details; missing items must be flagged as Missing or Unverified.
- When code is available, map documentation claims to file paths, tests, or runtime evidence and include exact pointers.
- For AI/ML projects, if model artifacts are unavailable, mark AI/ML sections as Unverified and recommend artifact inclusion.

# Maintenance

- Keep the evidence-collector and CI jobs up to date as documentation layout and project conventions evolve.
- Periodically review severity mapping and scoring weights with maintainers to align with project risk tolerance.

# Change log

- v1.0 — original skill (baseline).
- v1.1 — added severity taxonomy, evidence collector recommendation, diff-based review support, traceability requirement, automated link/example validation, templates, security/privacy checks, test-run verification, AI/ML checks, multiple report formats, CI integration, provenance metadata, remediation guidance, scoring system, example score, and automation artifact suggestions.

# Licensing & Ethics

- Do not expose secrets or PII in reports.
- Follow repository contributor policies and applicable regulations when surfacing security/privacy issues.

# Deployment

Verify:

- Hosting platform
- Deployment targets
- Frontend hosting
- Backend hosting
- Database hosting
- External services
- Environment variables
- Build process
- Deployment process
- Production configuration
- Domain configuration where relevant
- Service communication
- Health checks
- Logging
- Monitoring
- Release process
- Rollback/recovery where applicable

Check development versus production differences, including:

- Databases
- API URLs
- Secrets
- Debug configuration
- CORS

Deployment instructions should be actionable and consistent with the actual infrastructure.

# Developer Guide

The developer guide should enable another developer to work on the project.

Verify:

### Setup

- Prerequisites
- Repository structure
- Installation
- Environment configuration
- Local services
- Database setup
- Running the system

### Development

- Branching workflow where applicable
- Commit expectations
- Pull requests where applicable
- Code standards
- Formatting/linting
- Testing

### Debugging

- Common errors
- Logs
- Debugging locations
- Configuration problems

### Contribution

A contributor should be able to determine:

- Where to make changes
- How to test changes
- How to verify behavior
- What documentation or configuration must change with functionality

### Glossary

Explain project-specific acronyms and terminology where necessary.

# Cross-Section Consistency

After reviewing individual sections, compare related information across the documentation.

Check consistency of:

- Project name and description
- Users and roles
- Features
- Technologies
- Components
- File paths
- APIs/endpoints
- Database terminology
- Authentication
- Authorization
- Architecture
- AI behavior
- Deployment
- Environment configuration

Pay particular attention to:

**Architecture ↔ Backend ↔ Frontend ↔ Deployment**

**Authentication ↔ Frontend ↔ Backend ↔ Security**

**AI claims ↔ Architecture ↔ Backend ↔ AI Module ↔ Evaluation**

Do not flag intentional high-level summaries as duplication.

# References and Operational Details

## Links

### Internal links

Verify:

- Target exists
- Path is correct
- Anchor/reference works
- Removed pages are not referenced

### External links

Verify:

- URL is valid
- Destination is relevant
- Service/dependency belongs to the project
- No sensitive information is exposed

Do not introduce unrelated external resources merely to compensate for missing project documentation.

## Files and paths

Verify:

- File exists
- Directory exists
- Filename is correct
- Case is correct where relevant
- Path matches the project

## Commands

Verify where possible:

- Command matches project tooling
- Working directory is clear where necessary
- Required dependencies/configuration exist
- Required environment variables are documented
- Expected result is clear where useful

Do not invent corrected commands.

Do not execute destructive commands merely to verify documentation.

## Environment configuration

Documentation should distinguish:

- Required variables
- Optional variables
- Public configuration
- Secrets
- Development configuration
- Production configuration

Never require real secret values in documentation.

# Limitations and Status

Documentation should accurately distinguish:

- Implemented
- Partially implemented
- Experimental
- Planned
- Disabled
- Deprecated
- Unsupported

Flag planned functionality presented as existing functionality.

Check for relevant limitations involving:

- Supported platforms
- Unsupported workflows
- Data
- AI
- Offline behavior
- Performance
- Scale
- Security boundaries
- External dependencies
- Known bugs
- Incomplete features
- Experimental behavior

Do not require artificial limitations.

# Security and Privacy

Never reproduce sensitive information in the review output.

Flag:

- API keys
- Access tokens
- Passwords
- Private keys
- JWT signing secrets
- Database credentials
- Cloud credentials
- Service-account credentials
- Personal information
- Private URLs
- Sensitive infrastructure information

When reporting an exposure, identify the **type and location**, not the secret itself.

Example:

> A database credential appears in the deployment instructions.

Do not quote the credential.

Example placeholders such as `YOUR_API_KEY_HERE` are not exposed secrets.

# Guardrails

The reviewer must:

- Stay within the project scope
- Use project evidence for project-specific claims
- Never invent project facts
- Never treat generic best practices as evidence
- Never assume undocumented behavior exists
- Never treat planned functionality as implemented
- Distinguish missing, incorrect, and unverified information
- Report conflicting evidence rather than silently resolving it
- Avoid unsupported certainty
- Avoid over-flagging intentional differences
- Avoid under-flagging meaningful documentation failures
- Remain neutral and evidence-based
- Protect secrets and personal information
- Avoid turning the review into a code audit, architecture redesign, UX review, or technology-selection exercise
- Match review depth to project complexity
- Never equate documentation length with quality

The review process is:

> **Context → Evidence → Verify → Classify → Prioritize → Report**

not:

> **Assume → Infer → Rewrite**

# Review Process

## 1. Establish scope

Identify:

- Documentation being reviewed
- Intended project
- Available source code
- Configuration
- Infrastructure
- Tests
- Database information
- AI/model artifacts where applicable

State important evidence limitations before drawing conclusions.

## 2. Inventory documentation

Identify:

- Sections
- Links
- Referenced files
- Commands
- APIs
- Technologies
- Services
- Databases
- Models
- Deployment targets

## 3. Review sections

For each applicable section, evaluate:

- Purpose
- Completeness
- Accuracy
- Specificity
- Clarity
- Actionability
- Security
- Limitations
- Maintainability

## 4. Verify claims

Compare important claims against available project evidence.

Prioritize:

1. Security
2. Authentication/authorization
3. Incorrect implementation claims
4. Setup/run instructions
5. API behavior
6. Database behavior
7. Deployment
8. AI/model claims
9. User-visible functionality

## 5. Cross-check

Review:

- Cross-section contradictions
- Terminology
- Duplication
- References
- File paths
- Commands
- Configuration

## 6. Assess usability

Ask:

- Can a new developer understand the project?
- Can they set it up?
- Can they run it?
- Can they test it?
- Can they troubleshoot common problems?
- Can they find where to make changes?
- Can they deploy it when deployment is within scope?

## 7. Assign severity

Every finding receives a severity.

# Finding Categories

Use one or more of:

- Missing
- Inaccurate
- Unverified
- Ambiguous
- Inconsistent
- Unsupported claim
- Broken reference
- Security issue
- Privacy issue
- Scope issue
- Duplicate
- Outdated
- Incomplete instruction
- Missing limitation
- Placeholder
- Other

# Severity

### Critical

Serious security, correctness, operational, or misleading-documentation risk.

Examples:

- Exposed credentials
- Dangerous security misrepresentation
- Destructive deployment instructions
- Critical security control falsely documented as implemented

### High

Significantly prevents correct understanding, setup, development, or deployment.

Examples:

- Backend cannot be started using documented instructions
- Major endpoint is incorrectly documented
- Architecture materially contradicts implementation
- Important authentication behavior is incorrect
- Deployment procedure is unusable

### Medium

Meaningfully reduces clarity, usability, or maintainability without fundamentally preventing operation.

Examples:

- Important configuration is unclear
- Important limitation is missing
- Major component is insufficiently explained
- Terminology is inconsistent across important sections

### Low

Minor quality issue with limited impact.

Examples:

- Minor inconsistency
- Slightly unclear explanation
- Non-critical broken link

# Finding Quality

Each finding must be:

- **Specific** — identify the actual problem
- **Evidence-based** — identify supporting project evidence when available
- **Relevant** — explain why it matters
- **Actionable** — make clear what needs attention without rewriting the documentation
- **Non-speculative** — do not rely on assumptions

Use:

> "The documentation does not describe..."

> "The documented endpoint does not match the implementation..."

> "The claim could not be verified from the available project evidence."

Avoid:

> "The developer forgot..."

> "The author clearly..."

# Review Report

## Overall Assessment

Report:

- Overall status
- Major strengths
- Major risks
- Review boundary
- Confidence level
- Submission readiness

### Confidence

**High** — relevant documentation and implementation evidence were available and consistent.

**Medium** — substantial evidence was available, but some areas could not be verified.

**Low** — the review was primarily documentation-based or important implementation evidence was unavailable.

Confidence describes the **strength of the review evidence**, not the quality of the project.

## Section Results

For each applicable section:

### [Section Name]

- **Status:** Complete / Mostly Complete / Incomplete / Missing / Not Applicable
- **Purpose:** Met / Partially Met / Not Met
- **Accuracy:** Verified / Partially Verified / Unverified / Inconsistent
- **Actionability:** Usable / Partially Usable / Not Usable

### Findings

- **[Severity] [Category]** — Finding
  - Evidence:
  - Impact:

## Cross-Section Findings

Report:

- Contradictions
- Terminology inconsistencies
- Unnecessary duplication
- Conflicting architecture descriptions
- Conflicting API information
- Conflicting authentication information
- Conflicting AI information
- Conflicting deployment information

## Security Findings

Report separately.

Never reproduce secrets or sensitive values.

## Broken References

Report:

- Internal links
- External links
- File references
- Commands
- Endpoints

## Missing Information

List information that should be present but is not.

## Unsupported Claims

List material claims that cannot be verified.

# Final Verdict

Use one:

### Ready

Documentation is sufficiently complete, accurate, consistent, and usable for its intended purpose.

### Ready with Minor Fixes

Only low-severity issues remain.

### Needs Revision

One or more medium/high-severity issues should be addressed before submission.

### Not Ready

Critical or multiple high-severity issues prevent reliable use of the documentation.

# Final Quality Test

The review must ultimately answer:

1. Can a new reader understand what the project is?
2. Can a developer understand how it works?
3. Can a developer set up, run, and test it?
4. Can important documentation claims be verified against the available project evidence?
5. Can a developer maintain or deploy it without relying heavily on undocumented knowledge?

If any answer is no, identify the specific gap and its severity.

# Final Principle

The goal is not to make the documentation longer.

The goal is to make it **accurate, complete enough for its purpose, verifiable, usable, secure, and trustworthy**.

When evidence is unavailable:

> **Say that it cannot be verified.**

When evidence contradicts the documentation:

> **Report the contradiction.**

When information is missing:

> **Report the gap.**

When the documentation is correct:

> **Confirm it.**

## Never fill the gap with a plausible answer.
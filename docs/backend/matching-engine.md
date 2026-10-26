# Matching Engine

## 1. Purpose and boundaries

The Matching Engine assesses how an organisation's stored capabilities and evidence align with the requirements of a funding opportunity. It supports requirement-level assessments, eligibility conditions, a deterministic score calculation, partner suggestions and collective-coverage summaries.

The result is an evidence-based aid, not a funding prediction, certification or guarantee of eligibility. Missing evidence must not automatically be interpreted as proof that an organisation fails a requirement. Partner suggestions do not establish availability or agreement to participate.

## 2. Main components

| Component | Responsibility |
|---|---|
| `routers/matching.py` | HTTP interface for matching operations. |
| `services/matching_service.py` | Orchestrates matching requests and persistence. |
| `repositories/matching_repository.py` | Reads/writes match-related records. |
| `services/matching_engine/assessment.py` | Passage/provider protocol, retrieval inputs, status and quote validation. |
| `services/matching_engine/gemini_provider.py` | Gemini prompt, JSON response handling, retries and output checks. |
| `services/matching_engine/provider_factory.py` | Configures or disables the assessment provider. |
| `services/matching_engine/orchestrator.py` | Requirement loop, evidence filtering, scoring, eligibility and summary generation. |
| `services/matching_engine/eligibility.py`, `eligibility_rules.py` | Eligibility conditions and aggregate eligibility state. |
| `services/matching_engine/scoring.py` | Deterministic scoring and evidence-coverage calculation. |
| `services/matching_engine/contract.py` | Contract/status compatibility and output shapes. |
| `services/matching_engine/views.py` | Response views and partner suggestion logic. |
| `services/matching_engine/collective.py` | Aggregates requirement assessments across collective members. |
| `services/matching_batch_service.py`, `services/jobs/opportunity_matching.py` | Batch assessment and job entry point. |

## 3. Inputs and evidence retrieval

The request identifiers come from `schemas.matching.MatchCreateRequest`. Requirements are loaded from `opportunity_requirements`, including `requirement_text` and optional structured values such as `category`, `published_weight`, and `dedupe_key`.

The inspected assessment path filters evidence claims to `review_status == "CONFIRMED"` and non-empty `evidence_text`. A `Passage` carries claim/source/organisation identifiers and text. The lexical retrieval step tokenises text, keeps shared alphanumeric tokens of at least three characters, and sends at most five passages to the provider.

The following capabilities were **not found in this matching path** and should not be documented as implemented without further code evidence:

- Embedding generation.
- Vector similarity retrieval.
- Metadata-filtered hybrid keyword/vector retrieval.
- Page-level document chunking and a vector index.
- Vertex AI embeddings or pgvector-based matching retrieval.

The current candidate retrieval is lexical and limited; it may miss relevant evidence expressed with different wording. The quality and scope of retrieval should be evaluated before scaling.

## 4. Requirement assessment statuses

| Status | Meaning in the current contract |
|---|---|
| `MET` | Requirement assessed as fully met with an accepted, verified quote. |
| `PARTIALLY_MET` | Requirement assessed as partially met with an accepted, verified quote. |
| `NOT_MET` | The prompt instructs the provider to use this only for explicit contradiction, not merely missing evidence. |
| `UNKNOWN` | No relevant evidence, ambiguous output, unverified quote or insufficient certainty. |
| `NOT_CHECKED` | No provider is configured after relevant evidence is found; represents an assessment not performed. |
| `ERROR` | Provider failure or invalid output. |

A legacy contract maps `SUPPORTED` to `MET`, `PARTIALLY_SUPPORTED` to `PARTIALLY_MET`, and `UNVERIFIED` to `UNKNOWN`. The Gemini provider accepts `MET`, `PARTIALLY_MET`, `NOT_MET` and `UNKNOWN`. If the provider returns `NOT_CHECKED`, the wrapper converts it to `UNKNOWN`; the prompt does not ask the model to return `ERROR`. Preserve the distinctions in the application contract and avoid silently turning errors or missing evidence into negative results.

## 5. AI assessment and grounding

`GeminiAssessmentProvider` treats requirements and evidence as untrusted. It requests JSON containing a status, passage label, quote and short reasoning. It removes email-like strings and marker sequences, uses `response_mime_type=application/json` and temperature 0, and retries with exponential backoff. Logs include prompt version, attempt count, status, elapsed time and failure class rather than raw evidence prompts.

The wrapper validates the response using Pydantic, checks that the cited passage label maps to a retrieved claim, and checks that the quote exists in the stored passage after whitespace/case normalisation. An unverified citation is downgraded to `UNKNOWN`.

A valid quote does not prove that the model interpreted the evidence correctly. Human-labelled evaluation cases are still required, including contradictions, irrelevant passages, partial support and prompt-injection content.

## 6. Eligibility logic

`eligibility_rules.build_conditions()` currently handles these condition types:

- **Deadline:** if a deadline exists, compare it with current UTC time. A passed deadline is `FAILED`; otherwise it is `SATISFIED`.
- **Geography:** if eligible geographies are supplied, compare case-insensitive values against `organisation.countries_of_operation`. An overlap is `SATISFIED`; no overlap becomes `UNKNOWN` in the current implementation.
- **Organisation type:** if eligible organisation types are present, add an `UNKNOWN` condition. No mapping/evaluation logic is implemented in this path.

`assess_eligibility()`:

- Returns `NOT_ASSESSED` when no conditions exist.
- Changes `SATISFIED` or `FAILED` without evidence references/rule IDs to `UNKNOWN`.
- Returns `INELIGIBLE` if any effective condition failed.
- Returns `ERROR` if any condition is an error and no failed condition took precedence.
- Returns `ELIGIBLE` only when all conditions are satisfied.
- Returns `NEEDS_VERIFICATION` for remaining mixed/unknown states.

This is a limited rule set, not a comprehensive grant-eligibility engine. Confirm the product-required eligibility rules before presenting the result as a definitive eligibility decision.

## 7. Scoring formula

The deterministic scoring module defines these default category weights:

| Category | Weight |
|---|---:|
| `programme_strategic_alignment` | 0.25 |
| `programme_delivery_capability` | 0.25 |
| `experience_track_record` | 0.20 |
| `target_population_context` | 0.15 |
| `team_capacity` | 0.15 |

Status points are `MET` = 4, `PARTIALLY_MET` = 2, and `NOT_MET` = 0. The maximum is 4. Only these three statuses contribute to the score. An assessed status without evidence references is normalised to `UNKNOWN`.

When all requirements have positive published weights, those weights are normalised and used. If weights are absent or incomplete, default category weights are used and redistributed across applicable categories and requirements. Duplicate requirements are merged by `dedupe_key` or requirement ID; the first item is kept and conflicts produce warnings.

For assessed requirement *i*, with normalised weight *wᵢ*, status points *pᵢ*, and maximum points *pmax = 4*:

```text
Score = 100 × [Σ(i ∈ A) wᵢ × (pᵢ / pmax)] / [Σ(i ∈ A) wᵢ]
```

Here, *A* is the set of requirements the scoring module considers assessed. The score is rounded to one decimal place using `ROUND_HALF_UP`.

Evidence coverage is:

```text
Evidence Coverage = 100 × assessed weight / total applicable weight
```

- `COMPLETE`: all applicable weight was assessed.
- `PROVISIONAL`: some, but not all, applicable weight was assessed.
- `NOT_CALCULABLE`: no applicable requirement contributed to a score.

This formula describes the code, not an approved product policy. Confirm with the product owner whether a numeric Match Score and sorted results are permitted before treating them as a user-facing decision signal. The explanation text states that the score represents capability alignment, not funding probability or proof of eligibility.

## 8. Partner discovery

The current view logic:

1. Treats lead requirements in `UNKNOWN`, `NOT_CHECKED`, `ERROR`, `NOT_MET`, or `PARTIALLY_MET` as gaps.
2. Finds candidate assessments with `MET` or `PARTIALLY_MET` for the same requirement.
3. Emits a “may complement” label.
4. Returns candidate claim/source IDs for the relevant requirement.
5. Sets `participation_confirmed` to `false` and includes a disclaimer.
6. Sorts suggestions by the number of requirements they may cover, then by name and ID.

This is not an AI-driven semantic partner search. It uses stored assessment rows and deterministic coverage comparisons. Inspect candidate generation and the bulk-assessment path for performance and data-scope behaviour before scaling.

## 9. Collective coverage

`services/matching_engine/collective.py` aggregates requirement assessments per organisation. It tracks contributors and deduplicated claim IDs for each requirement.

- `COVERED`: at least one member has `MET`.
- `PARTIAL`: the best contributing status is `PARTIALLY_MET`.
- `UNRESOLVED`: no covered/partial result exists and unresolved member assessment states remain.
- `UNCOVERED`: all relevant available assessments are negative without unresolved states.

The summary counts covered, partial, uncovered, unresolved, and unconfirmed participants. This helper is not itself a provider call. The separate `services/coverage_service.py` describes its workflow as deterministic and states that it does not write `matches` or update `match_score`.

The application also has two coverage routes identified in the API inventory: a versioned matching-service response and an unversioned deterministic coverage response. Verify that both are intentional and document their different contracts.

## 10. Persistence and response construction

The schema stores:

- Workflow lifecycle in `match_runs`.
- Match identity and assessment result in `matches`.
- Partner suggestions in `match_partners`.
- Requirement definitions in `opportunity_requirements`.
- Evidence provenance in `sources` and `extracted_claims`.

The response builder reads `score_breakdown`, `requirement_assessments`, `missing_information`, strengths/gaps, score status, evidence coverage, eligibility status, run status and algorithm version. It detects legacy placeholder records and avoids presenting a placeholder zero as a real score.

Verify transaction boundaries, rollback behaviour and whether all related records are written atomically when a provider or database operation fails.

## 11. Outstanding work and verification

- Evaluate all eligibility rules required by the product.
- Agree canonical assessment statuses and score policy with the product owner.
- Add evidence freshness/version handling and stronger retrieval where justified.
- Test quote grounding and contradiction cases against labelled examples.
- Verify per-route authorization and cross-tenant access.
- Verify PostgreSQL transaction behaviour.
- Test duplicate request and idempotency behaviour.
- Document whether each matching path is synchronous or batch-only.
- Establish reliable production scheduling for batch jobs.
- Test provider-disabled, missing-key, timeout, malformed JSON, retry, prompt-injection and cost-limit scenarios.
- Measure retrieval quality, false positives/negatives, score stability and coverage semantics before making product claims.

See [API Reference](api-reference.md) for endpoint contracts and [Testing and Operations](testing.md) for the recommended test matrix.

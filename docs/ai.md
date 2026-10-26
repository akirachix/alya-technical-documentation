# AI Service

KNode uses Gemini for three jobs: reading organisation information, reading funding calls, and checking evidence against requirements. Everything else, including scores and eligibility, is calculated by fixed rules.

## Where AI is used

| Entry point                          | Trigger                                  | What it does                                                               |
| ------------------------------------ | ---------------------------------------- | -------------------------------------------------------------------------- |
| `services/jobs/scraper.py`           | Background job                           | Finds an organisation's website and extracts its profile                   |
| `services/jobs/midnight_scraper.py`  | Scheduled job (10 organisations per run) | Finds an organisation's website and extracts its profile, in small batches |
| `services/jobs/process_grants.py`    | Background job                           | Extracts structured requirements from stored grants (50 per run)           |
| `services/jobs/midnight_grants.py`   | Scheduled job (10 grants per run)        | Pulls Africa-related grants from Grants.gov and extracts them              |
| `POST /opportunities/extract`        | Funder or admin uploads a file           | Returns a draft opportunity for review                                     |
| `POST /organisations/{id}/documents` | User uploads a document                  | Extracts profile facts into `PENDING` claims                               |
| Matching engine                      | Match request                            | Checks evidence against each requirement (off by default)                  |

The model is set by `GEMINI_MODEL` (default `gemini-3.8-flash`). Every call needs `GEMINI_API_KEY`.

## Calling Gemini

The backend makes three kinds of call. All three ask for JSON back.

=== "Client setup"

    `services/ai_service.py` builds one shared client. It uses an API key if present, and otherwise Vertex AI.

    ```python
    from google import genai

    def _build_client():
        if settings.GEMINI_API_KEY:
            return genai.Client(api_key=settings.GEMINI_API_KEY)
        if settings.GCP_PROJECT_ID:
            return genai.Client(
                vertexai=True,
                project=settings.GCP_PROJECT_ID,
                location=settings.GCP_REGION,
            )
        return None

    client = _build_client()
    ```

=== "Extract from text"

    Used for website text and funding calls. The response is forced into a Pydantic schema, and the temperature is low so results are stable.

    ```python
    from google.genai import types

    response = client.models.generate_content(
        model=settings.GEMINI_MODEL,
        contents=prompt,
        config=types.GenerateContentConfig(
            response_mime_type="application/json",
            response_schema=ExtractedEvidence,
            temperature=0.1,
        ),
    )
    data = json.loads(response.text)
    ```

    The scraped text goes inside `<untrusted_text>` tags, and the prompt tells the model to ignore any instructions found there.

=== "Extract from a file"

    `services/document_extraction.py` sends the uploaded PDF, image or text directly to Gemini and validates the reply against a schema.

    ```python
    client = genai.Client(api_key=settings.GEMINI_API_KEY)
    interaction = client.interactions.create(
        model=settings.GEMINI_MODEL,
        input=_input_items(kind, data),
        response_format={
            "type": "text",
            "mime_type": "application/json",
            "schema": ExtractedProfile.model_json_schema(),
        },
    )
    profile = ExtractedProfile.model_validate_json(interaction.output_text)
    ```

    If the reply fails validation, the upload is rejected and nothing is saved.

=== "Assess a requirement"

    `services/matching_engine/gemini_provider.py` uses temperature 0 and a separate system instruction. It also sets a timeout.

    ```python
    class GenAIJsonGenerator:
        def __init__(self, api_key, model, timeout_seconds=30):
            from google import genai
            from google.genai import types

            self._types = types
            self._model = model
            self._client = genai.Client(
                api_key=api_key,
                http_options=types.HttpOptions(timeout=int(timeout_seconds * 1000)),
            )

        def generate(self, system, user):
            response = self._client.models.generate_content(
                model=self._model,
                contents=user,
                config=self._types.GenerateContentConfig(
                    system_instruction=system,
                    temperature=0.0,
                    response_mime_type="application/json",
                ),
            )
            return response.text
    ```

## 1. Organisation extraction

=== "From a website"

    **Code:** `jobs/scraper.py`, `jobs/midnight_scraper.py`

    1. Organisations are loaded from the IRS tax-exempt file (up to 100 per run; health, livelihoods and Africa-related names).
    2. A Serper search finds the official website (social media and directory sites are skipped).
    3. The page is scraped. If it has under 200 characters, the website is discarded.
    4. Gemini extracts the mission, capabilities, countries and focus areas.
    5. If both a mission and capabilities came back, they are saved onto the organisation.
    6. The job waits 4 seconds between organisations.

    !!! warning "No review step on this path"
        Website results are written directly onto the organisation record. They do not create `PENDING` claims or a source record.

=== "From an uploaded document"

    **Endpoint:** `POST /organisations/{organisation_id}/documents` (multipart form, field `file`)

    - Accepts PDF, PNG and DOCX. DOCX is converted to text first.
    - Each fact becomes a `PENDING` claim linked to a source record.
    - The response returns `source_id`, `fields_updated`, `claims_added` and the updated organisation.
    - A person moves each claim to `CONFIRMED` or `DISPUTED`. Only `CONFIRMED` claims are used in matching.

**Prompt rules:** extract only what is explicitly stated, focus area must be `HEALTH` or `LIVELIHOODS`, and countries must be African.

**Cleaning:** HTML is stripped and lengths are capped (mission 200 characters, 10 capabilities, 15 countries). Any error returns an empty result, so a failed call never invents data.

From `tests/test_ai_pipeline.py`, for the text _"We provide clean water in Kenya."_:

```json
{
  "mission": "To provide clean water.",
  "capabilities": ["WASH"],
  "countries_of_operation": ["Kenya"],
  "focus_areas": ["HEALTH"]
}
```

## 2. Opportunity extraction

=== "From Grants.gov"

    **Code:** `jobs/midnight_grants.py`

    1. Searches the Grants.gov API for the keyword `Africa`, posted in the last 60 days (20 rows).
    2. Skips any grant already stored.
    3. Fetches the grant's description and sends it to Gemini.
    4. Saves it only if Gemini returned a title and at least one requirement.
    5. Creates the funder if it does not exist yet, then the opportunity and each requirement.
    6. Saved with source `GRANTS_GOV`, status `OPEN` and currency `USD`.

=== "From stored grants"

    **Code:** `jobs/process_grants.py`

    Processes up to 50 grants per run, 7 seconds apart. Opportunities that have a description but no requirements yet are extracted and updated.

=== "From an uploaded file"

    **Endpoint:** `POST /opportunities/extract` (funder or admin only)

    Accepts a PDF, DOCX or TXT up to 10 MB and returns a **draft** opportunity. Nothing is saved until the funder reviews it and creates the opportunity.

The model returns no title, which rejects the call, when it is for a non-African region, is not an open call organisations can apply to, or is already past its deadline. Amounts are copied as written and never converted. With no currency stated, both amounts are set to null. At most 30 requirements are kept, each with a type (`eligibility`, `geography`, `thematic_fit`, `financial_capacity`, `partnership` or `evidence`).

From `tests/test_opportunity_extract.py`:

```json
{
  "title": "Clean water grants",
  "funder_name": "Test Water Fund",
  "eligible_geographies": ["Kenya"],
  "eligible_organisation_types": ["NGO"],
  "grant_min": 20000.0,
  "grant_max": 100000.0,
  "currency": "USD",
  "deadline": "2099-03-01",
  "focus_areas": ["HEALTH"],
  "requirements": [
    {
      "requirement_text": "Be a registered nonprofit",
      "requirement_type": "eligibility",
      "is_mandatory": true
    }
  ]
}
```

## 3. Requirement assessment

**Code:** `matching_engine/gemini_provider.py`, `assessment.py`

Turned on with `AI_ASSESSMENT_ENABLED=true`. For one requirement, the system finds up to 5 of the organisation's `CONFIRMED` claims by keyword overlap and asks Gemini for a verdict. If nothing overlaps, no call is made and the result is `UNKNOWN`.

The prompt built for the requirement `youth digital skills programme`:

```text
<<<REQUIREMENT
youth digital skills programme
REQUIREMENT>>>

<<<EVIDENCE P1
We delivered a youth digital skills programme across Kenya in 2024. Contact: [email removed]
EVIDENCE>>>
```

The reply Gemini must give:

```json
{
  "status": "MET",
  "passage": "P1",
  "quote": "youth digital skills programme",
  "reasoning": "The passage states the programme was delivered."
}
```

**Safeguards**

- Possible statuses: `MET`, `PARTIALLY_MET`, `NOT_MET`, `UNKNOWN`. Unclear or missing evidence is `UNKNOWN`, never `NOT_MET`.
- The quote must appear in the cited passage. If not, the result becomes `UNKNOWN` and is flagged for review.
- Database IDs never reach the model, emails are masked, and `<<<` and `>>>` are removed from evidence.
- A failed call is retried up to 2 times. If it still fails, the result is `ERROR`, which is not counted as a finding.

## How a match is scored

Four steps. Only step 2 uses AI.

1. **Eligibility** (rules): deadline not passed, geography overlap, organisation type.
2. **Assessment** (AI, optional): each requirement gets a status.
3. **Score** (rules): weighted by category, out of 100.
4. **Explanation** (template): built from the score and its breakdown.

??? info "Eligibility rules"
| Condition | Result |
|---|---|
| Deadline passed | `FAILED` |
| Deadline not passed | `SATISFIED` |
| Geography overlaps the organisation's countries | `SATISFIED` |
| No geography overlap | `UNKNOWN` (not a failure) |
| Opportunity lists organisation types | `UNKNOWN` (not yet checked) |

    Overall: `INELIGIBLE` if any condition failed, `ELIGIBLE` if all are satisfied, otherwise `NEEDS_VERIFICATION`.

??? info "Score weights (match-score-v1)"
`MET` counts 4, `PARTIALLY_MET` counts 2, `NOT_MET` counts 0, out of 4.

    | Category | Weight |
    |---|---|
    | Programme strategic alignment | 25% |
    | Programme delivery capability | 25% |
    | Experience and track record | 20% |
    | Target population and context | 15% |
    | Team capacity | 15% |

    Unassessed requirements (`UNKNOWN`, `NOT_CHECKED`, `ERROR`) are left out of the score and lower the **evidence coverage**. Score status is `COMPLETE`, `PROVISIONAL` or `NOT_CALCULABLE`. If the funding call publishes its own weights, those replace the defaults.

### Worked example

Five requirements, one per category. Four are assessed, one has no evidence.

| Category                      | Status          | Weight | Contribution |
| ----------------------------- | --------------- | ------ | ------------ |
| Programme strategic alignment | `MET`           | 0.25   | 0.250        |
| Programme delivery capability | `PARTIALLY_MET` | 0.25   | 0.125        |
| Experience and track record   | `MET`           | 0.20   | 0.200        |
| Target population and context | `UNKNOWN`       | 0.15   | not scored   |
| Team capacity                 | `MET`           | 0.15   | 0.150        |

```
(0.250 + 0.125 + 0.200 + 0.150) / 0.85 × 100 = 85.3
```

Result: score **85.3**, status `PROVISIONAL`, evidence coverage **85%**. The unknown requirement is left out, not counted as a failure. The same inputs always give the same score.

## Configuration

| Setting                        | Default            | Purpose                                       |
| ------------------------------ | ------------------ | --------------------------------------------- |
| `GEMINI_API_KEY`               | none               | Enables all Gemini calls                      |
| `GCP_PROJECT_ID`, `GCP_REGION` | none               | Vertex AI fallback (`ai_service.py` only)     |
| `GEMINI_MODEL`                 | `gemini-3.8-flash` | Model used by every call                      |
| `AI_ASSESSMENT_ENABLED`        | `false`            | Turns on requirement assessment               |
| `AI_BULK_ASSESSMENT_ENABLED`   | `false`            | Allows assessment across many organisations   |
| `GEMINI_TIMEOUT_SECONDS`       | `30`               | Per-call timeout (assessment)                 |
| `GEMINI_MAX_RETRIES`           | `2`                | Retries with backoff (assessment)             |
| `AI_MAX_TEXT_LENGTH`           | `20000`            | Text sent to extraction is cut to this length |

## Guardrails

These controls apply to the AI itself: what goes into the model, what comes out, and what it is allowed to do.

=== "Before the model"

    - Scraped text, documents and evidence are wrapped in markers and labelled as untrusted. The model is told to ignore any instructions inside them.
    - Any marker text found inside the input is removed first, so content cannot close the block early.
    - Text sent for extraction is cut to `AI_MAX_TEXT_LENGTH` (20,000 characters).
    - Assessment prompts contain no database IDs and no organisation names. Evidence passages are labelled `P1`, `P2` and so on, and mapped back afterwards.
    - Email addresses in evidence are replaced with `[email removed]`.
    - Prompts tell the model not to guess. Missing information must be left out.
    - Assessment runs at temperature 0 and extraction at 0.1, so results stay stable.

=== "After the model"

    | Check | Behaviour |
    |---|---|
    | Forced schema | Replies must be JSON matching a Pydantic schema. A reply that fails validation is rejected and nothing is saved. |
    | Field limits | Mission 5,000 characters, up to 30 capabilities, 60 countries, 12 impact metrics, 20 other facts |
    | Number checks | Annual revenue cannot be negative. Currency must be three capital letters. |
    | Allowed values | Focus areas are limited to `HEALTH` and `LIVELIHOODS`. Other values are dropped. |
    | HTML | Tags are stripped from every text field |
    | Quotes | An assessment must quote the passage it cites. If the quote is not in that passage, the result becomes `UNKNOWN` and is flagged for review. |
    | No evidence, no verdict | An assessed status with no evidence reference is downgraded to `UNKNOWN`. |
    | Unclear evidence | Treated as `UNKNOWN`, never `NOT_MET`. `NOT_MET` needs a passage that contradicts the requirement. |
    | Grant checks | A call with no title is rejected. Past deadlines, non-African regions and non-open calls are rejected. |
    | Money | Amounts are copied as written. Nothing is converted or guessed. |

=== "Human oversight"

    - Facts the AI extracts from uploaded documents are saved as `PENDING` claims. A person must confirm or dispute each one.
    - Only `CONFIRMED` claims are used as evidence when the AI assesses a requirement.
    - A funding call extracted from an upload comes back as a draft. Nothing is saved until the funder reviews and submits it.
    - The AI does not write the match explanation or calculate the score. Both come from fixed rules, so they cannot contain claims the AI made up.
    - Assessment is switched off by default and must be enabled with `AI_ASSESSMENT_ENABLED`.

=== "When the AI fails"

    | Situation | What happens |
    |---|---|
    | No API key | Document extraction returns `503` (`AI_NOT_CONFIGURED`). Assessment is skipped. |
    | Call fails or times out | Document extraction returns `502` (`EXTRACTION_FAILED`). Assessment is retried up to 2 times with backoff, then recorded as `ERROR`. |
    | Reply is invalid | Rejected. No data is saved. |
    | Website extraction returns nothing usable | The website is discarded and no organisation fields are changed |
    | Logging | Only the error type is logged, not the document content |

    An assessment `ERROR` is not counted as a finding about the organisation. It is left out of the score and lowers the evidence coverage.

!!! danger "What the AI must never do" - Decide who receives funding - Verify an organisation on its own - Declare an organisation legitimate without evidence - Approve organisations or form binding partnerships - Make legal decisions - Guarantee that an organisation will qualify for or win a grant - Replace funder due diligence

## Related pages

- [Architecture](architecture.md)
- [Backend](backend.md)
- [Integrations](integrations.md)
- [Glossary](glossary.md)

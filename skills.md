# Agent Skills

Skills are modular capabilities. A production implementation should load only the skills required by the current task.

## Skill 1 — Normalize Request

**Goal:** Convert a natural-language request into explicit constraints.

**Inputs:** scenario, counts, currency, statuses, dates, data-shape requirements.

**Output:** typed/structured request plus assumptions and validation constraints.

**Guardrails:**
- Preserve explicit constraints.
- Surface contradictions.
- Do not invent business rules to resolve ambiguity.

## Skill 2 — Build Test-Data Plan

**Goal:** Identify the minimum entities and relationships required to exercise the scenario.

Steps:
1. Identify entities.
2. Identify relationships.
3. Separate authoritative and derived fields.
4. Identify required invariants.
5. Choose deterministic identifiers/seeds when reproducibility matters.

## Skill 3 — Generate Synthetic Data

**Goal:** Create a realistic but synthetic candidate.

Rules:
- Use explicit test identifiers.
- Never copy production PII.
- Preserve requested constraints.
- Create relationships deterministically.
- Derive financial values from trusted inputs.
- Keep generation independent from validation.

## Skill 4 — Validate Independently

**Goal:** Determine whether the candidate is fit for downstream QA execution.

Validate in layers:

1. Structural
2. Referential
3. Financial
4. Temporal
5. Data quality
6. Scenario/business constraints

Every issue should contain:

- rule ID;
- severity;
- layer;
- JSON/data path;
- human-readable explanation;
- blocking/non-blocking status.

## Skill 5 — Classify Failure

Use available evidence to distinguish:

| Type | Typical evidence |
|---|---|
| PRODUCT | UI/API behavior, reproducible business-rule mismatch |
| AUTOMATION | script/assertion/selector/request behavior |
| DATA | invalid fixture or invariant violation |
| GENERATION | agent-produced invalid candidate |
| ENVIRONMENT | infrastructure/dependency telemetry |
| CONFIGURATION | flags, tenant, routing, setup |

Classification is evidence-driven and may remain uncertain until more evidence is collected.

## Skill 6 — Bounded Repair

Repair only deterministic derived values supported by trusted inputs.

After each repair:

1. record the triggering rule;
2. apply the smallest change;
3. validate again;
4. stop after a bounded retry limit.

## Skill 7 — Test the Validator

Start from a known-valid baseline and mutate it with known defects.

Examples:

- wrong invoice total → expected financial-rule failure;
- unknown customer reference → expected referential failure;
- payment greater than total → expected financial-rule failure.

The evaluator passes only when the expected defect is detected by the expected rule.

## Skill 8 — Compose Context Efficiently

Load:

```text
instructions
+ relevant skills
+ minimum domain context
+ output schema
+ current request
+ targeted execution evidence
```

Track token counts, retries, latency, cost, and repair rate in production.

## Skill 9 — Produce Evidence-First Output

Return:

- request summary;
- generated candidate summary;
- validations performed;
- issues found;
- repairs attempted;
- evaluator status;
- final ACCEPTED/REJECTED decision;
- audit trail.

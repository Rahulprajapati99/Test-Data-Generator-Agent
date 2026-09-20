# Agent Design

## 1. Objective

Design an agent-assisted workflow that can generate synthetic financial test data while maintaining clear separation between **generation, validation, repair, and final acceptance**.

## 2. Logical architecture

```text
                    ┌─────────────────────┐
                    │   Test Data Request  │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Normalize / Plan    │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Generate Candidate  │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Independent         │
                    │ Validation          │
                    └──────┬───────┬──────┘
                           PASS    FAIL
                             │       │
                             │       ↓
                             │  Classify Failure
                             │       │
                             │       ↓
                             │  Bounded Repair /
                             │  Regeneration
                             │       │
                             │       └──────→ Validate Again
                             ↓
                    ┌─────────────────────┐
                    │ Quality Gate        │
                    │ ACCEPT / REJECT     │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │ Evidence + Audit    │
                    └─────────────────────┘
```

## 3. Separation of concerns

### Agent / orchestrator
Coordinates the workflow. It should not be the sole authority for correctness.

### Generator
Creates a candidate dataset from the request and authorized context.

### Validator
Independently checks the candidate against deterministic structural, relational, financial, temporal, and scenario rules.

### Failure classifier
Uses available evidence to decide whether the issue is most consistent with PRODUCT, AUTOMATION, DATA, GENERATION, ENVIRONMENT, or CONFIGURATION.

### Repair / regeneration step
May correct a candidate only when the correct value is derivable from trusted inputs. Otherwise it should reject or escalate.

### Evaluator
Tests the validator itself by starting from known-valid data and injecting known faults.

### Audit trail
Records request, candidate version, rules checked, failures, repairs, retry count, evidence, and final decision.

## 4. Why the validator is separate

A common failure mode in AI-assisted automation is allowing the same component that generated an artifact to determine that the artifact is correct.

This design intentionally breaks that circular dependency.

```text
Generator  ──produces──→ Candidate
                           │
                           ↓
Validator  ──checks────→ Decision
```

The generator can be probabilistic. The final acceptance rules should be as deterministic as the domain permits.

## 5. Quality Assurance design choices

- Explicit quality gates instead of trusting a green status.
- Independent validation rather than self-certification.
- Mutation-based testing of the validator itself.
- Evidence-first failure reporting.
- Bounded repair attempts to avoid agent loops.
- Clear authoritative-vs-derived data ownership.
- Synthetic-data-only policy.
- Modular context to reduce unnecessary token consumption.
- Golden cases for regression across prompt/model/rule changes.

## 6. Failure classification model

Classification should be evidence-driven rather than guessed from an error message alone.

| Class | Typical evidence | Default action |
|---|---|---|
| PRODUCT | API/UI behavior, reproducible business-rule mismatch | Preserve evidence; do not hide it by changing test data |
| AUTOMATION | assertion, selector, request, script, trace | Inspect test implementation |
| DATA | invalid fixture, broken relationship, inconsistent arithmetic | Repair only if derivable |
| GENERATION | agent produced an invalid candidate | Regenerate or reject |
| ENVIRONMENT | dependency outage, network, infrastructure telemetry | Stop and surface evidence |
| CONFIGURATION | flags, routing, tenant/setup mismatch | Surface evidence; do not silently mutate data |

Real classification requires execution evidence. The design does not claim that an LLM can always infer root cause from limited information.

## 7. Production evolution

A production implementation could add:

0. Human approval for high-impact actions.
1. TypeScript orchestration.
2. LLM-backed planning/generation through a provider adapter.
3. Model/prompt/version telemetry.
4. Token, latency, retry, and cost monitoring.
5. Golden regression sets.


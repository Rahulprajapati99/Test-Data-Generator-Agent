# Guardrails

Guardrails define what the agent **must not do**, even when a change would make the overall result look successful.

## 1. No fake green

A successful generation step, script exit code, or tool response is not proof of correctness.

The workflow requires an independent validation layer and known-bad mutation checks for critical rules.

## 2. No self-certification

The component that creates the data must not be the sole authority that declares the data valid.

```text
Generator ≠ Validator
```

## 3. Synthetic data only

Never use:

- production customer records;
- real PII;
- real payment details;
- secrets or credentials;
- proprietary examples copied into prompts.

Use clearly synthetic identifiers and values.

## 4. Authoritative facts cannot be silently rewritten

The agent must not change authoritative inputs merely to make validation pass.

Examples:

- requested currency;
- requested record count;
- entity relationships;
- payment amount;
- scenario status;
- environment/configuration.

When these are wrong, regenerate, reject, or request explicit correction.

## 5. Repair is bounded

Repair is allowed only when:

- the field is derived;
- the source values are trusted;
- the correct calculation is deterministic;
- the change is recorded;
- validation is rerun.

Never use an unbounded repair loop.

## 6. Evidence before classification

Do not confidently label a failure as PRODUCT, AUTOMATION, ENVIRONMENT, or CONFIGURATION without appropriate evidence.

Useful evidence may include:

- API response;
- UI state;
- browser trace;
- assertion details;
- request/response payload;
- service logs;
- network errors;
- environment telemetry;
- configuration values.

## 7. No destructive actions by default

An autonomous implementation should not:

- modify production data;
- send external messages;
- trigger irreversible transactions;
- change infrastructure;
- disable quality gates.

Those actions require explicit authorization and, where appropriate, human approval.

## 8. Context minimization

Do not provide the model with unnecessary historical conversation or unrelated domain data.

Use modular skills and targeted context to reduce token use while preserving correctness.

## 9. Auditability

Every accepted result should be explainable after the fact.

Record:

- request/version;
- context/skill versions;
- model/provider when applicable;
- generated-data version;
- validation rules run;
- failures;
- repairs;
- retries;
- final decision.

## 10. Escalate uncertainty

When evidence is insufficient to make a safe decision, the agent should **stop and surface the uncertainty** rather than inventing a conclusion.

# Agentic QA — Test Data Generation & Validation Design

A focused QA design exercise for an **AI/agent-assisted test-data workflow**.

The purpose is not to present a production agent implementation. It is to show how I would design the **instructions, skills, context, validation, and guardrails** required to make an agentic QA workflow reliable and auditable.

> **Core principle:** AI can propose or generate test data, but an independent validation layer decides whether the result is trustworthy.

## Why this exists

Our interview discussion around agentic QA raised a practical question:

> **How would you build an agent that creates test data while making sure the generated data is actually correct?**

This repository is my structured answer to that problem.

## Design flow

```text
Test-data request
       ↓
Request normalization
       ↓
Minimum test-data plan
       ↓
AI-assisted generation
       ↓
Independent validation
       ↓
   ┌───┴────┐
   │        │
 FAIL     PASS
   │        │
   ↓        ↓
Classify   Quality gate
failure       │
   ↓          ↓
Bounded     ACCEPT
repair /      │
regenerate    ↓
   │       Audit trail
   └──→ re-validation
```

## Repository contents

| File | Purpose |
|---|---|
| [`AGENT_DESIGN.md`](AGENT_DESIGN.md) | Overall architecture, separation of concerns, and production evolution path |
| [`context.md`](context.md) | Domain knowledge the agent is allowed to use |
| [`instructions.md`](instructions.md) | System-level behavior and decision logic |
| [`skills.md`](skills.md) | Modular capabilities the agent can load as needed |
| [`validation_rules.md`](validation_rules.md) | Independent correctness rules and expected evidence |
| [`guardrails.md`](guardrails.md) | Safety, anti-fake-green, scope, repair, and data-handling constraints |

## What I am demonstrating

### 1. QA ownership

The design treats QA as more than test execution. A trustworthy workflow needs to establish:

- what was requested;
- what was generated;
- what business and technical rules were checked;
- what evidence supports the result;
- whether a failure is actually in the product, automation, data, environment, or configuration.

### 2. Trustworthy automation

A green automation result is not treated as proof by itself.

The design explicitly requires validation of the validation layer through **known-bad test-data mutations**. For example:

```text
Valid baseline
   ├── mutate invoice total      → expected validation failure
   ├── break customer reference  → expected validation failure
   └── exceed invoice payment    → expected validation failure
```

This is the "test the test" concept discussed during the interview.

### 3. Agentic test-data generation

The agent is designed to separate:

**Generation** — create a candidate

from

**Validation** — independently decide whether the candidate is acceptable.

That separation prevents the generator from effectively saying, "I created it, therefore it is correct."

### 4. Token-conscious design

The context is deliberately modular:

```text
stable instructions
      + required skills
      + minimum domain context
      + output contract
      + current request
      + targeted evidence
```

A production implementation should measure prompt tokens, completion tokens, retries, latency, and cost rather than blindly optimizing for the smallest prompt.

## Scope and honesty

This is a **design artifact / proof of thinking**, not a claim that I have deployed an autonomous production QA agent.

The domain is generic and synthetic. It intentionally avoids proprietary application details, customer data, internal prompts, or employer architecture.

The same design can later be implemented with TypeScript, Playwright/API automation, an LLM provider, or database/fixture tooling.

## Suggested implementation path

If implemented as a production workflow, I would evolve it in this order:

0. Human approval for destructive or externally visible actions.
1. TypeScript agent/orchestrator.
2. Structured input/output schemas.
3. Deterministic validation engine.
4. LLM adapter for scenario planning/generation.
5. Execution evidence for failure classification.


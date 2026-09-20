# Validation Rules

These rules are the **independent quality gate** for the synthetic billing dataset.

A blocking violation means the candidate is not acceptable for downstream testing.

| ID | Layer | Rule | Example failure | Blocking |
|---|---|---|---|---|
| S001 | Structural | Required fields exist | `invoice.total` missing | Yes |
| S002 | Structural | Monetary values have valid precision | 10.123 | Yes |
| R001 | Referential | Invoice customer exists | Unknown `customer_id` | Yes |
| R002 | Referential | Line item invoice exists | Unknown `invoice_id` | Yes |
| F001 | Financial | `line_total = quantity × unit_price` | 2 × 10 = 25 | Yes |
| F002 | Financial | `subtotal = sum(line_total)` | Subtotal mismatch | Yes |
| F003 | Financial | `total = subtotal + tax` | Total mismatch | Yes |
| F004 | Financial | `amount_paid <= total` | Overpayment | Yes |
| F005 | Financial | `balance = total - amount_paid` | Balance mismatch | Yes |
| T001 | Temporal | `invoice_date <= due_date` | Due date before invoice date | Yes |
| D001 | Data quality | IDs are unique | Duplicate invoice ID | Yes |
| D002 | Data quality | Records are synthetic/test-only | Production-style identifier | Yes |
| C001 | Scenario | Requested invoice count is met | 2 requested, 1 generated | Yes |
| C002 | Scenario | Requested currency is preserved | CAD request returns USD | Yes |
| C003 | Scenario | Status semantics/distribution are correct | PARTIALLY_PAID with zero payment | Yes |

## Severity

- **BLOCKING:** candidate cannot proceed to downstream execution.
- **WARNING:** useful telemetry but not itself a reason to reject.

## Source of truth

### Authoritative

- IDs
- relationships
- requested counts
- status intent
- currency
- dates
- payment amount
- explicitly supplied tax values

### Derived

- line total
- subtotal
- invoice total
- balance

This distinction is critical for safe agentic repair.

## Required evidence

Each validation issue should provide:

```text
rule_id
layer
severity
path
observed_value
expected_condition
evidence
```

Example:

```text
rule_id: F003
layer: Financial
severity: BLOCKING
path: invoice.total
observed_value: 1075.00
expected_condition: subtotal + tax = 1050.00
```

## Testing the validator

A validator should be tested against a known-good baseline with controlled mutations.

The evaluator should verify both:

1. the mutation causes a failure;
2. the expected validation rule identifies the failure.

That second check is important because a test suite can fail for the wrong reason and still appear useful.

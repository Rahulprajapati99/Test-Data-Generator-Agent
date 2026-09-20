# Context Pack — Generic Financial / Billing Domain

This context is intentionally generic and synthetic. It contains no proprietary application details or real customer data.

## Purpose

Provide only the domain knowledge required to generate and validate realistic QA test data for a billing workflow.

## Entities

### Customer

| Field | Meaning | Source of truth |
|---|---|---|
| `customer_id` | Unique synthetic identifier | Authoritative |
| `customer_type` | Customer classification | Authoritative |
| `country` | Country code | Authoritative |
| `currency` | Scenario currency | Authoritative |

### Invoice

| Field | Meaning | Source of truth |
|---|---|---|
| `invoice_id` | Unique invoice identifier | Authoritative |
| `customer_id` | Customer relationship | Authoritative |
| `invoice_date` | Issue date | Authoritative |
| `due_date` | Payment due date | Authoritative |
| `currency` | Invoice currency | Authoritative |
| `subtotal` | Sum of line totals | Derived |
| `tax` | Tax amount supplied by scenario | Authoritative for this demo |
| `total` | Subtotal + tax | Derived |
| `amount_paid` | Payment amount | Authoritative |
| `balance` | Total - amount paid | Derived |
| `status` | OPEN / PARTIALLY_PAID / PAID | Authoritative |

### Invoice line item

| Field | Meaning | Source of truth |
|---|---|---|
| `line_id` | Unique line identifier | Authoritative |
| `invoice_id` | Invoice relationship | Authoritative |
| `description` | Synthetic line description | Authoritative |
| `quantity` | Quantity | Authoritative |
| `unit_price` | Unit amount | Authoritative |
| `line_total` | Quantity × unit price | Derived |

## Core invariants

1. Every referenced customer exists.
2. Every invoice line references an existing invoice.
3. IDs are unique within their entity type.
4. `line_total = quantity × unit_price`.
5. `subtotal = Σ line_total`.
6. `total = subtotal + tax`.
7. `amount_paid <= total`.
8. `balance = total - amount_paid`.
9. `invoice_date <= due_date`.
10. OPEN means zero payment and full balance.
11. PARTIALLY_PAID means payment is greater than zero and less than total.
12. PAID means payment equals total and balance is zero.
13. Requested count, currency, and status distribution must be preserved.

## Authority model

The distinction between **authoritative inputs** and **derived values** is intentional.

A safe repair can recompute a derived value when its inputs are trusted.

A repair should not rewrite an authoritative business fact merely to satisfy a validation rule.

Example:

```text
quantity = 2
unit_price = 10.00
line_total = 25.00   ← invalid derived value
```

The correct repair is to recalculate `line_total` as 20.00.

By contrast:

```text
requested_currency = CAD
record_currency = USD
```

The agent should not silently change the record to make validation pass. That is a scenario-generation error requiring rejection/regeneration or explicit correction.

## Data safety

All identifiers and values should be synthetic and clearly marked for test usage. No production customer data or personally identifiable information should be copied into the workflow.

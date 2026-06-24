# Support Report Skill

Use this skill to generate daily, weekly, or ad-hoc customer support reports.

## Daily Report

Include:

- total new tickets
- resolved tickets
- unresolved tickets
- urgent and critical tickets
- SLA breaches
- top complaint categories
- repeated customers or repeated issues
- recommended next actions

## Weekly Report

Include:

- total tickets
- trend by category
- recurring root causes
- team workload
- unresolved aging tickets
- SLA performance
- improvement suggestions

## Report Style

Use clear sections.

Keep internal chat reports compact.

Use tables only when they improve readability.

Do not include unnecessary PII.

## Data Integrity

Do not invent numbers.

If the ticket API or database does not provide enough data, state what is missing.

Separate confirmed facts from assumptions.

## Example Internal Daily Report

```text
Daily Support Report

Summary:
- New tickets: 12
- Resolved: 8
- Still open: 4
- Urgent/Critical: 2
- SLA breaches: 1

Top Categories:
1. payment_deducted_no_service: 5
2. service_delay: 3
3. technical_error: 2

Needs Attention:
- 1 urgent payment issue is close to SLA breach.
- Repeated complaints appeared for supplier-related delay.

Recommended Actions:
- Check payment callback logs.
- Review supplier response time.

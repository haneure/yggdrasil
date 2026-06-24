# SLA Classification Skill

Use this skill to classify support issue severity and SLA.

## Severity Rules

critical:

- many users affected
- payment/service issue affecting multiple customers
- security or privacy issue
- public escalation
- owner/manager/VIP escalation
- incident likely to damage business reputation

urgent:

- payment deducted but service not delivered
- refund request
- customer angry or threatening escalation
- repeated complaint from same customer
- failed transaction with enough evidence

normal:

- single-user issue
- delayed service
- product issue without public escalation
- technical issue with workaround

low:

- general question
- unclear report
- missing required information
- no immediate action needed

## SLA Targets

critical:

- first response: 15 minutes
- internal escalation: immediate

urgent:

- first response: 30 minutes

normal:

- first response: 4 hours

low:

- first response: 1 business day

## Output

```json
{
  "severity": "urgent",
  "sla_target": "30 minutes",
  "escalate": true,
  "reason": "",
  "confidence": "medium"
}
```

```



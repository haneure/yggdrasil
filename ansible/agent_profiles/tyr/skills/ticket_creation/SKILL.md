# Ticket Creation Skill

Use this skill when converting a validated complaint into a support ticket.

## Goal

Create or update a ticket using structured complaint data.

## Required Steps

1. Validate required fields.
2. Search for active duplicate tickets.
3. If duplicate exists, update existing ticket.
4. If no duplicate exists, create new ticket.
5. Assign category, severity, team, and SLA.
6. Produce internal notification text.
7. Produce customer-safe update text if needed.

## Required Ticket Fields

```json
{
  "title": "",
  "severity": "urgent",
  "category": "payment_deducted_no_service",
  "status": "new",
  "customer_identifier": "",
  "source_chat_id": "",
  "source_message_id": "",
  "issue_summary": "",
  "evidence": [],
  "missing_fields": [],
  "assigned_team": "",
  "next_action": "",
  "sla_due_at": "",
  "duplicate_of": "",
  "confidence": "medium"
}
```

## Action Output

Use one of these actions:

- create_ticket
- update_ticket
- ask_missing_info
- ignore
- escalate

Output:

```json
{
  "action": "create_ticket",
  "reason": "",
  "ticket_payload": {},
  "internal_message": "",
  "customer_message": ""
}
```

## Internal Message Format

🎫 New Ticket: <ticket_id>
Severity: <severity>
Category: <category>
Customer: <masked_customer>
Summary: <summary>
Assigned: <team>
SLA: <sla_due_at>
Next action: <next_action>

## Customer Message Rules

Customer messages must not include:

- internal severity
- internal assignment logic
- staff names
- backend errors
- AI/model/prompt references

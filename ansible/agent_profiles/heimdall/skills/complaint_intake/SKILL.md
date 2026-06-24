# Complaint Intake Skill

Use this skill when reading customer Telegram messages.

## Goal

Convert raw customer messages into structured complaint candidates.

## Complaint Indicators

Indonesian indicators:

- komplain
- kendala
- error
- gagal
- saldo kepotong
- belum masuk
- refund
- pending
- lama
- tidak bisa
- kecewa
- uang belum balik
- transaksi gagal

English indicators:

- complaint
- failed
- deducted
- not received
- refund
- error
- pending
- delayed
- cannot use
- disappointed

## Non-Complaints

Do not create complaint candidates for:

- greetings only
- jokes
- unrelated chat
- general discussion
- internal team chatter
- repeated message already handled
- questions that can be answered directly from SOP

## Required Output

When a message is a complaint, produce:

```json
{
  "is_complaint": true,
  "category": "payment_deducted_no_service",
  "severity": "urgent",
  "customer_name": "",
  "phone": "",
  "transaction_id": "",
  "order_id": "",
  "summary": "",
  "missing_fields": [],
  "suggested_customer_reply": "",
  "suggested_internal_note": "",
  "confidence": "medium"
}
```

When a message is not a complaint, produce:

```json
{
  "is_complaint": false,
  "category": "general_question",
  "severity": "low",
  "reason": "",
  "suggested_customer_reply": "",
  "confidence": "high"
}
```

## Missing Information

For most transaction complaints, required fields are:

- phone number or customer identifier
- transaction ID or order ID
- issue description

Ask only for missing required fields.

## Customer Reply Style

Keep customer replies short and polite.

Example:

"Baik kak, kami bantu cek ya. Mohon kirim ID transaksi atau nomor yang digunakan untuk transaksi tersebut."

Do not mention severity or internal workflow.

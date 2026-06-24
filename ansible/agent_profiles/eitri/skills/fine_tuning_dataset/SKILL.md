# Fine-Tuning Dataset Skill

Use this skill when preparing datasets for future model fine-tuning or evaluation.

## Goal

Convert reviewed real support cases into clean training or evaluation examples.

## Important Rule

Do not fine-tune directly unless explicitly asked.

Prefer this order:

1. improve SOUL.md
2. improve AGENTS.md
3. improve skills
4. add examples
5. create eval dataset
6. run prompt regression tests
7. fine-tune only after enough verified data exists

## Data Privacy

Mask or remove:

- phone numbers
- customer names
- email addresses
- transaction IDs
- order IDs
- addresses
- private messages not needed for the example

## Dataset Example

```json
{
  "input": "Saldo kepotong tapi pulsa belum masuk",
  "expected": {
    "is_complaint": true,
    "category": "payment_deducted_no_service",
    "severity": "urgent",
    "missing_fields": ["transaction_id", "phone"],
    "suggested_customer_reply": "Baik kak, kami bantu cek ya. Mohon kirim ID transaksi atau nomor yang digunakan."
  }
}
```

## Evaluation Case Format

```json
{
  "id": "complaint-payment-001",
  "input": "",
  "expected_category": "",
  "expected_severity": "",
  "expected_missing_fields": [],
  "notes": ""
}
```

## Quality Checklist

Before adding an example:

- input is realistic
- sensitive data is masked
- expected output is correct
- category uses allowed enum
- severity uses allowed enum
- missing fields are reasonable
- customer reply is safe

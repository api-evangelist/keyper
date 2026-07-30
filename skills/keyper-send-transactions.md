---
name: Report gate entry transactions to keyper
description: Send access / entry transactions for scanned keys back to keyper and handle warnings.
api: openapi/keyper-access-openapi.yml
operations: [sendTransactions]
---

# Report gate entry transactions to keyper

Use this when an Access Control System scans keys (ticket barcodes) at a gate and
must report the entry back to keyper.

## Auth

Send `Authorization: APPSECRET <secret>` or `Authorization: AUTHTOKEN <guid>` and
`Content-Type: application/json`. Base URL `https://api.keyper.io/access`.

## Steps

1. Collect scanned entries into a `transactions[]` array. Each transaction needs:
   - `key_id` (required) — the scanned key / ticket barcode.
   - `permission_id` (required) — the permission (e.g. event id).
   - `permission_state` (required) — currently `Used`.
   - `timestamp` (optional) — ISO 8601 with offset; defaults to now if omitted.
   - `gate` (optional) — `{ id, name }` of the gate.
2. Call `sendTransactions` — `PUT /v1/transactions` with the array in the body.
3. Read the `200` response `warnings[]`. Each warning has `key_id`,
   `permission_id`, `message` (e.g. "Ticket already set to used.").
   Surface warnings to the operator; they do NOT mean the batch failed.

## Rules

- Transactions for keys or permissions unknown to keyper are ignored silently.
- Transactions conflicting with the stored key state are ignored WITH a warning.
- There is no idempotency key — do not blindly resend a batch on timeout without
  reconciling. See conventions/keyper-conventions.yml.
- A `422` returns detailed `errors[]` (error_index / error_number / error_message);
  a `401` means the credential is invalid or expired. See errors/keyper-error-codes.yml.

---
name: Sync the keyper permission whitelist to an access control system
description: Incrementally fetch the Grant / Block permission list from keyper and apply it at the gate.
api: openapi/keyper-access-openapi.yml
operations: [getPermissions]
---

# Sync the keyper permission whitelist

Use this to keep an Access Control System (ACS) in sync with the Grant / Block
decisions keyper holds for each key (ticket).

## Auth

Send an `Authorization` header on every request, either:
- `Authorization: APPSECRET <secret>` (app secret), or
- `Authorization: AUTHTOKEN <guid>` (per-user auth token).

Base URL: `https://api.keyper.io/access` (sandbox: `https://sandbox.api.keyper.io/access`).

## Steps

1. Track the last permission id you successfully processed (start at 0 on first run).
2. Call `getPermissions` — `GET /v1/permissions?lastID={lastID}&count={count}`.
   - `count` max is 500 (default 50). Set `suppressMetadata=true` if you do not need seat/row/etc.
3. For each returned `permissions[]` item, apply `action` at the ACS:
   - `Grant` → allow the `key_id` for `permission_id`.
   - `Block` → deny the `key_id` for `permission_id`.
   - Show `action_message.message` at the reader when the ACS supports it.
4. Advance `lastID` to `last_available_id` from the response and repeat until
   you have caught up (returned `last_available_id` equals your `lastID`).

## Rules

- Only reset `lastID` to a lower value to intentionally re-request data.
- Errors return the keyper envelope (`error_number` / `error_message` / `errors`);
  a 401 means the credential is invalid or expired. See errors/keyper-error-codes.yml.
- Datetimes are ISO 8601 with timezone offset. See conventions/keyper-conventions.yml.

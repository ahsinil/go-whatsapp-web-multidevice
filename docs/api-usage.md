# API Usage Guide

This guide explains how to use the REST API end-to-end, including multi-device scoping, authentication, and endpoint groups.

> Full schema and request/response definitions live in `docs/openapi.yaml`.

## 1) Base URL and Auth

Default local server:

```bash
http://localhost:3000
```

If Basic Auth is enabled, include credentials in each request:

```bash
-u username:password
```

If base path is configured (example `--base-path=/gowa`), prepend every API route:

```bash
http://localhost:3000/gowa
```

## 2) Device Scoping (Important)

Most API routes are **device-scoped** and require identifying which WhatsApp device should execute the action.

You can scope requests with either:

- Header: `X-Device-Id: <device_id>` (recommended)
- Query param: `?device_id=<device_id>`

If exactly one device exists, the server can default to that device.

Example:

```bash
curl -X GET "http://localhost:3000/app/status" \
  -H "X-Device-Id: 628123456789@s.whatsapp.net"
```

## 3) Bootstrap Flow (First-time Setup)

1. Add/list devices.
2. Login device (QR or pairing code).
3. Confirm `/app/status` says connected/logged-in.
4. Start sending messages.

### 3.1 List devices

```bash
curl -X GET "http://localhost:3000/devices"
```

### 3.2 Add a device slot

```bash
curl -X POST "http://localhost:3000/devices" \
  -H "Content-Type: application/json" \
  -d '{"device_id":"my-device"}'
```

### 3.3 Login with QR (device-scoped)

```bash
curl -X GET "http://localhost:3000/app/login" \
  -H "X-Device-Id: my-device"
```

### 3.4 Login with pairing code

```bash
curl -X GET "http://localhost:3000/app/login-with-code?phone=628123456789" \
  -H "X-Device-Id: my-device"
```

### 3.5 Check status

```bash
curl -X GET "http://localhost:3000/app/status" \
  -H "X-Device-Id: my-device"
```

## 4) Endpoint Groups and Features

Below is the complete endpoint map grouped by feature area.

---

## App APIs

- `GET /app/login`
- `GET /app/login-with-code`
- `GET /app/logout`
- `GET /app/reconnect`
- `GET /app/devices`
- `GET /app/status`

Use these for authentication lifecycle and connection health.

---

## Device APIs

- `GET /devices`
- `POST /devices`
- `GET /devices/{device_id}`
- `DELETE /devices/{device_id}`
- `GET /devices/{device_id}/login`
- `POST /devices/{device_id}/login/code`
- `POST /devices/{device_id}/logout`
- `POST /devices/{device_id}/reconnect`
- `GET /devices/{device_id}/status`

Use these for explicit multi-device management.

---

## User/Profile APIs

- `GET /user/info`
- `GET /user/avatar`
- `POST /user/avatar`
- `POST /user/pushname`
- `GET /user/my/privacy`
- `GET /user/my/groups`
- `GET /user/my/newsletters`
- `GET /user/my/contacts`
- `GET /user/check`
- `GET /user/business-profile`

Example (check if number exists on WhatsApp):

```bash
curl -X GET "http://localhost:3000/user/check?phone=628123456789" \
  -H "X-Device-Id: my-device"
```

---

## Send APIs

- `POST /send/message`
- `POST /send/image`
- `POST /send/audio`
- `POST /send/file`
- `POST /send/sticker`
- `POST /send/video`
- `POST /send/contact`
- `POST /send/link`
- `POST /send/location`
- `POST /send/poll`
- `POST /send/presence`
- `POST /send/chat-presence`

### Text message example

```bash
curl -X POST "http://localhost:3000/send/message" \
  -H "Content-Type: application/json" \
  -H "X-Device-Id: my-device" \
  -d '{
    "phone": "628123456789",
    "message": "Hello from API"
  }'
```

### Text with ghost mentions

```bash
curl -X POST "http://localhost:3000/send/message" \
  -H "Content-Type: application/json" \
  -H "X-Device-Id: my-device" \
  -d '{
    "phone": "1203630xxxxxxxx@g.us",
    "message": "Standup starts now",
    "mentions": ["628111111111", "628222222222"]
  }'
```

### Media example (image)

```bash
curl -X POST "http://localhost:3000/send/image" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789" \
  -F "caption=sample image" \
  -F "file=@/path/to/image.jpg"
```

---

## Message APIs

- `POST /message/{message_id}/revoke`
- `POST /message/{message_id}/delete`
- `POST /message/{message_id}/reaction`
- `POST /message/{message_id}/update`
- `POST /message/{message_id}/read`
- `POST /message/{message_id}/star`
- `POST /message/{message_id}/unstar`
- `GET /message/{message_id}/download`

Example (react to message):

```bash
curl -X POST "http://localhost:3000/message/3EB0.../reaction" \
  -H "Content-Type: application/json" \
  -H "X-Device-Id: my-device" \
  -d '{"phone":"628123456789","reaction":"👍"}'
```

---

## Chat APIs

- `GET /chats`
- `GET /chat/{chat_jid}/messages`
- `POST /chat/{chat_jid}/label`
- `POST /chat/{chat_jid}/pin`
- `POST /chat/{chat_jid}/disappearing`
- `POST /chat/{chat_jid}/archive`

Example (list messages with filters):

```bash
curl -X GET "http://localhost:3000/chat/628123456789@s.whatsapp.net/messages?limit=20&offset=0&search=invoice" \
  -H "X-Device-Id: my-device"
```

---

## Group APIs

- `GET /group/info`
- `POST /group`
- `GET /group/participants`
- `POST /group/participants`
- `GET /group/participants/export`
- `POST /group/participants/remove`
- `POST /group/participants/promote`
- `POST /group/participants/demote`
- `POST /group/join-with-link`
- `GET /group/info-from-link`
- `GET /group/participant-requests`
- `POST /group/participant-requests/approve`
- `POST /group/participant-requests/reject`
- `POST /group/leave`
- `POST /group/photo`
- `POST /group/name`
- `POST /group/locked`
- `POST /group/announce`
- `POST /group/topic`
- `GET /group/invite-link`

Example (create group):

```bash
curl -X POST "http://localhost:3000/group" \
  -H "Content-Type: application/json" \
  -H "X-Device-Id: my-device" \
  -d '{
    "name": "Ops Team",
    "participants": ["628123456789", "628987654321"]
  }'
```

---

## Newsletter APIs

- `POST /newsletter/unfollow`

List of newsletters is available from user API (`GET /user/my/newsletters`).

---

## Chatwoot APIs

- `POST /chatwoot/sync`
- `GET /chatwoot/sync/status`
- `POST /chatwoot/webhook`

Use `POST /chatwoot/sync` to import message history; use status endpoint to track progress.

Detailed integration steps: `docs/chatwoot.md`.

## 5) WebSocket Usage

Route:

```text
/ws?device_id=<device_id>
```

Use this for realtime event streaming scoped to one device.

## 6) Webhook Usage

Configure webhook URL(s) using:

- `--webhook`
- `WHATSAPP_WEBHOOK`

Optional controls:

- `--webhook-secret` / `WHATSAPP_WEBHOOK_SECRET`
- `--webhook-events` / `WHATSAPP_WEBHOOK_EVENTS`
- `--webhook-insecure-skip-verify` / `WHATSAPP_WEBHOOK_INSECURE_SKIP_VERIFY`

Payload schemas and event examples: `docs/webhook-payload.md`.

## 7) Error Handling

API errors return a JSON envelope with non-200 status and error code/message.

Recommended client behavior:

1. Check HTTP status code first.
2. Parse JSON body for `code` + `message`.
3. Retry only idempotent operations (status/list/get) unless you intentionally handle duplicates.

## 8) API Testing Workflow (Suggested)

1. `GET /devices`
2. `POST /devices`
3. Device login (`/app/login` or `/app/login-with-code`)
4. `GET /app/status`
5. `POST /send/message`
6. `GET /chats`
7. `GET /chat/{chat_jid}/messages`

## 9) Related Docs

- OpenAPI schema: `docs/openapi.yaml`
- Feature catalog: `docs/feature-list.md`
- External app integration guide: `docs/integration-guide.md`
- Webhook payloads: `docs/webhook-payload.md`
- Chatwoot integration: `docs/chatwoot.md`

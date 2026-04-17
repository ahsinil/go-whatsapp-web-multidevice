# Integration Guide for External Apps

This guide explains how to connect your own applications (backend services, web apps, mobile apps, automation tools) to Go WhatsApp Web Multi-Device.

## 1) Integration Architecture

Typical architecture:

1. Your app (CRM/ERP/Bot/Backend)
2. This WhatsApp API service (`go-whatsapp-web-multidevice`)
3. WhatsApp network

Recommended communication pattern:

- **Your app → WhatsApp API**: call REST endpoints for send/query/control.
- **WhatsApp API → Your app**: configure webhooks to receive inbound events.
- Use **device scoping** for multi-account scenarios.

## 2) Prerequisites

- Running WhatsApp API service (Docker or binary)
- At least one connected device (logged in)
- Stable identifier strategy for your recipients/users (phone number + your internal user ID)
- Public webhook endpoint in your app (if you need incoming events)

## 3) Authentication and Device Scoping

### Authentication

If Basic Auth is enabled, every request from your app must include Basic Auth credentials.

### Device Scoping (critical)

For most endpoints, include one of:

- `X-Device-Id: <device_id>` header (recommended)
- `?device_id=<device_id>` query parameter

If you integrate multiple WhatsApp accounts, map each tenant/brand/business-unit in your app to a `device_id`.

## 4) Minimal Integration Flow

### Step A — Register/find device

```bash
curl -X GET "http://localhost:3000/devices"
```

Optionally create one:

```bash
curl -X POST "http://localhost:3000/devices" \
  -H "Content-Type: application/json" \
  -d '{"device_id":"sales-team-1"}'
```

### Step B — Login device (one-time or when session expired)

```bash
curl -X GET "http://localhost:3000/app/login" \
  -H "X-Device-Id: sales-team-1"
```

or pairing code:

```bash
curl -X GET "http://localhost:3000/app/login-with-code?phone=628123456789" \
  -H "X-Device-Id: sales-team-1"
```

### Step C — Send outbound messages from your app

```bash
curl -X POST "http://localhost:3000/send/message" \
  -H "Content-Type: application/json" \
  -H "X-Device-Id: sales-team-1" \
  -d '{
    "phone": "628123456789",
    "message": "Hello from our CRM app"
  }'
```

### Step D — Receive inbound messages in your app (Webhook)

Run service with webhook configured:

```bash
WHATSAPP_WEBHOOK="https://your-app.com/webhooks/whatsapp"
WHATSAPP_WEBHOOK_SECRET="super-secret"
```

Your app should verify `X-Hub-Signature-256` and then process JSON payload by `event` type.

## 5) Integration Patterns

## Pattern 1: Backend-to-Backend (recommended)

Use your backend server as integration layer:

- Validate requests from frontend
- Call WhatsApp API securely (never expose Basic Auth credentials in browser)
- Store message IDs returned from send endpoints
- Correlate webhook events to internal conversations/orders/tickets

Use when:

- You have a web/mobile frontend
- You need audit logs, retries, and permission checks

## Pattern 2: CRM / Ticketing integration

Common mapping:

- CRM contact ↔ phone number
- CRM conversation/ticket ↔ WhatsApp chat JID
- CRM agent/team ↔ `device_id`

Suggested events to consume:

- `message`
- `message.ack`
- `message.reaction`
- `message.revoked`
- `group.participants` (if group workflows)

## Pattern 3: Event-driven automation

Use webhooks + queue:

1. Receive webhook
2. Verify signature
3. Push to queue (Kafka/RabbitMQ/SQS/etc.)
4. Async workers process business logic
5. Workers call `/send/*` endpoints when needed

This pattern improves reliability and prevents timeout issues.

## Pattern 4: No-code/low-code tools

Use tools like n8n/Zapier-like middleware:

- Trigger: webhook from WhatsApp API
- Action: transform payload
- Action: call your app API or DB
- Optional action: call WhatsApp API to reply

## 6) Multi-App / Multi-Tenant Strategy

If you have multiple apps sharing one WhatsApp API instance:

- Assign a unique `device_id` per app/tenant/channel
- Store mapping table in your DB:
  - `app_id`
  - `tenant_id`
  - `device_id`
  - status / last health check
- Enforce mapping in your integration service so each app only uses allowed device IDs

## 7) Reliability Best Practices

- Check `/app/status` or `/devices/{device_id}/status` before critical sends.
- Retry only safe/idempotent reads automatically.
- For send retries, implement deduplication key in your app to avoid duplicate outbound messages.
- Log all request/response pairs for observability.
- Use timeout + exponential backoff for transient network errors.
- Prefer queue-based sending for high throughput.

## 8) Security Best Practices

- Enable Basic Auth and keep credentials in secret manager.
- Verify webhook signatures (`X-Hub-Signature-256`).
- Restrict inbound webhook endpoint by IP allowlist (if possible).
- Use HTTPS end-to-end.
- Avoid exposing direct WhatsApp API credentials to frontend/mobile clients.

## 9) Example: Node.js integration snippet

```js
import fetch from "node-fetch";

const BASE_URL = process.env.WA_BASE_URL || "http://localhost:3000";
const DEVICE_ID = process.env.WA_DEVICE_ID || "sales-team-1";

export async function sendText(phone, message) {
  const response = await fetch(`${BASE_URL}/send/message`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Device-Id": DEVICE_ID,
      Authorization: "Basic " + Buffer.from(`${process.env.WA_USER}:${process.env.WA_PASS}`).toString("base64"),
    },
    body: JSON.stringify({ phone, message }),
  });

  if (!response.ok) {
    const body = await response.text();
    throw new Error(`WhatsApp send failed: ${response.status} ${body}`);
  }

  return response.json();
}
```

## 10) What to read next

- `docs/api-usage.md` for endpoint-by-endpoint REST usage
- `docs/openapi.yaml` for full request/response schema
- `docs/webhook-payload.md` for event payload structure + signature verification
- `docs/chatwoot.md` for CRM-like integration example

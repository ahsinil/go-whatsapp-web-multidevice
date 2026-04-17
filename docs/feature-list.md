# Feature List

This document summarizes what this project can do and where to find detailed docs for each capability.

## Core Modes

- **REST API server** for WhatsApp automation (`./whatsapp rest`).
- **MCP server** for AI-agent/tool integrations (`./whatsapp mcp`).
- **Web UI** (embedded front-end) to test most features from browser.

## Multi-Device Management

- Register, list, inspect, and remove WhatsApp devices.
- Login each device with **QR** or **pairing code**.
- Reconnect/logout per device.
- Device-scoped API via:
  - `X-Device-Id` request header (preferred), or
  - `device_id` query parameter.
- Device-scoped WebSocket stream via `/ws?device_id=<id>`.

## Authentication & Runtime Controls

- Optional **Basic Auth** (`--basic-auth` / `APP_BASIC_AUTH`).
- **Base path** deployments (`--base-path`) for reverse-proxy subpaths.
- Host/port/debug controls.
- Trusted proxy support for reverse proxies.

## App / Session Lifecycle

- Login (QR or pairing code), reconnect, logout.
- Connection health/status endpoint.
- Public `/health` endpoint for liveness/readiness probes.

## Sending Messages

- Text messages, including:
  - Mentions (`@phone` in text),
  - **Ghost mentions** using `mentions` array,
  - `@everyone` expansion for groups.
- Media send:
  - Image,
  - Audio,
  - Video,
  - File/document,
  - Sticker (including conversion to WhatsApp-compatible WebP).
- Rich payloads:
  - Contact card,
  - Link preview payload,
  - Location,
  - Poll.
- Presence controls:
  - global presence,
  - chat typing/recording presence.

## Message Operations

- Revoke/delete message.
- React to message.
- Edit/update recent message.
- Mark message as read.
- Star / unstar message.
- Download media from stored messages.

## Chat & Inbox Operations

- List chats with pagination/filtering.
- Get chat messages with filter support.
- Label/unlabel chats.
- Pin/unpin chats.
- Archive/unarchive chats.
- Configure disappearing message timer.

## User/Profile Operations

- My profile info.
- Avatar get/change.
- Pushname update.
- Privacy status.
- List groups, newsletters, contacts.
- Check whether a phone number is on WhatsApp.
- Query business profile.

## Group Operations

- Create group.
- Join via invite link.
- Inspect group details and invite metadata.
- List/export participants.
- Add/remove/promote/demote participants.
- Handle participant join requests (approve/reject).
- Leave group.
- Set group photo/name/topic.
- Toggle lock and announce mode.
- Retrieve/reset invite links.

## Newsletter Operations

- List subscribed newsletters/channels.
- Unfollow newsletter.
- Receive newsletter webhook events.

## Webhooks

- Outbound webhook forwarding for WhatsApp events.
- Event filtering via `WHATSAPP_WEBHOOK_EVENTS`.
- Signed payload verification (`X-Hub-Signature-256`, HMAC SHA256).
- Optional TLS verification bypass for development (`WHATSAPP_WEBHOOK_INSECURE_SKIP_VERIFY`).
- See detailed payload schemas in `docs/webhook-payload.md`.

## Chatwoot Integration

- Inbound WhatsApp → Chatwoot message forwarding.
- Outbound Chatwoot → WhatsApp send bridge.
- History sync endpoint + sync status endpoint.
- Multi-device aware outbound routing with `CHATWOOT_DEVICE_ID`.
- See setup guide in `docs/chatwoot.md`.

## Automation Behavior Flags

- Auto-reply incoming messages.
- Auto-mark incoming messages as read.
- Auto-download media from incoming messages.
- Auto-reject calls.
- Configurable presence on connect (`available` / `unavailable` / `none`).

## Storage & Deployment

- SQLite by default.
- PostgreSQL supported via `DB_URI`.
- Docker/Docker Compose support.
- Multi-architecture release artifacts (ARM/AMD64).

## API Documentation Index

- OpenAPI spec: `docs/openapi.yaml`
- API usage guide: `docs/api-usage.md`
- Webhook payload schema/examples: `docs/webhook-payload.md`
- Chatwoot integration guide: `docs/chatwoot.md`

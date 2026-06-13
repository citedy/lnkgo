---
name: lnkgo
title: "Lnkgo Branded Tracked Links for AI Agents"
description: >
  Create branded tracked short links with QR codes, click analytics,
  and custom domains — all through a REST API. Control API is hosted
  at api.lnkgo.app; short links use lnkgo.app by default.
  CLI and REST API — one API for short links, QR, analytics.
  Lnkgo is the independent link-service product surface.
version: "1.0.1"
author: Lnkgo
tags:
  - url-shortener
  - branded-links
  - qr-codes
  - analytics
  - custom-domains
  - link-management
  - tracking
  - agent-tools
metadata:
  openclaw:
    requires:
      env:
        - LNKGO_API_KEY
    primaryEnv: LNKGO_API_KEY
privacy_policy_url: https://lnkgo.app/privacy
security_notes: |
  API keys (prefixed cls_live_) authenticate against the Lnkgo Control API
  endpoints only (api.lnkgo.app/v1/*). All traffic is TLS-encrypted.
  Keys can be revoked/suspended by operators during PH MVP.
  Never expose Shlink internals to the customer — all traffic goes
  through the Control API with Authorization: Bearer cls_....
---

# Lnkgo — Skill Instructions

**Connection:** REST API over HTTPS
**Control API Base URL:** `https://api.lnkgo.app`
**Default Short-Link Domain:** `lnkgo.app`
**Auth:** `Authorization: Bearer $LNKGO_API_KEY`
**Brand boundary:** use only the Lnkgo endpoints listed in this skill for the
customer-facing product surface.
**Repository boundary:** `citedy/lnkgo` is the public skill repository only. Do
not add or deploy the `lnkgo.app` static website from this repository.
**Forbidden customer-facing URLs:** do not use company/operator hostnames as the
Lnkgo API base, default short-link domain, examples, fallback, or install
target. Customer-facing Lnkgo work uses `https://api.lnkgo.app` for API calls
and `lnkgo.app` for default short links.

---

## Overview

**Lnkgo** lets you create branded tracked short links, read click analytics, generate QR codes, and manage custom domains through a REST API and npm CLI designed for AI agents.

### Why this exists

Most URL shorteners force you into a web UI. Lnkgo is built for agents:

- **CLI** — `lnkgo create --url https://example.com` — JSON output
- **REST API** — `POST /v1/links` with `Bearer cls_...`
- **Agent Skills** — this file — works in Claude Code, Cursor, Codex, OpenClaw

---

## When to Use

Use this skill when the user asks to:

- Create a branded short link or tracked URL
- Generate a QR code for a link
- Check how many clicks a link got
- Add a custom domain for branded links
- Check domain status or DNS setup
- Get a free API key for link management
- Automate link creation from agent workflows

---

## Instructions

### Setup (run once)

#### 1. Get an API key

If you don't have a saved `LNKGO_API_KEY`, run:

```bash
# Step 1: Request a verification code
curl -X POST https://api.lnkgo.app/v1/keys/request \
  -H "Content-Type: application/json" \
  -d '{"email": "your@email.com"}'

# Step 2: Verify and get your API key
curl -X POST https://api.lnkgo.app/v1/keys/verify \
  -H "Content-Type: application/json" \
  -d '{"email": "your@email.com", "code": "<code-from-email>"}'
```

The response contains your API key (starts with `cls_live_`). Save it:

```bash
export LNKGO_API_KEY="cls_live_..."
```

> **Important:** The raw key is shown only once. Store it securely.

#### 2. Verify it works

```bash
curl -H "Authorization: Bearer $LNKGO_API_KEY" \
  https://api.lnkgo.app/health
```

#### 3. Create your first link

```bash
curl -X POST https://api.lnkgo.app/v1/links \
  -H "Authorization: Bearer $LNKGO_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000" \
  -d '{"url": "https://example.com"}'
```

---

## Core Workflows

### Workflow 1: Create a Branded Link

```http
POST https://api.lnkgo.app/v1/links
Authorization: Bearer $LNKGO_API_KEY
Content-Type: application/json
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{
  "url": "https://example.com/my-long-url",
  "slug": "spring-campaign",
  "tags": ["campaign", "social"],
  "title": "My Campaign"
}
```

**Response:**

```json
{
  "id": "lnk_abcdefghijklmnopqr",
  "short_url": "https://lnkgo.app/abc123",
  "target_url": "https://example.com/my-long-url",
  "domain": "<platform default>",
  "title": "My Campaign",
  "tags": ["campaign", "social"],
  "created_at": "2026-05-24T10:00:00Z"
}
```

**Optional parameters:**

- `title` — human-readable label (max 200 chars)
- `tags` — array of strings for categorization
- `slug` — requested public short-code (3-64 chars; letters, numbers, `_`, `-`)
- `domain` — active verified custom domain only; omit it for default `lnkgo.app`
- `expires_at` — ISO datetime for link expiration
- `max_visits` — maximum click count before link deactivates

Slug policy: the shared `lnkgo.app` namespace reserves system, protected, and
premium slugs, including their `-`/`_` segments. Sandbox namespaces follow the
same protection. Verified custom domains allow more brand-specific and generic
premium slugs while still blocking system and unsafe terms.

CLI note: `lnkgo create --expire 12h` is supported as sugar for `expires_at`.
Allowed duration units are `m`, `h`, and `d` (`30m`, `12h`, `20d`).

---

### Workflow 2: Read Click Analytics

```http
GET https://api.lnkgo.app/v1/links/{link_id}/analytics
Authorization: Bearer $LNKGO_API_KEY
```

**Response:**

```json
{
  "link_id": "lnk_abcdefghijklmnopqr",
  "total_clicks": 142,
  "period": {
    "start": "2026-05-10T00:00:00Z",
    "end": "2026-05-24T00:00:00Z"
  },
  "clicks_by_date": [
    { "date": "2026-05-23", "clicks": 38 },
    { "date": "2026-05-24", "clicks": 27 }
  ]
}
```

- Email-verified starter accounts get safe starter analytics. Domain-verified
  launch accounts unlock **14-day analytics retention**.
- Clicks beyond the domain-verified 50,000/month soft cap still redirect, but
  analytics detail can degrade.
- CLI note: `lnkgo analytics <link-id> --csv` formats this aggregate analytics
  response as local CSV. It is not a raw click export and does not create a
  hosted report link.

---

### Workflow 3: Generate QR Code

```http
GET https://api.lnkgo.app/v1/links/{link_id}/qr
Authorization: Bearer $LNKGO_API_KEY
```

Returns the QR code image directly (PNG). Use with image display or download.

---

### Workflow 4: Manage Custom Domains

**Request a domain:**

```http
POST https://api.lnkgo.app/v1/domains
Authorization: Bearer $LNKGO_API_KEY
Content-Type: application/json

{
  "domain": "links.yourdomain.com"
}
```

**Response:**

```json
{
  "id": "domain_abc123",
  "domain": "links.yourdomain.com",
  "status": "pending_dns",
  "dns": {
    "type": "CNAME",
    "name": "links.yourdomain.com",
    "target": "edge.lnkgo.app"
  },
  "proof": {
    "type": "TXT",
    "name": "_lnkgo.yourdomain.com",
    "value": "lnkgo-verify=abc123def456"
  },
  "created_at": "2026-05-24T10:00:00Z"
}
```

**Check domain status:**

```http
GET https://api.lnkgo.app/v1/domains/links.yourdomain.com
Authorization: Bearer $LNKGO_API_KEY
```

**Status flow:** `pending_dns → pending_tls → active`

- Add the TXT record to your DNS — the system verifies automatically
- After DNS verification, Cloudflare provisions a TLS certificate
- Once TLS is active, you can create links with this domain

**Domain limits:**

- Email-verified starter accounts can hold **1 pending custom domain**.
- Domain-verified launch accounts can hold **10 custom domains**.
- Up to **3 domains** can be in `pending_dns` at once after domain proof.
- **7 days** in `pending_dns` → auto-release (domain freed, 30-day cooldown)
- **90 days** inactive with no clicks in last 30 days → email notice at 76 days → auto-release at 90 days

---

### Workflow 5: Get Domain Status

```http
GET https://api.lnkgo.app/v1/domains/{domain}
Authorization: Bearer $LNKGO_API_KEY
```

**Response:**

```json
{
  "domain": "links.yourdomain.com",
  "status": "active",
  "checked_at": "2026-05-24T10:00:00Z",
  "checks": {
    "dns": "verified",
    "tls": "active",
    "shlink": "registered"
  }
}
```

---

## Examples

### Example 1 — Create and Share

**User:** "Create a short link for https://example.com/pricing and give me the QR code"

```bash
# Step 1: Create link
curl -X POST https://api.lnkgo.app/v1/links \
  -H "Authorization: Bearer $LNKGO_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000" \
  -d '{"url": "https://example.com/pricing", "title": "Pricing Page"}'

# Response: {"id": "lnk_abcdefghijklmnopqr", "short_url": "https://lnkgo.app/abc123"}

# Step 2: Get QR code
curl -o qr.png https://api.lnkgo.app/v1/links/lnk_abcdefghijklmnopqr/qr \
  -H "Authorization: Bearer $LNKGO_API_KEY"
```

**Reply:**

> Done! Your short link: https://lnkgo.app/abc123
> QR code saved to qr.png — share it anywhere.

---

### Example 2 — Check Analytics

**User:** "How many clicks did my link get?"

```bash
curl -s https://api.lnkgo.app/v1/links/lnk_abcdefghijklmnopqr/analytics \
  -H "Authorization: Bearer $LNKGO_API_KEY" | jq
```

**Reply:**

> Your link has **142 clicks** in the last 14 days.
> Daily trend: 38 yesterday, 27 today.

---

### Example 3 — Add Custom Domain

**User:** "I want to use links.mycompany.com for my short links"

```bash
# Step 1: Request domain
curl -X POST https://api.lnkgo.app/v1/domains \
  -H "Authorization: Bearer $LNKGO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "links.mycompany.com"}'

# Step 2: Add TXT record to DNS (instructions in response)
# Step 3: Check status
curl -s https://api.lnkgo.app/v1/domains/links.mycompany.com \
  -H "Authorization: Bearer $LNKGO_API_KEY"
```

**Reply:**

> I've requested links.mycompany.com. Here's what to do:
>
> 1. Add this TXT record to your DNS: `_lnkgo → lnkgo-verify=...`
> 2. Wait a few minutes for DNS propagation
> 3. I'll check the status automatically
> 4. Once active, you can create links with `"domain": "links.mycompany.com"`

---

### Example 4 — CLI Usage

From any environment with the public CLI:

```bash
# Initialize
lnkgo init --email your@email.com

# Create link
lnkgo create --url https://example.com
lnkgo create --url https://example.com/launch --slug spring-campaign
lnkgo create --url https://example.com/temporary --expire 12h

# Get QR
lnkgo qr <link-id> --output qr.png

# Read analytics
lnkgo analytics <link-id>
lnkgo analytics <link-id> --csv

# Check domain
lnkgo domain status links.mycompany.com
```

All CLI commands return JSON by default — ready for agent chaining.

---

## API Reference

All endpoints require `Authorization: Bearer <api_key>`.
Base URL: `https://api.lnkgo.app`

Never substitute this with another product or operator domain in
customer-facing examples. Lnkgo is the independent link-service product.

### Authentication

| Endpoint           | Method | Auth     | Description                     |
| ------------------ | ------ | -------- | ------------------------------- |
| `/v1/keys/request` | POST   | None     | Request email verification code |
| `/v1/keys/verify`  | POST   | None     | Verify code → receive API key   |
| `/health`          | GET    | Optional | Service health                  |

### Links

| Endpoint                   | Method | Scopes           | Description          |
| -------------------------- | ------ | ---------------- | -------------------- |
| `/v1/links`                | POST   | `links:create`   | Create a short link  |
| `/v1/links/{id}`           | GET    | `links:read`     | Get link details     |
| `/v1/links/{id}/analytics` | GET    | `analytics:read` | Read click analytics |
| `/v1/links/{id}/qr`        | GET    | `qr:read`        | Get QR code image    |

### Domains

| Endpoint               | Method | Scopes            | Description           |
| ---------------------- | ------ | ----------------- | --------------------- |
| `/v1/domains`          | POST   | `domains:request` | Request custom domain |
| `/v1/domains/{domain}` | GET    | `links:read`      | Get domain status     |

### Idempotency

All `POST /v1/links` requests require the `Idempotency-Key` header (UUIDv4):

```http
POST https://api.lnkgo.app/v1/links
Authorization: Bearer $LNKGO_API_KEY
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{"url": "https://example.com"}
```

- Replaying the same key returns the existing link (200), not a duplicate
- Key scope is per-account (keys from different accounts do not collide)
- Keys expire after 24 hours

---

## Launch Limits

Email verification starts with safe starter limits. Proving domain ownership
unlocks the Product Hunt launch offer.

| Resource            | Email verified starter | Domain-verified launch offer | When exceeded                      |
| ------------------- | ---------------------: | ---------------------------: | ---------------------------------- |
| Custom domains      |       1 pending domain |               10 per account | 422 + `Retry-After`                |
| Links/month         |                    100 |                       10,000 | 422 + `Retry-After`                |
| Link creates/hour   |                     10 |            plan/rate limited | 429 or 422 + `Retry-After`         |
| Links/domain/month  |                    n/a |                 1,000/domain | 422 + `Retry-After`                |
| API calls/month     |           starter safe |                       50,000 | 422 + `Retry-After`                |
| Clicks/month        |           starter safe |              50,000 soft cap | Analytics degrades, redirect works |
| Analytics retention |          24h aggregate |                      14 days | -                                  |
| API rate            |             60 req/min |                   60 req/min | 429 + `Retry-After`                |
| Trusted rate        |                    n/a |              up to 1,000/min | Configurable per key               |

---

## Rate Limits

| Type                        |           Limit | Response            |
| --------------------------- | --------------: | ------------------- |
| Per-key API                 |      60 req/min | 429 + `Retry-After` |
| Per-IP (unauthenticated)    |      10 req/min | 429 + `Retry-After` |
| Trusted rate (configurable) | up to 1,000/min | Configurable        |

Rate limit headers:

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
- `Retry-After` (on 429/422/503)

---

## Error Handling

| Code | Meaning                   | Action                                              |
| ---- | ------------------------- | --------------------------------------------------- |
| 400  | Invalid request           | Check parameters (URL format, missing fields)       |
| 401  | Missing/invalid API key   | Check `LNKGO_API_KEY` is set                        |
| 403  | Account suspended/revoked | Check key/account status with an operator           |
| 404  | Resource not found        | Verify link ID or domain                            |
| 409  | Idempotency conflict      | Different account already used this key             |
| 422  | Quota exceeded            | Wait `Retry-After` seconds or upgrade plan          |
| 429  | Rate limited              | Wait `Retry-After` seconds, then retry              |
| 500  | Server error              | Retry once after 5s, then report                    |
| 503  | Overloaded                | Wait `Retry-After` seconds, then retry with backoff |

All errors return a consistent schema:

```json
{
  "error": {
    "type": "quota_exceeded",
    "detail": "Monthly link limit reached: 10,000/10,000",
    "retry_after": 86400
  }
}
```

---

## Response Guidelines

- **Reply in the user's language** — match their language
- **Always show the short URL** after creation — that's what the user needs
- **Show analytics as readable summary** — not raw JSON. Bullet points with total clicks and daily trend
- **For QR codes** — save and present the image, or provide the download URL
- **For domain requests** — give clear DNS instructions, offer to check status automatically
- **On quota/rate limit errors** — explain what limit was hit and when it resets
- **Require idempotency** — every link create request must send `Idempotency-Key` to prevent duplicates
- **Offer CLI command** — `lnkgo ...` or `npx lnkgo ...`

---

## CLI Availability

```bash
npm install -g lnkgo
lnkgo init --email your@email.com
export LNKGO_VERIFICATION_CODE="<email-code>"
export LNKGO_API_KEY="$(
  lnkgo init \
    --email your@email.com \
    --code "$LNKGO_VERIFICATION_CODE" | jq -r '.api_key'
)"
lnkgo create --url https://example.com
```

Use the REST API directly when the environment cannot install npm packages.

---

## Limitations

- Email-verified starter accounts are intentionally low-limit to prevent abuse.
- Domain-verified launch accounts unlock 10 custom domains, 10,000 links/month,
  1,000 links/domain/month, 50,000 clicks/month soft cap, and 14-day analytics.
- Maximum link title: 200 characters
- Maximum tags: 20 per link
- Idempotency-Key TTL: 24 hours
- Custom domains require DNS TXT proof and TLS provisioning (may take a few minutes)
- Domain staleness: 7 days pending auto-release, 90 days inactive auto-release
- No bulk-create endpoint yet (use idempotent single-link creates with retries for high volume)

---

_Lnkgo Skill v1.0.0_
_https://lnkgo.app_

# Security & Data Handling

This document describes how AgentDo authenticates API calls, what data we store when a booking or order is placed, and how we handle personal data under GDPR / NAIH (the Hungarian data-protection authority).

If you're integrating AgentDo into an AI agent, your customers' booking data passes through our system — we want to be explicit about what happens to it.

## 1. Authentication

### Inbound (you → AgentDo)

All REST and MCP requests authenticate with an API key:

```
X-API-Key: ak_your_key_here
```

Keys are 32 bytes of CSPRNG randomness encoded as `ak_<hex>`. On creation the key is shown **once**; we store only a SHA-256 hash. If you lose a key, rotate — we cannot recover the plaintext.

Keys are per-user (developer account), and every privileged action (cancel, status) is **scoped to the calling key's history**: a key can only cancel or read bookings it originally created. You cannot enumerate other users' data even with a valid key.

Rate limit: 1000 requests / hour / key. 429 on exhaustion with `Retry-After` header. Contact us for higher limits.

### Outbound (AgentDo → provider)

When a booking or order is created, AgentDo forwards the event to the provider's webhook endpoint, signed with HMAC-SHA256:

```
X-AgentHub-Sig: <sha256_hex>
X-AgentHub-Ts:  <unix_ms>
X-AgentHub-Event: booking.created
```

The signature is computed as `hmac_sha256(provider_secret, timestamp + raw_body)`. Providers verify this on their side; replay window is ±5 minutes.

### Partner integrations (external CRM → AgentDo)

When AgentDo accepts webhooks from an external CRM (e.g. a booking platform pushing provider profiles), the partner signs with a per-partner secret:

```
X-AgentDo-Partner-Sig: <sha256_hex>
X-AgentDo-Partner-Ts:  <unix_ms>
```

Canonical input is `v1.<partner_slug>.<method>.partner/<partner_slug>.<ts>.<sha256(body)>` — so a valid signature for one partner cannot be replayed against another. Replay protection is enforced via a unique index on `sha256(sig)`.

## 2. What data is stored

When your agent books or orders on behalf of a customer, AgentDo stores:

| Field | Persisted? | Purpose |
|---|---|---|
| Customer name | yes | Shown to provider so they know who's arriving |
| Customer phone | yes | Provider may contact for confirmation / changes |
| Customer email | yes (optional) | Booking confirmation email |
| Booking time, service, provider | yes | Core booking record |
| Your API key ID | yes | Attribution for lead billing; scopes cancel/status to you |
| Payload logs | redacted + 90-day retention | PII fields (customer_name/email/phone) are one-way hashed with a fixed salt before persistence; raw payloads are **not** retained beyond the transactional window |
| IP address, user agent (request log) | 30 days | Security forensics |
| Audit log (every state change) | indefinite | Merkle-hash chained tamper-evident log; used for incident investigation |

### What we do NOT store

- Credit card details — AgentDo does not handle payment. Providers process payment on their side.
- Your customer's browsing history, device fingerprint, or any behavioural data unrelated to the booking.
- Persistent marketing profiles on end-customers.

## 3. PII hashing in internal logs

For operational logs (`inbound_webhook_log.payload_redacted`), we redact `customer_name`, `customer_email`, `customer_phone` — and the same fields nested under a `customer.{...}` object — before writing:

```
"customer_email": "pii:6a4b25c712acb74b"    # SHA-256(salt + value), first 16 hex
```

Provider names, service names, booking IDs are **not** considered PII and are retained verbatim for operational reasons.

The raw payload column in internal logs is persisted as `'{}'` — we have the redacted projection for debug and the booking/order row for state.

## 4. Retention policies

| Data | Retention | Reason |
|---|---|---|
| Bookings / orders | Until user requests erasure (GDPR Art. 17) or 2 years after last activity | Legal: provider may need proof of service; customer may dispute charges |
| `inbound_webhook_log` (redacted) | 90 days | Debugging + incident response; automatic daily cleanup |
| Request logs | 30 days | Security forensics |
| Audit log (Merkle chain) | Indefinite | Tamper-evidence integrity requires the full chain; but contains no PII |
| API keys (hashes only) | Until revoked | Authentication |
| Failed login attempts (IP + email) | 15 min sliding window | Rate limit |

## 5. GDPR / NAIH compliance

AgentDo operates under Hungarian and EU data protection law. Customer data is:

- Processed on the legal basis of **contract performance** (Art. 6(1)(b) GDPR) when your agent makes a booking on the customer's behalf;
- Stored on servers located in the EU (Hungarian hosting, AWS eu-central-1 for failover);
- Never sold, never shared with third parties beyond the provider being booked and the payment processor (if used);
- Subject to the rights listed in `/privacy` on https://agentdo.io (access, rectification, erasure, portability, objection).

A customer can request their data export or erasure at any time through the developer who made the booking, or directly via privacy@agentdo.me.

The data controller is AgentDo (company details on https://agentdo.io/imprint). Supervisory authority: **NAIH — Nemzeti Adatvédelmi és Információszabadság Hatóság** (https://naih.hu).

## 6. Incident response

If AgentDo experiences a data breach that affects your customers' data, we will notify affected developers within 72 hours, as required by GDPR Art. 33. The notification includes: scope, affected records, mitigation steps, and timeline.

You can reach us at security@agentdo.me for security reports. Please include the word "security" in the subject line and a reproduction if possible.

## 7. What we ask from you

- Do not log or persist our API keys in your own system beyond what's strictly necessary. Treat them like a password.
- Pass only the customer data you actually need for the booking. Our API does not require more than name + phone.
- If your agent can take booking actions on behalf of multiple customers, pass the same API key for all of them but ensure your customers understand that their booking details are visible to the provider they're booking with.
- For anything unusual (HIPAA-adjacent medical booking, legal services, minors), contact us before integrating.

## 8. Open source, audit trail

Our public artifacts (docs, manifest, API specs) live in [agentdo-public](https://github.com/AgentDo-io/agentdo-public). The MCP server implementation is proprietary; if you need a security audit for a regulated deployment, contact us for an NDA-gated architectural review.

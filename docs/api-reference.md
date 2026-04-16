# API Reference

Base URL: `https://api.agentdo.io`

All requests require an `X-API-Key` header. Generate keys from the [dashboard](https://agentdo.io/dashboard).

## Authentication

```
X-API-Key: ak_your_key_here
Content-Type: application/json
```

Keys are rate-limited to 1000 requests/hour by default.

## Endpoints

### `POST /v1/search`

Find providers by category + location.

**Request:**
```json
{
  "category": "restaurant",
  "lat": 47.4979,
  "lng": 19.0402,
  "radius_km": 5
}
```

**Response:** `200` with array of providers including ID, name, distance, services.

### `POST /v1/search/lookup`

Named-entity resolution. Pass a business name; returns confidence-scored matches.

```json
{ "name": "Café Gerbeaud", "lat": 47.4979, "lng": 19.0402 }
```

Confidence ≥0.90 → exact match · 0.50–0.89 → fuzzy candidates · <0.50 → not registered.

### `POST /v1/actions/book`

Reserve an appointment slot at a provider.

```json
{ "provider_id": 42, "slot_time": "2026-04-20T15:00:00Z", "customer": { "name": "...", "email": "..." } }
```

### `POST /v1/actions/cancel`

```json
{ "booking_id": "bk_xxx", "reason": "..." }
```

### `GET /v1/actions/status/:type/:id`

Check status of a booking or order. `type` is `booking` or `order`.

## Webhooks

If you provide a webhook URL when registering an API key, AgentDo will POST event notifications signed with HMAC-SHA256. See [webhooks.md](./webhooks.md).

## Errors

Standard HTTP status codes. Body always JSON: `{ "error": "...", "code": "..." }`.

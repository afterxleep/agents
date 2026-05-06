---
name: lemonsqueezy
description: "Interact with the LemonSqueezy REST API to manage FlowDeck's store — list and retrieve orders, subscriptions, customers, license keys, and store stats. Use when asked about orders, revenue, subscribers, refunds, license keys, or any LemonSqueezy data. Triggers on: show recent orders, list subscribers, check revenue, find customer by email, look up license key, refund order, cancel subscription, or any LemonSqueezy store query."
---

# LemonSqueezy Skill

## Setup

API key is in `~/.private/.env` as `LEMONSQUEEZY_API_KEY`. Load it before making requests:

```bash
source ~/.private/.env
```

Base URL: `https://api.lemonsqueezy.com/v1`
Auth header: `Authorization: Bearer $LEMONSQUEEZY_API_KEY`
Response format: JSON:API (`data[].attributes` contains the fields)

Rate limit: 300 req/min. Check `X-Ratelimit-Remaining` header.

## Common curl pattern

```bash
source ~/.private/.env
curl -s -H "Authorization: Bearer $LEMONSQUEEZY_API_KEY" \
     -H "Accept: application/vnd.api+json" \
     "https://api.lemonsqueezy.com/v1/ENDPOINT"
```

## Key Endpoints

| Resource | Endpoint |
|---|---|
| Store stats | `GET /stores` |
| Orders | `GET /orders` |
| Single order | `GET /orders/{id}` |
| Subscriptions | `GET /subscriptions` |
| Single subscription | `GET /subscriptions/{id}` |
| Update subscription | `PATCH /subscriptions/{id}` |
| Customers | `GET /customers` |
| Single customer | `GET /customers/{id}` |
| License keys | `GET /license-keys` |
| Single license key | `GET /license-keys/{id}` |

## Filtering & Pagination

Filters use `filter[field]=value` query params. Pagination uses `page[number]` and `page[size]` (default 10, max 100).

Common filters:
- `filter[store_id]=ID` — scope to a specific store
- `filter[status]=paid` — filter orders by status
- `filter[status]=active` — filter subscriptions by status
- `filter[user_email]=email` — find orders/subscriptions by email

Sort: `sort=-createdAt` (newest first), `sort=createdAt` (oldest first)

Example — recent paid orders:
```bash
GET /orders?filter[status]=paid&sort=-createdAt&page[size]=25
```

Example — find customer by email:
```bash
GET /customers?filter[email]=customer@example.com
```

## Reference Files

- **`references/api.md`** — Full field reference for orders, subscriptions, customers, license keys, stores
- **`references/subscription-actions.md`** — How to pause, cancel, resume, and change plans

Read these when you need field details or mutation patterns.

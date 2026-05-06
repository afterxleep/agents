# Subscription Actions

All mutations use `PATCH /subscriptions/{id}` with `Content-Type: application/vnd.api+json`.

## Cancel a Subscription

```bash
source ~/.private/.env
curl -s -X PATCH \
  -H "Authorization: Bearer $LEMONSQUEEZY_API_KEY" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data":{"type":"subscriptions","id":"SUB_ID","attributes":{"cancelled":true}}}' \
  "https://api.lemonsqueezy.com/v1/subscriptions/SUB_ID"
```

Sets `cancelled: true`. Subscription stays active until `ends_at`, then expires.

## Pause a Subscription

```bash
source ~/.private/.env
curl -s -X PATCH \
  -H "Authorization: Bearer $LEMONSQUEEZY_API_KEY" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data":{"type":"subscriptions","id":"SUB_ID","attributes":{"pause":{"mode":"void"}}}}' \
  "https://api.lemonsqueezy.com/v1/subscriptions/SUB_ID"
```

`mode` options:
- `void` — suspend billing (no charges, no invoices)
- `free` — continue service free of charge

Add `"resumes_at": "2025-12-01T00:00:00Z"` to auto-resume on a date.

## Resume a Paused Subscription

```bash
source ~/.private/.env
curl -s -X PATCH \
  -H "Authorization: Bearer $LEMONSQUEEZY_API_KEY" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data":{"type":"subscriptions","id":"SUB_ID","attributes":{"pause":null}}}' \
  "https://api.lemonsqueezy.com/v1/subscriptions/SUB_ID"
```

Setting `pause: null` resumes a paused subscription.

## Change Plan (Variant)

```bash
source ~/.private/.env
curl -s -X PATCH \
  -H "Authorization: Bearer $LEMONSQUEEZY_API_KEY" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data":{"type":"subscriptions","id":"SUB_ID","attributes":{"variant_id":NEW_VARIANT_ID}}}' \
  "https://api.lemonsqueezy.com/v1/subscriptions/SUB_ID"
```

Upgrades or downgrades to a different product variant/plan.

## Un-cancel a Subscription

Set `cancelled: false` before `ends_at` to reinstate:

```bash
source ~/.private/.env
curl -s -X PATCH \
  -H "Authorization: Bearer $LEMONSQUEEZY_API_KEY" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data":{"type":"subscriptions","id":"SUB_ID","attributes":{"cancelled":false}}}' \
  "https://api.lemonsqueezy.com/v1/subscriptions/SUB_ID"
```

## Disable a License Key

```bash
source ~/.private/.env
curl -s -X PATCH \
  -H "Authorization: Bearer $LEMONSQUEEZY_API_KEY" \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data":{"type":"license-keys","id":"KEY_ID","attributes":{"disabled":true}}}' \
  "https://api.lemonsqueezy.com/v1/license-keys/KEY_ID"
```

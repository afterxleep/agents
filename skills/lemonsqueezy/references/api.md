# LemonSqueezy API Reference

## Table of Contents
1. [Orders](#orders)
2. [Subscriptions](#subscriptions)
3. [Customers](#customers)
4. [License Keys](#license-keys)
5. [Stores](#stores)

---

## Orders

### Endpoints
- `GET /orders` — list all orders
- `GET /orders/{id}` — retrieve a single order

### Status values
- `pending`, `failed`, `paid`, `refunded`, `partial_refund`, `fraudulent`

### Key fields (attributes)
| Field | Type | Description |
|---|---|---|
| `store_id` | int | Store the order belongs to |
| `customer_id` | int | Customer who placed the order |
| `order_number` | int | Sequential order number |
| `user_name` | string | Customer full name |
| `user_email` | string | Customer email |
| `status` | string | Order status (paid, refunded, etc.) |
| `total` | int | Total in cents (order currency) |
| `total_usd` | int | Total in cents (USD) |
| `total_formatted` | string | Human-readable total (e.g. "$9.99") |
| `currency` | string | ISO 4217 currency code |
| `discount_total` | int | Discount in cents |
| `tax` | int | Tax in cents |
| `refunded` | bool | True if fully refunded |
| `refunded_at` | datetime | When the order was refunded |
| `first_order_item` | object | First product in the order (see below) |
| `urls.receipt` | string | Pre-signed receipt URL |
| `created_at` | datetime | ISO 8601 |
| `test_mode` | bool | Was this a test order? |

### first_order_item sub-object
| Field | Description |
|---|---|
| `product_id` | Product ID |
| `variant_id` | Variant ID |
| `product_name` | Product name |
| `variant_name` | Variant name |
| `price` | Price in cents |

### Filters for /orders
- `filter[store_id]`, `filter[customer_id]`, `filter[status]`, `filter[user_email]`

---

## Subscriptions

### Endpoints
- `GET /subscriptions` — list all subscriptions
- `GET /subscriptions/{id}` — retrieve a single subscription
- `PATCH /subscriptions/{id}` — update (pause, cancel, change variant)

### Status values
- `on_trial`, `active`, `paused`, `past_due`, `unpaid`, `cancelled`, `expired`

### Key fields (attributes)
| Field | Type | Description |
|---|---|---|
| `store_id` | int | Store |
| `customer_id` | int | Customer |
| `order_id` | int | Originating order |
| `product_id` | int | Product |
| `variant_id` | int | Variant/plan |
| `product_name` | string | Product name |
| `variant_name` | string | Plan name |
| `user_name` | string | Customer name |
| `user_email` | string | Customer email |
| `status` | string | Subscription status |
| `cancelled` | bool | Has been cancelled |
| `trial_ends_at` | datetime | When trial ends (null if not on trial) |
| `renews_at` | datetime | Next billing date |
| `ends_at` | datetime | When subscription expires (null if active) |
| `billing_anchor` | int | Day of month billing happens |
| `pause` | object | Pause config (`mode`, `resumes_at`) or null |
| `urls.customer_portal` | string | Pre-signed portal URL (24h validity) |
| `urls.update_payment_method` | string | Pre-signed payment update URL |
| `card_brand` | string | Card brand (visa, mastercard, etc.) |
| `card_last_four` | string | Last 4 digits |
| `payment_processor` | string | stripe or paypal |
| `test_mode` | bool | Test mode? |

### Filters for /subscriptions
- `filter[store_id]`, `filter[customer_id]`, `filter[order_id]`
- `filter[status]` — e.g. `active`, `cancelled`, `past_due`

---

## Customers

### Endpoints
- `GET /customers` — list all customers
- `GET /customers/{id}` — retrieve a single customer

### Key fields (attributes)
| Field | Type | Description |
|---|---|---|
| `store_id` | int | Store |
| `name` | string | Full name |
| `email` | string | Email address |
| `status` | string | Marketing status: `subscribed`, `unsubscribed`, `archived`, `bounced` |
| `city` | string | City |
| `region` | string | Region/state |
| `country` | string | Country code |
| `total_revenue_currency` | int | Total lifetime revenue in cents (USD) |
| `total_revenue_currency_formatted` | string | e.g. "$99.99" |
| `mrr` | int | Monthly recurring revenue in cents (USD) |
| `mrr_formatted` | string | e.g. "$9.99" |
| `urls.customer_portal` | string | Pre-signed portal URL (null if no subscriptions) |
| `created_at` | datetime | ISO 8601 |

### Filters for /customers
- `filter[store_id]`, `filter[email]`

---

## License Keys

### Endpoints
- `GET /license-keys` — list all license keys
- `GET /license-keys/{id}` — retrieve a single license key
- `PATCH /license-keys/{id}` — update (disable, set activation limit)

### Status values
- `inactive`, `active`, `expired`, `disabled`

### Key fields (attributes)
| Field | Type | Description |
|---|---|---|
| `store_id` | int | Store |
| `customer_id` | int | Customer |
| `order_id` | int | Order |
| `product_id` | int | Product |
| `user_name` | string | Customer name |
| `user_email` | string | Customer email |
| `key` | string | Full license key |
| `key_short` | string | Truncated key ("XXXX-…last12chars") |
| `activation_limit` | int | Max activations |
| `instances_count` | int | Current activation count |
| `disabled` | bool | Is key disabled? |
| `status` | string | License key status |
| `expires_at` | datetime | Expiry (null = perpetual) |

### Filters for /license-keys
- `filter[store_id]`, `filter[order_id]`, `filter[order_item_id]`, `filter[product_id]`, `filter[status]`

---

## Stores

### Endpoints
- `GET /stores` — list all stores
- `GET /stores/{id}` — retrieve a single store

### Key fields (attributes)
| Field | Type | Description |
|---|---|---|
| `name` | string | Store name |
| `slug` | string | Store slug |
| `domain` | string | Store domain |
| `url` | string | Full URL |
| `currency` | string | Default currency |
| `total_sales` | int | All-time total sales count |
| `total_revenue` | int | All-time total revenue in cents (USD) |
| `thirty_day_sales` | int | Sales in last 30 days |
| `thirty_day_revenue` | int | Revenue in last 30 days in cents (USD) |

---

## Pagination Meta

All list endpoints return a `meta.page` object:
```json
{
  "currentPage": 1,
  "from": 1,
  "lastPage": 5,
  "perPage": 10,
  "to": 10,
  "total": 48
}
```

Use `links.next` to fetch the next page.

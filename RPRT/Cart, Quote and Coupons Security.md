---
tags:
  - RPRT
  - database
  - commerce
  - cart
  - shipping-quotes
  - coupons
  - RLS
  - schema
  - reference
type: schema-reference
---
[[Studio Repertório]]
[[Database Schemas]]
[[Catalog Products Categories and Archival]]

# Cart, Quote, and Coupons Schema

> [!abstract] Commerce Schema
> The browser sends cart intent. PostgreSQL owns price, stock, shipping, discount, totals, expiry, and consumption truth.

> [!warning] Source Of Truth
> The repository migration, pgTAP tests, and `docs/database.md` are authoritative.

> [!example] Visual Explainers
> [Open the commerce schema visual explainer in Linear](https://uploads.linear.app/445023d5-264c-47cb-a70c-e4a15a58da5c/d8aaa184-da1a-43ec-9d24-ce7cae0fbc90/60f3ddad-9ccb-4a28-a292-57c3e649db5a)
> [Compare account and guest flows in Linear](https://uploads.linear.app/445023d5-264c-47cb-a70c-e4a15a58da5c/5caadee1-c691-4e53-ae4c-623fa2e1e62a/195367a3-e015-4fed-acfa-5cfd92ff2d7b)

## <span style="color:rgb(112, 48, 160)">Schema Boundary</span>

| Object | Purpose |
|---|---|
| `carts` | One non-expiring cart per authenticated account |
| `cart_items` | Product intent owned through the authenticated cart |
| `shipping_quotes` | Protected immutable quote evidence |
| `coupons` | Protected discount campaign definitions |
| Guest cart | Browser-local only; no guest cart row is stored |

## Guest Quote Flow

```mermaid
flowchart LR
    A[Guest browser cart intent] --> B[Trusted server validation]
    B --> C[Carrier quote]
    C --> D[Protected immutable quote]
    D --> E[Opaque quote ID]
    E --> F[Atomic order evidence consumption]
```

The protected quote binds the exact cart, destination, current resolved package data, parcel, provider, amount, and expiry evidence. Its finite validity is at most twenty-four hours. A changed, expired, mismatched, or reused quote fails.

## Cart Lifecycle

```mermaid
flowchart TD
    C[Persistent account cart] --> I[Cart line: product and quantity]
    I --> S{Product changes later}
    S -->|Archived, unavailable, assisted, or no stock| V[Line stays visible]
    V --> R[Owner can always remove it]
    V --> X[Quote and checkout reject it]
    C -->|Account cart deleted| D[Lines and short-lived quotes cascade]
    I -->|Product physical delete| K[Foreign key restricts delete]
```

- A cart line stores no price and reserves no stock.
- Product status updates do not trigger the product foreign key.
- The line alone does not permanently block a direct-to-assisted sales-mode change.
- Owner removal checks ownership only, so a stale line never traps the customer.

## Authority Boundary

| Browser can request | PostgreSQL must prove |
| --- | --- |
| Product and quantity | Current product status and stock |
| Destination CEP | Exact normalized destination |
| Coupon code | Validity, minimum subtotal, and global use limit |
| Opaque quote ID | Exact hashes, amount, provider, expiry, and unused state |

## Lock Order

Quote recording locks `cart` -> `products by UUID`. Evidence consumption locks `cart` -> `quote` -> `coupon`.

M3 order creation will extend this order with the remaining catalog and reservation locks. The current order gives deterministic outcomes for cart mutation, quote recording, quote consumption, and coupon-limit races.

## Concurrency Evidence

| Race | Proven result |
| --- | --- |
| Concurrent cart creation | Both sessions return one cart UUID |
| Concurrent quote use | One session consumes; one rejects |
| Final coupon use | One use commits; the losing quote stays unused |
| Cart mutation during consumption | Consume-before-mutate or stale evidence rejection |
| Quote expires while waiting | The post-lock clock rejects it |
| Quote recording expires while waiting | No expired quote row is stored |

## Verification

Passed: migration reset, database lint, 216 focused database assertions, architecture sync and drift check, format, ESLint, TypeScript, 39 unit tests, and production build.

The full database suite keeps one unrelated baseline failure in `outbox_replay.test.sql`: `service_role` does not have direct `SELECT` access to `outbox_jobs`. This same failure exists on clean `origin/dev`.

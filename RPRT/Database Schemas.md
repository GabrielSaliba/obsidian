---
aliases:
  - RPRT Database Schemas
  - Database Architecture
tags:
  - RPRT
  - database
  - schema
  - index
type: schema-index
---
[[Studio Repertório|Back to Studio Repertório]]

> [!abstract] Database Schemas
> A focused index of implemented Repertório table models, relationships, constraints, RLS boundaries, and protected database commands.

> [!warning] Source Of Truth
> Repository migrations, database tests, and `docs/database.md` are authoritative.

---

## <span style="color:#80352F">Schema Map</span>

```mermaid
flowchart TB
    DB[(Repertório Database)]
    ID[Identity and Access]
    CAT[Catalog]
    COM[Commerce]
    OPS[Reliability and Audit]

    DB --> ID
    DB --> CAT
    DB --> COM
    DB --> OPS
```

## <span style="color:rgb(112, 48, 160)">Identity And Access</span>

| Schema note | Database objects |
|---|---|
| [[Auth Users and Profiles]] | `auth.users`, `profiles`, `addresses`, Auth initialization triggers |
| [[Administrator Membership Schema]] | `admin_users`, `is_admin()` |

## <span style="color:rgb(0, 112, 192)">Catalog</span>

| Schema note | Database objects |
|---|---|
| [[Catalog Products Categories and Archival]] | `categories`, `products`, `product_media`, `merchandising_placements` |
| [[Product Media Storage]] | `storage.buckets`, `storage.objects`, product-media policies, archive guard |

## <span style="color:rgb(0, 176, 80)">Commerce</span>

| Schema note | Database objects |
|---|---|
| [[Cart, Quote and Coupons Security]] | `carts`, `cart_items`, `shipping_quotes`, `coupons` |

## <span style="color:rgb(255, 192, 0)">Reliability And Audit</span>

| Schema note | Database objects |
|---|---|
| [[Reliability and Audit Schema]] | `webhook_events`, `domain_transitions`, `operational_alerts`, outbox and replay tables |

---

> [!quote] Schema Boundary
> This index contains database schema references only. Delivery plans, issue state, review state, pull-request state, and work logs stay outside these notes.

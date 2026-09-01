---
aliases:
  - Studio Repertorio
tags:
  - RPRT
  - project-hub
  - dashboard
type: project-hub
status: Active
updated: 2026-09-01
---
> [!abstract] Studio Repertório
> Central workspace for project delivery, technical references, domain contracts, and product decisions.

> [!tip] Start Here
> Open [[RPRT - Tracklist|Work Log]] for daily progress. Use the sections below to find the current reference or contract by domain.

---

## <span style="color:#80352F">Project Navigation</span>

| Note                           | Purpose                                                         |
| ------------------------------ | --------------------------------------------------------------- |
| [[RPRT - Tracklist\|Work Log]] | Daily delivery notes, decisions, issues, and next work.          |
| [[Database Schemas]]           | Database tables, relationships, and schema security references. |
| [[Visual Explainers]]          | Local HTML diagrams and implementation walkthroughs.            |
| [[Stacked Pull Requests]]      | Review scope, dependency order, and stacked delivery judgment.   |

## <span style="color:#80352F">Data Architecture</span>

> [!example] Database Schemas
> Open [[Database Schemas]] for the implemented identity, catalog, commerce, reliability, and audit schemas.

| Hub | Coverage |
|---|---|
| [[Database Schemas\|Database Schema Index]] | Tables, relationships, diagrams, and schema security boundaries |
| [RPRT-69](https://linear.app/guisaliba/issue/RPRT-69/pin-exact-public-rls-expressions-and-routine-privileges) | Backlog follow-up for exact public RLS expressions and routine privileges |

## <span style="color:rgb(112, 48, 160)">Platform and Integrations</span>

> [!info] Foundation
> Identity, email delivery, and external service boundaries.

| Note | Category |
|---|---|
| [[Auth Users and Profiles]] | Authentication and profile data |
| [[Administrator Membership Schema]] | Administrator authorization |
| [[Sentry and Source Maps]] | Exception monitoring, source maps, credential roles, and initial setup |
| [[Resend]] | General transactional email operations reference |
| [[Resend Project Initialization]] | RPRT-62 project structure; M1 baseline complete |

## <span style="color:rgb(0, 112, 192)">Catalog and Product Media</span>

> [!note] Catalog Domain
> Product authority, categories, status, archival, media identity, and public visibility.

| Note | Category | State |
|---|---|:---:|
| [[Catalog Products Categories and Archival]] | Catalog reference | Approved |
| [[Product Media Storage]] | Storage and media reference | Complete |
| [[RPRT-50 - Catalog Authority and Product Status]] | Catalog contract | Approved |

## <span style="color:rgb(0, 176, 80)">Commerce and Customer Access</span>

> [!success] Storefront Boundary
> Cart security, quote evidence, coupon rules, and safe guest access.

| Note | Category |
|---|---|
| [[Cart, Quote and Coupons Security]] | Cart and quote security |
| [[Opaque Guest Access Token]] | Guest order access |
| [[Order and Settlement Security]] | Order creation, payment settlement, stock reservation, and protected commands |

## <span style="color:rgb(192, 0, 0)">Payments and Recovery</span>

> [!warning] Financial Safety
> Contracts that control settlement recovery, cancellation, refunds, and stock restoration.

| Note | Category |
|---|---|
| [[RPRT-41 - Payment Recovery Gate Contract]] | Payment recovery and settlement gates |
| [[RPRT-42 - Cancellation Intent and Paid-Stock Recovery Contract]] | Paid cancellation and stock recovery |

## <span style="color:rgb(255, 192, 0)">Reliability and External Effects</span>

> [!important] Durable Operations
> Audit evidence, outbox work, provider-call budgets, leases, retries, and protected replay.

| Note | Category | State |
|---|---|:---:|
| [[RPRT-52 - Reliability and Audit Foundation Plan]] | Implementation plan | Complete |
| [[Reliability and Audit Schema]] | Implemented schema | Reference |
| [[RPRT-43 - Provider Call Counting and Lease Contract]] | Provider-call and lease contract | Reference |

---

## <span style="color:#80352F">Visual Identity</span>

| Role | Preview | Value |
|---|---|---|
| Primary | <span style="display:inline-block;background:#80352F;color:#FFFFFF;padding:2px 10px;border-radius:12px;">Burgundy</span> | `#80352F` |
| Secondary | <span style="display:inline-block;background:#EAD9CE;color:#80352F;padding:2px 10px;border-radius:12px;border:1px solid #80352F;">Warm linen</span> | `#EAD9CE` |

> [!quote] Project Principle
> Keep product truth, financial truth, and external effects explicit, durable, and verifiable.

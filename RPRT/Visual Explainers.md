---
aliases:
  - RPRT Visual Explainers
  - Visual Explainer Index
tags:
  - RPRT
  - visual-explainer
  - index
type: visual-index
---
[[Studio Repertório|Back to Studio Repertório]]

# Visual Explainers

> [!abstract] Visual Library
> Central index for the Repertório architecture, environment, database, reliability, and recovery explainers.

> [!info] Library Status
> The vault contains eleven canonical HTML explainers in `RPRT/Visual Explainers/`. Current copies are attached to related Linear issues for workspace access.

---

## <span style="color:#80352F">Library Summary</span>

| Collection | Count | Storage |
|---|:---:|---|
| Physical HTML explainers | 11 | `RPRT/Visual Explainers/` |

## <span style="color:#80352F">Architecture And Environments</span>

> [!example] Studio Lifecycle
> [[Visual Explainers/Studio Repertorio Lifecycle Visual Explainer.html|Open the Studio lifecycle visual explainer]]

> [!example] Preview And Staging Controls
> [[Visual Explainers/PR-10 Preview and Staging Controls Visual Explainer.html|Open the Preview and staging controls visual explainer]]

| Related note | Coverage |
|---|---|
| [[2026-08-10]] | Lifecycle review, command ownership, guardrails, approval state, and diagram verification |
| [[2026-08-17]] | Environment identity, Preview isolation, migration gates, and deployment controls |

## <span style="color:rgb(112, 48, 160)">Identity And Access</span>

> [!example] Identity Schema
> [[Visual Explainers/RPRT-47 Identity Schema Visual Explainer.html|Open the identity schema visual explainer]]

| Related note | Coverage |
|---|---|
| [[Auth Users and Profiles]] | Auth identity, profiles, addresses, RLS, and account initialization |

## <span style="color:rgb(0, 112, 192)">Catalog</span>

> [!example] Catalog Authority
> [[Visual Explainers/RPRT-50 Catalog Authority Visual Explainer.html|Open the catalog authority visual explainer]]

| Related note | Coverage |
|---|---|
| [[Catalog Products Categories and Archival]] | Categories, products, media relations, merchandising, visibility, and guarded state |

## <span style="color:rgb(0, 176, 80)">Commerce And Product Media</span>

> [!example] Account And Guest Order Journey
> [[Visual Explainers/Account and Guest Order Journey Visual Explainer.html|Open the account and guest order journey visual explainer]]

> [!example] Cart, Quote, And Coupons
> [[Visual Explainers/RPRT-49 Cart Quote Coupon Visual Explainer.html|Open the commerce schema visual explainer]]

> [!example] Product Media Storage
> [[Visual Explainers/RPRT-10 Product Media Storage Visual Explainer.html|Open the product media storage visual explainer]]

| Related note | Coverage |
|---|---|
| [[Cart, Quote and Coupons Security]] | Cart ownership, protected shipping evidence, coupons, lock order, and consumption |
| [[Product Media Storage]] | Bucket rules, object identity, MIME validation, actor access, and public delivery |

## <span style="color:rgb(255, 192, 0)">Reliability And Audit</span>

> [!example] Reliability Foundation
> [[Visual Explainers/RPRT-52 Reliability and Audit Visual Explainer.html|Open the reliability and audit visual explainer]]

| Related note | Coverage |
|---|---|
| [[RPRT-52 - Reliability and Audit Foundation Plan]] | Audit evidence, durable outbox work, provider-call controls, leases, retries, and replay |
| [[Reliability and Audit Schema]] | Implemented reliability tables, relationships, RLS boundaries, and protected commands |

## <span style="color:rgb(192, 0, 0)">Payments And Recovery</span>

> [!example] Paid Cancellation
> [[Visual Explainers/RPRT-42 Cancellation Contract Visual Explainer.html|Open the paid-cancellation visual explainer]]

> [!tip] Order And Settlement
> [[Visual Explainers/PR-22 Order and Settlement Security Visual Explainer.html|Open the detailed PR #22 Order and Settlement visual explainer]].

> [!example] Order And Settlement Schema
> [[Visual Explainers/PR-22 Order and Settlement Schema Explorer.html|Open the interactive PR #22 schema explorer]].

| Related note | Coverage |
|---|---|
| [[RPRT-42 - Cancellation Intent and Paid-Stock Recovery Contract]] | Immutable cancellation intent, verified refunds, race controls, and exactly-once stock restoration |
| [[Order and Settlement Security]] | Order snapshots, settlement authority, stock reservations, guest access, recovery, and fulfillment gates |

---

## <span style="color:#80352F">Local File Index</span>

| Domain | Local HTML file |
|---|---|
| Studio architecture | [[Visual Explainers/Studio Repertorio Lifecycle Visual Explainer.html]] |
| Environments | [[Visual Explainers/PR-10 Preview and Staging Controls Visual Explainer.html]] |
| Identity and access | [[Visual Explainers/RPRT-47 Identity Schema Visual Explainer.html]] |
| Catalog | [[Visual Explainers/RPRT-50 Catalog Authority Visual Explainer.html]] |
| Commerce | [[Visual Explainers/RPRT-49 Cart Quote Coupon Visual Explainer.html]] |
| Customer order journey | [[Visual Explainers/Account and Guest Order Journey Visual Explainer.html]] |
| Product media | [[Visual Explainers/RPRT-10 Product Media Storage Visual Explainer.html]] |
| Reliability and audit | [[Visual Explainers/RPRT-52 Reliability and Audit Visual Explainer.html]] |
| Paid cancellation | [[Visual Explainers/RPRT-42 Cancellation Contract Visual Explainer.html]] |
| Order and settlement | [[Visual Explainers/PR-22 Order and Settlement Security Visual Explainer.html]] |
| Order and settlement schema | [[Visual Explainers/PR-22 Order and Settlement Schema Explorer.html]] |

> [!quote] Library Rule
> Store each canonical visual explainer as a physical HTML file in `RPRT/Visual Explainers/`. Add it to this index, attach its current copy to the related Linear issue, and use the attachment URL in Linear documents that reference it.

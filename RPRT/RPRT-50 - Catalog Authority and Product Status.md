---
tags:
  - RPRT
  - database
  - catalog
  - product-status
  - product-archival
  - contract-approved
issue: RPRT-50
status: Approved
pull_request: 19
final_commit: b266b21
approval_comment: f296a84e-3bc6-472a-aed6-d9cff70833ca
---
# RPRT-50 Catalog Authority and Product Status

> [!abstract] Final Catalog Contract Review
> RPRT-50 owns all M1 product archival work. The status-retention model, database implementation, tests, repository guidance, Technical Architecture, and Lifecycle Diagrams now use one approved contract.

> [!success] Current Approval Evidence
> RPRT-50 and PR #19 are approved against current head `b266b21`. The seven contract findings have explicit decisions and matching evidence. Two independent final reviews found no remaining issue. PR #19 was not merged by this work.

| Reference | Link |
|---|---|
| Linear issue and current approval comment | [RPRT-50](https://linear.app/guisaliba/issue/RPRT-50/implement-catalog-and-content-schema-security) - comment `f296a84e-3bc6-472a-aed6-d9cff70833ca` |
| Technical Architecture | [Normative architecture](https://linear.app/guisaliba/document/technical-architecture-487c0e8e7e8d) |
| Lifecycle Diagrams | [Normative lifecycle diagrams](https://linear.app/guisaliba/document/lifecycle-diagrams-7784b41c07ee) |
| Pull request | [PR #19](https://github.com/guisaliba/repertorio/pull/19) |
| Historical architecture clarification | [`7b55cfb`](https://github.com/guisaliba/repertorio/commit/7b55cfb) - unchanged |
| Final contract commit | [`b266b21`](https://github.com/guisaliba/repertorio/commit/b266b21) |

## <span style="color:rgb(112, 48, 160)">Approval Evidence</span>

The current Linear approval evidence confirms that `7b55cfb` remains unchanged and valid for category package fallback, product override precedence, category-parent immutability, media deactivation, explicit public visibility, zero-stock behavior, the direct audit-harness edge, and `primary keys` wording.

Commit `b266b21` adds the final archival contract and its current evidence: 189 catalog tests, 23 committed-session concurrency tests, and 39 unit tests. The SQL, repository guidance, Technical Architecture, Lifecycle Diagrams, PR description, and this note agree with that contract.

Approval: **LGTM for the current RPRT-50 contract and PR #19 head `b266b21`.**

## <span style="color:rgb(0, 112, 192)">Approved State Model</span>

The only product statuses are `draft`, `published`, and `unavailable`. Archival is the independent nullable `archived_at` retention value. All six stored combinations are valid.

| Status | Active | Archived |
|---|---|---|
| `draft` | Hidden and editable | Hidden and frozen |
| `published` | Public when all visibility rules pass | Hidden and frozen; status can move one way to `draft` |
| `unavailable` | Public when all visibility rules pass | Hidden and frozen; status can move one way to `draft` |

Approved transition rules:

- Archive preserves the current status.
- An archived status can stay unchanged or move one way to `draft`.
- An archived product cannot enter `published` or `unavailable`.
- Restore always clears `archived_at` and sets status to `draft`.
- Restore does not run publication validation.
- Later explicit publication runs normal category and package validation.
- `published_at` records the first real public activation and is never reset.

## <span style="color:rgb(112, 48, 160)">Finding Decisions</span>

| # | Finding | Final decision |
|---:|---|---|
| 1 | Normative ownership and state matrix | RPRT-50 owns the complete archive contract. All six `status x retention` combinations are valid. |
| 2 | Archived publication and `published_at` | Archived products cannot enter a public status. Restore returns to draft. Later publication validates eligibility and sets `published_at` only if this is the first real public activation. |
| 3 | Commerce behavior | Block new direct checkout and assisted issuance. Keep existing cart lines and assisted drafts, but block their next purchase step. Existing reservations, payments, late recovery, placed orders, and fulfillment continue from immutable snapshots. |
| 4 | ABA and retry identity | Archive and restore require expected `products.updated_at`. A stale request writes nothing and returns `applied = false`, current archive state, and current update time. |
| 5 | Mutation while archived | Freeze content, category assignment, stock, sales mode, media, and placements. Only restore and the one-way status move to draft are allowed. |
| 6 | Placement restoration | Archive atomically disables enabled placements. Restore never enables them. |
| 7 | Public media URLs | Archive removes catalog discovery only. It hides relational media rows, but a known public Storage URL or cached object can remain accessible. |

Additional decisions:

- Archived product dependencies do not block category activity or package-default changes.
- Media active state is preserved. Active media returns to catalog reads only after restore and explicit publication.
- The product row remains and keeps its unique slug reserved.
- Application roles have no hard-delete path.
- M1 has no product purge. A future retention decision must define purge authority, time, and evidence.
- Zero stock and archival remain separate concerns.

## <span style="color:rgb(0, 176, 80)">Implemented Controls</span>

- `public.set_product_archived(uuid, timestamptz, boolean)` returns `applied`, current `archived`, and current `updated_at`.
- The command locks the product, checks expected `updated_at`, disables enabled placements during archive, restores to draft, and writes immutable transition evidence.
- Archive, media writes, and placement writes use one catalog-relation transaction advisory guard before the product lock.
- Archive-aware RLS blocks direct edits and related inserts or updates for archived products.
- A product freeze trigger protects security-definer stock and sales-mode commands and future command paths.
- Product public RLS requires `archived_at is null`, public status, and an active category path.
- Media and placement public reads continue to depend on product visibility.
- The category advisory protocol still serializes publication with package-default removal.

## <span style="color:rgb(0, 112, 192)">Commerce Contract For Dependent Issues</span>

| Flow | Archived-product rule |
|---|---|
| New direct order | Reject before reservation or payment work starts. |
| Existing cart line | Keep the line. Block checkout until removal or valid active eligibility. |
| New assisted issuance | Reject while any referenced product is archived or otherwise ineligible. |
| Existing assisted draft | Keep the draft. Do not issue it while blocked. |
| Active reservation | Continue its normal consume or release lifecycle. |
| Pending or late payment | Continue verified reconciliation and recovery. |
| Placed order | Continue from immutable item and commercial snapshots. |
| Fulfillment | Continue from order settlement and fulfillment authority, not current catalog state. |

RPRT-50 documents these future command guards. It does not add commerce tables or commands.

## <span style="color:rgb(0, 176, 80)">Verification</span>

| Gate | Result |
|---|:---:|
| Catalog pgTAP | 189 / 189 passed |
| Catalog committed-session concurrency pgTAP | 23 / 23 passed |
| Combined archival contract | 212 / 212 passed |
| Database reset and migration list | Passed |
| Database lint | Passed |
| Format and diff checks | Passed |
| TypeScript and ESLint | Passed |
| Unit tests | 39 / 39 passed |
| Production build | Passed |
| Architecture freshness | Passed locally and in both remote checks |
| Independent final reviews | Two reviews, no findings |

The complete `pnpm db:verify` run applies and lints all migrations and passes every database test file except the unchanged `outbox_replay.test.sql` service-role permission failure. The run reached 542 assertions before that known file stopped at assertion 34 of 37.

Vercel still reports the known Git-author project-access failure. The local production build passes. This is not an application build defect.

## <span style="color:rgb(255, 192, 0)">Residual Risk</span>

The catalog-relation advisory guard intentionally serializes archive, media, and placement writes across products. This is acceptable for M1 administrator traffic. A future mixed product-relation command must take this guard before it locks a product, or it can create a new lock-order inversion.

## <span style="color:rgb(0, 112, 192)">Review State</span>

| Item | Final state |
|---|---|
| Product and architecture decisions | Approved and normative |
| Database implementation | Matches contract |
| Test evidence | Complete for RPRT-50 seams |
| Current approval evidence | Linear comment `f296a84e-3bc6-472a-aed6-d9cff70833ca` |
| Historical architecture commit | `7b55cfb`, unchanged |
| PR #19 | Open, updated, pushed, and mergeable |
| Final commit | `b266b21` |
| Contract blockers | Cleared |
| Known unrelated database failure | `outbox_replay.test.sql` service-role permission |
| Known external failure | Vercel Git-author access |

> [!summary] Current Rule
> Archive is reversible catalog retention, not a fourth status, stock state, or media-confidentiality control. Archived products remain retained and frozen. Restore always returns to draft. New purchase work stops, while valid existing commerce continues from immutable evidence.

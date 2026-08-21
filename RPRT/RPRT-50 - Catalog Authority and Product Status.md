---
tags:
  - RPRT
  - database
  - catalog
  - product-status
  - product-archival
  - decision-required
issue: RPRT-50
status: Blocked
---
# RPRT-50 Catalog Authority and Product Status

> [!abstract] Catalog Contract Review
> RPRT-50 defines the M1 database authority for categories, products, product media, and timed merchandising placements. The three-state product status model is approved. Product archival is implemented provisionally, but it is not yet part of the approved normative architecture.

> [!danger] Merge Gate
> **PR #19 must not merge yet.** The findings in this note require explicit product and architecture decisions. They are not approved definitions. After the decisions are recorded in the normative Linear documents, the implementation and tests must receive a new full contract review.

| Reference | Link |
|---|---|
| Linear issue | [RPRT-50](https://linear.app/guisaliba/issue/RPRT-50/implement-catalog-and-content-schema-security) |
| Technical Architecture | [Normative architecture](https://linear.app/guisaliba/document/technical-architecture-487c0e8e7e8d) |
| Lifecycle Diagrams | [Normative lifecycle diagrams](https://linear.app/guisaliba/document/lifecycle-diagrams-7784b41c07ee) |
| Pull request | [PR #19](https://github.com/guisaliba/repertorio/pull/19) |
| Catalog baseline | [`fca3bda`](https://github.com/guisaliba/repertorio/commit/fca3bda) |
| Lock-order fix | [`ff5cff7`](https://github.com/guisaliba/repertorio/commit/ff5cff7) |
| Provisional archival change | [`32f070a`](https://github.com/guisaliba/repertorio/commit/32f070a) |
| Current PR head | [`141498b`](https://github.com/guisaliba/repertorio/commit/141498b) |

## <span style="color:rgb(0, 112, 192)">Approved Baseline</span>

The database accepts exactly three product statuses.

<table style="width:100%; border-collapse:separate; border-spacing:0 8px;">
<thead>
<tr>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Status</th>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Public state</th>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Approved meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><code>draft</code></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><span style="color:#5b6472; background:#f1f3f5; border:1px solid #cfd4da; border-radius:999px; padding:2px 8px;">Hidden</span></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">Unpublished product state. The approved contract permits catalog editing.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><code>published</code></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><span style="color:#176b3a; background:#e8f7ee; border:1px solid #8bc7a3; border-radius:999px; padding:2px 8px;">Public when valid</span></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">Visible only with an active category path and valid resolved package data.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><code>unavailable</code></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><span style="color:#805500; background:#fff7d6; border:1px solid #e4c767; border-radius:999px; padding:2px 8px;">Still public</span></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">Remains in the public catalog when visibility rules pass. A direct product with zero stock remains visible and unavailable.</td>
</tr>
</tbody>
</table>

Approved baseline rules:

- Status and stock are separate concerns.
- Zero stock does not delete or archive a product.
- Public product reads require `published` or `unavailable` status and an active category path.
- Public media requires active media and a public product.
- Public placement reads require an enabled placement, a public product, and a valid time window.
- Application roles have no product hard-delete path.
- Publication and inherited package removal use the category advisory-lock protocol.

## <span style="color:rgb(112, 48, 160)">Provisional Archival Implementation</span>

> [!warning] Implemented Does Not Mean Approved
> Commit `32f070a` adds archival behavior to the repository. The implementation has useful foundations, but the normative Technical Architecture and Lifecycle Diagrams do not yet define this behavior.

| Area | Current provisional behavior |
|---|---|
| Retention field | Nullable database-authored `products.archived_at` |
| Product status | Still limited to `draft`, `published`, and `unavailable` |
| Guarded command | `public.set_product_archived(product_id, expected_archived, archived)` |
| Public product reads | Require `archived_at is null` |
| Related public reads | Media and placements are hidden through product visibility |
| Restoration | Clears `archived_at` and revalidates category and package rules for public statuses |
| Audit | Successful archive and restore actions write immutable `domain_transitions` evidence |
| Hard delete | No application-role delete path was added |
| Slug and data retention | Product, slug, status, stock, media, placements, and `published_at` are retained |

The repository search confirms that `archived` is not a valid product status. Its remaining SQL string uses are archival audit states and negative tests that prove status rejection.

## <span style="color:rgb(192, 0, 0)">Blocking Findings</span>

<table style="width:100%; border-collapse:separate; border-spacing:0 10px;">
<thead>
<tr>
<th style="width:7%; padding:8px; border-bottom:1px solid #d0d7de; text-align:center;">#</th>
<th style="width:38%; padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Finding</th>
<th style="width:55%; padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Why a decision is required</th>
</tr>
</thead>
<tbody>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb; text-align:center;"><strong>1</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><strong>The normative architecture does not define product archival.</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">The Technical Architecture defines publication state, visibility, stock, media, and purchase checks. It does not define <code>archived_at</code>, restoration, or an archival transition matrix. Local repository files now describe behavior that has no normative approval.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb; text-align:center;"><strong>2</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><strong>An archived draft can become published without publication validation.</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">The archival migration skips publication checks for every archived product. The existing status command can change an archived draft to <code>published</code> and set <code>published_at</code> although the product was never public. This conflicts with the documented first-publication meaning.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb; text-align:center;"><strong>3</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><strong>Purchase behavior is not defined.</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">RLS hides archived catalog rows, but no authoritative order command exists yet. The contract must separately decide new purchases, existing carts, active reservations, pending payments, late-payment recovery, existing orders and fulfillment, and assisted sales. Archive must stop new purchases without automatically breaking valid existing commerce.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb; text-align:center;"><strong>4</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><strong>The command has an ABA retry problem.</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">The command compares only an expected Boolean state. An active to archived to active cycle lets an old active-state request succeed. The contract must select an operation key, row version, expected timestamp, or another deterministic retry identity.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb; text-align:center;"><strong>5</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><strong>Mutation while archived is undefined.</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">Administrators can still change status, stock, sales mode, category, content, media, and placements. Editing can be valid or invalid, but the architecture must decide whether archived products are editable, partly editable, or frozen.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb; text-align:center;"><strong>6</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><strong>Restoration can reactivate old merchandising automatically.</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">Retained enabled placements become visible after restoration when their time window is valid. The product contract must decide whether this automatic return is intended or whether placement review is required.</td>
</tr>
<tr>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb; text-align:center;"><strong>7</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;"><strong>Public media URLs are a separate concern.</strong></td>
<td style="padding:10px 8px; border-bottom:1px solid #e6e8eb;">Database RLS hides media rows. It does not revoke a known public Storage URL or a cached image. The architecture must decide whether archive means catalog removal only or media-access revocation.</td>
</tr>
</tbody>
</table>

## <span style="color:rgb(255, 192, 0)">Decision Matrix To Define</span>

> [!important] Open Decisions
> The archived cells below are not definitions. They identify combinations for which the team must make explicit decisions.

| Product status | Active retention | Archived retention |
|---|---|---|
| `draft` | Approved hidden draft behavior | **Undecided:** editability, status transitions, restoration target, and validation |
| `published` | Approved public behavior when valid | **Undecided:** status mutation, `published_at`, purchase effects, and restoration visibility |
| `unavailable` | Approved visible unavailable behavior when valid | **Undecided:** stock changes, purchase effects, and restoration visibility |

The decision record must answer all of these questions:

| Decision area | Required answer |
|---|---|
| Valid combinations | Which `status x retention` combinations are permitted? |
| Status mutation | Can status change while a product is archived? |
| Publication time | Does `published_at` mean first public exposure, first status transition, or another event? |
| Archived editing | Which content, category, sales-mode, stock, media, and placement fields remain editable? |
| Retry identity | Does the command use an operation key, row version, expected timestamp, or another method? |
| New commerce | How do archive and restore affect new carts, checkout, direct orders, and assisted sales? |
| Existing commerce | What happens to existing carts, reservations, pending payments, late-payment recovery, orders, and fulfillment? |
| Merchandising | Do retained placements return automatically, stay disabled, or require review? |
| Media access | Do public URLs and caches remain usable, or must Storage access be revoked? |
| Slug identity | Do archived products reserve their slugs? Can a slug ever be reused? |
| Permanent purge | Will permanent purge exist? Who can run it, and what retention and evidence rules apply? |

## <span style="color:rgb(0, 176, 80)">Correct Foundations</span>

The contract review accepts these implementation foundations as useful, but not sufficient for approval:

- Status and archival are separate dimensions.
- Archived products, media rows, and placements are hidden by database RLS.
- Restoration revalidates active category paths and resolved package data.
- Archival uses an administrator-only guarded command.
- The command locks the product row.
- A successful change writes immutable audit evidence.
- Application roles have no product hard-delete path.
- Zero stock and archival remain separate concepts.
- Failed concurrent restoration writes no false archival transition.

## <span style="color:rgb(0, 112, 192)">Security And Concurrency Evidence</span>

- All four catalog tables have RLS.
- Application roles cannot directly update status, stock, sales mode, archival time, IDs, database timestamps, or media object identity.
- Security-definer commands use a fixed empty `search_path` and explicit grants.
- Publication and category package removal use one advisory-lock protocol per category.
- Restoration reuses publication validation after the category guard is available.
- The committed-session test proves package removal can commit while the product remains archived and that concurrent invalid restoration fails without a deadlock.
- Storage object policy is deferred and cannot be inferred from database row visibility.

## <span style="color:rgb(0, 176, 80)">Verification Snapshot</span>

| Gate | Result |
|---|:---:|
| Product status remnant search | Pass: no valid `archived` status |
| Catalog contract pgTAP | 154 / 154 |
| Catalog concurrency pgTAP | 19 / 19 |
| Unit tests after `dev` merge | 39 / 39 |
| Database lint when run alone | Pass |
| Format, TypeScript, and ESLint | Pass |
| Production build | Pass |
| Local architecture freshness | Pass |
| Remote Architecture / freshness | 2 / 2 pass |
| Independent implementation review | No code-level findings after test additions |

The full database gate still reaches the pre-existing `outbox_replay.test.sql` service-role permission failure. Vercel remains blocked because the Git author does not have project access; the local production build passes.

> [!warning] Verification Limit
> Green implementation checks do not resolve missing product and architecture decisions. Technical correctness against the provisional behavior is not contract approval.

## <span style="color:rgb(255, 192, 0)">Review State</span>

| Item | State |
|---|---|
| PR #19 | Open and technically mergeable |
| Archival implementation | Provisional |
| Normative archival contract | Missing |
| Product and architecture decisions | Required |
| Merge approval | **Blocked** |
| Next review | Full contract review after decisions and normative document updates |

The earlier recommendation only justified `archived_at` as a database modeling choice. It did not compare archival with every approved architecture and lifecycle rule. The technical lead was correct to stop the merge and request the full contract review.

## <span style="color:rgb(0, 112, 192)">Source Map</span>

Normative sources:

- [Technical Architecture](https://linear.app/guisaliba/document/technical-architecture-487c0e8e7e8d)
- [Lifecycle Diagrams](https://linear.app/guisaliba/document/lifecycle-diagrams-7784b41c07ee)
- [RPRT-50](https://linear.app/guisaliba/issue/RPRT-50/implement-catalog-and-content-schema-security)

Repository sources:

- `supabase/migrations/20260820070049_add_catalog_schema.sql`
- `supabase/migrations/20260820201205_add_product_archival.sql`
- `supabase/tests/catalog_schema.test.sql`
- `supabase/tests/catalog_schema_concurrency.test.sql`
- `docs/database.md`
- `src/architecture/graph.ts`
- `src/architecture/coverage.json`
- `src/architecture/measured.generated.ts`

## <span style="color:rgb(0, 176, 80)">Approval Sequence</span>

1. Product and architecture owners decide every blocking finding.
2. The team records those decisions in the normative Linear architecture and lifecycle documents.
3. The implementation, repository documentation, and tests change to match the approved contract.
4. Reviewers compare all behavior with the full status and retention matrix.
5. PR #19 can receive merge approval only when that review has no blocking finding.

> [!summary] Current Rule
> Three product statuses are approved. Archival is provisional. The findings require decisions; they do not define the answers. PR #19 must not merge until the normative contract is complete and the implementation passes a new full review.

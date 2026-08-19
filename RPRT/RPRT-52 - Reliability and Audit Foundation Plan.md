# RPRT-52 Reliability and Audit Foundation Plan

> [!summary] Implementation Status
> RPRT-66 was approved and merged through PR #12. RPRT-52 is now in progress on `feat/rprt-52-reliability-audit-schema`, created from updated `dev`. The final PR will remain draft until the user requests review readiness.

## Goal

Build the M1 PostgreSQL boundary for immutable provider evidence, domain audit, durable outbox work, bounded provider calls, dead letters, and protected replay. Follow the approved [RPRT-43 Outbox Retry and Protected Replay Policy](https://linear.app/guisaliba/document/rprt-43-outbox-retry-and-protected-replay-policy-95a7108529e8) without implementing real Resend, AbacatePay, or Jadlog adapters.

## Fixed Decisions

- Resolve the `admin_users` dependency cycle with a small prerequisite issue and PR.
- Protect worker commands with `service_role` and PostgreSQL-generated lease owner and lease tokens. RPRT-55 will later protect the HTTP scheduler boundary.
- Test public RPC signatures, grants, RLS, constraints, and true races with pgTAP and controlled `dblink` sessions.
- Use imperative, CLI-generated, ordered migrations. Do not edit the existing baseline migration.
- Use public RPC wrappers with an empty fixed `search_path`, explicit actor checks, and minimal grants. Keep non-RPC helpers in a non-exposed private schema.
- Make business keys globally unique and no longer than 256 characters. Preview-owned fixtures must include the approved Preview identifier in their owning ID or key.
- Resolve the RPRT-43 operation matrix in a versioned database function and copy every value into an immutable job policy snapshot when a job is created.
- Record dead-letter alerts in an append-only `operational_alerts` table in the same transaction. External alert delivery is outside this schema task.
- Do not hand-edit `src/types/database.ts`. RPRT-51 owns generated types after reviewed staging promotion.
- Do not promote migrations to staging or modify production resources during this work.

## Phase 1: Administrator Membership Prerequisite

### Linear Work

1. Create an urgent 2-point M1 database issue under RPRT-3 named `Establish administrator membership schema security`.
2. Make it block RPRT-52 and the remaining RPRT-47 work.
3. Update RPRT-47 so it consumes this prerequisite and continues to own profiles, addresses, guest access, profile initialization, and the welcome-email outbox contract.
4. Keep RPRT-52 out of ready work until the prerequisite is merged.

### TDD Slice

1. Create a feature branch from `dev` using the new Linear issue ID.
2. Add a failing pgTAP test for the public administrator-membership seam.
3. Prove that `admin_users` is keyed by `auth.users.id`, RLS is active, and direct access is denied to all application roles.
4. Prove that anonymous users cannot execute `is_admin()`, authenticated non-members receive false, authenticated members receive true, and customers cannot add themselves.
5. Add the smallest migration with `public.admin_users`, Auth-user delete cascade, no personal data, default-deny RLS, explicit revocations, and the approved `public.is_admin()` function.
6. Document the trusted manual first-admin bootstrap without a real user ID or personal data.
7. Run the focused and full gates, open a focused PR into `dev`, and wait for required review and merge.

## Phase 2: RPRT-52 Implementation

After the prerequisite is merged, update `dev`, move RPRT-52 to In Progress, and create `feat/rprt-52-reliability-audit-schema`.

Use vertical red-green slices. For each slice, add one failing behavior at a confirmed public seam, create the smallest ordered migration change, and run the focused test before continuing.

### Slice 1: Immutable Evidence

Add:

- `webhook_events` with provider-scoped event deduplication and safe payload fingerprint evidence;
- `domain_transitions` with aggregate, state, actor, source, reason, provider-event, operation, and database-time evidence;
- `operational_alerts` for append-only independent alert evidence;
- immutable-row guards that reject update and delete;
- default-deny RLS, complete privilege revocation, required indexes, and narrow service-role commands.

Tests prove deduplication, actor denial, and immutability.

### Slice 2: Enqueue With Immutable Policy

Add `outbox_jobs` and the internal RPRT-43 policy resolver.

The enqueue command must:

- accept only the nine approved job types;
- validate the exact stable-ID payload for each type;
- enforce one global business key and the 256-character limit;
- copy all policy limits, timeouts, backoff, jitter, idempotency, and lease values into immutable columns;
- create the job in `pending` with database-owned time;
- reject secrets, raw provider payloads, full requests, and avoidable personal data;
- return an existing job only for an identical idempotent request.

Tests use literal RPRT-43 values and prove that policy changes affect only new jobs.

### Slice 3: Claim and Reserve One Provider Call

Add service-role RPCs and append-only execution evidence:

- claim eligible jobs with `FOR UPDATE SKIP LOCKED`;
- generate the lease owner and lease token in PostgreSQL;
- keep claims separate from provider-call counts;
- add immutable `outbox_provider_calls` reservations;
- add immutable one-to-one `outbox_provider_call_results` records;
- verify lease ownership, one call per claim, all ceilings, and request fingerprint before dispatch;
- reserve the next call ordinal and increase the cumulative provider-call count in one transaction.

Tests prove exclusive claims, stale-lease denial, one call per claim, pre-dispatch counting, and concurrent ceiling enforcement.

### Slice 4: Complete, Reschedule, and Dead-Letter

Add guarded commands that:

- complete exactly once and keep `completed` terminal;
- insert immutable results for all approved outcome classes;
- reschedule only retry-safe work with stored equal jitter;
- recover expired leases only when repetition is safe;
- dead-letter permanent, exhausted, ambiguous unsafe, or non-idempotent results;
- write immutable audit and an independent alert in the same dead-letter transaction;
- keep unverified AbacatePay and Jadlog writes fail-closed.

Tests prove exact transitions, atomic rollback, jitter bounds, Resend idempotency limits, immutable fingerprints, and the permanent payment-session replay prohibition.

### Slice 5: Administrator-Protected Replay

Add immutable:

- `outbox_replay_authorizations` for administrator identity and safety proof;
- `outbox_replay_consumptions` so proof consumption is append-only.

Authenticated RPCs must require `public.is_admin()` and enforce dead-letter state, an approved replayable operation, one unused proof, unchanged job evidence, remaining ceilings, and the same original job. Replay does not restore automatic calls.

Tests prove non-admin denial, proof single use, cumulative counts, unchanged identity, race safety, and return to dead letter after a failed replay call.

## Test Infrastructure

- Keep normal pgTAP files transactional with `begin`, exact `plan`, `finish`, and `rollback`.
- Simulate `anon`, `authenticated`, and `service_role` with explicit roles and JWT claims.
- Add a shared `.inc` helper only after two test files need the same support.
- Use test-only `dblink` for true race tests with advisory serialization, deterministic cleanup, and no credential output.
- Document the narrow concurrency-test exception in `docs/database.md`.
- Test behavior through public RPCs. Catalog assertions support but do not replace behavior tests.

## Expected Files

Administrator prerequisite PR:

- one new migration under `supabase/migrations/`;
- one new pgTAP file under `supabase/tests/`;
- `docs/database.md` bootstrap guidance.

RPRT-52 PR:

- several ordered migrations under `supabase/migrations/`;
- focused pgTAP files for evidence, policy, worker commands, replay, and concurrency;
- an optional shared `.inc` test helper;
- `docs/database.md` role-simulation and concurrency guidance.

No provider adapter, scheduler route, seed data, generated database type, or staging migration belongs in this work.

## Verification and Delivery

For each PR:

1. Check the pinned Supabase CLI help.
2. Run each focused pgTAP file during its red-green slice.
3. Run `pnpm test:db` and `pnpm db:lint`.
4. Run `pnpm db:verify` from a clean disposable local database.
5. Run `pnpm format:check`, `pnpm lint`, `pnpm test`, `pnpm typecheck`, and `pnpm build`.
6. Review grants, RLS, function ownership, fixed `search_path`, indexes, immutable guards, and migration order.
7. Keep commits atomic and use Conventional Commits with Linear references.
8. Open each PR into `dev`, record non-secret evidence, and wait for required human review before merge.
9. Do not promote migrations to shared staging or change production resources without a separate explicit request.

## Final Branch and Evidence Workflow

- PR #12 merged into `dev` as `77a68a8` before RPRT-52 implementation began.
- RPRT-52 uses one feature branch created directly from updated `dev`.
- Each green vertical slice receives one atomic Conventional Commit with `Refs: RPRT-52`.
- The RPRT-52 PR targets `dev` and remains draft after implementation.
- If `dev` changes, merge `origin/dev` with a normal merge commit. Do not rebase or force-push published work.
- After each slice, record the behavior, focused command, expected red result, minimal migration, green result, test count, and commit SHA below and in a Linear comment.
- Evidence must not contain raw logs, database URLs, keys, tokens, passwords, fixture emails, or personal data.

## Implementation Evidence

Evidence will be appended here after each completed red-green slice.

### Slice 1 Evidence: Immutable Reliability Evidence

- **Behavior:** durable provider-event deduplication, immutable domain audit, and independent operational-alert evidence with default-deny access.
- **Focused command:** `pnpm test:db -- supabase/tests/reliability_evidence.test.sql`.
- **Red:** 3 of 3 assertions failed because `webhook_events`, `domain_transitions`, and `operational_alerts` did not exist.
- **Minimal change:** `20260818215648_add_immutable_reliability_evidence.sql` added the three append-only tables, indexes, RLS, privilege revocation, immutable guards, and the service-role webhook evidence command.
- **Green:** 36 focused assertions passed; 69 total pgTAP assertions passed; database lint reported no schema warnings.
- **Commit:** `25c7146 feat(database): add immutable reliability evidence`.

### Slice 2 Evidence: Policy-Based Outbox Enqueue

- **Behavior:** approved job types enter one durable pending job with exact immutable RPRT-43 policy and stable-ID payload contracts.
- **Focused command:** `pnpm test:db -- supabase/tests/outbox_enqueue.test.sql`.
- **Red:** 2 of 2 assertions failed because `outbox_jobs` and `enqueue_outbox_job` did not exist.
- **Minimal change:** `20260818220114_add_policy_based_outbox_enqueue.sql` added the outbox, versioned policy resolver, payload validation, immutable policy guard, queue index, default-deny access, service-role enqueue RPC, idempotent business-key handling, and one creation transition per job.
- **Green:** 34 focused assertions passed; 103 total pgTAP assertions passed; database lint reported no schema warnings.
- **Commit:** `9b86373 feat(database): add policy-based outbox enqueue`.

### Slice 3 Evidence: Outbox Claim and Provider-Call Reservation

- **Behavior:** workers claim eligible pending jobs with immutable execution policy, and a lease owner reserves exactly one automatic provider call before dispatch without exceeding automatic, lifetime, fingerprint, or provider-window limits.
- **Focused command:** `pnpm test:db -- supabase/tests/outbox_worker_commands.test.sql supabase/tests/outbox_concurrency.test.sql`.
- **Red:** the initial 4 of 4 assertions failed because provider-call evidence tables and worker RPCs did not exist. Later focused checks also proved the missing automatic-budget and provider-idempotency-window guards before their minimal fixes.
- **Minimal change:** `20260818220811_add_outbox_claim_and_call_reservation.sql` added append-only call and result evidence, bounded `FOR UPDATE SKIP LOCKED` claims, database-generated leases, atomic automatic call reservation, stored execution-policy output, stale-lease denial, request-fingerprint stability, and automatic, lifetime, and idempotency-window enforcement. A local-only `dblink` test proves lock-skipping claims and concurrent one-call-per-lease enforcement.
- **Green:** 49 focused assertions passed; 152 total pgTAP assertions passed from a clean database; database lint reported no schema warnings.
- **Commit:** `07ec42f feat(database): add outbox claim and call reservation`.

### Slice 4 Evidence: Completion, Retry, Dead-Letter, and Lease Recovery

- **Behavior:** successful calls complete exactly once; retry-safe failures store bounded equal jitter; unsafe, permanent, exhausted, or provider-window-expired work dead-letters with immutable result, transition, and independent alert evidence; expired leases recover only when repetition is proven safe.
- **Focused command:** `pnpm test:db -- supabase/tests/outbox_completion.test.sql supabase/tests/outbox_failure_commands.test.sql supabase/tests/outbox_lease_recovery.test.sql`.
- **Red:** completion first failed because its RPC did not exist; safe reschedule and direct dead-letter each failed because their RPCs did not exist; lease recovery failed because its RPC did not exist. Later focused checks exposed and proved the provider-window terminality boundary before the minimal fix.
- **Minimal change:** `20260818223049_add_outbox_completion_and_dead_letter_controls.sql` added service-role-only complete, reschedule, dead-letter, and expired-lease recovery RPCs; terminal completion enforcement; append-only classified results; equal-jitter scheduling; safe repetition rules; and atomic dead-letter audit plus alert evidence.
- **Green:** 41 focused assertions passed; 193 total pgTAP assertions passed from a clean database; database lint reported no schema warnings; review found no remaining high- or medium-severity issue.
- **Commit:** `1878314 feat(database): add outbox completion and dead-letter controls`.

### Slice 5 Evidence: Administrator-Protected Outbox Replay

- **Behavior:** an authenticated administrator creates one immutable safety proof for a replayable dead-letter job; one proof returns the same job to pending and permits one replay call without resetting automatic or lifetime counts. Replay failure returns to dead letter, and concurrent administrators cannot consume one proof twice.
- **Focused command:** `pnpm test:db -- supabase/tests/outbox_replay.test.sql supabase/tests/outbox_replay_concurrency.test.sql`.
- **Red:** authorization first failed because replay evidence tables and `authorize_outbox_replay` did not exist; proof consumption failed because `replay_outbox_job` did not exist; replay reservation initially followed the automatic idempotency path; focused recovery checks exposed reported-failure and pre-dispatch replay lease gaps before their minimal fixes.
- **Minimal change:** `20260818224616_add_protected_outbox_replay.sql` added append-only authorization and consumption evidence, authenticated administrator RPCs, composite proof/job/ordinal constraints, one active replay slot, replay-aware call reservation, immediate replay-failure dead-letter behavior, pre-dispatch replay lease recovery, and permanent `payment-session-create` terminal enforcement. A local-only `dblink` test proves single proof consumption under contention.
- **Green:** 39 focused assertions passed; 232 total pgTAP assertions passed from a clean database; database lint reported no schema warnings; final review found no remaining high- or medium-severity Slice 5 issue.
- **Commit:** `b461036 feat(database): add protected outbox replay`.

### Draft PR Delivery Evidence

- **Draft PR:** [#13 feat(database): implement reliability and audit schema security](https://github.com/guisaliba/repertorio/pull/13), targeting `dev` from `feat/rprt-52-reliability-audit-schema`.
- **Slice commits:** `25c7146`, `9b86373`, `07ec42f`, `1878314`, and `b461036`.
- **Commit #5 scope:** `b461036` includes the generated local Supabase and agent artifact ESLint exclusions; there is no separate delivery-fix commit.
- **Database verification:** seven ordered migrations applied from empty, database lint reported no warnings, and 232 pgTAP assertions passed.
- **Repository verification:** formatting passed, lint passed, 36 unit tests passed, TypeScript passed, and the production build passed.
- **Branch sync:** `origin/dev` remained at `77a68a8`; no merge was required. The feature branch is pushed and tracks its remote.
- **Status:** RPRT-52 remains In Progress while draft PR #13 waits for required human review. No migration was promoted to staging and no deployment or production resource changed.
- **External check:** `Vercel Preview Comments` passed. The Vercel deployment check failed before deployment because Git author `GabrielSaliba` does not have access to the Studio Repertório Vercel project. The local production build passed; no Vercel membership or permission was changed.
# RPRT-52 Reliability and Audit Foundation Plan

> [!summary] Planning Status
> Draft implementation plan prepared on August 18, 2026. The plan requires one administrator-membership prerequisite before [RPRT-52](https://linear.app/guisaliba/issue/RPRT-52/implement-reliability-and-audit-schema-security) can start safely.

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
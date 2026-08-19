Commit 1:
Create tables and verify with tests
- webhook_events
- domain_transitions
- operational_alerts

Commit 2:
Create table outbox_jobs and its tests
Create enqueue_outbox_job Postgres function
- accept only approved job types;
- copy all policy limits, timeouts, backoff, jitter, idempotency, and lease values into immutable columns
- create the job in `pending` with database-owned time;
- return an existing job only for an identical idempotent request.

Commit 3:
Add service-role database function calls and its execution evidence
Create tables: 
- outbox_provider_calls
- outbox_provider_call_results
 Create functions: 
 - claim_outbox_jobs
 - reserve_outbox_provider_call

Tests should cover:
- claim eligible jobs
- generate the lease owner and lease token
- verify lease ownership, one call per claim, all retry and replay ceilings
- concurrency against multiple workers trying to claim the same job

dblink creates two separate database sessions: worker_one and worker_two to test concurrency

Commit 4:

- adds the final worker commands for an outbox job 
- verify if a completed outbox job state is terminal
- create function complete_outbox_job
- after a job is completed verify and audit the record in domain_transitions table

Commit 5:

Create tables:
- outbox_replay_authorizations (administrator that requested replay)
- outbox_replay_consumptions (proof of replay consumption)

Create replay_outbox_job function to trigger a replay requested by a admin


Além das migrations, tables e functions essa task gerou muitos testes tbm, eu olhei não linha por linha de cada teste. 
Só foquei em ver o que cada teste faz e o resultado esperado

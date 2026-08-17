> [!abstract] RPRT-41 Contract Guide
> How dedicated recovery cases, authoritative resolutions, and guarded commands prevent unsafe payment settlement and fulfillment.

| Reference | Link |
|---|---|
| Linear issue | [RPRT-41](https://linear.app/guisaliba/issue/RPRT-41/define-the-recovery-gate-persistence-contract) |

## <span style="color:rgb(0, 112, 192)">Purpose</span>

A payment recovery case records one financial incident that the normal order flow cannot resolve safely. An unresolved blocking case prevents settlement and fulfillment until authoritative financial evidence resolves the incident.

> [!important] Recovery Gate Rule
> `has_open_payment_recovery_gate(order_id)` returns `true` only when the order has at least one unresolved blocking recovery case.

## <span style="color:rgb(112, 48, 160)">Settlement</span>

A settlement is the system's final accounting decision about which successful payment it accepts for an order. A settled order can continue to fulfillment when all other conditions are valid.

One order normally has one settlement. Extra successful payments are not settlements and must be refunded. In this contract, settlement is not the later transfer of provider funds to a bank account.

## <span style="color:rgb(0, 176, 80)">Dedicated Recovery-Case Rows</span>

Each recovery incident gets a separate database row. A row identifies the order, payment, incident type, blocking state, current status, creation time, and authoritative resolution.

```text
payment_recovery_cases
-------------------------------------------------
id              rc_123
order_id        order_456
payment_id      pay_789
reason          unsafe_late_payment
blocking        true
status          open
resolution      null
created_at      2026-08-16
resolved_at     null
```

Separate rows preserve incident identity. If an order has two unresolved incidents, resolution of one incident does not remove the gate for the other incident.

## <span style="color:rgb(192, 0, 0)">Unsafe Gate Designs</span>

A mutable Boolean such as `orders.recovery_blocked` is unsafe. It does not identify the incident. If two incidents open the gate, resolution of one incident can incorrectly set the Boolean to `false` while the other incident is still open.

Pure derivation from payment and audit rows is also unsafe. It creates a complex safety predicate across payment attempts, refunds, webhooks, and audit events. It also makes ownership, diagnosis, and the current incident status difficult to see.

An administrator action or timer cannot directly close a blocking case. Neither action proves what happened to the money. An administrator can approve an action, and a timer can start another provider check, but only authoritative settlement or verified full-refund evidence can resolve the case.

## <span style="color:rgb(255, 192, 0)">Unsafe Late Payment</span>

An unsafe late payment occurs when the provider confirms payment success after the order expired, was cancelled, or could no longer continue safely.

`payment-recovery-required` means that the system must create a recovery case for this incident. It is an event or reason for case creation, not a mutable Boolean on the order.

The case remains blocking until one of these results is authoritative:

1. The system safely accepts the payment as the order settlement.
2. The provider verifies a full refund of the payment.

## <span style="color:rgb(192, 0, 0)">Failed All-Or-Nothing Stock Recovery</span>

After a late payment, the system can try to recover the complete stock reservation. The recovery must succeed for all required items or for none of them.

If only part of the stock is available, the system releases the partial reservation. It keeps settlement unset because it cannot fulfill the complete order. The recovery case stays blocking so that no later process can settle or fulfill the order by mistake.

A refund request does not resolve the case. The case closes only after the provider verifies a full refund. Settlement remains unset because the payment was returned.

```text
Incomplete stock recovery -> no settlement
Refund requested          -> case remains open
Full refund verified      -> case can resolve
```

## <span style="color:rgb(255, 192, 0)">Unresolved Provider Result</span>

A timeout, unknown status, conflicting response, or missing webhook does not prove payment or refund success or failure.

The system keeps the recovery case open and blocking. It creates separate operational alert evidence with the order ID, payment ID, recovery-case ID, provider, operation, last known result, time, and retry count.

The alert and recovery case have different meanings:

| Record | Question |
|---|---|
| Operational alert | Does an operator need to investigate? |
| Recovery case | Is the financial outcome authoritative and safe? |

An operator can acknowledge or close the alert without resolving the recovery case. The system must not infer settlement or refund from a timeout, elapsed time, an administrator action, or the absence of a webhook.

## <span style="color:rgb(0, 176, 80)">Protected Workers And Guarded Commands</span>

Customers, guests, staff, and workers cannot directly insert, update, or delete recovery case or resolution rows.

A protected worker receives permission only to execute specific commands, for example:

```text
request_full_refund(recovery_case_id)
record_verified_refund(recovery_case_id, provider_evidence)
retry_stock_recovery(recovery_case_id)
accept_payment_as_settlement(recovery_case_id)
```

Each guarded command has one narrow purpose. It verifies the case state, payment identity, lease or authorization, provider evidence, required amount, idempotency, and concurrent state inside one database transaction.

The worker must not have permission to run an arbitrary direct change such as:

```sql
UPDATE payment_recovery_cases
SET status = 'resolved';
```

This permission model limits damage from worker defects, compromised credentials, duplicate jobs, stale leases, and concurrent recovery attempts.

> [!summary] Payment Recovery Rule
> Operational actions can start or retry recovery. Only authoritative settlement or verified full-refund evidence can complete recovery and remove the blocking gate.

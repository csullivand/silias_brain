---
tags: [silia, billing, rta, dunning, analysis]
created: 2026-05-07
---

# RTA Dunning — Gap Analysis

Analysis of existing dunning system vs. requirements for per-agent RTA suspension.

## What Already Exists (Reusable)

| Component | What it does | File |
|-----------|-------------|------|
| Webhook `invoice.payment_failed` | Escalating dunning (8 attempts), emails to client and commercial team | `StripeWebhookHandler.ts:718-794` |
| Webhook `customer.subscription.updated` | When status → `unpaid`, suspends entire account | `StripeWebhookHandler.ts:796-852` |
| AccountStatusService | `changeStatus('inactive')` → deactivates ALL chatbots + pauses subscriptions | `AccountStatusService.ts:26-91` |
| Safety net batch | Scans subscriptions >9 days overdue, suspends if webhook dropped | `deactivateOverdueChats` |
| NotificationService | Escalating dunning emails, pre-suspension, suspension, reactivation | `NotificationService.ts:250-354` |
| Stripe pause/resume | `pause_collection: { behavior: 'void' }` / `null` | `StripeGateway.ts:422-436` |
| Auto-reactivation | When payment succeeds + account inactive → reactivates | `StripeWebhookHandler.ts:520-540` |

## Current Dunning Flow

```
Payment Failure → Stripe Smart Retries (7 days)
  → invoice.payment_failed webhook (escalating emails)
  → attemptCount >= 7: pre-suspension warning
  → Stripe marks subscription → 'unpaid' (after 8 attempts)
  → customer.subscription.updated webhook
  → AccountStatusService.changeStatus('inactive')
    → ALL chatbots status → 0
    → ALL subscriptions paused
    → Account suspension email
  → [Service Suspended]

Safety Net (Daily 2:00 AM UTC):
  → Scan subscriptions > 9 days overdue
  → AccountStatusService.changeStatus('inactive')
  → performedByUserId: 'system-safety-net'

Reactivation:
  → Payment received → handlePaymentSuccess()
  → If account was 'inactive' → changeStatus('active')
  → ALL chatbots status → 1, subscriptions resumed
```

## What's Missing for Per-Agent RTA Dunning

### 1. Suspension by AGENT (not account) — 1-2d
**Problem:** Current system suspends the ENTIRE account (all chatbots). Task requires suspending ONLY the RTA agent with failed payment.

**Needed:**
- New method: `suspendAgent(chatbotId)` — deactivates 1 chatbot + pauses its individual subscription
- Modify `handleSubscriptionStatusChange` to detect if subscription is for an individual RTA agent
- Don't trigger account-level suspension for individual agent failures

### 2. Webhook `invoice.marked_uncollectible` — 0.5d
**Not implemented.** The webhook handler doesn't listen for this event. Per the task, this is the trigger for RTA agent suspension.

**Needed:** Add case `'invoice.marked_uncollectible'` in webhook switch.

### 3. Stop metering on suspended agent — 0.5d
**Partially exists.** AudioStream checks `stripeSubscriptionItemId` but not `chatbot.status`. If chatbot is `status: 0`, the RTA widget shouldn't be able to start calls.

**Needed:** Verify batch `reportUsageToStripe` doesn't report usage for chatbots with `status: 0`.

### 4. Manual reactivation (SuperAdmin only from BO) — 0.5-1d
**Partially exists.** `PATCH /accounts/{id}/status` with superuser middleware exists, but reactivates entire account.

**Needed:** Endpoint or logic to reactivate an individual agent.

### 5. Dunning suspension banner in frontend — 1d
**Not implemented.** Frontend doesn't distinguish between 'disabled manual' and 'suspended by dunning'.

**Needed:** New field on chatbot (e.g. `suspensionReason: 'dunning'`) and differentiated banner component.

### 6. Audit log of event — 0.5d
**Partially exists.** `AccountStatusAuditModel` logs account status changes, not individual agent changes.

## Estimated Effort

| Requirement | Status | Effort |
|-------------|--------|--------|
| Webhook `invoice.marked_uncollectible` | Missing | 0.5d |
| Suspension by agent (not account) | Missing | 1-2d |
| Stop metering on suspended agent | Partial | 0.5d |
| Manual reactivation by SuperAdmin | Partial | 0.5-1d |
| Banner UI (dunning vs manual) | Missing | 1d |
| Audit log per agent | Partial | 0.5d |
| **Total** | | **~4-5d** |

## Key Decision Pending

**Per-agent vs. per-account suspension:** The current system is per-account. If a client has 3 RTA agents and only 1 has failed payment, the task says only that 1 should be suspended. This requires restructuring the suspension flow from account-level to agent-level.

## Related Notes
- [[RTA Billing Flow End-to-End]]
- [[RTA Module Architecture]]
- [[Billing Code Review Learnings]]
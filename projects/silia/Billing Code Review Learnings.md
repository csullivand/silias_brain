---
tags: [silia, billing, code-review, learnings]
created: 2026-05-06
---

# Billing Code Review Learnings

Patterns and issues found during SL-1146 metered billing PR reviews. Reference this before writing billing code.

## Critical Patterns

### Never save billing state without Stripe confirmation
If a Stripe API call fails (price creation, subscription item), do NOT save the related field to DynamoDB. Return a 500 instead. Otherwise the chatbot ends up with `perMinuteRate` but no `stripeSubscriptionItemId`, and usage is silently lost forever.

### IAM permissions must match actual operations
If a Lambda does `PutItem`, `UpdateItem`, AND `Query` on a table, all three must be in the IAM policy. Include `/index/*` for GSI queries. A `.catch(() => 0)` in the code will make missing permissions fail silently — the feature just returns wrong data with no error.

### DynamoDB operations belong in the model, not handlers
Don't import `DynamoDBClient`, `UpdateCommand`, `ScanCommand` in Lambda handlers or AudioStream. Use model methods (`claimForReporting()`, `scanUnreported()`, etc.) that call `this.update()` / `this.scan()` from the `DB` base class. This keeps handlers clean and logic testable.

### class-validator: nullable fields need @IsOptional()
`@IsNumber()` on a `number | null` field will reject `null`. Always pair with `@IsOptional()` when the type allows null.

## Stripe-Specific

### automatic_tax must be explicitly managed
We disabled Stripe Tax (`automatic_tax: { enabled: false }`) because Silia manages tax via its own Tax Management system. Always add the comment explaining why. Don't re-enable it without coordinating with the tax feature.

### Metered pricing: per-second price, per-minute display
Stripe invoice shows `$0.001333/unit` (per-second). Frontend shows `$0.080 /min USD` with a smaller `$0.001333 /sec USD` subtitle. Both rates must be shown to avoid confusion between invoice and UI.

### Create metered price BEFORE subscription, not after
Adding a subscription item after creation generates extra invoices. Pre-create the metered price and pass it via `additionalPriceIds` in the subscription creation call. Skip the $0 base price for metered-only subscriptions.

### Stripe always creates a $0 invoice on subscription creation
This is unavoidable with API version `2024-12-18.acacia`. `billing_mode: 'flexible'` would fix it but isn't supported in this version. Accept the cosmetic $0 Paid invoice.

### usage_records quantity is integer only
Can't send `0.583` minutes — Stripe rejects decimals on legacy `usage_records`. That's why we use per-second pricing with integer seconds. Meter Events API supports decimals but doesn't work for multi-chatbot accounts (usage is per-customer, not per-subscription-item).

### updateSubscriptionItemPrice swaps price on same item
Stripe's `subscriptions.update({ items: [{ id: itemId, price: newPriceId }] })` changes the price in-place — the `stripeSubscriptionItemId` stays the same. No need to update it after rate changes.

## DynamoDB Patterns

### Atomic claim/unclaim for double-reporting prevention
Use `ConditionExpression: 'reportedToStripe = :false'` to atomically claim a record. If it fails with `ConditionalCheckFailedException`, another process already claimed it — skip, don't retry.

### Mark unreportable records as reported
If a chatbot is deleted, subscription cancelled, or `stripeSubscriptionItemId` missing — set `reportedToStripe: true` to stop retrying forever. Log as "unreportable" for audit.

### Paginated scans for batch jobs
DynamoDB `Scan` returns max 1MB per call. Always loop on `LastEvaluatedKey` for batch operations. At scale, replace with a sparse GSI or SQS queue.

## Frontend

### Show both rate units when they differ from Stripe
If Stripe bills per-second but the business model is per-minute, show both: primary rate (`/min`) bold, Stripe rate (`/sec`) as subtitle. Use `formatUnitCost()` and `formatUnitCostPerSec()` from `assistantBillingUtils.ts`.

### RTA billing cards should all be equal width
Use `flex: '1 0 0'` for all RTA billing cards (Usage, Unit Cost, Usage Cost, Taxes, Total) so they distribute evenly.

## General

### API responses in English, localization on frontend
Don't put Spanish strings in API response messages. Use English as the base API language.

### Don't create a Stripe Product per chatbot
Consider reusing a single platform-level product (stored in SSM/env) and only creating a new Price per rate. Avoids polluting the Stripe product catalog. (Future improvement.)

### Remove `as any` casts when types are available
If a field was added to the model interface (e.g. `stripeSubscriptionItemId` on `ChatbotModel`), drop the `(chatbot as any)` cast. The cast hides refactoring bugs.


## Query Pagination

### Always paginate DynamoDB queries that aggregate data
DynamoDB Query returns max 1MB per call. If you `reduce()` over `result.Items` without looping on `LastEvaluatedKey`, you get a partial sum. The billing dashboard will show an under-count for busy chatbots. Always paginate — same pattern as `scanUnreported()`.

## DynamoDB Attribute Removal

### Use delete on spread object, not null, for PutItem models
Setting a field to `null` in DynamoDB preserves the attribute. If the model uses PutItem (full replace via `save()`), `delete obj.field` before saving removes the attribute. For UpdateExpression-based updates, use `REMOVE field` instead.

### Always findOne() before update() on models that use PutItem
If a model's `update()` does `Object.assign(this, updates)` then `save()` (PutItem), calling `update()` on a bare `new Model({ id })` will overwrite the entire DynamoDB item with defaults. Always `findOne()` first to load the full record.


## Race Conditions

### Claim-then-report has a crash window
If the process crashes between `claimForReporting()` returning true and `reportUsage()` completing, the record is stuck as reported=true but never billed. The batch job won't retry it. Future fix: separate `claimedAt` timestamp from `reportedToStripe`, and reconcile records claimed but with no `stripeUsageRecordId` after a TTL window.

## Audit Trail

### Pass all available data to audit records at creation time
Don't create audit records before the data they reference exists. If `stripePriceId` is set in the first-time rate branch, pass it to the audit constructor — otherwise the audit has `stripePriceId: undefined` forever.

## Cycle Boundary Logic

### getCycleEnd returns inclusive last day
`getCycleEnd('2026-05-01')` returns `'2026-05-31'` (inclusive). The condition `todayStr <= cycleEnd` means 'still in current cycle — skip'. The new rate applies when `todayStr > cycleEnd` (first day of new cycle). This is correct — document it with a comment to prevent future reviewers from flagging it as off-by-one.
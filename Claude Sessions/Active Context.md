---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** feat/SL-682-audit-logs
- **Project:** silia
- **Topic:** [[Claude Sessions/silia/billing-audit-tax/2026-05-25|Billing Audit + Tax]]
- **Last updated:** 2026-05-25

## What's Happening
- US-TAX-01 implemented: automatic_tax conditional on country=US across all 5 Stripe integration points
- Removed hardcoded MX=16% taxRate — now taxRate=0 for all (Stripe handles US after invoice finalization)
- Audit log handler imports were reverted by user (handlers simplified back)
- 8 files modified total

## Current State
- automatic_tax logic: complete across StripeGateway (4 places) + StripeWebhookHandler (1 place)
- taxRate=0 defaults: createSubscription.ts + Assistant/Put/index.ts (2 places)
- Not yet committed

## How to Continue
1. Verify if user wants audit log handlers re-added or kept stripped
2. Commit and PR
3. FE tasks: US-TAX-02, US-TAX-03 (show tax_amount in UI)
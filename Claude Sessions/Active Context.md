---
tags: [active-context]
---

# Active Context

## Current Session
- **Note:** [[SL-1149 RTA Metered Scheduling/2026-05-11]]
- **Branch:** feat/SL-1146-metering-minutes
- **Project:** silia
- **Last updated:** 2026-05-11

## What's Happening
- Fixed RTA metered subscription scheduling for future dates (3 billing files on fix/SL-1149-dunning-rta)
- Removed RTA submodule noise from PR #914 — committed and pushed
- Fixed billing rounding mismatch: monthlyTotal now matches sum of displayed usageCost + taxes

## Current State
- **Uncommitted:** 6 files (settings, csvProcessor, fileValidator, yarn.lock, RTA submodule, transformers.ts)
- **transformers.ts:** Rounding fix for both RTA and conversational billing paths
- **Billing changes:** On fix/SL-1149-dunning-rta (StripeGateway, createSubscription, StripeWebhookHandler) — not yet committed

## How to Continue
1. Commit transformers.ts rounding fix
2. Switch to fix/SL-1149-dunning-rta to commit billing changes
3. Deploy and test both fixes
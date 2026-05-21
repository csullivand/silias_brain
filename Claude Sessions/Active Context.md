---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** develop
- **Project:** silia
- **Last updated:** 2026-05-21

## What's Happening
- Billing suspension + reactivation feature audit
- Two tickets analyzed: Suspensión automática + Reactivación automática post-pago

## Current State
- **Suspension audit:** Complete — 4 frontend gaps identified (banner, module access, 30-day grace, read-only)
- **Reactivation audit:** Complete — core backend done, email template basic, critical bug found
- **Critical bug:** `AccountStatusService.updateChatbotsStatus()` reactivates ALL chatbots blindly. Fix designed using `suspensionReason` field as filter. Pending implementation.

## Designed Fix: suspensionReason-based reactivation
- Add `reason?: 'dunning' | 'manual'` to `ChangeStatusInput`
- Deactivate: only touch active chatbots, mark with `suspensionReason`
- Reactivate: only restore chatbots matching the reason
- File: `Accounts/domain/services/AccountStatusService.ts`
- Callers to update: `BillingService.ts:185`, `StripeWebhookHandler.ts:897,553`

## Gaps Still to Build
1. Global suspension banner in MainLayout
2. Module access control by account status
3. 30-day grace timeline
4. Read-only mode for suspended accounts
5. Richer reactivation email template

## Recent Work
- [[Billing Suspension Audit]] — current session
- [[RTA Billing Flow End-to-End]] — billing flow reference
- [[RTA Dunning Analysis]] — dunning gap analysis
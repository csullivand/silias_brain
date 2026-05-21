---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** fix/reactivation-suspension-reason
- **Project:** silia
- **Last updated:** 2026-05-21

## What's Happening
- Implemented 2 fixes for billing reactivation ticket
- Fix 1: suspensionReason-based filter in AccountStatusService (don't reactivate user-disabled agents)
- Fix 2: RTA sync on account-level suspend/reactivate operations

## Current State
- **Implementation:** Complete, lint clean
- **Pending:** Commit confirmation from user
- **Files changed:**
  - `Accounts/domain/services/AccountStatusService.ts` — main logic (reason filter + RTA sync)
  - `Billing/application/handlers/StripeWebhookHandler.ts` — 2 callers pass reason: 'dunning'
  - `Billing/application/services/BillingService.ts` — safety-net caller passes reason: 'dunning'

## Still To Do (other tickets)
1. Global suspension banner in MainLayout
2. Module access control by account status
3. 30-day grace timeline
4. Read-only mode for suspended accounts
5. Richer reactivation email template

## Recent Work
- [[Billing Suspension Audit]] — current session
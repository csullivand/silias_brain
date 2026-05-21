---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** develop
- **Project:** silia
- **Last updated:** 2026-05-21

## What's Happening
- Billing suspension feature audit — mapping acceptance criteria to existing code
- Ticket: BILLING: FE: Suspensión automática por falta de pago

## Current State
- **Audit complete:** Full gap analysis done
- **Backend dunning/suspension:** Fully implemented (StripeWebhookHandler, AgentSuspensionService, AccountStatusService, NotificationService, safety-net cron)
- **Frontend gaps identified:** 4 items to build

## Gaps to Build
1. **Global suspension banner** in MainLayout (highest priority)
2. **Module access control by account status** — block certain modules for suspended accounts, allow Billing
3. **30-day grace timeline** — suspendedAt field + scheduled job
4. **Read-only mode** for suspended accounts

## Recent Work
- [[Billing Suspension Audit]] — full audit session
- [[Detail View Layout Implementation]] — previous work
- [[RTA Billing Flow End-to-End]] — billing flow reference
- [[RTA Dunning Analysis]] — dunning gap analysis
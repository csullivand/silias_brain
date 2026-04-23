---
tags: [claude-session, active]
updated: 2026-04-23
---

# Active Context

Updated when something changes. If disconnected, START HERE.

---

## Current Session
- **Date:** 2026-04-23
- **Project:** silia
- **Topic:** SL-678 Billing Retrocompatibility
- **Session note:** `Claude Sessions/silia/SL-678 Suspension No Payment/2026-04-23.md`
- **Branch:** feat/SL-678-suspension-no-payment

## Last Checkpoint
- **What was just done:**
  - Analyzed billing retrocompatibility for existing accounts (pre-billing)
  - Implemented auto-subscribe logic in Assistant/application/Put/index.ts
  - When chatbot billing plan is updated and chatbot has no stripeSubscriptionId:
    - Auto-onboards account (creates Stripe customer) if needed
    - Creates Stripe subscription
    - Saves stripeSubscriptionId + taxRate on chatbot
  - Non-blocking, follows existing patterns
- **Status:** Implemented, not yet tested or committed
- **Files changed this session:**
  - Assistant/application/Put/index.ts (+51 lines: AccountModel import + auto-subscribe else-if branch)
- **Next action:** Test the 4 scenarios, then commit
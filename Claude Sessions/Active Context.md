---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** feat/SL-664-pre-suspension-email
- **Project:** silia
- **Topic:** Debugging + Dunning email testing (continued)
- **Session note:** [[Claude Sessions/silia/debugging-and-dunning/2026-05-26]]
- **Last updated:** 2026-05-28

## What's Happening
- New session started, branch changed to feat/SL-664-pre-suspension-email
- Prior work: fixed findOne() bug, daysOverdue/dueDate, URL double app., sendInvoiceNotification AccountModel
- Found 3 more AccountModel.findOne() without params bugs
- 8 files with uncommitted changes carried over

## Uncommitted Changes
- .claude/settings.local.json
- Assistant/infrastructure/utils/csvProcessor.ts
- Assistant/infrastructure/utils/fileValidator.ts
- Billing/application/handlers/StripeWebhookHandler.ts
- Billing/application/services/BillingService.ts
- Billing/application/services/NotificationService.ts
- Billing/infrastructure/aws/handlers/cancelSubscription/cancelSubscription.ts
- DynamicTables/application/Tables/get/listTables.ts

## Next
- Awaiting user direction
- Pending: fix remaining 3 AccountModel.findOne() bugs, deploy to DEV, create day-6 email template
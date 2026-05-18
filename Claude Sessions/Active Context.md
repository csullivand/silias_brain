---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** feat/SL-1146-metering-minutes + fix/SL-1149-dunning-rta + feat/detail-view-layout
- **Project:** silia
- **Last updated:** 2026-05-18

## What's Happening
- DynamicTables: Detail View Layout feature — backend complete
- Column visibility sync with layout (TransactWriteItems atomic)
- Create column auto-adds to layout (leftColumn or actionsState for buttons)
- PATCH /tables/{tableId} accepts detailViewConfig with validations
- Feature 4 endpoint: POST column with default visibility table_view + layout position

## Current State
- **Detail View Layout:** Backend complete, frontend API switch pending
- **Metering PR:** Merged/deployed, working in dev
- **Dunning PR:** Implemented, pending deploy

## Recent Work
- [[Detail View Layout Implementation]] — full notes
- [[RTA Billing Flow End-to-End]] — billing flow reference
- [[RTA Dunning Analysis]] — dunning gap analysis
- [[Billing Code Review Learnings]] — patterns to follow
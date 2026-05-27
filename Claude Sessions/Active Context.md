---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** develop
- **Project:** silia
- **Topic:** Suspension FE — Account-level banner
- **Last updated:** 2026-05-27

## What's Happening
- Implemented account-level suspension banner in MainLayout
- Added status field to Account interface + AccountContext
- Inline Alert in MainLayout (no separate component — same pattern as per-agent dunning banner)

## Uncommitted Changes
- app/src/utils/accountUtils.ts: added status to Account interface
- app/src/store/AccountContext.tsx: passes status from accountDetails
- app/src/Components/layouts/MainLayout.tsx: Alert banner when status !== 'active'

## Next
- Test banner visually in browser
- Module access blocking (Config, Deploy disabled when suspended)
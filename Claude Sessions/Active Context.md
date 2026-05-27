---
tags: [active-context]
---

# Active Context

## Current Session
- **Branch:** develop
- **Project:** silia
- **Topic:** Suspension FE — Banner + Login unblock
- **Last updated:** 2026-05-27

## What's Happening
- Suspension banner in MainLayout (reverted to real check)
- Login unblock for inactive accounts
- User modified StripeWebhookHandler.ts (separate change)

## Uncommitted Changes (9 files)
- Auth/login.ts: removed account status check
- MainLayout.tsx: suspension banner
- AccountContext.tsx + accountUtils.ts: status field
- StripeWebhookHandler.ts: user modification

## Next
- Module access blocking when suspended
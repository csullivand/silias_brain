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
- Removed login block for inactive accounts (Auth/login.ts line 113)
- Added suspension banner in MainLayout (Alert with StopOutlined)
- Added status to Account interface + AccountContext
- Currently isSuspended = true (hardcoded for visual testing)

## Uncommitted Changes (8 files)
- Auth/login.ts: removed account.status !== 'active' check
- MainLayout.tsx: suspension banner (hardcoded true for testing)
- AccountContext.tsx: passes status
- accountUtils.ts: status field on Account interface

## Next
- Test banner visually
- Revert isSuspended to real check
- Module access blocking (Config, Deploy disabled when suspended)
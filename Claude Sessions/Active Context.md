---
tags: [claude-session, active-context]
updated: 2026-06-17
---

# Active Context

## Current Session
- **Project:** Silia
- **Topic:** Dynamic Tables Refactor (Phase 0-2 done)
- **Session notes:** [[Claude Sessions/silia/dynamic-tables-refactor/2026-06-17]]
- **Branch:** feat/SL-1273-folder-crud

## What Was Done
- Dynamic Tables: Phase 0 (feature flag), Phase 1 (schema + models + SAM), Phase 2 (migration script)
- New AgentTableConnection model for N:N agent-table relationship
- accountId + folderId added to DynamicTable model
- New DynamoDB table + GSI in SAM template

## Next
- Phase 3: Refactor 30+ handlers (replace requireChatbotAccess)
- Not committed yet — needs separate branch
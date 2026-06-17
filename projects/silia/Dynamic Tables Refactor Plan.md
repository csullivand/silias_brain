---
tags: [silia, dynamic-tables, refactor, plan]
created: 2026-06-16
---

# Dynamic Tables Refactor — First-Class Objects

## Problem
Tables are tightly coupled to chatbotId (1:N). 138 references across 50 files. Need to decouple to account-scoped N:N with agents.

## Phases

### Phase 0: Prep (1 day)
- Feature flag TABLES_FIRST_CLASS_ENABLED

### Phase 1: Schema (2 days)
- Add accountId + folderId to DynamicTable model
- New AgentTableConnection junction table + model
- New GSI: accountId-createdAt-index

### Phase 2: Migration (2 days)
- Idempotent, per-account, retomable script
- For each table: set accountId, folderId=null, create AgentTableConnection
- Handle orphaned tables, name conflicts
- Validation: count before = count after

### Phase 3: Handler Refactor (4 days)
- 30+ handlers: replace requireChatbotAccess with requirePermission + accountId guard
- Update publishDynamicTableEvent
- New endpoints: POST/DELETE/GET /tables/{id}/agents

### Phase 4: Cleanup (1 day)
- Remove chatbotId field, old GSI, old middleware

## Total: 10 dev days

## Decisions for Tech Lead
1. Migration: blocking per-account or zero-downtime dual-write?
2. WebSocket events: all account users or connected agent users only?
3. publishTable/unpublishTable: which PRD permission?
4. Export: table.export permission?

## Files
- Full plan: docs/dynamic-tables-refactor-plan.md
- Handler checklist: 32 handlers mapped with permissions
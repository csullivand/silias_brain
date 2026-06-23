---
tags: [project, silia, runbook, deployment]
---

# Tables Refactor Deployment Runbook

## Pre-requisites
- PR #1264 merged to develop
- All 3 stacks deployed: DynamicTables, Folders, Assistant
- AWS SSO login: `aws sso login --profile silia-engineer-operator-817389378997`

## Step 1: Run Tables Migration (per environment)

Sets `accountId` on existing tables and creates `AgentTableConnection` records.

```bash
# Dev
./scripts/migrate-tables-to-first-class.sh dev-app-silia-com silia-engineer-operator-817389378997

# Staging
./scripts/migrate-tables-to-first-class.sh staging-app-silia-com silia-engineer-operator-817389378997

# Prod
./scripts/migrate-tables-to-first-class.sh prod-app-silia-com silia-engineer-operator-817389378997
```

**What it does:**
1. Scans all DynamicTables
2. For each table without `accountId`, looks up its chatbot → gets `accountId`
3. Sets `accountId` + `folderId: null` on the table
4. Creates `AgentTableConnection` row linking table → original chatbot
5. Idempotent — safe to re-run

**Verify:** Check output shows `Validation PASSED` and all tables have `accountId`.

## Step 2: Run Folder Item Counts Backfill (per environment)

Computes and writes `itemCounts` for all folders. **Must run before non-root folders show correct content.**

```bash
# Dev
FOLDERS_TABLE=dev-app-silia-com-Folders \
CHATBOT_TABLE=dev-app-silia-com-Chatbot \
DYNAMIC_TABLES_TABLES_TABLE=dev-app-silia-com-DynamicTables-Tables \
AWS_REGION=us-east-1 \
AWS_PROFILE=silia-engineer-operator-817389378997 \
npx ts-node scripts/backfill-folder-item-counts.ts

# Staging
FOLDERS_TABLE=staging-app-silia-com-Folders \
CHATBOT_TABLE=staging-app-silia-com-Chatbot \
DYNAMIC_TABLES_TABLES_TABLE=staging-app-silia-com-DynamicTables-Tables \
AWS_REGION=us-east-1 \
AWS_PROFILE=silia-engineer-operator-817389378997 \
npx ts-node scripts/backfill-folder-item-counts.ts

# Prod
FOLDERS_TABLE=prod-app-silia-com-Folders \
CHATBOT_TABLE=prod-app-silia-com-Chatbot \
DYNAMIC_TABLES_TABLES_TABLE=prod-app-silia-com-DynamicTables-Tables \
AWS_REGION=us-east-1 \
AWS_PROFILE=silia-engineer-operator-817389378997 \
npx ts-node scripts/backfill-folder-item-counts.ts
```

**What it does:**
1. Scans all folders, chatbots, and tables
2. For each folder, counts subfolders/agents/tables with matching `folderId`
3. Writes computed `itemCounts` to the folder
4. Idempotent — skips folders with correct counts

## Step 3: Verify

```bash
# Test listTables (legacy chatbotId path)
curl -H 'Authorization: Bearer <token>' \
  'https://api.dev.app.silia.com/dynamic-tables/tables?chatbotId=<id>'

# Test listTables (new accountId path)
curl -H 'Authorization: Bearer <token>' \
  'https://api.dev.app.silia.com/dynamic-tables/tables'

# Test unified folder listing
curl -H 'Authorization: Bearer <token>' \
  'https://api.dev.app.silia.com/folders?parentFolderId=ROOT'
```

## Important Notes

- **Order matters:** Step 1 before Step 2 (backfill needs accountId on tables)
- **Non-root folders:** Until Step 2 completes, non-root folders appear empty (counters default to 0, listing skips agent/table queries)
- **ROOT level unaffected:** Root always queries unconditionally (`isRoot` bypass)
- **Re-runnable:** Both scripts are idempotent — safe to run multiple times
- **No downtime:** Scripts run while the service is live

## Rollback

If issues arise:
- The old `requireChatbotAccess` path still works for unmigrated tables (requireTableAccess has legacy fallback)
- Counter drift can be fixed by re-running the backfill
- No data is deleted by either script

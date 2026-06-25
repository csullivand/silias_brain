# DynamicTables Migration Script

## Purpose
Migrate DynamicTables from chatbot-scoped to account-scoped (first-class objects).

## Location
`scripts/migrate-tables-to-first-class.sh`

## Usage
```bash
./scripts/migrate-tables-to-first-class.sh <STACK_NAME> <AWS_PROFILE> [REGION]

# Example
./scripts/migrate-tables-to-first-class.sh dev-app-silia-com silia-engineer-operator-817389378997 us-east-1
```

## What It Does
1. **Scan** all DynamicTables
2. **For each table without accountId:**
   - Look up chatbot → get accountId
   - Set `accountId` + `folderId=null` on the table
   - Create `AgentTableConnection` linking table → original chatbot
3. **Validate** counts at the end

## Features
- **Idempotent**: skips tables that already have accountId
- **Per-account**: can be run incrementally
- **Resumable**: logs progress, resume from where it left off
- **Validation**: counts before/after to verify
- **Safe**: does NOT delete any data

## Related Script
After running this, also run the folder itemCounts backfill:
```bash
AWS_PROFILE=silia-engineer-operator-817389378997 \
FOLDERS_TABLE=dev-app-silia-com-Folders \
CHATBOT_TABLE=dev-app-silia-com-Chatbot \
DYNAMIC_TABLES_TABLES_TABLE=dev-app-silia-com-DynamicTables-Tables \
AWS_REGION=us-east-1 \
npx tsx scripts/backfill-folder-item-counts.ts
```

## Run History
- **2026-06-25**: Dev environment - 31 tables migrated, 29 connections created ✅

---
Tags: #silia #dynamictables #migration #scripts
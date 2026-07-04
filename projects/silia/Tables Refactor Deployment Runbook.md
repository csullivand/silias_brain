---
tags: [project, silia, runbook, deployment]
---

# Tables Refactor Deployment Runbook

## Overview

This runbook covers deploying the DynamicTables refactor (PR #1353) and running all necessary migrations.

## Pre-requisites

- PR #1353 merged to develop
- AWS SSO login: `aws sso login --profile silia-engineer-operator-817389378997`

---

## Step 1: Deploy Stacks

Deploy the stacks (order doesn't matter — no cross-stack exports):

```bash
# Any order is fine
cd Folders/infrastructure && sam deploy ...
cd Assistant/infrastructure && sam deploy ...
cd DynamicTables/infrastructure && sam deploy ...
cd Access/infrastructure && sam deploy ...
```

---

## Step 2: Seed Access Resources

Seeds `access.view` and `access.manage` permissions for the Access module.

```bash
# Dev (note: table names are LOWERCASE)
AWS_PROFILE=silia-engineer-operator-817389378997 \
RESOURCE_TABLE=dev-app-silia-com-resource \
PERMISSION_TABLE=dev-app-silia-com-permission \
ROLE_TABLE=dev-app-silia-com-role \
AWS_REGION=us-east-1 \
npx tsx scripts/migrations/seed-access-resources.ts

# Staging
AWS_PROFILE=silia-engineer-operator-817389378997 \
RESOURCE_TABLE=staging-app-silia-com-resource \
PERMISSION_TABLE=staging-app-silia-com-permission \
ROLE_TABLE=staging-app-silia-com-role \
AWS_REGION=us-east-1 \
npx tsx scripts/migrations/seed-access-resources.ts

# Prod
AWS_PROFILE=silia-engineer-operator-817389378997 \
RESOURCE_TABLE=prod-app-silia-com-resource \
PERMISSION_TABLE=prod-app-silia-com-permission \
ROLE_TABLE=prod-app-silia-com-role \
AWS_REGION=us-east-1 \
npx tsx scripts/migrations/seed-access-resources.ts
```

**What it seeds:**
| Resource | Granted To |
|----------|------------|
| `access.view` | admin, supervisor |
| `access.manage` | admin |

---

## Step 3: Run Tables Migration

Sets `accountId` on existing tables and creates `AgentTableConnection` records.

```bash
# Dev
./scripts/migrations/migrate-tables-to-first-class.sh dev-app-silia-com silia-engineer-operator-817389378997

# Staging
./scripts/migrations/migrate-tables-to-first-class.sh staging-app-silia-com silia-engineer-operator-817389378997

# Prod
./scripts/migrations/migrate-tables-to-first-class.sh prod-app-silia-com silia-engineer-operator-817389378997
```

**What it does:**
1. Scans all DynamicTables
2. For each table without `accountId`, looks up its chatbot → gets `accountId`
3. Sets `accountId` + `folderId: null` on the table
4. Creates `AgentTableConnection` row linking table → original chatbot
5. Idempotent — safe to re-run

**Verify:** Check output shows `Validation PASSED` and all tables have `accountId`.

---

## Step 4: Run Folder Item Counts Backfill

Computes and writes `itemCounts` for all folders.

```bash
# Dev
AWS_PROFILE=silia-engineer-operator-817389378997 \
FOLDERS_TABLE=dev-app-silia-com-Folders \
CHATBOT_TABLE=dev-app-silia-com-Chatbot \
DYNAMIC_TABLES_TABLES_TABLE=dev-app-silia-com-DynamicTables-Tables \
AWS_REGION=us-east-1 \
npx tsx scripts/migrations/backfill-folder-item-counts.ts
```

---

## Step 5: Run Table Row Counts Backfill

Initializes `activeRowCount` on META rows for existing tables.

```bash
# Dev (dry run first)
AWS_PROFILE=silia-engineer-operator-817389378997 \
DYNAMIC_TABLES_TABLES_TABLE=dev-app-silia-com-DynamicTables-Tables \
DYNAMIC_TABLES_ROWS_TABLE=dev-app-silia-com-DynamicTables-Rows \
AWS_REGION=us-east-1 \
DRY_RUN=true \
npx tsx scripts/migrations/backfill-table-row-counts.ts

# Dev (actual run — remove DRY_RUN)
```

---

## Step 6: Verify

```bash
# Test unified folder listing with rowCount
curl -H 'Authorization: Bearer <token>' \
  'https://api.dev.app.silia.com/folders?parentFolderId=ROOT'

# Test Access grants endpoint
curl -H 'Authorization: Bearer <token>' \
  'https://api.dev.app.silia.com/access/grants'
```

---

## Scripts Summary

| Script | Purpose |
|--------|---------|
| `seed-access-resources.ts` | Seed access.view/access.manage permissions |
| `migrate-tables-to-first-class.sh` | Add accountId to tables, create AgentTableConnection |
| `backfill-folder-item-counts.ts` | Initialize folder itemCounts |
| `backfill-table-row-counts.ts` | Initialize table activeRowCount |

---

## Run History

| Date | Environment | Script | Result |
|------|-------------|--------|--------|
| 2026-06-25 | Dev | migrate-tables-to-first-class.sh | ✅ 31 tables, 29 connections |
| 2026-07-03 | Dev | seed-access-resources.ts | ✅ 2 resources, 3 permissions |

---
Related: [[Dynamic Tables Refactor Plan]] [[DynamicTables Migration Script]] [[SAM Template Patterns]]
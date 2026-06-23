---
tags: [project, silia, guide, refactor, dynamictables, folders]
updated: 2026-06-23
---

# Tables Refactor — Complete Guide

## Overview

PR #1264 refactors DynamicTables from chatbot-scoped (1:N) to account-scoped first-class workspace objects with N:N agent connections. This guide covers the full deployment, migration, testing, and troubleshooting process.

---

## Part 1: Pre-Deployment Checklist

### Code Changes Included
- [ ] `requireChatbotAccess` → `requireTableAccess` across 28 handlers
- [ ] `AgentTableConnectionModel` for N:N agent-table linkage
- [ ] `folderId` on ChatbotModel + DynamicTableModel
- [ ] `itemCounts` on FolderModel with `safeIncrementItemCount`
- [ ] Unified listing: `GET /folders?parentFolderId=ROOT` returns folders + agents + tables
- [ ] Cross-stack IAM + KMS + env vars for Folders↔Assistant↔DynamicTables
- [ ] `encoder.json` + `vocab.bpe` in DynamicTables and Folders modules
- [ ] Migration scripts: `migrate-tables-to-first-class.sh` + `backfill-folder-item-counts.ts`

### Infrastructure Requirements
- [ ] DynamicTables stack: new `AgentTableConnections` DynamoDB table + KMS key + GSIs
- [ ] DynamicTables stack: `FOLDERS_TABLE` env var + IAM GetItem/UpdateItem + KMS on Folders table
- [ ] Assistant stack: `FOLDERS_TABLE` env var + IAM GetItem/UpdateItem + KMS on Folders table
- [ ] Folders stack: `CHATBOT_TABLE` + `DYNAMIC_TABLES_TABLES_TABLE` env vars + IAM Query/Scan on both tables
- [ ] No fixed `RoleName` on `DtAgentConnectionRole` (auto-generated to prevent orphan collisions)

### Pre-Deploy Checks
```bash
# Login to AWS
aws sso login --profile silia-engineer-operator-817389378997

# Verify no orphaned resources that would block deployment
aws dynamodb describe-table --table-name <StackName>-AgentTableConnections --profile silia-engineer-operator-817389378997 2>&1 | head -3
aws kms list-aliases --query 'Aliases[?AliasName==`alias/<StackName>-agent-table-connections`]' --profile silia-engineer-operator-817389378997
# Both should return errors/empty — if they exist, clean up before deploying
```

---

## Part 2: Deployment (Per Environment)

### Order of Stack Deployment
1. **DynamicTables** — creates AgentTableConnections table, updates handlers
2. **Folders** — adds cross-stack env vars + IAM for unified listing
3. **Assistant** — adds folderId support + folder counter wiring

### Deploy Commands

#### Dev
```bash
# 1. DynamicTables
cd DynamicTables/infrastructure && yarn deploy-dev

# 2. Folders
cd Folders/infrastructure && yarn deploy-dev

# 3. Assistant
cd Assistant/infrastructure && yarn deploy-dev
```

#### Staging
```bash
cd DynamicTables/infrastructure && yarn deploy-staging
cd Folders/infrastructure && yarn deploy-staging
cd Assistant/infrastructure && yarn deploy-staging
```

#### Prod
```bash
cd DynamicTables/infrastructure && yarn deploy-prod
cd Folders/infrastructure && yarn deploy-prod
cd Assistant/infrastructure && yarn deploy-prod
```

### Verify Deployment
```bash
# Check Lambda functions updated (check LastModified)
aws lambda get-function --function-name <StackName>-tables-get-list --query 'Configuration.LastModified' --profile silia-engineer-operator-817389378997

# Check new table exists
aws dynamodb describe-table --table-name <StackName>-AgentTableConnections --profile silia-engineer-operator-817389378997 --query 'Table.TableStatus'
```

---

## Part 3: Migration Scripts

### Step 1: Tables Migration

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
5. Validates: count of tables with accountId should match total (minus orphans)

**Expected output:**
```
Tables total:           N
Migrated this run:      N
Already migrated:       0
Orphaned (no chatbot):  0
Connections created:    N
Tables with accountId:  N/N
✅ Validation PASSED
```

**Troubleshooting:**
- `Orphaned (no chatbot)`: Table's chatbot was deleted. Table goes to root level.
- `Errors`: Check AWS CLI credentials/profile
- Re-run is safe (idempotent — skips already migrated)

### Step 2: Folder Item Counts Backfill

**MUST run after Step 1.** Computes and writes `itemCounts` for all folders.

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
2. For each folder, counts items with matching `folderId`
3. Writes computed `{ folders: N, agents: N, tables: N }` to each folder
4. Skips folders with already-correct counts

**Expected output:**
```
Found N active folders
Found N chatbots
Found N tables
Backfill complete: N folders updated out of N
```

**Important:** Until this runs, non-root folders appear empty (counters = 0, listing skips queries). ROOT level always works (isRoot bypass).

---

## Part 4: Testing

### 4.1 List Tables (Legacy Path)
```bash
# Should return tables for the chatbot
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/dynamic-tables/tables?chatbotId=<chatbot-id>' | jq '.count'
```
**Expected:** Same tables as before the refactor.

### 4.2 List Tables (New Account Path)
```bash
# Should return ALL tables for the account
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/dynamic-tables/tables' | jq '.count'
```
**Expected:** All tables across all chatbots in the account.

### 4.3 List Tables (Folder Filter)
```bash
# Root-level tables
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/dynamic-tables/tables?folderId=ROOT' | jq '.count'

# Tables in a specific folder
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/dynamic-tables/tables?folderId=<folder-id>' | jq '.count'
```

### 4.4 Unified Folder Listing
```bash
# Root level — should return folders + agents + tables
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/folders?parentFolderId=ROOT' | jq '.data[] | {type, name}'

# Inside a folder
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/folders?parentFolderId=<folder-id>' | jq '.data[] | {type, name}'
```
**Expected:** Each item has `type: "folder" | "agent" | "table"`.

### 4.5 Agent-Table Connections
```bash
# List connected agents for a table
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/dynamic-tables/tables/<table-id>/agents' | jq '.count'

# Connect an agent to a table
curl -s -X POST -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  'https://api.<env>.app.silia.com/dynamic-tables/tables/<table-id>/agents' \
  -d '{"agentId": "<chatbot-id>"}' | jq '.'

# Disconnect
curl -s -X DELETE -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/dynamic-tables/tables/<table-id>/agents/<agent-id>'
```

### 4.6 Chatbot folderId
```bash
# List chatbots filtered by folder
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/chatbot/account/<account-id>?folderId=ROOT' | jq '.chatbots | length'

# Move chatbot to a folder
curl -s -X PUT -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  'https://api.<env>.app.silia.com/chatbot/<chatbot-id>' \
  -d '{"folderId": "<folder-id>"}' | jq '.folderId'
```

### 4.7 Document Operations (Regression)
```bash
# These should still work after the auth refactor:
# - Upload document (presigned)
# - Confirm document upload
# - Get document access URL
# - Delete document
# Test via the Data Views UI in the app
```

### 4.8 Folder Item Counts
```bash
# After creating an agent in a folder, check the folder's counts updated
curl -s -H 'Authorization: Bearer <token>' \
  'https://api.<env>.app.silia.com/folders/<folder-id>' | jq '.itemCounts'
```
**Expected:** `{ "folders": N, "agents": N, "tables": N }`

---

## Part 5: Troubleshooting

### 502 Bad Gateway / Lambda Crash
- **encoder.json / vocab.bpe missing:** Check `DynamicTables/application/encoder.json` and `Folders/application/encoder.json` exist. Rebuild dist.
- **Check CloudWatch logs:** `aws logs tail /aws/lambda/<function-name> --since 5m --profile <profile>`

### 403 Permission Denied
- **assertPermission blocking:** The user needs the CASL permission (e.g., `table.view`). Check the user's role has the permission in the permissions catalog.
- **Superuser should bypass:** `assertPermission` bypasses for `superuser` and `superadmin` roles.
- **Local testing:** SAM local without authorizer won't have role info. Set up local authorizer or test against deployed API.

### Empty Folder Listings
- **Non-root folders empty:** Run the backfill script (Step 2). Counters default to 0, which causes the listing to skip agent/table queries.
- **ROOT always works:** The `isRoot` bypass fetches agents/tables unconditionally.

### Tables Not Showing accountId
- **Migration not run:** Run `migrate-tables-to-first-class.sh`
- **Orphaned tables:** Tables whose chatbot was deleted. They stay at root level with no accountId.

### Counter Drift
- **Re-run backfill:** `backfill-folder-item-counts.ts` recomputes all counts from source data.
- **Counters are best-effort:** All counter updates use try/catch — failures are logged as warnings, never block operations.

### KMS Access Denied
- **safeIncrementItemCount fails:** Check IAM role has `kms:Decrypt, kms:DescribeKey, kms:Encrypt, kms:GenerateDataKey` for the Folders KMS key alias.
- **Alias ARN format:** `arn:aws:kms:REGION:ACCOUNT:alias/STACKNAME-folders`

### Cross-Tenant Access
- **requireTableAccess 3-way check:** accountId match → chatbot ownership fallback → deny orphans
- **safeIncrementItemCount:** Validates folder.accountId === callerAccountId before counter write
- **connectAgent:** Validates agent.accountId === access.accountId before linking

---

## Part 6: Rollback Plan

1. **Code rollback:** Revert to previous commit. `requireTableAccess` has legacy fallback — unmigrated tables still work via chatbot ownership.
2. **Data is safe:** Migration scripts are additive — they add `accountId` and connections without deleting anything. Old `chatbotId` field is preserved.
3. **Counter fix:** Re-run `backfill-folder-item-counts.ts` to reconcile any drift.
4. **New table/KMS:** `AgentTableConnections` table and KMS key remain (standard CloudFormation rollback behavior). No data loss.

---

## Part 7: Local Development

### Running DynamicTables Locally
```bash
# Build the handlers
cd DynamicTables/infrastructure && yarn build

# Start local API (requires Docker)
cd DynamicTables/infrastructure && yarn start-api

# OR with specific port
sam local start-api -t aws.template.yml --env-vars env.json --parameter-overrides Stage=local --port 3001
```

### Local Authorizer Setup
The new `assertPermission` middleware requires an authorizer. Without it, all requests get 403.

1. Build auth middleware: `cd Auth/infrastructure && yarn build:prod-Auth-Middleware`
2. Add `LocalAuthFunction` to DynamicTables template (see local-authorizer-testing skill)
3. Add `LocalAuthFunction` entry to `env.json` with USER_TABLE, STACK_NAME, etc.
4. Point authorizer to `!GetAtt LocalAuthFunction.Arn`
5. **Do NOT commit** these local changes

### Required env.json entries
```json
{
  "LocalAuthFunction": {
    "REGION": "us-east-1",
    "STACK_NAME": "dev-app-silia-com",
    "USER_TABLE": "dev-app-silia-com-User",
    "STAGE": "dev"
  },
  "TablesGetList": {
    "REGION": "us-east-1",
    "STACK_NAME": "dev-app-silia-com",
    "DYNAMIC_TABLES_TABLES_TABLE": "dev-app-silia-com-DynamicTables-Tables",
    "CHATBOT_TABLE": "dev-app-silia-com-Chatbot",
    "FOLDERS_TABLE": "dev-app-silia-com-Folders"
  }
}
```

---
tags: [claude-session, active-context]
updated: 2026-06-30
---

# Active Context

## Current Session
- **Project:** Silia
- **Topic:** Feature 6 - Permission Cache Invalidation on Object Moves
- **Session notes:** [[Claude Sessions/silia/permission-invalidation/2026-06-30]]
- **Branch:** feat/SL-1274-refactor-tablas-dinamicas

## What Was Done (2026-06-30)

### Feature 6: Permission Cache Invalidation — implemented earlier; TODAY = infra/template review

Implementation (done in prior session): automatic cache invalidation when objects (folders, agents, tables) move between folders. New `Access/domain/services/PermissionInvalidationService.ts`, SQS processor, 25 passing tests. Hooks added to `Folders/.../moveFolder.ts`, `Assistant/application/Put/index.ts`, `DynamicTables/.../patch/updateTable.ts`.

### TODAY: Debugged "PATCH /dynamic-tables/tables/{id} → no CloudWatch logs" → reviewed DynamicTables template

**Root-cause findings (analysis only, NO code changed today):**

- 🔴 **Finding 1 — Deploy-time blocker (most likely root cause):** The template diff adds `PERMISSION_INVALIDATION_QUEUE_URL: !ImportValue ${StackName}-permission-invalidation-queue-url` to `Globals` (DynamicTables + Folders + Assistant). That export is owned ONLY by the **untracked, not-yet-deployed** `Access` stack (`Access/.../aws.template.yml:436`). Until Access is deployed, DynamoTables/Folders/Assistant stacks FAIL to deploy (CFN: "No export named ... found") → new code never lands → explains no logs.
- 🟠 **Finding 2 — Runtime IAM gap:** No DT role grants `dynamodb:GetItem/Query` on `AccessGrant`/`TeamUser` or `sqs:SendMessage` on the queue. Bites non-superusers; current superuser token masks it (bypass @ `requirePermission.ts:383`).
- 🟡 **Finding 3 — Blast radius:** import is in `Globals` → all ~40 functions hard-depend on it.

**Route wiring itself is correct** (`TablesPatch` @ template:1563). Secondary live-symptom candidate: PATCH body fails `UpdateTableModel` (`ValidateBody:true`, `minProperties:1`) → 400 with no logs.

## Pending
1. **Confirm Finding 1:** `aws cloudformation list-exports --query "Exports[?Name=='dev-app-silia-com-permission-invalidation-queue-url']"` (+ check DynamicTables dev stack for ROLLBACK).
2. **Deploy ordering:** deploy `Access` stack to dev FIRST, verify export, THEN Folders/Assistant/DynamicTables.
3. **Fix IAM (Finding 2):** add AccessGrant/TeamUser read + SQS SendMessage to DtTableWriteRole (+ peers); import queue ARN via `${StackName}-permission-invalidation-queue-arn`.
4. **Get PATCH status code** to disambiguate deploy-blocker vs 400 body-validation.
5. Commit changes; include in PR #1353.

## Related Notes
- [[Dynamic Tables Refactor Plan]]
- [[Permission Invalidation Design]]
- [[CASL Authorization Plan]]
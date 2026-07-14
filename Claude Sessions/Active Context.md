---
tags: [claude-session, active-context]
updated: 2026-07-14
---

# Active Context

## Current Session
- **Project:** Silia
- **Topic:** CASL IAM grants for Folders & DynamicTables (admin delete fix)
- **Branch:** develop (edits uncommitted)
- **Session note:** [[Claude Sessions/silia/casl-iam-role-tables/2026-07-14]]

## What Was Done (2026-07-14)

### Root cause
Non-superuser **admins** got `AccessDeniedException` on `createFolder` (and could not delete folders/tables). Cause: the CASL permission check does `dynamodb:Scan` on `${StackName}-role`, but the Lambda exec roles were not granted the CASL tables (`-role`, `-permission`, `-resource`). Superusers short-circuit to `manage:all` and skip the DB read — hence "superuser can but admin doesn't". There is NO "last item" delete rule anywhere; the requirement was really "unblock admins".

### Fix (IAM only, matches Assistant/Access pattern)
Added the canonical 3-statement CASL grant (Scan on role, Query on permission+index, GetItem on resource; no KMS needed):
- `Folders/infrastructure/aws.template.yml` — `FolderReadRole` + `FolderWriteRole`
- `DynamicTables/infrastructure/aws.template.yml` — 10 CASL roles: table read/write, column read/write, row read/write, document, agent-connection, filterbar-config, assistant-suggestions

Agents were never affected (delete uses `assertRole` = JWT only, no DB).

`cfn-lint` clean on both (only a pre-existing unrelated `AWS::Serverless::Api SecurityPolicy` warning).

## Pending
1. Branch + commit both templates (suggested: `fix/casl-iam-role-tables-folders-dynamictables`).
2. Deploy Folders + DynamicTables stacks.
3. Test as an admin: create/list/delete folder + table incl. the last one.
4. Audit other modules using `assertPermission`/`requireTableAccess` for the same missing grant.

## Related Notes
- [[Claude Sessions/silia/casl-iam-role-tables/2026-07-14]]
- [[SAM Template Patterns]]
- Previous: [[Claude Sessions/silia/access-deployment/2026-07-03]] (Access module deploy, same table naming)
---
tags: [claude-session, active-context]
updated: 2026-07-03
---

# Active Context

## Current Session
- **Project:** Silia
- **Topic:** Feature 2 - Visible Objects Endpoint (`visible_to=me`)
- **Branch:** fix/SL-1278-endpoint-objetos-visibles-usuario

## What Was Done (2026-07-03)

### Feature 2: Visibility Filter — COMPLETE ✅

Implemented `?visible_to=me` query parameter on `GET /folders` endpoint.

**New Files Created:**
- `Access/domain/services/VisibilityResolver.ts` — Core visibility resolution
- `Access/domain/services/__tests__/VisibilityResolver.test.ts` — 15 unit tests

**Modified Files:**
- `Folders/application/Folders/get/listFolders.ts` — Added visible_to filter
- `Folders/domain/models/Folder.model.ts` — Added findByIds() method
- `Teams/domain/models/TeamUser.model.ts` — Fixed to use GSI queries
- `Folders/infrastructure/aws.template.yml` — Added IAM for AccessGrant + TeamUser
- `tsconfig.eslint.json` — Added Access + Folders modules
- Postman collections updated

**Key Features:**
- Direct grants, team grants, folder inheritance (up to 5 levels)
- Admin/Superuser/Superadmin bypass
- Graceful degradation on errors (fail-open)
- Tenant isolation via accountId filtering

**Commits:**
- `a17cc3a31` — feat(folders): add visible_to=me filter
- `a04a285c7` — fix(folders): add IAM permissions and graceful degradation

**Documentation:**
- Created `docs/folders-and-permissions-frontend-guide.md`
- Added to Obsidian: [[Folders and Permissions Frontend Guide]]

## Tests
- 15 unit tests passing
- 28 scenario tests passing
- 5 filterVisibleItems tests passing

## Adversarial Verify
All 5 lenses PASS ✅ after fixes

## Related Notes
- [[Dynamic Tables Refactor Plan]]
- [[Tables Refactor Deployment Runbook]]
- [[Folders and Permissions Frontend Guide]]
---
tags: [concept, silia, rbac, deployment, migration, gotcha]
---

# Deploy Before Role Migration

**Rule:** When a data migration renames/consolidates a role, DEPLOY the code that recognizes the new role FIRST, then run the migration. Migrating ahead of the deploy locks users out.

## Why (the SL-1557 incident, 2026-08-04)
- Migrated `superuser → superadmin` in DEV data before deploying superadmin support.
- `assertRole` (shared/middleware/requireRole.ts) does a **strict allow-list check** and (before the fix) never bypassed super roles. ~39 handler allow-lists list `SUPERUSER` but not `SUPERADMIN`.
- Result: every migrated user got `403 "You do not have permission to perform this action"` (the exact string from assertRole's buildForbiddenResponse — distinguishes it from assertPermission and the API GW authorizer).

## Key facts
- `assertPermission` DID bypass superadmin (god-mode `can('manage','all')`); `assertRole` did NOT — asymmetry was the bug. Fix: super-role bypass added to assertRole.
- `getCallerRole` reads `event.requestContext.authorizer.role` (raw string from JWT). Token role is minted at login from the DB user's role → a user must **log out/in** after a role change to get a token reflecting the new role.
- Roles are canonical lowercase; comparisons are exact-string (`=== Role.SUPERADMIN`).

## Checklist for future role migrations
1. Land + deploy the code that treats the new role correctly (allow-lists / bypass / CASL).
2. Run migration DRY_RUN, review counts.
3. Run migration for real (back up table first if prod).
4. Affected users re-authenticate to refresh their JWT.

Related: [[Claude Sessions/silia/SL-1557-custom-roles/2026-08-04]]
# Claude Sessions Index

Master index of all Claude Code work sessions.
Organized by project > topic/ticket > dated sessions.

---

## Projects

### silia
Repo: `/Users/sulli/Projects/silia/06-11-25/silia/`

#### Access Module Deployment
- **Branch:** fix/SL-1278-endpoint-objetos-visibles-usuario
- **Status:** Complete — template fixed, resources seeded
- **Sessions:**
  - [[Claude Sessions/silia/access-deployment/2026-07-03|2026-07-03]] — Fixed SQS VisibilityTimeout, seeded access.view/access.manage permissions

#### Permission Cache Invalidation (Feature 6)
- **Branch:** feat/SL-1274-refactor-tablas-dinamicas
- **Status:** Complete — 25 tests passing, PR reviewed, ready to commit
- **Sessions:**
  - [[Claude Sessions/silia/permission-invalidation/2026-06-30|2026-06-30]] — Full implementation: PermissionInvalidationService, SQS processor, hooks in 3 modules

#### DynamicTables Refactor PR #1353
- **Branch:** feat/SL-1273-folder-crud
- **Status:** In progress — rowCount feature complete, pending deploy
- **Sessions:**
  - [[Claude Sessions/silia/refactor-tables-pr/2026-06-26|2026-06-26]] — Fixed Adversarial Verify BLOCK issues (env var + deploy order)
  - [[Claude Sessions/silia/refactor-tables-pr/2026-06-23|2026-06-23]] — Encoder/vocab fix, infra cross-stack, incrementItemCount CAS

#### Initial Obsidian Setup
- **Branch:** develop
- **Status:** Complete
- **Sessions:**
  - [[Claude Sessions/silia/Initial Obsidian Setup/2026-04-09|2026-04-09]] — Set up full Obsidian session tracking system

#### Close Conversation Endpoint
- **Branch:** feat/SL-1144-close-conversation-endpoint
- **Status:** Complete
- **Sessions:**
  - [[Claude Sessions/silia/Close Conversation Endpoint/2026-04-13|2026-04-13]] — Created PUT /conversation/{id}/close

#### SL-1162 Template Security
- **Branch:** fix/SL-1162-template-security
- **Status:** Complete
- **Sessions:**
  - [[Claude Sessions/silia/SL-1162 Template Security/2026-04-14|2026-04-14]] — Fixed API Gateway model mismatches, security lint

#### SL-1146 Metering Minutes
- **Branch:** feat/SL-1146-metering-minutes
- **Status:** Complete
- **Sessions:**
  - [[Claude Sessions/silia/SL-1146 Metering Minutes/2026-05-04|2026-05-04]] — IAM for BillingRateAudit table

#### SL-1149 RTA Metered Scheduling
- **Branch:** fix/SL-1149-dunning-rta
- **Status:** In progress — code complete, pending deploy/test
- **Sessions:**
  - [[Claude Sessions/silia/SL-1149 RTA Metered Scheduling/2026-05-11|2026-05-11]] — Fixed RTA metered subscriptions

#### Billing Suspension Audit
- **Branch:** develop
- **Status:** In progress — audit complete, implementation pending
- **Sessions:**
  - [[Claude Sessions/silia/Billing Suspension Audit/2026-05-21|2026-05-21]] — Full codebase audit of suspension feature

#### SL-682 Billing Audit + Tax + RTA Sync
- **Branch:** feat/SL-682-audit-logs
- **Status:** Complete — PRs merged
- **Sessions:**
  - [[Claude Sessions/silia/billing-audit-tax/2026-05-25|2026-05-25]] — Audit log PR fixes, US-TAX-01, RTA sync, security hardening

#### Debugging + Dunning Email Testing
- **Branch:** feat/SL-1296-tax-management
- **Status:** In progress — findOne bug fixed, template data fixed, pending deploy
- **Sessions:**
  - [[Claude Sessions/silia/debugging-and-dunning/2026-05-26|2026-05-26]] — Multi-day: prod chatbot fix, escalation logger bug, dunning email testing

#### SL-1178 Suspension FE: Banner + Module Blocking
- **Branch:** feat/SL-1178-account-suspended-banner
- **Status:** In progress — banner PR created, module blocking implemented
- **Sessions:**
  - [[Claude Sessions/silia/suspension-fe/2026-05-27|2026-05-27]] — Suspension banner, login unblock, module access blocking (sidebar + routes)

#### Folders CRUD Backend Module
- **Branch:** TBD (needs own branch)
- **Status:** In progress — code complete, not compiled/tested
- **Sessions:**
  - [[Claude Sessions/silia/folders-crud-module/2026-06-01|2026-06-01]] — Full Folders/ module: 14 files, DynamoDB model, 6 REST endpoints, SQS async, SAM template

#### Filter Bar Config Backend
- **Branch:** TBD
- **Status:** In progress — code complete, not compiled/tested
- **Sessions:**
  - [[Claude Sessions/silia/filter-bar-config/2026-06-02|2026-06-02]] — GET/PUT endpoints in DynamicTables module, per-user per-table column config

#### CASL Authorization POC
- **Branch:** feat/SL-1318-filter-bar-configuration-model
- **Status:** Complete — 3 files, 0 new tables, 15 tests passing
- **Sessions:**
  - [[Claude Sessions/silia/casl-authorization-poc/2026-06-03|2026-06-03]] — CASL POC complete: uses existing Role/Permission/Resource tables, 15 tests passing

---

### loteria
Repo: `/Users/sulli/Documents/PersonalWork/Loteria/loteria-suerte/`

#### Setup Session Tracking
- **Status:** In progress
- **Sessions:**
  - [[Claude Sessions/loteria/Setup Session Tracking/2026-05-13|2026-05-13]] — Set up Obsidian session tracking
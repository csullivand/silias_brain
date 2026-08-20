# Claude Sessions Index

Master index of all Claude Code work sessions.
Organized by project > topic/ticket > dated sessions.

---

## Projects

### silia
Repo: `/Users/sulli/Projects/silia/06-11-25/silia/`

#### Feature 4.3 / SL-1477 — Kanban view config (BE)
- **Branch:** feat/SL-1477-persistencia-vista-usuario-kanban · **commit a079d4d0e** (local, sin push)
- **Status:** COMPLETO + reviews PASS (adversarial ✅ + PR review ⚠️) + commiteado (solo DynamicTables, 11 archivos +924, 32 tests). NO deployado. Doc FE fuera del commit. Decisión clave: config COMPARTIDA por tabla (reemplaza al PRD por-usuario); orden manual de tarjetas NO persiste.
- **Sessions:**
  - [[Claude Sessions/silia/Feature-4.3-kanban-view-config/2026-08-17|2026-08-17/19]] — store per-tabla (PK=tableId, sin userId/GSI); GET table.view / PUT table.column.edit; validador de enums; cascade-delete; infra SAM; fix updatedBy @IsOptional tras reviews

#### SL-1576 Feature 2.1 — Row Fill Rules persistence (BE)
- **Branch:** feat/SL-1576-persistencia-reglas-fill · **PR #1991** (base develop)
- **Status:** Implementado + reviews PASS; 4 commits pusheados; NO deployado. Doc FE removido del PR.
- **Sessions:**
  - [[Claude Sessions/silia/SL-1576-row-fill-rules/2026-08-14|2026-08-14]] — coloreado condicional de filas per-usuario; validador por-id; store per-user + GSI + cascade-delete; paleta 10; 400+errorCode, updatedAt segundos

#### SL-1432 Feature 7 — Detail-view reconcile fix
- **Branch:** feat/SL-1432-user-level-detail-config
- **Status:** 3 commits pusheados; 60 tests; NO verificado en vivo (falta sam deploy)
- **Sessions:**
  - [[Claude Sessions/silia/SL-1432-detail-view-reconcile/2026-08-13|2026-08-13/14]] — reconcile reponía columnas quitadas; fix con sello propio del layout

#### Data Views FE contracts + fix role_name
- **Status:** Fix role_name COMPLETO (15/15 tests); contratos FE de Feature 4.3 y 2.1 entregados
- **Sessions:**
  - [[Claude Sessions/silia/data-views-contracts-grants-rolename/2026-08-14|2026-08-14]] — role_name en GET /access/grants; docs FE kanban-view-config (4.3) y row-fill-rules (2.1)

#### Escalación forzada por admin (inbox)
- **Branch:** feat/SL-1579-force-escalation — IMPLEMENTADO (backend), 23/23 tests, sin commitear
- **Sessions:**
  - [[Claude Sessions/silia/escalacion-forzada-admin/2026-08-06|2026-08-06]] — Análisis flujo escalación→ticket→asignación
  - [[Claude Sessions/silia/SL-1579-manual-escalation-debug/2026-08-07|2026-08-07]] — Depuración cadena de bugs; bloqueo: falta sam deploy

#### SL-1528 KPI Aggregations (Feature 6)
- **Branch:** feat/SL-1528-agregaciones-dataset-kpis
- **Status:** Complete — commit 943b6902b, 17/17 tests, pr-review + adversarial PASS, no push

#### Feature 11 / SL-1290 — Role dropdowns con roles custom (BE)
- **Branch:** (sin rama propia; en working tree sobre feat/SL-1477) · SIN commit
- **Status:** IMPLEMENTADO + 2 rondas review (PR Approved/Changes-fixed + adversarial PASS). 75 tests. NO deployado. Ticket FE, se implementaron los gaps de BE.
- **Sessions:**
  - [[Claude Sessions/silia/SL-1290-role-dropdowns/2026-08-19|2026-08-19]] — fix seguridad grants (cross-tenant + no-asignables); GET /role ?assignable&light; POST /role/{id}/reassign (reasignación masiva); asignabilidad allowlist relativa al caller (superadmin asigna todo); doc FE
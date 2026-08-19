# Active Context

## Sesión actual: [Feature 4.3][BE] Kanban View Config (SL-1477) — COMPLETA + COMMITEADA
Nota: [[Claude Sessions/silia/Feature-4.3-kanban-view-config/2026-08-17]]

**Rama feat/SL-1477-persistencia-vista-usuario-kanban · commit a079d4d0e (local, SIN push) · 11 archivos +924 · 32 tests verdes.**

Config de vista Kanban **COMPARTIDA por tabla** (una sola para todos, NO per-usuario): blob JSON (groupByColumnId, laneOrder, laneColors, showUncategorized, cardTitleColumnId, cardFields[], laneSorts, summaryAggs). Espejo simplificado de SL-1576 RowFillRules.

### Hecho
- Store PK=tableId (sin userId/GSI); GET/PUT /tables/{tableId}/kanban-view-config; GET table.view, PUT table.column.edit; updatedAt SEGUNDOS; error 400+errorCode; guarda VERBATIM (FE reconcilia).
- Validador solo enums (direction asc/desc, calcs x13, __no_status__) + forma.
- Cascade-delete fail-open en deleteTable.ts. Infra SAM: KMS, tabla sin GSI, env var, DtKanbanViewConfigRole (CASL+AccessGrant+KMS), 2 Lambdas, RequestModel.
- Reviews: adversarial verify PASS ✅ + PR review Approved-with-suggestions ⚠️. Convergieron en updatedBy sin @IsOptional → FIX + test de regresión.
- Commit a079d4d0e (Conventional Commits, scope eca, hooks OK). SOLO DynamicTables.

### ⚠️ Pendiente
1. git push + PR (descripción no técnica ya lista).
2. sam deploy a dev + round-trip real.
3. Doc FE docs/kanban-view-config-api.md actualizado (compartida/seg/400/table.column.edit) pero FUERA del commit — decidir si va aparte (como SL-1576).
4. Corregir PRD/ticket: 'por-usuario' → 'compartida por tabla'.

### Decisiones clave (producto)
- Kanban config = COMPARTIDA por tabla (reemplaza al PRD que dice por-usuario; 'el PRD está mal, se corrige').
- Orden manual de tarjetas NO persiste (PRD fuera de alcance). Sin rank.
- Convención de casa (seg/400/table.column.edit), no el doc congelado original (ms/422/table.view).

### Contexto previo
- SL-1576 Feature 2.1 Row Fill Rules — PR #1991, reviews PASS, NO deployado. [[Claude Sessions/silia/SL-1576-row-fill-rules/2026-08-14]]
- SL-1432 Feature 7 detail-view reconcile. [[Claude Sessions/silia/SL-1432-detail-view-reconcile/2026-08-13]]
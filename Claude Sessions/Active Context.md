# Active Context

## Sesión actual: Data Views FE contracts + fix role_name (2026-08-14)
Nota: [[Claude Sessions/silia/data-views-contracts-grants-rolename/2026-08-14]]

**Rama `feat/SL-1432-user-level-detail-config` — cambios en working tree, SIN commitear (política no-auto-commit).**

### Hecho esta sesión
1. **Fix bug (Antonio):** `GET /access/grants` ahora devuelve `role_name` además de `role_id`. Helper `attachRoleNames` en `Access/application/Grants/get/listGrants.ts` (espejo de attachSubjectNames), en ambas rutas. Sin infra/IAM. **15/15 tests.** PR desc no-técnica lista.
2. **Contrato FE Feature 4.3 (Kanban view config):** `docs/kanban-view-config-api.md`. Recomendación: store dedicado estilo KpiPrefs (PUT blob = last-write-wins), `GET/PUT /dynamic-tables/tables/{tableId}/kanban-view-config`, permiso table.view.
3. **Contrato FE Feature 2.1 (Row Fill rules):** `docs/row-fill-rules-api.md`. Regla = condición de filtro `{column, op, value, valueTo, values}` + `color` (reusa ALLOWED_OPERATORS). Endpoint dedicado `GET/PUT /dynamic-tables/tables/{tableId}/row-fill-rules`.

### Pendiente
- Commit/push del fix role_name (esperando OK).
- BE de 4.3 y 2.1 NO implementados — solo contratos.
- Row Fill: 2 open questions de producto (por-vista vs por-tabla; reordenar reglas con drag).

### Aprendizajes reutilizables
- **Filtros/orden de Data Views NO se persisten server-side** (son query-string). Persistencia = shared (registro table) o per-user stores (KpiPrefs/FilterBarConfig/UserViewConfig, key userId+tableId, GSI tableId-index para cascade).
- **Motor de filtros ya server-side:** `ALLOWED_OPERATORS` + `validateAndResolveFilters` en DynamicTables/domain. Paleta de color es FE-only.
- **GitHub Desktop + husky:** node de nvm no está en el PATH mínimo de GH Desktop → `~/.config/husky/init.sh` carga nvm.

### Contexto previo
- SL-1579 Manual escalation — bugs, falta sam deploy real. [[Claude Sessions/silia/SL-1579-manual-escalation-debug/2026-08-07]]
- SL-1528 KPI aggregations / SL-1526 KPI prefs (en develop).
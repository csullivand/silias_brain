# Active Context

## Sesión actual: SL-1528 Feature 6 — KPI aggregations endpoint (2026-08-05/06)
Nota completa: [[Claude Sessions/silia/SL-1528-kpi-aggregations/2026-08-05]]

**Rama `feat/SL-1528-agregaciones-dataset-kpis` — commit Feature 6 `943b6902b` (SIN push):**
- `POST /tables/{tableId}/aggregations` — calcula el VALOR de cada KPI card sobre el set filtrado completo. Complemento de Feature 5 (persistencia).
- Basada en develop (ya trae Feature 5.1 mergeado: `708cfe04c`).

### Qué se hizo
1. **Feature 6 completo:** service `kpiAggregationService.ts` (streamea 1 vez con paginateFilteredRows + filterBuilder reusados; mapea 13 kinds a computeAggregation), handler `postAggregations.ts` (auth table.view, valida vs KPI_KINDS máx 24, 422 en filtro malo), tipos, template `RowsAggregations` (DtRowReadRole), build entry. **17 tests verdes.** pr-review + adversarial = Approved/PASS.
2. **Arquitectura:** DynamoDB (no SQL) -> agregación in-memory. Soft cap `KPI_AGG_MAX_ROWS` (default 50000) -> `partial:true`. Reusé la maquinaria del table assistant (AGE-109/110).
3. **Docs FE (untracked, sin commitear):** `docs/kpi-aggregations-frontend-integration.md` (ES, para Camila) + `docs/kpi-prefs-frontend-integration.md` (EN).

### Contrato
- Body `{filters?, search?, metrics:[{columnId|"__records__", kind}]}` -> `{tableId, rowCount, partial, results:[{columnId, kind, value|null, computable, reason?}]}`.
- Asimetría: metrics usa `columnId`; filters usa `column` (NOMBRE) = payload del listado de filas.
- 0 filas -> porcentajes `null` (no 0).

### Pendiente / dudas
- **Max cards 6 vs 4:** código+PRD+docs = **6** (`KPI_MAX_CARDS=6`); Camila creía 4. A confirmar (cambio de 1 línea si es 4).
- **BUG develop:** Feature 5.1 en develop NO trae los build entries de KpiPrefs -> no despliega. Re-agregados en esta rama; **falta hotfix a develop** (mi fix `c5b4c527b` solo en rama SL-1526).
- Push + PR de SL-1528. Suggestion MEDIUM review: bajar cap default o subir page size (500 RTTs vs límite 29s API GW).

### Contexto previo
- SL-1526 Feature 5 (persistencia KPI prefs) — mergeado a develop. [[Claude Sessions/silia/SL-1526-kpi-prefs-persistence/2026-08-04]]
- Nota útil: `yarn` roto sin token -> `GH_PACKAGES_TOKEN=dummy npx jest`.
# Active Context

## Sesión actual: SL-1576 Feature 2.1 [BE] Row Fill Rules (2026-08-14)
Nota: [[Claude Sessions/silia/SL-1576-row-fill-rules/2026-08-14]]

**Rama `feat/SL-1576-persistencia-reglas-fill` · PR #1991 (base develop) · 4 commits pusheados.**

BE para persistir reglas de coloreado condicional de filas (columna+operador+valor→color), **per-usuario** (por vista, como filtros), espejo de FilterBarConfig.

### Hecho
- Validador propio (`domain/utils/rowFillRules.ts`) — resuelve columna por **id** (no por name como `validateAndResolveFilters`), DROP columnas borradas, REJECT op/shape/color con 400+errorCode. Paleta cerrada de **10** (+orange,cyan,pink,brown).
- Store per-user (PK userId, SK tableId) + GSI tableId-index; GET `table.view`, PUT `table.filter`; 400+errorCode, updatedAt en segundos.
- Cascade-delete al borrar tabla (se suma a KpiPrefs/UserViewConfig en deleteTable).
- Reviews: PR review Approved-with-suggestions + adversarial verify independiente **PASS** (5 lentes).
- Doc FE `docs/row-fill-rules-api.md` **removido del PR** (commit 822190cdc) — se trackea fuera del PR de BE.
- 24 tests row-fill + deleteTable verdes, tsc limpio.

### ⚠️ Pendiente
1. `sam deploy` a dev + round-trip real (tabla nueva se crea con GSI).
2. Tokens de los 4 colores nuevos: los define Design System → pedir a Camila (para el doc FE, fuera del PR).
3. `.changeset/beige-friends-sip.md`: `major` → debería ser `minor`; typo "persit configration".
4. FE integración: único cambio vs contrato original = 422→400 y updatedAt ms→seg.

### Contexto previo
- SL-1432 Feature 7 detail-view reconcile fix. [[Claude Sessions/silia/SL-1432-detail-view-reconcile/2026-08-13]]
- Feature 4.3 (Kanban view config per user) — ticket aparte, NO desarrollado. Doc FE `docs/kanban-view-config-api.md` (untracked).
- SL-1579 Manual escalation. [[Claude Sessions/silia/SL-1579-manual-escalation-debug/2026-08-07]]
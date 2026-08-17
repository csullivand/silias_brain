# Active Context

## Sesión actual: [Feature 4.3][BE] Kanban View Config per user (2026-08-17)
Nota: [[Claude Sessions/silia/Feature-4.3-kanban-view-config/2026-08-17]]

**Análisis entregado. Sin rama aún (en feat/SL-1576). Implementación NO iniciada. 2 decisiones de contrato BLOQUEANTES.**

Feature 4.3 = persistir config de vista Kanban per-(userId,tableId): blob JSON (groupByColumnId, laneOrder, laneColors, showUncategorized, cardTitleColumnId, cardFields[], laneSorts, summaryAggs). Clon del patrón SL-1576 Row Fill.

### Conclusión
- **Molde:** SL-1576 RowFillRules (modelo PK=userId/SK=tableId + GSI tableId-index; handlers GET/PUT /tables/{tableId}/kanban-view-config; cascade fail-open deleteTable.ts:44-57; infra tabla+GSI+KMS+rol). Esfuerzo BAJO ~1 día.
- BE guarda VERBATIM (reconcilia FE). Solo valida enums direction/calcs/__no_status__.

### ⚠️ BLOQUEADO — 2 decisiones de producto
1. **Contrato casa vs doc FE congelado** (docs/kanban-view-config-api.md):
   - updatedAt: doc=ms vs casa=SEGUNDOS · error: doc=422 vs RowFill=400+errorCode · permiso PUT: doc=table.view vs RowFill=table.filter · éxito: doc envuelto vs casa desenvuelto.
   - Rec: alinear a casa + actualizar doc.
2. **Orden manual de cards (mover/arreglar posición):**
   - Mover ENTRE lanes = muta group_by en la fila = DATA compartida, endpoint row-update existente (NO 4.3).
   - Arreglar DENTRO de lane = GAP: contrato solo tiene laneSorts (por columna), no orden manual.
   - ¿per-user (blob laneManualOrder) o COMPARTIDO (rank/position en fila)? ⚠️ per-user en blob crece O(filas) → revienta item 400KB DynamoDB.
   - Rec: COMPARTIDO → rank/position en la fila (fractional/lexo-rank).

### Cómo continuar
1. Resolver las 2 decisiones.
2. Confirmar mover-entre-lanes = endpoint row-update existente.
3. Espejar RowFillRules → KanbanViewConfig; infra SAM + cascade +1 línea; tests.

### Contexto previo
- SL-1576 Feature 2.1 [BE] Row Fill Rules — PR #1991 base develop, reviews PASS, NO deployado. [[Claude Sessions/silia/SL-1576-row-fill-rules/2026-08-14]]
- SL-1432 Feature 7 detail-view reconcile. [[Claude Sessions/silia/SL-1432-detail-view-reconcile/2026-08-13]]
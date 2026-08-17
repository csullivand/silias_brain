# Active Context

## Sesión actual: [Feature 4.3][BE] Kanban View Config (2026-08-17)
Nota: [[Claude Sessions/silia/Feature-4.3-kanban-view-config/2026-08-17]]

**Análisis + decisiones CERRADAS. Sin rama aún. Implementación NO iniciada. Listo para codificar.**

Feature 4.3 = persistir config de vista Kanban (groupByColumnId, laneOrder, laneColors, showUncategorized, cardTitleColumnId, cardFields[], laneSorts, summaryAggs).

### ✅ DECISIONES FINALES (confirmadas por usuario + PRD)
1. **COMPARTIDA por tabla** (NO per-user). Clave solo tableId, sin userId, sin GSI. Usuario: 'el PRD está mal, ya se corrige' (PRD dice por-usuario pero queda invalidado).
2. **Orden manual de tarjetas NO persiste** (PRD lo pone fuera de alcance; Feature 7 = sort automático client-side). Sin rank.
3. **Contrato = convención casa SL-1576:** updatedAt SEGUNDOS, error 400+{errorCode,message}, GET sin config → {config:null,updatedAt:null}.
4. **Permiso:** GET=table.view; PUT=table.column.edit (config compartida = permiso de edición, como reorderColumns). Pendiente confirmar mapeo.

### Molde a copiar
SL-1576 RowFillRules, PERO simplificado: PK=tableId (no userId), SIN GSI, cascade trivial. Handlers GET/PUT /tables/{tableId}/kanban-view-config. Validar solo enums (direction asc/desc, calcs x13, __no_status__). BE guarda VERBATIM (FE reconcilia).

### Cómo continuar (implementar)
1. (Opcional) confirmar PUT=table.column.edit.
2. Modelo KanbanViewConfig (PK=tableId) + util validador + handlers GET/PUT + cascade deleteTable.ts + infra SAM (tabla+KMS+rol, SIN GSI) + tests.
3. Actualizar docs/kanban-view-config-api.md (quitar userId, seg/400).

### Docs a corregir (no-código)
Ticket + PRD (por-usuario→compartida); docs/kanban-view-config-api.md.

### Contexto previo
- SL-1576 Feature 2.1 Row Fill Rules — PR #1991, reviews PASS, NO deployado. [[Claude Sessions/silia/SL-1576-row-fill-rules/2026-08-14]]
- SL-1432 Feature 7 detail-view reconcile. [[Claude Sessions/silia/SL-1432-detail-view-reconcile/2026-08-13]]
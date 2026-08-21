# Active Context

## Sesión actual: [Feature 7] SL-1477 Kanban laneSorts PER-USER + fix board controlado (2026-08-21)
Nota: [[Claude Sessions/silia/SL-1477-kanban-lanesorts-per-user/2026-08-21]]

**Rama feat/SL-1477-persistencia-vista-usuario-kanban · commits 42f4873cc + 098673731 PUSHEADOS · 4e32abb1c LOCAL. NO deployado.**

### Hecho (esta sesión)
1. **laneSorts PER-USER** (única pieza per-user del config Kanban; el resto es compartido por tabla). Store PK=userId/SK=tableId/GSI tableId-index (espeja RowFillRules/KpiPrefs).
2. **BLOCK #1 (42f4873cc):** el GET perdía laneSorts cuando config compartida era null. Fix: lee laneSorts INCONDICIONAL + top-level siempre + FE los inyecta en config efectivo.
3. **BLOCK #2 (098673731):** async-seed race — el `useState(() => ({...config.laneSorts}))` (init de una vez) no re-sembraba cuando el GET resolvía DESPUÉS del mount → sort descartado (reproduce en nav tabla-a-tabla). Fix: **board CONTROLADO** — renderiza config.laneSorts directo, reporta vía onLaneSortsChange; padre hace setKanbanLaneSorts inmediato + debounce save. Quitó useState/useEffect/didHydrateRef.
4. **Tests:** ControlledBoard wrapper + regression guard "applies laneSorts that arrive AFTER mount, no key change". KanbanBoard 73/73, BE Kanban 37/37.
5. **Reviews:** PR review Approved-with-suggestions ⚠️; adversarial PASS ✅ (confirmó que NO se reintrodujo wrong-table-write: el setTimeout captura tableIdParam al agendar).
6. **Fix comentario stale (4e32abb1c, LOCAL):** el comentario de key={tableIdParam} describía el useState seed ya eliminado.
7. **BLOCK de reversibilidad (2da adversarial shadow) → MOOT:** afirmaba pérdida de datos porque #2014 (8d3e5a421, en develop) persistía laneSorts en el config COMPARTIDO. Mecánicamente correcto (GET spread-ea vacío sobre stored.config; PUT escribe laneSorts:{}), PERO el usuario confirmó que NO hay datos reales de lane sort (feature en construcción, nunca deployado con datos). → sin migración/backfill/fallback. Merge OK.

### Commits (SIN docs/Agent/Voice)
- 42f4873cc fix(eca): don't drop per-user laneSorts when a table has no shared config
- 098673731 fix(eca): make Kanban board controlled so async-loaded laneSorts apply
- 4e32abb1c docs(eca): correct stale key={tableIdParam} comment (LOCAL, sin push)

### Pendientes
1. (Opcional) push de 4e32abb1c (fix comentario, quedó local).
2. **Nit NO-bloqueante** (1ra adversarial, correctness): debounce timer en handleLaneSortsChange es un ÚNICO ref compartido — ordenar tabla A, cambiar a B <600ms → save de A cancelado (clearTimeout). Bajo impacto. Fix: flush del save pendiente al cambiar tableIdParam. Usuario aún no decide.
3. (Opcional) sam deploy DynamicTables a dev para round-trip Feature 7.
4. (Deferido) hardening BE: validar select cell por value O label case-insensitive.

### Contexto previo
- Feature 11 SL-1293/SL-1290 Role dropdowns — commits 1f178ffce + 6aa248924 pusheados. [[Claude Sessions/silia/SL-1293-role-dropdowns-casing/2026-08-20]]
- Feature 4.3 Kanban view config compartido (SL-1477) — commit a079d4d0e. [[Claude Sessions/silia/Feature-4.3-kanban-view-config/2026-08-17]]
- SL-1576 Row Fill Rules — PR #1991. [[Claude Sessions/silia/SL-1576-row-fill-rules/2026-08-14]]

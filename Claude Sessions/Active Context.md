# Active Context

## Sesión actual: SL-1526 aislar Feature 5 (KPI prefs) + quitar accountId (2026-08-04)
Nota completa: [[Claude Sessions/silia/SL-1526-kpi-prefs-persistence/2026-08-04]]

**Rama `feat/SL-1526-persistencia-kpis-usuario-tabla` (nueva) — 2 commits SIN push:**
- `bf63ab643` feat(data-views): persist per-user KPI card preferences by table (Feature 5.1) — cherry-pick de `a969064ab` (venía de feat/SL-1272-template-update).
- `273d17a11` refactor: drop unused accountId from KPI prefs.

### Qué se hizo hoy
1. **Recuperación de "archivos perdidos":** los 4 access-control docs (flows/roadmap/system-overview/permission-mapping-for-pms) NO se perdieron por cambio de rama — están en `.gitignore` (líneas 95-98, "keep local"). Restaurados a disco desde `fix/SL-1289-default-roles-migration`, pero son untracked-ignored (no commiteables). Scripts pr-review/adversarial-verify tampoco estaban perdidos (viven en develop).
2. **Aislé Feature 5 en rama SL-1526:** basada en develop + cherry-pick del commit KPI. (El trabajo estaba solo en feat/SL-1272-template-update.)
3. **Quité `accountId`** del modelo KPI prefs (Option A): nunca se usa (key = (userId, tableId)), y se tomaba del authorizer = cuenta home → mal para superadmin en cuenta cambiada (SL-1278). Alineado con PRD spec. 19/19 tests. Commit + pr-review + adversarial = **Approved/PASS**.
4. **Doc FE** creado `docs/kpi-prefs-frontend-integration.md` (untracked, usuario pidió NO commitear docs).

### Contrato con Feature 6 (ticket aparte)
- SL-1526 = Feature 5 (persistencia: guarda QUÉ cards). Feature 6 = `POST /data-views/:id/aggregations` (calcula los VALORES) — pendiente, ticket separado.
- Compartido: `KPI_KINDS` (13) + `KPI_RECORDS_ID='__records__'`. **Feature 6 debe importar de `DynamicTables/domain/types/tables.types.ts`, no redeclarar** (evitar drift).
- Gaps de producto/FE (no bloquean SL-1526): defaults-seeding, validación calc-vs-tipo-columna, poda de cards con columna borrada.

### Notas útiles
- `yarn` roto sin `GH_PACKAGES_TOKEN` → usar `GH_PACKAGES_TOKEN=dummy npx jest --testPathPattern="..."`.

### Otras ramas SIN push (contexto previo)
- fix/SL-1557-validacion-estricta — RBAC estricto + account 403 fix. [[Claude Sessions/silia/SL-1557-rbac-strict/2026-08-04]]
- feat/SL-1272-template-update (tiene el a969064ab original + otras cosas), feat/SL-1545-role-count, feat/SL-1545-role-batch-uodate.

### Pendiente viejo
- escalation-inbox STAGING 500: redeploy Assistant→staging. [[project_casl_iam_grants_gap]]
- SL-1557: abrir createOrgUser/patchOrgUser a roles custom (resuelve CSV import + Feature 11 FE).
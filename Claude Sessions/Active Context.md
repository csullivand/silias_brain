# Active Context

## Sesión actual: Feature 5.1 KPI prefs (BE) + contexto agent-access 9.1 (2026-08-03)
Nota completa: [[Claude Sessions/silia/Feature-5.1-kpi-prefs/2026-08-03]]

**Feature 5.1 commiteado `a969064ab` (9 files +732) SIN PUSH, en rama `feat/SL-1272-template-update`.** 19 tests verdes, tsc limpio, YAML validado.

### Feature 5.1 — KPI prefs por usuario y tabla (HECHO)
- GET/PUT `/tables/{tableId}/kpi-prefs` — config de KPI cards por (usuario, tabla), cross-device, aislada por usuario.
- Modelo `DynamicTableKpiPrefs` clave {userId, tableId} + GSI tableId-index; `normalizeCards` (retrocompat enabled→true/unit→auto); cleanup on deleteTable (fail-open).
- GET: vacío (no 404) si no configuró; cards tal cual (colgantes se quedan, FE las omite). PUT: valida ≤6 cards, kind/unit enum, __records__ único; acepta colgantes.
- Auth **table.view** (config personal, no permiso nuevo). Espejé `FilterBarConfig`.
- Infra: KMS+tabla(GSI)+rol+2 funciones+Globals env+perms cleanup en DtTableWriteRole. Build webpack auto-descubre los handlers.
- **kind enum del PRD F6** — sync con kanban.ts cuando Camila portee 1.1 (comentario lo marca).

### Contexto agent-access (9.1 de David) → `docs/agent-access-model.md` (untracked)
- 'Menú del agente' = ShareAccessModal compartido → /access/grants (object_type=agent). Modelo DUAL: **AccessGrant** (canónico con roleId) vs **AgentAccess** (solo wizard, sin rol). Resolver YA existe: `PermissionResolver.resolveEffectivePermissions` (directo>team>herencia). 9.1 = decisión de unificar en AccessGrant, no crear sharedUsers nuevo.

### 1.1 [FE] KpiCards
NO necesita backend (puro, computa sobre props). El feature sí: Feature 6 (agregación BE sobre dataset paginado) + Feature 5 (persistencia = esto).

### Continuar por
1. Feature 5.1: mover a rama propia antes de PR; push cuando el usuario diga; pr-review/adversarial si se pide.
2. Sync enum kind con kanban.ts (1.1).
3. 9.1: David define unificación con dueño del PRD + forma del resolver para 8.1.

### Ramas abiertas (sesiones previas)
- feat/SL-1545-role-count — enrichment role list (userCount + sections/módulos PRD). [[Claude Sessions/silia/SL-1545-role-count/2026-07-30]]
- feat/SL-1545-role-batch-uodate — fix timeout PUT /role (batch writes). [[Claude Sessions/silia/SL-1545-role-batch-update/2026-07-31]]

### Pendiente viejo
- escalation-inbox STAGING 500: redeploy Assistant→staging. [[project_casl_iam_grants_gap]]
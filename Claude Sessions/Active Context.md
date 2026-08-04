# Active Context

## Sesión actual: SL-1557 RBAC estricto + Feature 11 análisis + account 403 fix (2026-08-04)
Nota completa: [[Claude Sessions/silia/SL-1557-rbac-strict/2026-08-04]]

**Rama `fix/SL-1557-validacion-estricta` — 3 commits SIN push:**
- `4f509fe9a` roles anti-escalamiento (findUngrantableResourceIds + assertRole floor + FE excluye god-mode).
- `7d2de6a79` flag PERMISSION_STRICT_MODE (14 templates).
- `f9fe20f96` account graduated payload (fix 403 operator).

### Qué se hizo hoy
1. **SL-1557 (commiteado, cambios del usuario):** un caller no puede autorizar un rol con permisos > los suyos (bloquea 'clonar Super Admin'); piso assertRole en handlers de roles; flag PERMISSION_STRICT_MODE por ambiente (enforce vs log). assertRole SIGUE vigente (95 handlers, 53 usan ambos).
2. **Fix 403 GET /accounts/{id} (commiteado f9fe20f96):** el operator recibía 403 al arranque porque AccountContext hace GET /accounts/{id} para cargar nombre/status, y el endpoint exigía account.information.view (admin+). Solución: **payload graduado** — piso canAccessAccount + full/summary según permiso REAL (no según flag cliente) + ?view=summary opcional (solo baja) + scope en respuesta. FE no cambia. Test 6 casos.

### Gaps analizados (NO implementados aún)
- **CSV import:** mismatch FE↔BE — FE preview acepta roles custom pero createOrgUser/patchOrgUser solo aceptan [ADMIN,SUPERVISOR,OPERATOR] → filas custom fallan al crear.
- **Feature 11 [FE] dropdowns roles custom:** backend PARCIAL. Endpoint de roles ✅, grants/share access ✅, PERO **crear/editar usuario con rol custom ❌** (mismo gap del CSV). = EL gap central.
- **Pendiente clave (resuelve CSV + Feature 11):** abrir createOrgUser/patchOrgUser a roles custom de la cuenta, manteniendo bloqueo god-mode + guards SL-1557.

### Nota pre-existente
Accounts/application/get/getById.test.ts roto de antes (require .handler que ya no existe). No es de mi cambio.

### Otras ramas SIN push
- feat/SL-1272-template-update — Feature 5.1 KPI prefs (a969064ab). [[Claude Sessions/silia/Feature-5.1-kpi-prefs/2026-08-03]]
- feat/SL-1545-role-count, feat/SL-1545-role-batch-uodate.

### Pendiente viejo
- escalation-inbox STAGING 500: redeploy Assistant→staging. [[project_casl_iam_grants_gap]]
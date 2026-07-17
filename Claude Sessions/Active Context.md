# Active Context

## Sesión actual: SL-1289 — Feature 10 roles + hardening de seguridad (2026-07-16/17)
Nota completa: [[Claude Sessions/silia/feature-10-roles/2026-07-16]]

**Rama:** fix/SL-1289-default-roles-migration (~23 commits, PR NO abierto).
**Estado:** implementación completa + cadena de reviews cerrada (6 hallazgos reales). Review PASS ✅ / Adversarial PASS ✅. Falta DEPLOY + SEED + PR.

### Qué se hizo
- Feature 10 BE completo (12 subtareas): modelo roles custom por cuenta, CRUD+clone+catálogo+audit, resolución account-aware (S12 desbloquea Feature 11), invalidación efectivos, seed.
- Contadores efectivos con herencia (management + teams).
- Fixes de sesión: TDZ en listOrgUsers, visibilidad fail-closed en folders, logging getLambdaLogger.
- **Cadena de seguridad (6 fixes reales):** stale-name invalidation, seed account-scope (evita lockear rol de cliente), IDOR en create/update/delete/clone, inmutabilidad pre-seed por nombre, fugas de lectura (get/get-permissions/permission-get), **escalada de privilegios vía /permission CRUD**, empty-name en update. CMK dedicada para RoleAuditLog (patrón Billing).

### Continuar por (en orden)
1. DEPLOY infra (Roles: GSI accountId-name-index ACTIVE, RoleAuditLog+CMK, clone fn, envs; Teams: FOLDERS/CHATBOT envs) + código Lambdas. Verificar cloneMin.js.
2. Correr scripts/migrations/seed-default-roles.ts (pobla isDefault).
3. Verificar: management counts≠0, teams herencia, visible_to=me con operador, tenant-isolation en /role y /permission.
4. Abrir PR a develop.

### Bloqueos de producto (datos, no código)
- Matriz ~102 permisos validada → seed rol→permiso + category.
- 'Implementador' (PRD) vs superuser/superadmin (enum).
- Canal Slack alerta >20 roles.

### Follow-ups documentados
- get.ts findAll() scan → optimizar (defaults no están en el GSI).
- rename de rol no cascadea a user.role → guardar roleId en user (Feature 11).
- Roles IAM dedicados del template definidos-pero-no-cableados (ruido).
- Feature 11 (asignación) · KMS Opción A para conteo de grants en delete.